# Website UI Route Audit & Performance Testing

A Python-based website auditing tool built with **Playwright, Python, and Pandas** to automatically discover website routes, test page accessibility, capture HTTP response statuses, and measure page load performance.

The project uses browser automation with headless Chromium to perform route auditing and generate structured QA reports in CSV format.

## Features

- **Automatic Route Discovery** – Extracts hyperlinks (`a[href]`) from a target landing page.
- **Page Load Measurement** – Measures page navigation time in milliseconds using Playwright.
- **HTTP Status Monitoring** – Captures HTTP response codes for each tested page.
- **QA Status Classification** – Automatically classifies pages as `PASS` or `FAIL`.
- **Error Detection** – Identifies common issues such as timeout, DNS resolution failure, connection refusal, and HTTP errors.
- **Sequential Execution** – Audits routes sequentially to maintain consistent measurement conditions.
- **Automated Reporting** – Generates a CSV report containing URL, QA status, HTTP code, load time, and error details.
- **Performance Analysis** – Sorts results by load time to help identify slow-loading pages.

## Project Structure

```text
.
├── endpoint_load_time.ipynb   # Main notebook for crawling and auditing
├── qa_ui_route_audit.csv      # Generated QA audit report / sample result
├── .gitignore                 # Git ignore configuration
└── README.md                  # Project documentation
```

## Technology Stack

| Technology | Purpose |
|---|---|
| Python | Main programming language |
| Jupyter Notebook | Interactive development and execution |
| Playwright | Browser automation and page auditing |
| Chromium | Headless browser engine |
| Asyncio | Asynchronous browser operations |
| Pandas | Data processing and report generation |

## Requirements

- Python 3.9+
- pip
- Internet connection
- Chromium browser (installed through Playwright)

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/ardfahriarr/endpoint_load_time.git
cd YOUR_REPOSITORY
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate the environment:

**Linux / macOS**
```bash
source .venv/bin/activate
```

**Windows**
```bash
.venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install playwright pandas jupyter
```

### 4. Install Chromium

```bash
playwright install chromium
```

## Configuration

Open `endpoint_load_time.ipynb` and modify the configuration section:

```python
# --- CONFIGURATION ---

BASE_URL = "https://example.com"
TIMEOUT_MS = 20000
```

| Parameter | Description | Default |
|---|---|---|
| `BASE_URL` | Target website landing page used for route discovery | Example URL |
| `TIMEOUT_MS` | Maximum navigation time per page | 20,000 ms |

## How It Works

The auditing process consists of three main stages.

### 1. Route Discovery

The script opens the configured landing page and extracts hyperlinks using Playwright.

It uses Python's `urljoin()` to resolve relative URLs and removes URL fragments to eliminate duplicate routes.

```python
routes_to_audit = await get_all_routes(
    scrape_page,
    BASE_URL
)
```

The discovered routes are stored in a set to avoid duplicate URLs.

### 2. Route Auditing & Performance Measurement

Each discovered URL is opened using a new browser page.

The script captures:

- URL
- HTTP response status
- Page load time (milliseconds)
- QA status (`PASS` / `FAIL`)
- Error details

Load time is measured using Python's `time.perf_counter()`.

The QA classification follows these rules:

| Condition | QA Status | HTTP Code |
|---|---|---|
| Successful HTTP response below 400 | PASS | Actual HTTP code |
| HTTP response 400 or higher | FAIL | Actual HTTP code |
| Navigation timeout | FAIL | TIMEOUT |
| DNS resolution failure | FAIL | DNS_ERROR |
| Connection refused | FAIL | CONN_REFUSED |
| Other navigation errors | FAIL | ERROR |
| No response received | FAIL | No Response Received |

### 3. Reporting & Data Analysis

After all routes have been audited, the collected results are converted into a Pandas DataFrame.

The report is sorted by load time in descending order, making the slowest pages easier to identify.

```python
df = pd.DataFrame(audit_results)

df_sorted = df.sort_values(
    by="Load Time (ms)",
    ascending=False
)
```

The results are exported automatically:

```python
filename = "qa_ui_route_audit.csv"

df_sorted.to_csv(filename, index=False)
```

The script also displays a summary containing the total number of tested pages, passed pages, and failed pages.

## Running the Project

1. Open the Jupyter Notebook:

```bash
jupyter notebook endpoint_load_time.ipynb
```

2. Update `BASE_URL` with the target website.
3. Adjust `TIMEOUT_MS` if necessary.
4. Execute all notebook cells using **Run All**.
5. Review the audit summary and generated CSV report.

## Sample Audit Results

The repository includes `qa_ui_route_audit.csv`, containing an example audit result from Ahmad Dahlan University (UAD).

**Audit Summary**

| Metric | Result |
|---|---:|
| Total Pages Tested | 46 |
| Passed | 34 |
| Failed | 12 |
| Pass Rate | 73.91% |
| Fail Rate | 26.09% |

**Sample Findings**

| URL | QA Status | HTTP Code | Load Time (ms) |
|---|---|---|---:|
| `bimawa.uad.ac.id` | PASS | 200 | 16,596.49 |
| `oia.uad.ac.id` | PASS | 200 | 9,284.04 |
| `pmb.uad.ac.id` | FAIL | TIMEOUT | 20,002.35 |
| `eprints.uad.ac.id` | FAIL | TIMEOUT | 20,006.45 |

*Note: These figures represent the included sample audit report, not a continuous or real-time monitoring result.*

## Example Data Analysis

The generated CSV can also be analyzed using Pandas.

**Basic statistics**

```python
df["Load Time (ms)"].describe()
```

**Identify the 10 slowest pages**

```python
df.sort_values(
    by="Load Time (ms)",
    ascending=False
).head(10)
```

**Get QA status distribution**

```python
df["Status QA"].value_counts()
```

**Filter failed pages**

```python
df[df["Status QA"] == "FAIL"]
```

## Limitations & Considerations

- Route discovery is performed from the configured landing page, not recursively across all discovered pages.
- The current implementation collects HTTP and HTTPS hyperlinks found on the landing page, including external links.
- Routes are tested sequentially rather than concurrently.
- Load time represents browser navigation completion using Playwright's `wait_until="load"` setting. It is not a complete Core Web Vitals assessment.
- Network conditions, server response times, browser caching, and third-party resources can affect measurements.
- Authentication and protected pages may require additional Playwright session configuration.
- Run repeated tests when comparing performance to reduce measurement variability.

**Responsible Usage:** Only audit websites that you own or have explicit permission to test. Avoid excessive requests and respect the target website's usage policies.

## License

No license file is currently included in this repository. Add a `LICENSE` file if you intend to distribute the project under an open-source license.