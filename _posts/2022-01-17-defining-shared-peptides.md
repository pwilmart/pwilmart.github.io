---
layout: post
title: "Defining Shared Peptides"
date: 2022-01-17
---

# Shared peptides are context-dependent

## Introduction

I have written several blog posts on quantification in bottom-up proteomic experiments. The big picture is that peptides that map to a single "protein" are informative and can be used for quantitative analysis. Peptides that map to more than one "protein" have ambiguous information (potentially a mixture of relative protein abundances) and cannot be used for quantitative analysis. Not everyone agrees with this big picture view. That is not being debated here.

What we will be discussing is how definitions of unique and shared peptides are not fixed. Protein inference is an iterative process and each iteration changes how unique and shared peptides are defined. This is a key point that needs specific elaboration.

## Protein Contexts

In the opening paragraph the word "protein" was placed in quotes. That is because what we consider to be a "protein" changes with and depends on the protein inference, and how protein-level results are summarized in a particular workflow. We need to define and explore the protein contexts associated with bottom-up proteomics.

### FASTA protein sequence collections

FASTA files come from many sources (NCBI, UniProt, etc.) and come in many flavors (in a spectrum from one-protein-per-gene to more "complete" sequence collections). The choice of FASTA file is the first protein context used in the first iteration of protein inference. Database search engines *in silico* digest these FASTA sequences to make peptides to assign the fragment ion spectra. Either the search engine or a post processing step will list all of the proteins that could have produced each peptide. If there is only one FASTA entry that contained the peptide sequence, that is a unique peptide. If there are more than one proteins that contain the peptide (considering I and L indistinguishable), then the peptide is shared.

The first step of protein inference is to invert the peptide-to-protein mappings and create protein-to-peptide-set mappings. We start with a large number of peptides and their lists of proteins. We end up a much smaller list of proteins and all of their confidently identified peptides (we do protein inference after we have filtered PSMs using methods like target/decoy). If a peptide maps to more than one FASTA entry, that peptide is placed in all of the appropriate peptide sets. Each peptide set can be a mixture of unique and shared peptides (or 100% unique or 100% shared).

The basic rules of [protein inference](https://www.mcponline.org/article/S1535-9476(20)30061-X/fulltext) have been detailed and agreed upon for two decades. The first step is to compare the peptide sets of all proteins and combine proteins with identical peptide sets into protein groups.  

---
Phil Wilmarth <br> January 17, 2022
