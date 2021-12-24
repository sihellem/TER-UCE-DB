 # /!\ UNDER CONSTRUCTION /!\ 

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
[phyluce](https://github.com/faircloth-lab/phyluce) was used to design the bait set and to extract UCEs.

Assuming that all contributions are made available as individual package, and samples labelled using the unique identification code (TER_X_UCEDB), selected samples can easily be extracted with [SeqKit](https://bioinf.shenwei.me/seqkit/usage/) to be incorporated in further phylogenies.

### B.1/ Sample selection
Here, you typically want to add already published UCE data (outgroups, etc.) to your newly acquired data. You will want to select samples with a lot of loci in order to maximize the number of UCEs kept in the final supermatrices, based on %-completeness treshold.
 
```
### Generate the database
cat contrib_1.fasta ... contrib_n.fasta > database.fasta

### Samples to extract from the database
cat <<__END__> ids.txt
TER_4_UCEDB
TER_12_UCEDB
TER_13_UCEDB
TER_14_UCEDB
__END__

### Extract the required samples with seqkit grep -nrif
seqkit grep --by-name --use-regexp --ignore-case --pattern-file ids.txt database.fasta > database_subset.fasta
```
### B.2/ Alignments and generation of supermatrices
```
### Merge your own data with the database subset
cat my_own_data.fasta database_subset.fasta > samples_to_align.fasta
```

## C/ Reference
Hellemans S, Wang M, Hasegawa N, Šobotník J, Scheffrahn RH, Bourguignon T. 2021. Using ultraconserved elements to reconstruct the termite tree of life. _bioRxiv_ [2021.12.09.472027](https://doi.org/10.1101/2021.12.09.472027)
