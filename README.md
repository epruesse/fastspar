# SparCpp
A c++ implementation of the SparCC algorithm published here: Friedman, J. & Alm, E. J. Inferring correlation networks from genomic survey data. PLoS Comput. Biol. 8, e1002687 (2012). There is an approximately 50x speed up compared to the original Python2 implementation.

Much of the implmentation was ported from original SparCC implmentation found here: https://bitbucket.org/yonatanf/sparcc

Additionally, SparCC's method of p-value has been replaced with exact p-value calculation. This code was adopted from the permp function from the statmod R package. The advantages of this alternative method are discussed here: Phipson, B. & Smyth, G. K. Permutation p-values should never be zero: calculating exact p-values when permutations are randomly drawn Stat Appl Genet Mol Biol. 9: Article 39 (2010).


## Installing
SparCpp has been written using c++ with Armadillo, GNU Scientific Library (GSL), and getopt. Compilation will require these libraries. Further, the exact p-value executable requires compilation of FORTRAN code.


### Prerequisities
For compilation the following is required:
```
C++11
Gfortran
Armadillo 6.7+
GNU Scientific Library 2.1+
GNU getopt
GNU make
```


### Installing
To compile SparCpp executables, clone this repository and run GNU make:

```bash
git clone https://github.com/scwatts/sparcpp.git
cd sparcpp
./configure
make
make install
```
Once done, the SparCpp executables will be in PATH


## Usage
### Correlation inference
To run SparCpp, you must have a OTU absolute counts in BIOM tsv format file (with no metadata). The `fake_data.txt` (from the original SparCC implementation) will be used as an example:

```bash
./sparcpp --otu_table fake_data.txt --correlation median_correlation.tsv --covariance median_covariance.tsv
```

The number of iterations (each iteration re-estimated fractions by drawing from a dirichlet distrubition) can also be changed. The number of exclusion iterations (the number of times highly correlation OTU pairs are discovered and excluded) can also be tweaked. Here's an example:

```bash
./sparcpp --iterations 500 --exclude_iterations 20 --otu_table fake_data.txt --correlation median_correlation.tsv --covariance median_covariance.tsv
```

Further, the minimum threshold to exclude correlated OTU pairs can be increased:
```bash
./sparcpp --threshold 0.2 --otu_table fake_data.txt --correlation median_correlation.tsv --covariance median_covariance.tsv
```


### Calculation of exact p-values
To calculate the p-value of the infered correlations, bootstraping can be performed. This process involves infering correlation from random permutations of the original OTU count data. The p-values are then calculated from the bootstrap correlations. In the below example, we calculate p-values from 1000 bootstrap correlations.


First we generate the 1000 boostrap counts:

```bash
mkdir bootstrap_counts
./bootstrap --otu_table fake_data.txt --number 1000 --prefix bootstrap_counts/fake_data
```

And then infer correlations for each bootstrap count (running in parallel with all processes available):

```bash
mkdir bootstrap_correlation
parallel ./sparcpp --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 5 ::: bootstrap_counts/*
```

From these correlations, the p-values are then calculated:
```bash
./exact_pvalues --otu_table fake_data.txt --correlation median_correlation.tsv --prefix bootstrap_correlation/cor_fake_data_ --permutations 1000 --outfile pvalues.tsv
```


## License
This project is licensed under the GNU GPLv3 Licence
