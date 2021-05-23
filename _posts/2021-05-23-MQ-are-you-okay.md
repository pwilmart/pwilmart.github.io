---
layout: post
title: "Is MaxQuant holding back proteomics?"
date: 2021-05-23
---

## Background

The original [MaxQuant publication](https://www.nature.com/articles/nbt.1511) has almost 9900 citations according to Google Scholar. I saw recent tweets where 3500 scientists have already registered for the upcoming [MaxQuant Summer School](https://maxquant.org/summer_school/). Those are truly impressive numbers. The impact that MAxQuant has on current proteomics work cannot be understated. A widely used, freely available proteomics "black box" with summer schools, [YouTube videos](https://www.youtube.com/c/MaxQuantChannel/videos), and a [Google discussion group](http://groups.google.com/group/maxquant-list) has been a tremendous benefit to the proteomics community.

---

**Design choices for PAW pipeline:**

- wide parent ion mass tolerance
- scores are discriminant function (XCorr and deltaCN)
- accurate mass conditional score histograms
- target/decoy PSM FDR by peptide subclasses

---

**Design choices for MaxQuant:**

- very narrow tolerance searches
- scores are Andromeda primary match scores
- target/decoy PSM FDR (peptide length and maybe other subclasses)

---

![MaxQuant-like score distributions](https://pwilmart.github.io/images/MQ-like.png)

![PAWt-like score distributions](https://pwilmart.github.io/images/PAW-like.png)

## PSM performance

Comparing numbers of PSMs at a given FDR is a straightforward, fair comparison. Comparing proteins with some differences in protein acceptance criteria is much more apples-to-oranges.


## Conclusions

MaxQuant/Andromeda is not a particularly sensitive PSM pipeline. Meta-scoring (combinations of scores and other quantities) and better use of parent search ion tolerances could offer **dramtically** improved performance. Maximizing numbers of identified PSMs will do the most to improve the quantitative data quality. Summing information from more PSMs per protein will improve the protein-level data summaries. Comparing numbers of protein identifications is more challenging than it may at first seem. There are many ways to inadvertently inflate protein numbers. Counting PSMs is far safer.     
