# Data Processing Steps :

The following are the steps used to process the data in the paper and obtain the heatmaps therein.

## 1) Mapping 5C Fastqfiles:

For 5C mapping, one needs to use novoalign instead of bowtie2. 
On UMass cluster this can be done as follows: 

`load module novocraft/V3.02.08`

Then run this script for mapping:

`perl ~/cMapping/scripts/utilities/processFlowCell.pl -i /farline/umw_job_dekker/HPCC/tmp_cshare/solexa/19MAY17_C-Monster-A_HB/ -s /home/hb67w/farline/scratch  -o /farline/umw_job_dekker/HPCC/tmp_cshare/cData/ --gdir /farline/umw_job_dekker/HPCC/tmp_cshare/genome/ -f -g 6217-KelliherMyc`

## 2)Combine5C:

After mapping we use read-pairs

`perl ~/cMapping/scripts/utilities/combine5C.pl -i /farline/umw_job_dekker/HPCC/tmp_cshare/cData/19MAY17_C-Monster-A_HB/ -s /home/hb67w/farline/scratch -o /farline/umw_job_dekker/HPCC/tmp_cshare/fiveCData/ --gdir /farline/umw_job_dekker/HPCC/cshare/genome/ -g 6217-KelliherMyc --short`

## 4) Column2matrix:

After combine we get the file that contain 3 columns "Lucio5C-D1-HB-20-30-Combined__6217-KelliherMyc.gz" locusI" "locusJ" "interactions score"
   	
`perl ~/cworld-dekker/scripts/perl/column2matrix.pl -i /farline/umw_job_dekker/HPCC/tmp_cshare/fiveCData/Lucio5C-D1-HB-20-30-Combined__6217-KelliherMyc.gz --oxh ReversePrimers.txt --oyh ForwardPrimers.txt -v`
   	
In this particular case, we provided our forward and reverse primers based on the number of the probe. So we will have the right sorted heatmap. If you don't provide the sorted probes you get the 4 heatmaps in 1 ( f-LR)(F-R)(LF-LR)(LF-R)
	
## 5)Diagonal removal: 

In order to remove LR & F that belong to the same fragment coming from self-circles, we run the subsetMatrix.pl script. 

`perl ~/cworld-dekker/scripts/perl/subsetMatrix.pl -i Lucio5C-A1-HB-20-30-Combined__6217-KelliherMyc.binary.matrix.gz  --minDist 1 -v`

 Note that the input provided in the -i arguments is the output matrix we got from column2matrix in the previous step.
	
## 6)Singleton removal:

This is done by the script SingletonRemoval.pl.

`perl ~/cworld-dekker/scripts/perl/singletonRemoval.pl -i ~/5C/Lucio-Myc/subsetMatrix_DiagonalRemoval/Lucio5C-A1-HB-20-30-Combined__6217-KelliherMyc.score.subset.matrix.gz --ic --ca 0.02 --caf 2500 --cta 20 --ez  -v`

## 7)Singleton removal Using combine toRemove.txt of all libraries we need to compare:

In the previous step, we remove the singleton from each library independently. But afterwards, we need to make sure that the combined toRemove.txt of all libraries we want to compare is used for singletons removal.

## 8) Anchor removal: anchorPurge.pl This script detects and removes outlier row/cols.
	
`perl ~/cworld-dekker/scripts/perl/anchorPurge.pl -i ~/5C/Lucio-Myc/singletonRemoval/CombineSingletonRemoval_4libraries/Lucio5C-D2-HB-20-30-Combined__6217-KelliherMyc.score.subset.singletonFiltered.matrix.gz --ic --ca 0.02 --caf 2500 --ez  --lof ~/5C/Lucio-Myc/singletonRemoval/Lucio5C-D2-HB-20-30-Combined__6217-KelliherMyc.score.subset_cta20--ic--it--ez--caf2500--ca0.02.loess.object.gz --im 1.5 -v` 

## 9) Combine Anchor removal: 

From the 4 libraries, we are comparing in the previous step, we remove the anchor from each library  independently, but afterwards, we need to make sure that the combined AnchorRemoval.txt of all libraries we want to compare is used for Anchor removal.
	
## 10) Making the Matrix Symmetrical Matrix2symmetrical: 

In order to run hdf2tab, we need a symmetrical matrix 

`perl ~/cworld-dekker/scripts/perl/matrix2symmetrical.pl -i ~/5C/Lucio-Myc/subsetMatrix_DiagonalRemova/Lucio5C-D1-HB-20-30-Combined__6217-KelliherMyc.score.subset.matrix.gz -v`

## 11) Scaling the Matrix: 

Since we are comparing four libraries, we scale the four libraries to 2600000 reads.

`perl ~/cworld-dekker/scripts/perl/scaleMatrix.pl -i Lucio5C-A2-HB-20-30-Combined__6217-KelliherMyc.score.subset.singletonFiltered.anchorFiltered.symmetrical.matrix.gz --st 52000000 -v`

## 11) Converting tab separated files to hdf5 format: 

In order to balance our matrices, we need the data in hdf5 format. The conversion is done as follows.

`python ~/tab2hdf/scripts/tab2hdf.py -i Lucio5C-A1-HB-20-30-Combined__6217-KelliherMyc.score.subset.symmetrical.selfMerged.matrix.gz -v`
		
## 12) Balancing the matrix:

`python ~/balance/scripts/balance.py -i Lucio5C-A1-HB-20-30-Combined__6217-KelliherMyc.score.subset.symmetrical.selfMerged.hdf5 -v`

## 13) Converting hdf5 files back to tab separated files: 

Note that the binning script works on .matrix files.

`python ~/hdf2tab/scripts/hdf2tab.py -i Lucio5C-A1-HB-20-30-Combined__6217-KelliherMyc.score.subset.symmetrical.selfMerged.balanced.hdf5 -v`

## 14) Binning the data:

`perl ~/cworld-dekker/scripts/perl/binMatrix.pl -i Lucio5C-D1-HB-20-30-Combined__6217-KelliherMyc.score.subset.symmetrical.matrix.gz --bsize 20000 --bstep 8 --bmode median -v` 

## 14) Making a heatmap:

`perl ~/cworld-dekker/scripts/perl/heatmap.pl -i Lucio5C-D1-HB-20-30-Combined__6217-KelliherMyc.score.subset.symmetrical--bsize20000--bstep8--bmodemedian.matrix.gz -v`
