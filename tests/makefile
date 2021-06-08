all: test-ucsc1 test-gisaid1

mkdir:
	mkdir -p expected

test-gisaid1: mkdir
	../multiSub conv gisaid1/ucsc_gi_csrun005_all_seqs_v1.fasta gisaid1/20210403_ucsc_gi_csrun005_metadata_v1.xls expected/gisaid1/

test-ucsc1: mkdir
	../multiSub conv ucsc1/all.fa ucsc1/all.tsv expected/ucsc1/ --skip ucsc1/SUB9389947-detailed-error-report.txt
