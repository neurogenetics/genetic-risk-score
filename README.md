# Genetic-risk-score calculation

Most common and easiest way to do this in plink

https://www.cog-genomics.org/plink/1.9/score

another way to do this is using PRSice -> https://choishingwan.github.io/PRSice/ 

## Plink commands

`module load plink #if on biowulf`

`plink --bfile $yourfile --score $scorefile --out $outputfilename`

$yourfile = standard binary files .bed .bim .fam

$outputfilename = whatever you want it to be, the output will have the extension .profile

$scorefile = file with variant-name, allele and score-value:

~~~~
1:154898185	C	0.2812
1:155135036	A	0.6068
1:155205634	T	-0.7467
1:161469054	C	0.065
~~~~

see in this thread the risk score file for Parkinson's disease based on the most recent PD GWAS (META5) Nalls et al 2019

META5_GRS_chr_bp.txt -> has variants structured as chromosome:basepair

META5_GRS_RSid.txt -> has variants structured as RS-IDs

## optional follow-up analyses using case-control data

`module load R #if on biowulf`
~~~~
R
library(dplyr)
# load in data from the step above
data1 = read.table("$yourfile.profile",header=T)
# also load in covariates
data2 = read.table("$yourfile_covariates.txt",header=T)
# drop the IID column to prevent double columns
data2$IID <- NULL
# merge by FID column, can also make shorter in this case -> by='FID'
MM = merge(data2,data1,by.x='FID',by.y='FID')
temp <- anti_join(data1, data2, by='FID')
data <- rbind(temp, MM)
# convert score values to Z-score
meanGRS <- mean(data$SCORE)
sdGRS <- sd(data$SCORE)
data$SCOREZ <- (data$SCORE - meanGRS)/sdGRS
# boxplot case-control data 
pdf("$FILENAME.pdf",width=4)
boxplot(data$SCOREZ~data$PHENO.x,col=c('powderblue', 'pink1'),xlab="0 control, 1 PD-case",ylab="GRS Z-score")
grid()
dev.off()
# calculate if the GRS is statistical significant between cases and controls using AGE,
# SEX and first 5 principal components (PC)
thisFormula1 <- formula(paste("PHENO ~ SCOREZ + SEX_COV + AGE + PC1 + PC2 + PC3 + PC4 + PC5"))
model1 <- glm(thisFormula1, data = data, ,family=binomial)
print(summary(model1))
~~~~

