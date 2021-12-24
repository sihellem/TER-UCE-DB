 # /!\ UNDER CONSTRUCTION /!\ 
 
[![Generic badge](https://img.shields.io/badge/bioRxiv-10.1101/2021.12.09.472027-<COLOR>.svg)](https://doi.org/10.1101/2021.12.09.472027)
[![sihellem - TER-UCE-DB](https://img.shields.io/static/v1?label=sihellem&message=TER-UCE-DB&color=red&logo=github)](https://github.com/sihellem/TER-UCE-DB "Go to GitHub repo")
[![forks - TER-UCE-DB](https://img.shields.io/github/forks/sihellem/TER-UCE-DB?style=social)](https://github.com/oist/TER-UCE-DB?organization=oist&organization=oist)

# The Termite UCE Database

This database is maintained by the [Evolutionary Genomics Unit](https://groups.oist.jp/egu) of OIST.

Centralization of termite UCE data and assignation of unique identification code. We invite researchers to communicate us the metadata relating to samples from which UCE data were extracted using the bait set made available at (_link_pending_). This will ensure a long-term (re-)usability of all published data.

For any question or request, please open an issue.

## A/ Referenced contributions of UCE data
Listed contributions to the [Database](termite_uce_db_ids.tsv).
| Contribution  | Number of samples | Reference | Data location |
| --------  | ------------------- | --------------------- | ------------------- |
| #1 | 45 | Hellemans _et al_. [_bioRxiv_](https://doi.org/10.1101/2021.12.09.472027) | _Pending_ |

## B/ Methods and suggested database usage

_Dependencies_: [bioawk](https://github.com/lh3/bioawk), faToTwoBit from [BLAT](http://hgdownload.soe.ucsc.edu/admin/exe/), [phyluce](https://github.com/faircloth-lab/phyluce), [SeqKit](https://bioinf.shenwei.me/seqkit/usage/).

### B.1. Extraction of UCEs from newly-generated assemblies
Start by using the termites baits (_link_pending_) to extract UCE loci from your formatted assemblies.
```
### 1. Assembly formatting
## Rename the headers of contigs in your assemblies for processing by phyluce
bioawk -c fastx '{ print ">node_" ++i"\n"$seq }' < ${SAMPLE}_assembly.fasta > ${SAMPLE}_assembly_new_header.fasta

## Convert assemblies in 2bit format
faToTwoBit ${SAMPLE}_assembly_new_header.fasta ${SAMPLE}_assembly_new_header.2bit
```

Now use the following suite of commands on your assemblies from the official documentation of [phyluce](https://github.com/faircloth-lab/phyluce): [Finding UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-3.html#tutorial-iii-harvesting-uce-loci-from-genomes), followed by [Extracting UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-1.html#uceextraction). In short, the list of command will be the following, with the parameters used in the [_bioRxiv_](https://doi.org/10.1101/2021.12.09.472027) paper for full compatibility with deposited UCE data:
```
### 2. Finding UCEs
phyluce_probe_run_multiple_lastzs_sqlite --identity 50
phyluce_probe_slice_sequence_from_genomes --flank 200

### 3. Extracting UCEs
phyluce_assembly_match_contigs_to_probes --min-coverage 67
phyluce_assembly_get_match_counts
phyluce_assembly_get_fastas_from_match_counts --output my_own_termite_uce_data-incomplete.fasta
```
### B.2. Leveraging the database
Here, you typically want to add already published UCE data (outgroups, etc.) to your newly acquired data. You will want to select samples with a lot of loci in order to maximize the number of UCEs kept in the final supermatrices, based on %-completeness treshold. This assumes that contributions are made available as individual package, and samples labelled using the unique identification code (TER_X_UCEDB) as specified in the [Database](termite_uce_db_ids.tsv) file.
 
```
### 4. Generate the database
cat contrib_1.fasta ... contrib_n.fasta > database.fasta

### 5. Extract the required samples from the database
## Write the list of samples to a text file
cat <<__END__> ids.txt
TER_4_UCEDB
TER_12_UCEDB
TER_13_UCEDB
TER_14_UCEDB
__END__

## Extract the required samples with seqkit grep -nrif
seqkit grep --by-name --use-regexp --ignore-case --pattern-file ids.txt database.fasta > database_subset.fasta
```
### B.3. Alignments and generation of supermatrices
Now, it is time to align your newly-extracted UCE data in combination with the selected samples from the database. This mostly follows phyluce official docs: [Aligning UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-1.html#aligning-uce-loci)
```
### 6. Merge your own data with the database subset
cat my_own_termite_uce_data-incomplete.fasta database_subset.fasta > samples_to_align.fasta

### 7. Align the loci with internal trimming
phyluce_align_seqcap_align --fasta samples_to_align.fasta
phyluce_align_get_gblocks_trimmed_alignments_from_untrimmed
phyluce_align_remove_locus_name_from_nexus_lines

### 8. Construct your tresholded supermatrices
phyluce_align_get_only_loci_with_min_taxa
## Export as phylip or nexus
phyluce_align_format_nexus_files_for_raxml
phyluce_align_format_nexus_files_for_raxml --nexus
```
### B.4. What is next?
With your tresholded matrices, you are all set for phylogenetic inferences. Once you finished your analyses and submit your manuscript to a journal, it would be great to contact us with the metadata of your samples, such as [these](termite_uce_db_ids.tsv). We will then assign each sample to a unique identification code (TER_X_UCEDB).
_Contact us!_
<div align="center">
<a href="mailto:simon.hellemans@gmail.com?cc=xxx@xxx, xxx@yyy&subject=TER-UCE-DB
"><img src="https://img.shields.io/badge/gmail-%23DD0031.svg?&style=for-the-badge&logo=gmail&logoColor=white"/></a>
</div>

Replacing your sample code by the assigned TER_X_UCEDB ID in the UCE data you will submit with your paper (the file produced by `phyluce_assembly_get_fastas_from_match_counts`) will ensure an easy long-term (re-)usability of all published data.
```
### 9. Replace the original sample tags in your file by the assignated identification codes
## Create the correspondance list
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

## C/ Reference
Hellemans S, Wang M, Hasegawa N, Šobotník J, Scheffrahn RH, Bourguignon T. 2021. Using ultraconserved elements to reconstruct the termite tree of life. _bioRxiv_ [2021.12.09.472027](https://doi.org/10.1101/2021.12.09.472027)
