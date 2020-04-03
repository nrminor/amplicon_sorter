# amplicon_sorter

Sorts amplicons from Nanopore sequencing data based on similarity, creates groups with highly similar sequences and create the consensus sequence for each group of sequences.

(Only works on Linux because it uses multiprocessing)

(This is a work in progress, but already producing good enough results to share the script.)

Requirements:

-   Python 3
-   python3-dev, python3-setuptools, python3-pip, python3-wheel (`sudo apt-get install python3-dev python3-setuptools python3-pip python3-wheel`)
-   c-implementation of Levenshtein: [https://pypi.org/project/python-Levenshtein/](https://pypi.org/project/python-Levenshtein/) (`python3 -m pip install python-Levenshtein`)
-   biopyton (`sudo apt-get install python3-biopython`)
-   matplotlib (`sudo apt-get install python3-matplotlib`)

### Options:

`-i, --input`: Input file in fastq or fasta format (auto-detect). Make sure the inputfile is named as .fasta or .fastq because it replaces the extension in parts of the script.

`-min, --minlength`: Minimum readlenght to process. Default=300

`-max', '--maxlength`: Maximum readlenght to process. Default=No limit

`-maxr', '--maxreads`: Maximum number of reads to process. Default=10000

`-np', '--nprocesses`: Number of processors to use. Default=1

`-sg', '--similar_genes`: 'Similarity to sort genes in groups (value between 50 and 100). Default=55.0

`-ss', '--similar_species`: Similarity to sort to species level (value between 50 and 100). Default=85.0

`-sfq', '--save_fastq`: Save the results also in fastq files (fastq files will not contain the consensus sequence)

`-ra', '--random`: Takes random reads from the inputfile if --maxreads is lower than total number of reads that passed criteria

`-of', '--outputfolder`: Save the results in the specified outputfolder. Default = current working folder

### How it works (in short):

1.  The script reads the inputfile and creates a read length histogram of all the reads in the file.
2.  It starts processing a selection of the reads (based on minlength, maxlenght, maxreads). It saves the result in .group files that contain reads of the same gene (e.g. group_1 with 18S reads, group_2 with COI reads, group_3 with ITS reads).
3.  It processes the group files to sort out the genes to species or genus level and saves this to different files. (e.g. file_1_1.fasta is 18S from species1, file1_2.fasta is 18S from species2, ...)
4.  Each outputfile contains at the end the consensus file of the sequences in the file.
5.  Files are produced per group with all consensus sequences for that group. A file is produced with all consensus sequences.
6.  Files are produced with 'unique' sequences (script does not find a group where it belongs to according to the settings)

### Command examples:

`python3 amplicon_sorter.py -i inputset1.fastq` : process file with default settings.

`python3 amplicon_sorter.py -i inputset1.fastq --similar_species 80 --nprocesses 8`: process file on 8 cores with allowing the similarity between sequences of the same gene as low as 80%

### My experience so far:

-   Guppy produces reads with Q-score >=7 by default. If I filter reads to Q-score >= 10 or Q-score >= 11, the end result (similarity % of the consensus to a species in BLAST) is for all comparable. The only difference is that I get more reads per species group for Q-score 11 than for Q-score 7, and that there are less reads in the 'unique' groups.
-   The same results if I lower the `--similar_species` value from 90% to 80%. Above 90% the similarity in Blast is getting lower, between 80 and 90 the similarity is comparable. The lower I go, the more sequences in the species groups.
-   10000 reads (`--maxreads`) on 12 cores takes several hours to finish.

### Todo:

-   options to create read length histogram only
-   option to filter to gene groups only and later start from there to play with the `--similar_species` option

### Release notes:

2020/3/5:

-   fasta or fastq files as input (autodetect)
-   fastq files as output option (when input is fastq)

2020/3/12:

-   option to take a random selection of the reads from the inputfile.
-   little improvement to sort out the groups in step 2.

2020/4/3:

 - option to save results in a subfolder.
 - more little improvement to sort out the groups in step 2. (The result is still not what I had in mind: sometimes reads of ITS and 18S are in the same group.  The reason for this is that ITS can be very different from species to species.  If I set similarity to 55%, it can put some ITS in the 18S group.  If I set it higher, than COI is not in one group.  This has no influence on the end results in step 3-4) 

