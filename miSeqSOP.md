*last updated October 11, 2020*  
This script is used for mothur written in C++ language for processing of Illumina miSeq sequences (v3v4 primer pair)  
based on Schloss *et al*. (2009) and Kozich *et al*. (2013)  
and some modifications from https://github.com/ianpgm
https://mothur.org/wiki/miseq_sop/

This project is under **mabini**

>**required files**  
>  1) **trim.contigs.fasta**: fasta file from make.contigs after nextera adapters were removed using TrimGalore (see usegalaxy guide to obtain this file)  
>  2) **metadata.txt** file  
>  3) **silva.nr_v132.pcr.align**: below is the script for creating this file using silva.nr_v132.align as starting file
```
  mothur > pcr.seqs(fasta=silva.nr_v128.aligm, start=6388, end=25316, keepdots=F, processors=20)
  mothur > unique.seqs()
  summary.seqs(name=current)
```

1. The first step is to look at the summary of your sequences.
```
  mothur > summary.seqs(fasta=trim.contigs.fasta)
```
2. Next we remove reads with ambiguous base calls and long contigs. This also generates a name file.
```
  screen.seqs(fasta=trim.contigs.fasta, group=basename.contigs.groups, maxambig=0, minlength=400, maxlength=500)
```
3. We then look for unique sequences only.
```
  unique.seqs(fasta=trim.contigs.good.fasta)
```
4. To keep track of the unique sequences among different samples.
```
  count.seqs(name=basename.trim.contigs.good.names, group=basename.contigs.good.groups)
```
5. Align the sequences to the trimmed silva database that you prepared earlier. Replace reference=/path/to/database/silva.nr_v123.pcr.align iwht the path to your own database.
```
align.seqs(fasta=basename.trim.contigs.good.unique.fasta, reference=/path/to/database/silva.nr_v123.pcr.align)
```
6. Look again at the summary of the sequences.
```
summary.seqs(fasta=basename.trim.contigs.good.unique.align, count=basename.trim.contigs.good.count_table)
```
7. Most will align from position 6428 to 23440, but some will not. Use the following command to remove those sequences and as well as those with a string of identical bases (homopolymers) longer than 8 bases using the maxhomop command.
```
screen.seqs(fasta=basename.trim.contigs.good.unique.align, count=basename.trim.contigs.good.count_table, summary=basename.trim.contigs.good.unique.summary, start=6428, end=23440, maxhomop=8)
```
8. By then your sequences should have aligned with actual base pairs in the database and not the flanking gaps (marked by .). We make sure this is the case by filtering out any bases that aligned outside the region of interest.
```
filter.seqs(fasta=basename.trim.contigs.good.unique.good.align, vertical=T, trump=.)
```
9. After this we are only interested in the unique sequences.
```
unique.seqs(fasta=basename.trim.contigs.good.unique.good.filter.fasta, count=basename.trim.contigs.good.good.count_table)
```
10. We then perform pre-clustering to merge near-identical sequences together (we remove sequencing error = 1 in every 100 bases, not true biological variation). Since our amplicons are around 450 bp long, we'll precluster based on 4 mismatches.
```
pre.cluster(fasta=basename.trim.contigs.good.unique.good.filter.unique.fasta, count=basename.trim.contigs.good.unique.good.filter.count_table, diffs=4)
```
11. Now we look for chimeras using the vsearch algorithm.
```
chimera.vsearch(fasta=basename.trim.contigs.good.unique.good.filter.unique.precluster.fasta, count=basename.trim.contigs.good.unique.good.filter.unique.precluster.count_table, dereplicate=t)
```
12. And proceed to remove the chimeras from our fasta file.
```
remove.seqs(fasta=basename.trim.contigs.good.unique.good.filter.unique.precluster.fasta, accnos=basename.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.accnos)
```
13. We now want to focus on archaeal andbacterial sequences with the use of a Bayesian classifier.
```
classify.seqs(fasta=basename.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=basename.trim.contigs.good.unique.good.filter.unique.precluster.denovo.uchime.pick.count_table, reference=/path/to/database/silva.nr_v123.pcr.align, taxonomy=/path/to/database/silva.nr_v123.pcr.tax)
```
14. Now we want to separate our data to Bacteria and Archaea.  
for Bacteria:
```
remove.lineage(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.taxonomy, taxon=Chloroplast-Mitochondria-unknown-Archaea-Eukaryota)
```
for Archaea:
```
remove.lineage(fasta=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.fasta, count=stability.trim.contigs.good.unique.good.filter.unique.precluster.denovo.vsearch.pick.count_table, taxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pds.wang.taxonomy, taxon=Chloroplast-Mitochondria-unknown-Bacteria-Eukaryota)
```
15. For each database, we then remove singleton sequences similar to the first step in UPARSE method (Prodan et al., 2020).
```
split.abund(fasta=remove.lineage.fasta, count=remove.lineage.count_table, cutoff=1)
```
16. And in preparation for clustering to OTUs which is distance-based greedy clustering, we obtain distance data based on sequence similarity, with default output = column-formatted matrix.
```
dist.seqs(fasta=remove.lineage.abund.fasta, cutoff=0.10)
```
17. In the clustering step, we specify our method which is average neighbor which is a middle ground between the other two algorithms.
```
cluster.split(column=remove.lineage.abund.dist, count=bacteria.abund.count_table, method=average, cutoff=0.03)
```
18. We then produce a shared file to know how many sequences are shared in each OTU. We specify that we're only interested in the 0.03 cutoff level.
```
make.shared(list=Bacteria.abund.an.list, count=Bacteria.abund.count_table, label=0.03)
```
19. Now the OTUs are ready to be classified based on the database we used earlier using the taxonomy file we produced with classify.seqs.
```
classify.otu(list=Bacteria.abund.an.list, count=Bacteria.abund.count_table, taxonomy=Bacteria.taxonomy, label=0.03)
```
20. We then pick representative sequences from each otu to generate a distance matrix for phylogenetic analysis. This command generates a fasta file containing only a representative sequence from each OTU.
```
get.oturep(column=Bacteria.abund.dist, list=Bacteria.abund.an.list, fasta=Bacteria.abund.fasta, count=Bacteria.abund.count_table, label=0.03, method=distance)
```
21. Before we generate the distance matrix, we reassign names in our fasta file with a python script borrowed from Haemophiluser of mothur forums.
```
from Bio import SeqIO          
import re                             
fasta = SeqIO.parse(handle='File.fasta', format='fasta')   
new_names = []                    
for seq in fasta:
print(seq.id)                     
print(seq.description)            
otu = re.search('(Otu[0-9]+)\|[0-9]+', seq.description)
print(otu.group(1))               
seq.id = otu.group(1)
new_names.append(seq)
SeqIO.write(sequences=new_names, format='fasta', handle='File_renamed.fasta')             
test = SeqIO.parse(handle='File_renamed.fasta', format='fasta')
for seq in test:
print(seq.id)
```
22. Finally, we are ready to generate the distance matrix.
```
dist.seqs(fasta=Archaea.0.03.rep.fasta, output=lt)
```
23. To generate a tree we use clearcut.
```
clearcut(phylip=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.phylip.dist)
```
