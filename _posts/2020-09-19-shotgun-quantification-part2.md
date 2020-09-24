---
layout: post
title: "Unique peptides and shotgun quantification"
date: 2020-09-19
---

## Introduction

> Apologies to non-USA readers: "." is used for the decimal point and "," is the thousands separator.

The first post on [shotgun quantification](https://pwilmart.github.io/blog/2019/09/21/shotgun-quantification) talked about how protein expression in shotgun proteomics is a bit like trying to put [Humpty Dumpty](https://en.wikipedia.org/wiki/Humpty_Dumpty) back together. That post is complementary to much of what is written here.

In this post, we will explore how choice of [protein FASTA file](https://en.wikipedia.org/wiki/FASTA_format) affects shotgun quantification using **unique** peptides. [Protein inference](https://www.mcponline.org/content/4/10/1419.short) has been a central part of shotgun proteomics for well over 15 years. All possible proteins will not be present in every sample, particularly for higher eukaryotic organisms with specialized tissues. We do not flash freeze humans, grind them up into homogeneous samples, and then run those on the mass specs. Our samples for higher organisms will have defined subsets of the set of all possible gene products present (as represented by the protein FASTA file). We use a technique called protein inference to figure out the likely proteins in our samples.

> Some background from genomics (which I know very little about): DNA has regions (genes) that code for proteins (gene products). The complexity of genes and gene products is not the same across the tree of life. Many organisms have simple gene structures that produce simple gene products. Higher organisms have more complicated genes (introns and exons) that can produce a variety of gene products in a variety of ways. Alternative splicing is one way a gene can make more than one protein form (proteoforms or isoforms). Genetic variation (mutations) can affect proteins. There are also a wide variety of post-translational mechanisms to increase protein diversity.

> There are other genomic factors that affect proteomics. Many housekeeping genes are duplicated for robustness and the duplicated genes may not code sequence-identical proteins (they are probably functionally equivalent). Some genes, such as immunoglobulins have mechanisms to make diverse gene products on purpose. Other proteins, like the HLA system (or MHC in other species), have large numbers of alleles in the population. There are notable protein families that have many members with similar, but not identical, protein sequences.  

If you only work with yeast and E. coli samples, you are probably blissfully ignorant of the complexities posed by higher organisms. If you work with human or mouse samples and are not up to speed on these topics, you are probably unintentionally reducing the quality of your proteomics work. This is (tragically) an almost completely ignored proteomics topic. Education is the salvation and [Ravi's 2016 Masters thesis](https://digitalcollections.ohsu.edu/concern/etds/c534fp149) is where to start. You have to understand the different sources of protein FASTA files and how they reflect (or do not reflect) the underlying genomic complexity to think through the implications for protein inference. Once the inadequacy of basic protein inference is recognized, the need for protein grouping can be seen.

We have many choices for protein FASTA files for higher species. These choices encapsulate different biological information by design. You have to match your FASTA file choice to the goal of your experiment. A goal in many experiments is to measure relative protein expression levels. We do not measure protein abundance for every protein in the FASTA file (because many of them will not actually be in our sample). We want to measure the expression of the proteins in the sample (i.e, the **inferred** proteins). How the choice of FASTA file affects protein inference and how protein inference affects protein expression measurements is what we will touch on here.

## Mouse UniProt FASTA choices

UniProt is the overwhelming choice for higher eukaryotic protein FASTA files for proteomics use. NCBI and Ensembl may be the first choice for other organisms. I will use mouse sequence collections at UniProt as an example. Mouse has a taxonomy number of 10090 and you can see what sequenced genomes have been done by using the [taxonomy browser at NCBI](https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi). _Mus musculus_ is really the genome for the C57BL/6 strain from Jackson Labs. There are other strains and other wild mice that also have sequenced genomes. We used to have species with no sequenced genome. Then we had species with one genome. Now we have multiple related strains with sequenced genomes. Soon we will have sequenced genomes from individual mice (this one is from Fuzzy and that one is from Lumpy). The idea of a single mouse reference genome still exists today, but I make no promises...

Alright then. We all agree we want a reference proteome for C57BL/6 mice from UniProt. That should be a "no brainer", right? Do you want ["Reviewed" or "Unreviewed"](https://www.uniprot.org/help/uniprotkb_sections) sequences? Do you want all [mouse sequences](https://www.uniprot.org/taxonomy/10090), a [mouse proteome](https://www.uniprot.org/help/proteome), or a mouse [reference proteome](https://www.uniprot.org/help/reference_proteome)? Do you want to download the FASTA file from the REST API (from the web page) or using the FTP site at UniProt? Are the databases at the FTP site the same ones that I can get from the web site? Does your proteomics analysis tool provide UniProt protein FASTA files for you? Exactly which mouse UniProt FASTA file is that? There are clearly a finite number of mouse proteins, but it seems like there are an almost unlimited number of mouse FASTA files at UniProt!

[UniProt](https://www.uniprot.org/help/uniprotkb) has a confusing array of FASTA file options for any given species. UniProt has two general sequence collections: Swiss-Prot (Reviewed with "sp" prefixes in FASTA files) and TrEMBL (Unreviewed with "tr" prefixes in FASTA files). The reviewed sequences are manually curated. That involves a lot of processing by the curator. The unreviewed sequences are run through an automated processing pipeline which can add some limited annotations. The number of proteins that have been reviewed is very small compared to the total number of protein sequences at UniProt.

Sequences make a one-way trip from TrEMBL to Swiss-Prot during the curation process. Protein isoforms are annotated as differences with respect to a chosen canonical sequence in the Swiss-Prot database record. Often the longest protein sequence is used because it is easier to describe deletions rather than insertions. This means that several TrEMBL sequences corresponding to the isoforms for a given protein all disappear from the unreviewed (TrEMBL) sequences and are replaced by one canonical sequence in the reviewed collection. The important thing is that protein isoforms are explicitly represented as different sequences in TrEMBL unreviewed collections. However, all isoforms (except the canonical one) are hidden within the Swiss-Prot database record for the reviewed sequences.

> Isoforms at UniProt are mostly from alternative splicing. Mutations and protein processing (mature protein form, signal peptide, etc.) are annotated but I am not 100% sure that they are part of the isoform collections.

This has very practical (and poorly understood) implications. If you want to download a reviewed (Swiss-Prot) protein database, what do you want to do with the annotated isoforms? There are two options for downloads: canonical (no isoforms) or canonical+isoforms (addition of all annotated isoforms for each gene product). This has a well-defined meaning only for Swiss-Prot (reviewed) FASTA files.

However, UniProt offers many other flavors of FASTA files for every species. Many species have designated (complete) proteomes where the reviewed and unreviewed sequences are processed into a simplified set to provide a good compromise of gene coverage without being too redundant. The mouse [UP000000589 reference proteome](https://www.uniprot.org/proteomes/UP000000589) is a combination of reviewed (17,042) and unreviewed (38,429) sequences. Because they have reviewed (Swiss-Prot) sequences, the question of what to do with isoforms comes back to haunt us. It is difficult to consider a sequence set complete with missing isoforms for all of the reviewed sequences, but, unfortunately, that is an available option to download. Proteomes and complete proteomes also have the canonical or canonical+isoform download options.

You can also download all of the Swiss-Prot and TrEMBL sequences for an organism (without the special "proteome" processing). Once again, because these are a combination of reviewed and unreviewed sequences, there are options for canonical or canonical+isoform downloads. For mouse, we have six FASTA file variants so far: Swiss-Prot canonical, Swiss-Prot canonical+isoforms, UP000000589 canonical, UP000000589 canonical+isoforms, Swiss-Prot+TrEMBL canonical (canonical only applies to Swiss-Prot entries), or Swiss-Prot+TrEMBL canonical+isoforms.

There are even more FASTA files that we can find at UniProt. There is an [FTP site](https://www.uniprot.org/downloads) with different sequence collections for a subset of organisms. These are conceptually one gene producing one protein sequence collections. Human has 20.2K Swiss-Prot sequences and the database from the FTP site is about 21K sequences. Mouse Swiss-Prot is about 17K sequences with 21K sequences from the FTP site. The databases from the FTP site are the best way to get minimally redundant, complete sequence sets for the available organisms (see https://github.com/pwilmart/fasta_utilities for tools to assist in downloading FASTA files from the FTP site).

Remember the situation with isoforms and reviewed sequences? Something like that exists for the FTP sequence collections as well. This is more of an automated isoform cataloging situation for the FTP collections as opposed to the manual curation for reviewed sequences. Details on how these FTP databases are created are not documented at UniProt (that I am aware of). Ortholog relationships between species are important in how they are constructed. The canonical sequence collections are often similar sizes for similar species. The number of isoforms is much more variable species-to-species as some organisms are better studied than others.

So, for mouse, we have at least eight distinct (and completely valid) UniProt protein FASTA file choices to use in database searching. Other sources of sequences, like NCBI or Ensembl, can also have different protein database choices for organisms. The situation for other sources is typically a little simpler than the quagmire that is UniProt.

Okay. We have eight mouse FASTA files. How do they compare? There are [tools at GitHub](https://github.com/pwilmart/fasta_utilities) that we can use to analyze the FASTA files. The table below summarizes some key characteristics:

Database|Proteins|Tryptic Peptides|Non-redundant Peptides|Shared Peptide Fraction
--------|--------|----------------|----------------------|-----------------------
Swiss-Prot Canonical|17,053|2,062,581|2,009,613|1.9%
Swiss-Prot Canonical+Isoforms|25,323|3,200,117|2,058,426|32.3%
FTP Reference Canonical|21,989|2,467,386|2,343,477|2.9%
FTP Reference Canonical+Isoforms|63,739|6,129,368|2,599,761|56.3%
UP000000589 Canonical|55,471|4,992,259|2,565,266|47.2%
UP000000589 Canonical+Isoforms|63,739|6,129,368|2,599,761|56.3%
Swiss-Prot+TrEMBL Canonical|87,975|7,472,572|2,815,157|61.7%
Swiss-Prot+TrEMBL Canonical+Isoforms|96,245|8,610,108|2,846,122|66.9%

We see that the number of proteins varies by a factor of almost 6. The total number of tryptic peptides (at least 7 amino acids and up to 2 missed cleavages) varies 4-fold. Many of those peptides are duplicates and the non-redundant numbers only vary by a factor of 1.4. The degree of shared peptides ranges from 2% to 67%.

## How does FASTA file affect actual sample results?

Peptide redundancy is not just a function of the FASTA file choice, it also depends on the samples, what proteins are in the samples, and the abundances of the proteins (we see less of lower abundance proteins). We can use the data from [this paper](https://www.nature.com/articles/leu2015163) to see how FASTA file choice affects actual results lists. The data are mouse bone marrow cell cultures exposed to leukemia exosomes. The proteins were digested with trypsin and labeled with 10-plex TMT reagents. This data has been used in some of my data analysis notebooks at GitHub (see https://github.com/pwilmart/MaxQuant_and_PAW). The data is not publicly available, but it is a nice modest sized data set (275,695 MS2 scans in total) for mouse with a decent number of detectible proteins. To see how the 8 different FASTA files affect proteomics results, I will use MaxQuant v1.6.17, one of the most widely used proteomics "black boxes". I will change the FASTA file for the 8 runs. 10-plex SPS MS3 TMT was specified. Other parameters are the usual (default) suspects.

Let's see how the FASTA file affects protein identifications:

Database|Number of Proteins
--------|------------------
Swiss-Prot Canonical|5,201
Swiss-Prot Canonical+Isoforms|5,220
FTP Reference Canonical|5,306
FTP Reference Canonical+Isoforms|5,391
UP000000589 Canonical|5,382
UP000000589 Canonical+Isoforms|5,354
Swiss-Prot+TrEMBL Canonical|5,416
Swiss-Prot+TrEMBL Canonical+Isoforms|5,408

MaxQuant implements basic parsimony logic for protein reporting. We see about 100 fewer IDs with the somewhat incomplete Swiss-Prot FASTA file. Mostly, the number of identified proteins is independent of the FASTA file choice. This makes sense. What we can detect in the sample should be a function of the sample itself and the mass spec settings. It should not depend strongly on the data analysis choices (like FASTA file) as long as we have not inadvertently screwed something up.

## Quantification using "unique" peptides

We start with proteins and chop them into peptides in shotgun proteomics. For some measurement of a peptide, we need to know how the peptide measurement relates to the protein. This is the peptide's information content. One of the things we measure about the peptides is it sequence (and existence). If the peptide sequence (considering isoleucine and leucine indistinguishable) can only come from a single protein sequence, then we have a one-to-one relationship between the peptide's existence and the protein's existence. That is unambiguous information content.

We can have conserved protein regions that can give rise to the same tryptic peptides. Sometimes these are in similar protein sequences (some protein complexes can have sequence similarity, for example), or in different proteins (like zinc-finger proteins). When we detect one of those peptides, we have ambiguous information. The peptide does have information content - we know that one or more related proteins have to be in the sample, we just do not have enough information to say more based on the single peptide.

The basics of [inferring which proteins](https://www.mcponline.org/content/4/10/1419.short) are in the sample has been spelled out. You can also see [this presentation](https://github.com/pwilmart/Cascadia_2013) for a recap. The peptide sequences that map to just one protein sequence are called **"unique"** peptides and they are the drivers for protein identification evidence. Peptides that map to more than one protein are called **shared** peptides. I put unique in quotes because that is the heart of this discussion. Since we saw above that different FASTA files for the same organism can have very different sets of protein sequences, it stands to reason that definitions of shared and unique peptides will depend on the set of proteins we are considering (i.e., the **context**).

You might be imagining mapping peptide sequences to protein sequences as some sort of sequence alignment process. That is not really how it is done. For each protein in the FASTA file, we can form the set of all peptides that map to its sequence (many proteins will not have any peptides and we ignore those). Then, protein inference is a series of set operations (combining identical sets and removing formal subsets) to simplify this list of proteins using Occams' razor (what is the minimal set of proteins that can explain all of the observed peptides - a minimum set cover problem). Protein inference rules are trivial for unique peptides (we would not need to do protein inference if we only detected unique peptides). Thus, protein inference is all about the shared peptides. In next generation genomic sequencing, reads that map to multiple genes are typically ignored. We do not have the same sampling depth in proteomics and the reality is that ignoring all shared peptides would result in losing too much information.

Consider two proteins where all of the handful of observed peptides were the same. We cannot distinguish the two proteins from the peptides we detected. Every peptide is a shared peptide with respect to the FASTA file. We can combine the two identical peptide sets into one **protein group** that consists of the two proteins. Do we still think of all of the peptides associated with the protein group as shared peptides? Each peptide is now associated with a single protein group. If we change the context from the FASTA file (all possible proteins) to the list of proteins we think are present in the sample, then we need to change the definitions of shared and unique peptides accordingly.

What does this have to do with quantification? While shared peptides have some useful information for protein inference, that is much less true for quantification. The abundance of a shared peptide is a mixture of the abundances of the proteins that can produce the peptide. We do not have simple, reliable ways to know how to split up the signal from shared peptides. We need to exclude shared peptides and use unique peptides when we do protein quantification.

We still have this question of what context do we use to define shared and unique peptides. We saw above that there is no single definition of a FASTA file. There is not really any justification to think that unique peptides in the FASTA file context are somehow **better** than unique peptides in the inferred protein list context. I think if protein identifications are what the study is about, then measuring protein expression in a framework that is consistent with the protein identifications is clearly the correct answer. That means we should be doing quantification of the inferred proteins with shared and unique peptides defined in that context.

## How does context affect quantification?

MaxQuant produces many results files. The `evidence.txt` file has a lot of information. This is really an MS1 feature table where each row represents a feature (remember that MaxQuant was developed for SILAC). The table rows have columns that list protein accessions in different contexts (see table below). We can filter table rows for unique peptides (they have only one accession) and count things (rows [like spectral counts], MS1 intensities, TMT intensities, etc.). These different quantities (spectral counts, MS1 intensities, or reporter ion intensities) represent the quantitative information content (QIC).

Column Header|Description (from MaxQuant)
-------------|-----------
Proteins|The identifiers of the proteins this particular peptide is associated with.
Leading proteins|The identifiers of the proteins in the proteinGroups file, with this protein as best match, this particular peptide is associated with. When multiple matches are found here, the best scoring protein can be found in the 'Leading Razor Protein' column.
Leading razor proteins|The identifier of the best scoring protein, from the proteinGroups file this, this peptide is associated to.   

The `Proteins` context is the FASTA protein sequence file. If a peptide maps to a single sequence in the FASTA file, there will be just one accession. Keep in mind that peptide to protein sequence matching is not string matching. Leucine and isoleucine have identical masses and cannot be distinguished with mass spectroscopy.

The `Leading proteins` context is basic parsimony protein grouping where proteins having indistinguishable peptide sets are grouped together and reported as one protein group. Proteins whose peptide sets are formal subsets of larger protein peptide sets are not reported. Two proteins with indistinguishable peptide sets would have every peptide being shared in the `Proteins` context. After grouping, all peptides could end up as unique to the group (some of the peptides could still be shared with other proteins or protein groups).

The `Leading razor protein` context goes a step farther than the basic parsimony logic. There are families of proteins (often housekeeping genes) that can share many peptides (sometimes the majority) but still have some unique peptides so that they are distinguishable. The unique peptides can be used to construct some ranking function and order the proteins in the “razor” set (a reference to Occam’s razor, I think). Then, any shared peptides can be 100% assigned to the top ranked protein for quantitative rollup. The other proteins in the razor set do not get any contributions from the shared peptides and are quantified by their unique peptides only (which may be less reliable). The razor concept does not discard any shared peptides, but also does not over count any shared peptides.

> We will define the QIC as the fraction (%) of quantitative data that is usable (from unique things) out of the total amount of information.

The `Leading razor proteins` column has a single accession for each row (by definition), so the total number of rows and the unique peptide rows in the razor protein context are always the same. In other words, there are no shared peptides in the razor protein scheme. Every bit of quantitative data is assigned uniquely to just one razor protein and the QIC is always 100%.

We can count `evidence.txt` table rows that have single accessions in the `Proteins` and `Leading proteins` columns. This will be similar to spectral counting.

#### Based on `Proteins` column:

Database|Total Rows|Unique Rows|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|53,872|50,349|93.5%
Swiss-Prot Canonical+Isoforms|54,006|39,254|72.7%
FTP Reference Canonical|54,518|50,262|92.2%
FTP Reference Canonical+Isoforms|55,093|20,602|37.4%
UP000000589 Canonical|55,016|24,667|44.8%
UP000000589 Canonical+Isoforms|54,819|20,521|37.4%
Swiss-Prot+TrEMBL Canonical|55,030|10,002|18.2%
Swiss-Prot+TrEMBL Canonical+Isoforms|55,171|7,866|14.3%

<br />

#### Based on `Leading proteins` column:

Database|Total Rows|Unique Rows|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|53,872|51,492|95.6%
Swiss-Prot Canonical+Isoforms|54,006|50,842|94.1%
FTP Reference Canonical|54,518|51,888|95.2%
FTP Reference Canonical+Isoforms|55,093|51,278|93.1%
UP000000589 Canonical|55,016|51,559|93.7%
UP000000589 Canonical+Isoforms|54,819|50,784|92.6%
Swiss-Prot+TrEMBL Canonical|55,030|50,874|92.5%
Swiss-Prot+TrEMBL Canonical+Isoforms|55,171|50,737|92.0%

<br />

---

We can count total MS1 intensity instead of rows. This would be more like LFQ intensities.

#### Based on `Proteins` column:

Database|Total MS1 Intensity|Unique MS1 Intensity|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|3,834,929,859,686|3,068,264,365,266|80.0%
Swiss-Prot Canonical+Isoforms|3,843,142,257,018|2,647,937,358,538|68.9%
FTP Reference Canonical|3,808,397,193,568|2,912,312,056,218|76.5%
FTP Reference Canonical+Isoforms|3,830,886,651,358|1,336,917,648,830|34.9%
UP000000589 Canonical|3,829,057,768,188|1,510,283,477,330|39.4%
UP000000589 Canonical+Isoforms|3,824,050,727,018|1,335,710,234,560|34.9%
Swiss-Prot+TrEMBL Canonical|3,811,084,659,628|481,346,639,560|12.6%
Swiss-Prot+TrEMBL Canonical+Isoforms|3,820,561,142,258|388,392,323,450|10.2%

<br />

#### Based on `Leading proteins` column:

Database|Total MS1 Intensity|Unique MS1 Intensity|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|3,834,929,859,686|3,221,253,168,216|84.0%
Swiss-Prot Canonical+Isoforms|3,843,142,257,018|3,136,236,774,858|81.6%
FTP Reference Canonical|3,808,397,193,568|3,138,897,125,988|82.4%
FTP Reference Canonical+Isoforms|3,830,886,651,358|2,998,961,894,298|78.3%
UP000000589 Canonical|3,829,057,768,188|3,068,816,509,728|80.1%
UP000000589 Canonical+Isoforms|3,824,050,727,018|2,986,664,848,358|78.1%
Swiss-Prot+TrEMBL Canonical|3,811,084,659,628|2,907,292,776,968|76.3%
Swiss-Prot+TrEMBL Canonical+Isoforms|3,820,561,142,258|2,844,596,073,438|74.5%

<br />

---

We can count total reporter intensity instead of rows. This would be for TMT experiments.

#### Based on `Proteins` column:

Database|Total MS1 Intensity|Unique MS1 Intensity|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|685,289,186|608,855,857|88.8%
Swiss-Prot Canonical+Isoforms|686,577,033|503,191,425|73.3%
FTP Reference Canonical|687,949,202|596,733,058|86.7%
FTP Reference Canonical+Isoforms|692,656,986|256,191,136|37.0%
UP000000589 Canonical|692,078,208|295,851,381|42.7%
UP000000589 Canonical+Isoforms|696,162,376|257,408,848|37.0%
Swiss-Prot+TrEMBL Canonical|691,379,671|102,739,149|14.9%
Swiss-Prot+TrEMBL Canonical+Isoforms|692,454,101|84,342,052|12.2%

<br />

#### Based on `Leading proteins` column:

Database|Total MS1 Intensity|Unique MS1 Intensity|QIC
--------|----------|-----------|---
Swiss-Prot Canonical|685,289,186|630,636,056|92.0%
Swiss-Prot Canonical+Isoforms|686,577,033|621,988,149|90.6%
FTP Reference Canonical|687,949,202|628,412,174|91.3%
FTP Reference Canonical+Isoforms|692,656,986|616,840,307|89.1%
UP000000589 Canonical|692,078,208|620,799,282|89.7%
UP000000589 Canonical+Isoforms|696,162,376|617,668,770|88.7%
Swiss-Prot+TrEMBL Canonical|691,379,671|605,315,451|87.6%
Swiss-Prot+TrEMBL Canonical+Isoforms|692,454,101|602,241,509|87.0%

## Summary

We see that, for any type of quantitative measurements, the QIC has a strong dependence on the FASTA file choice when we use the `Proteins` column (the FASTA file context). That is what we expected. What might be surprising is how much quantitative information can be lost. We can end up with only 10% to 15% of the measurements being used. That quickly turns your great experiment into a pretty poor experiment.

The `Leading proteins` context (the list of inferred proteins) makes a dramatic improvement. It mostly removes the FASTA file effect and stabilizes the QIC. This is just a basic parsimony treatment. There can be further gains from additional protein grouping. The `Leading razor proteins` results in QIC of 100% (but has other issues that I do not like so much).

In the spirit of **do the least harm** to your data, there are some recommendations:

- Use minimally redundant protein FASTA files unless there is a compelling reason not to
- protein expression is more robust than peptide expression (there is more averaging)
- quantification should be informed by protein inference for protein expression studies

Like everything in life, there can be specific cases where everything I have outlined is total bullshit and should be completely ignored. If you know enough about your system to think that all of this is BS, then you already know how to do your experiment differently to achieve your goals. I am outlining the basics for quantitative proteomics that applies to many studies. What I have presented here is based on many years of experience with a wide variety of data.

It can take several years of hard work to successfully apply the cutting edge methods you see at conferences or in the top journals to the routine studies we do everyday. Getting things to really work often requires some simplifications and assumptions. These foundational basic proteomics concepts are what the exciting, new methods are being built upon. We need to do a better job in proteomics to make sure the foundation is solid and the right shape for what we want to build. Some days is seems like we are caught up in surface finishes and fancy fixtures while the foundation crumbles.  

---

Thanks for reading.
Phil Wilmarth, September 19, 2020.
