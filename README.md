# CAT, BAT

- [Introduction](#introduction)
- [Dependencies and where to get them](#dependencies-and-where-to-get-them)
- [Installation](#installation)
- [Getting started](#getting-started)
- [Usage](#usage)
- [Interpreting the output files](#interpreting-the-output-files)
- [Marking suggestive taxonomic assignments with an asterisk](#marking-suggestive-taxonomic-assignments-with-an-asterisk)
- [Optimising running time, RAM, and disk usage](#optimising-running-time-ram-and-disk-usage)
- [Examples](#examples)

## Introduction
Contig Annotation Tool (CAT) and Bin Annotation Tool (BAT) are pipelines for the taxonomic classification of long DNA sequences and metagenome assembled genomes (MAGs/bins) of both known and (highly) unknown microorganisms, as generated by contemporary metagenomics studies. The core algorithm of both programs involves gene calling, mapping of predicted ORFs against the nr protein database, and voting-based classification of the entire contig / MAG based on classification of the individual ORFs. CAT and BAT can be run from intermediate steps if files are formated appropriately (see [Usage](#usage)).

A paper describing the algorithm together with extensive benchmarks can be found at https://doi.org/10.1186/s13059-019-1817-x. If you use CAT or BAT in your research, it would be great if you could cite us:

* *von Meijenfeldt FAB, Arkhipova K, Cambuy DD, Coutinho FH, Dutilh BE. Robust taxonomic classification of uncharted microbial sequences and bins with CAT and BAT. Genome Biology. 2019;20:217.*


## Dependencies and where to get them
Python 3, https://www.python.org/.

DIAMOND, https://github.com/bbuchfink/diamond.

Prodigal, https://github.com/hyattpd/Prodigal.

CAT and BAT have been thoroughly tested on Linux systems, and should run on macOS as well.

## Installation
No installation is required. You can run CAT and BAT by supplying the absolute path:

```
$ ./CAT_pack/CAT --help
```

Alternatively, if you add the files in the CAT\_pack directory to your `$PATH` variable, you can run CAT and BAT from anywhere:

```
$ CAT --version
```

*Special note for Mac users: since the macOS file system is case-insensitive by default, adding the CAT\_pack directory to your `$PATH` variable might replace calls to the standard unix `cat` utility. We advise Mac users to run CAT from its absolute path.*

CAT and BAT can also be installed via Bioconda, thanks to Silas Kieser:

```
$ conda install -c bioconda cat
```

## Getting started
To get started with CAT and BAT, you will have to get the database files on your system. You can either download preconstructed database files, or generate them yourself which will get you the latest versions of nr and the taxonomy files.

### Downloading the database files.
To download the database files, find the most recent version on [tbb.bio.uu.nl/bastiaan/CAT\_prepare/](https://tbb.bio.uu.nl/bastiaan/CAT_prepare/), download and extract, and you are ready to go!

```
$ wget tbb.bio.uu.nl/bastiaan/CAT_prepare/CAT_prepare_20210107.tar.gz

$ tar -xvzf CAT_prepare_20210107.tar.gz
```

Your version of DIAMOND should be the same as with which the database is constructed. For this reason the DIAMOND executable is supplied within the CAT prepare folder. Alternatively, you can find the DIAMOND version used for database construction within the database log file:

```
$ grep version 2021-01-07.CAT_prepare.fresh.log
```

### Generating the database files yourself.

```
$ CAT prepare --fresh
```

This will download the taxonomy files from NCBI taxonomy to a taxonomy folder, and the nr database to a database folder. A DIAMOND database is constructed from the nr file. CAT prepare also generates a fastaid2LCAtaxid file, as the first accession numbers in the headers of nr are not necessarily the Last Common Ancestor (LCA) of all accession numbers in it. Moreover, the file taxids\_with\_multiple\_offspring is generated. CAT prepare will typically take a few hours to create a fresh database, and will use up to 200GB of memory.

If some of the files are already on your system (say the taxonomy files and the nr database) you can run:
```
$ CAT prepare --existing -d {folder containing nr} -t {folder containing taxonomy files}
```

CAT prepare will assess which files need to be downloaded and created and start from that point. CAT prepare only checks if the necessary files are there, not if they are correctly formatted.

### Running CAT and BAT.
The taxonomy folder and database folder created by CAT prepare are needed in subsequent CAT and BAT runs. They only need to be generated/downloaded once or whenever you want to update the nr database.

To run CAT on a contig set, each header in the contig fasta file (the part after `>` and before the first space) needs to be unique. To run BAT on set of MAGs, each header in a MAG needs to be unique within that MAG. If you are unsure if this is the case, you can just run CAT or BAT, as the appropriate error messages are generated if formatting is incorrect.

### Getting help.
If you are unsure what options a program has, you can always add `--help` to a command. This is a great way to get you started with CAT and BAT.

```
$ CAT --help

$ CAT contigs --help

$ CAT summarise --help
```

## Usage
After you have got the database files on your system, you can run CAT to annotate your contig set:

```
$ CAT contigs -c {contigs fasta} -d {database folder} -t {taxonomy folder}
```

Multiple output files and a log file will be generated. The final classification files will be called `out.CAT.ORF2LCA.txt` and `out.CAT.contig2classification.txt`.

Alternatively, if you already have a predicted proteins fasta file and/or an alignment table for example from previous runs, you can supply them to CAT, which will then skip the steps that have already been done and start from there:

```
$ CAT contigs -c {contigs fasta} -d {database folder} -t {taxonomy folder} -p {predicted proteins fasta} -a {alignment file}
```

The headers in the predicted proteins fasta file must look like this `>{contig}_{ORFnumber}`, so that CAT can couple contigs to ORFs. The alignment file must be tab-seperated, with queried ORF in the first column, nr protein accession number in the second, and bit-score in the 12th.

To run BAT on a set of MAGs:

```
$ CAT bins -b {bin folder} -d {database folder} -t {taxonomy folder}
```

Alternatively, BAT can be run on a single MAG:

```
$ CAT bin -b {bin fasta} -d {database folder} -t {taxonomy folder}
```

Multiple output files and a log file will be generated. The final classification files will be called `out.BAT.ORF2LCA.txt` and `out.BAT.bin2classification.txt`.

Similarly to CAT, BAT can be run from intermidate steps if gene prediction and alignment have already been carried out once:

```
$ CAT bins -b {bin folder} -d {database folder} -t {taxonomy folder} -p {predicted proteins fasta} -a {alignment file}
```

If BAT is run in single bin mode, you can use these predicted protein and alignment files to classify individual contigs within the MAG with CAT.

```
$ CAT bin -b {bin fasta} -d {database folder} -t {taxonomy folder}

$ CAT contigs -c {bin fasta} -d {database folder} -t {taxonomy folder} -p {predicted proteins fasta} -a {alignment file}
```

You can also do this the other way around; start with contig classification and classify the entire MAG with BAT in single bin mode based on the files generated by CAT.

## Interpreting the output files
The ORF2LCA output looks like this:

ORF | lineage | bit-score
--- | --- | ---
contig\_1\_ORF1 | 1;131567;2;1783272 | 574.7

Where the lineage is the full taxonomic lineage of the classification of the ORF, and the bit-score the top-hit bit-score that is assigned to the ORF for voting. The BAT ORF2LCA output file has an extra column where ORFs are linked to the MAG in which they are found.

The contig2classification and bin2classification output looks like this:

contig or bin | classification | reason | lineage | lineage scores
--- | --- | --- | --- | ---
contig\_1 | taxid assigned | based on 14/15 ORFs | 1;131567;2;1783272 | 1.00; 1.00; 1.00; 0.78
contig\_2 | taxid assigned (1/2) | based on 10/10 ORFs | 1;131567;2;1783272;1798711;1117;307596;307595;1890422;33071;1416614;1183438\* | 1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;0.23;0.23
contig\_2 | taxid assigned (2/2) | based on 10/10 ORFs | 1;131567;2;1783272;1798711;1117;307596;307595;1890422;33071;33072 | 1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;1.00;0.77
contig\_3 | no taxid assigned | no ORFs found

Where the lineage scores represent the fraction of bit-score support for each classification. **Contig\_2 has two classifications.** This can happen if the *f* parameter is chosen below 0.5. For an explanation of the **starred classification**, see [Marking suggestive taxonomic assignments with an asterisk](#marking-suggestive-taxonomic-assignments-with-an-asterisk).

To add names to the taxonomy id's in either output file, run:

```
$ CAT add_names -i {ORF2LCA / classification file} -o {output file} -t {taxonomy folder}
```

This will show you that for example contig\_1 is classified as Terrabacteria group. To only get official levels (*i.e.* superkingdom, phylum, ...):

```
$ CAT add_names -i {ORF2LCA / classification file} -o {output file} -t {taxonomy folder} --only_official
```

Or, alternatively:

```
$ CAT add_names -i {ORF2LCA / classification file} -o {output file} -t {taxonomy folder} --only_official --exclude_scores
```

If you have named a CAT or BAT classification file with official names, you can get a summary of the classification, where total length and number of ORFs supporting a taxon are calculated for contigs, and the number of MAGs per encountered taxon for MAG classification:

```
$ CAT summarise -c {contigs fasta} -i {named CAT classification file} -o {output file}

$ CAT summarise -i {named BAT classification file} -o {output file}
```

CAT summarise currently does not support classification files wherein some contigs / MAGs have multiple classifications (as contig\_2 above).

## Marking suggestive taxonomic assignments with an asterisk
When we want to confidently go down to the lowest taxonomic level possible for a classification, an important assumption is that on that level conflict between classifications could have arisen. Namely, if there were conflicting classifications, the algorithm would have made the classification more conservative by moving up a level. Since it did not, we can trust the low-level classification. However, it is not always possible for conflict to arise, because in some cases no other sequences from the clade are present in the database. This is true for example for the family Dehalococcoidaceae, which in our databases is the sole representative of the order Dehalococcoidales. Thus, here we cannot confidently state that an classification on the family level is more correct than an classification on the order level. For these cases, CAT and BAT mark the lineage with asterisks, starting from the lowest level classification up to the level where conflict could have arisen because the clade contains multiple taxa with database entries. The user is advised to examine starred taxa more carefully, for example by analysing sequence identity between predicted ORFs and hits, or move up the lineage to a confident classification (i.e. the first classification without an asterisk).

If you do not want the asterisks in your output files, you can add the `--no_stars` flag to CAT or BAT.

## Optimising running time, RAM, and disk usage
CAT and BAT may take a while to run, and may use quite a lot of RAM and disk space. Depending on what you value most, you can tune CAT and BAT to maximize one and minimize others. The classification algorithm itself is fast and is friendly on memory and disk space. The most expensive step is alignment with DIAMOND, hence tuning alignment parameters will have the highest impact:

- The `-n / --nproc` argument allows you to choose the number of cores to deploy.
- You can choose to run DIAMOND in sensitive mode with the `--sensitive` flag. This will increase sensitivity but will make alignment considerably slower.
- Setting the `--block_size` parameter lower will decrease memory and temporary disk space usage. Setting it higher will increase performance.
- For high memory machines, it is adviced to set `--index_chunks` to 1. This parameter has no effect on temprary disk space usage.
- You can specify the location of temporary DIAMOND files with the `--tmpdir` argument.
- You can set the DIAMOND --top parameter (see below).

### Setting the DIAMOND --top parameter
You can speed up DIAMOND considerably, and at the same time greatly reduce disk usage, by setting the DIAMOND `--top` parameter to lower values. This will govern hits within range of the best hit that are written to the alignment file.

You have to be very carefull to 1) not confuse this parameter with the `r / --range` parameter, which does a similar cut-off but *after* alignment and 2) be aware that if you want to run CAT or BAT again afterwards with different values of the `-r / --range` parameter, your options will be limited to the range you have chosen with `--top` earlier, because all hits that fall outside this range will not be included in the alignment file. **Importantly**, CAT and BAT currently do not warn you if you choose `-r / --range` in a second run higher than `--top` in a previous one, **so it's up to you to remember this!**

If you have understood all this, or you do not plan to tune `-r / --range` at all afterwards, you can add the `--I_know_what_Im_doing` flag and enjoy a huge speedup with much smaller alignment files! For CAT you can for example set `--top 11` and for BAT `--top 6`.

## Examples
Getting help for running the prepare utility:

```
$ CAT prepare --help
```

First, create a fresh database. Next, run CAT on a contig set with default parameter settings deploying 16 cores for DIAMOND alignment. Finally, name the contig classification output with official names, and create a summary:

```
$ CAT prepare --fresh -d CAT_database/ -t CAT_taxonomy/

$ CAT contigs -c contigs.fasta -d CAT_database/ -t CAT_taxonomy/ -n 16 --out_prefix first_CAT_run

$ CAT add_names -i first_CAT_run.contig2classification.txt -o first_CAT_run.contig2classification.official_names.txt -t CAT_taxonomy/ --only_official

$ CAT summarise -c contigs.fasta -i first_CAT_run.contig2classification.official_names.txt -o CAT_first_run.summary.txt
```

Run the classification algorithm again with custom parameter settings, and name the contig classification output with all names in the lineage, excluding the scores:

```
$ CAT contigs --range 5 --fraction 0.1 -c contigs.fasta -d CAT_database/ -t CAT_taxonomy/ -p first_CAT_run.predicted_proteins.fasta -a first_CAT_run.alignment.diamond -o second_CAT_run

$ CAT add_names -i second_CAT_run.contig2classification.txt -o second_CAT_run.contig2classification.names.txt -t CAT_taxonomy/ --exclude_scores
```

First, run BAT on a set of MAGs with custom parameter settings, suppressing verbosity and not writing a log file. Next, add names to the ORF2LCA output file:

```
$ CAT bins -r 10 -f 0.1 -b ../bins/ -s .fa -d CAT_database/ -t CAT_taxonomy/ -o BAT_run --quiet --no_log

$ CAT add_names -i BAT_run.ORF2LCA.txt -o BAT_run.ORF2LCA.names.txt -t CAT_taxonomy/
```

### Identifying contamination/mis-binned contigs within a MAG.

We often use the combination of CAT/BAT to explore possible contamination within a MAG.

Run BAT on a single MAG. Next, classify the contigs within the MAG individually without generating new protein files or DIAMOND alignments.

```
$ CAT bin -b ../bins/interesting_MAG.fasta -d CAT_database/ -t CAT_taxonomy/ -o BAT.interesting_MAG

$ CAT contigs -c ../bins/interesting_MAG.fasta -d CAT_database/ -t CAT_taxonomy/ -p BAT.interesting_MAG.predicted_proteins.faa -a BAT.interesting_MAG.alignment.diamond -o CAT.interesting_MAG
```

Contigs that have a different taxonomic signal than the MAG classification are probably contamination.

Alternatively, you can look at contamination from the MAG perspective, by setting the *f* parameter to a low value:

```
$ CAT bin -f 0.01 -b ../bins/interesting_MAG.fasta -d CAT_database/ -t CAT_taxonomy/ -o BAT.interesting_MAG

$ CAT add_names -i BAT.interesting_MAG.bin2classification.txt -o BAT.interesting_MAG.bin2classification.names.txt -t CAT_taxonomy/
```

BAT will output any taxonomic signal with at least 1% support. Low scoring diverging signals are clear signs of contamination!
