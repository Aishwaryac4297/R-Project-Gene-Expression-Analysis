# 🧬 Gene Expression Analysis in R

---

## 📘 Overview

This R project focuses on the **import, cleaning, reshaping, and visualization** of gene expression data, particularly related to cancer research. It uses metadata and gene expression data to explore patterns based on tissue types and metastasis status.

---

## 📁 Project Files

- `R Project Gene Expression csv file.csv` – Metadata from GEO
- `Gene_new.csv` – Gene expression data
- `README.md` – Documentation (this file)

---

## 🔧 Libraries Used

```r
library(dplyr)
library(tidyverse)
install.packages("BiocManager")
BiocManager::install("GEOquery")
library(GEOquery)

```
## 📂 Workflow Steps
### 1️⃣ Load Data
setwd("C:/Users/Desktop/project+notes")
Bio_new <- read.csv("R Project Gene Expression csv file.csv")
Gene_new <- read.csv("Gene_new.csv")

### 2️⃣ Prepare Metadata
metadata <- Bio_new %>%
  select(title, characteristics_ch1, characteristics_ch1.1, description)

metadata.modified <- metadata %>%
  rename(
    tissue_raw = characteristics_ch1,
    metastasis_raw = characteristics_ch1.1
  ) %>%
  mutate(
    tissue = gsub("tissue:\\s*", "", tissue_raw),
    metastasis = gsub("metastasis:\\s*", "", metastasis_raw)
  ) %>%
  select(title, tissue, metastasis, description)
  
### 3️⃣ Prepare Gene Expression Data
data.long <- Gene_new %>%
  rename(Gene_Name = Gene) %>%
  gather(key = 'sample', value = 'FPKM', -Gene_Name)

### 4️⃣ Merge Metadata with Expression
data.long1 <- data.long %>%
  left_join(metadata.modified, by = c("sample" = "description"))

### 5️⃣ Summary Statistics
DM <- data.long1 %>%
  group_by(Gene_Name, title) %>%
  summarize(mean_FPKM = mean(FPKM), median_FPKM = median(FPKM)) %>%
  arrange(mean_FPKM)

## 📊 Visualizations
- ### Bar Plot
data.long1 %>% ggplot(.,aes(x=sample,y=FPKM,fill=metastasis))+geom_col()+theme(axis.text.x = element_text(angle = 45, hjust = 1))

![Image](https://github.com/user-attachments/assets/c4272293-a322-4864-b198-bd7bee1bd634)

- ### Density Plot
ggplot(data.long1, aes(x = sample, fill = metastasis)) +
  geom_density(alpha = 0.3)+theme(axis.text.x = element_text(angle = 45, hjust = 1))
  
![Image](https://github.com/user-attachments/assets/baf14654-5965-4533-a3ca-eb153f4c9294)

- ### Scatter Plot
data.long1 %>%
  spread(key = Gene_Name, value = FPKM) %>%
  ggplot(aes(x = SCYL3, y = TSPANG)) +
  geom_point()

![Image](https://github.com/user-attachments/assets/62d4a8fc-61b9-47ab-b439-5918ed1030b6)



