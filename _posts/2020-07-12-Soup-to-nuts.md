---
layout: post
title: "Soup to nuts - PAW style"
date: 2020-07-12
---

## Introduction

Are you looking for good resources for newbies to learn the proteomics ropes? Have you used my [PAW pipeline](https://github.com/pwilmart/PAW_pipeline)? It is a little messy and rough around the edges on purpose. It is very linear, explicit, and visual. After you have done a couple of datasets with my pipeline, then other pipelines will make more sense. All of the pipelines are doing the same steps, whether they are implemented one-by-one or rolled up into a black box. You do not need to switch to my pipeline for all of your proteomics work (there are many great alternatives); however, your conceptual understanding of proteomic data analysis will probably be improved.

> The documentation for the [PAW pipeline](https://github.com/pwilmart/PAW_pipeline) uses publicly available data that is described in detail. Some statistical analysis of that data can be found [in this repository](https://github.com/pwilmart/Yeast_CarbonSources). Another good end-to-end example of using my pipeline is [the sea lion](https://github.com/pwilmart/Sea_lion_urine_SpC) data (also publicly available).

---

I have added the Python script names for the various PAW pipeline steps as headers to the more general descriptions of proteomics data analysis steps. All of these steps are done in **every** proteomics data analysis whether you realize it or not. I am sorry that there are many complicated steps. It is an involved process and these steps just reflect reality.

---


## `msconvert_GUI.py`

The RAW files typically need processing. It used to be that search engines could not read the binary files. That isn’t really true anymore, so the need to convert RAW files to MGF or DTA or MS2 is mostly historical. Converting to mzML seems kind of wasteful. If a search engine can read mzML, it can probably read the RAW file. However, there can be other information in the RAW files besides the MS2 scans, like the reporter ion intensities for SPS-MS3 TMT. MaxQuant or other MS1 feature extraction tools also need to process the RAW files to get the quant data out. The key point is that an analysis usually starts with a RAW file processing step.

## `comet_GUI.py`

We almost always want to assign peptide sequences to MS2 scans. A search engine or spectral library matching are equivalent ways to do that. With Prosit, the distinction between a search engine and a spectral library is even more fuzzy. Search engines do not need additional experimental data to make assignments and are good for instructional purposes. They also have a rather prominent position in the history of proteomics.

You need to configure a search engine to run it. That involves finding, downloading, checking, and preparing a FASTA file (contaminants and decoys). That is really important, and you have to learn a bit and use some tools to do that well. The other search parameters range from the obvious to the obscure. A pretty small subset is all that is needed and thinking about tools to make that easy and prevent you from messing up less-common parameters are important to think about. Contrasting the basic parameters to the full parameter set can be quite eye opening.

## `histo_GUI`

Search engine results by their very nature are a can of worms. You get sequence assignments to every spectrum (almost) and we know we cannot interpret every single spectrum. Luckily, the target decoy method is a two-fer. It is conceptually straightforward and works well in practice. The nature of target/decoy can be made explicit and visual, or completely hidden. Hidden is not really a terrible thing, except when you are learning what is target/decoy and how do we implement it. My pipeline is explicit with target/decoy and the user has to interact with visual score histograms to set thresholds. I think it is hard not to understand target/decoy a little better after using my pipeline.

## `PAW_results.py`

At any rate, we need to find confident PSMs and remove the other PSMs before inferring proteins. Yes, inferring proteins is complicated but essential since most experiments are shotgun proteomics. Digesting proteins is not the only complication. We often fractionate samples and we have multiplexing in many quantitative methods. There are extra dimensions to most datasets. We have to think about how the LC runs relate to the samples and we have to think about how peptide sequences relate to protein sequences. These dual complications are intertwined and need to be considered together. A key point to learn is that chopping your proteins into pieces has consequences. We lose information and that loss of information is mostly what complicates assembling peptides into proteins. Seeing how the steps in protein inference (raw mapping, identical peptide set grouping, peptide subset removal) affect the number of proteins is educational and my pipeline outputs all of the protein inference action to the console. You can literally see what these steps are doing.

## `PAW_protein_grouper.py` (runs automatically)

Housekeeping protein families complicate quantitative interpretation due to excessive shared peptides. Thinking about what your quantitative measurements are actually able to measure is important. The criteria for protein identification (such as two distinct confident peptide sequences) and for quantification are different. Quantification is more demanding of the data and the number of quantifiable proteins is almost always smaller than the list of confidently identified proteins.

Okay. Lots of complicated biological methods and work created the samples in the study. More bench work was needed to extract proteins, digest into peptides, do any enrichments, do any stable isotope labeling. Further analytical work may have been required to fractionate the samples using chromatographic separations. Final analytical reverse phase chromatography and electrospray mass spectrometry was necessary to produce the data (from a working and calibrated system). All of the paragraphs above are just about how to try and get biological results back from the mass spec data. Now we have a parsimonious list of proteins likely to be in the samples.

## `add_TMT_intensities.py` and `pandas_TMT_IRS_norm.py`

That is not the end of the story. We need biologically actionable results. What that means depends on the biological questions that the study was probing. Biological results are things like statistically significant differential expression candidates, activated or deactivated pathways, clear cause and effect relationships (this protein does this thing to the system), kinetic rates, dose curves, protein turnover rates, etc. How do we turn what we observed with the mass spec into these biological measurements?

If we were doing quantitative proteomics to measure differential expression, we need to compute quantitative abundances for the things we are interested in (proteins, enriched phosphopeptides, etc.). There are many issues associated with taking quantitative measurements, typically at the PSM or scan level, and creating aggregated peptide or protein abundances. It is a [complicated topic](https://pwilmart.github.io/blog/2019/09/21/shotgun-quantification). My pipeline supports TMT labeling with single or multiple plexes. Statistical analysis of quantitative data is an even larger topic that what we are working on here and will be saved for a later day.

## [Annotations](https://github.com/pwilmart/annotations), [Ortholog mapping](https://github.com/pwilmart/PAW_BLAST), etc.

There are often additional informatics steps to annotate proteins with function and pathway details and to see if those change. There can be quantitative data processing steps followed by statistical modeling to test for large scale (many proteins at once) expression changes. The goal in discovery experiments is to suggest biological hypotheses to test. The testing (a.k.a. validation) can take many forms: targeted proteomics, Western blotting, more experiments on new or larger cohorts of subjects, etc.

The results can suggest new biological experiments that can take considerable time and effort to design and perform. We hit a fundamental snag at this point in analyses. Biological interpretation of a proteomics cannot (in most cases) be done by the analyst that can do the proteomics data processing. Much proteomics data processing is a bit decoupled from the biology behind the experiment. Interpretation is **not**. Any informatics from this point on benefit tremendously from expert domain knowledge. All this follow up work can take months to years to complete. Finally, a reasonably complete biological story can be told and published.

## Summary

It takes many people with many talents many man-years of effort to do modern biological experiments. Taking the time to learn all of the steps (and there are many) to just one part of these large studies (the proteomic data analysis) is not too much to ask. There is an even larger collection of scientists and engineers that design and manufacture the reagents, techniques, devices, instruments, hardware, and software to make all of this possible. Biology is extremely complex and doing experimental biology is truly challenging. It is inspiring and humbling to be a part of these studies in any capacity.

---

Phil Wilmarth
July 12, 2020
