
# Documentation and softare for the "Predicting drug-pair synergy from the predicted synergy probabilities of individual drugs"

This file provides a high-level overview of the steps needed to reproduce the results in drug-pair synergy publication. The three primary steps are:

1. Download and process the data
2. Run the single drug-based prediction code
3. Run the drug pair-based prediction code

## Download and preprocess the data

### Cell line gene expression features

The cell line gene expression features are the RNA Seq RPKM values (provided by the CCLE) for the LINC1000 genes. The CCLE RNA Seq RPKM values are available [here](https://portals.broadinstitute.org/ccle); note that you will need to create an account and login to access the CCLE data. The data file that should be downloaded is called: `CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct`. A perl script, `CCLE_format_rnaseq.pl`, is provided to format the CCLE data and extract the LINCS100 gene expression values corresponding to the LINCS1000 genes. After downloading the CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct file, run and redirect the script:

`CCLE_format_rnaseq.pl CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct > <destination file name>`

to produce a tab-delimited file with cell-line samples corresponding to the rows and gene names corresponding to the column. Note that some of the LINCS1000 gene names are not found in the CCLE intput file (which will generate `Failed to match LINCS gene` warnings. However, a total of 958 genes should be matched.

### Drug features

The drug features are the 1021-bit binary fingerprints computed from desalted, 2D chemical structres using the [OpenBabel program](http://openbabel.org/wiki/Main_Page). The 2D drug structures for both the NCI-ALMANAC and Merck datasets are contained in the `ALMANAC_and_Merck.sdf` SDF file. The fingerprints, computed using the following commands: 

```obabel ALMANAC_and_Merck.smiles -ofps -OALMANAC_and_Merck.obable.FP2 -xfFP2`
cat ALMANAC_and_Merck.obable.FP2 | ./expand_fingerprints.pl > ALMANAC_and_Merck.obable.FP2.csv```

are provided in the file `ALMANAC_and_Merck.obable.FP2.csv`. The `expand_fingerprints.pl` script converts the hex-encoded drug fingerprints into a text-based binary fingerprint (i.e. strings of 1's and 0's).

  * **Drug synergy data**
  
    * **NCI-ALMANAC synergy data**
    * **Merck synergy data**

2. **Run the single drug-based synergy prediction code**

3. **Run the drug pair-based synergy prediction code**

