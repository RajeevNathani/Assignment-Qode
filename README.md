# Assignment-Qode
Technical Assignment

- **Collectors (virtual threads):** Headless Chromium via Playwright scraping **public** live search pages.
- **Resilience:** Rate limiting + retry; graceful backpressure.
- **Dedup:** tweetId + Bloom filter on (user|time|hash(content)).
- **Storage:** Parquet (`ZSTD`, row-group tuning), UTF-8 safe, Indian scripts supported.
- **Signals:** Lucene tokenization → TF-IDF + domain lexicon → normalized composite signal with **95% Wilson CI**.
- **Viz:** Time-bucketed series and reservoir sampling; PNG via XChart.

## Getting Started

### Prerequisites
- Java 21+
- Maven 3.9+
- Chrome dependencies (Playwright downloads its own browser)
