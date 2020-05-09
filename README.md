# DNAmethylation
Script containing every step to download and install required software (from PacBio GitHub pages and biocontainers) to analyse DNA methylome 
#https://github.com/PacificBiosciences/Bioinformatics-Training/wiki/Base-Modification-Tools


############################ bax2bam ##########################

#Convert all bax.h5 to one bam file (only PacBio RS) https://github.com/PacificBiosciences/blasr/wiki/bax2bam-wiki:-installation,-basic-usage-and-FAQ
cd /pwd/ #pathway where the files (.bax.h5) source

# 3 by 3 to generate bam file meaning one for reverse and one for forward (~ 5min)
bax2bam file1.bax.h5 file2.bax.h5 file3.bax.h5 -o movie --subread --pulsefeatures=DeletionQV,DeletionTag,InsertionQV,IPD,MergeQV,SubstitutionQV,PulseWidth,SubstitutionTag



############################ pbcoretool ##########################

# to work PacBio files http://pacificbiosciences.github.io/pbcoretools/pbcoretools.html, here I use it to merge two movie.subreads.bam files (one by each SMRT cell)

######## Installation
# launch docker
docker pull quay.io/biocontainers/pbcoretools:0.2.4--py27_4
docker images # giving the id to create the new path
mkdir pbcoretool
alias pbcoretool='docker -it -v /pwd/pbcoretool:/data bd15b30ffe16 bash' #replace pwd by your pathway

# Put the files (subreads.bam & subreads.bam.pib ) to work with in the blasr folder 

######## Running
source ~/.profile
pbcoretool
cd /data  # localisation des dockers

dataset create --type SubreadSet test2cell.subreadset.xml \ movie1.subreads.bam movie2.subreads.bam #each subreads are from 2 SMRT cells contening 3 movies each



############################ Blasr ########################## 

#to align Bam files to reference genome (one SMRT Cell or multiple SMRT cells by merging them before with pbcoretool)
######## Installation
docker pull muccg/blasr
docker images
mkdir blasr
alias blasr='docker -it -v /pwd/blasr:/data ce131ca5b920 bash' #replace pwd by your pathway

# Put the files (fasta & subreadsset.xml with each subreads.bam and subreads.bam.pib) to work with in the blasr folder

######## Running
source ~/.profile
blasr
cd /data  # necessary for some containers


#blasr movie1.subreads.bam HSJ1_workedv2.fasta --out HSJ1_2.aligned_subreads.bam --bam #one by one

blasr test2cell.subreadset.xml ref.fasta --bam --out alignments.bam  # using the merged or created file from pbcoretool



############################ samtools ##########################  

#https://github.com/samtools/samtools
# To sort alignment file and to create index for fasta and alignment file

######## Installation caution requires htslib and bcftools
#htslib
git clone git://github.com/samtools/htslib.git
cd htslib
autoreconf
./configure
make
make Install
make clean

#bcftools
git clone git://github.com/samtools/bcftools.git
cd
cd htslib
autoreconf
./configure
make
sudo make Install
make clean

export BCFTOOLS_PLUGINS=/pwd/bcftools/plugins #allows us to use the tools #replace pwd by your pathway

#samtools https://github.com/samtools/samtools.git
cd
cd samtools
autoreconf
./configure
make
sudo make Install
make clean

######## Running
cd samtools
# create a pbi file for the aligned data
samtools sort -o alignments.sorted.bam alignments.bam
samtools index  alignments.sorted.bam
pbindex alignments.sorted.bam

# create a fai file for fasta
samtools faidx ref.fasta

############################ kinetictools ########################## 

# Identification of DNA modifications only for PacBio RS, for PacBio Sequel they recommended using SMRT link
#https://github.com/PacificBiosciences/kineticsTools
######## Installation
docker pull 
mkdir kinetictools
docker images
alias samtools='docker -it -v /Users/pauline/samtools:/data 50ba2fd75f03 bash'


######## Running
source ~/.profile
kinetictools
ipdSummary alignments.sorted.bam --reference ref.fasta --identify m6A,m4C --methylFraction --mapQvThreshold 30 --numWorkers 16 --gff basemod.gff  --csv kinetics.csv # required pbi and fai files too in the folder



############################ MotifMaker-master  ##########################

#https://github.com/PacificBiosciences/MotifMaker
cd /Users/pauline/MotifMaker-master
java -Xms8192m -Xmx8192m -jar MotifMaker-assembly-0.3.1.jar find -f /pwd/MotifMaker-master/ref.fasta  -g /pwd/MotifMaker-master/basemod.gff  -o HSJ1testbasemod_motifsQ40.csv -m 40 #replace pwd by your pathway
