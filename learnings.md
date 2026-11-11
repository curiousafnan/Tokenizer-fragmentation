# Day 50 — Learnings

## In my own words

- A **parallel corpus** is needed because a comparison only works when both sides
  are tested on the same thing: the same sentence, the same meaning, written in
  different languages.
- Different languages express the same sentence with different numbers of words,
  depending on grammar and vocabulary. **Correction:** dividing by words does not
  remove that — word counts themselves depend on the language, and Urdu's spacing
  makes them unreliable. That's why **parity** (tokens vs English tokens, same
  sentence) is the cleaner metric: no words are counted at all.
- On cl100k, Urdu is tokenized almost letter by letter. On o200k it's mostly whole
  words or large pieces, which is why its cost falls from 4.46× to 1.68×.

## Which predictions were wrong

1. **"Pashto is worse than Urdu everywhere."** Wrong on four of six tokenizers —
   Pashto is cheaper on GPT-2, cl100k, Qwen2.5 and mBERT.
2. **The first explanation I reached for — English words — was also wrong.** The
   Pashto translators kept 637 words in English letters, even though Pashto has its
   own vocabulary for most of them. That was clearly visible in sentence 0, and I
   hadn't thought about it. But removing every sentence with English letters left
   the gap unchanged (0.88). It's a real property of the dataset, not the cause.
3. **The actual cause:** four Urdu-specific letters (ہ ھ ے ں). cl100k never learned
   them. Swapping them for the shared Arabic/Persian forms makes Urdu cheaper than
   Pashto on cl100k — and more expensive on o200k, which did learn them.

**Lesson:** an explanation that fits isn't an explanation that's true. Test it
by removing the suspected cause and checking whether the effect goes away.

## Correction on Pashto across tokenizers

Pashto does **not** improve only a little from cl100k to o200k — it improves a lot,
from 3.90× to 1.78×. It improves **less than Urdu** (4.46× → 1.68×). The likely reason
is less Pashto training data than Urdu. That part is a hypothesis; Day 50 didn't
test it.

## Statistically real is not the same as meaningful

mBERT's Urdu–Pashto gap passes the bootstrap, but 16.68% of its Pashto tokens are
`[UNK]`. Missing letters (ځ, ښ, ږ) turn whole words into one unknown token, which
makes the token count look *better*. A stable number can still measure nothing.

## What broke or took more than one try

- Loading only the `text` column dropped the ids, so the alignment check had to be
  added by reloading with ids.
- The Urdu–Pashto explanation took three attempts: guess (English words) → test →
  rejected → second hypothesis (letters) → test → confirmed.

## Process note

Data loading, metrics, the reproduction, the English-word test and the bootstrap
were my work. Charting and the final experiments were tool-assisted.