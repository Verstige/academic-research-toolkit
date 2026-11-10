# SEC EDGAR Research — Public Company Financials

Use this when you need financials, filings, or SEC data on a public company (CIK, ticker, company name).

---

## Core API: SEC XBRL Company Facts

The authoritative, free, rate-limited endpoint for all XBRL-tagged financial data:

```
https://data.sec.gov/api/xbrl/companyfacts/CIK{0-padded-CIK}.json
```

Example: Investview CIK `0000862651` → `https://data.sec.gov/api/xbrl/companyfacts/CIK0000862651.json`

Headers required:
```
User-Agent: Mozilla/5.0
Accept: application/json
```

Rate limit: ~10 req/sec, be polite.

**XBRL data shape:**
```python
facts = data['facts']['us-gaap']
# Revenue
rev_data = facts['Revenues']['units']['USD']  # list of {end, val, form, frame}
# Net income
ni_data = facts['NetIncomeLoss']['units']['USD']
# Total assets
assets_data = facts['Assets']['units']['USD']
```

Annual entries filter: `end` ending in `12-31` (calendar year) or `03-31` (fiscal year ending March).

---

## Finding the Right CIK

If you only have a ticker or company name, search the EDGAR full-text index:

```
https://efts.sec.gov/LATEST/search-index?q={company}+{term}&forms=10-K
```

Returns hits with `adsh`, `period_ending`, `file_date`, and `form`. The `adsh` contains the CIK in the middle digits.

---

## 10-K / Annual Report URL

From search results, construct:
```
https://www.sec.gov/Archives/edgar/data/{CIK}/{adsh-with-dashes}/{adsh}.txt
```

Note: Direct .txt access frequently returns 403. Fall back to XBRL data which is more accessible.

---

## Known Data Gaps

XBRL data can be sparse for OTC/pink sheet companies, companies that filed NT 10-K, or haven't updated in 12+ months. Supplement with Google search, company IR page, or SimilarWeb estimates.

---

## Key XBRL Tags

| Concept | Tag |
|---|---|
| Revenue | `Revenues` |
| Net Income/Loss | `NetIncomeLoss` |
| Total Assets | `Assets` |
| Cash | `CashAndCashEquivalentsAtCarryingValue` |

---

## Python Extraction Pattern

```python
import json, urllib.request

def get_company_facts(cik_padded):
    url = f"https://data.sec.gov/api/xbrl/companyfacts/CIK{cik_padded}.json"
    req = urllib.request.Request(url, headers={
        "User-Agent": "Mozilla/5.0",
        "Accept": "application/json"
    })
    with client.get(url) as r:
        return json.loads(r.read())

def annual_entries(facts_list):
    return [x for x in facts_list if x.get('form') == '10-K' or '12-31' in x.get('end','')]

data = get_company_facts('0000862651')
us_gaap = data['facts']['us-gaap']
for item in sorted(annual_entries(us_gaap['Revenues']['units']['USD']), key=lambda x: x['end']):
    print(item['end'], item['val'])
```

---

## EDGAR Search Index

```
https://efts.sec.gov/LATEST/search-index?q={query}&forms={form}
```

Returns filing metadata including `adsh`, `period_ending`, `file_date`.

---

## Pink Sheet / OTC Companies

- Still file 10-K/10-Q when registered under Section 12(g)
- Data completeness varies — some go dark for years
- Stock price and shares outstanding on OTCMarkets.com, not in XBRL
- Supplement with Crunchbase, LinkedIn, or SimilarWeb estimates
