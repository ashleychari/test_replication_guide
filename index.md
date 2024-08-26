---
layout: default
---

Text can be **bold**, _italic_, or ~~strikethrough~~.

[Link to another page](./another-page.html).

There should be whitespace between paragraphs.

There should be whitespace between paragraphs. We recommend including a README, or a file with information about your project.

# ABCD Sex Differences replication project

This is a normal paragraph following a header. GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere.

# Code Documentation:

### Part 1: Discovery and Replication sample generation
  > You'll need to change the save filename/paths in the script
  
  1. Run the [get_data_for_ridge.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/discovery%20and%20replication%20sample%20setup%20scripts/get_data_for_ridge.R) script created by Arielle Keller using the data in the `FilesForAdam` folder originally from `/cbica/projects/abcdfnets/scripts/FilesForAdam/`

  1. Next, run the [create_discovery_replication_set_siblings_removed.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/discovery%20and%20replication%20sample%20setup%20scripts/create_discovery_replication_sets_siblings_removed.py) script to create the final discovery and replication sets with the removal of siblings. These samples are used in subsequent steps.


### Part 2: Atlas Generation and Network Parcellation
  > Will need to change paths to discovery and replication datasets and the paths that you want to save the resulting files

  1. Run the [calc_group_average_mat.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/atlas_visualization/calc_group_average_mat.py) script to get the group average matrix for the networks for all of the subjects in the discovery and replication sets combined

  1. Run the [create_hard_parcel.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/atlas_visualization/get_subject_parcels.py) script to get the hard parcellations from networks 3, 4, and 12

  1. Run the [create_soft_parcel.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/atlas_visualization/get_subject_parcels.py) script to get the soft parcellations from networks 3, 4, and 12

  1. Run the [get_subject_parcels.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/atlas_visualization/get_subject_parcels.py) script to get the hard and soft parcellations for 4 random subjects with 2 subjects being male and 2 being female

### Part 3: Univariate Analysis
  1.  Use [convert_mat.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/discovery%20and%20replication%20sample%20setup%20scripts/convert_mat.py) to convert the final_UV.mat files into csvs

  _NOTE: This step can be skipped if you change the script to use `readMat` from the `R.Matlab` package instead of base R's `read.csv` function. However, you will have to setup a new singularity container with the necessary packages_

  2. Create a singularity container or use the one I created stored in `ashpfnsexdiffabcd/software/containers/` called `sex_differences_replication_0.0.3.sif`

  3. Change the filepaths for the results folder and subject data in the [abcd_unvariate_analysis.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/GAMs%20scripts/R%20scripts/abcd_univariate_analysis.R) script for the discovery data, which will run a GAM at each vertex to determine the effect of sex while controlling for age and motion. The script assumes that the subject data has siblings removed. Next, run the wrapper script [run_univariate_analysis.sh](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/GAMs%20scripts/shell%20(job)%20wrappers/run_univariate_analysis.sh) via SGE using the following command:

  ```bash
  qsub -l h_vmem=25G,s_vmem=25G /cbica/projects/ash_pfn_sex_diff_abcd/dropbox/run_univariate_analysis.sh
  ```

  4. Similarly, do the same thing but for the replication data and use [abcd_unvariate_analysis_replication.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/GAMs%20scripts/R%20scripts/abcd_univariate_analysis.R) script and then run the wrapper script [run_univariate_replication.sh](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/GAMs%20scripts/shell%20(job)%20wrappers/run_univariate_replication.sh) via SGE using the following command:

  ```bash
  qsub -l h_vmem=25G,s_vmem=25G /cbica/projects/ash_pfn_sex_diff_abcd/dropbox/run_univariate_replication.sh
  ```

  5. Once the gams models have been run, create the unthresholded absolute sum map from the results using the [write_effect_map_abs_sum.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/workbench%20setup%20scripts/write_effect_map_abs_sum.py) script. You'll need to change the arguments in the two function calls in the main execution block of the script.

  6. Convert the file obtained from the previous step into a CIFTI format to be loaded into workbench by using the [write_effect_map_to_cifti.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/workbench%20setup%20scripts/write_effect_map_to_cifti.py). 

  _NOTE: The script uses a dscalar.nii file of the fslr format as a temporary header to make the new CIFTI file. It is only used to structure the file._
  
  The script can be run like the following example command:

  ```bash
  python3 write_effect_map_to_cifti.py '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/multivariate_analysis/svm_072324_run/abs_sum_weight_brain_mat_discovery_haufe_100_runs_072324.npy' '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/multivariate_analysis/svm_072324_run/svm_discovery_abs_sum_haufe_weights_072324.dscalar.nii'
  ```

  7. Load the resulting dscalar.nii file from the previous step into workbench to get the visualization.

  8. To get the significant veritces barplot, first run the [make_barplot_mat.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/barplot%20scripts/make_barplot_mat.py) script to get a matrix of the significant vertices for each network.

  9. Use the [create_barplot.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/barplot%20scripts/create_barplot.R) to get the barplot of significant vertices for each network. I ran this in Rstudio.

  10. To create the significant vertices for networks 3, 9, and 12, use the [get_individual_network_mat.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/univariate_analysis/individual%20network%20scripts/get_individual_network_mat.py) script to get each individual network from the matrix in the previous step.


### Part 4: Multivariate Analysis
  1. 
  1. 

### Part 5: Genetics
  > The files generated from the steps below will be used in the enrichement analyses.

  1. Download files from [](https://github.com/PennLINC/S-A_ArchetypalAxis) to be used in the `flsr_to_fsaverage5.sh` script

  2. Use the [fslr_to_fsaverage5.sh](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/fslr_to_fsaverage5.sh) script to resample the gams uncorrected abs sum discovery data, gams uncorrected abs sum replication data, gams fdr abs sum discovery data, and gams fdr abs sum replication data from fslr to fsaverage5. Parameters include path to the map (dscalar.nii file) you're trying to resample to fsaverage5 space and the name that you want to label the resulting files. The file that you'll want to use in the enrichment analyses will end with `_LH.fsaverage5.func.gii`.
  
  Example command below:

  ```bash
  ./fslr_to_fsaverage5.sh /Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/univariate_analysis/univariate_analysis_results/uncorrected_abs_sum_matrix_discovery.dscalar.nii gams_uncorrected_discovery
  ```

  3. Download Download `AHBAProcessed.zip` and `AHBAData.zip` from [https://figshare.com/articles/dataset/AHBAdata/6852911](https://figshare.com/articles/dataset/AHBAdata/6852911)   

  4. Read annotation file `AHBAData/data/genes/parcellations/lh.Schaefer1000_7net.annot` from figshare into R and use [read_gene_annot_files.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/read_gene_annot_files.R) to save the following files
      > save column 5 from `ct.table` as `lh_Schaefer1000_7net_cttable.csv`
      <br>  
      > save `L` as `lh_Schaefer1000_7net_L.csv`

  5. Save probe annotation file using `ROIxGene_Schaefer1000_INT.mat` and [create_probe_annotation.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/create_probe_annotation.py)
      > Save the filename as `GeneSymbol.csv`
  
  6. Grab the rest of the files needed for the scripts below from `/cbica/projects/ash_pfn_sex_diff_abcd/dropbox/genetics_files/` which originally came from the `abcdpfnsexdiff` project folder on cubic


### Chromosomal enrichment analysis
  1. Run the [ABCD_wSex_cor_gene_schaefer403_net7_discovery_20242108.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/chromosome_enrichment/ABCD_wSex_cor_gene_schaefer403_net7_discovery_20242108.R) script to get the chromosomal enrichments, recommend running this in Rstudio

  > Make sure to change the filepaths to use the files created above

  > Also make sure to change the filepaths for saving the intermediate files such as pvalues table and top 20 X chromosome genes

### Cell Type enrichment analysis
  1. Run the [cell_types_LAKE.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/celltype_enrichment/cell_types_LAKE.R) script to get the cell type enrichments, recommend running this in Rstudio

  > Make sure to change the filepaths to use the files created above
    
  > Also make sure to change the filepaths for saving the intermediate files such as pvalues table and top 20 tables for astro, ex5b, ex1, and oli

### PSP enrichment analysis
  1. Run the [PSP_plots.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/genetics/PSP_enrichment/PSP_plots.R) script to get the PSP enrichments, recommend running this in Rstudio

  > Make sure to change the filepaths to use the files created above and for the pvalue table

### Part 6: Spin tests
  1. Add medial wall back in to all of the data (gams uncorrected discovery abs sum, gams uncorrected replication abs sum, svm discovery abs sum haufe transformed, svm replication abs sum haufe transformed) using [add_medial_wall.R](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/add_medial_wall.R). Save results into csv that will later be converted into gii files.

  2. Use the [convert_to_gifti.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/convert_to_gifti.py) script to convert the data in the previous step into gii files. The following command can be run in terminal (the last argument indicated whether the data is from PNC or not): 

  ```bash
  python3 convert_to_gifti.py "medial_wall_maps/gams_discovery_medial_wall_map.csv" "results_gifti/gams_abs_sum_discovery_map.gii" N
  ```

  3. Run the [spin_test.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/spin_test.py) script using the converted gii files. The script can be run using the following example command:

  ```bash
  python3 spin_test.py '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/data/results_gifti/gams_abs_sum_discovery_map.gii' '/Users/ashfrana/Desktop/code/ abcd_sex_pfn_replication/spin_tests/data/results_gifti/gams_abs_sum_replication_map.gii' "fsLR" 'Gams discovery vs replication' '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/results'
  ```

  4. Once all spin tests have been completed, you can compile all of the results from the different tests as long as they are in the same folder by using the [compile_spin_tests.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/compile_spin_tests.py). The script can be run by calling the name of the script followed by the folder that the spin test results are stored in (might need to be the full path if the script is not in the same directory as the results):

  ```bash
  python3 compile_spin_tests.py spin_test_results_072524
  ```

  _The following instructions apply to the PNC data only_
  5. Grab the PNC gams abs sum data from `/cbica/projects/abcdpfnsexdiff/funcParcelSexDiff/inputData/spintest/`

  6. Convert the csv files of the gams abs sum LH and RH results into gii files using [convert_to_gifti.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/convert_to_gifti.py)

  ```bash
  python3 convert_to_gifti.py '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/data/PNC_data/GamSexAbssum_lh.csv' '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/data/PNC_data/gams_abs_sum_lh.gii' Y
  ```

  7. Use the [PNC_to_fslr.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/PNC_to_fslr.py) to convert the PNC results to flsr space

  8. Run [spin_test.py](https://github.com/ashleychari/abcd_sex_pfn_replication/blob/main/spin_tests/spin_test.py) using the following command:

  ```bash
  python3 spin_test.py '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/data/PNC_data/Gam_abs_sum_fslr_test.gii' '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/data/ABCD_data/gams_abs_sum_discovery_uncorrected.gii' "fsLR" 'PNC gams discovery vs ABCD gams discovery fslr' '/Users/ashfrana/Desktop/code/abcd_sex_pfn_replication/spin_tests/results'
  ```
  





## Header 2

> This is a blockquote following a header.
>
> When something is important enough, you do it even if the odds are not in your favor.

### Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![Octocat](https://github.githubassets.com/images/icons/emoji/octocat.png)

### Large image

![Branching](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
