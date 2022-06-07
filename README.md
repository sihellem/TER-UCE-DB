[![Generic badge](https://img.shields.io/badge/MPE-10.1016/j.ympev.2022.107520-<COLOR>.svg)](https://doi.org/10.1016/j.ympev.2022.107520)
[![sihellem - TER-UCE-DB](https://img.shields.io/static/v1?label=sihellem&message=TER-UCE-DB&color=red&logo=github)](https://github.com/sihellem/TER-UCE-DB "Go to GitHub repo")
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
 
```
### 5. Generate the database
## Contribution #1
wget https://datadryad.org/stash/downloads/file_stream/1543952 --output-document=TER_UCE_DB_CONTRIB_1.fasta
## Contribution #2
wget https://datadryad.org/stash/downloads/file_stream/1427659 --output-document=TER_UCE_DB_CONTRIB_2.zip && unzip TER_UCE_DB_CONTRIB_2.zip && rm TER_UCE_DB_CONTRIB_2.zip && rm -fr __MACOSX
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
```
### B.3. Alignments and generation of supermatrices
Now, it is time to align your UCE data in combination with the selected samples from the database. This step mostly follows the phyluce official docs: [Aligning UCEs](https://phyluce.readthedocs.io/en/latest/tutorials/tutorial-1.html#aligning-uce-loci)
```
### 7. Merge your own data with the database subset
cat my_own_termite_uce_data-incomplete.fasta database_subset.fasta > samples_to_align.fasta

### 8. Align the loci with internal trimming
phyluce_align_seqcap_align --fasta samples_to_align.fasta
phyluce_align_get_gblocks_trimmed_alignments_from_untrimmed
phyluce_align_remove_locus_name_from_nexus_lines

### 9. Construct your tresholded supermatrix
phyluce_align_get_only_loci_with_min_taxa
## Export as phylip or nexus
phyluce_align_format_nexus_files_for_raxml
phyluce_align_format_nexus_files_for_raxml --nexus
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

## C/ How to cite
Hellemans S, Wang M, Hasegawa N, Šobotník J, Scheffrahn RH, Bourguignon T. 2022. Using ultraconserved elements to reconstruct the termite tree of life. _Molecular Phylogenetics and Evolution_ __173__. doi: [10.1016/j.ympev.2022.107520](https://doi.org/10.1016/j.ympev.2022.107520)

[Preprint on _bioRxiv_ [2021.12.09.472027](https://doi.org/10.1101/2021.12.09.472027)]
