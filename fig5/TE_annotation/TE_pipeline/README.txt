~~~ Virginia's TE Pipeline ~~~
A very clunky TE annotation pipeline designed for (and only tested on) Brachypodium, but is 
appropriate for any grasses. Is appropriate for any plant if you get the right public TE database(s).

When I started making this, EDTA (https://github.com/oushujun/EDTA) was not available yet,
or rather it was but it was still buggy. But now EDTA is available. EDTA is similar to my 
pipeline, but is honestly strictly better.

The premise of this pipeline is as follows: First, monocot TEs are pulled from the RepeatMasker database, 
and these are concatenated to the TREP database to create an initial TE library. To expand this library 
to include TEs from our new Brachypodium genomes (or whatever genome you'd like), we run a suite of TE 
discovery tools: LTR-Harvest and LTR_retriever, TransposonPSI, MITE-Tracker, and RepeatModeler2. 
These de novo TEs are added to the library and redundancy is removed with CD-HIT according to the 80-80-80 rule. 
Sequences are clustered if they have 80% identity locally, and the alignment had to cover at least 80% of 
the shorter sequence. Only the longest sequence (the representative sequence) from each cluster is retained. 
Representative sequences less than 80bp are discarded. Next, ProtExcluder from the MAKER-P pipeline is 
used to search the TE library against a plant protein database, and TEs with significant hits to genes are 
removed. The result of this process is a single non-redundant 'master library' containing TE exemplars from 
a variety of monocots plus whatever genome(s) you've annotated. You can then annotate an assembly with
RepeatMasker using this new, custom library.

Once we have a TE library, we use RepeatMasker to annotate the genome with its own library.
Then we use 'one code to find them all' to assemble the fragments identified by RepeatMasker
into TE copies. This helps us resolve nested elements. It's not as reliable as manual curation, 
but it's better than nothing.

The most useful output file is called allfragments.classified and looks like this:
fragcode        chrom   frag_start      frag_end        LTRfeature      TEcopycode      TEcopy_start    TEcopy_end      strand  exemplar        isFL    classification
frag0000001     Bd1     43      298     NA      onecode000001   43      298     -       Oswald.trep00437#DNA/CACTA      False   DNA_transposon_CACTA
frag0000002     Bd1     374     551     NA      onecode000002   374     551     +       rnd-1_family-497#LTR/Copia      False   LTR_retrotransposon_RLC
frag0000003     Bd1     15562   15731   NA      onecode000003   15562   15731   +       rnd-4_family-1468#LTR/Copia     False   LTR_retrotransposon_RLC
frag0000004     Bd1     16512   16742   NA      onecode000004   16512   16742   -       rnd-4_family-1524#LTR/Copia     False   LTR_retrotransposon_RLC
frag0000005     Bd1     22190   22265   NA      onecode000005   22190   22265   -       Copia.chain04912#LTR/Copia      False   LTR_retrotransposon_RLC
frag0000006     Bd1     24749   24846   NA      onecode000006   24749   24846   +       mite740#DNA/Unknown     False   DNA_transposon_MITE-Unclassified
frag0000007     Bd1     26344   28089   NA      onecode000007   26344   28089   -       LINE.chain00001#LINE/LINE       False   non_LTR_retrotransposon
frag0000008     Bd1     31669   31749   NA      onecode000008   31669   31749   -       BdisEhu.trep00750#DNA/Mutator   False   DNA_transposon_Mutator
frag0000009     Bd1     36030   40512   NA      onecode000009   36030   40512   -       Gypsy.chain05930#LTR/Gypsy      False   LTR_retrotransposon_RLG
frag0000010     Bd1     45460   47237   NA      onecode000010   45460   47237   -       Gypsy.chain05930#LTR/Gypsy      False   LTR_retrotransposon_RLG
frag0000011     Bd1     61294   61494   LTR     onecode000011   61294   63257   +       LTRharvest00112#LTR/unknown     True    LTR_retrotransposon_RLX
frag0000012     Bd1     61495   61644   int     onecode000011   61294   63257   +       LTRharvest00112#LTR/unknown     True    LTR_retrotransposon_RLX
frag0000013     Bd1     61894   62326   int     onecode000011   61294   63257   +       LTRharvest00112#LTR/unknown     True    LTR_retrotransposon_RLX
frag0000014     Bd1     62371   63054   int     onecode000011   61294   63257   +       LTRharvest00112#LTR/unknown     True    LTR_retrotransposon_RLX
frag0000015     Bd1     63055   63257   LTR     onecode000011   61294   63257   +       LTRharvest00112#LTR/unknown     True    LTR_retrotransposon_RLX
frag0000016     Bd1     61320   61494   NA      onecode000012   61320   63054   -       Dahlia.trep01088#LTR    False   LTR_retrotransposon_RLX
frag0000017     Bd1     61645   61661   NA      onecode000012   61320   63054   -       Dahlia.trep01088#LTR    False   LTR_retrotransposon_RLX
frag0000018     Bd1     62607   63054   NA      onecode000012   61320   63054   -       Dahlia.trep01088#LTR    False   LTR_retrotransposon_RLX
frag0000019     Bd1     64091   69434   NA      onecode000013   64091   69434   -       BdisC053.trep00593#LTR/Copia    False   LTR_retrotransposon_RLC
frag0000020     Bd1     70159   70245   NA      onecode000014   70159   70245   +       BdisA.trep00677#DNA/Harbinger   False   DNA_transposon_PIF-Harbinger

You'll also get a file called TEstats.txt with some basic metrics about the annotation.

This pipeline relies on several conda environments and it was designed to be run on NERSC's
cori cluster using the SLURM scheduler. Some of the steps are wrapped with snakemake. 
The pipeline involves several steps that each need to be run manually by the user. 
Basically, it was never really designed to be a pipeline, much less a piece of software for 
general use. If you want to use it, you'll need to tweak stuff. Like, a lot. Like, it
might be more efficient to just use this as "inspiration" but to basically build your own pipeline.

The pipeline comes as several folders, most of which contain scripts. The user must keep 
the directory structure as follows (the name of the top-level directory and the genome can 
be anything):

top_dir/:
myGenome.fasta      
timeForTE.py
step1
step2
step3
step4
step5
step6

step1/:
step1a
step1b
step1c
step1d

SUPER DUPER IMPORTANT:
timeForTE.py contains various functions and variables used by other python scripts. 
The user MUST define the variables at the top of this script before running the pipeline! (Do once per genome)
The user MUST also put this script in their python path! (Do every time)

Start EVERY session with the following commands to set up your environment:
module load python/3.6-anaconda-5.2
export PYTHONPATH="${PYTHONPATH}:/global/projectb/scratch/vstartag/TEannotation/top_dir"
^Replace with the path to your own working directory.

Unless I say otherwise, always work in the TEdefault environment:
source activate /global/projectb/sandbox/plant/hybridum/software/TEdefault

For step 1d you must also do:
export PATH=$PATH:/global/projectb/sandbox/plant/hybridum/software
export PATH=$PATH:/global/projectb/sandbox/plant/hybridum/software/nseg


A less-important note:
If you already have a TE library, you can still run the latter half of this pipeline to get an annotation.
Just edit timeForTE.py as usual, put the genome you want to annotate in the top directory,
and put the library you want to use in step4/. Then start the pipeline from step 4 with a 
modified Snakefile. Make the following modifications:
-Comment out or delete the entire rule finish_step_3
-Make the input for rule setup the TE library you have. (in single quotes) For example, change:
'TElib.{}.fa'.format(timeForTE.genome)
to:
'TElib.Bhyb26.fa'
-Change the last line of rule make_array_script to reflect your TE library name.
So this:
file2.write("/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/RepeatMasker -lib TElib.{}.fa -gff -pa 31 -nolow -dir . ${{SLURM_ARRAY_TASK_ID}}.fa\n".format(timeForTE.genome))
Becomes this:
file2.write("/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/RepeatMasker -lib TElib.Bhyb26.fa -gff -pa 31 -nolow -dir . ${SLURM_ARRAY_TASK_ID}.fa\n")
Note the single curly brackets. Make sure the final script runRepeatmasker.sh has only one set of curly brackets around SLURM_ARRAY_TASK_ID.
~~~




~~~ TABLE OF CONTENTS ~~~
Installation and setup
Step 0: Make a database of publicly available TEs (only needs to be done once)
Step 1: Get ab initio TEs:
Step 1a: LTRRetriever (takes 1 to several hours)
Step 1b: TransposonPSI (takes roughly 10 minutes to half an hour)
Step 1c: MITE-Tracker (takes several hours)
Step 1d: RepeatModeler (takes several days or more)
Step 2: Combine public and de novo TE databases and remove redundancy
Step 3: Remove non-TE gene fragments (takes ~3-8 hours)
Step 4: Run RepeatMasker with final TE database
Step 5: Assemble TE fragments into TE copies with 'one code to find them all'
Step 6: Barcode TE fragments and copies for downstream analysis
~~~~




############################################## Installation and setup ##############################################

First, create the main conda environment we will use throughout this pipeline, which I call TEdefault.
As always, make sure you're using python/3.6-anaconda-5.2. 
Next, do this in the order shown:
conda create --prefix /global/projectb/sandbox/plant/hybridum/software/TEdefault -c bioconda genometools-genometools=1.5.10 snakemake=3.13.3 bedtools=2.29.2 hmmer=3.3
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/TEdefault -c conda-forge -c bioconda bzip2=1.0.8 perl-text-soundex=3.05
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/TEdefault -c BioBuilds muscle=3.8.31 perl=5.26.2 perl-bioperl=1.7.2

Create some more conda environments, since conda won't install blast in a way that is compatible with RepeatMasker. 
(It screws up our perl libraries.)
conda create --prefix /global/projectb/sandbox/plant/hybridum/software/step4 blast=2.5.0 snakemake=3.13.3
conda create --prefix /global/projectb/sandbox/plant/hybridum/software/step7 -c bioconda blast=2.9.0 snakemake=3.13.3 bedtools=2.29.2 emboss=6.6.0 muscle=3.8.31 hmmer=3.3
Later on, I did: conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step7 biopython
(This installed biopython-1.77 build py36h7b6447c_0)
Ack! Also later did: 
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step7 -c bioconda mafft
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step7 -c bioconda trimal
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step7 -c bioconda modeltest-ng=0.1.6 raxml-ng
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step7 more-itertools

Yet another conda env for MITE-Tracker (Do install packages separately, in this order):
conda create --prefix /global/projectb/sandbox/plant/hybridum/software/step1c biopython
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step1c blast
conda install --prefix /global/projectb/sandbox/plant/hybridum/software/step1c pandas

Next, install Repeatmasker. Be in the TEdefault env for this.
Install the Dfam libraries as described in the installation protocol.
Install TRF within the top-level RepeatMasker/ dir.
Install rmblast from source code. Do the configuration like so:
./configure --with-mt --without-debug --without-krb5 --without-openssl --with-projects=scripts/projects/rmblastn/project.lst --prefix=/global/projectb/sandbox/plant/hybridum/software/rmblast
Follow the directions to compile, which is just one command but it takes many hours.

Finally, I configured RepeatMasker like so, still while in the TEdefault env:
cd /global/projectb/sandbox/plant/hybridum/software/RepeatMasker
perl ./configure

I told it to use this version of perl:
/global/projectb/sandbox/plant/hybridum/software/TEdefault/bin/perl
This version of RepeatMasker:
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker
This version of trf:
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/trf
And this version of rmblast:
/global/projectb/sandbox/plant/hybridum/software/ncbi-blast-2.9.0+-src/c++/ReleaseMT/bin

Next we need to install RepeatModeler, starting with its dependencies:
First I installed RepeatScout. Compiled from source while in the TEdefault env. 
Just did command 'make' from the top level dir (contains Makefile).
Next, RECON. Unpack the tarball and go into the src/ folder, then run "make" and then "make install". 
Next, in the script "recon.pl" in the scripts directory, on the third line, add the path 
to the binaries (the bin directory here) between the double quotes. 
So the top of the script looks like:
#! /usr/bin/perl

$path = "/global/projectb/sandbox/plant/hybridum/software/RECON-1.08/bin";
Very frustrating that the authors did not mention this!!!
 
Next I installed nseg. This installation is a little janky but it works. 
mkdir nseg in your software directory and cd into it.
Then do the following:
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/genwin.c
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/genwin.h
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/lnfac.h
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/makefile
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/nmerge.c
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/nseg.c
wget ftp://ftp.ncbi.nih.gov/pub/seg/nseg/runnseg
Now do 'make'.
They will all raise warnings, but as long as you don't get errors you should be fine.
Next I downloaded the RepeatModeler tarball (v.2.0.1) from http://www.repeatmasker.org/RepeatModeler/
Finally, I also downloaded Ninja from https://github.com/TravisWheelerLab/NINJA/releases/tag/0.95-cluster_only
and unpacked the tarball with tar -xzf NINJA-0.95-cluster_only.tar.gz. Then I did:
cd NINJA-0.95-cluster_only/NINJA
make all
Now configure RepeatModeler as you did with RepeatMasker, using perl ./configure
I told RepeatModeler to use this version of perl:
/global/projectb/sandbox/plant/hybridum/software/TEdefault/bin/perl
This version of RepeatMasker:
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker
This version of RECON:
/global/projectb/sandbox/plant/hybridum/software/RECON-1.08/bin
This version of RepeatScout:
/global/projectb/sandbox/plant/hybridum/software/RepeatScout-1
This version of trf:
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/trf
This version of rmblast:
/global/projectb/sandbox/plant/hybridum/software/ncbi-blast-2.9.0+-src/c++/ReleaseMT/bin
This version of GenomeTools:
/global/projectb/sandbox/plant/hybridum/software/TEdefault/bin
This version of LTR_retriever:
/global/projectb/sandbox/plant/hybridum/software/LTR_retriever
This version of MAFFT:
/global/projectb/sandbox/plant/hybridum/software/step7/bin
This version of ninja:
/global/projectb/sandbox/plant/hybridum/software/NINJA-0.95-cluster_only/NINJA
This version of CD-HIT:
/global/projectb/sandbox/plant/hybridum/software/cd-hit-v4.8.1-2019-0228
This version of nseg: (DEFUNCT!! We do need nseg in our path, but RepeatModeler2 does not configure it for you)
/global/projectb/sandbox/plant/hybridum/software/nseg

This is a good time to install TransposonPSI.
It's extremely easy, just download the script from sourceforge. Done.

MITE-Tracker installation next!
git clone https://github.com/INTABiotechMJ/MITE-Tracker.git
All of the dependencies are in your conda environment /global/projectb/sandbox/plant/hybridum/software/step1c
(see above), except vsearch. MITE-Tracker seems to fail when I install vsearch using conda.
MITE-Tracker requires vsearch v. 2.7.1.
So I am installing vsearch like so:
wget https://github.com/torognes/vsearch/releases/download/v2.7.1/vsearch-2.7.1-linux-x86_64.tar.gz
tar xzf vsearch-2.7.1-linux-x86_64.tar.gz
mv vsearch-2.7.1-linux-x86_64 vsearch-2.7.1

Here's how I installed LTR_retriever:
git clone https://github.com/oushujun/LTR_retriever.git   #Installed in /global/projectb/sandbox/plant/hybridum/software
I also installed CD-Hit separately.
Then go into the LTR_retriever dir and edit the file called 'paths', and add the paths to the dirs 
where you installed CD-Hit, and RepeatMasker, including the slash at the end:
e.g. /global/projectb/sandbox/plant/hybridum/software/cd-hit-v4.8.1-2019-0228/
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/

One last thing: I am installing the ProtExcluder software and plant protein database:
http://weatherby.genetics.utah.edu/MAKER/wiki/index.php/Repeat_Library_Construction-Advanced
I unpacked the tarball and cd'd into it, then installed like so:
export PEPATH=/global/projectb/sandbox/plant/hybridum/software/ProtExcluder1.2/
export HMMPATH=/global/projectb/sandbox/plant/hybridum/software/TEdefault/bin/
^The / at the end of the path is important, I think
Then I did:
./Installer.pl -m $HMMPATH -p $PEPATH
Next, edit /global/projectb/sandbox/plant/hybridum/software/ProtExcluder1.2/mspesl-sfetch.pl
line 17. Change it from e.g. this:
 `/global/projectb/sandbox/plant/hybridum/software/timeForTE/bin/binaries/esl-sfetch --index $ARGV[0]`;
to this:
 `/global/projectb/sandbox/plant/hybridum/software/timeForTE/bin/esl-sfetch --index $ARGV[0]`;
 Next, do the same with the two other lines where that path appears in mspesl-sfetch.pl.
############################################################################################################



















########################## Step 0: Prepare your library of known repeats from related organisms ##########################
This step only needs to be done once ever, as long as you are working with monocots.
Remember that additional info on the TREP exemplars is in step0/TREP.names!

Use conda environment /global/projectb/sandbox/plant/hybridum/software/step7 for this step.

Here I created a library of monocot TEs by combining TREP (The Triticiae Repeat database) with 
Repbase. It looks like Repbase does not contain the entire TREP database (although it does 
contain some TREP TEs for sure), so I will combine them and then eliminate redundancy using CD-HIT.

Create your main work directory (e.g. Bd21), and put my script timeForTE.py into it.
Also put your unmasked genome into it. Configure timeForTE.py by editing the first few
lines, putting in new paths as appropriate.
Make a directory called step0/ and cd into it. step0/ should contain the following files:
removeNullFromTREP.py
parse_blast_aln.py 
prepare_TREP_for_blast.py
reformatTREPheadersNEW.py
removeSimpleReps.py
trepblast.sh
trep-db_nr_Rel-16.fasta #This is the (non-redundant) TREP database, and can be found online easily.
(Snakefile, optional)

TO DO THIS STEP WITH SNAKEMAKE: just enter the command 'snakemake'. That's it. Let it finish, then proceed to step 1.

TO DO THIS STEP WITHOUT SNAKEMAKE:
First, we'll fetch a relevant library from the RepeatMasker database:
/global/projectb/sandbox/plant/hybridum/software/RepeatMasker/util/queryRepeatDatabase.pl -species monocotyledons > monocotyledons.repeatmasker.lib

Next, TREP contains some very annoying "TEs" whose "sequence" is just the word NULL.
I wrote a super quick script to remove those since they are obviously useless.
python removeNullFromTREP.py

We want to concatenate these two libraries. However, the TREP headers are problematic.
They are nice and long and informative, but they contain spaces and weird characters.
So I wrote a script called reformatTREPheadersNEW.py to parse these.

It produces two files: 
(1) TREP.RMcompatible.preliminary.fasta: A fasta file that is the original TREP library,
except the headers are now unique and in the same format as RepeatMasker headers.
(2) TREP.names: A two column-file: the new names are in the first column and original names are in the second.

Let's do one more processing step: get internal LTR boundaries for TREP exemplars,
and use those to put TREP LTR-RTs in our database as two sequences: an LTR and an internal sequence.
This aids in the identification of solo-LTRs, and it's also what our curation program (onecode) expects.
Do:
python3 prepare_TREP_for_blast.py
./trepblast.sh
python3 -B parse_blast_aln.py

Now we can concatenate the TREP and RM libraries like so:
cat monocotyledons.repeatmasker.lib TREP.RMcompatible.fasta > TREP.RM.combined

The monocot library from RepeatMasker has a lot of simple repeats I'd like to remove.
Let's do that quickly:
python removeSimpleReps.py

Finally, let's remove redundancy, since as I said it looks like some TREP TEs are in RepeatMasker.
Let's say that exemplars sharing 80% identity over 80% of their length are redundant. I'm choosing this  
because our family assignments will be based on exemplars. At the end of this pipeline, 
I'll define a family as hits to one given exemplar. (It's just easier that way!)
In Flutre et al. 2011, they argue for higher stringency for exemplars than for TE copies. 
("Considering Transposable Element Diversification in De Novo Annotation Approaches")
However, since I have so many similar TEs from many different monocots, I think a lower stringency
makes intuituve sense, in addition to the family definition mentioned above.

Since CD-HIT is a greedy algorithm, sort elements by length, descending, like so:
python sortFasta.py TREP.RM.combined
Next, run CD-HIT like so:
/global/projectb/sandbox/plant/hybridum/software/cd-hit-v4.8.1-2019-0228/cd-hit-est -i TREP.RM.combined.sorted.fasta -o TREP.RM.combined.almost -c 0.8 -G 0 -aS 0.8 -n 5 -T 0 -d 0 -M 0 > CD-HIT.stdout
(-c: Need 80% identity)
(-G: Need 80% identity locally, not globally)
(-aS: the alignment must cover 80% of the shorter sequence)
(-n, -T, -d, -M: word size, threads, header length in .clstr file, memory usage)

TREP.RM.combined.final contains the exemplars!
Quickly polish it off with 50bp-paragraphs:  
python reformatFastaParagraphs.py TREP.RM.combined.final TREP.RM.combined.final 50
##########################################################################################################################################











#################################### Step 1: Feature-based identification of novel TEs ###################################
In step1, we will use feature-based TE identification programs to identify novel TEs based on 
characteristic features of that type of TE. The substeps of step1 can be run in any order or at the same time.
Store your unmasked genome file and the script timeForTE.py in the top-level directory.
Make sure you have entered the correct paths at the top of the timeForTE.py script. 
In your top-level directory, you should have a dir called step1/. In it, you should have four subdirectories 
called step1a/, step1b/, step1c/, and step1d/.
###########################################################################################################################











#################################### Step 1a: Identify new LTR-RTs with LTRHarvest ####################################
To start out, do source activate /global/projectb/sandbox/plant/hybridum/software/TEdefault

This step searches for LTR-RTs, where LTRs are >= 85% similar and range in size from 100 bp to 20000 bp.
(Parameters are copied from Wicker et al. 2018 "Impact of transposable elements on genome structure and evolution in bread wheat")

Your step1a dir should contain a Snakefile and reformatLTR_retriever.py.
Enter the command "snakemake" while in step1a/. (Takes a few minutes.)
Next, do conda deactivate and SWITCH to /global/projectb/sandbox/plant/hybridum/software/step7
Then, submit the batch script runLTRretriever.sh with sbatch.

Almost there! Once that's finished, run my script reformatLTR_retriever.py like so (substituting your own output file names):
python3 reformatLTR_retriever.py Bhybridum_463_v1.0.unmasked.fa.LTRlib.fa Bhybridum_463_v1.0.unmasked.fa.pass.list
(Do NOT use the 'nmtf' files.)

Double-check that LTRharvest.exemplars.RMready looks okay.
############################################################################################################################












############################################## Step 1b: Get older transposons with TransposonPSI ##############################################
Work in my TEdefault env.

step1b/ should contain the following files:
reformatTransposonPSI.py
Snakefile
envs.yaml

Do: 
snakemake --use-conda
Takes maybe 10 or 20 minutes. That completes step1b.

################################################################################################################################################













############################################## Step 1c: Get MITEs with MITE-Tracker ##############################################
NO SNAKEMAKE OPTION FOR THIS STEP.
Do:
source activate /global/projectb/sandbox/plant/hybridum/software/step1c

We will now run MITE-Tracker to identify MITEs in the genome. 
Unfortunately, this program only seems to run properly if you
submit your batch script from the directory where the software is located.
So cd to that directory e.g. /global/projectb/sandbox/plant/hybridum/software/MITE-Tracker.
Make sure that before you run MITE-Tracker, there is a results/ directory in this 
directory, e.g. /global/projectb/sandbox/plant/hybridum/software/MITE-Tracker/results/.
If this is not your first time running MITE-Tracker, then results/ should already exist.
Now put your batch script in the .../software/MITE-Tracker/ directory, something like this:

#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=59G
#SBATCH -t 5:00:00
#SBATCH -q genepool_shared
#SBATCH -A plntanot
#SBATCH -C haswell
#SBATCH --mail-user=vstartaglio@lbl.gov
#SBATCH --mail-type=end
python3 -m MITETracker -g /global/projectb/scratch/vstartag/TEannotation/Bd21/Bdistachyon_314_v3.0.unmasked.fa -w 32 -j Bd21


Step 2 will expect your results to be in {softwaredir}/MITE-Tracker/results/{genome}/families_nr.fasta
as defined in your timeForTE module, so you'll need to copy that file back to that location
once MITE-Tracker is done running, e.g.
cp /global/projectb/sandbox/plant/hybridum/software/MITE-Tracker/results/Bd21/families_nr.fasta step1/step1c
####################################################################################################################################












############################################## Step 1d: Identify more TEs de-novo with RepeatModeler ##############################################
VERY IMPORTANT: Before running RepeatModeler, do:
export PATH=$PATH:/global/projectb/sandbox/plant/hybridum/software
export PATH=$PATH:/global/projectb/sandbox/plant/hybridum/software/nseg
(I'm doing both to be paranoid.)
And use my TEdefault env.

Now we run RepeatModeler to get more TEs of various kinds. 
We are using snakemake to create a database and a batch script.
So to start, just do:
snakemake
Once that's done, look over the script and if it looks good, submit it:
sbatch runRepMod.sh.

RepeatModeler is very fickle. Here are some tips:
-RepeatModeler uses a ton of memory and creates a ton of temporary files, so make sure
you have enough space and enough ram. 
-I find that if I try to run two RepeatModeler jobs at the same time, even if they 
are on different genomes and in different directories, both jobs will probably fail. 
So I always run one job at a time only.
-Runtimes for RepeatModeler can vary wildly. The exact same run might take twice 
as long as it did last time, for no apparent reason. 
-USUALLY RepeatModeler hangs at step 5 or 6. That is, it never finishes, it just times out. 
You can try recovering the run, but this just brings you back to the beginning of the last step, 
and it usually just hangs again. In my experience, the final step takes more than 72 hours, 
which is the limit for a regular cori job. So you can either submit it to a long qos, 
or just use the data from the incomplete run. I generally do the latter. Not sure how much of
a difference this makes.

If you want to try to recover the run, you can make a new script that looks like this:
(use a longer queue if you are willing to wait > 72 hours)

#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=64
#SBATCH --time=72:00:00
#SBATCH --mem=118G
#SBATCH -q genepool
#SBATCH -A plntanot
#SBATCH -C haswell
#SBATCH --mail-user=vstartaglio@lbl.gov
#SBATCH --mail-type=end

/global/projectb/sandbox/plant/hybridum/software/RepeatModeler-2.0.1/RepeatModeler -database Bd21 -pa 4 -recoverDir RM_57941.TueOct200849412020

If you just want to use the data you've already gotten, you can just run RepeatClassifier on what you have, and
call it a day. Here's my RepeatClassifier batch script:

#!/bin/bash
#SBATCH -N 1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=64
#SBATCH --time=1:00:00
#SBATCH --mem=118G
#SBATCH -q genepool
#SBATCH -A plntanot
#SBATCH -C haswell
#SBATCH --mail-user=vstartaglio@lbl.gov
#SBATCH --mail-type=end

/global/projectb/sandbox/plant/hybridum/software/RepeatModeler-2.0.1/RepeatClassifier -consensi RM_57941.TueOct200849412020/consensi.fa -pa 64

Once you have a consensi.fa.classified file, you are finished with step 1d.
Please delete the temporary files produced by RepeatModeler. step2 will 
search the step1d directory for a file called consensi.fa.classified, and 
the search will go very slowly if all your temporary files are still there. 
################################################################################################################################################



















####################################### Step 2: Gather exemplars from Step 1 and eliminate redundancy #######################################

To do this step with snakemake: all you need in step2/ is the Snakefile. Just do snakemake. That's it.

To do this step without snakemake (not recommended): 
Now we will remove redundancy between the libraries we've made so far, including TREP + RepBase.
While in step2/:
cp ../step1/step1b/TransposonPSIresults.fasta.RMready .
cp ../step1/step1a/LTRharvest.exemplars.RMready .
cp ../step1/step1d/RM_*/consensi.fa.classified .
cp ../../Bd21/step0/TREP.RM.combined.final .

In python, do:
import timeForTE 
outlist = []
counter = 1
with open("{}/MITE-Tracker/results/{}/families_nr.fasta".format(timeForTE.softwaredir, timeForTE.genome), 'r') as inF:
	for line in inF.readlines():
		if line.startswith('>'):
			outlist.append( '>mite{}#DNA/Unknown\n'.format(str(counter)) )
			counter += 1
		else:
			outlist.append(line)

with open('families_nr.fasta.clean', 'w') as outF:
	for line in outlist:
		outF.write(line)



Concatenate all of these into one master library:
cat TransposonPSIresults.fasta.RMready LTRharvest.exemplars.RMready families_nr.fasta.clean consensi.fa.classified TREP.RM.combined.final > alllibs.redundant.fasta

Since CD-HIT is a greedy algorithm, sort elements by length, descending. In python do:
import timeForTE
timeForTE.sort_fasta("alllibs.redundant.fasta")

Let's once again say that TEs sharing 80% identity over 80% of their length are redundant.
/global/projectb/sandbox/plant/hybridum/software/cd-hit-v4.8.1-2019-0228/cd-hit-est -i alllibs.redundant.sorted.fasta -o alllibs.nr.fasta -c 0.8 -G 0 -aS 0.8 -n 5 -T 0 -d 0 -M 0 > CD-HIT.stdout
(-c: Need 80% identity)
(-G: Need 80% identity locally, not globally)
(-aS: the alignment must cover 80% of the shorter sequence)
(-n, -T, -d, -M: word size, threads, header length in .clstr file, memory usage)
################################################################################################################################################
















############################################## Step 3: Remove gene fragments from library ##############################################
Here we search the novel TEs collected so far against a plant protein database where proteins from transposons are excluded. 
Elements with significant hits to genes are removed, along with 50 bp upstream and downstream of the blast hit. 
Remaining sequence that is less than 50 bp is removed completely. Unfortunately the database of gene/protein sequences is a bit old.

For this step, do:
source activate /global/projectb/sandbox/plant/hybridum/software/step4

Your working dir should contain alluniRefprexp070416.

Do snakemake. It will submit runBLASTX.sh, a job that takes several hours to run.
#######################################################################################################################################

















################################################ Step 4: Final repeat masking ################################################
Now we are running RepeatMasker with the RepBase monocots library + the exemplars from homology + de novo searching. 
Do this one chromosome at a time. This not only speeds up the RepeatMasker step, it also allows us to do 
the curation in manageable chunks. 

Start out with this conda env: 
source activate /global/projectb/sandbox/plant/hybridum/software/step4
Then do 'snakemake'.
Then do:
conda deactivate
source activate /global/projectb/sandbox/plant/hybridum/software/TEdefault

The Snakefile should have produced a script to run the chromosomes
as a job array, runRepeatmasker.sh. 
Make sure the array script looks good, then submit. Give it more time if your genome is bigger than B. distachyon.
Note Repeatmasker delivers interesting stats in the .tbl file, e.g. Bd21.Bd1.fasta.tbl
###############################################################################################################################















########################################### Step 5: Obtain individual TE copies #############################################
Once all your jobs have finished running, it's time to parse the RepeatMasker output 
with 'one code to find them all'. This delivers final assembled TE copies.
Basically, RepeatMasker gives us lots of hits in the
genome to your exemplar library. I call these hits TE fragments. 'Onecode' assigns these
fragments to individual TE copies, based on the following criteria:
"Two hits located on the same scaffold or chromosome are considered to be fragments 
of the same copy if they abide by the three following conditions: 
1) they have the same orientation; 
2) the extremities of the fragments respect a distance criterion: 
by default the furthest extremities should be separated by less than twice the length of 
the reference TE element (see the --insert option for non-default behavior); and 
3) the second fragment starts and ends after the first one respectively starts and ends 
(that is, the two fragments can overlap but cannot be included in one another)."

So a drawback is that if you have large regions of heavily nested insertions, 
it will probably mis-identify those, calling the outer pieces separate TE copies. 
But honestly I think that's true of any automated curation method. 

Your step5 dir should contain:
step5setup.py
tidydict.py
Snakefile

TO DO THIS STEP WITH SNAKEMAKE: Just do 'snakemake'. That's it.

TO DO THIS STEP WITHOUT SNAKEMAKE (not recommended):
First, do python3 step5setup.py.
Next, do:
/global/projectb/sandbox/plant/hybridum/software/one_code_to_find_them_all/build_dictionary.pl --rm Bd21.all.fasta.out --unknown > Bd21.dict
^This command makes a 'dictionary' so onecode can determine which LTR
sequences belong with which internal sequences. Currently I'm NOT using the 
--fuzzy option. --fuzzy pairs LTRs and ints with similar family names, as assigned by 
repeatmasker, even if those names are not perfectly identical. So I'm requiring
identical names.

python3 tidydict.py
^This script overwrites the .dict file we just made, pairing up int and
LTR sequences which come from the same LTR-RT exemplar. (Theoretically the dict 
we just made should already have this, but for some reason "one code" didn't
recognize my LTR-harvest pairings.) So now "one code"  will correctly assign 
int and LTR fragments to the proper fl-LTR-RT.

This command runs onecode (takes a few minutes):
/global/projectb/sandbox/plant/hybridum/software/one_code_to_find_them_all/one_code_to_find_them_all.pl --rm Bd21.all.fasta.out --ltr Bd21.dict --fasta Bd21.all.fasta --flanking 100 --strict
^I don't know how to avoid getting so many lines to stdout, redirecting with > does nothing. :(
Note that I am including the --strict option, which means that I'm requiring 
families to adhere to the 80-80-80 rule. 
I'm not using the --choice option, which tells the program that you want to manually curate the copies. 
I did develop a strategy for manual curation using 'one code', but there are just SO many elements to 
curate, that it would be super arduous to do, especially on multiple genomes.
I even emailed the authors for advice at one point, and they sort of hinted that the 
manual curation isn't worth it if you have a "big" genome. (i.e. eukaryotic) 
###############################################################################################################################






















########################### Step 6: Barcode individual TE copies and fragments, get TE stats ########################
Just do 'snakemake'. That's it.

Additional info:
Now we have a bunch of TE copies and fragments. Let's give a unique name to every TE copy and every fragment.
Snakemake will grab the necessary output files from step5/ and then run three scripts I wrote:
onecode_to_flLTR-RTs.py, classify_fragments.py, getTEstats.py.

Note: See the header of classify_fragments.py for more details on how I unified these
different TE databases into one classification system. 
Note: getTEstats.py can be run with an option command-line argument. You can include
the name of a file with user-defined regions of interest, and the script will include
TE statistics for those regions, e.g. python3 getTEstats.py Bd_KEEs.bed
That file should have three columns (tab-separated): chromosome, start, and end.
It doesn't matter if the regions are contiguous or are separated into bins.

###############################################################################################################################





















################################################# Step 7: LTR-RT dating analysis ##############################################
See an older version of this document, TEannotation_2020_NEW.txt, for a description of my attempt 
to date solo-LTRs and fl-LTR-RTs together a la Stritt et al. 2020 ("Diversity, dynamics and effects 
of long terminal repeat retrotransposons in the model grass Brachypodium distachyon"),
i.e. getting terminal branch lengths from a tree. That effort did not exactly fail per se, but the 
trees always came out looking a little funky and it didn't seem worth learning phylogenetics just to get solo-LTRs. 

Use conda env /global/projectb/sandbox/plant/hybridum/software/step7

Now we will start working with TE families. I am focusing on LTR-retrotransposon (LTR-RT) families 
because these are important in plants, and methods for estimating LTR-RT insertion times are
well-established. 

This method is fairly standard practice, and it's simple, too. The only drawback is that it only works on
intact LTR-RTs. We align the 3' and 5' LTRs of individual LTR-RTs to each other with MAFFT. For each alignment 
(LTR-RT) we then calculate the evolutionary distance between the LTRs with EMBOSS, and then finally calculate 
the date of insertion based on the formula and substitution rate. 

See Vitte 2007 for a nice description of this method: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC1940013/ 
I believe Ma and Bennetzen 2004 is the proper citation for this substitution rate: 1.3*10^-8
I am using a substitution rate of 1.3*10^-8 and the formula T = D/2t, where T is the time elapsed since the 
insertion, D the estimated LTR divergence and t the substitution rate per site per year.

From my step7/ working dir:

Snakemake should have created a directory called LTRs_paired/ and populated it with fasta files.
cd into LTRs_paired and check that runmafft.sh looks okay. Submit that with sbatch. 
Once it's finished, do ./runEMBOSS.sh (very quick). Finally, run calcLTRages.py (also quick). 
The ages are in a file called finalAges.txt.

##############################################################################################################################



