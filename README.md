# multiSub

multiSub is a command-line tool to prepare and/or submit a SARS-CoV-2 genome
sequence to the NCBI Genbank and GISAID sequence repository. It can also
convert between various formats. These features are sometimes used by a "data
broker", when a single institution collects sequences from labs and submits
them for the labs to the target databases. 

## Overview

multiSub accepts input sequences in fasta format and meta data in tsv, csv or GISAID xls format.
It will make some effort to clean the input data, e.g. skip missing sequences
or remove empty meta data and warn about it. It can then create one or multiple output files, 
in NCBI, NCBI-tag, NCBI-ftp or GISAID-csv format. An automatic upload to NCBI is in development
and one for ENA and GISAID is planned via their respective APIs.

## Input 
The first input is a fasta file with multiple sequences, where each sequence has
a unique ID.

The second input file is a comma- or tab-separated table where the
first column contains the sequence identifier, and the other columns contain
the sequence annotations, sometimes called meta data, or "source tags" by
Genbank. The first row includes the field names. 
For tsv and csv input, the required meta field names are "date" and "isolate".
For GISAID input, only the fields "covv_location", "covv_collection_date" and 
"covv_virus_name" are used.

## Output file formats

By default files in all formats are created. If you only want to create a subset, use the -f option and list the
formats that you need:

- "ncbi" - for manual Genbank upload, as a single fasta file with integrated tags: genbank.seqAndSource.fa
  For manual upload on https://submit.ncbi.nlm.nih.gov/sarscov2/
- "ncbiSep" - for manual Genbank submission: genbankSeq.fa + genbankSource.tsv, two separate files.
  For manual upload on https://submit.ncbi.nlm.nih.gov/sarscov2/
- "ncbi-ftp" - for automated Genbank upload: genbankFtp.zip + submission.xml. See below for details.
- "gisaid" - for GISAID upload in .csv format.

## Requirements

The script was tested on Python 2.7 and 3.6. GISAID xls import requires the xlrd Python package that you can install with
"pip install xlrd" or, if you are not administrator, with "pip install xlrd --user". 

If you use Mac OSX and do not have pip installed yet, run the command "curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py"

## Examples

Convert sequences from seqs.fa with annotations (fields: seqId, date, isolate) in seqs.tsv to 
the directory mySub/ and create files for NCBI and GISAID upload:

    multiSub convert seqs.fa seqs.tsv mySub

Convert sequences from seqs.fa with annotations in GISAID format to the directory mySub/ and only create the NCBI and NCBI-ftp files:

    multiSub convert seqs.fa GISAID.xls mySub -f ncbi,ncbi-ftp

## Automated Genbank uploads

Request an FTP username and password from gb-admin@ncbi.nlm.nih.gov

Create a config file with settings about your group (name, address, email, etc):

    curl https://hgwdev.gi.ucsc.edu/~max/multiSub/multiSub.conf > ~/.multiSub.conf 

Edit the file ~/.multiSub.conf with your institutional information, name,
authors, email, etc. and also the NCBI username and NCBI password.

For full details see https://www.ncbi.nlm.nih.gov/viewvc/v1/trunk/submit/public-docs/genbank/SARS-CoV-2/ 

Then, convert your submission files into the NCBI FTP format:

    multiSub convert seqs.fa seqs.tsv mySub -f ncbi-ftp

Upload sequences in mySub/ to the NCBI FTP server as a test submission:

    multiSub up-ncbi mySub
    
Upload sequences in mySub/ to the NCBI FTP server as a real submission:

    multiSub up-ncbi mySub --prod
    
