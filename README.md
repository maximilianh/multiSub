# multiSub

multiSub is a command-line tool to prepare and/or submit a SARS-CoV-2 genome
sequence to the NCBI Genbank, EBI ENA and GISAID sequence repositories. It can also
convert between these formats. This tool can be used by a "data
broker", a single institution that collects sequences from labs and submits
them to the sequence databases. It's early research software, you will probably 
find bugs, please open a ticket on Github or email maxh@ucsc.edu if that happens, most of them
should be easy to fix now.

## Overview

multiSub accepts input sequences in fasta format and meta data in tsv, csv or GISAID xls format.
It will make some effort to clean the input data, e.g. skip missing sequences
or remove empty meta data and warn about it. It can then create one or multiple output files, 
in NCBI, NCBI-tag, NCBI-ftp, ENA-xml or GISAID-csv format and directly upload
to NCBI, ENA or GISAID.

The script takes care of the different ways to format the virus names (for
example, hCov-19 for GISAID, SARS-CoV-2 for NCBI), translates the different ways to specify 
the country, checks the date format and adds sequence IDs where needed. It does
not support more than the date and isolate and country fields, but other fields
can be easily added, just email examples to maxh@ucsc.edu.

Many thanks to Stephan Fuchs and Kyanoush Yahosseini, Robert Koch Institut,
Berlin, for sending me their Python ENA uploader code, from which I copied. Also
thanks to the ENA Helpdesk and the NCBI Helpdesk for their quick replies.
Without all of these people, this program would not have been possible.

## Installation and requirements

The script has no dependencies in most cases. Just download it, make it executable and run it:

    wget https://raw.githubusercontent.com/maximilianh/multiSub/main/multiSub
    chmod a+x multiSub
    multiSub --help

This script was tested on Python 2.7 and 3.6. If you do not plan to read GISAID xls files,
you do not need to do anything else. If you want to read GISAID xls files, the
script needs the xlrd Python package. You can install it with "pip install
xlrd" or, if you are not administrator, with "pip install xlrd --user".  If you
use Mac OSX and do not have pip installed yet, run the command "curl
https://bootstrap.pypa.io/get-pip.py -o get-pip.py && python3 get-pip.py"

Using Microsoft Windows ? Please contact me at maxh@ucsc.edu. The script runs
in the Windows WSL, but I can also provide a normal Windows .exe version, if that is helpful.

## Input 

The first input is a fasta file with multiple sequences, where each sequence has
a unique ID.

The second input file is the usual comma- or tab-separated table where the
first column contains the sequence identifier, and the other columns contain
the sequence annotations, sometimes called meta data, or "source tags" by
NCBI. The first row contains the field names, sometimes called "headers". 
This file can be in NCBI, ENA or GISAID format. For tsv and csv input,
the required meta field names are "date" and "isolate". For GISAID input, only
the fields "covv_location", "covv_collection_date" and "covv_virus_name" are 
used at the moment. GISAID files can be in .xls or .csv format.

## Output file formats

The basic files seqs.fa and meta.tsv will always be created.

By default, files in all possible output formats are created. If you only want to create a
subset, use the -f option and list the formats that you need:

- "ncbi" - for manual Genbank upload, as a single fasta file with integrated tags: genbank.seqAndSource.fa
  For manual sequence upload on https://submit.ncbi.nlm.nih.gov/sarscov2/
- "ncbi-ftp" - for automated Genbank upload: genbankFtp.zip + submission.xml. See below for details.
- "gisaid" - for GISAID upload in .csv format: gisaid.csv and gisaid.fa
- "ena" - for ENA automated sample uploads in XML format: ena.xml


## Example: Convert files

Convert sequences from myseqs.fa with annotations (fields: seqId, date, isolate) in myseqs.tsv to 
the directory mySub/ and create files for NCBI, ENA and GISAID at the same time:

    multiSub conv myseqs.fa myseqs.tsv mySub

Read all sequences and all annotation files (csv, tsv, xls) from mySeqs/ and write
files for NCBI and GISAID into mySub/:

    multiSub convDir mySeqs mySub -f ncbi,gisaid

## Manual NCBI uploads

Go to https://submit.ncbi.nlm.nih.gov/subs/genbank/, create a new submission
and when prompted, upload the ncbiSeqsAndSource.fa file. The file contains both
the sequences and the source tags, so you should not have to do anything else,
just click the "Continue" buttons.

If you receive an email with error messages some times later, you can download
the error report file and provide it via --skipFile to "multiSub conv".
Any sequences with errors will then be skipped. This will create a new
ncbiSeqsAndSource.fa file which you can upload. However, you can also check 
the box "ignore errors" during the submission so you do not have to worry
about skipping the sequences with errors.

## Manual ENA uploads

Go to https://www.ebi.ac.uk/ena/submit/sra/#newSubmission-sequenceChoice-start.

TODO.

## Automated Genbank uploads

Request an FTP username and password from gb-admin@ncbi.nlm.nih.gov

Make sure that you have run "multiSub init" and have a file ~/.multiSub/config.

Edit the file ~/.multiSub/config with your institutional information, name,
authors, email, etc. and also the NCBI username and NCBI password under "ncbiUser" and
"ncbiPass".

For more details see https://www.ncbi.nlm.nih.gov/viewvc/v1/trunk/submit/public-docs/genbank/SARS-CoV-2/ 

Convert your submission files into the NCBI FTP format:

    multiSub conv mySeqs.fa mySeqs.tsv mySub -f ncbi-ftp

Upload sequences in mySub/ to the NCBI FTP server as a test submission:

    multiSub up-ncbi mySub
    
Wait for a few hours. Retrieve the status and accessions of your submission
and write them to the file mySub/ncbiReport.xml

    multiSub down-ncbi mySub
    
Upload sequences in mySub/ to the NCBI FTP server as a real submission:

    multiSub up-ncbi mySub --prod

Download the report:

    multiSub down-ncbi mySub --prod

## Automated ENA uploads

Go to https://www.ebi.ac.uk/ena/submit/sra/#home to create an account.
Paste your new username and password into ~/.multiSub/config under enaUser and enaPass.

Go to https://www.ebi.ac.uk/ena/submit/sra/#newSubmission-studyChoice-start, create a study
aka project and paste its identifier into ~/.multiSub.conf as enaProj. It starts with "PRJEB".

Then, convert your submission files to the ENA XML format:

    multiSub conv mySeqs.fa mySeqs.tsv mySub

Upload sequences in mySub/ to the ENA server as a test submission:

    multiSub up-ena mySub
    
If all is fine, upload sequences in mySub/ to the ENA production server as a real submission:

    multiSub up-ena mySub --prod
    
You can then find the raw receipt with your ENA accessions in mySub/enaReceiptSample.DATE.xml
and a parsed tsv table with the accessions and your internal identifiers in mySub/enaAcc.tsv

The sequence upload is rather slow, every sequence takes a few seconds. Split the input
files into chunks and run the script in parallel if you have several thousand sequences. 
Contact us if you have trouble with this step.

If your sequence or sample uploads fail somewhere within a batch and you change
the meta data or sequences, note that some of them may have been uploaded
already. To force a re-upload of everything, use the --prefix option. If the
error happened on the production server, not in testing mode, it may be best to
read the ENA API documentation on how to reset your upload (look at the receipt XML)
or contact the ENA helpdesk or us. 

## Automated GISAID uploads

NOTE: I was unable to test this, because GISAID will not send me an upload token. It should
work though. Please contact me if you were able to test it or would share a token for testing.

Email CLIsupport@gisaid.org and request an upload token.

Download the file https://www.epicov.org/content/gisaid_uploader into the same directory where 
multiSub is located or in some directory in your PATH.

Run 

   ./gisaid_uploader CoV authenticate --cid YOURUPLOADTOKEN

Convert your data:

    multiSub conv seqs.fa seqs.tsv mySub

And upload it:

    multiSub up-gisaid mySub

GISAID upload error messages are written to mySub/gisaidFail.csv
