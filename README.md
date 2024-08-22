# Mutation correlation prediction using the Clique model
## Step1. Construction of protein mutant structures
You need to prepare the sequence file of the target protein, usually a FASTA sequence file.You can find the FASTA sequence files for the needed proteins in the [RCSB PDB](https://www.rcsb.org/). 

For already available FASTA sequence files, you can manually edit the sequences using the ***vim*** editor on Linux systems. The FASTA sequences were then predicted by the online version of [AlphaFold2](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb#scrollTo=kOblAo-xetgx) or the local ColabFold environment to obtain the accurate protein structure. 

The structure with the highest pLDDT score in the prediction result was selected for follow-up use

For large-scale structural predictions it is recommended to use the local ColabFold environment.

## Step2. Construction of Residual-Contact Network on Protein Structures
In the previous step, we got the structure of the protein and the file was saved in PDB format.
``
1stn.pdb
``

Generation of protein interaction force networks by Residue Interaction Network Generator [RING](https://ring.biocomputingup.it/)



After a successful operation you will get the node and edge files of the amino acid residue interaction force network graph.

This step can be deployed through an online website or local environment,Large-scale amino acid residue interaction force network generation is recommended using a local environment deployed in linux [DOWNLOAD – BioComputingUP](https://biocomputingup.it/services/download/).

The local environment runs as follows, take RING 4.0 as an example

#See how to use RING
``
ring -h
``

#Import the pdb file and set the folder where the results are stored.
``
ring -i 1stn.pdb --out_dir dirname
``

You will get these two files containing node information and edge information, respectively, for the amino acid residue interaction network graph in the structure 1stn
``
1stn.pdb_ringEdges  1stn.pdb_ringNodes
``

RING 4.0 supports the adjustment of parameters for the formation of amino acid residue interaction force networks, including thresholds for hydrogen bonds, van der Waals forces, etc. You can refer to the settings in the online version, or use the ``ring -h `` command to view and adjust them manually.
## Step3. Calculation of 3-Clique Community in Amino Acid Residue Interaction Force Network Graphs
**#Put our script in the same folder as the two files we got in the previous step**
You need to place the node and edge information files of the amino acid residue interaction force network graph in the same folder directory as our script ``all_atom_clique.py``

The folder needs to contain the following files:
``
1stn.pdb_ringEdges  1stn.pdb_ringNodes  all_atom_clique.py
``

**#Change the variable for filename in the all_atom_clique.py script to match the prefix of the node and edge files**
Using the editor to edit our script ``all_atom_clique.py``

and change the filename line in the code.
``
filename = '1stn'
``

The variables in filename need to be the same as the filenames of the nodes and side files.
``
1stn.pdb_ringNodes 
``
For example, the **1stn** in the node filename

**#Save changes and run the script**
``
python all_atom_clique.py	
``
**#output result**
``
[frozenset({72, 11, 12}), frozenset({16, 24, 25, 14, 15}), frozenset({18, 19, 22, 45}), frozenset({35, 36, 21}), frozenset({34, 27, 76}), frozenset({34, 22, 23}), frozenset({90, 91, 37}), frozenset({42, 53, 54}), frozenset({49, 50, 46}), frozenset({69, 70, 71, 95}), frozenset({72, 73, 93}), frozenset({121, 91, 75}), frozenset({80, 117, 118, 79}), frozenset({83, 86, 87}), frozenset({88, 33, 87}), frozenset({88, 89, 35}), frozenset({100, 99, 92}), frozenset({101, 102, 104, 105}), frozenset({137, 138, 105, 106, 107, 139}), frozenset({130, 126, 127}), frozenset({129, 130, 133})]
21
``
The amino acid sites contained in each 3-Clique Community are shown in parentheses.The last number is the number of 3-Clique Communities formed.

**Notice**: To ensure that the scripts run properly please install the pandas, numpy, and NetworkX libraries.

## Step4. Predicting mutation correlations
Double mutation sites in the same 3-Clique Community will be non-additive, and mutations will be considered additive if no 3-Clique Community is formed between the two sites.
To account for more complicated scenarios that cannot be covered by this simple rule, we have added the following additional terms to the model: 
1. When two mutations are introduced into the same α-helix to form a new, larger hydrophobic core, the model predicts that it is an additive double mutation even when a 3-clique community is formed
2. Two mutations in adjacent α-helices that form a 3-clique community typically show additive effects. These mutation pairs may be considered potential additive sites;
3. Two interacting mutations that are isolated from others cannot form a 3-clique community, yet they may still exhibit non-additivity.
