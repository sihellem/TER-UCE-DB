[![sihellem - TER-UCE-DB](https://img.shields.io/static/v1?label=sihellem&message=TER-UCE-DB&color=red&logo=github)](https://github.com/sihellem/TER-UCE-DB "Go to GitHub repo")
[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC_BY--NC--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Generic badge](https://img.shields.io/badge/MPE-10.1016/j.ympev.2022.107520-<COLOR>.svg)](https://doi.org/10.1016/j.ympev.2022.107520)
[![forks - TER-UCE-DB](https://img.shields.io/github/forks/sihellem/TER-UCE-DB?style=social)](https://github.com/oist/TER-UCE-DB?organization=oist&organization=oist)

# The Termite UCE Database

This database is maintained by the [Evolutionary Genomics Unit](https://groups.oist.jp/egu) at OIST.

The purpose of this database is the centralization of termite UCE data and assignation of unique identification codes. We invite researchers to communicate to us the metadata related to samples from which UCE data were extracted using the bait sets made available on [_Dryad_](https://doi.org/10.5061/dryad.x0k6djhn0). This will ensure a long-term (re-)usability of all published data.

For any question or request, please open an issue.

## A/ Referenced contributions of UCE data
Listed contributions to the [Database](termite_uce_db_ids.tsv).
| Contribution  | Number of samples | Reference | Data location |
| --------  | ------------------- | --------------------- | ------------------- |
| #1 | 45 | Hellemans _et al_. [_MPE_](https://doi.org/10.1016/j.ympev.2022.107520) | [_Dryad_](https://doi.org/10.5061/dryad.x0k6djhn0) |
| #2 | 18 | Buček _et al_. [_MBE_](https://doi.org/10.1093/molbev/msac093) | [_Dryad_](https://doi.org/10.5061/dryad.5mkkwh77v) |
| #3 | 196 | Arora _et al_. [_Proc. B_](https://doi.org/10.1098/rspb.2023.0619) | [_Dryad_](https://doi.org/10.5061/dryad.tmpg4f53w) |
| #4 | 56 | Hellemans _et al_. | [_Dryad_](https://doi.org/10.5061/dryad.02v6wwqbm) |

## B/ Methods and suggested database usage

_Dependencies_: [bioawk](https://github.com/lh3/bioawk), faToTwoBit from [BLAT](http://hgdownload.soe.ucsc.edu/admin/exe/), [phyluce](https://github.com/faircloth-lab/phyluce), [SeqKit](https://bioinf.shenwei.me/seqkit/usage/).

### B.1. Extraction of UCEs from newly-generated assemblies
Note that two sets of baits were made available on the [_Dryad_](https://doi.org/10.5061/dryad.x0k6djhn0) repository. The first set targets all 50,616 UCE loci ([_Dryad_](https://doi.org/10.5061/dryad.x0k6djhn0): Supplementary Data 5), while the second set targets a reduced set of 5,934 UCE loci, each present for at least 75% of samples ([_Dryad_](https://doi.org/10.5061/dryad.x0k6djhn0): Supplementary Data 8). The first set is intended to be used for _in silico_ mining approach, while the second could be synthetized for the more traditional hybridization approach.

Start by using the termites baits to extract UCE loci from your formatted assemblies.

```
### 1. Get the baits
## Dryad: Supplementary Data 5
wget https://datadryad.org/stash/downloads/file_stream/1543950 --output-document=termite_uce_baits.fasta

### 2. Assembly formatting
## Rename the headers of contigs in your assemblies for phyluce compatibility
bioawk -c fastx '{ print ">node_" ++i"\n"$seq }' < ${SAMPLE}_assembly.fasta > ${SAMPLE}_assembly_new_header.fasta
## Convert assemblies in 2bit format
faToTwoBit ${SAMPLE}_assembly_new_header.fasta ${SAMPLE}_assembly_new_header.2bit
```

Now, use the following suite of commands on your assemblies from the official documentation of [phyluce](https://github.com/faircloth-lab/phyluce): [Finding UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-3.html#tutorial-iii-harvesting-uce-loci-from-genomes), followed by [Extracting UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-1.html#uceextraction). In short, the list of commands will be the following (with the parameters used in the [_MPE_](https://doi.org/10.1016/j.ympev.2022.107520) paper) for full compatibility with the deposited UCE data:
```
### 3. Finding UCEs
phyluce_probe_run_multiple_lastzs_sqlite --identity 50
phyluce_probe_slice_sequence_from_genomes --flank 200

### 4. Extracting UCEs
phyluce_assembly_match_contigs_to_probes --min-coverage 67
phyluce_assembly_get_match_counts
phyluce_assembly_get_fastas_from_match_counts --output my_own_termite_uce_data-incomplete.fasta
```
### B.2. Leveraging the database
At this point, you typically want to add already published UCE data (outgroups, etc.) to supplement your dataset. Think about selecting samples for which a lot of loci are available to maximize the number of UCEs kept in your final supermatrix, based on %-completeness threshold. This assumes that contributions are made available as an individual package with samples labelled using the unique identification code (TER_X_UCEDB) as specified in the  [Database](termite_uce_db_ids.tsv) file.

Since 2024, command-line direct download is prevented for non-browser access. You can either use the links of each contribution and download them one-by-one using your favorite internet navigator. Datasets can still be accessed by mimicking browser access through ```--user-agent``` option of ```wget``` (e.g., ```wget --user-agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)"``` as suggested on this StackExchange [post](https://unix.stackexchange.com/questions/158352/curl-wget-403-forbidden). Note that ```--user-agent``` needs to be changed after each use.
For a more permanent solution, you can install a random user-agent generator, such as [randomua](https://github.com/picatz/randomua) written in ```ruby``` (to be installed with the command: ```gem install randomua```).

Using the above user-agent generator, the database can be generated by the following snippets:
```
### 5. Generate the database
## Contribution #1
wget --user-agent="$(randomua -d)" https://datadryad.org/stash/downloads/file_stream/1543952 --output-document=TER_UCE_DB_CONTRIB_1.fasta
## Contribution #2
wget --user-agent="$(randomua -d)" https://datadryad.org/stash/downloads/file_stream/1427659 --output-document=TER_UCE_DB_CONTRIB_2.zip && unzip TER_UCE_DB_CONTRIB_2.zip && rm TER_UCE_DB_CONTRIB_2.zip && rm -fr __MACOSX
## Contribution #3
wget --user-agent="$(randomua -d)" https://datadryad.org/stash/downloads/file_stream/2255906 --output-document=TER_UCE_DB_CONTRIB_3.tar.gz && tar -xvf TER_UCE_DB_CONTRIB_3.tar.gz && rm TER_UCE_DB_CONTRIB_3.tar.gz
## Contribution #4
wget --user-agent="$(randomua -d)" https://datadryad.org/stash/downloads/file_stream/3273623 --output-document=TER_UCE_DB_CONTRIB_4.fasta.gz && gzip -d TER_UCE_DB_CONTRIB_4.fasta.gz
## Combine fastas
cat TER_UCE_DB_CONTRIB_1.fasta TER_UCE_DB_CONTRIB_2.fasta ... TER_UCE_DB_CONTRIB_N.fasta > database.fasta

### 6. Extract the samples you want from the database
## Write the list of samples in a text file
cat <<__END__> ids.txt
TER_4_UCEDB
TER_12_UCEDB
TER_13_UCEDB
TER_14_UCEDB
__END__

## Extract the selected samples with seqkit grep -nrif
seqkit grep --by-name --use-regexp --ignore-case --pattern-file ids.txt database.fasta > database_subset.fasta

## Get the corresponding ID-species ids
wget https://raw.githubusercontent.com/sihellem/TER-UCE-DB/main/termite_uce_db_ids.tsv
```
### B.3. Alignments and generation of supermatrices
Now, it is time to align your UCE data in combination with the selected samples from the database. This step mostly follows the phyluce official docs: [Aligning UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-1.html#aligning-uce-loci)
```
### 7. Merge your own data with the database subset
cat my_own_termite_uce_data-incomplete.fasta database_subset.fasta > samples_to_align.fasta

### OPTIONAL: (re-)create separated locus or taxon files from the samples_to_align.fasta
### This can be useful for subsetting purposes, or parrallelizing the alignment of loci
## OPT.1: Exploding the monolithic fasta into one locus per file
phyluce_assembly_explode_get_fastas_file --input samples_to_align.fasta --output exploded-loci
## OPT.2: Exploding the monolithic fasta into one sample per file
phyluce_assembly_explode_get_fastas_file --by-taxon --input samples_to_align.fasta --output exploded-taxa
### NOTE: if process stops and does not write fasta files, you may need to break the samples_to_align.fasta into two parts (or more)

### 8. Align the loci with internal trimming
phyluce_align_seqcap_align --fasta samples_to_align.fasta
phyluce_align_get_gblocks_trimmed_alignments_from_untrimmed
phyluce_align_remove_locus_name_from_nexus_lines

### 9. Construct your tresholded supermatrix
phyluce_align_get_only_loci_with_min_taxa
## Export as phylip or nexus
phyluce_align_format_nexus_files_for_raxml
phyluce_align_format_nexus_files_for_raxml --nexus
## You might want to export your alignment in .fasta for conveniently run phylogenies with FastTree.
# For this, use the .phy file produce by 'phyluce_align_format_nexus_files_for_raxml', and:
sed '1d' aln.phylip > aln.fasta
sed -i 's/^/>/' aln.fasta && sed -i "s/ /\\n/g" aln.fasta
```
### B.4. What is next?
With your tresholded matrices, you are all set for phylogenetic inferences. Once you finished your analyses and submit your manuscript to a journal, please contact us with the metadata of your samples, such as [these](termite_uce_db_ids.tsv). We will then assign each sample to a unique identification code (TER_X_UCEDB).
_Contact us!_
<div align="center">
<a href="mailto:simon.hellemans@gmail.com?cc=Thomas.Bourguignon@oist.jp, Simon.Hellemans@oist.jp&subject=GitHub: TER-UCE-DB
"><img src="https://img.shields.io/badge/gmail-%23DD0031.svg?&style=for-the-badge&logo=gmail&logoColor=white"/></a>
</div>

Replacing your sample code by the assigned TER_X_UCEDB ID in the UCE data you will submit with your paper (the file produced with `phyluce_assembly_get_fastas_from_match_counts`) will ensure an easy long-term (re-)usability of all published data.
```
### 10. Replace the original sample tags in your file with the assignated identification codes
## Create the correspondence list
TAB="$(printf '\t')"
cat <<__END__> db_ids.txt
Termitus_uceensis${TAB}TER_AA_UCEDB
Neotermitus_uceensis${TAB}TER_BB_UCEDB
__END__

## Name replacement
cp my_own_termite_uce_data-incomplete.fasta my_own_termite_uce_data-incomplete_renamed.fasta
while read i; do
	a=$(echo "$i" | cut -d$'\t' -f1)
	b=$(echo "$i" | cut -d$'\t' -f2)
	sed -i "s/$a/$b/g" my_own_termite_uce_data-incomplete_renamed.fasta
done < db_ids.txt
```

## C/ Diagnostic tool for subfamily determination within Termitidae
The current version of the diagnostic tool for the subfamilies of Termitidae is available in the [_Dryad_](https://doi.org/10.5061/dryad.02v6wwqbm) repository. For the list of UCEs covered by this tool, please refer to the Supplements published in _Hellemans et al_.
Using this tool requires the command line version of [_NCBI-BLAST_](https://www.ncbi.nlm.nih.gov/books/NBK569861/).

_NB_: The current tool is not diagnostic for subfamilies represented by only one sample (i.e., Crepititermitinae, Forficulitermitinae, and Protohamitermitinae) in the database.
```
## Downloading the current diagnostic database
wget --user-agent="$(randomua -d)" https://datadryad.org/stash/downloads/file_stream/3273621 --output-document=termitidae_diagnosing_database_v1.fasta.gz && gzip -d termitidae_diagnosing_database_v1.fasta.gz

## Get list of UCEs covered by the tool
# keep the underscore to ensure unique match
grep ">" termitidae_diagnosing_database_v1.fasta | awk -F'_' '{print $1}' | sed 's/>//g' | uniq | sed 's/$/_/g' > tool_ids_uniq.txt

## Extract the UCEs from your samples from the covered list (e.g., from database_subset.fasta file above)
seqkit grep --by-name --use-regexp --ignore-case --pattern-file tool_ids_uniq.txt database_subset.fasta > subset_to_blast.fasta

## Converting the tool to database format
makeblastdb -in termitidae_diagnosing_database_v1.fasta -dbtype nucl -parse_seqids

## Using the tool on your database_subset_to_blast.fasta restricted to covered UCEs
blastn -task megablast -db termitidae_diagnosing_database_v1.fasta -query subset_to_blast.fasta -out blasted_uces.txt -outfmt '6 qseqid sseqid bitscore evalue length qlen qcovs pident' -max_hsps 1 -max_target_seqs 1
```

## D/ How to cite
Hellemans S, Wang M, Hasegawa N, Šobotník J, Scheffrahn RH, Bourguignon T. 2022. Using ultraconserved elements to reconstruct the termite tree of life. _Molecular Phylogenetics and Evolution_ __173__: 107520. doi: [10.1016/j.ympev.2022.107520](https://doi.org/10.1016/j.ympev.2022.107520)

[Preprint on _bioRxiv_ [2021.12.09.472027](https://doi.org/10.1101/2021.12.09.472027)]
