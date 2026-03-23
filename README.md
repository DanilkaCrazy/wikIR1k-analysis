## README.md – Theoretical and Mathematical Foundations

### Project Overview

This repository contains an analysis of the **wikIR1k** dataset, a collection of Wikipedia articles from the [WikiR project](https://github.com/getalp/wikiR). The data is already cleaned, tokenized, and lowercased, making it suitable for information retrieval experiments. The analysis follows the tasks outlined in **HA1.pdf**, covering basic collection statistics, word and bigram frequency distributions, Zipf’s and Heaps’ laws, and morphological processing (stemming, lemmatization, and BERT tokenization).

### Dataset Description

The dataset consists of 369,721 documents extracted from Wikipedia. Each document is represented by its text (`text_right` column) and a unique identifier. The text is already split into tokens (words) and all characters are lowercased.

---

### Basic Collection Statistics

We computed:
- **Number of documents**: total count of articles.
- **Total number of words**: sum of all tokens.
- **Average document length**: total words divided by number of documents.
- **Number of unique words (types)**: distinct tokens in the entire collection.
- **Average word length**: mean character length across all token occurrences.
- **Average unique word length**: mean character length of distinct tokens.

These statistics give a first impression of the collection’s size, richness, and complexity.

---

### Zipf’s Law

Zipf’s law states that for a natural language corpus, the frequency of a word is inversely proportional to its rank in the frequency table:

\[
f(r) \propto \frac{1}{r^\alpha}
\]

where \(f(r)\) is the frequency of the \(r\)-th most frequent word, and \(\alpha\) is often close to 1. This relationship appears as a straight line on a log‑log plot. The law reflects the highly skewed distribution of word frequencies in language: a few words (like “the”, “of”) dominate, while most words occur very rarely. We plot rank vs. frequency on log‑log axes to verify the law.

---

### Heaps’ Law

Heaps’ law describes how the vocabulary size \(V\) grows as a function of the total number of tokens \(N\) in a collection:

\[
V = k \cdot N^{\beta}
\]

with \(0 < \beta < 1\) (typically \(\beta \approx 0.5\)). On a log‑log plot this is a linear relationship. The law captures the fact that as we add more text, new words continue to appear, but at a decreasing rate. It is important for estimating vocabulary growth and for designing efficient indexing structures.

---

### Bigram Analysis

A bigram is a sequence of two consecutive words. Bigrams capture local word co‑occurrence and can form meaningful multi‑word expressions (e.g., “united states”). In information retrieval, bigrams can be used as index terms to improve phrase matching. We:
- Count all bigrams in the collection.
- Identify the most frequent ones.
- Propose criteria for selecting bigrams as dictionary entries:
  1. **Frequency threshold** (e.g., > 5 occurrences) to filter out noise.
  2. **Exclusion of stopword-only bigrams** (e.g., “of the”) to keep semantically meaningful pairs.
  3. Optional: use a statistical measure like mutual information to assess significance.

---

### Morphological Processing

Morphological processing normalises word forms to reduce vocabulary size and improve matching:

- **Porter Stemmer** (NLTK): a rule‑based algorithm that strips common suffixes (e.g., “running” → “run”). It is fast but can produce non‑dictionary stems.
- **spaCy Lemmatizer**: uses a dictionary and part‑of‑speech information to return the base form (lemma) of a word (e.g., “better” → “good”). It is more accurate but slower.
- **BERT Tokenizer** (HuggingFace): a subword tokenizer that splits words into pieces (e.g., “unhappiness” → [“un”, “##happiness”]). It is used in modern neural IR models to handle out‑of‑vocabulary words and morphological variation.

We report for each version the total number of tokens, number of unique tokens, and average token length to compare the effects of each transformation.

---

### Conclusion

The analysis demonstrates fundamental statistical properties of natural language (Zipf, Heaps) and applies text preprocessing techniques essential for building effective search engines. The results provide a baseline for further experiments in information retrieval.

---

## Plots_Description.md – Interpretation of Graphical Results

### 1. Zipf’s Law Plot

<img width="524" height="315" alt="Снимок экрана 2026-03-23 204252" src="https://github.com/user-attachments/assets/21b5dcea-6643-46f7-bd34-bfca76ddfa49" />

**Figure 1:** Log‑log plot of word rank versus word frequency.

- The points lie approximately on a straight line, confirming Zipf’s law.
- The slope is close to -1, indicating that the second most frequent word occurs about half as often as the most frequent, the third about one third, etc.
- The small deviations at the high‑rank (low‑frequency) end are typical and caused by the finite size of the corpus and the presence of rare words (hapax legomena).

**Implication:** The distribution is highly skewed; a small set of very frequent words dominates the total word count. This has direct consequences for indexing and compression.

---

### 2. Heaps’ Law Plot

<img width="541" height="317" alt="Снимок экрана 2026-03-23 204301" src="https://github.com/user-attachments/assets/387bda37-11be-49c2-acd1-88af4dacb676" />


**Figure 2:** Log‑log plot of total words (cumulative) versus vocabulary size (unique words).

- The points form a straight line, indicating that vocabulary growth follows a power law.
- The slope \(\beta \approx 0.5\) suggests that adding more text continues to introduce new words, but at a decreasing rate.
- Even after processing 73 million words, the vocabulary is still growing, meaning that the collection is not “closed” – a property of large, diverse corpora like Wikipedia.

**Implication:** For information retrieval, this justifies the need for compression and efficient handling of rare terms, because new words will always appear as the collection grows.

---

### 3. Top‑30 Frequent Words and Stopword Analysis

**Table:** The most frequent words are overwhelmingly function words (“the”, “of”, “in”, …).  
- **Total stopword occurrences:** 29,565,380 (about 40% of all words).
- **Top‑30 words in stopword list:** 28 out of 30 are stopwords (the exceptions are “first” and “also”).
- **Recommended additions to stopword list:** “first” (217,204 occurrences) and “also” (186,055 occurrences) – these are very frequent but not in the standard list. Adding them would reduce index size and may improve retrieval precision for many queries.

---

### 4. Top‑20 Bigrams and Selection Criteria

**Top bigrams:**  
- The most frequent bigrams are almost all composed of stopwords (e.g., “of the”, “in the”, “to the”). These carry little semantic content and would be poor dictionary entries.
- **Selected bigrams** after applying our criteria (frequency > 5 and not only stopwords) include:
  - “the first” (104,264 occurrences)
  - “one of” (70,278 occurrences)

**Interpretation:** These bigrams are meaningful and frequent enough to be useful for phrase‑based indexing. The criteria successfully filter out noisy stopword combinations while retaining potentially valuable phrases.

---

### 5. Morphological Processing Statistics

| Version              | Total tokens | Unique tokens | Avg token length |
|----------------------|--------------|---------------|------------------|
| Original (tokenized) | 73,093,729   | 794,568       | 4.80             |
| Porter stemmer       | –            | –             | –                |
| spaCy lemmatizer     | –            | –             | –                |
| BERT tokenizer       | –            | –             | –                |

*Note: Actual numbers will be obtained after running the second notebook.*  
- **Stemming** reduces unique tokens drastically, but may introduce conflations (e.g., different words becoming the same stem).
- **Lemmatization** yields dictionary forms, often preserving more distinct forms than stemming.
- **BERT tokenizer** produces a much larger number of tokens (subwords) and a different distribution of token lengths, reflecting its subword‑level encoding.

These transformations have different trade‑offs in terms of vocabulary size, retrieval effectiveness, and computational cost.

---

## Summary of Code Output

- **Basic statistics:**
  - Documents: 369,721
  - Total words: 73,093,729
  - Average document length: 197.7 words
  - Unique words: 794,568
  - Average word length: 4.80 characters
  - Average unique word length: 7.71 characters

- **Frequency list:** Top word is “the” with 5,311,943 occurrences.

- **Stopword analysis:** 28 of the top‑30 words are in the stopword list; “first” and “also” are recommended additions.

- **Bigram count:** 12,993,004 unique bigrams. The top bigrams are dominated by stopword combinations. After filtering, meaningful bigrams like “the first” and “one of” remain.

- **Plots:** Both Zipf and Heaps laws are clearly satisfied, confirming the expected statistical behaviour of natural language.
