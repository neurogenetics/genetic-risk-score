# Genetic Risk Score Calculation

- **Date Written:** 22.04.2019
- **Date Last Updated:** 23.04.2019

## Description
Genetic risk scores (AKA: polygenic scores, polygenic risk scores, or genome-wide scores) is a summary measure of a set of risk-associated genetic variants and can be easily calculated using PLINK

## Requirements 
1. **PLINK** (v1.9 is used here)
	 - The most common and easiest way to do this in PLINK ([v1.9](https://www.cog-genomics.org/plink/1.9/score))
	 - Another way to do this is using [PRSice](https://choishingwan.github.io/PRSice/)
2. **PLINK Binary Files** (`.bed`, `.bim`, `.fam`) 
3. **Score File** 
	- This is a file with a variant identifier, allele, and an associated score value
	- These scores come from GWA Studies 
	- This [repository](https://github.com/neurogenetics/genetic-risk-score) has the risk score file for Parkinson's disease based on the most recent PD GWAS (META5) [Nalls _et al._, 2019](https://www.biorxiv.org/content/10.1101/388165v3)
		- `META5_GRS_chr_bp.txt` has variants structured as chromosome:basepair
		- `META5_GRS_RSid.txt` has variants structured as RS-IDs
	- The format of the file should match this example: 
```
1:154898185	C	0.2812
1:155135036	A	0.6068
1:155205634	T	-0.7467
1:161469054	C	0.065
```



## PLINK Commands

```bash 
module load plink #if on Biowulf, this loads v1.9
plink --bfile $yourfile --score $scorefile --out $outputfilename
```
Where: 
- `$yourfile` = standard binary file prefix (will point to `.bed`, `.bim`, and `.fam` files)
- `$outputfilename` = whatever you want it to be, the output will have the extension `.profile`
- `$scorefile` = file with variant-name, allele and score-value


## Optional Follow-Up Analyses (Using Case-Control Data)

```bash, R
module load R #if on Biowulf

# Load in the necessary packages 
library(dplyr)

# Load in data from the step above
data1 = read.table("$yourfile.profile",header=T)

# Also load in covariates (this can be done in PLINK or using flashPCA)
data2 = read.table("$yourfile_covariates.txt",header=T)

# Drop the IID column to prevent double columns
data2$IID <- NULL

# Merge by FID column, can also make shorter in this case -> by='FID'
MM = merge(data2,data1,by.x='FID',by.y='FID')
temp <- anti_join(data1, data2, by='FID')
data <- rbind(temp, MM)

# Convert score values to Z-score
meanGRS <- mean(data$SCORE)
sdGRS <- sd(data$SCORE)
data$SCOREZ <- (data$SCORE - meanGRS)/sdGRS

# Generate boxplot case-control data 
pdf("$FILENAME.pdf",width=4)
boxplot(data$SCOREZ~data$PHENO.x,col=c('powderblue', 'pink1'),xlab="0 control, 1 PD-case",ylab="GRS Z-score")
grid()
dev.off()

# Calculate if the GRS is statistical significant between cases and controls using AGE, SEX and first 5 principal components (PC)
thisFormula1 <- formula(paste("PHENO ~ SCOREZ + SEX_COV + AGE + PC1 + PC2 + PC3 + PC4 + PC5"))
model1 <- glm(thisFormula1, data = data, ,family=binomial)
print(summary(model1))

# note sometimes the P-value shows this < 2e-16
# then use this : summary(model1)$coefficients[,4] 
# this will output the real P-value
```
