# DASHit Pipeline



### gRNA设计参数

使用各种方法设计gRNA时，均需要选择一些参数以保证gRNA的质量

- GC%：GC含量高，效率高但易脱靶

  > It is reported that GC content of sgRNA is important for the efficiency of CRISPR/Cas9 system [(Ren et al., 2014)](http://www.sciencedirect.com/science/article/pii/S2211124714008274), and 97% of sgRNAs have a GC Content between 30% and 80% [(Liang et al., 2016)](http://www.nature.com/articles/srep21451).





### Log in docker

##### Overview

1. 获取fasta文件

2. 去除fasta文件中的接头序列

3. 使用`crispr——sites`发现一个gRNA集合（20 nt）

   > 可选：输入on/off-target的fasta文件，使用cripsr-sites得到txt格式的位点列表

4. QC：

   - 使用`dashit_filter`筛除低质量gRNA（GC%, repeats, homopolymers）（自己设置参数）
   - 根据on/off-target sites筛选gRNA

5. 设定gRNA数量，使用`optimize_guides`得到gRNA集合（应该多设置一些数量尝试一下）（迷惑参数，需要多少个gRNA匹配才从未匹配集合中剔除read，pipeline中一般设置为1）

6. 打开生成的csv文档，看gRNA的饱和情况，选择（5）中的参数

7. 使用`score_guides`打分验证csv的效果



##### Practice (On PC)

1. 使用github提供的命令下载docker镜像，会从官网直接下载。将本地含有fasta文件的目录与容器中的/data目录挂载

   ```dockerfile
   docker run -it -v ~/desktop/data:/data czbiohub/dashit bash
   #镜像中已经安装了go和python环境
   ```

2. 在/data目录中下载seqtk

   ``` dockerfile
   apt install seqtk #网络上的github源方法会出错，这个最好用
   #如有必要使用seqtk转化文件格式为fasta
   cd seqtk
   make #seqtk需要make一下, 可以使用seqtk seq检查安装
   seqtk seq -A ../NG-mix-rRNA_combined_R1.fastq > ../input.fasta
   ```

3. 使用cutadapt去接头

   ``` dockerfile
   python3 -m pip install cutadapt==2.4 #安装cutadapt
   cutadapt -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -o cut-input.fasta input.fasta
   
   #日志如下：
   This is cutadapt 1.15 with Python 3.6.9
   Command line parameters: -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -o cut-input.fasta input.fasta
   Running on 1 core
   Trimming 1 adapter with at most 10.0% errors in single-end mode ...
   Finished in 3705.10 s (259 us/read; 0.23 M reads/minute).
   
   === Summary ===
   
   Total reads processed:              14,279,717
   Reads with adapters:                 1,436,690 (10.1%)
   Reads written (passing filters):    14,279,717 (100.0%)
   
   Total basepairs processed: 2,141,957,550 bp
   Total written (filtered):  2,090,538,144 bp (97.6%)
   
   === Adapter 1 ===
   
   Sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC; Type: regular 3'; Length: 34; Trimmed: 1436690 times.
   
   No. of allowed errors:
   0-9 bp: 0; 10-19 bp: 1; 20-29 bp: 2; 30-34 bp: 3
   
   Bases preceding removed adapters:
     A: 1.7%
     C: 79.7%
     G: 0.9%
     T: 17.5%
     none/other: 0.2%
   ```

4. 运行crispr_sites程序：

   ```
   cat cut-input.fasta | crispr_sites -r > input_sites_to_reads.txt
   ```

   > 注意，这里如果docker默认内存（2G）不够，会显示killed，程序失败
   >
   > 本次运行时，我们提取了完整fasta文件的1/3（10000000行）,否则PC内存不足（8G）
   >
   > 本次运行过夜，不能确定程序完全运行结束（在output阶段卡死了）

5. gRNA的QC

   ```dockerfile
   dashit_filter --gc_freq_min 5 --gc_freq_max 15 input_sites_to_reads.txt > input_sites_to_reads_filtered.txt
   #INFO:dashit_filter.dashit_filter:filtering sites for poor structure with parms {'gc_frequency': (5, 15), 'homopolymer': 5, 'dinucleotide_repeats': 3, 'hairpin': {'min_inner': 3, 'min_outer': 5}}
   #INFO:dashit_filter.dashit_filter:removed 974538 sites from consideration due to poor structure
   #INFO:dashit_filter.dashit_filter:Done filtering guides, removed 974538 out of 2123489 guides
   ```

6. 从中选取500个guides观察效果

   ```dockerfile
   optimize_guides input_sites_to_reads_filtered.txt 500 1 > guide.csv
   #Total number of reads: 5000000
   #Largest # of reads hit by a single guide is 1152841
   #500 sites covered 4472925 reads, # reads: 5000000
   ```

7. 观察输出的gRNA

   ![image-20210308143101020](/Users/zhiwei/Library/Application Support/typora-user-images/image-20210308143101020.png)

   > 500个guides最多可以覆盖到89%左右，这与rRNA的比例符合吗？

8. 对原fasta文件做一个打分
  ```dockerfile
  score_guides guide.csv input.fasta
  ```

##### Practice (On Cnode)

1. 根据官网教程，需要分别安装go、python环境。使用conda安装即可。

   > 如果创建了conda虚拟环境，就要在环境中独立下载python，不可以直接使用服务器的python

   ```bash
   #安装miniconda
   wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
   bash miniconda/Miniconda3-latest-Linux-x86_64.sh
   
   #创建一个虚拟环境，并配置go和git
   conda create -n testEnv
   source activate testEnv #从此进入了testEnv环境
   conda install go
   # conda install python 注意cutadapt不支持3.9以上python。cnode自带3.6可用，所以其实不需要安装python……
   conda install git
   go version
   python -V
   git --version #检查一下安装
   ```

2. 安装dashit

   ```bash
   git clone https://github.com/czbiohub/dashit
   cd dashit
   python3 -m venv ../.virtualenvs/dashit
   source ../.virtualenvs/dashit/bin/activate
   make # 这里使用make很容易出现error，有各种bug，我删除testEnv环境再来一次，试了两三次make就好了……
   sudo make install #如果并非root用户，需要修改Makefile文件
   ```

   ``` bash
   #修改Makefile文件的方法：
   #在make后的dashit文件夹中找到Makefile
   vi Makefile
   #修改第一行的Prefix，把/usr/local改为虚拟环境的bin路径，我这里是：
   PREFIX ?= /BioII/lulab_b/guzhiwei/DASH/.virtualenvs/dashit
   make install
   ```

3. 安装seqtk和cutadapt

   搜索conda资源，使用conda安装即可

   ``` bash
   cd /BioII/lulab_b/guzhiwei/DASH
   #安装cutadapt时，注意python版本
   conda install -c bioconda seqtk 
   conda install -c bioconda cutadapt
   ```

4. 文件操作阶段

   ```bash
   seqtk seq -A ./data/NG-mix-rRNA_combined_R1.fastq > ./data/input.fasta
   #这里用了参数-a，对应的是read1 3'引物
   cutadapt -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -o ./data/cut-input.fasta ./data/input.fasta
   
   
   #扫描全部PAM sites
   cd ./data
   cat cut-input.fasta | crispr_sites -r > input_sites_to_reads.txt
   #Finished reading input.
   #Total lines: 28559434
   #Total bases: 2090538144
   #Sorting 346987983 candidate guides.
   #Unique sites to reads: 11381423
   #Guide 243335882 had largest number of reads: 3313865
   #Outputting 11381423 unique guides.
   
   ```

   ```bash
   #cutadapt的log
   Processing reads on 1 core in single-end mode ...
   Finished in 154.16 s (11 us/read; 5.56 M reads/minute).
   
   === Summary ===
   
   Total reads processed:              14,279,717
   Reads with adapters:                 1,436,690 (10.1%)
   Reads written (passing filters):    14,279,717 (100.0%)
   
   Total basepairs processed: 2,141,957,550 bp
   Total written (filtered):  2,090,538,144 bp (97.6%)
   
   === Adapter 1 ===
   
   Sequence: AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC; Type: regular 3'; Length: 34; Trimmed: 1436690 times.
   
   No. of allowed errors:
   0-9 bp: 0; 10-19 bp: 1; 20-29 bp: 2; 30-34 bp: 3
   
   Bases preceding removed adapters:
     A: 1.7%
     C: 79.7%
     G: 0.9%
     T: 17.5%
     none/other: 0.2%
   ```

   ```bash
   #产生ontarget位点
   cat rRNA.fa | crispr_sites > ontarget_sites.txt
   #crispr_sites v1.2-3-ge65da89
   #crispr_sites -h for usage
   #Finished reading input.
   #Total lines: 1542
   #Total bases: 104213
   #Sorting 27831 candidate guides.
   #Outputting 4324 unique guides.
   
   #重装，永远的神
   dashit_filter --gc_freq_min 5 --gc_freq_max 15 --ontarget ontarget_sites.txt input_sites_to_reads.txt > input_sites_to_reads_filtered.txt
   #INFO:dashit_filter.dashit_filter:Killing ontarget server
   #INFO:dashit_filter.dashit_filter:offtarget file not specified with --offtarget, will not perform any offtarget filtering
   #INFO:dashit_filter.dashit_filter:Filtering guides for quality
   #INFO:dashit_filter.dashit_filter:filtering sites for poor structure with parms {'gc_frequency': (5, 15), 'homopolymer': 5, 'dinucleotide_repeats': 3, 'hairpin': {'min_inner': 3, 'min_outer': 5}}
   #INFO:dashit_filter.dashit_filter:removed 1406 sites from consideration due to poor structure
   #INFO:dashit_filter.dashit_filter:Done filtering guides, removed 11380466 out of 11381423 guides
   
   
   optimize_guides input_sites_to_reads_filtered.txt 300 1 > guides.csv
   # 300 sites covered 12157902 reads, # reads: 14279717
   
   #下载文件，发现大约饱和在85%
   scp -P10000 guzhiwei@166.111.156.19:/BioII/lulab_b/guzhiwei/DASH/data/guides.csv ~/desktop
   #回到节点，计算一个更小的gRNA集合（其实125个也偏大）
   optimize_guides input_sites_to_reads_filtered.txt 125 1 > guides_less.csv
   #125 sites covered 12137934 reads, # reads: 14279717
   
   #最后打分
   score_guides guides_less.csv input.fasta
   #125 guides in guides_less.csv vs. input.fasta hit 12137795/14279717 = 85.00%
   #Parsing FASTA took 35m36.267950245s
   #Total is: 14279717
   ```

   


-----------

# DASH Introduction

[TOC]

Got junk sequences in your sequencing fastqs? Many RNA-seq experiments are riddled with repetitive sequences which are of no value to the scientific question you are asking. Some examples include ribosomal RNA and hemoglobin. DASH is a CRISPR-cas9, NGS technology which depletes abundant sequences, increasing coverage of your sequences of interest.

In the lab, you take final NGS libraries ready for sequencing, and combine them with cas9-gRNAs which cut up your repetitive sequences into small fragments. You then size select and amplify your remaining DNA, which should contain most or only your sequences of interest. Your library can then be sequenced. For more details, see our protocol on protocols.io: dx.doi.org/10.17504/protocols.io.y9gfz3w

# What is DASHit?

In order for DASH to be effective, you want to pick the fewest guides which hit the most of your repetitive sequences. The best way to know the most repetitive sequences in your sequencing is by designing guides which hit your actual reads. David has implemented a greedy algorithm to solve this set cover problem, and created a software package called DASHit to perform automated guide design for DASH experiments.

DASHit uses your reads from a preliminary, low-depth sequencing of your samples, and identifies crispr-cas9 cut sites. It has a filtering function for the guides it identifies, which allows you to specify on and off target sequences, and filter based on GC content and secondary structure. It then optimizes your guide list by giving you a set number of guides which hit the most reads.

DASHit also has functionality to score guides against a fasta. You can use this to test how many of your reads would be hit by a given guide set.

# Cupcakes 05/16/19 - let's get started!

Today, we will be using a set of preliminary reads on mouse tissue to design a set of 96 DASH guides. I have written a wrapper script which encompasses most of this work, which will be made available on the DASHit github repository, and remain up-to-date as a reference for anyone wishing to run this pipeline.

## Launching an AWS instance

The czbiohub-specops image has dashit installed on it. However, that is an outdated version, so we will be running dashit on a clean instance, with an image I made containing seqtk, bedtools and cutadapt.

Replace the instance name with your name.

```
 aegea launch --iam-role S3fromEC2 --ami-tags Name=dashit-image-v3 -t m4.large --no-dns dashit-yourname --duration-hours 2
```

### Login to your AWS instance

```
aegea ssh ubuntu@dashit-yourname
```

Change directory into volume /mnt/data

```
cd /mnt/data/
```

## Install DASHit (SKIP SINCE USING IMAGE)

Installing DASHit requires python3 and GO.

```
aws s3 cp s3://alyden-bucket/Mouse_DASH/cupcakes/DASHit_installation.sh . --region "us-east-2"
```

You will be prompted to read the license agreement. Click q when you are done, and type I agree, and hit enter.

```
bash DASHit_installation.sh
```

Here are the contents of the script:

```
wget https://dl.google.com/go/go1.12.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.12.5.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
python3 -m venv ~/.virtualenvs/dashit
source ~/.virtualenvs/dashit/bin/activate
git clone https://github.com/czbiohub/guide_design_tools.git
cd guide_design_tools
make install
ln -s /mnt/data/guide_design_tools/vendor/special_ops_crispr_tools/offtarget/offtarget /home/ubuntu/bin/offtarget
ln -s /mnt/data/guide_design_tools/vendor/special_ops_crispr_tools/crispr_sites/crispr_sites /home/ubuntu/bin/crispr_sites
```

## Enter DASHit virtual environment

```
source ~/.virtualenvs/dashit/bin/activate
```

## Copy files from AWS

Make a new directory and change into it

```
mkdir DASH
cd DASH
```

Sync files needed for this from AWS. This will take a few minutes.

```
aws s3 sync s3://alyden-bucket/Mouse_DASH/cupcakes/ . --region "us-east-2"
```

`ls` to take a look at what you downloaded! There should be a series of paired-end reads, two genome fastas and a bed file.

## Preparing your reads to run DASHit

## SUBSAMPLE

**Subsample files**: if your files are large, we recommend randomly subsampling your files to 100k reads. This can be done on a fastq or fastq.gz using `seqtk`. Use the same seed to maintain paired information.

### Install seqtk (SKIP SINCE USING IMAGE)

```
sudo dpkg --configure -a
sudo apt install seqtk
```

**Example commands**

```
seqtk sample -s100 input_R1.fastq 100000 > sub100k_input_R1.fastq
seqtk sample -s100 input_R2.fastq 100000 > sub100k_input_R2.fastq
```

### Subsample all of your reads to 10000 in a for loop

```
for i in GI01_*fastq.gz; do seqtk sample -s100 $i 10000 > sub10k_${i:0:-3}; done
```

## TRIM ADAPTORS

This step is crucial for maintaining library integrity! The last thing we want is to design a guide which cuts your adaptor sequence in it, which would render your library unsequencable. To avoid this, trim all adaptors off your reads using your favorite bioinformatics tool. We do this using `cutadapt`, but other programs such as `trimmomatic` or `fastp` also work well.

### Install cutadapt (SKIP SINCE USING IMAGE):

```
pip install cutadapt
```

or

```
sudo apt install python-cutadapt
```

**Example command**

```
cutadapt --report=minimal -j 8 -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT -o cut-sub10k_input_R1.fastq -p cut-sub10k_input_R2.fastq sub10k_input_R1.fastq sub100k_input_R2.fastq
```

### Cut adaptors off all of your reads in a for loop

```
for i in sub10k*R1*; do cutadapt -j 8 -a AGATCGGAAGAGCACACGTCTGAACTCCAGTCAC -A AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT -o cut-$i -p cut-${i:0:-11}2_001.fastq $i ${i:0:-11}2_001.fastq -m 40; done
```

## CONVERT TO FASTA

**Convert from fastq to fasta if needed**: you can convert from fastq to fasta in many ways; we use `seqtk` for this as well.

**Example commands**

```
seqtk seq -A  cut-sub100k_input_R1.fastq > cut-sub100k_input_R1.fasta
seqtk seq -A  cut-sub100k_input_R2.fastq > cut-sub100k_input_R2.fasta
```

### Convert all reads from fastq to fasta in a for loop

```
for i in cut-sub10k_GI01_*fastq; do seqtk seq -A $i > ${i:0:-1}a; done
```

Let's take a look and confirm they are fastas with trimmed adaptors:

```
head *fasta
```

## Running crispr_sites to identify all PAM sites in your reads

```
crispr_sites -h
```

crispr_sites will take your reads from stdin and output a list of guides. If you use -r flag (for reads), it will track which read it hit.

```
cat cut-sub10k_GI01_*fasta | crispr_sites -r > mouse_reads_crispr_sites.txt
```

Take a look at the crispr_sites file!

## Filtering guides with dashit-filter (optional)

If you want to use these guides as is, you can run straight into optimize guides, to identify the guides which hit the most reads, using the above crispr_sites file. However, most people want to filter their guides for on or off target activity.

Let's filter by both on and off target activity.

**On-target**: I want to hit the ribosomal regions of the mouse.

**Off-target**: I do not want to hit the rest of the mouse genome, because I am interested in mouse transcriptome, and I also do not want to hit the *Candida albicans* genome, because I am trying to quantify infection.

### Preparing your on and off target files

You can use `bedtools` to help generate your on-target and off-target fastas.

### Install bedtools (SKIP SINCE USING IMAGE)

```
sudo apt install bedtools
```

Note: having trouble with sudo sometimes on instances because username not in /etc/host, works well otherwise...try some of these options if it is not working https://bedtools.readthedocs.io/en/latest/content/installation.html

### **On-target files**

If you have a bed file annotating on-target regions, you can use `bedtools getfasta` to generate an on-target fasta. I created a bed file from UCSC Genome Browser, pulling all regions which were annotated as ribosomal in the mm10.fa genome. I then downloaded the mm10.fa.

Using these two files, bedtools will pull out the on-target regions of the mm10 genome.

```
bedtools getfasta -fi mm10.fa -bed mm10_rRNA.bed -fo on-target_mm10_rRNA.fa
```

### **Off-target files**

You can use the same bed file to mask on-target regions of a fasta to generate an off-target fasta, using `bedtools maskfasta`, with the `-mc -` option. This will replace all regions in the bed file with a `-` instead of a nucleotide. Since I want to preserve other elements of the mouse genome, I will mask everything that is not rRNA, as annotated by my bed file. [This step will take 1-3 minutes]

```
bedtools maskfasta -bed mm10_rRNA.bed -fi mm10.fa -fo off-target_mm10.fa -mc -
```

Quick check:

```
grep "-" off-target_mm10.fa | head
```

Since I am also hoping to avoid hitting the *C. albicans* genome with my guides, I will add that to my off-target file. [This step will take 1-3 minutes]

```
cat off-target_mm10.fa CalbicansSC5314_genome_allNs.fasta > off-target_mm10_SC5314.fa
```

*Note*: Both fastas should contain no letters other than A, G, C, T or N.

### Running crispr_sites on your on and off target fastas

We will now run crispr_sites (like before), without the -r flag since these are not reads. This may take 4-5 minutes.

```
cat on-target_mm10_rRNA.fa | crispr_sites > on_target_crispr_sites.txt
cat off-target_mm10_SC5314.fa | crispr_sites > off_target_crispr_sites.txt
```

### Running dashit-reads-filter to include your on-target guides and exclude off-target guides

We can now use our on-target crispr sites file, our off-target crispr sites file and our reads crispr sites file to run dashit-reads-filter. Run the following command to see all the options for filtering, including structural and GC content flags.

```
dashit-reads-filter -h
```

On default settings, a site will be removed if any of the following are true:

1. G/C frequency too high (> 15/20) or too low (< 5/20)
2. Homopolymer: more than 5 consecutive repeated nucleotides
3. Dinucleotide repeats: the same two nucelotides alternate for > 3 repeats
4. Hairpin: complementary subsequences near the start and end of a site can bind, causing a hairpin

We will require perfect matches to define on and off target activity.

```
dashit-reads-filter mouse_reads_crispr_sites.txt --offtarget off_target_crispr_sites.txt --offtarget_radius 5_10_20 --ontarget on_target_crispr_sites.txt --ontarget_radius 5_10_20 --filtered_explanation why_final_mouse_rRNA_guides.csv > mouse_rRNA_final_crispr_sites.txt
```

## Optimize guides

Now that we have a filtered guide list, we can use this to pick the fewest guides which will hit the greatest number of reads. We run optimize guides, which allows us to specify the number of desired guides, as well as how many times a read has to be cut before it is considered 'hit'. For us, we will design 500 and say each read must only be hit 1 time.

```
optimize_guides mouse_rRNA_final_crispr_sites.txt 500 1 > mouse_rRNA_DASH_500_guides.csv
```

Take a look at this file! How much can we DASH with 500 guides? 250? 100?

The file will contain five components.

1. Guide (number)
2. Site (the sequence)
3. Number of reads covered by site
4. cumulative number of reads covered
5. cumulative percent of reads covered

```
less mouse_rRNA_DASH_500_guides.csv
```

Let's take the first 96 guides and score them against our original files.

```
head -98 mouse_rRNA_DASH_500_guides.csv > mouse_rRNA_DASH_96_guides.csv
```

## Score guides

We can check *in silico* see how each file would be DASHed if we were to use this guide set.

**Example command**

```
score_guides mouse_rRNA_DASH_96_guides.csv sub10k_GI01_7_S3_R2_001.fasta
```

Run score_guides in a for loop to check out all of the files:

```
for i in cut-sub10k_GI01_*fasta; do score_guides mouse_rRNA_DASH_96_guides.csv $i >> mouse_rRNA_final96_guides_scored.txt; done;
```

Take a look at the `mouse_rRNA_final96_guides_scored.txt` file!

If you want, you can format this txt file into a CSV using a custom Python script:

```
python3 ../guide_design_tools/dashit/contrib/score_guides_scripts/DASH_csv_format.py mouse_rRNA_final96_guides_scored.txt
```

Your formatted CSV will contain five columns

1. Guide library
2. Your filename
3. Total Reads DASHed (hit by score_guides)
4. Total Reads in sample
5. Percent DASHed

```
less mouse_rRNA_final96_guides_scored.csv
```

## Running the design_guides_wrapper script

This script will run crispr_sites, optimize_guides, score_guides, and the formatting Python script. Once you have created your on-target and off-target fastas, you can input the following into the wrapper:

Use bash to run the script and follow with 3 arguments

1. the file prefix for all of your read fastas
2. the file name of your on-target fasta
3. the file name of your off-target fasta

```
bash design_guides_wrapper.sh cut-filt-85-98-90_sub100k_GI01 ontarget_mm10-rRNA_region_UPPERcase.fa masked_mm10_SC5314_genomes_UPPERcase.fa
```

### Your final guides/optimize_guides output file

Your final guides CSV (`final_guides.csv`) will contain five components.

1. Guide (number)
2. Site (the sequence)
3. Number of reads covered by site
4. cumulative number of reads covered
5. cumulative percent of reads covered

You should use this file to generate an elbow plot, with guide sequence on the x-axis and number of reads covered by site/total reads hit on the y-axis.

### Your formatted score_guides output file

We will then run optimize_guides to determine the number of reads hit by each guide, and run score_guides on your original reads with the full guide set to see how much is DASHable. The optimize_guides output can be used to make an elbow plot to determine the optimal number.

Your formatted CSV (`final_guides_scored.csv`) will contain five columns

1. Guide library
2. Your filename
3. Total Reads DASHed (hit by score_guides)
4. Total Reads in sample
5. Percent DASHed

This file will tell you how much of your library would be DASHed by the full guide set. Use the elbow plot to determine how many guides would be most effective and rerun score_guides.

### Other files created

Files with `crispr_sites.txt` will contain guide sequences in your reads, on-target or off-target fastas which will be compatible with optimize_guides. A txt file called `final_guides_scored.txt` is generated containing score_guides output before it is formatted into a CSV.