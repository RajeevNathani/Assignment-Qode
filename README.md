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

## Technical Approach

### Efficient scraping with Playwright + virtual threads
- **Why Playwright?** Robust against minor DOM changes, smooth infinite scrolling, powerful selector engine.
- **Rate limiting:** `resilience4j` caps interactions; exponential **retry** only for transient errors.
- **Backpressure:** Bounded `BlockingQueue<TweetRecord>` so producers naturally block when consumers lag.
- **Dedup:** Exact `HashSet` on `tweetId` + **Guava BloomFilter** on `(username|timestamp|hash(content))` to avoid near-dupes while streaming.

### Collection
- Navigates to **public** X/Twitter live search per hashtag; scrolls with polite delays.
- Extracts `username`, `timestamp` (from ``<time datetime="...">``), `content`, engagement counts, `mentions`, and `hashtags`.
- **Virtual threads** run collectors concurrently, feeding a bounded queue for backpressure.

### Processing & Storage
- Unicode-safe normalization; preserves whitespace and emoji.
- Dedup: exact (`tweetId`) + approximate (Bloom on `user|time|hash(content)`).
- **Stream writes to Parquet** with **ZSTD** compression, 256 KB pages, default row groups (columnar + compact).

### Analysis & Insights
- Tokenization via **Lucene** (stopwords, stemming).
- Compute **TF-IDF/global term weights**; engineer features (caps/emoji/hashtag length).
- Domain lexicon → per-tweet **sentiment score** in **[-1, +1]**.
- **Aggregation:** 5-minute buckets → mean lexical score, bullish fraction *p* with **Wilson 95% CI**.

### Visualization (memory-efficient)
- Time-bucket **downsampling**; **reservoir sampling** for large scatters.
- Render charts with **XChart** to PNG (no heavy GUI dependencies).

### Performance & Scalability
- **Concurrency:** Java **virtual threads**; hash-based hashtag partitioning.
- **I/O:** Streaming **Parquet**; no large in-RAM datasets.
- **Throughput:** Tunable rate limiter; batch flush; selector-based parsing.
- **Scale 10×:** Multiple stateless workers beh

