# ISEScan

## The Automated Identification of Insertion Sequence Elements in Genomes

Contact:
* Zhiqun Xie: xiezhq@hotmail.com

Last revision: 04/08/2020

## Overview

ISEScan is a python pipeline to identify IS (Insertion Sequence) elements in genome. It includes an option to report either complete IS elements or both complete and partial IS elements. It might be a good idea to try reporting both complete and partial IS elements when it is used to identify the IS elements in the assemblies of metegenome. ISEScan reports both complete and partial IS elements by default.

ISEScan was developed using Python3. It 1) scanes genome (or metagenome) in fasta format; 2) predicts/translates (using FragGeneScan) genome into proteome; 3) searches the pre-built pHMMs (profile Hidden Markov Models) of transposases (two files shipped with ISEScan; clusters.faa.hmm and clusters.single.faa) against the proteome and identifies the transposase gene in genome; 4) then extends the identified transposase gene into the complete IS (Insertion Sequence) elements based on the common characteristics shared by the known IS elements reported by literatures and database; 5) finally reports the identified IS elements in a few result files (e.g. a file containing a list of IS elements, a file containing sequences of IS elements in fasta format, an annotation file in GFF3 format).

## Citation

Zhiqun Xie, Haixu Tang. ISEScan: automated identification of Insertion Sequence Elements in prokaryotic genomes. *Bioinformatics*, 2017, 33(21): 3340-3347. 

URL: [https://doi.org/10.1093/bioinformatics/btx433](https://doi.org/10.1093/bioinformatics/btx433). 

Download: [publication/btx433.pdf](publication/btx433.pdf), [publication/SupplementaryMaterials.docx](publication/SupplementaryMaterials.docx), [publication/SupplementaryMaterials.xlsx](publication/SupplementaryMaterials.xlsx).

## Installation

### Linux:

1. Download the latest ISEScan from https://github.com/xiezhq/ISEScan/releases. The downloaded package is automatically saved as master.zip (Source code (zip)) or master.tar.gz (Source code (zip)).

2. Uncompress the .zip (or .tar.gz) file.
   * Use unzip command to uncompress the zip file:  
   `unzip master.zip`
   * Use tar command to uncompress the tar.gz file:  
   `tar -zvxf master.tar.gz`

## Pre-required packages and libraries

* Python 3.3.3 or later
* numpy-1.8.0 or later
* scipy-0.13.1 or later
* fastcluster, latest version recommended, https://pypi.python.org/pypi/fastcluster
* FragGeneScan1.30 or earlier, (The .faa file output by version1.31 is not compatible with ISEScan!), http://omics.informatics.indiana.edu/FragGeneScan
* HMMER-3.1b2 or later, http://hmmer.org/download.html
* BLAST 2.2.31 or later
* SSW Library, the latest version is not tested with ISEScan and the tested version of SSW library is shipped with ISEScan, please find it at ssw201507 subdirectory.
  * To use the shipped SSW library in ISEScan, please go to ssw201507 and then compile the codes by gcc:  
  `gcc -Wall -O3 -pipe -fPIC -shared -rdynamic -o libssw.so ssw.c ssw.h`
  * And then copy sswlib.so to the directory of ISEScan and set the search path as:   
  `cp libssw.so ../`
  `export LD_LIBRARY_PATH=/path/to/libssw.so:$LD_LIBRARY_PATH # export LD_LIBRARY_PATH=/home/xiezhq/projects/isescan/libssw.so:$LD_LIBRARY_PATH`
  * The latest SSW library can be found at https://github.com/mengyao/Complete-Striped-Smith-Waterman-Library.
* biopython 1.62 or later (required by SSW library)

## Configure ISEScan

1. Open constants.py, and find two lines marked with 'Config packages'
2. Modify the path variables (FragGeneScan, phmmer, hmmsearch, blastn, blastp, makeblastdb, file4clusterSeqFile4phmmer and file4clusterHMM) to specify the correct paths of the required packages and data files on your computer.
3. Save and close constants.py

## Usage example

Let's try an example, NC_012624.fna.

* The command below scans NC_012624.fna (genome sequence of Sulfolobus_islandicus_Y_N_15_51, ~42 kb), and outputs all results in `prediction` directory:   
`python3 isescan.py NC_012624.fna proteome hmm --nthread 2`

* Wait for its finishing. It may take a while (~40 seconds) as ISEScan uses the HMMER to scan the genome sequences and it will use 621 profile HMM models to scan each protein sequence (predicted by FragGeneScan) in the genome sequence. HMMER searching is usually more sensitive but slower than the regular BLAST searching for remote homologs. The running time for larger genome will increase quickly, e.g. about 20 minutes for NC_000913.fna (genome sequence of Escherichia coli str. K-12 substr. MG1655, ~4.6 Mb) with two cpu cores on my virtual machine.

* After ISEScan finish running, you can find the output files in prediction directory: 
  * NC_012624.fna.sum: the summarization of IS copies for each IS family
  * NC_012624.fna.raw: details about IS copies in NC_012624, one copy per line
  * NC_012624.fna.gff: listing each IS copy and its TIR, gff3 format
  * NC_012624.fna.is.fna: the nucleic acid sequence of each IS copy, fasta format
  * NC_012624.fna.orf.fna: the nucleic acid sequence of the Tpase gene in each IS copy, fasta format
  * NC_012624.fna.orf.faa: the amino acid sequence of the Tpase in each IS copy, fasta format

* Details about NC_012624.fna.sum:
  * The title line starts with `#`, followed by the summarization of IS content for each sequence in NC_012624. The last line is the summarization of IS content for all sequences in NC_012624.
  * Summarization of IS content for each sequence in NC_012624:
    * seqid: sequence identifier, extracted from head lines begining with `>` in NC_012624.fna, usuall the texts between `>` and the first blank character in a head line
    * family: family name of IS element
    * nIS: number of IS copies assigned to the specific family in a sequence
    * %Genome: percentage of genome sequence content spaned by IS elements in a sequence, calculated by bps4IS/dnaLen (see the following columns)
    * bps4IS: length of sequence segments spaned by IS elements in a sequence
    * dnaLen: length of the specific sequence

* Details about NC_012624.fna.raw:
  * The first line is title line with the column identifier for each column.
  * The lines following the 2nd line are the main content of NC_012624.fna.raw file, one IS copy per line.
  * Columns in NC_012624.fna.raw:
    * seqID: sequence identifier
    * family: family name of IS element
    * cluster: Tpase cluster
    * isBegin and isEnd: genome coordinates of the predicted IS element
    * isLen: length of the predicted IS element
    * ncopy4is: number of predicted IS copies including full-length and partial IS copies
    * start1, end1, start2, end2: genome coordinates of the IRs
    * score: score of the IRs
    * irId: number of identical matches in pairwise alignment of left and righ hand invered repeats
    * irLen, length of inverted repeats
    * nGaps: number of gaps in IRs
    * orfBegin, orfEnd: genome coordinates of the predicted Tpase ORF
    * strand: strand where the Tpase is
    * orfLen: length of predicted Tpase ORF
    * E-value: the best E-value among all IS copies for the same IS element, the smaller the better
    * E-value4copy: the E-value of the reported IS copy, the smaller the better
      * Note: the E-value is the E-value returned by hmmer when searching profile HMMs against proteome translated from a genome sequence
    * type: type of IS element copy, 'c' for complete IS element and 'p' for partial IS element
    * ov: ov number returned by hmmer search
    * tir: terminal inverted repeat sequences

### Tips:
* How to run a set of genomes in a row
  Based on a few assumptions:
  - You can successfully run ISEScan on one genome by executing `python3 /home/qiime2/ISEScan-1.7/isescan.py genome1.fa proteome hmm` where genome1.fa is your genome sequence file in fasta format. By default, ISEScan will use one CPU core but you can change it using command option `--nthread NTHREAD`, e.g. `python3 isescan.py NC_000913.fna proteome hmm --nthread 2`.
  - You are working and running ISEScan jobs on a Linux computer instead of a Linux cluster system.
  - Your Linux computer has `nproc` (nproc could be 2 or 4 or 6 or 8 or ....) CPU cores.
  - You want to run ISEScan on ngenome (ngenome could be 1 or 2 or 3, ...) fasta file(s) (genome) in parallel on your Linux computer.

  Now, let's run 200 genomes in one line of command and then wait for all computing jobs to complete (probably several days or weeks, depending on how many hours are required for each of your 200 genomes in average). If your computer has 8 CPU cores and You can execute the command below:
  `nohup cat test.fna.list | xargs -n 1 -P 4 -I{} python3 /home/qiime2/isescan/isescan.py {} proteome hmm --nthread 2 > log.txt &`

  In the command line, 
  - **_test.fna.list_** is a text file which includes 200 fasta files, one fasta file per row, for example:
  `
  /N/dc2/scratch/zhiqxie/hmp/HMASM/SRS014235.scaffolds.fa
  /N/dc2/scratch/zhiqxie/hmp/HMASM/SRS049959.scaffolds.fa
  /N/dc2/scratch/zhiqxie/hmp/HMASM/SRS020233.scaffolds.fa
  /N/dc2/scratch/zhiqxie/hmp/HMASM/SRS022609.scaffolds.fa
  /N/dc2/scratch/zhiqxie/hmp/HMASM/SRS024132.scaffolds.fa
  ` 
  - **_-n 1_** tells your computer to pick only one fasta file from **_test.fna.list_** for each ISEScan computing job.
  - **_-P 4_** tells your computer to spawn 4 processes at the same time (run 4 ISEScan jobs in parallel, namely, run 4 genomes at the same time). When one job completes with success or exits with error, a new ISEScan job on the next fasta file (e.g. 5th fasta file) in **_test.fna.list_** is spawned. So, the command line will keep 4 ISEScan computing jobs (one fasta file per ISEScan job) running on your computer, and each job utilizes two CPU cores by default. It means all of 8 CPU cores on your computer have been utilized by your 4 ISEScan computing jobs till the last fasta file is processed by ISEScan.
  - **> log.txt** tells your computer to write the screen messages output by ISEScan to the file `log.txt`.
  - **&** tells your computer to run jobs in the background without interrupting you on the current terminal (e.g. xterm), in order that you can work on other things on the same terminal.
  You can check your job status by the command `top -c -u qiime2` (assuming your user name is `qiime2`). 

  It might take several days or weeks for 200 genomes to complete. It depends on how many CPU cores you have on your computer and how fast each CPU core is. Please do not load too many ISEScan jobs because each ISEScan job will consume part of your RAM on your computer. However, you can always test and estimate how many GB RAM and how many hours are required for a genome.

* ISEScan will run much faster if you run it on the same genome sequence more than once (e.g., trying different optimal parameters of near and far regions (see our paper [...] for the definitions of near and far regions)) to search for IS elements in your genome). The reason is that it skips either FragGeneScan or both FragGeneScan and phmer/hmmsearch steps which are most time-consuming steps in ISEScan pipeline.
* If you prefer ISEScan recalculating the the results, you can simply remove the proteome file and HMMER search results which are related to your genome sequence file name. For example, you can delete NC_012624.fna.faa in proteome directory and clusters.faa.hmm.NC_012624.fna.faa and clusters.single.faa.NC_012624.fna.faa in hmm directory, and then rerun it:  
`python3 isescan.py NC_012624.fna proteome hmm`

## Release History 
* 1.7.2.
  * Add command options `--removeShortIS` and `--no-FragGeneScan`, and remove `removeShortIS` and `translateGenome` from constants.py. (Thanks EricDeveaud for his suggestion and codes)
  * Add command option `--nthread` to isescan.py, and remove `nthread` and `nproc` from constants.py.
  * Remove useless parallel testing codes from code base.
* 1.7.1
  * fix a bug in constants.py, which fails to locate the correct path pointing to profile HMM files (clusters.single.faa and clusters.faa.hmm). Thank giuliodimaria92 for it.
* 1.7
  * Set removeShortIS = False in constants.py for ISEScan to report both complete and partial IS elements by default. One additional column (type) was added accordingly in .raw output file to label each IS element copy as either complete (c) or partial (p) IS element. For details refer to the section 'Details about NC_012624.fna.raw' in Readme.
* 1.6
  * Update Readme about the configuration of ISEScan where the paths to clusters.faa.hmm and clusters.single.faa should also be correctly specified in constants.py (Thank Ania Gorska for it).
* 1.5.4.3
  * Fix the bug which failed to report the Tpase ORFs in multi-copy IS elements, and ISEScan now output a .raw file with one additional column E-value4copy which is the E-value of the reported IS copy while the column E-value is the best E-value among all IS copies for the same IS element.
* 1.5.4.1
  * fix bug for batch4bacteria.py when *.sum files were created by either outputIndividual() or outputIS4multipleSeqOneFile() in pred.py
* 1.5.4
  * Add removeFalsePositive() to remove the potentail false positive in the 'new' family: 1) single-copy hits with e-value > e-50 or no tir or nGaps > 0 or irId < 20 or irId/irLen < 0.75; 2) multi-copy hits with evalue > e-50 and (irId < 13 or (irId < 20 and ngaps > 0))
  * Modify refineHits() to remove the single-copy partial IS elements: 1) if evalue > e-50 or (irId < 13 or (irId < 20 and ngaps > 0 for familys other than IS200/IS605)
  * Modify refineHits() to remove the multi-copy partial IS elements: 1) if evalue > e-50 for IS200/IS605 family; 2) if irId < 10 for familys other than ten familys which could have the full IS without perfect TIR (irId < 10), IS110, IS4, IS5, IS6, ISAS1, ISH3, ISNCY.
  * Change irSim4singleCopy in constants.py from 0.85 to 0.75, for the use in removeFalsePositive()
* 1.5.3
  * Fix bug in getFullIS4seqOnStream() for genome sequence with long multi-copy fregments containing the common IS element
  * Use 'average' instead of 'single' method in fastcluster.linkage()
  * Fix bug in removeOverlappedOrfhits() to correctly count single-copy IS elements for genome sequence without multi-copy IS elements
* 1.5.2
  * Fix bug for genome sequence without multi-copy IS elements
* 1.5.1
  * Change: changed consensusBoundaryByCutoff() to consensusBoundaryByCutoffBySeparated()
  * Change: added consensusBoundaryByCutoffByCombined() and getbds4opt4start(), to determine the left and right boundaries of multi-copy pro-IS element simultaneously, namely, to determine the optimal combined left and right boundaries instead of separated left and right boundaries.
* 1.5
  * Change: add consensusBoundaryByCutoff() and ncopyByCutoff() in tools.py, to determine the optimal boundary of multi-copy pro-IS element.
* 1.4
  * Change: recruit the IS copies without predicted Tpase when search for multi-copy IS elements
* 1.3
  * Remove buildHMM.py from ISEScan
* 1.2
  * CHANGE: pHMMs `clusters.faa.hmm` and `clusters.single.faa`, both files are now built upon the curated ACLAME dataset (ACLAME is a mobile genetic element database.)
* 1.1.1
  * Add option in `constants.py` to report either complete IS elements or both complete and partial IS elements
* 1.0
  * The first proper release

## License

Distributed under the GNU General Public License.
