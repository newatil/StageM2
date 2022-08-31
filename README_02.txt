methode script python + emboss pour l'assemblage des amplicons pacbio laisse de cote pour suivre la methode pbaa 

###22_Demultiplexing_v2.py

Script python qui prends en argument les couples des amorces propres à chaque géne afin de démultiplixer les fichiers par géne. En obtient en sortie 18 fichiers pour chaque accession de blé soit 18*96= 1728 fichiers mulifasta, chaque fichier contient des milliers de séquences qui correspondent à tous les amplicons d'un seul géne.

###21_demux_python-emboss_level2.sh

Script bash pour éxécuter le script "22_Demultiplexing_v2.py"

###23_Run_extract_target_Seq.sh

Script bash pour éxécuter le script "24_extract_representativeSeq.py"

###24_extract_representativeSeq.py

Script en language python pour flitrer les fichiers de sortie du script précédent, j'ai utilisé d'abort l'outil cd-hit 
(https://github.com/weizhongli/cdhit.git)
pour garder uniquement les séquences les plus représentatives ( le cluster qui contient le max de séquence avec un pourcentage d'identité le plus élevé >99,9), et j'ai appliquer ce script python aux fichiers de sortie de cd-hit. 

###25_blastn_pid95.sh

Code utilisant l'outil BLAST contre les séquences de Refseqv1 de CS (Chinese Spring); pour filtrer d'avantage les séquences selon un pourcentage d'identité suppérieur ou égale à 95%. et contre le brins moins et plus (paramétre strand minus/plus).

###26_getfasta_from_blast95_stminus.sh

Pour les séquences blaster contre le brins moins,j'ai générer la révérse complémentaire grace à se script;

###27_merge_fasta_files.sh

se code sert à parser les séquences fasta qui blaster contre le brin plus avec celle que j'ai généré avec revcom.pl;

###28_cdhit_merged_fasta.sh

j'ai réutiliser l'outil cd-hit sur les séquences ayant un pourcentage d'identité suppérieur à 95%, pour filtrer d'avantage les séquences; le output renvoies un seul cluster pour tout les génes (séquences homogénes bien nettoyé); il s'agit d'une étape de véréfication.


###29_emboss_with_plurality.sh

Script pour la génération d'une seule séquence consensuse par géne et par individu; j'ai essayé avec des assembleurs des longreads Canu (https://github.com/marbl/canu.git); Falcon_unzip (https://github.com/PacificBiosciences/FALCON_unzip.git) et aussi Mira (https://github.com/bachev/mira.git), ces assembleurs fonctionne sur des données de type GWAS uniquement, et n'arrivent pas à générer un assemblage correcte des amplicons qui correspand au méme géne, c'est l'étape qui m'a pris beaucoup de temps dans mon stage, j'ai utilisé donc comme alternative l'outil emboss cons (https://galaxy-iuc.github.io/emboss-5.0-docs/cons.html) mais la qualité des séquences consensuses de sortie de l'outil est médiocre, et j'ai donc abondonné ce pipline; et j'ai réaliser la suite de mes analyses avec la pbaa-methode.
