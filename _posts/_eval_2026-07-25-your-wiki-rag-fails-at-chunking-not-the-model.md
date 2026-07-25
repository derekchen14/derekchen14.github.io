---
layout: post
title: "Your Wiki RAG Fails at Chunking, Not the Model"
date: '2026-07-25 12:01:50'
tags: [rag, chunking, wiki, retrieval, embeddings, company wiki, search]
color: 
excerpt_separator: <!--more-->
---

## Chunking Is the Bottleneck
I rebuilt our retrieval pipeline three times chasing better embeddings, then found the embeddings were fine and the chunks were the problem. I swapped models twice and recall barely moved. The real issue sat upstream: fixed 512-token splits cut arguments in half, so a claim landed in one chunk and its supporting evidence landed in the next, and the retriever pulled only one of the two.

A token counter has no sense of where a sentence ends, let alone an argument. When I switched to chunking by heading instead of token count, recall on our internal eval set of roughly 200 queries rose from 61 percent to 84 percent, and none of that gain came from touching the model.

## Structure-Aware Splits
Wiki pages already carry their own chunking scheme: the headings. Splitting on h2 and h3 boundaries instead of a token counter turns each chunk into a self-contained unit that matches how a person would summarize the page. A troubleshooting page with six numbered subsections becomes six chunks, each answerable on its own, instead of token-sized slabs that straddle two subsections at once.

Chunk sizes stopped being uniform, ranging from 80 tokens for a short FAQ entry to 600 for a dense configuration section, and that variance turned out fine. The one exception: any section under 40 tokens gets merged into its neighbor, since a heading followed by a single line adds noise without giving the retriever anything to act on.

## Overlap and Context
While heading-aware chunks solve the mid-argument cut, they introduce a fresh headache: a chunk titled "Configuration" is useless contextually without its parent section and page title. Embed that chunk alone and the retriever can't tell a database Configuration section from a firewall one. I prepend the page title and the immediate parent heading to every chunk before embedding it, plus a 50-token overlap pulled from the end of the previous chunk. That overlap fixed a specific failure: a query about "the timeout value," where the timeout was defined in the paragraph just before the boundary. Without the overlap, the winning chunk never mentioned the term the user searched for.

## The Case for a Bigger Model
The obvious pushback: none of this should matter once context windows got large. A model that holds 100,000 tokens can skip chunking and take the whole page, then sort out what matters. That's a fair objection, and for a five-page wiki it might be the right call.

It stops working at real scale for two reasons. Cost and latency scale with tokens sent, so stuffing full pages into every query gets slow and expensive fast. And models pay uneven attention across a long context: Liu et al. (2023), in "Lost in the Middle," found that information placed in the middle of a long context gets used less reliably than information near the start or end. A 400-page wiki can't be stuffed into one prompt anyway, so something still has to pick which pages matter first, and that selection step, the retrieval step Lewis et al. (2020) built the original RAG architecture around, is where chunk quality decides whether the right page gets picked at all.

## Measuring Retrieval Quality
I track two numbers now: recall at 5, the standard measure from Manning, Raghavan, and Schütze's Introduction to Information Retrieval for whether the right chunk shows up in the top five results, and a staleness check that compares each page's last-edited date against its content. Chunking got us to 84 percent recall, but stale pages still surfaced confidently wrong answers. One page from 2021 kept winning retrieval for a policy that had changed eight months earlier and was never updated. We couldn't solve both with the same tool. While chunking is a one-time preprocessing decision during reindexing, catching stale content requires an active monitor—specifically, a nightly job that flags pages older than related topics.