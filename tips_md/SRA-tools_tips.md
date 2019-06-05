# `Eutils` related , also some `enaBrowserTools`

### Getting basic info from a BioProject in xml

`esearch -db sra -query PRJNA257197`

### `Eutils`, getting a `csv` table with info about experiments

This can be used with `sra` category, run,  experiment, sample, project

`esearch -db sra -query PRJNA257197|efetch -format runinfo > runinfo.txt`

`cat runinfo.csv | cut -f 1 -d ',' | head`

### Creating a list of IDs and downloading them
`cat runinfo.csv | cut -f 1 -d ',' | grep SRR | head > runids.txt`

`cat runids.txt | parallel fastq-dump --split-files {}`

### Display info about samples from GEO (normal text as output)

`esearch -db gds -query GSE80357 | efetch -format runinfo|grep -E
'^[1-9]+|Sample\s+Accession'`

### Docsum

` -format runinfo` sometimes won't give you all the information needed, and
it may miss important details, for example if a sample is a control or not, or
any information about experimental conditions, etc. `--format docsum` is useful
in this case:

`esearch -db sra -query PRJNA257197 | efetch -format docsum > docsum.xml`

`cat docsum.xml | xtract -pattern DocumentSummary -element
Bioproject,Biosample,Run@acc,Experiment@name | head`

Make it a bit more readeable:

`alias pretty="python -c 'import sys;import xml.dom.minidom;\
s=sys.stdin.read();print(xml.dom.minidom.parseString(s).toprettyxml());'"`

`cat docsum.xml | pretty | less`

### Downloading with `fastq-dump`

<https://edwards.sdsu.edu/research/fastq-dump/>

`fastq-dump --outdir fastq --gzip --skip-technical  --readids --read-filter pass --dumpbase --split-3 --clip SRR_ID`

Also, see
<https://github.com/ncbi/sra-tools/issues/127>

### Check if a sequence from SRA is single or paired-end with `fastq-dump`

`wc -l` should output 8 if it is paired-end data

`fastq-dump -I -X 1 -Z --split-spot SRS2158877 |wc -l`

```bash
x=$( $SRAtool -I -X 1 -Z --split-spot "$SRAfolder/$SRAbase" | wc -l )
if [ "$x" == "8" ]; then 
  echo "$SRAfolder/$SRAbase contains paired-end sequencing data, dumping..."
  fastq-dump -I --split-files
else
    $SRAtool -O $fastqout $SRAfolder/$SRAbase
fi
```
Check this <https://github.com/MaayanLab/archs4/blob/master/docker/awsstar/scripts/sradump.sh>


### About `enaDataGet`,`enaGroupGet` and their types of accessions accepted

<https://github.com/enasequence/enaBrowserTools/issues/42>

>Both of those are sample accessions and therefore can only be used with enaGroupGet regardless of how many runs or experiments they contain.

>enaDataGet is for experiment (ERX, DRX, SRX) and run (ERR, DRR, SRR) accessions

>enaGroupGet is for sample (ERS, DRS, SRS, SAME, SAMN, SAMD) and project (ERP,DRP, SRP, PRJE, PRJD, PRJN) accessions

>enaDataGet knows what data type you want to download based on the accession.
>enaGroupGet needs you to tell it which type of data (read data, sequences,
>assemblies) should be downloaded for that sample/project.

```bash
# Aspera client 3.6 failed, `-a` option (cluster's module)
# SRP073391 is a study
#Try `parallel`

enaGroupGet --group read  -f fastq -m  SRP073391 
```
## EDirectCookbook

https://ncbi-hackathons.github.io/EDirectCookbook/