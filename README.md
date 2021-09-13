# multiSub

multiSub is a command-line tool to prepare and/or submit a SARS-CoV-2 genome
sequence to the NCBI Genbank, EBI ENA and GISAID sequence repositories. It can also
convert between these formats. This tool can be used by a "data
broker", a single institution that collects sequences from labs and submits
them to the sequence databases or it can be used by an individual lab. It's early research software, you will probably 
find bugs, please open a ticket on Github or email maxh@ucsc.edu if that happens, most of them
should be easy to fix now.

## Overview

multiSub accepts input sequences in fasta format and meta data in tsv, csv or GISAID (xls or csv) formats.
It will make some effort to clean the input data, e.g. skip missing sequences, strip flanking Ns,
or remove empty meta data and output warnings if that happens. It can then create one or multiple output files, 
in NCBI, NCBI-tag, NCBI-ftp, ENA-xml or GISAID-csv format and directly upload
to NCBI, ENA or GISAID.

The script takes care of the different ways to format the virus names (for
example, hCov-19 for GISAID, SARS-CoV-2 for NCBI), translates the different ways to specify 
the country, checks the date format and adds sequence IDs where needed. It does
not support more than the date and isolate and country fields, but other fields
can be easily added, just email examples to maxh@ucsc.edu.

There is really only a single table and a single fasta file needed. The different export steps
will pick out of the meta data table what they need. E.g. the field "Genome
Coverage" will be exported by the NCBI Genbank step into a "structured-comment"
field "Genome Coverage", and will also end up in the ENA fields "coverage" and
GISAID's "covv_coverage". The meta table field names should
either follow NCBI standards or be a GISAID file. As a matter of fact, there is 
an order to the steps: you first need to upload to NCBI Biosamples to obtain
Biosamples accessions, then you re-convert to add these IDs to the files, then
you can upload the new files to Genbank or SRA, with the Biosamples accessions
in them. The examples below should make this clear.

Many thanks to Stephan Fuchs and Kyanoush Yahosseini, Robert Koch Institut,
Berlin, for sending me their Python ENA uploader code, from which I copied. Also
thanks to the ENA Helpdesk and the NCBI Helpdesk for their quick replies.
Also to Kelsey Florek and Ethan Wang for bug reports. The NCBI bulk upload 
draws heavily from examples provided by Danny Park at the Broad Institute.
Without all of these people, this program would not have been possible.

## Installation and requirements

The script has usually no software requirements (see below). Just download it:

    wget https://raw.githubusercontent.com/maximilianh/multiSub/stable/multiSub

or:
 
    curl -O https://raw.githubusercontent.com/maximilianh/multiSub/stable/multiSub
    
Make it executable (not on Windows):

    chmod a+x multiSub

And run it:

    ./multiSub --help

Or run it on Windows (install Python first from https://www.microsoft.com/en-us/p/python-38/9mssztt1n39l):

    python multiSub

The script was tested on Python 2.7 and 3.6. If you do not plan to read GISAID xls files,
you do not need to install anything else.

If you want to read GISAID xls files, the script needs the xlrd Python package.
You can install xlrd with "pip install xlrd" or, if you are not administrator,
with "pip install xlrd --user".  If you use Mac OSX and do not have pip
installed yet, run the command "curl https://bootstrap.pypa.io/get-pip.py -o
get-pip.py && python3 get-pip.py"

Using Microsoft Windows ? Little command line experience? Please contact me at
maxh@ucsc.edu. The script runs on Windows, if you install Python from the
Microsoft Store. I can also provide a normal Windows .exe version.
Also, a drag-and-drop file box could be added for Windows GUI use.

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
- "gisaid" - for GISAID batch or API upload in .csv format: gisaid.csv and gisaid.fa
- "ena" - for ENA automated API sample uploads in XML format: ena.xml

## Configuration

Run

   multiSub init  # 'python multiSub init' on Windows)

Then edit the file ~/.multiSub/config with your favorite text editor. You can
set the name of your institute, your email address, your country, etc. To try
the examples below, you do not even need to run the 'init' step yet, the script
comes with default values.

## Example: convert files

The absolutely minimal example:

    printf '>CA-UCSC-123\nNNNNACTGT' > seq.fa
    printf 'isolate,date\nCA-UCSC-123,2021-03-03' > meta.csv
    ./multiSub conv seq.fa meta.csv mini/

Or on Windows:
    curl -O https://github.com/maximilianh/multiSub/blob/main/tests/mini/seqs.fa
    curl -O https://raw.githubusercontent.com/maximilianh/multiSub/main/tests/mini/meta.csv 
    python multiSub conv seqs.fa meta.csv mini/

This converts sequences from mySeqs.fa with annotations in mySeqs.tsv (fields:
seqId, date, isolate) to the directory mySub/. It fixes up the isolate names to
conform to the INSDC and GISAID formats, trims the sequences from flanking N
nucleotides and adds the location name ("USA" by default, but you can change
this in the configuration file). It creates files for NCBI Genbank and
Biosamples, ENA Analysis/Biosample and GISAID submission, all at the same time.

Here is a bigger example:

    mkdir my
    curl https://raw.githubusercontent.com/maximilianh/multiSub/main/tests/ucsc1/mySeqs.fa -o my/mySeqs.fa
    curl https://raw.githubusercontent.com/maximilianh/multiSub/main/tests/ucsc1/mySeqs.tsv -o my/mySeqs.tsv

    ./multiSub conv my/mySeqs.fa my/mySeqs.tsv mySub

A full copy of the output files is here: https://genome-test.gi.ucsc.edu/~max/multiSub/out/ucsc1/

Read all sequences and all annotation files (csv, tsv, xls) from mySeqs/ and write
files for NCBI and GISAID into mySub/:

    ./multiSub convDir mySeqs mySub -f ncbi,gisaid


## Possible meta annotation table field names (columns) and what happens to their data

There are four types of possible field names for the meta data:

1) the minimal ones: "isolate", "date" (alias "collection_date") and "location" (alias "country")
2) the following GISAID field names: covv_collection_date, covv_virus_name, covv_location, covv_assembly_method, covv_coverage, covv_seq_technology
3) NCBI source tags, listed here: https://www.ncbi.nlm.nih.gov/WebSub/html/help/genbank-source-table.html, most importantly isolate, collection_date and country (misnomer, it usually includes region and town)
4) the NCBI Structured comment field names: "Assembly Name", "Assembly Method", "Genome Coverage", "Sequencing Technology", see: https://www.ncbi.nlm.nih.gov/genbank/structuredcomment

You can also rename any of your own input fields with other names but similar
content to the NCBI names using the name mapping table in the configuration
statement "metaFieldMap". See the sample config file config.sample in this repository.

## Submission of the sample attributes as a manual NCBI Biosample upload

NCBI Biosample is a database that connects reads and assemblies and contains a couple of key-value entries, 
like sequencing coverage, the instrument or even the vaccination status. The list of all possible keys
for SARS-CoV-2 is here: https://www.ncbi.nlm.nih.gov/biosample/docs/packages/SARS-CoV-2.cl.1.0/
A Biosample accession is only required if you want to upload the raw FASTQ reads later to the SRA.

For interactive uploads, go to https://submit.ncbi.nlm.nih.gov, create a new Biosamples submission
and when prompted, upload the biosample.tsv file. 
Just click the "Continue" buttons until you are down. You should get 
the table with the accessions by email. Download this table into the output directory as 
biosampleAccs.tsv. The next "conv" run will then add these accessions to the
various files as cross-references.

## Submission of the consensus assembly as a manual NCBI Genbank upload

This is the most important database for public health purposes. 
Go to https://submit.ncbi.nlm.nih.gov/subs/genbank/, create a new submission
and when prompted, upload the ncbiSeqsAndSource.fa file. The multiSub fasta
file contains both the sequences and the source tags (meta data), so you should
not have to do anything else, just click the "Continue" buttons a few times in
the assistant.

Check the box "ignore errors" on the NCBI website, so you don't have to worry
about sequences that NCBI thinks are invalid, these will simply be skipped. 

If you forgot the check this box, and you receive an email with error messages
later and a rejected submission, you can download the error report file and
provide it via --skipFile to "multiSub conv".  Any sequences with errors will
then be skipped. This will create a new ncbiSeqsAndSource.fa file which you can
upload again. But the skip checkbox is easier.

Getting rejected is always difficult. See this page https://github.com/czbiohub/covidtracker_notes/tree/main/submission_to_public_repo with notes on how to deal with rejection by NCBI.

## Submission as manual ENA uploads

Go to https://www.ebi.ac.uk/ena/submit/sra/#newSubmission-sequenceChoice-start.

TODO: Is this actually possible to do interactively ? Need more help from ENA.

## Submission as automated NCBI uploads

Request an FTP username and password from gb-admin@ncbi.nlm.nih.gov

Make sure that you have run "multiSub init" and have a file ~/.multiSub/config.

Edit the file ~/.multiSub/config with your institutional information, name,
authors, email, etc. and also the NCBI username and NCBI password under "ncbiUser" and
"ncbiPass".

For more details see https://www.ncbi.nlm.nih.gov/viewvc/v1/trunk/submit/public-docs/genbank/SARS-CoV-2/ 

Convert your submission files into the NCBI FTP format:

    ./multiSub conv mySeqs.fa mySeqs.tsv mySub -f ncbi-ftp

Upload sequences in mySub/ to the NCBI FTP server as a test submission:

    ./multiSub up-genbank mySub
    
Wait for a few hours. Retrieve the status and accessions of your submission
and write them to the file mySub/ncbiReport.xml:

    ./multiSub down-ncbi mySub
    
Upload sequences in mySub/ to the NCBI FTP server as a real submission:

    ./multiSub up-genbank mySub --prod

Download the report:

    ./multiSub down-genbank mySub --prod

The sequence accessions should be in mySub/genbankAccs.tsv

## Submission as automated ENA uploads

Go to https://www.ebi.ac.uk/ena/submit/sra/#home to create an account.
Paste your new username and password into ~/.multiSub/config under enaUser and enaPass
(see above under NCBI Genbank how to create this file).

Go to https://www.ebi.ac.uk/ena/submit/sra/#newSubmission-studyChoice-start, create a study
aka project and paste its identifier into ~/.multiSub/config under enaProj (it starts with "PRJEB", include this prefix).

Then, convert your submission files to the ENA XML format:

    ./multiSub conv mySeqs.fa mySeqs.tsv mySub

Upload sequences in mySub/ to the ENA server as a test submission:

    ./multiSub up-ena mySub
    
If all is fine, upload sequences in mySub/ to the ENA production server as a real submission:

    ./multiSub up-ena mySub --prod
    
You can then find the raw receipt with your ENA accessions in mySub/enaReceiptSample.DATE.xml
and a parsed tsv table with the accessions and your internal identifiers in mySub/enaAcc.tsv

The sequence upload is rather slow, every sequence takes a few seconds. Split the input
files into chunks and run the script in parallel if you have several thousand sequences
Please contact me if this sentence is unclear or if you have trouble with the upload.

If your sequence or sample uploads fail somewhere within a batch and you change
the meta data or sequences, note that some of them may have been uploaded
already. But the upload alias can be used only once. To allow a re-upload of
everything, use the --prefix option, which will add a custom prefix to all
aliases. If the error happened on the production server, not in testing mode,
it may be best to read the ENA API documentation on how to reset your upload
(look at the receipt XML) or contact the ENA helpdesk or me. 

## Submission as automated GISAID uploads

Email CLIsupport@gisaid.org and request an upload token. For testing, use the
token "TEST-EA76875B00C3".

Convert your data, it will automatically create files in the format for GISAID:

    ./multiSub conv seqs.fa seqs.tsv mySub

Then, try to start the upload (it will automatically get the GISAID upload client into ~/.multiSub):

    ./multiSub up-gisaid mySub

This will fail on the first run, because you need to authenticate first, so let's authenticate now:

    python3 ~/.multiSub/gisaid-uploader CoV authenticate --cid TEST-EA76875B00C3

The GISAID uploader will ask you for your GISAID username and password. If you don't have one yet, create one on gisaid.org.
When this command is successful, note that a new file in the current working directory was created, "gisaid_uploader.authtoken".
The up-gisaid will work without authentication for the next 100 days from this working directory.

Now run the upload:

    ./multiSub up-gisaid mySub

With the token TEST-EA76875B00C3, data will never be processed or released.
Rejected sequences with GISAID upload errors are written to
mySub/gisaidFail.txt.

Note this page with how to deal with rejections: https://github.com/czbiohub/covidtracker_notes/tree/main/submission_to_public_repo

## Unusual cases

Mixing different file types: with the convDir command, you can merge datasets in different formats,
e.g. you can mix GISAID and NCBI data or mix different table formats. Fields
that exist in one input file, but not in the others, will be set to the empty
string.

