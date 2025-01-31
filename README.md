# MetaMinimac2
MetaMinimac2 is an efficient tool to combine genotype data imputed against multiple reference panels.

## Installation
Prerequisite: [cget](https://cget.readthedocs.io/en/latest/index.html)>=0.1, [cmake](https://cmake.org)>=3.2
```bash
git clone https://github.com/yukt/MetaMinimac2.git
cd MetaMinimac2
bash install.sh
```

## Usages
The first step is to impute _**pre-phased**_ target haplotypes against reference panels separately using [Minimac4](https://github.com/statgen/Minimac4) with `--meta` option on, which will generate a `.empiricalDose.vcf.gz` file (which is required for MetaMinimac2) in addition to the `.dose.vcf.gz` file. Please see http://genome.sph.umich.edu/wiki/Minimac4 for detailed documentation for Minimac4.

```bash
minimac4 --refHaps refPanelA.m3vcf \
         --haps targetStudy.vcf \
         --prefix PanelA.imputed \
         --meta
         
minimac4 --refHaps refPanelB.m3vcf \
         --haps targetStudy.vcf \
         --prefix PanelB.imputed \
         --meta
```

The second step is to integrate the imputed results using MetaMinimac2.

```bash
MetaMinimac2 -i PanelA.imputed:PanelB.imputed -o A_B.meta.testrun
```

## Options
```
-i, --input  <prefix1:prefix2 ...>  (Required) Colon-separated prefixes of input data to meta-impute
-o, --output <prefix>               (Required) Output prefix [MetaMinimac.Output.Prefix]
-f, --format <string>               Comma-separated output FORMAT tags [GT,DS,HDS]
-p, --skipPhasingCheck              OFF by default. If ON, program will skip phasing consistency check before analysis. 
-s, --skipInfo                      OFF by default. If ON, the INFO fields are removed from the output file
-n, --nobgzip                       OFF by default. If ON, output files will NOT be bgzipped
-w, --weight                        OFF by default. If ON, weights will be saved in [MetaMinimac.Output.Prefix].metaWeights(.gz)
-l, --log                           OFF by default. If ON, log will be written to $prefix.logfile
-h, --help                          OFF by default. If ON, detailed documentation on options and usage will be displayed
```


## Output Files
The meta-imputed result will be saved in `[MetaMinimac.Output.Prefix].metaDose.vcf.gz`.

When `--weight` is ON, the weights for meta-imputation will be saved in `[MetaMinimac.Output.Prefix].metaWeights.gz`. The weight file is also in VCF format, which is good for individual filtering by [vcftools](https://vcftools.github.io) or [bcftools](http://samtools.github.io/bcftools/bcftools.html). In the format field, `WT1` stands for the weight on reference panel 1 (which is related to input files with prefix1), `WT2` stands for the weight on reference panel 2, etc.

## Imputation Server
[Michigan Imputation Server](imputationserver.sph.umich.edu) offers the option to generate `.empiricalDose.vcf.gz` file for the convenience of downstream meta-imputation using MetaMinimac2. 

Imputation steps: 
1. Choose `Genotype Imputation (Meta-Imputation Option)` in the `Run` tab;
2. Set up the reference panel, input files, etc. following the [instruction here](https://imputationserver.readthedocs.io/en/latest/getting-started/);
3. Tick the checkbox `Generate Meta-imputation file` before `Submit Job`;
4. After the job has finished, imputation results can be downloaded from the server. The zip archive contains both `.dose.vcf.gz` and `.empiricalDose.vcf.gz` files.

[TOPMed Imputation Server](https://imputation.biodatacatalyst.nhlbi.nih.gov) will stage the same option soon.


## Important Notes
### 1. The reference panels must be on the same build.
The reference panels used for meta-imputation must be on the same build. For example, for meta-imputation using 1000G and TOPMed, user should impute using [1000G GRCh38](https://www.internationalgenome.org/announcements/updated-GRCh38-liftover/) instead of the original GRCh37 build. 

### 2. Phasing must be consistent across input files.

If the phasing is different between input imputed files, the resulting meta dosage (which is supposed to be a weighted average of imputed dosages on the same haplotype) will be messed up. Therefore, we highly recommend that users always keep `--skipPhasingCheck` OFF  to avoid any risk.

The best practice is to do the phasing first and use the same pre-phased vcf file for imputation against different reference panels. 

An alternative is to use `.empiricalDose.vcf.gz` file from imputation using one reference panel as input vcf file for imputation using other reference panels. 

For example, when the phasing step is performed by [Michigan Imputation Server](imputationserver.sph.umich.edu) where the phased vcf file is not provided as output, user can use the output `.empiricalDose.vcf.gz` file for imputation using other reference panels. 

Please note that `.empiricalDose.vcf.gz` file contains overlapping sites between the genotype array and the reference panel in use, and does not include the sites existing in the genotype array only, so the results generated by imputation using empiricalDose.vcf could be different from that using a pre-phased vcf including all genotyped sites.

