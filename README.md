# TANA
TANA (Transferring Annotation via Network AlignmenT) is a tool for inferring protein function which is based on our approach MAPPIN for GNA (Global
Network Alignment).

Written By: Warith Eddine DJEDDI (waritheddine@yahoo.fr),
            Sadok BEN YAHIA (sadok.ben@taltech.ee) and
            Engelbert MEPHU NGUIFO (engelbert.mephu_nguifo@uca.fr)

This README describes the usage of the command line interface of TANA. 
The executable TANA is compiled for Linux x86_64 platform.

The program TANA finds a global alignment of multiple input
networks. Given multiple networks with N1, N2,..., Nk nodes each,
it returns a matching between the input networks, each match corresponding to
best-matching nodes from the multiple networks.

To understand how to use the algorithm, let's start with an example of
pairwise alignment of two input networks. The multiple case is similar.


(1) Suppose the species are named 'A' and 'B'

(2) Create the graph files and results from the BLAST runs (between the
    nodes) in a single directory. You'll need 5 files:

  (2.1) Network files:
     You'll need A.pin and B.pin , tab-separated files
     where each line contains an interaction. For example, the first 5 lines of
     A.pin are:

     ====== BEGIN ========
     INTERACTOR_A INTERACTOR_B
     a0 a1
     a0 a2
     a0 a3
     a0 a4
     ====== END ========

      Columns are separated by tabs. The first line is a header line of the
     form as shown above. All other lines describe an interaction, one per
     line.

     There may be a third column which contains edge weights (0 < wt <= 1) 
     and in that case the header line should've a third column titled Weight_VAL.

  (2.2) Results of BLAST runs.
     You'll need to perform an the all-against-all run
     of BLAST between all the nodes of the two networks. You should store the results in 3 files:

        A-B.sim,  A-A.sim, B-B.sim

     The files contain, as their names indicate, the results of BLAST runs
     between species A & B, A & A, and B & B, respectively. IMPORTANT: for
     files containing Blast scores between two species, the filename should
     have the species names in lexicographic order, i.e., A-B.sim is
     expected, not B-A.sim.

     The first 5 lines of the A-B.sim file are:

     ====== BEGIN ========
     a0      b0      1
     a1      b1      1
     a2      b2      1
     a3      b3      1
     a4      b4      1
     ====== END ========

     Each line is of the form:
     <id1>  <id2>   <Bit-Score>



  (2.3)  Gene ontology (GO) File
 
     IMPORTANT: Download gene ontology file from the Website "http://www.geneontology.org/" and uncompress it 
     under the "TANA\data" folder. The name should be "gene_ontology.1_2.obo" under the data folder.


  (2.4) GO annotation file which contains GO annotations for proteins in the input networks. The format of this GO annotation file should be compliant with the GO consortium.
     
    A) File: goa_uniprot_gcrp.gaf → This set contains all GO annotations for canonical accessions
      from the UniProt reference proteomes for all species, which provide one protein per gene.
      
    B) Files: goa_<species>.gaf for each species → This set contains all GO annotations for canonical accessions 
      from the UniProt reference proteome for the species, which provides one protein per gene. 
      
      IMPORTANT: Download gene annotation files for each species from the Uniprot Website "http://www.ebi.ac.uk/goa/downloads" and uncompress them under the "TANA\data\goa" folder.

(3) Create a file that specifies the file locations, species names etc.
   In the folder config/, the file "policy.input".

   For example, suppose we have three PPI networks: a.pin, b.pin, c.pin, GO association file for the 3 species: a.gaf, b.gaf, c.gaf,
   six blast e-value (or bitscore) files: a-a.sim, a-b.sim, ..., c-c.sim, then the policy file should looks like:

          --------------BEGIN OF POLICY--------------
                    PARAMETER VALUE
                    network   dataset/a.pin
                    network   dataset/b.pin
                    network   dataset/c.pin
                    goafile   data/goa/a.gaf
                    goafile   data/goa/b.gaf
                    goafile   data/goa/c.gaf
                    efile     dataset/a-a.sim
                    efile     dataset/a-b.sim
                    efile     dataset/a-c.sim
                    efile     dataset/b-b.sim
                    efile     dataset/b-c.sim
                    efile     dataset/c-c.sim
                    scorefile     dataset/score_composit.model
                    alignmentfile ./result/alignment_TANA_rnd1.data
                    logfile       ./result/measure_time.txt
          --------------END OF POLICY--------------

 

(4) Call the code. Here are samples:
    
   (4.1) Network files using blast e-value:
      
      ./tana -alignment -alpha 0.3 -nmax 1000 -temp 50 -thr 0.3 -numspecies 3 -numthreads 8 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/
    
   (4.2) Network files using blast bitscore:
      
      ./tana -alignment -alpha 0.3 -nmax 1000 -temp 50 -thr 0.3 -numspecies 3 -bscore true -numthreads 8 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/

   The options are as follows (you can also use the "-h" or "--help" flag):

     Usage:
     ./tana -version|-alignment [--help|-h|-help] [-alignmentfile str]
       [-alpha num] [-bscore] [-edgefactor num] [-nmax int] [-numspecies int]
       [-numthreads int] [-resultfolder str] [-temp int] [-thr num]
     Where:
      --help|-h|-help
         Print a short help message
      -alignment
         Execute the alignment algorithm.
      -alignmentfile str
         The filename of alignment of protein-protein interaction networks
      -alpha num
         Prameter controlling how much biological score contributes to the alignment score. Default is 0.3.
      -bscore
         Define bitscore as the similarity for the edges.
      -edgefactor num
         The factor of the power law normalization. Default is 0.1.
      -nmax int
         The parameter N for Simulated Annealing algorithm.
      -numspecies int
         Number of the species used in the aligning process. Default is 3.
      -numthreads int
         Specifies the number of threads used to run TANA. 
         The recommended number of threads is the number of cores available in the computer.
      -resultfolder str
         The folder which was used to store the alignment results.
      -temp int
         The number of iteration for Simulated Annealing algorithm.
      -thr num
         The threshold for the alignment of protein-protein interaction networks.
      -version
         Show the version number of TANA.

(5)  The output

      The main files are :
       (5.1) The output of the alignment is located in the file "result/alignment_TANA_rnd1.data". Each protein is represented 
       by a string (separated by a tab), and each interaction is on a single line.
       (5.2) The affectation from UniprotGOA for for each protein belonging to the N input networks is located 
       in the file "result/affectProtToGene.txt".
       (5.3) The number of unkown functional protein is located in the file "result/alignUnknown_rnd1.result".
       (5.4) The predictions for the unannotated protein using the "CAFA id" are in the file "Predictions/CAFA-Predicted-GOTerms.txt".
       (5.5) The predictions for the unannotated protein using the "Uniprot id" are in the file "Predictions/List-Predicted-GOTerms_rnd1".
       (5.6) The file "Predictions/scoreGOTerms_rnd1.txt" contains the scoring of GO Terms by transferring shared annotation to unannotated protein (marked with *) in each cluster belonging to the file "result/alignUnknown_rnd1.result".
       (5.7) The required time to align the input networks is located in the file "measure_time.txt".
       (5.8) The sequence, topological and functional similarities for each pair of compared protein 
       is located in the file "result/scoreRecords.txt".
       (5.9) The informations : Alignment edges conserved for each species, the number of alignment nodes, 
       the Alpha parameter, nmax and the Alignment score, are located in the file "result/alignment_statistics.data".
       (6.1) The number of unknown alignment records, and the number of proteins annotated with MF,BP and CC in alginment graph,
       are located in the file "result/statistics.result".

(6)  EXAMPLE
       (6.1) Download TANA freely available at Github website: https://github.com/waritheddine/TANA

       (6.2) Run TANA on our test dataset with command:
       ./tana -alignment -alpha 0.3 -nmax 1000 -temp 50 -thr 0.3 -numspecies 8 -bscore true -numthreads 8 -alignmentfile ./result/alignment_TANA.data -resultfolder ./result/

       (6.3)Then you can find the all the involved output files in ./result/ . There are many other functions which you can see with "-help" option.
       (6.4) We note that checking the format of the files in the data folder after reading the execution instructions above might be quite helpful.
       (6.5) For this example, download the gene annotation file (goa_arabidopsis.gaf.gz, goa_worm.gaf.gz, goa_fly.gaf.gz, goa_ecoli.gaf.gz, goa_human.gaf.gz, goa_mouse.gaf.gz, goa_rat.gaf.gz, goa_yeast.gaf.gz) for the eight species 
       (Arabidopsis, Worm, Fly, Ecoli, Human, Mouse, Rat and Yeast) from the Uniprot Website "http://www.ebi.ac.uk/GOA/downloads" and uncompress them under the "TANA\data\goa" folder. In addition, download the compress file "goa_uniprot_gcrp.gaf.tar.gz" from the same Url, and also uncompress it under the "TANA\data\goa" folder.
       Finally, download gene ontology file from the Website "http://www.geneontology.org/" and uncompress it under the "TANA\data" folder.
