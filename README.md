# multiSub

multiSub is a command-line tool to prepare and/or submit a SARS-CoV-2 genome
sequence to the Genbank sequence repository.

## Summary 

multiSub accepts input sequences in fasta format and meta data in tsv, csv or GISAID xls format.
It will make some best effort to clean the input data, e.g. skip missing sequences
or remove empty meta data and warn about it.

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

## Output

These files are created:

- For manual Genbank submission: genbankSeq.fa + genbankSource.tsv, two separate files.
  For manual upload on https://submit.ncbi.nlm.nih.gov/sarscov2/
- For manual Genbank upload, as a single fasta file with integrated tags: genbank.seqAndSource.fa
- For automated Genbank upload: genbankFtp.zip + submission.xml. See below for details.

## Examples

Convert sequences from seqs.fa with annotations (fields: seqId, date, isolate) in seqs.tsv to 
the directory mySub/:

    multiSub convert seqs.fa seqs.tsv mySub 

Convert sequences from seqs.fa with annotations in GISAID format to the directory mySub/:

    multiSub convert seqs.fa GISAID.xls mySub 

## Automated Genbank uploads

TODO: request username/password. Setup FTP.

Create a config file with settings about your group (name, address, email, etc):

    curl https://hgwdev.gi.ucsc.edu/~max/multiSub/multiSub.conf > ~/.multiSub.conf 

Edit the file ~/.multiSub.conf with your institutional information, name, authors, email, etc.

Convert your submission:

    multiSub convert seqs.fa seqs.tsv mySub 

Upload sequences in mySub/ to the NCBI FTP server:

    multiSub put mySub
    
