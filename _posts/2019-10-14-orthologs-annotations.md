---
layout: post
title: "Ortholog mapping and annotations"
date: 2019-10-14
---

Your proteomics experiment worked really well. You identified more proteins than the number of lies conservative politicians can tell in a day. The quantification even worked - or at least the 95% confidence interval surrounding the probability that it worked did not include zero. Now you want to know something about those proteins, something like: what are they and what do they do?

It seems every organism under the sun (and the sea) has a sequenced genome these days, so you can do amazing proteomics experiments on all kinds of samples. What do you do with all of the "uncharacterized proteins" that you identified? What is its uncharacterized biological function? Maybe it has other uncharacterized isoforms. Maybe it is involved in an uncharacterized signaling pathway. What if it forms an uncharacterized protein complex with other uncharacterized proteins to perform an essential uncharacterized function?

You try to find out the answers to these questions using any of the many online resources (NCBI, UniProt, Ensembl, etc.). You run into the proverbial brick wall. Who knew that we (the collective "we") do not know anything about sea lion proteins? Or horse proteins, or rat proteins, or chicken proteins, or cow proteins, or lettuce proteins, or just about any organism you can think of.

If you study only mouse or human samples, you have probably stopped reading by now. Those organisms have well annotated proteomes and many bioinformatics tools can tell you many interesting things about human or mouse proteins. Many of those tools can't do much with any other organisms. If your samples are not human or mouse, are you up a creek without a paddle?

What are your options? There are accession mapping tools. Some are online or part of online resources, some are Bioconductor packages, and some are part of commercial packages. What must these tools be doing? They are ultimately relying on sequence alignments between your organism of interest's proteins and the proteins of some better annotated organism (this is a code word [or dog whistle if you like politics] for human or mouse). Someone somewhere at some time was not using a computer to make bitcoin and, instead, did a lot of sequence alignments between all sorts of species. Based on the results of these alignments, they determined that protein X in species A was "the same" as protein Y in species B. Protein X and protein Y are known as orthologs ([ortholgos, paralogs, and homologs](https://biology.stackexchange.com/questions/4962/what-is-the-difference-between-orthologs-paralogs-and-homologs) are essentially synonymous for this casual discussion).

This might be a great solution to your problem. Or it might be the start of a very frustrating exercise. There are FASTA protein databases available from many sources (NCBI, UniProt, Ensembl, to name a few) for the same organisms. They do not have the same accessions. Protein entries at all of the major sequence collections are under constant (do not confuse constant with frequent) revision. Protein accessions can (and do) change over time. Protein sequence collections for specific organisms can change. It takes time and money to generate these accession maps, so they are not being continually updated. How many of your proteins can have no match before the remainder are too biased to give you the correct biological picture?

Personal computers are about a thousand times a thousand times faster than they were about 5 minutes ago (okay - that is a slight exaggeration). What about just doing the sequence alignments on the fly and not relying on someone else to do that for us? NCBI distributes the BLAST sequence alignment program for multiple platforms. The BLAST program has a nice feature where you can align all of the sequences in one FASTA file against all of the sequences in a second FASTA file in a single command line call. The scripts in the [PAW_BLAST](https://github.com/pwilmart/PAW_BLAST) repository do just that.

The inputs are two FASTA files. The first is the **query** database. It can have one or more sequences. The second FASTA file is the **hit** database. Each sequence in the query database is aligned with every sequence in the hit database. The results are saved in a large XML file. This file is parsed to find the top matches. A common definition of orthologs in this context is proteins that are reciprocal best matches. With some logic and book keeping, we can find those reciprocal best matches and make a summary table. BLAST does not do full length sequence alignments (which might be better for this), so we can sometimes have rather short alignment regions. We need to do some categorization of the quality of the match. All of that is detailed in the repository's README documentation.

We need to think a little about the two FASTA files. What do we want those to be? Let's take a step back. We are trying to find similar genes across species. Any excess redundancy in the gene lists is going to make this a lot harder to do with any confidence. We really need protein FASTA files that are in the **one gene, one protein** category. UniProt makes canonical reference proteomes for many species (see the [fasta_utilities](https://github.com/pwilmart/fasta_utilities) for more details and scripts to aid in this). We will eventually be getting annotations for human or mouse (and arabidopsis for plants), and UniProt only has annotations for Swiss-Prot (Reviewed) proteins. Our choices for the **hit** database will be pretty simple (human or mouse; canonical or Swiss-Prot).

We have several options for the query sequences. We could try and determine the orthologs before we do the database searching. That will work best if the database is a canonical sequence collection (no isoforms). Comparing 20K sequences to 20K sequences is very doable, although it can take a few hours. We would need to parse the ortholg map and modify the FASTA header lines to add the ortholog information. That option does work.

We can alternatively use whatever FASTA file we want in the search and compile the parsimonious list of proteins. The basic steps in protein inference will simplify the final protein list and get it closer to a one gene, one protein state. Some pipelines (Scaffold, Mascot, and PAW) have additional protein grouping algorithms. These do a good job of making the final list of proteins as minimally redundant as possible. We need to take the final list of inferred proteins and our starting FASTA file, and then extract protein sequences into a subset FASTA file. We can then BLAST the subset database against the human or mouse database to find the orthologs.

The pros and cons of these two options depends on several factors. I would probably use the second option to start. If I ended up having multiple results files and needing to do a lot of BLAST runs, I might think about making a modified FASTA file with embedded ortholog information. Your milage may vary.

Let's recap where we are. We did an awesome proteomics experiment on our cool obscure organism. We got a nice, tight final list of identified proteins. We made a subset FASTA file with just those final proteins. We used BLAST to find the ortholgs to a human Swiss-Prot database and generate an ortholog mapping file.

Now what? How about adding some of the neat information (annotations) about the proteins like what we see when we explore individual proteins at the UniProt website? We could add protein names, other common protein names, gene symbols, hyperlinks to the UniProt database, keyword terms, GO terms, and pathway information.

UniProt has flat text files (DAT files) that have a lot more information than the FASTA protein files have. We can download those files and parse out annotations to add for the proteins seen in our samples. For practical purposes, the choices for well-annotated species are human, mouse, or arabidopsis (we could probably also do yeast or E. coli). These are the species that we need to do the ortholog mapping to. A GUI script, [add_uniprot_annotations.py](https://github.com/pwilmart/annotations) has been written to make this easier to do. The README for the repository has more details and instructions for using the script.

The goal of the script will be to help you add extra annotation columns to your proteomics results table. Once you have a column of human gene symbols associated with your proteins, you can make use of many downstream tools for functional and pathway enrichment analysis, for example. The extra annotation information is quite useful by itself and may be all you need to interpret your results.

Here is the recap of this common proteomics workflow:

- identify the proteins in your samples
- map them to the orthologs for a well-annotated model system
- add extra annotation information for the orthologs

---

-Phil Wilmarth, OHSU, 20191014    
