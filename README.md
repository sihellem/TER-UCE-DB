# The Termite UCE Database
This database is maintained by the [Evolutionary Genomics Unit](https://groups.oist.jp/egu) of OIST.

Centralization of termite UCE data and assignation of unique identification code. We invite researchers to communicate us the metadata relating to samples from which UCE data were extracted using the bait set made available at (_link_pending_). This will ensure a long-term (re-)usability of all published data.

For any question or request, please open an issue.

## Referenced contributions of UCE data
Listed contributions to the [Database](termite_uce_db_ids.tsv).
| Contribution  | Number of samples | Original reference |
| --------  | ------------------- | --------------------- |
| #1 | 45 | Hellemans _et al_. |

## Methods and suggested usage
[phyluce](https://github.com/faircloth-lab/phyluce) was used to design the bait set and to extract UCEs.

Assuming that all contributions are made available as individual package, and samples labelled using the unique identification code (TER_X_UCEDB), wanted samples can easily be extracted with [SeqKit](https://bioinf.shenwei.me/seqkit/usage/) to be incorporated in further phylogenies.

```
### First combine data contributions together
cat contrib_1.fasta ... contrib_n.fasta > database.fasta

### Then extract the required samples
seqkit grep -f id.txt database.fasta > extracted_samples.fasta
```

## Reference
Hellemans S, Wang M, Hasegawa N, Šobotník J, Scheffrahn RH, Bourguignon T. 2021. Using ultraconserved elements to reconstruct the termite tree of life. _bioRxiv_ [2021.12.09.472027](https://doi.org/10.1101/2021.12.09.472027)
