# LightMetaEbwt 

LightMetaEbwt is a novel lightweight alignment-free and assembly-free framework for metagenomic classification that is combinatorial by nature and allows us to use little internal memory. In [1], a [preliminary version of LightMetaEbwt](https://github.com/veronicaguerrini/LightMetaEbwt_Alcob) has been introduced together with a new sequence similarity measure based on the properties of the extended Burrows–Wheeler transform. We now implement two variants of the above similarity measure and improve the overall classification. In addition, the new version of our tool allows to use multiple processors/cores.

Let *S* be a large collection of biological sequences comprising both reads and genomes. For simplicity, we denote by *R* the subset of reads (metagenomic sample) and by *G* the subset of genomes (reference database).

It takes in input:
- the extended Burrows–Wheeler transform (ebwt), or multi-string BWT, of collection *S*;
- the longest common prefix array (lcp) of collection *S*;
- the document array (da) of collection *S*.

By calculating a similarity degree between any read in *R* and any genome in *G*, LightMetaEbwt performs the metagenomic classification by assigning any read to a unique taxon of belonging. It is able to classify the reads to several taxonomic levels such as genomes, species or phylum.

The underlying method can be summarized in three main steps: 

1. By reading da(*S*) and lcp(*S*), we detect *alpha*-clusters, *i.e.*, blocks of of ebwt(*S*) containing symbols belonging both to *R* and to *G* and whose associated suffixes share a common context of a minimum length *alpha*; 

2. We analyze *alpha*-clusters in order to evaluate a degree of similarity between any read and any genome in *S* by using two different approaches: (a) by reading both da(*S*) and ebwt(*S*), and (b) by reading only da(*S*).

3. We perform the read assignment: every read in *R* either is assigned to a particular taxon, or it is reported as not classified if the maximum similarity score achieved by *R* is lower than *beta* (threshold value).

The above strategy is suitable for classifying reads belonging to a single read collection or a paired-end read collection. For paired-end read collections, the procedures at Step 1. and Step 2. have to be repeated for the data structures of both strands (forward and reverse complement) of both paired-end reads.

### Preprocessing step

There are mainly two options to obtain the required data structures (ebwt, lcp, da) for *S*:
- one could build ebwt, lcp, and da starting from a fasta file containing both the reads in *R* and the genomes in *G*;
- or, one could build the data structures ebwt, lcp, and da separately for the set *G* and the set *R*, and then merge them to obtain ebwt, lcp, and da for the entire collection *S*.

The advantage of the latter choice lies on building the data structures of the set *G* of genomes only once if *G* is the same for each experiment.

To build ebwt, lcp, and da files from scratch from a single fasta file, one could use BCR [https://github.com/giovannarosone/BCR_LCP_GSA], or egsa [https://github.com/felipelouza/egsa] for instance. Note that egsa tool returns the three datastructures in a single file (fastaFile.K.gesa, with K being the number of sequences). The executable file EGSAtoBCR is to convert fastaFile.K.gesa into fileFasta.ebwt, fileFasta.lcp, and fileFasta.da -- use command EGSAtoBCR filefasta K.

To merge the two ebwts associated with the sets *R* and *G*, one could use eGap [https://github.com/felipelouza/egap] and set both options *--lcp* and *--da* to have the lcp and da computed. Note that, the running time of eGap can be decreased if a certain threshold *k* is settled for computing the lcp values. Indeed, by using the option *--trlcp k*, as an altenative to *--lcp*, eGap stops merging the two ebwts when the lcp value calculated is *k*, and computes a lcp array in which all lcp values greater than *k* are replaced by the value *k*.

On the other hand, exploiting the mathematical properties of the permutation associated with the
ebwt and lcp, one could use BCR [https://github.com/giovannarosone/BCR_LCP_GSA] *incrementally* in order to update the data structures obtained for *G* with th symbols in *R* and to obtain the data structures for the collection *S* (without constructing the eBWT and lcp for *R* from scratch) .

### Install

```sh
git clone https://github.com/veronicaguerrini/LightMetaEbwt
cd LightMetaEbwt
```
### Compile
One can choose between the two strategies of Step 2. by setting parameter EBWT: 

for the first approach (a) set EBWT=1 (default)

```sh
make
```
while for running the second approach (b) set EBWT=0

```sh
make EBWT=0
```
Moreover, LightMetaEbwt allows to classify reads at any rank between species and phylum (without specifying a fixed taxonimic level) by taking advantage of taxonomic lineages of taxa using the option HIGHER=1 (default HIGHER=0).

```sh
make HIGHER=1
```
### Run

The three steps are accomplished by running:

(1) ClusterLCP with input parameters: name of the fasta file, total number of reads in *S*, total number of genomes in *S*, *alpha* and number of threads;

(2) ClusterBWT_DA with input parameters: name of the fasta file, length of the reads, *beta* and number of threads;

Recall that in order to run ClusterLCP we need to have the two datastructures fileFasta.lcp and fileFasta.da computed, while to run ClusterBWT_DA we need only fileFasta.da if EBWT=0, both fileFasta.da and fileFasta.ebwt if EBWT=1.

(3) Classify with input parameters: number of files, files obtained by running ClusterBWT_DA, total number of reads in *S*, total number of genomes in *S*, OutputFile, LineageFile, TaxRank and number of threads.

OutputFile is the ','-separated file where the classification results will be stored according to the format:

C/U/A/H,IdSeqRead,TaxID,maxSim

where C=Classified, U=not classified, A=Ambiguous, H=classified at higher ranks (if HIGHER=1).

LineageFile is a ';'-separated file where genome taxonomy information are stored according to the format:

Seq_ID;Phylum_TaxID;Class_TaxID;Order_TaxID;Family_TaxID;Genus_TaxID;Species_TaxID.

TaxRank is an integer in the range [1,6] that stands for any classification level between phylum(=1) and species(=6).


Alternatively, one could run LightMetaEbwt_single.sh (for single read collections) or LightMetaEbwt_paired.sh (for paired-end read collections) that use default values *alpha*=16 and *beta*=0.25.



### Example

In Datasets, we provide some examples of simulated metagenomic samples. (See for details Datasets/Experiments_links.txt).

The dataset setB2 is a sample of 20,249,373 paired end short reads (100 bps) stored in setB2_1.fasta and setB2_2.fasta.

As preprocessing, we construct the datastructures ebwt, lcp, and da for the set *G* -- Refs.fasta is the fasta file of reference genomes (number of genomes: 930). 
Then, we merge the datastructures (ebwt, lcp, da) associated with Refs.fasta to those associated with the sets of reads (setB2_1.fasta and setB2_2.fasta, and setB2_1_RC.fasta and setB2_2_RC.fasta, which contain the reverse complement of setB2_1.fasta and setB2_2.fasta) as to obtain the datastructures for the four collections: 
setB2_1+Refs.fasta, setB2_1_RC+Refs.fasta, setB2_2+Refs.fasta, setB2_2_RC+Refs.fasta.

## References

[1] V. Guerrini and G. Rosone. Lightweight Metagenomic Classification via eBWT. Alcob 2019. LNCS, vol 11488, pp 112-124.

## Thanks

<small> Supported by the project Italian MIUR-SIR [CMACBioSeq][240fb5f5] ("_Combinatorial methods for analysis and compression of biological sequences_") grant n.~RBSI146R5L. P.I. Giovanna Rosone</small>

[240fb5f5]: http://pages.di.unipi.it/rosone/CMACBioSeq.html
