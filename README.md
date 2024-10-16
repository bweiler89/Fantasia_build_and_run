### This is solely to be used as a resource for downloading and installing/running FANTASIA. 

Bits of this code is taken directly from their [Github](https://github.com/MetazoaPhylogenomicsLab/FANTASIA/tree/main?tab=readme-ov-file), however I figured it usedful to document how I got it to work. Unfortunately, This is likely too resource heavy to be run on a standard work computer, therefore I would run this on a local HPC/Cluster.

--- 
```
 ######   ##   #    # #####   ##    ####  #   ##   
 #       #  #  ##   #   #    #  #  #      #  #  #  
 #####  #    # # #  #   #   #    #  ####  # #    # 
 #      ###### #  # #   #   ######      # # ###### 
 #      #    # #   ##   #   #    # #    # # #    # 
 #      #    # #    #   #   #    #  ####  # #    # 
                                                   

FANTASIA (Functional ANnoTAtion based on embedding space SImilArity)
```
First, I think it's important to understand what this program does, how it leverages protT5 and goPredSim to annotate transcript proteins to GO terms. Let's break it down: 

**GOPredSim** is a tool that predicts Gene Ontology (GO) terms for proteins by leveraging a set of proteins with known GO annotations and comparing them to the proteins you're interested in annotating. To do this, it uses protein embeddings, which are representations of protein sequences that capture biological features. GOPredSim compares every protein in the reference dataset to every protein in your dataset by calculating pairwise distances between their embeddings. Embeddings are numerical representations of protein sequences generated by models like ProtT5 (which is a transformer-based model). These embeddings capture meaningful biological properties, allowing the comparison between proteins based on their sequence characteristics in a high-dimensional space. The distance between embeddings is a measure of how "similar" two proteins are - a smaller distance indicates more similarity.

In FANTASIA, ProtT5 is used to generate these embeddings, and GOPredSim transfers GO terms based on the similarity of these embeddings between known and target proteins. ProtT5 uses two modes to define similarity:

1. k-nearest neighbors (k-NN): This mode selects the k most similar proteins from the lookup dataset for each target protein. The GO terms of these neighbors are combined and assigned to the target protein.
2. Distance threshold: This mode selects all proteins within a certain distance d from the target protein in the embedding space. GO terms are transferred from these neighbors to the target protein.

Then, the distance between a target protein and its nearest neighbors (in the embedding space) is converted into a similarity score. Proteins that are closer to the target protein provide more reliable predictions. If a GO term is annotated to multiple neighbors of a target protein, it gets a higher reliability score. This is because multiple sources agreeing on the same function make the prediction more trustworthy.

## Summary of the Fantasia Pipeline:

Input: Your protein sequences (filtered through CD-HIT).

Steps:
1. Use ProtT5 to create embeddings for these sequences.
2. GOPredSim compares these embeddings to known proteins (reference DB) with GO annotations.
3. Transfer GO terms from the reference DB to your target proteins based on similarity.
4. Convert the output into formats (like topGO) suitable for further downstream RNAseq analyses.


# <p align="center">Downloading Fantasia</p>

### Install from source from the [Fantasia Github](https://github.com/MetazoaPhylogenomicsLab/FANTASIA/tree/main?tab=readme-ov-file)

This is a large download ~13GB so make sure that you are connected via ethernet or have good wifi

Once downloaded, scp it to your local cluster for analyses, this program takes a fair amount of ram

```
scp -r FANTASIA.tar.gz user@super.computer.edu:/scratch/projects/path/to/directory
```
In the directory you want to decompress this file
```
tar -xvzf FANTASIA.tar.gz
```

Note: This next section is pulled directly from their Github

However, there were a lot of dependency issues, leading to failures along the way. Just read the outputs carefully and it will guide you in how to proceed. 

```
conda create --name gopredsim --file tf_conda_env_spec.txt python=3.9.13

conda activate gopredsim #Check that everything is correctly installed

conda install -c bioconda seqtk #Install seqtk

# Create the python environment and install the required packages

#Note: make sure that you are executing the python from the conda environment. Otherwise, there may be version conflicts

#If encountering any error saying that the version of the package is not available or not compatible with other package's version, modify requirements.txt and remove the version (it is indicated by "==XX.XX.X" and it will automatically find the one that is compatible with the rest of the packages). Be aware that the changing the version of some packages may affect the inference of the embeddings and the validity of results obtained.

#Note: to deactivate python environments, just type "deactivate"

#General enviromment used for ProTT5 model (and could potentially be used for other models except SeqVec)

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt

deactivate

# Change "/path-to-FANTASIA-folder/" to the correct path in the scripts for launching the program.

FANTASIA_PATH=$(pwd) #You should be inside teh FANTASIA folder

sed -i "s|/path-to-FANTASIA-folder|$FANTASIA_PATH|g" launch_*.sh

# Change "/path-to-FANTASIA-folder/" to the correct paths in the script to generate the config files required by the software.

sed -i "s|/path-to-FANTASIA-folder|$FANTASIA_PATH|g" generate_gopredsim_input_files.sh

# GOPredSim can be run using CPUs or GPUs. CPUs is the default option selected in these scripts. However, GPUs can be used to increase the speed when dealing with large amounts of data. In clusters, make sure to select GPU nodes and to load the correspondent CUDA module when launching the pipeline. A sample script for executing the pipeline is included. Paths for execution and conda activation should be double checked.

# By default, ProtT5 model files is included in the GOPredSim/models folder. If those files are removed, GOPredSim will automatically download it in the user .cache. If GOPredSim is used in a cluster, this download is user-specific, and, as files are quite big in some cases, it may rise an error if not enough space. To add more models, download files using the links found in /GOPredSim/venv/lib/python3.9/site-packages/bio_embeddings/utilities/defaults.yml and put them in the GOPredSim/models folder. In addition, this information must be added to the configuration files for the embedding part (.yml files) as explained in https://github.com/sacdallago/bio_embeddings/issues/114.

# The folder includes two additional scripts to convert the format on the output to the input of topGO and to collapse GO annotation of several isoforms into GO annotation per gene. Additional information on how to run the pipeline and these steps can be found on GitHub (https://github.com/MetazoaPhylogenomicsLab/FANTASIA).

```

# <p align="center">Using Fantasia</p>

Fantasia uses an input protein sequence file and annotates GO terms using GOPredSim. 
![FANTASIA_pipeline](https://github.com/user-attachments/assets/bc62f7b9-b9ec-4446-8ed4-6b9c21028956)

Filtered (using transdecoder) protein file should end in .pep and should look something like this
```
>TRINITY_DN10008_c0_g1_i1.p1 TRINITY_DN10008_c0_g1~~TRINITY_DN10008_c0_g1_i1.p1  ORF type:complete len:264 (-),score=53.73 TRINITY_DN10008_c0_g1_i1:64-747(-)
MTADTMTAATLPVASSTWVSPSVQKWTFHQPSDGDGHKGRNLTWSEALELMENSPRFREM
LISKLQESPFNAFFWECTPLTEHTAQHRSFEFVIMEGSHLDSARPDTESFSDYLADFKGQ
PVARAFSNLGGDSSLISPAQATKNPEDYKHIGNFFRRAPMEQRHAVFQTLGQELQRRLAQ
KPEAPYWVSTEGSGVAWLHMRIDPRPKYYHHREYRSPEYGLVQKGEL
>TRINITY_DN10011_c0_g1_i1.p1 TRINITY_DN10011_c0_g1~~TRINITY_DN10011_c0_g1_i1.p1  ORF type:5prime_partial len:218 (+),score=9.82 TRINITY_DN10011_c0_g1_i1:2-655(+)
HLPLAPPSYLAAFPVSTLACTQIFFIRLYACVLHVSAIMDFSAWFEDVRRMNKRQLFYQV
LNFAMIVSSALMIWKGLMVVTGSESPIVVVLSGSMEPAFQRGDLLFLTNYKEDPIRVGEI
VVFKVEGREIPIVHRVLKIHEKKDGAIRFLTKGDNNSVDDRGLYAPGQLWLQRKDVVGRA
RGFVPYVGMVTILMNDYPKFKYAILVALGAFVLLHRE
>TRINITY_DN10011_c0_g1_i2.p1 TRINITY_DN10011_c0_g1~~TRINITY_DN10011_c0_g1_i2.p1  ORF type:5prime_partial len:218 (+),score=9.82 TRINITY_DN10011_c0_g1_i2:2-655(+)
HLPLAPPSYLAAFPVSTLACTQIFFIRLYACVLHVSAIMDFSAWFEDVRRMNKRQLFYQV
LNFAMIVSSALMIWKGLMVVTGSESPIVVVLSGSMEPAFQRGDLLFLTNYKEDPIRVGEI
VVFKVEGREIPIVHRVLKIHEKKDGAIRFLTKGDNNSVDDRGLYAPGQLWLQRKDVVGRA
RGFVPYVGMVTILMNDYPKFKYAILVALGAFVLLHRE
```

Fantasia uses GOPredSim with the protein language model ProtT5 to achieve GO terms used in downstream analyses. 

In order to do this correctly, I needed to build a job script, which would run the .sh

#pstr_generate_gopredsim_input_files.job
```
#!/bin/bash
#BSUB -J pstr_generate_gopredsim_input_files
#BSUB -e pstr_generate_gopredsim_input_files.err
#BSUB -o pstr_generate_gopredsim_input_files.out
#BSUB -q bigmem #this is the queue, we need bigmem for some of this work
#BSUB -P coralma #this is my personal project, use yours
#BSUB -n 8
#BSUB -R "rusage[mem=10000]"
#BSUB -B
#BSUB -W 120:00
#BSUB -N
#BSUB -u user@school.edu #input your user ID

# Change to the working directory
cd /scratch/projects/path/to/directory/FANTASIA

# Define variables for paths and options
PEP_FILE="Trinity_filtered.fasta.transdecoder.pep"
OUTPUT_PATH="/scratch/projects/path/to/directory/FANTASIA/gopredsim_output/"
PREFIX="pstr"  # Customize this as needed
CONFIG_PATH="/scratch/projects/path/to/directory/FANTASIA/"  # Current directory; change if you have a specific config path
MODE="cpu"  # Set to "gpu" if you want to use GPU

# Create the output directory if it doesn't exist
mkdir -p $OUTPUT_PATH

# Generate GOPredSim input files
echo "Generating GOPredSim input files..."
./generate_gopredsim_input_files.sh -i $PEP_FILE -o $OUTPUT_PATH -c $CONFIG_PATH -p -m $MODE --prefix pstr
```

This execution uses CD-HIT to remove identical sequences and any seqs longer than 5000 amino acids (model constraint). The output files should be:
1. [filt_fasta]_cdhit100_5k_removed.pep
2. [filt_fasta]_cdhit100_headers_larger5k.txt
3. [filt_fasta]_cdhit100_headers_smaller5k.txt
4. [filt_fasta]_cdhit100.pep
5. [filt_fasta]_cdhit100.pep.clstr
6. [filt_fasta]_cdhit100_seqs_larger5k.txt

This step also produces a config_files directory with two sub directories, embeddings and gopredsim. In embeddings and gopredsim, you should have a [PREFIX]_prott5.yml

Your job script will also shoot out an .err file (sources of error if there are any) and an .out file (outcome of the job).

Next, we want to actually run the launch_gopredsim_pipeline.sh, so lets build a job script

### Errors you may encounter

1. The pipeline itself is hardcoded to access python from /opt/conda/envs/gopredsim/bin/python, where it was actually downloaded to /nethome/user/miniconda3/envs/gopredsim/bin/python
2. Packages h5py, sklearn (scikit-learn), numpy, and pathlib are needed prior to running this next step
3. The launch_gopredsim_pipeline.sh has conda activate gopredsim in the script, however if you run conda activate gopredsim before running the job it is redundant and causes the script to exit, so hash out both #conda activate gopredsim and #conda deactivate gopredsim at the end
```
conda install conda-forge::h5py #this worked over other methods
pip install scikit-learn #conda did not work to install this package
```

```
#!/bin/bash
#BSUB -J pstr_launch_gopredsim_pipeline
#BSUB -e pstr_launch_gopredsim_pipeline.err
#BSUB -o pstr_launch_gopredsim_pipeline.out
#BSUB -q bigmem #this is the queue, we need bigmem for some of this work
#BSUB -P coralma #this is my personal project, use yours
#BSUB -n 8
#BSUB -R "rusage[mem=10000]"
#BSUB -B
#BSUB -W 120:00
#BSUB -N
#BSUB -u user@school.edu #input your user ID

# Change to the working directory
cd /scratch/projects/path/to/directory/FANTASIA

# Load conda and activate the environment, change the source according to where conda.sh is (alternatively you can activate the conda env before submitting the script and omit this step)
source /nethome/user/miniconda3/etc/profile.d/conda.sh
conda activate gopredsim

# Run the GOPredSim pipeline with the correct path to the config file
./launch_gopredsim_pipeline.sh -c /scratch/projects/path/to/directory/FANTASIA/ -x pstr -m prott5 -o /scratch/projects/cpath/to/directory/FNTASIA/gopredsim_output
```

This job took roughly 12 hours to run with 80GB provided, though it only used ~34GB.

### The output files are: 

1. gopredsim_{prefix}_prott5_1_bpo.txt #Biological Process Ontology
2. input_parameters_file.yml
3. {prefix}_prott5_embeddings/ #this is a folder containing input_parameters_file.yml  ouput_parameters_file.yml  reduced_embeddings_file.h5
4. gopredsim_{prefix}_prott5_1_cco.txt #Cellular Components Ontology (or subcellular localizations)
5. mapping_file.csv
6. remapped_sequences_file.fasta
7. gopredsim_{prefix}_prott5_1_mfo.txt #Molecular Function Ontology
8. ouput_parameters_file.yml
9. sequences_file.fasta

## Note on the three ontologies
1. MFO predictions are often more straightforward, as they deal with specific molecular activities of proteins that are more directly tied to sequence features​.
2. BPO is considered more challenging due to its complexity. Predicting biological processes involves more layers of interaction and tends to have lower performance scores compared to MFO and CCO​.
3. CCO often falls in between MFO and BPO in terms of prediction accuracy. Cellular component predictions, while still complex, tend to have fewer hierarchical dependencies than BPO, making them slightly easier to predict than biological processes​.

## Helpful information
After running this on my data, I found that it added 79,610 unique go terms to my transcripts. 

| # eggNOG Go Terms | # Fantasia Go Terms | # Unique eggNOG GOs | # Unique Fantasia GOs | # Shared GOs |
|-------------------|---------------------|---------------------|-----------------------|--------------|
| 630655            | 88389               | 621,896             | 79,610                | 8779         |
