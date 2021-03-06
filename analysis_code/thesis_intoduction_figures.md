---
title: "Figures for Thesis Introduction"
author: "Analysis by Rachel Spicer, github:RASpicer"
date: "22/09/2018"
output: 
  html_document:
    code_folding: hide
    number_sections: yes
    theme: cerulean
    keep_md: true
---



# Omics Publications per Year

Data were obtained from Web of Science by searching articles for ‘omic’ OR ‘ome’ for each omic i.e. DOCUMENT TYPES: (ARTICLE) AND ‘metabolomic’ OR ‘metabolome’ etc. Data were collected on the 12<sup>th</sup> January 2018.


```r
# Read csv of frequency of omics publications per year
Omics <- read.csv("../data/omics.csv", check.names = FALSE)

# Replace missing values with 0
Omics[is.na(Omics)] <- 0

# Melt data for plotting
Omics1998 <- melt(Omics[5:24, ])
Omics1998$value <- as.numeric(Omics1998$value)
colnames(Omics1998)[2] <- "Omic"
```

## The increase in the number of ‘omic’ publications since 1998


```r
ggplot(Omics1998, aes(x = Year, y = value, fill = Omic, shape = Omic)) + 
  geom_point(aes(colour = Omic))+
  scale_y_log10(breaks = c(1,10,100,1000,10000), "No. Journal Articles Published") +
  annotation_logticks(sides = "l")+
  scale_color_manual(values = c("#D55E00","#0072B2", "#009E73", "#CC79A7")) +
  scale_x_discrete(breaks=c("2000","2005","2010","2015")) +  theme_bw() +
  theme(
    axis.text = element_text(colour = "black"),
    axis.line.x = element_line(color="black", size = 0.5),
    axis.line.y = element_line(color="black", size = 0.5),
    strip.text.x = element_text(size = 8),
    # Remove gridlines and borders
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    panel.border = element_blank(),
    panel.background = element_blank())
```

<img src="figs/omicsyear-1.png" style="display: block; margin: auto;" />

# Publicly available metabolomics datasets with raw data available to download

Data were obtained on the 23<sup>rd</sup> January 2018.

```r
# Read data
ML <- read.csv("../data/ML.csv", stringsAsFactors = FALSE, check.names = FALSE)
MW <- read.csv("../data/MW.csv", stringsAsFactors = FALSE, check.names = FALSE)
GNPS <- read.csv("../data/GNPS.csv", stringsAsFactors = FALSE, check.names = FALSE)
MeRyB <- read.csv("../data/MeRyB.csv", stringsAsFactors = FALSE, check.names = FALSE)
MP <- read.csv("../data/MetaPhen.csv", stringsAsFactors = FALSE, check.names = FALSE)

# Extract only studies with raw data Metabolomics Workbench
Rawdata <- grep("\\*", MW$Download)
RawdataMW <- MW[Rawdata, ]
# Extract study publication year
RawdataMW$Year <- gsub("^.*/", "", RawdataMW$ReleaseDate)

# Extract GNPS study publication year
GNPS$Year <- sapply(strsplit(GNPS$`Upload Date`, ","), function(x) x[2])
# Remove spaces
GNPS$Year <- gsub(" ", "", GNPS$Year)

# Extract Public MetaboLights studies
PublicML <- ML[ML$Status == "Public", ]
# Extract study publication year
PublicML$Year <- gsub("^.*/", "", PublicML$StudyPublicationDate)
# Add 20 to year for format 20XX
PublicML$Year <- paste0("20", PublicML$Year)

# Extract number of studies released per year
MLYear <- as.data.frame(table(PublicML$Year))
MWYear <- as.data.frame(table(RawdataMW$Year))
GNPSYear <- as.data.frame(table(GNPS$Year))
MPYear <- as.data.frame(table(MP$Year))
MeRyBYear <- as.data.frame(table(MeRyB$Year))

# Change column names
colnames(MLYear) <- c("Year", "MetaboLights")
colnames(MWYear) <- c("Year", "Metabolomics Workbench")
colnames(GNPSYear) <- c("Year", "GNPS")
colnames(MPYear) <- c("Year", "MetaPhen")
colnames(MeRyBYear) <- c("Year", "MeRy-B")

# Combine data frames
OpenStudies <- join_all(list(GNPSYear, MeRyBYear, MLYear, MWYear, MPYear), by = "Year", 
    type = "full")
# Replace NAs with 0
OpenStudies[is.na(OpenStudies)] <- 0
# Reorder dates
OpenStudies$Year <- as.numeric(as.character(OpenStudies$Year))
OpenStudies <- OpenStudies[order(OpenStudies$Year), ]

# Calculate cumulative values
MLCum <- cumsum(OpenStudies$MetaboLights)
MWCum <- cumsum(OpenStudies$`Metabolomics Workbench`)
GNPSCum <- cumsum(OpenStudies$GNPS)
MPCum <- cumsum(OpenStudies$MetaPhen)
MeRyBCum <- cumsum(OpenStudies$`MeRy-B`)

# Combine data frames of culumative values
OpenStudiesCum <- as.data.frame(cbind(OpenStudies[, 1], GNPSCum, MeRyBCum, MLCum, 
    MWCum, MPCum))
colnames(OpenStudiesCum) <- c("Year", "GNPS", "MeRy-B", "MetaboLights", "Metabolomics Workbench", 
    "MetaPhen")

# Melt data frame
OpenStudiesCumM <- melt(OpenStudiesCum[6:nrow(OpenStudiesCum), ], id = "Year")
OpenStudiesCumM$value <- as.numeric(OpenStudiesCumM$value)
colnames(OpenStudiesCumM)[2] <- "Repository"

# Create duplicate dataframe
OpenStudiesCumP <- OpenStudiesCum
# Calculate the total number of datasets oer year
OpenStudiesCum$Total <- rowSums(OpenStudiesCum[2:ncol(OpenStudiesCum)])
# Calculate the percentage coverage by each repository per year
OpenStudiesCumP$GNPS <- OpenStudiesCum$GNPS/OpenStudiesCum$Total * 100
OpenStudiesCumP$`MeRy-B` <- OpenStudiesCum$`MeRy-B`/OpenStudiesCum$Total * 100
OpenStudiesCumP$MetaboLights <- OpenStudiesCum$MetaboLights/OpenStudiesCum$Total * 
    100
OpenStudiesCumP$`Metabolomics Workbench` <- OpenStudiesCum$`Metabolomics Workbench`/OpenStudiesCum$Total * 
    100
OpenStudiesCumP$MetaPhen <- OpenStudiesCum$MetaPhen/OpenStudiesCum$Total * 100

# Melt data frame
OpenStudiesCumPM <- melt(OpenStudiesCumP[6:nrow(OpenStudiesCumP), ], id = "Year")
OpenStudiesCumPM$value <- as.numeric(OpenStudiesCumPM$value)
colnames(OpenStudiesCumPM)[2] <- "Repository"
```

## The total number of publicly available metabolomics datasets
Plots are coloured by repository. Figure is inspired by Figure 5 from Aksenov et al. (2017) [Global chemical analysis of biology by mass spectrometry](https://www.nature.com/articles/s41570-017-0054).


```r
ggplot(OpenStudiesCumM, aes(x = Year, y = value, fill = Repository)) + 
  geom_area()+
  scale_fill_manual(values = c("#377eb8","#ff7f00", "#4daf4a", "#984ea3", "#e78ac3"))+
  ylab("Publicly Available Datasets") +
  scale_x_discrete(limit = c(2007, 2009, 2011, 2013, 2015, 2017)) +
  scale_y_continuous(expand = c(0, 0), limits = c(0, 1600)) +
  theme_bw() +
  theme(
    axis.text = element_text(colour = "black", size = 6),
    axis.line.x = element_line(color="black", size = 0.5),
    axis.line.y = element_line(color="black", size = 0.5),
    strip.text.x = element_text(size = 8),
    # Remove gridlines and borders
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    panel.border = element_blank(),
    panel.background = element_blank(),
    legend.position="top")
```

<img src="figs/freqyear-1.png" style="display: block; margin: auto;" />

## The relative percentage of datasets per metabolomics repository
Plots are coloured by repository. Figure is inspired by Figure 5 from Aksenov et al. (2017) [Global chemical analysis of biology by mass spectrometry](https://www.nature.com/articles/s41570-017-0054).


```r
ggplot(OpenStudiesCumPM, aes(x = Year, y = value, fill = Repository)) + 
  geom_area()+
  scale_fill_manual(values = c("#377eb8","#ff7f00", "#4daf4a", "#984ea3", "#e78ac3"))+
  ylab("Publicly Available Datasets (%)") +
  scale_x_discrete(limit = c(2007, 2009, 2011, 2013, 2015, 2017)) +
  scale_y_continuous(expand = c(0, 0), limits = c(0, 101)) +
  theme_bw() +
  theme(
    axis.text = element_text(colour = "black", size = 6),
    axis.line.x = element_line(color="black", size = 0.5),
    axis.line.y = element_line(color="black", size = 0.5),
    strip.text.x = element_text(size = 8),
    # Remove gridlines and borders
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank(),
    panel.border = element_blank(),
    panel.background = element_blank(),
    legend.position="top")
```

<img src="figs/peryear-1.png" style="display: block; margin: auto;" />

# Size of Data

Caluculate the total size of data per repository.


```r
# Find the size of data in repositories GPNS
GNPSsizeKB <- sum(GNPS$`Total Size (KB)`)

# MetaboLights
MLsizeMB <- sum(PublicML$`Size (MB)`)

# Metabolomics Workbench Extract study sizes
RawdataMW$StudySize <- gsub("Uploaded data \\(|\\)\\*", "", RawdataMW$Download)
# Convert to KB
RawdataMW$StudySizeKB <- ifelse(grepl("G", RawdataMW$StudySize) == T, paste0(gsub("[^0-9]", 
    "", RawdataMW$StudySize), "000000"), ifelse(grepl("M", RawdataMW$StudySize) == 
    T, paste0(gsub("[^0-9]", "", RawdataMW$StudySize), "000"), ifelse(grepl("K", 
    RawdataMW$StudySize) == T, paste0(gsub("[^0-9]", "", RawdataMW$StudySize), 
    ""), "Other")))
# Convert class to numeric
RawdataMW$StudySizeKB <- as.numeric(RawdataMW$StudySizeKB)
# Total data size
MWsizeKB <- sum(RawdataMW$StudySizeKB)
```
