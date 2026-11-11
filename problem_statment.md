# Day 50 — Problem Statement

## The question

How many more tokens does a tokenizer need to say the same thing in Urdu or Pashto
than in English? That multiplier is the **tokenizer tax**: it raises API cost,
shrinks how much text fits in a context window, and gives the model less to learn
from per word.

The method is published (*The Tokenizer Tax*, arXiv 2607.24276) for Urdu and nine
Indian languages, on six tokenizers, using the FLORES-200 dev set. **Pashto is in
none of the published fertility studies.** This day produces that measurement and
is the seed of Project #1.

## Why a parallel corpus

A comparison is only fair when both sides are tested on the same thing. FLORES gives
the same 997 sentences, with the same meaning, professionally translated into each
language. Any difference in token count then comes from the tokenizer and the
language, not from the content.

With 997 random Pashto sentences and 997 random English ones, a difference could
just mean the Pashto sentences were longer, or about harder topics.

## Predictions, made before running

- GPT-2 worst for Pashto, XLM-R best
- Pashto worse than Urdu on most tokenizers — less Pashto text on the web

## How we know the pipeline works

1. **Alignment:** sentence ids match at all 997 positions in all three languages
2. **English sanity:** cl100k fertility on English should be about 1.2–1.3
3. **Reproduction:** the Urdu tax must match the published numbers before any Pashto
   number is trusted

## Data and compute

FLORES+ (`openlanguagedata/flores_plus`), `dev` split, `eng_Latn`, `urd_Arab`,
`pbt_Arab`. Six tokenizers: cl100k_base, o200k_base, GPT-2, Qwen2.5, mBERT, XLM-R.
Runs in minutes on a laptop.