### Display the matched term and its column number (useful for headers)

`awk -F"," '{for(i=1;i<=NF;++i) if($i~/BioProject/) print i, $i}' file`

### Display the whole column of the matched term
`awk -F"," '{for(i=1;i<=NF;++i) if($i~/BioProject/) a=i; print $a}'`

#### Create a list of files with full paths

**Remember** - Always quote command substitutions to avoid problems

`find "$(pwd -P)" -type f -name '*fastq.gz' > fastq_fullpath.txt` Can pipe find to
`sort -h` for human sorting before saving

### Soft-linking files to the actual directory 

```bash
files="$(find /Mulla_elife_2017/rnaseq_reads/ -type f -name '*[0-9].fastq.gz')"

for f in $files; do ln -s $f ./; done
```

#### To add `n` number of tabs - useful for creating tables where a header will be added later

Here two tabs
`paste samples.txt -d '\t' fullpaths.txt |sed 's-\t-\t\t-g' > test.tsv`

#### Insert a header in a file (`1 = first line; i=insert, -i=in place`)
`sed -i '1 i\sample\tunit\tfq1\tfq2' test.tsv`

#### Commenting the matched line and x lines after it
This example works for commenting out environments from `snakemake` rules
`grep -rl conda * | xargs sed -i -e '/conda/, +1 s/^/#/'`

### This prints the name of the files and then head, -n8 because those are two reads

`parallel "echo ------{}------ && zcat {} | head -n8" ::: *fastq.gz`

### `tar` and compress `gzip` using multiple cores

By default, `pigz` uses all available cores and in this way is not possible
to specify any parameter

`tar -I pigz -cf snake_cluster.tar.gz scripts/ snakemake-wrappers/ Files/ ...`

This way we can specify the number of cores used and any other parameters:
`tar -cf - scripts/ snakemake-wrappers/ | pigz -p 4 > snake_cluster.tar.gz`

#### Sort `du -sh *`

"human sorting" `-h`, takes into account units (Gigas at last), `-c` produces a
grand total

`du -shc * | sort -k1 -h` 

#### `fastqc` doesn't parallelize computations per file

The `-t n` option makes `fastqc` to run in a maximum of `n` files in parallel,
but only one core per file is used. So, `GNU parallel` isn't necessary here.


#### Using `sed` to replace a dot in the limit of a word
For example, this works for replacing a dot, at the beginning of a column, with two dots.

`sed -i 's-\B\.-..-g' units.tsv `

### Running `jupyter notebook/lab` in the cluster and executing it in `localhost`

In the cluster execute this and remember do it from a relatively "root"
directory, to have access to as many as possible directories,
also,remember to active the proper `conda env` :

```
jupyter-lab --no-browser --port=8889 --ip=0.0.0.0

jupyter-notebook --no-browser --port=8889 --ip=0.0.0.0
```
`ssh` tunel from your local machine:

`ssh -N -f -L 8889:nodename:8889 user@cluster.edu`

### `grep` for errors

`grep -Ein 'warning|error|fail' *files`


### `parallel --plus` to replace multiple "extensions" and others tricks.

tutorial

<https://www.gnu.org/software/parallel/parallel_tutorial.html#More-pre-defined-replacement-strings-with---plus>

#### Take out 3 "extensions" plus the `/` to take the path out
```
$ parallel --plus echo {/...} ::: dir/sub/file.ex1.ex2.ex3
file
```

```
$parallel --plus echo {..} ::: dir/sub/file.ex1.ex2.ex3
dir/sub/file.ex1
```
> {+foo} matches the opposite of {foo}
```
$parallel --plus echo {+..} ::: dir/sub/file.ex1.ex2.ex3
ex2.ex3
```

```
parallel --plus echo {/sub/pre} ::: dir/sub/file.ex1.ex2.ex3
dir/pre/file.ex1.ex2.ex3
```


#### `parallel --colsep`

Pass columns from a `tsv`. It May be combined with `--plus`

`parallel --colsep '\t' --plus "echo {1..} --- {2}" :::: reads.tsv`


#### When using `awk` with `parallel`. Avoiding problems with expanded and scaped 
#### characters



```
AWK="awk 'NR==2 || NR==3 {print \$0, FILENAME}'"
parallel "$AWK {}" ::: *.quality
```

### `find` output sorted by modification date

`find . -printf "%T@ %Tc %p\n" | sort -n`

### Sort files from `find` with `ls`
for examples by size:

`find . -type f -name '*fastq.gz' |xargs ls -lSrh`

### `seqtk`

#### Sample according to a list of reads names

`seqtk subseq ../SRP073391/reads/SRR3396381_1.fastq.gz <( cut -f 1 -d ' '
salmon_quant/SRR3396381/aux_info/unmapped_names.txt)| pigz -p 12 -c - >
out.fq.gz`

#### subsample using `pigz`

`seqtk sample -s100 ../rnaseq_reads/SRR5494630.fastq.gz 1000000 | pigz -p 4 -c - > test_gzip_pip.fastq.gz`
