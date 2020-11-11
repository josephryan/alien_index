<a href="https://zenodo.org/badge/latestdoi/15686/josephryan/alien_index"><img src="https://zenodo.org/badge/15686/josephryan/alien_index.svg" alt="10.5281/zenodo.21029"></a>

Citation: Ryan, JF, 2014. Alien Index: identify potential non-animal transcripts or horizontally transferred genes in animal transcriptomes. DOI:http://dx.doi.org/10.5281/zenodo.21029

alien_index
===========

identify potential contaminants or horizontally transferred genes in transcriptomes.

install
=====
To install `alien_index` and documentation, type the following:

    perl Makefile.PL
    make
    sudo make install

To install without root privelages try:

    perl Makefile.PL PREFIX=/home/myuser/scripts
    make
    make install

prerequisite
============

    BLAST+  (BLAST+ programs are not used directly by alien_index, but you will need it to create input files to be processed by alien_index)

    JFR-PerlModules (https://github.com/josephryan/JFR-PerlModules) 
    only needed for the remove_aliens script (not for alien_index)

howto guide
=====

1. Create a FASTA database of non-alien sequences (e.g., representative animal protein sequences) (NOTE: nucleotide sequences would work too)
    * an example database (meta.fa.gz) is available here: http://ryanlab.whitney.ufl.edu/downloads/alien_index/

2. Create a FASTA database of alien sequences (e.g., representative non-animal protein sequences; e.g., bacteria, non-metazoan euks, archae)
    * an example database (non_meta.fa.gz) is available here: http://ryanlab.whitney.ufl.edu/downloads/alien_index/

3. Add unique code to definition line of all alien sequences. For
       example:

    perl -pi -e 's/^>/>ALIEN_/' non_meta.fa

4. Make sure that your unique code is not found in your non-alien fasta

    grep '^>' meta.fa | grep ALIEN_

5. create BLAST database. For example:

    cat non_meta.fa meta.fa > ai.fa
    
    makeblastdb -dbtype prot -in ai.fa

6. BLAST query sequences (e.g., transcriptome) against combined db. Be sure to use -outfmt 6 (tabbed). For example:

    blastx -query myseqs.fa -db ai.fa -outfmt 6 -seg yes -evalue 0.001 -out myseqs_v_ai.blastx

7. Run alien_index

    alien_index --blast=myseqs_v_ai.blastx --alien_pattern=ALIEN_ > myseqs.alien_index

8. OPTIONAL: If you want to create a FASTA file with all sequences that have an entry in your alien_index output file removed, run the following:

    remove_aliens myseqs.alien_index myseqs.fa > myseqs_without_aliens.fa

algorithm
=========

algorithm is based on alogorithm described in the following:
  Gladyshev, Eugene A., Matthew Meselson, and Irina R. Arkhipova.
    "Massive horizontal gene transfer in bdelloid rotifers."
    science 320.5880 (2008): 1210-1213.

briefly: a transcriptome is BLASTed against proteomes from various metazoans and non-metazoans.  If the best hit is non-metazoan the e-value of the best hit is compared to the best metazoan hit and the difference is reported.  

Gladyshev et al (2008) was describing horizontally transferred genes. Here is the relevant quotation from the paper describing when a gene was reliably considered having a non-metazoan origin:

> We characterized the foreignness of such genes with an alien index (AI), 
> which measures by how many orders of magnitude the BLAST E-value for the
> best metazoan hit differs from that for the best nonmetazoan hit (Table 1
> and table S1) (19). We tested the AI validity by phylogenetic analyses of
> all CDS with AI > 0 yielding metazoan hits, excluding those with repetitive
> sequences. We found that AI ≥ 45, corresponding to a difference of ≥20 orders
> of magnitude between the best nonmetazoan and metazoan hits, was a good
> indicator of foreign origin, as judged by phylogenetic assignment to
> bacterial, plant, or fungal clades (Fig. 2 and fig. S2A). Genes with
> 0 < AI < 45 were designated indeterminate, because their phylogenetic
> analysis may or may not confidently support a foreign origin."

And here is the description of the score itself described in the supplement of the same paper:

> Alien Index (AI) as given by the formula AI = log((Best E-value for Metazoa)
> \+ e-200) - log((Best E-value for Non- Metazoa) + e-200). If no hits were
> found, the E-value was set to 1. Thus, AI is allowed to vary in the interval
> between +460 and -460, being positive when top non-metazoan hits yielded
> better E-values than the top metazoan ones. Entries with incomplete taxonomy,
> such as X-ray structures, or hits belonging to the same phylum as the query
> sequence (i.e, Rotifera, Arthropoda or Nematoda), were excluded from the
> analysis. The top four BLASTP hits were used to assign the consensus
> taxonomic placement of each query sequence. Tables S1 and S3 summarize the
> results of BLASTP searches. Based on AI, genes were classified as foreign
> (AI ≥ 45), indeterminate (0<AI<45), or metazoan (AI ≤ 0). All sequences
> coding for proteins composed of multiple repeated units (such as TPR, NHL,
> FG-GAP, and Kelch repeats, found in all kingdoms) with AI ≥ 45 were
> re-assigned to the indeterminate category, because their relative BLASTP
> scores can vary significantly when minor changes in each repeat unit
> propagate throughout the entire protein length.


# Embedded Documentation (POD2MARKDOWN)

# NAME

**alien\_index** - identify non-animal sequence in animal transcriptomes

# AUTHOR

Joseph F. Ryan <joseph.ryan@whitney.ufl.edu>

# SYNOPSIS

alien\_index --blast=BLASTREPORT --alien\_pattern=PATTERN\_IN\_ALIEN\_DEFLINES \[--long\_output\] \[--version\] \[--help\]

# OPTIONS

- **--blast**

    BLAST report run with -outfmt 6 

- **--alien\_pattern**

    a pattern that appears in 

- **--long\_output**

    output will contain all blast info from the best alien BLAST high scoring pair (blast tab output begins with A e.g. "Aevalue") and best non-alien BLAST high scoring pair (blast tab output begins with NA e.g. "NAqstart")
     1.	 qseqid 	 query (e.g., unknown gene) sequence id
     2. 	 sseqid 	 subject (e.g., reference genome) sequence id
     3. 	 pident 	 percentage of identical matches
     4. 	 length 	 alignment length (sequence overlap)
     5. 	 mismatch 	 number of mismatches
     6. 	 gapopen 	 number of gap openings
     7. 	 qstart 	 start of alignment in query
     8. 	 qend 	 end of alignment in query
     9. 	 sstart 	 start of alignment in subject
     10. 	 send 	 end of alignment in subject
     11. 	 evalue 	 expect value
     12. 	 bitscore 	 bit score

- **--help**

    Print this manual

- **--version**

    Print the version. Overrides all other options.

# DESCRIPTION

Generates a measure (alien index) which measures by how many orders of magnitude the BLAST E-value for the best non-alien hit differs from that for the best alien hit.  Examples of non-alien and alien are: (metazoan and non-metazoan; plant and non-plant; non-nematodes and nematodes--if for example you expected nematode contamination).

# STEP-BY-STEP

1\. Create a FASTA database of non-alien sequences (e.g., representative animal protein sequences) (NOTE: nucleotide sequences would work too)
    \* an example database (meta.fa.gz) is available here: http://ryanlab.whitney.ufl.edu/downloads/alien\_index/

2\. Create a FASTA database of alien sequences (e.g., representative non-animal protein sequences; e.g., bacteria, non-metazoan euks, archae)
    \* an example database (non\_meta.fa.gz) is available here: http://ryanlab.whitney.ufl.edu/downloads/alien\_index/

3\. Add unique code to definition line of all alien sequences. For
       example:

    perl -pi -e 's/^>/>ALIEN_/' non_meta.fa

4\. Make sure that your unique code is not found in your non-alien fasta

    grep '^>' meta.fa | grep ALIEN_

5\. create BLAST database. For example:

    cat non_meta.fa meta.fa > ai.fa
    
    makeblastdb -dbtype prot -in ai.fa

6\. BLAST query sequences (e.g., transcriptome) against combined db. Be sure to use -outfmt 6 (tabbed). For example:

    blastx -query myseqs.fa -db ai.fa -outfmt 6 -max_target_seqs 1000 -seg yes -evalue 0.001 -out myseqs_v_ai.blastx

7\. Run alien\_index

    alien_index --blast=myseqs_v_ai.blastx --alien_pattern=ALIEN_ > myseqs.alien_index

# COPYRIGHT

Copyright (C) 2016-2020, Joseph F. Ryan

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see &lt;http://www.gnu.org/licenses/>.
