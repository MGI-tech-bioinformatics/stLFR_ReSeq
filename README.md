# stLFR_ReSeq
Tools of stLFR(Single Tube Long Fragment Reads) Resequencing data analysis.

Introduction
----------------
    Tools of stLFR (Single Tube Long Fragment Reads) data analysis.
    stLFR FAQs is directed to bgi-MGITech_Bioinfor@genomics.cn.

Versions
----------------
    V1.4
    Updates:
	1. Use a new program that can bring 3-5 times efficiency improvement for barcode splitting
	2. Upgrade SOAPnuke (v2.1.7) to improve operating efficiency
	3. Optimize the operation mode and provide help information
	4. Repair haplotype visualization graphics BUG
	5. Added hg38 option for human sample

Setup
----------------
    1. Install docker follow the official website
      https://www.docker.com/
    2. Then do the following for the workflow:
      docker pull rjunhua/stlfr_reseq:v1.4
    3. Download and unzip the database
      From BGI Cloud Drive:
        https://pan.genomics.cn/ucdisk/s/Jvmuii
      Or from OneDrive:
        ... INCOMMING ...
      After downloading, please check the file integrity through the .md5 file first via using md5sum command:
         md5sum -c db.tar.md5
	  If the md5 check is passed, then unzip it via following command:
	     tar -xf db.tar.gz
    Notes:
      1. Please make sure that you run the docker container with at least 60 GB memory and 45 CPU.
      2. The input is sample list and output directory which descripted below (Main progarm arguments).
      3. The log files and temp results will not be removed now, so the memory must be double-checked.
    
Running
----------------
    1. Please set the following variables on your machine:
      (a) $DB_LOCAL: directory on your local machine that has the database files.
      (b) $DATA_LOCAL: directory on your local machine that has the sequence data and "samplelist" file.
          "samplelist" must follow the format descripted bellow,
          and the *PATH* in "samplelist" must be absolute dicrtory of $DATA_LOCAL.
      (c) $RESULT_LOCAL: directory for result.
    2. Run the command:
        docker run -d -P --name $STLFRNAME \
        -v $DB_LOCAL:/stLFR/db -v $DATA_LOCAL:$DATA_LOCAL -v $RESULT_LOCAL:$RESULT_LOCAL \
        rjunhua/stlfr_reseq:v1.4 /stLFR/bin/stLFR -l $DATA_LOCAL/samplelist -o $RESULT_LOCAL
    3. After report is generated:
        docker rm $STLFRNAME

Usage
----------------
    1. Run the command:
        docker run -P -v $DB_LOCAL:/stLFR/db --name usage rjunhua/stlfr_reseq:v1.4 /stLFR/bin/stLFR
    2. Then, you will get the usage on screen:
        NAME
    	  stLFR - process stLFR data
  	SYNOPSIS
      	  stLFR [options]
      	  # run with default options
      	  stLFR -l SAMPLELIST
      	  # brief help
      	  stLFR -h
  	OPTIONS
	    Sample List
	      -list FILE
                  Name of input file. This is required.

                  Five columns in the list file, which the front three columns are required:
                  1. name    : unique sample ID in this analysis
                  2. path    : fastq path(s) for this sample split with colon(:)
                  3. barcode : sample-barcode for each path split with colon(:), 0 means all used
                  4. reffile : reference with index, inner options are 'hg19','hs37d5' and 'hg38', NULL or '-' means 'hs37d5'
                  5. vcffile : dbsnp file, default  is NULL or '-'
                  6. blacklist : black list file(BED format) for SV
                  7. controllist : sorted control list file(BEDPE format) for SV
	  Output Directory
	      -outputdir [ ./stLFR_out ]
                  Output directiry path.
	  stLFR barcode position
	      -position [ 201_10,217_10,233_10 ]
                  Position of stLFR barcodes on read2.
  	  Align Software
	      -analysis [ all ]
                  Workflow of analysis, default = all
                  There are 'filter', 'align', 'phase' and 'cnvsv' in this software,
                    you can set 'filter' for running filtration only,
                    or 'all', 'base' for more steps:
                  1. all    = filter + align + phase + cnvsv
                  2. base   = filter + align + phase
	      -cpu [ 45 ]
                  CPU number   
	   Usage
	      -help       Show brief usage synopsis.
	      -notrun     Don not run workflow after main shells built.
    3. Finish docker:
        docker rm usage

Demo data
----------------
    The demo data for stLFR library can be found in the link below:
        https://db.cngb.org/search/project/CNP0003896/

Input: Sample List
----------------

         Name of input file. This is required.

         Three columns are required:
         1. name    : unique sample ID in this analysis
         2. path    : fastq path(s) for this sample split with colon(:)
         3. barcode : sample-barcode for each path split with colon(:), 0 means all used

         eg:  
            SAM1   /DATA/slide1/L01     1-4,5,7-9
            SAM2   /DATA/slide1/L02:/DATA/slide2/L02     1-5:6-9
         
Result
----------------
    After all analysis processes ending, you will get these files below:
      1.  HTML report:                              *_cn.html, *_en.html
      2.  raw data summary:                         *.fastqtable.xls
      3.  stLFR barcode summary:                    *.fragtable.xls
      4.  alignment summary:                        *.aligntable.xls
      5.  variant summary:                          *.varianttable.xls
      6.  haplotype phasing summary:                *.haplotype.xls, *.haplotype.pdf
      7.  evaluation summary (NA12878):             *.evaluation.xls
      8.  quality distrubution in cleanfq:          *.Cleanfq.qual.png
      9.  base distribution in cleanfq:             *.Cleanfq.base.png
      10. depth distribution in alignment:          *.Sequencing.depth.pdf
      11. accumulated depth distribution:           *.Sequencing.depth.accumulation.pdf
      12. insert size distrubition:                 *.Insertsize.metrics.txt, *.Insertsize.pdf
      13. GC bias distrubution:                     *.GCbias.metrics.txt, *.GCbias.pdf
      14. Fragment coverage figure:                 *.frag_cov.pdf
      15. Fragment length distribution figure:      *.fraglen_distribution_min5000.pdf
      16. Fragment per barcode distribution figure: *.frag_per_barcode.pdf
      17. variant CIRCOS:                           *.circos.svg, *.circos.png, *.legend_circos.pdf

    Meanwhile, the following shows the directory structure when the process is executed:
       |-- 01.filter   // for align
       |   |__ SAMPLE
       |       |__ SAMPLE.clean_1.fq.gz
       |       |__ SAMPLE.clean_2.fq.gz
       |-- 02.align    // for phase, cnv and sv
       |   |__ SAMPLE
       |       |__ SAMPLE.sortdup.bqsr.bam
       |       |__ SAMPLE.sortdup.bqsr.bam.bai
       |       |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz
       |       |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz.tbi
       |-- 03.phase    // for cnv
       |   |__ SAMPLE
       |       |__ phasesplit
       |           |__ hapblock_SAMPLE_CHROMOSOM
       |-- 04.cnv
       |   |__ SAMPLE
       |       |__ SAMPLE.CNV.result.xls
       |-- 05.sv
       |   |__ SAMPLE
       |       |__ SAMPLE.SVresult.xls
       |__ file
           |__ SAMPLE
               |-- alignment
               |   |__ SAMPLE.sortdup.bqsr.bam
               |   |__ SAMPLE.sor
               |-- CNV
               |   |__ SAMPLE.CNV.result.xls
               |-- haplotype
               |   |__ SAMPLE.hapblock
               |-- sequence
               |   |__ SAMPLE.clean_1.fq.gz
               |   |__ SAMPLE.clean_2.fq.gz
               |-- SV
               |   |__ SAMPLE.SV.result.xls
               |__ variant
                   |__ SAMPLE.sortdup.bqsr.bam.HaplotypeCaller.vcf.gz

Additional Information
----------------
    If you like to use clean FASTQ files to run other pipelines or software, you need to set -analysis parameter of docker running to filtefilter:	
    docker run -d -P \
        --name $STLFRNAME \
        -v $DB_LOCAL:/stLFR/db \
        -v $DATA_LOCAL:$DATA_LOCAL \
        -v $RESULT_LOCAL:$RESULT_LOCAL \
        rjunhua/stlfr_reseq:v1.4 \
        /stLFR/bin/stLFR \
        -l $DATA_LOCAL/samplelist \
        -o $RESULT_LOCAL -analysis filter
        
    Then you will get the FASTQ files from subdirectory(named after sample name) in the 01.filter/ directory of result, such as:
       |-- 01.filter/ 
           |__ SAMPLE/
               |__ SAMPLE.clean_1.fq.gz
               |__ SAMPLE.clean_2.fq.gz
           
License
----------------
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions： 
  
The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
  
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
