# Google Ads Search Term Wastage Scanner

## Overview
This tool scans your raw Google Ads Search Terms CSV report, flags queries wasting your ad spend with zero return, and compiles them into a ready-to-use exact match negative keyword file (`[search term]`).

---

## 1. Setup & Configuration Variables

* `csv_file = "search_terms.csv"`
  The filename of your exported Google Ads search terms report. Must sit in the same folder as the script.
* `MAX_SPEND_NO_CONV = 45.0`
  Spend threshold. Flags any query that spent this amount or more without generating a single conversion.
* `MIN_CLICKS_NO_CONV = 12`
  Traffic threshold. Flags queries with high interaction volume (e.g., 12+ clicks) but 0 conversions.
* `junk_words = [...]`
  A watchlist of informational or low-intent keywords. Any query containing these words that fails to convert is immediately flagged.

---

## 2. Logic Walkthrough

### Reading & Sanitizing (`csv.DictReader`, `encoding="utf-8-sig"`)
* `utf-8-sig` strips out invisible Byte Order Marks (BOM) commonly injected by Excel and Google Ads exports.
* `row.get(...)` grabs target metrics safely, falling back to clean defaults if headers differ slightly.
* Strips currency symbols (`$`) and thousand-separator commas (`,`) so numbers parse properly into standard Python `float` and `int` types.

### Filtering Rules
A term gets flagged if it meets either condition:
1. **Performance Bleed (`is_bleeding`):** Has `0` conversions AND meets either the spend cap or the click cap.
2. **Intent Bleed (`has_junk_token`):** Matches any banned word in `junk_words` on an exact word boundary (using `query.lower().split()`) and produced `0` conversions.

### Output Generation
* **Console Metrics:** Displays the total query count flagged and sums the total dollars wasted.
* **Sorting:** Orders the terms from highest spend to lowest spend so the biggest budget drains are tackled first.
* **File Export (`add_to_negatives.txt`):** Surrounds every flagged search term with square brackets `[...]` to apply **Exact Match Negative** targeting, preventing accidental broad phrase collateral damage.

---

## 3. Workflow Steps
1. In Google Ads, navigate to **Campaigns** > **Insights & reports** > **Search terms**.
2. Select your date window (e.g., Last 30 Days).
3. Click **Download** > **CSV**.
4. Rename the downloaded file to `search_terms.csv`.
5. Run the script:
   ```bash
   python audit.py
