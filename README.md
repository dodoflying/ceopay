# Source and Structure Executive Compensation from SEC Edgar Databases

### Overview of Process Steps
1. **Download** Index Files: critical metadata for each Edgar filing (company name, filing date, filing url, etc.) 
2. **Extract** Filing Header: additional metadata *inside* each filing
3. **Download** [Def14A Document](https://www.sec.gov/fast-answers/answersproxyhtfhtm.html): annual proxy filings containing 100s of tables, one of which has executive compensation.
4. Detect Executive Compensation Table: compensation for top company executives, including base, bonus, options, and an overall annual salary
5. Html Table to Dictionary

- Persists data in S3 in such a way that enables codified direct access and Athena querying facilities
### What's the use-case?
- iShares ETF holdings track well-defined indices - baskets of stocks that represent well-defined areas of the market.
- Having a history of over 300 index-tracking ETF holdings allows you to:
    1. build universes for stock-selection modeling (e.g. R3000, FTSE, etc.)
    2. proxy stock level exposures to MSCI GICS sectors and industries (via Sector ETFs)
    3. track market behavior (e.g. value vs growth)
    4. Anything else your curious quant heart desires :)
### Example Usage
```
from ishares.utils import s3_helpers, athena_helpers

# get a single file from s3
df_small = s3_helpers.get_etf_holdings('IWV', '2020-06-30')

# query a lot of data with Athena
df_big = athena_helpers.query('select * from qcdb.ishares_holdings '
                              'where etf=\'IWV\' '
                              'and asofdate between date \'2020-01-31\' and date \'2020-06-30\'')
```

### Project Details
#####  `config.py`
Contains critical AWS configuration parameters within the `Aws` class (e.g. `S3_ETF_HOLDINGS_BUCKET`, `AWS_KEY`, etc.)

#####  `ishares/build_etf_master_index.py`
This script scrapes the iShares landing page for their "universe" of ETFs. That source webpage (and resulting output) provides etf-level information (namely, inception dates and product page url's) required for downloading holding histories. Output is sent to `./data/ishares-etf-index.csv`
![./data/ishares-etf-index.csv](https://raw.githubusercontent.com/talsan/ishares/master/assets/img/ishares-etf-index.PNG)

#####  `ishares/queue_etfdownloaders.py`
This script builds a queue of events that are executed by `etfdownloader.py`. Specifically, for a given iShares ETF ticker, this script determines which holding dates need to be downloaded, based on which holdings were downloaded in prior sessions (with an `overwrite=True` parameter to re-process everything, if desired).

#####  `ishares/etfdownloader.py`
Given an ETF and a holdings date (i.e. a single "event"), this script downloads the csv, validates its structure, formats it, and uploads it to aws s3.

### S3 Storage Example

### Athena Query Output Example
