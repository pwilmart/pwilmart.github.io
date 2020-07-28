---
layout: post
title: "Houston, we have a discrepancy!"
date: 2020-07-28
---

## MSstatsTMT and I seem to disagree...

### A little back story:

Recently, a [new paper](https://www.mcponline.org/content/early/2020/07/17/mcp.RA120.002105) from Olga Vitek's group describes a new Bioconductor package called [MSstatsTMT](https://www.bioconductor.org/packages/release/bioc/html/MSstatsTMT.html) for analysis of TMT experiments. Among its features is support for multiple-plex TMT experiments. The package starts with low level TMT data, such as the PSM export file from Proteome Discoverer or the evidence.txt file from MaxQuant. A complicated linear modeling approach is used for the statistical testing.

TMT reagents have increased in multiplexing capacity steadily over the years from 6 to 16. Despite increased numbers of channels, many experiments can require more than one plex (plex: a single TMT labeling kit) to handle all of the samples. A simple experimental design and methodology for normalizing (sometimes called bridging) between plexes was described by Plubell, et al. in this [2017 paper](https://www.mcponline.org/content/16/5/873).

I first conceived of the Internal Reference Scaling (IRS) method in 2015, not long after our Thermo Fusion Tribrid was installed and running. Deanna and Nathalie's project was the first to get published using the method. I think that IRS is the best method for processing multi-plex TMT data so far. I eventually learned about Jupyter notebooks and GitHub and have had TMT analysis examples publicly available since 2018. You can find a [roadmap to the content here](https://github.com/pwilmart/Start_Here). I have added content regularly over time and there are many analysis examples covering a variety of datasets.

### A common dataset:

One of the analyses I added this spring while working from home is some [metaplastic breast cancer data](https://github.com/pwilmart/Metaplastic-BC_PXD014414) from Alexey Nesvizhskii's group. I started with the RAW files from PXD014414 at PRIDE and processed the data with my open source [proteomics pipeline](https://github.com/pwilmart/PAW_pipeline). A detailed Jupyter notebook analyzing the major groups (normal tissue, triple negative breast cancer, and a few types of metaplastic breast cancer) can be found [here](https://pwilmart.github.io/Metaplastic-BC_PXD014414/PXD014414_comparisons_major.html). The data was SPS MS3 acquisition of 27 samples in 3 TMT 10-plexes.

This data was also analyzed in the MSstatsTMT paper (called the "Human_3mix_balanced" dataset) and summarized in Figure 8. The authors also implemented a workflow similar to those I use called the *Sum+IRS+edgeR* and tallied some statistical testing results numbers for this data that I could compare to my re-analysis results.

### The test results:

**From MSstatsTMT manuscript:**

Method|Testable Proteins|Normal vs Triple Negative|Normal vs Metaplastic|Triple Negative vs Metaplastic|Total
------|-----------------|-----------------------------|-------------------------|----------------------------------|-----
Ratio-Median-Limma|5728|243|399|5|647
Sum-IRS-edgeR|3870|235|313|9|557
MSstatsTMT|5728|202|381|0|583

There is general agreement between the three testing methods with their implementation of my methods coming out on the bottom. The starting data for their analysis were not the RAW files. They got MSFragger results files from Alexey Nesvizhskii's group. In my re-analysis, I had about 6% more identified PSMs with my processing, but generally similar numbers of identifications to MSFragger.

I did an IRS adjustment using the single reference channel in each plex (I strongly recommend using two channels) to put all 27 samples on a common intensity scale. The numbers in the table above are at a 5% FDR threshold. I checked my re-analysis notebook and I had 1644 total differentially expressed proteins - a factor of **2.8** more than MSstatsTMT. That seemed improbable to me. The above table had three methods in rough agreement with far fewer candidates than I was seeing. Such a large discrepancy made me worry that my analysis methods must somehow be inflating the candidate number despite numerous QC check in my notebooks.

The manuscript suggests that edgeR has to be used in my workflows. That is not at all true. The *Sum+IRS* steps simply create a properly normalized data table suitable for any downstream statistical testing methods. I ran several alternative methods on the data from my re-analysis. I felt I needed some additional validation that my analysis numbers were okay before commenting on any discrepanacies.

**My re-analysis:**

Method|Testable Proteins|Normal vs Triple Negative|Normal vs Metaplastic|Triple Negative vs Metaplastic|Total
------|-----------------|-----------------------------|-------------------------|----------------------------------|-----
t-test, BH|4132|399|1068|0|1467
t-test, Bonferroni|4132|1|84|0|85
edgeR exact, BH|4132|628|997|19|1644
edgeR exact, Bonferroni|4132|72|205|8|285
edgeR glm, BH|4132|554|941|2|1497
limma, BH|4132|521|820|0|1341
voom-limma, BH|4132|515|816|0|1331

The t-test is a plain 2-sample, 2-tailed test with equal variance. I can also use any of several multiple-testing correction algorithms. Benjamini-Hochberg is the best choice (no need to rehash that debate). Some folks might think that statistical test p-values come from a burning bush on a stone tablet. They are quite variable. The more conservative Bonferroni correction dramatically reduces candidate numbers. That is why there are all of the other multiple testing algorithms. The point is, you have the freedom to use all kinds of statistical methods on the table of numbers.

### That is some discrepancy!

The numbers of DE candidates from Figure 8A (the first table above) are in the same ballpark and have an average total of 596. The number for my starting data are also similar (ignoring the Bonferroni correction numbers) and average 1456. That is a factor of 2.5 more DE candidates. The number of putative testable proteins is also rather different: 5728 versus 4132. My testing input starts with 72% as many proteins. The extra 1600 testable proteins for MSstatsTMT might have been expected to add to the DE candidate numbers.

This situation is like throwing three darts at a dartboard to see which is closer to the bullseye. However, the three darts all completely missed the dartboard and stuck in the wall instead. You can still see which one was closest to the bullseye, but what is the point? Why are the results in MSstatsTMT so different?

For this dataset, the starting data (the RAW files) is the same. Things diverge after that. We have two different search engines (MSFragger versus Comet) using different FASTA files (a large, redudant database versus a canonical database). The other (big picture) search setting are similar. PSM FDR control is a little different (modeling versus empirical). There is quite a bit of filtering of data described in the MSstatsTMT paper. Long story short, the starting data for the statistical testing presented in Figure 8A is different from the starting data for the statistical testing that I did. I think the different starting data is where the discrepancy lies.

### Shared and unique peptides are the key

Section 3.4 in Raviteja Madhira's [Masters thesis](https://digitalcollections.ohsu.edu/concern/etds/c534fp149) discusses the concept of quantitative information content (QIC). The underlying theoretical peptide redundancy/structure of a particular protein FASTA file and rules for protein inference are intimately coupled. You have to understand the underlying structure of protein databases to do proteomics. If you think you would benefit from more reading on this topic, I highly recommend Ravi's thesis. You can also find useful information in the [fasta_utilities repo](https://github.com/pwilmart/fasta_utilities).

The basic concept of QIC (defined as the fraction of unique PSMs out of the total PSMs for a dataset) is that what we call unique or shared peptides depends on the context. We can only use unique peptides in quantitative studies, so the QIC is rather important. What are the relevant contexts? We start with the context of the original FASTA file. Peptides are unique if they map to just one protein sequence and shared if they map to more than one. Then we do protein inference according to the eloquent rules outlined in this [seminal paper](https://www.mcponline.org/content/4/10/1419.short) from 2005. After these basic parsimony rules, we have a list of the likely proteins (and groups of proteins) present in the sample. (Those lists used to be considerably smaller that the original FASTA file.) It is common practice to redefine the context for shared and unique peptides to the list of inferred proteins. This can change many shared peptides into unique peptides (in the new context) and increase the QIC. There are additional protein grouping methods, such as clustering in Mascot, grouping in Scaffold, and extended parsimony grouping in my pipeline. These can make yet another context where more shared peptides can become unique and QIC further increased.

What does this have to do with the discrepancy you ask? The data used in the MSstatsTMT paper comes from the MSFragger search described in the [original publication](https://www.nature.com/articles/s41467-020-15283-z). The FASTA file was a human proteome with 74K sequences. There are only about 20K human genes (and 20K sequences in the canonical Swiss-Prot sequence collection). The extra 54K sequences in the FASTA file used in the MSFragger search are alternative protein isoforms. This creates a very high degree of peptide redundancy in the context of the original FASTA file. If only unique peptides in that context were used by MSstatsTMT, that could easily result in a drop in QIC by more than a factor of 2. The impact of context on QIC depends both on the protein FASTA file and on the proteins that are actually in the samples.

I have a suspicion that only unique peptides were used for quantitation in MSstatsTMT and that the context was the FASTA file. I suspect that a large decrease in QIC had a big effect on the statistical testing for this data where sample-to-sample variability is pretty large. I have a blog entry about [putting Humpty Dumpty back together](https://pwilmart.github.io/blog/2019/09/21/shotgun-quantification) that digs into understanding quantitative shotgun studies.

### Why is MSstatsTMT starting with PSM-level data?

There are many upstream processing steps to produce a count table in an RNA-seq experiment. I do not know if every statistician that tests for differential gene expression asks for all of the lower level data that contributed to the final count table, but every time I have given a protein expression table to a statistician at OHSU, the first thing they ask for is the PSM data. This is always a source of friction. Protein inference and protein summarization is a complicated topic that few statisticians have studied and are qualified to tackle. Note: many are perfectly capable of doing so.

I have never understood why there is such fascination with noisy scan level data. If you are curious about what PSM, peptide, and protein level TMT data is like, please see this [dilution series notebook](https://pwilmart.github.io/TMT_analysis_examples/MAN1353_peptides_proteins.html). Aside from the clearly better protein-level data, we are talking about shotgun proteomics data. The numbers of PSMs per protein are all over the map. They depend on the samples, the separation scheme, and the instrument method. In most samples, there are a few proteins with many PSMs, and most of the proteins with one, two, or three PSMs. Concepts like median value summarization do not make much sense to me until you get to at least 5 observations. Summation is simple and effective; it is essentially a weighted measure (larger values contribute more to the totals).

Once you let go of the notion that important variance information is going to be lost if you sum up the PSM data, then you can see that a protein intensity table looks a lot like a gene count table. Then you realize that you can use established Bioconductor packages like limma and edgeR. You do not have to spend time trying to invent your own wheel. What we need is more focus on extracting biological meaning from the DE candidates, not another tool for DE candidate determination. (This is like "yet another search engine" work). We need analysis methods that show more quality control details about the data before we do the statistical testing. What is the point of doing statistical testing on poor data? There are also many aspects of the statistical testing that need more QC and visualizations to make sure things are okay (p-value distributions, CVs by condition, cluster views, possible outlier detection, etc.). Most analysis methods try to hide these things from the users.

### Conclusions

I would not recommend using MSstatsTMT in its current form starting with PSM-level data. Protein inference concepts and protein summarization could probably be fixed in MSstatsTMT. I shudder to think about trying to code protein inference in R, though.

This raises the bigger issue in my mind. Both Proteome Discoverer and MaxQuant provide protein level reporter ion summaries based on implemented protein inference algorithms with correct definitions of shared and unique peptides. I do not understand why one would not start with the protein level data for the DE testing. Maybe there is that option in MSstatsTMT. I have not read everything at Bioconductor. I am just going from what was in the manuscript. Then again, you could just use limma starting with a protein summary file.

There is some discussion of missing data modeling in the MSstatsTMT manuscript. There is not really any missing data problem in TMT data, and perceptions that is a problem are based on poor assumptions. The lowest abundance proteins contribute the vast majority of the missing data. However, those proteins are not abundant enough to be quantified. If they are excluded, the missing data problem mostly goes away.

I have benchmarked my pipeline against both Proteome Discoverer and MaxQuant. It just works a lot better. You will probably get better TMT results if you use my pipeline and analysis methods.     

---

Phil Wilmarth
July 28, 2020
