# Data Processing Steps :
The following are the steps used to process the data in the paper and obtain the heatmaps therein.
## 1) Mapping 5C Fastqfiles:
		For 5C mapping, one needs to use novoalign instead of bowtie2. 
    On UMass cluster you can do this via 
    `load module load novocraft/V3.02.08`
		Then run this script for mapping:
   	`perl ~/cMapping/scripts/utilities/processFlowCell.pl -i /farline/umw_job_dekker/HPCC/tmp_cshare/solexa/19MAY17_C-Monster-A_HB/ -s /home/hb67w/farline/scratch  -o /farline/umw_job_dekker/HPCC/tmp_cshare/cData/ --gdir /farline/umw_job_dekker/HPCC/tmp_cshare/genome/ -f -g 6217-KelliherMyc`
		After you run the mapping and script request novocraft/3.02.00/novoalign for the aligner Path here is exact path
		alignerPath []: /home/bl73w/cMapping/alpha/aligners/novocraft/3.02.00/novoalign
