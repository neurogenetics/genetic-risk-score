# Genetic-risk-score calculation

Most common and easiest way to do this in plink

https://www.cog-genomics.org/plink/1.9/score

another way to do this is using PRSice -> https://choishingwan.github.io/PRSice/ 

# Plink commands

`module load plink #if on biowulf`

`plink --bfile $yourfile --score $scorefile --out $outputfilename`

$yourfile = standard binary files .bed .bim .fam

$outputfilename = whatever you want it to be

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
