---
layout: post
title: "How do we try and put Humpty Dumpty back together?"
date: 2019-09-20
---

- it is just spectral counting
- two-peptide rule and protein ranking functions
- identifiable versus quantifiable
  - missing data and low abundance cutoff
- grouping pros and cons
  - razor peptide method is not good
- negative controls vs positive controls

## Introduction

This second installment on data plumbing will tackle the interplay between [protein inference](https://www.mcponline.org/content/4/10/1419.short) and quantitative data rollup in shotgun proteomics. TMT labeling will be the example type of data. The discussion will be about the general problem of how we try to put [Humpty Dumpty](https://en.wikipedia.org/wiki/Humpty_Dumpty) back together again; i.e. how do we form protein expression values from measurements of peptide spectrum matches (PSMs) or MS1 features.

## Shotgun quantification is all the same

Many people might think [spectral counting](https://pubs.acs.org/doi/abs/10.1021/ac0498563), [intensity-weighted spectral counting](https://pubs.acs.org/doi/abs/10.1021/pr401017h) (also [this reference](https://onlinelibrary.wiley.com/doi/abs/10.1002/pmic.200700426)), [accurate mass and time](https://onlinelibrary-wiley-com.liboff.ohsu.edu/doi/full/10.1002/mas.20071) (AMT) tag methods, [MaxQuant LFQ](https://www.mcponline.org/content/13/9/2513.short), [MaxQuant iBAQ](https://www.nature.com/articles/nature10098), [SILAC](https://www.mcponline.org/content/1/5/376.short), [iTRAQ](https://www.mcponline.org/content/3/12/1154.short), and [TMT](https://pubs.acs.org/doi/abs/10.1021/ac0262560) are all rather different quantitative proteomics techniques. If you take a step back, no - a bigger step, keep going, STOP!, you will see that all shotgun quantitation is just weighted spectral counting. First, I need to climb onto my [soapbox](https://en.wikipedia.org/wiki/Soapbox) and


> **Rant against ratios:** We are going to be spending a lot of time talking about rolling up lower level measurements to higher levels (like proteins). We are not going to talk about ratios, median ratios, signal-to-noise ratios, golden ratios, the 80:20 ratio, pi, or any other kind of ratios. Ratios and statistics do not mix (ask any statistician). We want quantitative proteomics to be serious quantitative work; something that uses real statistics so that we can say this result is **[statistically significant](https://www.nature.com/articles/d41586-019-00857-9)**.

> **Ratio drawbacks:** There are few reasons to use ratios and many reasons not to use them. Measurements are made on some natural scale that has physical meaning. We often have good intuition about the data in its natural scale. Ratios change the natural scale. Ratios compress the range of the numbers. Half of the ratios are between 0 and one. The other half are between 1 and infinity. This asymmetry in numerical values is **fixed** by taking the logs of the values. This further compresses the range of values. No one on this planet has any intuition about numbers in reciprocal or logarithmic spaces. There are no good reasons to work with ratios (in most cases) other than this being the poor way that quantitative proteomics has been done in the past.

What all of these methods have in common is that we have proteins in our samples, but we do not make measurements of proteins. We choose to digest the proteins and measure the liberated peptides instead. The question is what are the relationships of the measured peptides to the proteins originally in the sample? The [answer is not 42](https://en.wikipedia.org/wiki/The_Hitchhiker%27s_Guide_to_the_Galaxy_(novel)); it depends on a lot of things.

## Context is everything

A (tryptic) peptide is pretty short, so the amino acid sequence may not be **unique**. Unique is defined in **this context** as the peptide mapping to a single gene product (a protein). It is not a simple case of peptide length for uniqueness; although longer peptides are more likely to be unique.  Nature, unlike humans, tries to recycle things. There are motifs in proteins that do useful things and these tend to be conserved. We can have conserved motifs between different proteins within the same organism and we can have conserved motifs between species.

> It is becoming more rare to have data from a species that does not have a sequenced genome. It is also possible now to create custom genomes for the samples being studied. One context is the genome of the samples under study, another is the available sequenced genome in one of the genomic sequence warehouses. There are additional contexts that come from predicting the protein sequences coded by the genomic sequences. The FASTA file used in analyzing your data is not the only genomic context for your samples.

Higher eukaryotic organisms have genes that produce multiple, similar protein products to increase functional diversity. These protein isoform families can liberate many peptides that are the same. There can be **more complete** (isoforms present) or **canonical** (single gene products) FASTA database choices to make. Most organisms also try to cheat death by having multiple protein coding genes for critical housekeeping functions. The duplicated genes can produce identical or nearly identical gene products. Many peptide sequences identified in shotgun studies can come from more than one protein.

What is the relationship of these **shared** peptides to the proteins, and what can we say about the measured values for these shared peptides? The bottom line is that the **information content** is different for unique and shared peptides. In next generation mRNA sequencing, they just toss out the shared reads. We can have a lot of shared peptides in proteomics, so we generally avoid rejecting them.

Shotgun quantification boils down to how do we define shared and unique peptides, in what context, and what do we do with shared peptides. This becomes a tension between quantitative resolution (how finely can we quantify proteins or even parts of proteins) and trying to recover as much quantitative information from the shared peptides as possible. We worry about the later situation because many proteins in shotgun experiments are not abundant and we have limited measurements for them. Discarding shared peptides too aggressively can compromise the data quality. How we make decisions about the shared peptides depends on the [protein inference](www.mcponline.org/content/4/10/1419.short) process.

This is how shotgun quantification and protein inference are intimately coupled. You might have a biological question where maintaining lots of quantitative resolution is much more important than trying to reduce noise and simplify the results. Lucky you! You can stop reading this. However, most pipelines such as mine, or Proteome Discoverer, or MaxQuant, or Scaffold are going to make efforts to recover quantitative information from shared peptides. Keep reading if you want to gain some understanding of how that is done.       

## Relevant pipeline steps

I did a blog post [about pipelines](https://pwilmart.github.io/blog/2019/09/08/Keeping-pipeline-flowing) where typical processing steps were outlined. Most upstream steps are about filtering PSMs with some acceptable error rate, typically 1%. If you had 250,000 MS2 scans and could identify 40% of those (these are pretty typical numbers for shotgun experiments on Orbitraps), you would have 100,000 PSMs after filtering out most of the incorrect matches. A 1% FDR implies that there would be 1000 incorrect PSMs. You do not know which of the 100,000 PSMs might be the 1000 incorrect ones. Would you trust the quantitative information from those 1000 PSMs?

## Protein inference

Generally, we combine proteins having (mass spec) indistinguishable sets of matching peptides into single groups, and remove any proteins whose set of peptides can be explained by other proteins (with larger peptide sets). This constitutes basic parsimony principles. We (usually) do not have more than one identical protein sequence in the FASTA protein database file. While the protein sequences tend to always be unique, because we only observe partial sequence coverage, the observed peptides may not be capable of distinguishing proteins. Our ability to distinguish proteins is reduced as sequence coverage decreases. We know that PSM counts are correlated with abundance, so we have better odds of distinguishing abundant proteins (where coverage is higher) compared to lower abundance proteins.

Each biological sample starts with intact proteins that get digested in peptides. Strictly speaking, we should map peptides to proteins separately for each biological sample. We do not  know for sure if peptides coming from different biological samples came from the same protein because we do not know what proteins are present in each different sample yet. Unfortunately, many proteins have few peptides and whether or not those peptides are distinguishable can vary sample-to-sample. This can make aligning identifications between samples challenging.

A strategy that [kills two birds with one stone](https://idioms.thefreedictionary.com/kill+two+birds+with+one+stone) is to collectively map all of the peptides from all of the samples in an experiment-wide protein inference process. This gives more sequence coverage to make identical set and subset calls. It also takes care of aligning protein identifications across all samples in the experiment. We may need to do some per sample cleanup if we have some minimum protein identification criteria to meet. This strategy tends to minimize protein grouping in basic parsimony logic. It can lead to proteins that are indistinguishable in a given sample being reported as distinguishable despite each protein has no unique peptides (this seems rare). Every choice has its consequences.

## Protein error control

As we have moved through the data processing steps, we have reduced the number of PSMs by rejecting the majority of the incorrect matches. We have mapped those peptide sequences to the protein sequences that were in the FASTA file and followed some parsimony rules to simplify the protein list. Remember that we still have incorrect PSMs that might lead to incorrect protein identifications to deal with.  

There are two ways to control the number of incorrect proteins: _ad hoc_ ranking functions or basic probability theory. _Ad hoc_ ranking functions approach the problem like it is similar to the statistical treatment of PSM scores. It is not. While a target or decoy protein counting does give some ballpark estimate of the protein error rate, there are insufficient numbers of target and (especially) decoy proteins for any valid statistical assessment. You would need to know the decoy protein score distribution and that is not possible because nearly all incorrect PSMs have been filtered out.

The process by which incorrect peptide sequences manifest as incorrect protein sequences is not at all like how incorrect PSM scores end up in the correct pile. We all know that the number of peptides per protein is somehow important. That is how the **two-peptide rule** came to pass.  The Supplemental materials for the [MAYU paper](https://www.mcponline.org/content/8/11/2405.short) worked out the basic probability equation.

The formula can be simplified for the typical proteomics case. If we just want to get the gist of how numbers of incorrect proteins depend on numbers of incorrect peptides, we can make some approximations. The number of incorrect peptide sequences are usually much smaller than the number of proteins in the FASTA database. The numbers of incorrect peptides are typically a few hundred to a thousand. The number of proteins in protein databases are typically many tens of thousands. Consequently, one of the terms in the hypergeometric probability equation will be so close to 1.0 that we can drop it. Truncated factorials can be replaced with power expressions (100 x 99 x 98 is pretty close to 100^3). There is explicit dependence on the number of peptides per protein. The number of incorrect proteins drops off so rapidly with increasing numbers of peptides per protein, that explicit forms for 1, 2 or 3 peptides per protein are all that are relevant.

![protein error equations](https://pwilmart.github.io/images/protein_errors.png)

**There are a few key points:**

- there is a direct dependence on number of incorrect peptides
- there is an inverse relationship with the protein database size
- formulae predict the number of incorrect proteins
- single peptide per protein IDs have a one-to-one error relationship
- three peptides per protein error numbers are typically very small
- two peptides per protein is [**the sweet spot**](https://idioms.thefreedictionary.com/the+sweet+spot)

**There are some caveats**
- lengths of the protein sequences are ignored
- database size should be proportional to theoretical peptide search space
- incorrect matches to correct proteins are ignored

> These formulae are in the ballpark if the number of proteins identified in the sample are a reasonably small fraction of the database size. Scalings of the estimates have to be done if the number of protein IDs become too large compared to the target database.

With the typical numbers mentioned above, the 1000 incorrect PSMs would result in 1000 incorrect proteins if we allowed single peptide per protein IDs (without ranking functions and an FDR analysis). We would only have a handful of incorrect proteins (estimate of 25 for a 20,000 sequence protein database) with the two-peptide rule. Protein ranking functions are untested and could suffer from inaccurate error estimates given the fluctuations in counts of small numbers. The two-peptide rule is derived from simple probability theory and effectively controls the number of incorrect proteins.

## Shared peptides are a moving target

The proteins that the measured peptides suggest could be present in our sample are not a simple subset of the starting FASTA file. We have peptide ambiguity and we have partial sequence coverage. We have grouped indistinguishable proteins, distinguishable proteins, and some proteins that we thought we did not have evidence to report. We also have some peptide errors.

> See the 2005 definitive paper on [protein inference](https://www.mcponline.org/content/4/10/1419.short) for definitions of the terminology.

We started by mapping peptide sequences to the protein sequences in the FASTA file. Peptides that mapped to a single protein sequence were unique, and those that mapped to more than one were shared. We formed peptide sets for each of the proteins pointed to by any of the identified peptides. We compared peptide sets to determine distinguishable proteins, indistinguishable protein groups, and remove proteins with subset peptide sets. That defines a new context for defining shared and unique peptides with respect to the parsimonious list of proteins inferred in the experiment. Many (most) of the peptides in indistinguishable protein groups, which were all shared in the context of the FASTA file, become unique peptides in the new context.

These days most datasets are enormous. We identify a large number of PSMs at fixed false discovery rates. That can lead to not insignificant numbers of incorrect peptides. These large numbers of identified peptides result in large number of proteins being inferred (reports of 5000 to 10000 proteins in studies are not uncommon). It becomes more likely that correct proteins can have incorrect associated peptides. Those stray matches can throw a wrench into the parsimony logic. We might have identical peptides sets no longer being identical or not remove a subset because of an incorrect peptide match. We need to add a little fuzziness to the parsimony logic.

We can define extended parsimony logic to allow grouping of proteins having **nearly** identical peptide sets. We can remove proteins if their peptide set are **nearly** fully contained in larger peptide sets. We can circle back to the two-peptide rule the define **nearly**. If we have less than one or two unique peptides and a lot of shared peptides, we can group those proteins (or remove as a subset). We can still have abundant, highly homologous housekeeping proteins where we may have mostly shared peptides and just a few unique peptides. When the shared peptides dominate, it might be safer to lump the proteins into one big family, and quantify the family rather than the family members.

Once again, we redefine shared and unique peptides after doing the extended parsimony analysis. This also changes many formerly shared peptides to unique peptides and increases the quantitative information content of the experiment that we can use.

There are alternative ways to group similar proteins that are implemented in Scaffold and Mascot. These may seem like some really bad ideas given what we know about **some** proteins in **some** biological systems. The truth is that the alternatives are worse. Without grouping, we have predominantly shared peptides that we cannot use for quantification. The leftover unique peptides may be low abundance or even incorrect matches. The quantification would be unreliable in many cases.

The razor peptide concept implemented in Proteome Discover, MaxQuant, and other pipelines is kind of a one good protein and all of the others end up bad situation. It is like one protein of a family gets a gigantic job promotion to the **razor** protein. It gets all of the shared peptides. The other family members get little or nothing. They are left with the shirt on their backs (their unique peptides) to fend for themselves. One protein ends up with a really nice house on easy street and the others are left scrounging dumpsters. It can be difficult in results files to figure out which are the **poor** proteins with **poor** quantitative information.   

## Where are we at and where are we going?

The discussion so far has not had included much of anything about quantitative data. It has all been about how we define the relationships between peptides and proteins in shotgun experiments. The process of going from MS2 spectra to PSMs to peptides to proteins creates a variety of contexts that affect how we defined shared and unique peptide status. At the end of all of this, we finally know enough about the information content of each peptide that we can decide what to do with their associated quantitative data.

We are now faced with many choices about how to proceed with the quantitative data. A motivation for this blog was to shed some light on how discovery quantification differs from more targeted quantification strategies. In discovery phase, we are trying to learn more about our system so we can form more directed hypotheses. Only when we have some concrete hypotheses can we formulate more precise questions to guide targeted studies. There may be many situations where the strategies discussed below will be wrong and lead to incorrect conclusions. A very interesting question is whether or not there is some signature in discovery data that can suggest when its assumptions are being violated. The first thing is to try and describe how to aggregate PSM-level quantitative data into protein-level summaries. We have to understand that before we can dissect the fundamental differences between discovery and target quantification strategies.

This is where we come to forks in the road. There are many choices for how to relate measurements of the abundances of pieces of the protein to estimates of the whole protein abundance. One way to group the choices is by what kind of abundance they seem to be measuring. Common abundance scales used in proteomics are molar concentrations and percentage scales (weight or volume). Things like the [top 3 method](https://www.mcponline.org/content/5/1/144.short), [NSAF spectral counting](https://www.pnas.org/content/103/50/18928.short), or [iBAQ](https://www.nature.com/articles/nature10098) are more like molar concentrations. Spectral counting, MaxQuant LFQ, TMT in MaxQuant or Proteome Discoverer or my pipeline are more like weight percentages.

We often need large scale peptide separations in discovery phase because we do not know ahead of time if the interesting players will be high, medium, or low abundance proteins. Methods that require more specific measurements of an analyte's signal need to have more reproducible chromatography conditions that can be at odds with large-scale separations. Weight percentage methods are essentially lump everything we measure about a protein together, so they are more forgiving of large-scale separations.

## How to "lump everything together"

In [spectral counting](https://pubs.acs.org/doi/abs/10.1021/ac0498563), the numbers of PSMs associated with a protein are proportional to the protein's abundance. In a great oversimplification, this is because instruments have a linear measurement range and a lower limit of detection. If a protein is more abundant, all of its pieces will tend to be more abundant. More of the pieces will have signals that fall above the detection limit. As the overall protein abundance decreases, more of the pieces fall below the limit leaving fewer above the limit. The figures below are a protein with 20 peptides. The signals are random values between zero and 100,000 for the **Full** plot. The red dotted line represents the detection limit. We have 13 peptides above the limit.        

![full](https://pwilmart.github.io/images/full.png)

Now we have cut the overall intensity values in half, and we have 9 peptides above the dotted line.

![half](https://pwilmart.github.io/images/half.png)

If we cut the signals in half again, we now have just 5 peptides. Any more decrease in signals and we will no longer see the protein at all.

![quarter](https://pwilmart.github.io/images/quarter.png)

I made the argument at the beginning that shotgun quantification is just weighted spectral counting. So we will do the most simple thing that we can and just sum up the intensities for all of the peptides that are above the detection limit into protein total intensities. In the Full case, we sum up 13 intensities, in the Half case we sum up 9 intensities, and in the Quarter case we sum up 5 peptides.

Do we sum up all signals from all PSMs all of the time? No, we want to exclude any shared peptides. In what context are we defining shared and unique you might (and should) ask? In whatever final protein context that we deemed appropriate for the results. Remember that the changing context involves tradeoffs. In many cases, if we sacrifice a little protein quantification resolution, we can dramatically increase the fraction of peptides that are unique (in that context). Check out Section 3.4 on Quantitative Information Content in [Ravi's thesis](https://digitalcollections.ohsu.edu/concern/etds/c534fp149) for more details and some examples.

Data quality, as measured by a few metrics, improves as we aggregate from the PSM to the peptide level and as we go from peptides to proteins. We look at these improvements in [this dilution series](https://pwilmart.github.io/TMT_analysis_examples/MAN1353_peptides_proteins.html) notebook. That data is one mouse brain sample labeled with 6-plex TMT and diluted in 25:20:15:10:5:2.5 factors across the channels. There are 60K PSMs, 20K peptides, and 2700 proteins. The protein total intensities span 7 orders of magnitude in intensity. It is kind of the product of the 2-3 orders of magnitude in spectral counts times the 4-5 orders of magnitude of individual PSM intensities.

## Does this protein rollup work?

That is [the 64 thousand dollar question](https://en.wikipedia.org/wiki/The_$64,000_Question). There are a lot of proteins in the world and biological systems seem to enjoy finding unusual ways to solve problems. There are no doubt hundred (maybe thousands) of journal articles detailing cases where all of this falls flat on its face. If you can think of your favorite example and pinpoint exactly where these protein rollup efforts fail, the we have made some real progress. Discovery quantification is not the answer to every question. Understanding its compromises and tradeoffs is crucial to knowing when **not** to use it. There are tradeoffs to targeted quantification and it is not the solution to every problem either.

The above question deserves some kind of answer, however. How can we answer it in a clear way? This is truly one of the challenges in most proteomics method development. Common ways to try and answer such questions are to make some artificial test mixtures where the know outcome is thought to be known. These are usually spike-in mixtures where they address quantification dynamic range and accuracy rather than overall sanity. These typically fall into the positive control category. The most common drawbacks are that the mixtures can be too simple to mimic the real biological samples, and many times the spike-in proteins are medium to higher abundant proteins. Most of the warts in proteomics methods are from the lowest abundance proteins where the information gets too sparse to test anything.

If you have done any normalization methods work, you learn to ignore the proteins that are changing in abundance. They are just a nuisance. The meat and potatoes are the proteins that are not changing in abundance. They are the critical proteins that nearly all normalization methods absolutely have to have to work. The sources of variation that we are trying to remove in the normalization methods cause the measurement of a constant (the expression level of an unchanging protein) to be variable between samples. We adjust the data in ways designed to make those constant values converge. These unchanging proteins are a negative control that we can exploit in the actual experiments we are most interested in studying. We can do sanity checks on real data without having to rely on test mixture data that may be a poor proxy for the real case.

## Some sanity checks from TMT data

![E. coli background](https://pwilmart.github.io/images/E_coli_no-change.png)

---

---

## Extra content from previous blog

I want to clearly state that detecting proteoforms, detecting alternative slicing, detecting disease associated mutations, detecting novel protein processing, and detecting novel post-translational modifications are all **extremely important**. However, those are advanced proteomics topics. They are much harder than you have been told. The game is not over once you find one putative PSM that suggests something interesting. That is really more like the pregame warmup. You have to have a hypothesis and an experimental design with controls. You have to **validate everything**. You have to bust your butt to try and prove yourself wrong.

## Do not bite off more than you can chew

A protein expression study is about comparing protein abundance measures between samples. Shotgun techniques are looking at pieces of proteins. The unmodified peptides from a canonical sequence will constitute the vast majority of the peptide signals from each protein. I do not want to burst bubbles here, but that is the reality. Modifications are substoichiometric without enrichment. Most protein isoforms do not have detectible distinguishable peptides unless those proteins are very abundant and the sequence coverage is high. There is nothing to be gained by looking for things that are not there or not abundant enough to matter.

Every rule has exceptions. However, questions about isoforms, variants, PTMs, etc. are separate questions from differential protein expression. They should be explored in independent, separate analyses. If we had extensive PTMs in our samples and the PTMs varied by condition, then not specifying PTMs in the database search will bias the quantitative results. Studies with samples like that will be rare. Some characterization of samples for PTMs and how they vary by condition needs to be done first to see if a quantitative bottom up study can be done for those cases. If PTMs are going to be important, the more abundant one have to be determined and the quantitative experiment altered accordingly. A quantitative shotgun experiment has limitations and is not always the right thing to do. That is why there are other methods like top-down and targeted analyses.

Use a protein grouping or clustering method to combine protein families that have high sequence homology instead of a razor peptide method. Some housekeeping genes have multiple coding regions that make identical or very similar sequences. This redundancy helps keep you alive. It can mean that protein members of families like alpha-tubulins, beta-tubulins, actins, histones, etc. can have few **non-shared** tryptic peptides. Razor peptide methods pick one protein of the family to get all of the shared peptides. The other family members just get a handful of their unique peptides and no shared peptides. That means that one family member gets large quantitative signals and the other family members get small, unreliable values. It is better to remove the poor quality family members by grouping the entire family into a single entity.

## Keep it simple

We need to have robust, simple lists of proteins and their quantitative estimates to compare many samples in typical studies. Aligning protein identifications between samples can be challenging and overly complete (large) protein FASTA files can make that harder. Some protein grouping steps can simplify protein lists when canonical databases are not an option.

Simplified protein lists not only make the differential expression testing easier to do and interpret, most annotation information is gene-centric. It makes sense to have a one gene, one protein relationship when adding annotations. If ortholog mapping needs to be done to a better-annotated species, mapping canonical sequences to canonical sequences is much easier. Most downstream steps rely on minimal ambiguity to work well. Yes, there is some loss of information associated with minimizing ambiguity. There is also some compromise in quantifying **proteins** from their **tryptic peptides**. Some consistency in the compromises of the analysis across the pipeline steps is the workable approach.

## Final Thoughts

I am not proposing that we all do only the lowest common denominator experiments. What fun is that? I am proposing that there is no such thing as a **one and done** analysis. Start with a simple search if doing protein expression and see what you get. Explore other things with additional analyses. Are there non-tryptic peptides in my sample? How abundant are they? Can I ignore them? Are the lots of PTMs in my sample? How abundant are they? Can I ignore them? Are there different protein isoforms present? How abundant are they? Can I ignore them?


---

Phil Wilmarth, September 20, 2019.