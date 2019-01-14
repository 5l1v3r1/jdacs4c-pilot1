
# Documentation and softare for the "Predicting drug-pair synergy from the predicted synergy probabilities of individual drugs"

This file provides a high-level overview of the steps needed to reproduce the results in drug-pair synergy publication. The three primary steps are:

1. Download and process the data
2. Run the single drug-based prediction code
3. Run the drug pair-based prediction code

## Download and preprocess the data

### Cell line gene expression features

The cell line gene expression features are the RNA Seq RPKM values (provided by the CCLE) for the LINC1000 genes. The CCLE RNA Seq RPKM values are available [here](https://portals.broadinstitute.org/ccle); note that you will need to create an account and login to access the CCLE data. The data file that should be downloaded is called: `CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct`. A perl script, `jdacs4c-pilot1/jason/preprocess/CCLE_format_rnaseq.pl`, is provided to format the CCLE data and extract the LINCS100 gene expression values corresponding to the LINCS1000 genes. After downloading the CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct file, run and redirect the script:

`CCLE_format_rnaseq.pl CCLE_DepMap_18q3_RNAseq_RPKM_20180718.gct > <destination file name>`

to produce a tab-delimited file with cell-line samples corresponding to the rows and gene names corresponding to the column. Note that some of the LINCS1000 gene names are not found in the CCLE intput file (which will generate `Failed to match LINCS gene` warnings. However, a total of 958 genes should be matched.

### Drug features

The drug features are the 1021-bit binary fingerprints computed from desalted, 2D chemical structres using the [OpenBabel program](http://openbabel.org/wiki/Main_Page). The 2D drug structures for both the NCI-ALMANAC and Merck datasets are contained in the `jdacs4c-pilot1/jason/preprocess/ALMANAC_and_Merck.smiles` SMILES file. The fingerprints, computed using the following commands: 

```obabel ALMANAC_and_Merck.smiles -ofps -OALMANAC_and_Merck.obable.FP2 -xfFP2`
cat ALMANAC_and_Merck.obable.FP2 | ./expand_fingerprints.pl > ALMANAC_and_Merck.obable.FP2.csv```

are provided in the file `jdacs4c-pilot1/jason/preprocess/ALMANAC_and_Merck.obable.FP2.csv`. The `jdacs4c-pilot1/jason/preprocess/expand_fingerprints.pl` script converts the hex-encoded drug fingerprints into a text-based binary fingerprint (i.e. strings of 1's and 0's).

### NCI-ALMANAC drug synergy data
  
The NCI-ALMANAC drug-pair synergy data is available from [ComboDrugGrowth_Nov2017.zip](https://wiki.nci.nih.gov/download/attachments/338237347/ComboDrugGrowth_Nov2017.zip). 

The first preprocessing step for the NCI-ALMANAC data is to (a) extract the subset of the data that belongs to one of the CCLE cell lines that will be used and (b) converting the NSC-based drug ID used by NCI-ALMANAC to the CID-based drug ID used to label the drug features. Both of these steps are performed by the `dacs4c-pilot1/jason/preprocess/ALMANAC_to_cid_and_CCLE.pl` perl script. This script contains the hard-coded mappings between NSC <-> CID drug ids and the CCLE <-> NCI60 cell line names. The script is run as:

`ALMANAC_to_cid_and_CCLE.pl ComboDrugGrowth_Nov2017.csv > ComboDrugGrowth_Nov2017.CID.CCLE.csv`

where the output (written to STDOUT) is redirected to a filename you select (I used `ComboDrugGrowth_Nov2017.CID.CCLE.csv` in the example above).

The next step of preprocessing the NCI-ALMANAC data is to compute the drug pair synergy values. Since the data file provided by NCI contains both the single agent and drug pair responses (for the different concentrations tested), parsing and processing this file is somewhat challenging. To manage the complexity, this preprocessing step is performed by a C++ - based program called `gemini_prep` that must first be compiled by running the `make` command in the `dacs4c-pilot1/jason/preprocess` directory. 

Please note that the `gemini_prep` program requires the [GNU Scientific Library](https://www.gnu.org/software/gsl/), in addition to C++ compiler (the code has been tested with g++ ver. 4.4.7). After install the GNU Scientific Library, you will need to edit `acs4c-pilot1/jason/preprocess/Makefile` to specify the directories that contain the GSL library files (`GSL_LIB_DIR`) and GSL include files (`GSL_INCLUDE_DIR`). If your compiler does not support OpenMP (i.e. you have a Mac with the stock clang compiler as of 2018) then comment out the `-fopenmp` and the program should still compile (and run more slowly, since multi-threading will now be disabled).

The `gemini_prep` command is run as:

```./gemini_prep \
	-i /home/jgans/Cancer/data/ALMANAC/ComboDrugGrowth_Nov2017.CID.CCLE.csv \
	-p ALMANAC_CID_CCLE_
 ```

where the input file (passed as the argument to the `-i` flag) is the output of the `dacs4c-pilot1/jason/preprocess/ALMANAC_to_cid_and_CCLE.pl` script and the argument to the `-p` flag specifies the output prefix that will be used for all of the output files (i.e. `ALMANAC_CID_CCLE_` in the above example). The `gemini_prep` program will generate a total of six output files (all starting with the specified prefix, i.e. `ALMANAC_CID_CCLE_`):

* `ALMANAC_CID_CCLE_bliss_ave_synergy.csv`
* `ALMANAC_CID_CCLE_bliss_stdev_synergy.csv`
* `ALMANAC_CID_CCLE_loewe_ave_synergy.csv`
* `ALMANAC_CID_CCLE_loewe_stdev_synergy.csv`
* `ALMANAC_CID_CCLE_pair_ave_growth.csv`
* `ALMANAC_CID_CCLE_pair_stdev_growth.csv`

The required synergy for training and testing machine learning models is contained in two of these files:

* `ALMANAC_CID_CCLE_bliss_ave_synergy.csv` for the Bliss synergy model
* `ALMANAC_CID_CCLE_loewe_ave_synergy.csv` for the Loewew synergy model

These files contain the average (over replicate experiments) minimum synergy (i.e. most synergistic over all tested concentrations).

### NCI-ALMANAC drug synergy data

The Merck drug-pair synergy data is contained in the supplementary online data for On'Neil et. al. "An Unbiased Oncology Compound Screen to Identify Novel Combination Strategies", Molecular Cancer Therapeutics, 2016 Jun;15(6):1155-62. The single agent response data is stored in one [Excel file](http://mct.aacrjournals.org/highwire/filestream/53222/field_highwire_adjunct_files/1/156849_1_supp_0_w2lh45.xlsx) and the combination response data is stored in another [Excel file](http://mct.aacrjournals.org/highwire/filestream/53222/field_highwire_adjunct_files/3/156849_1_supp_1_w2lrww.xls).

To process the Merck data, both the single agent and combination response files must be manually convered to comma delimited files. The provided perl script, `jdacs4c-pilot1/jason/preprocess/process_merck.pl`, converts these files into a single file that contains the most synergistic measurements for each drug pair and cell line tested. This file is run as:

`./process_merck.pl --single <single agent response CSV file> --pair <combination response CSV file> > <output CSV synergy file>`

where the output (written to STDOUT) must be redirected to a filename you specify.

2. **Run the single drug-based synergy prediction code**

3. **Run the drug pair-based synergy prediction code**

