# A distinct CRC module drives oncogenes in KMT2A-r leukemia
            
Jérémie Kalfon - CDS meeting - 02/02

---

## Recap 📖


### enhancer and TFs

![enhancers](images/enhancers.png)

  ➡ enhancers drive expression <!-- .element: class="fragment" data-fragment-index="1" -->


### super enhancers & TADs.

<div class="r-stack">
  <img src="images/TAD.png">
  <img class="fragment" src="images/TAD_HIC.png">
</div>


### Core Regulatory Circuitry & cell state.

![CRC](images/CRC.png)


![cell state](images/cell_state.png)

---

## What is the link with AML and dependencies 🤔.


### Many effective drugs in AML and Cancer target TFs.

- [BET protein inhibitor JQ1 downregulates chromatin accessibility and suppresses metastasis of gastric cancer via inactivating RUNX2/NID1 signaling](https://www.nature.com/articles/s41389-020-0218-z)
- [Mission Possible: Advances in MYC Therapeutic Targeting in Cancer](https://link.springer.com/article/10.1007/s40259-019-00370-5) <!-- .element: class="fragment" data-fragment-index="1" -->


![cancer stem cell](images/csc.png)

1. There is the famous theory of **cancer** as being in a **stem cell state** & reusing these stem cell states (immortality, fast division, proliferation) to become cancerous.
2. We know that we can create and revert stem cell states (Yamanaka Factors) by changing the epigenetics of cells. <!-- .element: class="fragment" data-fragment-index="1" -->

---

### What we have:


###  A set of CRCs:

![CRC mapper plot](images/CRC.png)

CRCmapper: from gene expression + TF motifs + super enhancer bedfile + gene location, will compute the CRC.


> Usually this is done with CRCmapper but the tool is not giving all CRC members based on their definition of CRC and what we know about that cell line.


<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->

![CRC max plot](images/maxlist.png) <!-- .img: class="r-stretch" data-fragment-index="0" -->


<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->

![meme of pile of data](images/pile_data.png) 


- H3K27ac chipseq of a 140 PDXs
- \~250 chipseq on MV411
- many replicates,
- good/bad qualities,
- different seq methods, different labs (anything I could find on SRA)


<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->

<iframe allowfullscreen data-src="https://docs.google.com/spreadsheets/d/1yFLjYB1McU530JnLgL0QIMAKIkVl3kl0_LCHje2gk8U#gid=738732237" style="width:100%; height:545px"></iframe>


### What I am using 👷:


![nfcore image](images/nfcore.png)


- Nextflow + GCP + NFcore ( + good file management 😅 )
- Bedtools + DeepTools + A package I built  with many epigenetic plot/helper functions <!-- .element: class="fragment" data-fragment-index="1" -->

---

### First step:


<iframe data-src="images/Chip_res_multiqc_multiqc_report_pe.html" style="width:100%; height:545px"></iframe>


  - Frip score, read counts
  - Look at igv signal and found peaks 🔍


### Second step:

##### A tool for merging chipseq replicate peakfiles.
 
  - Computes which one is the best
    - from user defined annotations of quality
    - from overlap within replicates
  - Computes which ones are too bad to even be used
  - Plots important metrics: venn overlap, correlation.


<!--- ![plot of IGV with results + signal](images/igv.png) -->
<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->

<iframe allowfullscreen src="https://igv.org/web/release/2.7.4/embed.html?sessionURL=blob:tVTBbts6EPyVgKcWkEjJki3Lx5dDUjQFCgNtDkURUNKKYiORKknZcQz_e5ayA7.D6wRNahg.7Ix3hkPubskKjJVakQWZ0IymJCAGajCgSiCLLZEVIo1I5ggo3mGNXA8dVxcfrpaXTTJnHvuIYM2t49.WN57uXG8XjNmEVgNvsV7e08GGgJQwprzjj1rxtaWl7pgUK1oYzSuprJNucEC1EUyA0h1YZuH3KDH.0JqjklQVPPx7JfxKVCs3ThdcVX8piAr0WWHszpXSjjuM3DLf.z_s_akCTd2Do.KR7ALS6nKwKFY2JlvMpkEWzYMkn4WzWRDNJ0GaTtGXMyiNrB9b4ja9vxc8wDBeW0C0qcCQRZhHURbn.WSaZmmU5_Eu2JLBtEgW_hS8a3ujf0Hp2GUje1bqAsOVSrBizVkHRkB1IwvDzYYVUqylYF2fReGX72kch9fJ50nGy7tlTBG8leL4RF5g1dp03CFv3_RoOH2DwY6Xlo33.xX4_Wmjdz1Cdv8KPOsFxyfoR.u.6PGj..w94o2T2WvyPU_7Y8Dz9wz4lIdzCb.Sfy7i3Ps_NFxCjU_.4goU2P__DfcXTpwfg_1B3z6zqiwkqnml5zE9rCHA9VhjTwjISlpZyFa6zS1Ceo3jF_tl2ukVL1o0fOAdThJH4ycgDUjRoG8s7H7ungA-" style="width:100%; height:545px"></iframe>



![image from igv showing peaks overlapping ](images/nonoverlap_igv.png)
<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->


<img class="r-stretch" src="images/CEBPA_before_venn_venn.pdf.png"> 

Confirms peaks under region where some replicates have peaks but not other.
Using an algorithm similar to MACS2.


We recover a peak if the distance between the poisson distribution of peak region and poisson from the full chromosome is above a certain cutoff.

<img class="r-stretch" src="images/CEBPA_before_pairplot.pdf.png">

We get all peaks found in at least 2 replicates or the "main/best" replicate.

---

## Making a cobinding matrix.

> Peaks are stored in to a bed file (similar to what our segment cn file is)

--> We are merging them together into a conscensus peak.

_Notes: Changing the window size does not seem to drastically change the results._


<!-- .slide: data-background-color="rgb(255, 255, 255)"> -->

![plot of the sorted cobinding matrix](images/v3_remove_single_150_clustermap_cobinding_scaled_full_annotation_sorted.pdf.png)


Now  we have a matrix of region bound by CRC TFs:
- How they co-bind, 
- with what intensity and
- what these regions are enriched for.


### Making gene - Enhancer assignments.

<!-- ![ABC model plot](images/ABCmodel.png) -->
<img class="r-stretch" src="images/ABCmodel.png">

ABC model: computes gene<->enhancer mapping from gene expression, H3K27ac ChIP-seq & HiC-seq.

---

## Adding more features.

Here I will present a couple of tools that I have used or I am planning to use to enrich the dataset with additional features.


- ROSE
- ChromHMM
- MEME


- ROSE: computes super enhancer by stitching nearby enhancers and computing which ones are above a certain threshold. (you might have seen this feature already in some plots).

![igv](images/igv.png)


<!-- ![Rose threshold plot](images/rose_ranking.png) -->

<img src="images/rose_ranking.png">


![chromHMM plot](images/chromhmm.png)

- ChromHMM: computes from many chromatin marks, the chromatin state of the genome by learning to pred ict the marks across the genome. State might be more meaningful than a bunch chromatin mark signal?


![meme logo](images/memechip_icon.png)

- MEME: motif analysis package
  - QCs and confirms ChIPseq data from looking a the motifs under peaks and comparing with known motifs for that TF
  - adds motifs to the conscensus peaks in the cobinding matrix.
  - we used it on a personalized genome (from CCLE WGS)

---

# Results


## Super Enhancers define distinct AML clusters


<img src="images/KMT2Aclusters.png" style="height:500px">

Suggesting unique differences in the transcriptional circuitry imparted by the KMT2A translocations.


## Divergent CRCs mark context-specific transcriptional addictions


KMT2Ar AML has some specificities in its CRC program. Which related to its dependency score.

<div class="r-stack" >
  <img src="images/2A.png" style="height:600px">
  <img class="fragment" src="images/2E.png" style="height:600px"> 
</div>


> We have really identified the reason and principle of CRC dependency in AML. __Which, What, How, Why.__


## Synthetic lethality reveals redundancy of co-expressed TF paralogs 


<img src="images/2I.png" style="height:500px">

some paralog CRC members can rescue the cell and act as partially redundant elements.


## CRC hierarchy revealed by patterns of genomic co-occupancy and knockout response


<img src="images/4E.png" style="height:70%;width:70%">

CRC is actually hierarchical, partially connected, with antagonistic regulation patterns.


<iframe src="images/v3_motifs_150enrichment_correlation.html" style="width:100%; height:545px; self-align:center;" style=""></iframe>


![](images/newcirc.png)


## super resolution microscropy as an orhtogonal confirmation


<div class="r-stack">
  <img src="images/superres1.png">
  <img class="fragment" src="images/superres2.png">
  <img class="fragment" src="images/superres3.png">
</div>

also with some quantitative analysis


## The IRF8/MEF2 axis regulates key oncogenes in KMT2Ar AML

In depth review of the IRF8/MEF2 axis an how/what it regulates in KMT2Ar AML.
won't show too much detail. very experimental.


<div class="r-stack">
  <img src="images/indeptIRF8_1.png">
  <img class="fragment" src="images/indeptIRF8_2.png" style="height:500px">
  <img class="fragment" src="images/indeptIRF8_3.png" style="height:500px">
  <img class="fragment" src="images/indeptIRF8_4.png" style="height:500px">
</div>


## A ressource

Hubs of CRC. No SE signature. Hierarchical structure


<img src="images/3A.png" style="width:600px;height:700px">

---

## Questions:

1. Do we have unidirectional effects?


<!-- ![plot of overlap](images/)
![plot of enrichment](images/) -->
<img class="r-stretch" src="images/v3_remove_double_150pairwise_overlap_clustermap.pdf.png">


2. Can these typical cobinding can be code of recruitment


<img class="r-stretch" src="images/v3_remove_single_150_kmeans_50enrichment_on_cluster_metasubset.pdf.png">
<!-- ![plot of enrichment on clusters](images/) -->


3. What is the behavior now when we filter TF binding by motif only?

<!-- ![enrichment on motif filtered binding](images/v3_remove_single_150_kmeans_100_enrichment_ofmotifs_on_cluster.pdf.png) -->

<img class="r-stretch" src="images/v3_remove_single_150_kmeans_100_enrichment_ofmotifs_on_cluster.pdf.png">


Can we see differences with unfiltered data?


2. Can we find interesting relationship between motif regions and TF binding?

<img class="r-stretch" src="images/v3_remove_single_150_enrichment_crc_tfs_onmotifs.pdf.png">


<img class="r-stretch" src="images/v3_remove_single_150_enrichment_of_non_motif_tfs_onmotifs.pdf.png">


3. Can we see cobinding typical of superenhancer?


1. Comparing the dataset and looking for interesting relationship and changes when filtering for motif/ changing some parameters, etc..


2. See if some set of TF can predict binding of some other TF.


3. Finish adding the ABC model:

  - adding gene set analysis to see if a TF or a set of TF refers to some known gene set.
  - seeing if some TF or set of TF explains some known dependencies. By regulating some key genes in AML. 
  - seeing if we can predict gene expression from TF binding signal? (of CRC genes at least).


4. Incorporating differential binding data. Differential binding is doing chipseq of some TF after having done fast knock out of some other TF and computing the differential peaks (binding in condition only / in both / in control only).

  - to know how some TF are changing cobinding from dropping other TF.
  - see if some set of TF can predict it.


5. Adding RNP data: our RNP dataset is composed of expression profile of our cell line 24h after fast knock out of each CRC members:

  - can we predict the differential expression of some gene types from the binding code of their associated enhancer?
  - can we find a relationship between differential binding profile of some TF after knock out and differential expression after some knock out?
