# Facebook Comment Sentiment Analysis

Two Python scripts that collect the comments on a Facebook post and measure whether they are positive, negative or neutral.

1. **`Getdata.py`** downloads every comment on a post through the **Facebook Graph API** and saves them to `comment_data.json`.
2. **`SentementAnalysis.py`** scores each comment with **TextBlob**, prints the share of positive, negative and neutral comments, and saves a score for each comment to `sentiment_comments.json`.

---

## How it works

```
Facebook Graph API ──▶ Getdata.py ──▶ comment_data.json ──▶ SentementAnalysis.py ──▶ console summary
  (post comments)      (requests,                            (TextBlob polarity)       + sentiment_comments.json
                        pandas)
```

### 1. Collecting comments: `Getdata.py`

- Builds the request URL from a page ID and a post ID:
  `https://graph.facebook.com/v2.12/<page_id>_<post_id>/comments?limit=100&access_token=<token>`
- Fetches 100 comments at a time and follows the `paging.next` link until no pages are left, printing progress as it goes.
- Keeps each comment's text (`message`) and timestamp (`created_time`).
- Puts them in a pandas DataFrame with extra columns (`day_of_week`, `year`, `month`, `count`) for time-based analysis, and writes it to **`comment_data.json`**.

### 2. Scoring sentiment: `SentementAnalysis.py`

- Loads `comment_data.json`.
- For each comment, computes TextBlob's **polarity** score, which ranges from `-1.0` (most negative) to `+1.0` (most positive):

  | Polarity | Counted as |
  |----------|------------|
  | `> 0` | Positive |
  | `< 0` | Negative |
  | `= 0` | Neutral |

- Prints a summary like:

  ```
  Positive Comment:  45.0 %
  Negative Comment:  15.0 %
  Neutral Comment:  40.0 %
  ```

- Writes each comment and its score to **`sentiment_comments.json`**:

  ```json
  [{"id": "0", "comment": "Great post!", "sentiment_score": 1.0}, ...]
  ```

> **About the method:** By default, TextBlob's `.sentiment` uses its built-in, lexicon-based `PatternAnalyzer`, not a Naive Bayes model. To use TextBlob's Naive Bayes classifier trained on the NLTK movie reviews dataset, see [Using the Naive Bayes analyzer](#using-the-naive-bayes-analyzer).

---

## Project structure

| File | Purpose |
|------|---------|
| `Getdata.py` | Downloads the comments on a Facebook post into `comment_data.json` |
| `SentementAnalysis.py` | Scores the comments and writes `sentiment_comments.json` |
| `Access token.txt` | Link to the Graph API Explorer, where you can generate an access token |
| `LICENSE` | MIT License |

---

## Requirements

- Python 3
- [requests](https://pypi.org/project/requests/)
- [pandas](https://pandas.pydata.org/)
- [TextBlob](https://textblob.readthedocs.io/)
- A Facebook access token that can read the post's comments

---

## Installation

```bash
git clone https://github.com/UtshaDas/Facebook-Comment-Sentiment-Analysis.git
cd Facebook-Comment-Sentiment-Analysis
pip install requests pandas textblob
```

---

## Usage

### 1. Get an access token

Generate a token in the [Graph API Explorer](https://developers.facebook.com/tools/explorer/). Reading comments on a Page post requires a **Page access token** with the right permissions (such as `pages_read_engagement`). See Meta's [Graph API documentation](https://developers.facebook.com/docs/graph-api/) for current requirements.

Keep the token out of the code. Set it as the environment variable `FB_TOKEN`, which `Getdata.py` reads:

```bash
# macOS / Linux
export FB_TOKEN="your-access-token"

# Windows (Command Prompt)
set FB_TOKEN=your-access-token

# Windows (PowerShell)
$env:FB_TOKEN="your-access-token"
```

### 2. Choose the post

In `Getdata.py`, set the page and post you want to analyze:

```python
fb_pageid = "<your page id>"
fb_postid = "<your post id>"
```

A post's ID usually appears in its URL, or as `<page_id>_<post_id>` in the Graph API Explorer.

### 3. Collect the comments

```bash
python Getdata.py
```

This creates `comment_data.json`.

### 4. Analyze sentiment

```bash
python SentementAnalysis.py
```

The summary prints in the console, and the score for each comment is saved to `sentiment_comments.json`.

---

## Using the Naive Bayes analyzer

To classify comments with TextBlob's Naive Bayes model trained on movie reviews, download its corpora once:

```bash
python -m textblob.download_corpora
```

Then, in `SentementAnalysis.py`:

```python
from textblob import TextBlob
from textblob.sentiments import NaiveBayesAnalyzer

analyzer = NaiveBayesAnalyzer()
blob = TextBlob(comment, analyzer=analyzer)
blob.sentiment   # Sentiment(classification='pos', p_pos=0.8, p_neg=0.2)
```

This analyzer returns a `pos` / `neg` label with probabilities instead of a polarity score, so the counting logic needs a small change. It is also much slower than the default analyzer.

---

## Known issues and compatibility notes

The scripts were written in 2018, and some of the libraries and APIs they use have changed since then.

- **Graph API version.** The URL uses `v2.12`, which Facebook no longer supports. Change it to a current version from Meta's [Graph API changelog](https://developers.facebook.com/docs/graph-api/changelog/).
- **pandas `weekday_name`.** `df['dates'].dt.weekday_name` was removed in pandas 1.0. Replace it with `df['dates'].dt.day_name()`.
- **Comments without text.** Comments made only of a photo or sticker have no `message`, so `x1.get('message').encode(...)` fails on them. Use `x1.get('message', '')` to skip them safely.
- **Bytes instead of text.** `.encode('utf-8')` turns each comment into bytes in Python 3. Keeping the text as a plain string (`x1.get('message', '').strip()`) avoids JSON and TextBlob issues.
- **Missing-token check.** When `FB_TOKEN` is not set, the script prints a message but keeps running, because `exit` is missing its parentheses. Use `sys.exit("Set FB_TOKEN variable")`.
- **English only.** TextBlob's default analyzer is built for English. Comments in Bangla or other languages, and emoji-only comments, will mostly score as neutral.
- **Empty input.** If `comment_data.json` has no comments, the percentage calculation divides by zero.

---

## License

This project is licensed under the [MIT License](LICENSE).

## Author

**Utsha Das**
