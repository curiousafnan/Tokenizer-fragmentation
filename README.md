# Day 50 — The Tokenizer Tax on Urdu and Pashto

Part of **Deep Building**, a from-scratch ML/AI challenge. Week 10, Day 50.

How many more tokens six tokenizers need for the same sentence in Urdu and Pashto
than in English, measured on the 997 parallel sentences of the FLORES-200 dev set.

## Headline

**Tokens needed vs English (parity), 95% bootstrap intervals in `approach.md`:**

| Tokenizer | Urdu | Pashto |
|---|---|---|
| GPT-2 | 6.35× | 5.49× |
| cl100k (GPT-4) | 4.46× | 3.90× |
| Qwen2.5 | 3.24× | 2.93× |
| mBERT | 1.73× | 1.68× ⚠️ 17% `[UNK]` |
| o200k (GPT-4o) | 1.68× | 1.78× |
| XLM-R | 1.26× | 1.42× |

- The pipeline **reproduces the published Urdu tax** on all six tokenizers within ~1%.
- **Pashto numbers are new** — Pashto is absent from published fertility studies.
- **English-centric tokenizers charge Urdu more than Pashto.** Cause, tested: they
  never learned four Urdu-specific letters (ہ ھ ے ں). Swapping those for the shared
  Arabic/Persian forms makes Urdu cheaper than Pashto on cl100k.
- **Tokenizers trained on real Urdu reverse it:** Pashto pays more on o200k and XLM-R.
- **mBERT cannot represent Pashto:** one token in six is `[UNK]`.

## Files

| File | Contents |
|---|---|
| `research.ipynb` | Data, alignment check, six tokenizers, metrics, bootstrap, both hypothesis tests |
| `problem_statement.md` | The question, why a parallel corpus, predictions |
| `approach.md` | Metrics, validation against the paper, all results, the two tests |
| `learnings.md` | What I got wrong and how the explanation was found |