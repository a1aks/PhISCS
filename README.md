# PhISCS

PhISCS is a tool for sub-perfect tumor phylogeny reconstruction via integrative use of Single Cell and Bulk Sequencing data, with latter being an optional. However, if provided, Bulk Sequencing data is expected to be of high depth of coverage (>= 1000x haploid coverage). As output, PhISCS reports (single) tree of tumor evolution together with a set of mutations for which Infinite Sites Assumption is violated. 

## Installation
```
git clone https://github.com/haghshenas/PhISCS.git
cd PhISCS
./configure
```

## Running
### Usage
```
usage: toolname [-h] -f FILE -fn FNPROBABILITY -fp FPPROBABILITY -o OUTDIR
              [-w COLPROBABILITY] [-kmax MAXMUT] [-t THREADS] [-b BULK]
              [-e DELTA] [--truevaf] [--timeout TIMEOUT]

Required arguments:
   -f, --file           STR    Input matrix file
   -fn, --fnProbability INT    Probablity of false negative
   -fp, --fpProbability INT    Probablity of false positive
   -o, --outDir         STR    Output directory

Optional arguments:
   -w, --colProbability INT    Probablity of eliminated columns
   -kmax, --maxMut      INT    Max number mutations to be eliminated [0]
   -t, --threads        INT    Number of threads [1]
   -b, --bulk           STR    Bulk sequencing file [""]
   -e, --delta          FLT    Delta in VAF [0.01]
   --truevaf                   Use tree VAFs
   --timeout            INT    Max time allowed for the computation

Other arguments:
   -h, --help                  Show this help message and exit
```

### Input
The input matrix file is assumed to be tab-delimited. This file has the following format for a matrix with _C_ cells and _M_ mutations:
```
cellID/mutID      mut1     mut2     mut3     ...      mutM
cell1             x        x        x        ...      x
cell2             x        x        x        ...      x
cell3             x        x        x        ...      x
...
cellC             x        x        x        ...      x
```
Where _x_ is in {0, 1, 2}. More specifically:
* The first line is the header line. First string in upper left corner must be **cellID/mutID** and next _M_ strings are the names of mutations.
* Each of the next _C_ lines contains mutation information for a single cell. The first string is cell name and next _M_ integers show if a mutation is observed (1) or not (0). The value 2 means the information is not available (missing).

### Output
The program will generate two files in **OUT_DIR** folder (which is set by argument -o or --outDir).
#### 1. log file
Suppose the input file is **INPUT_MATRIX.ext**, the log will be stored in file **OUT_DIR/INPUT_MATRIX.log**. For example:
```
input file: simNo_1-n_100-m_40-s_4-minVAF_0.05-cov_10000-k_0-fn_0.05-fp_0.0001-na_0.15.SCnoisy
  log file: simNo_1-n_100-m_40-s_4-minVAF_0.05-cov_10000-k_0-fn_0.05-fp_0.0001-na_0.15.ILP.log

input file: wang.txt
  log file: wang.ILP.log
```
The log file contains a summary for running the program on the input file. It should be in the following format:
<pre><code>FILE_NAME: STR
NUM_CELLS(ROWS): INT (INT >= 1)
NUM_MUTATIONS(COLUMNS): INT (INT >= 1)
FN_WEIGHT: INT (INT >= 1)
FP_WEIGHT: INT (INT >= 1)
NUM_THREADS: INT (INT >= 1)
MODEL_SOLVING_TIME_SECONDS: FLOAT (FLOAT > 0)
RUNNING_TIME_SECONDS: FLOAT (FLOAT > 0)
IS_CONFLICT_FREE: YES/NO
LIKELIHOOD: FLT
MIP_Gap_Value: FLT
TOTAL_FLIPS_REPORTED: INT (INT >= 0)
0_1_FLIPS_REPORTED: INT (INT >= 0)
1_0_FLIPS_REPORTED: INT (INT >= 0)
2_0_FLIPS_REPORTED: INT (INT >= 0)
2_1_FLIPS_REPORTED: INT (INT >= 0)
MUTATIONS_REMOVED_UPPER_BOUND: INT (INT >= 0)
MUTATIONS_REMOVED_NUM: INT (0 <= INT <= MUTATIONS_REMOVED_UPPER_BOUND)
MUTATIONS_REMOVED_INDEX: [INT<sub>1</sub>,INT<sub>2</sub>,...,INT<sub>MUTATIONS_REMOVED_NUM</sub>] (1 <= INT<sub>i</sub> <= NUM_MUTATIONS for each 1 <= i <= MUTATIONS_REMOVED_NUM)
MUTATIONS_REMOVED_ID: STR
</code></pre>
#### 2. output matrix file
Suppose the input file is **INPUT_MATRIX.ext**, the output matrix will be stored in file **OUT_DIR/INPUT_MATRIX.output**. For example:
```
 input file: simNo_1-n_100-m_40-s_4-minVAF_0.05-cov_10000-k_0-fn_0.05-fp_0.0001-na_0.15.SCnoisy
output file: simNo_1-n_100-m_40-s_4-minVAF_0.05-cov_10000-k_0-fn_0.05-fp_0.0001-na_0.15.ILP.conflictFreeMatrix

 input file: wang.txt
output file: wang.ILP.conflictFreeMatrix
```
The output file is also a tab-delimited file with the exact same format as the input file. The only difference compared to the input file is that _x_ values of the matrix are modified so that the matrix is conflict free.

## Example
As a toy example, an input file is created and named as *input.txt* which contains:
```
cellID/mutID mut0 mut1 mut2 mut3 mut4
cell0 2 0 1 1 2
cell1 1 0 1 1 1
cell2 2 2 0 1 2
cell3 1 1 2 1 1
cell4 0 1 1 2 1
cell5 1 1 1 2 1
```

For running PhISCS without VAFs information:
```
python ilp.py -f input.txt -fn 0.05 -fp 0.0001 -o result/ -w 0 -kmax 3 --timeout 86400
```

For running PhISCS with VAFs information:
```
python ilp.py -f input.txt -fn 0.05 -fp 0.0001 -o result/ -w 0 -kmax 3 -b input.bulk -e 0.05 --timeout 86400
```



## Contact
If you have any questions please e-mail us at smalikic@sfu.ca or frashidi@iu.edu.

