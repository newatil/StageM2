##La génération de CCS (Circular Consensus Sequence) à partir des données brutes (Subreads) à été faite par Vincent Pailler (ingénieur de la plateforme de séquencage GENTYANE)
<br>
Les commandes suivantes sont utilisables avec la suite d'outils smrttools/10.1.0.119588 disponible sur HPC2 .
Doc disponible [https://www.pacb.com/wp-content/uploads/SMRT_Tools_Reference_Guide_v10.1.pdf] {: #https://www.pacb.com/wp-content/uploads/SMRT_Tools_Reference_Guide_v10.1.pd}
On est parti d'un pool d'amplicons/gènes multiplexés sur 96 cultivars de blé. A partir des données brutes (=subreads).

###CCS
 Les données CCS : généré avec le fichiers de subreads en input et le fichier ccs.bam en output:
```
module load smrttools/10.1.0.119588
ccs -j 128 --min-passes 3 *.subreads.bam ccs.bam
```
On a obtenu obtenu:
number of CCS:     2129864
total CCS length:  13198910525
mean CCS size:      6197.07

 les données brutes sont accessibles avec ce lien : 
[https://gentyane-share.s3.mesocentre.uca.fr/E779/ccs-E779-amplicons.bam?AWSAccessKeyId=33e6912b6a9f4ee7a0f48866691375ba&Expires=1646228054&Signature=PZSKDBTh%2FRgv3Pe0hZfUP%2F5i%2BJI%3D]
{: #https://gentyane-share.s3.mesocentre.uca.fr/E779/ccs-E779-amplicons.bam?AWSAccessKeyId=33e6912b6a9f4ee7a0f48866691375ba&Expires=1646228054&Signature=PZSKDBTh%2FRgv3Pe0hZfUP%2F5i%2BJI%3D}
<br>
###LIMA
Ensuite on a démultiplexé les données selon les 96 cultivars avec en input le fichier de reads CCS , le fichier de barcodes .fasta (n=96) et en output les 96 potentiels échantillons (demux.bc20XX--bc20XX.bam).

```
module load smrttools/10.1.0.119588
lima --split-bam-named -j 32 --ccs --peek-guess  ccs.bam /home/vipailler/Barcodes/SMRTbell_Barcoded_Adapter_Plate_3.0_bc2001-bc2096.fasta demux.bam
```

Le premier niveau de démultiplexage selon les 96 cultivars (94 réellement retrouvés) est fait. Maintenant on s'intéresse aux amplicons/gènes ; soit 18 amplicons * 96 cultivars . 
<br>

###Methode PBAA
##01_ccs_lima_pbaa.sh:

Ce script à été dévellopé en bash pour démultiplixé le CCS obtenus, en 96 fichiers bam propre à chaque accession de blé, en utilisant les barcodes.

##02_split_pbaa_multifasta.sh

Ce script correspand à l'outil pbaa (pacbio hifi amplicon analysis); command pbaa cluster, fournit avec la suite des outils SMARTTOOLS de PacBio, avec se script j'ai généré le deuxiéme niveau de dumltiplixage par 18 génes.
pbaa prend en input les fichiers bam propres et un fichiers multifasta avec des séquences guide pour chaque géne. Ici on a extrait les séquences guide dans la Refseqv1 de la variété Chinese Spring en utilisant les couples d'amorces PCR propres à chaque génes.

On obtient en output 3 fichiers de sorties pour chaque fichier d'entrée:
1 {prefix}_passed_cluster_sequences.fasta
2 {prefix}_failed_cluster_sequences.fasta
3 {prefix}_read_info.txt

pbaa cluster est basé sur des approches de clustering en utilisant les séaquences guide fournit en input,les séquences d'intérét se trouvent dans le fichier {prefix}_passed_cluster_sequences.fasta, qui correspand à un fichiers multifasta contenant une séquence fasta par géne. Qu'on a spliter pour obtenir 18 fichier fasta (un fichier fasta par géne).

https://github.com/PacificBiosciences/pbAA.git


##03_blastn_pbaa_revcom.sh

Certains de nos séquences fasta correspand à des amplicons sur les brins moins et d'autre sur les brins plus, j'ai donc dévellopé un code en utilisant des approches blast afin de distinguer les séquences qui corresspondent aux brins moins de celle de brins plus. Pour les séquences du brins moins j'ai généré le brin complémentaire avec l'outil revcom.pl (outil interne du gdec) dévelloper par Frédéric CHOULET (chef de la plateforme bioinfo inrae crouel).

##04_clustal.sh

Script d'alignement multiple, un aligement multiple par géne soit 18 aligement multiple, avant cette étape j'ai parser les fichiers fasta des 96 cultivar par géne.

##05_msa2vcf.sh

http://lindenb.github.io/jvarkit/MsaToVcf.html

Outil installé en local, j'ai pas pu l'installer sur HPC2 à cause des raison de sécurité d'administation du systéme (Proxy).
msa2vcf est un outil qui prent en input le fichiers d'aligements multiple pour générer un fichier de variant (vcf file), qui indique les positions des SNPs; et les indels.


##06_vcftools.sh

https://github.com/vcftools/vcftools.git

Le vcftools préalablement installé sur HPC2, à été utilisé pour la génération des statistiques sur les fichiers de variant obtenu avec msa2vcf, dont les fréquences alléliques,statistiques de diversité génétique.


##07_gmap_18genesCDS_GLU_vs_18_consensus.sh

Ce code utilise l'outil gmap pour extraire les position start et end des 18 génes, dans le but de calculer des statistiques sur la partie codantes (CDS) et non codante.


##08_glutenin_diversity_analysis.R

Script développer en R, pour extraire les génes et pour générer les haplotypes, et calculer le taux de mutations sur la partie codante et non codante;
