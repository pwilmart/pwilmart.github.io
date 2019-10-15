---
layout: post
title: "Keeping your pipeline from getting clogged"
date: 2019-09-08
---

## Introduction

This installment on data plumbing will focus on how to make sure something useful actually comes out of the far end of your pipeline. Most pipelines involve **many** steps, whether you are made aware of that or not. There are many wrappers for pipelines that are commonly used that try to hide the steps. While this can make life easier (sometimes), it results in unpleasant user expertise atrophy.

Proteome Discoverer is a pipeline wrapper. MaxQuant is a pipeline wrapper. Scaffold is a pipeline wrapper. You get the idea. When you do not realize all of the steps in the pipeline, it is easy to pick some upstream options that can clog up downstream steps and prevent anything useful from getting to the end of the pipe (or something undesirable coming out of the pipe).   

## Pipeline steps

As proteomics has matured, more and more steps get added to most analyses. We started with lists of proteins identified in single LC runs. Then we had multi-dimensional separations and several LC runs had to be combined into one protein list. Then we started comparing multiple samples in experiments, so we needed reports where the proteins from multiple samples were lined up. Then we added relative quantitative measurements for the proteins. We would not be doing a quantitative experiment if we did not want a differential expression statistical analysis added. Biologists need to know something about the proteins to interpret results, so annotations (gene names, functional terms, pathways, database links, etc.) need to be added. We may need to map identified proteins to orthologs of well-annotated species to add annotations. We may need to integrate quantitative data and annotations to explore the biology (eg. gene set enrichment).

What are the major steps in typical proteomics pipelines?

**Shotgun Pipeline Steps:**

- RAW file conversion/access (some example steps)
  - quality control metrics
  - data preprocessing
  - format conversions
  - feature extraction
  - fragment ion scan extraction
  - quantitative scan data extraction
- Search engine run to produce peptide spectrum matches (PSMs)
  - FASTA database selection
  - search engine parameters
- PSM error control and false discovery rates
  - score transformation and classifier functions
  - heuristics
  - target/decoy analysis
- Basic protein inference and parsimony
  - indistinguishable peptide sets
  - remove subset peptide sets
- Protein error control and false discovery rates
  - heuristics
  - ranking functions
- Extended parsimony and protein grouping
- Adding protein inference dependent quantitative data
  - what data is usable?
  - what quantities do we want?
  - what aggregation methods to use?
- Differential expression testing
- Adding useful annotations
- Other analyses (functional and/or pathway enrichment, etc.)

If you are thinking that this list is too long (or longer than what you usually do), what I am outlining here is getting the data to **the finish line** and publishing the results. Not all of these steps are done at once, or by the same person. I consider them all steps in a **single pipeline** that the data went through. You do not really get any points for trying in science. There is no self-esteem trophy. You and your collaborators have to get the data all of the way through the pipeline to get a publication.

## Tool Builder / Tool User dichotomy

Interestingly (or frustratingly), most of the **tool builders** only seem to focus on the first two or three major steps. The goal might seem to be to write a new search engine for each and every dataset. That joke only works because there is some truth to it. The reality is that **tool users** need to get data all the way through pipelines and not just through the search engine step. Another reality is that the perceived value of the pipeline steps goes **way down** as you go from the top to the bottom. The steps at the top have a **novel discovery** flashing neon sign. The steps at the bottom just scream **boring and practical**. Good luck getting a boring and practical publication!

I am always struck by how much excitement and focus at ASMS meetings is on new instrumentation, new quantitative reagents, new acquisition methods, new data pre-processing methods, and new peptide and PTM identification methods. There can be many years delay between when ideas are run up the flagpole at meetings and when successful applications of those ideas to answer scientific questions becomes routine. The talks and posters with applications of methods to solve biological problems always seem a bit dated compared to all of the new stuff. Meetings seem to reinforce the perceived importance of upstream pipeline steps at the expense of downstream steps.

I want to clearly state that detecting proteoforms, detecting alternative slicing, detecting disease associated mutations, detecting novel protein processing, and detecting novel post-translational modifications are all **extremely important**. However, those are advanced proteomics topics. They are much harder than you have been told. The game is not over once you find one putative PSM that suggests something interesting. That is really more like the pregame warmup. You have to have a hypothesis and an experimental design with controls. You have to **validate everything**. You have to bust your butt to try and prove yourself wrong.

## Do not bite off more than you can chew

Cool, exciting stuff at the beginning of the pipeline may lead to clogs at downstream steps. Do not overcomplicate (or overthink) protein expression studies. The mental model for most folks seems to be that each new instrument produces datasets resembling ever larger Swiss army knives. The dataset will have all of the proteins, all of the possible variants, super precise and accurate quantitative data, all of the PTMs in NextProt, all of the PTMs not in NextProt, and everything else that I have not even imagined yet. Oh, and ion mobility, or data from across the 8th dimension, or data back from the future. So why not try and get everything out of the data in one go?

A protein expression study is about comparing protein abundance measures between samples. Shotgun techniques are looking at pieces of proteins. The unmodified peptides from a canonical sequence will constitute the vast majority of the peptide signals from each protein. I do not want to burst bubbles here, but that is the reality. Modifications are substoichiometric without enrichment. Most protein isoforms do not have detectible distinguishable peptides unless those proteins are very abundant and the sequence coverage is high. There is nothing to be gained by looking for things that are not there or not abundant enough to matter.

Every rule has exceptions. However, questions about isoforms, variants, PTMs, etc. are separate questions from differential protein expression. They should be explored in independent, separate analyses. If we had extensive PTMs in our samples and the PTMs varied by condition, then not specifying PTMs in the database search will bias the quantitative results. Studies with samples like that will be rare. Some characterization of samples for PTMs and how they vary by condition needs to be done first to see if a quantitative bottom up study can be done for those cases. If PTMs are going to be important, the more abundant one have to be determined and the quantitative experiment altered accordingly. A quantitative shotgun experiment has limitations and is not always the right thing to do. That is why there are other methods like top-down and targeted analyses.

## Recommendations

What are some recommendations for protein expression experiments? Use FASTA protein databases with low peptide redundancy. The canonical reference proteomes from UniProt are good. Do not specify more PTMs than you need to. Oxidized methionine is usually abundant enough and variable enough by sample that it needs to be specified. Protein N-terminal acetylation is not so good because it is variable by protein. The sequence may or may not have the initial Met removed, some residues are more likely to be acetylated than others (if Met is removed). Unless differential protein N-terminal acetylation is the point of the experiment, it is best skipped.

Tryptic searches work well for most cell lysate samples. Biofluids (serum, saliva, tear, urine, sweat, proximal fluids) should use semi-tryptic searches. There are signal peptide removal, protein processing, endogenous proteases, and other protein degradation processes that generate peptides that will be missed in tryptic searches. Glycosylated proteins can be common in these samples as well, and those are hard to detect in shotgun studies. Differential glycosylation between conditions could bias results. I am not sure how to check for that.  

Narrow tolerance parent ion searches (like 5-10 PPM) compromise database searching and subsequent classifiers by eliminating accurate mass as something that distinguishes a correct match from an incorrect match. All matches (correct **and** incorrect) have masses within 5-10 PPM by definition. There are also some scoring values that depend on getting a meaningful estimate of a noise match score for a specific spectrum (deltaCN, other gap scores, and expectation values), so there have to be a statistically significant number of scored sequences for each spectrum. That is not guaranteed with narrow tolerance searches. Deamidation and C13 first isotopic peaks triggers are also concerns. So is how to create decoy database sequences.       

Use a protein grouping or clustering method to combine protein families that have high sequence homology instead of a razor peptide method. Some housekeeping genes have multiple coding regions that make identical or very similar sequences. This redundancy helps keep you alive. It can mean that protein members of families like alpha-tubulins, beta-tubulins, actins, histones, etc. can have few **non-shared** tryptic peptides. Razor peptide methods pick one protein of the family to get all of the shared peptides. The other family members just get a handful of their unique peptides and no shared peptides. That means that one family member gets large quantitative signals and the other family members get small, unreliable values. It is better to remove the poor quality family members by grouping the entire family into a single entity.

## Keep it simple

Getting too fancy/greedy with the database search configuration in protein expression studies can often complicate FDR analysis and possibly result in increased false positive PSMs. We need to have robust, simple lists of proteins and their quantitative estimates to compare many samples in typical studies. Aligning protein identifications between samples can be challenging and overly complete (large) protein FASTA files can make that harder. Some protein grouping steps can simplify protein lists when canonical databases are not an option.

Simplified protein lists not only make the differential expression testing easier to do and interpret, most annotation information is gene-centric. It makes sense to have a one gene, one protein relationship when adding annotations. If ortholog mapping needs to be done to a better-annotated species, mapping canonical sequences to canonical sequences is much easier. Most downstream steps rely on minimal ambiguity to work well. Yes, there is some loss of information associated with minimizing ambiguity. There is also some compromise in quantifying **proteins** from their **tryptic peptides**. Some consistency in the compromises of the analysis across the pipeline steps is the workable approach.

## Final Thoughts

I am not proposing that we all do only the lowest common denominator experiments. What fun is that? I am proposing that there is no such thing as a **one and done** analysis. Start with a simple search if doing protein expression and see what you get. Explore other things with additional analyses. Are there non-tryptic peptides in my sample? How abundant are they? Can I ignore them? Are the lots of PTMs in my sample? How abundant are they? Can I ignore them? Are there different protein isoforms present? How abundant are they? Can I ignore them?

I am not aware of any tools that help you compare different analyses of the same data with different databases and/or search settings. There is a lot of opportunity for tool builders here. For example, sets of identified peptides can be used to pair up protein results from searches of different databases, kind of a poor man's BLAST. That can be used to see if there are protein sequences in your sample that might be missing from simpler databases that were found in messier complete databases. Strategies like that get a little ragged as the numbers of peptides per protein decrease, but I never said it was easy.   

---

Phil Wilmarth, September 8, 2019.
