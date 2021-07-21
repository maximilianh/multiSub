all: test-ucsc1 test-gisaid1 test-bulk1

mkdir:
	mkdir -p out/{ucsc1,gisaid1,bulk1,kelsey}


test-mini: mkdir
	../multiSub conv mini/seqs.fa mini/meta.tsv out/mini/

test-gisaid1: mkdir
	../multiSub conv gisaid1/ucsc_gi_csrun005_all_seqs_v1.fasta gisaid1/20210403_ucsc_gi_csrun005_metadata_v1.xls out/gisaid1/

test-ucsc1: mkdir
	../multiSub conv ucsc1/mySeqs.fa ucsc1/mySeqs.tsv out/ucsc1/ --skip ucsc1/SUB9389947-detailed-error-report.txt -a ucsc1/gisaidAccs.tsv

test-kelsey:
	../multiSub conv kelsey/test.fa kelsey/test.csv out/kelsey/
	
test-bulk1: mkdir
	mkdir -p bulk1
	rm -f bulk1/*.fa bulk1/*.tsv
	cp gisaid1/* ucsc1/* bulk1
	../multiSub convDir bulk1 out/bulk1/ -a bulk1/gisaidAccs.tsv

test-csv-gisaid: mkdir
	../multiSub conv out/ucsc1/gisaid.fa out/ucsc1/gisaid.csv out/csv-gisaid

test-combine: mkdir
	../multiSub conv ucsc1/mySeqs.fa ucsc1/mySeqs.tsv combine/mySeqsFnames.tsv out/combine/

test-kelsey: mkdir
	../multiSub conv kelsey/test.fa kelsey/test.csv out/kelsey


up-ncbi:
	../multiSub up-ncbi out/gisaid1/

down-ncbi:
	../multiSub down-ncbi out/gisaid1/

up-ena:
	../multiSub up-ena out/ucsc1/

up-gisaid:
	../multiSub up-gisaid expected/gisaid1/
