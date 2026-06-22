<div align="center">

# Spain Public Procurement Dataset

**A broad, machine-readable collection of Spanish tender notices for commercial intelligence and procurement research.**

27,260 records · 19,618 unique tender IDs · 15 sectors · JSONL

[Explore the data](#repository-layout) · [Understand the schema](#record-structure) · [Start analysing](#quick-start)

</div>

## Why this repository exists

Procurement opportunities are easier to understand when the underlying notices can be analysed as data instead of read one page at a time. This repository provides structured Spanish tender records that can be filtered by purchaser, deadline, sector, CPV code, value, and other normalized fields.

The collection is useful for:

- suppliers researching public-sector demand in Spain;
- analysts studying purchasers, sectors, CPV codes, values, and procurement activity;
- data teams building search, classification, or alerting prototypes;
- researchers experimenting with information extraction from tender text.

This is a historical dataset, not a live tender feed. Always verify deadlines and official requirements with the original procurement authority before acting on a notice.

## Dataset at a glance

| Attribute | Value |
|---|---:|
| Country | Spain |
| Format | JSON Lines (`.jsonl`) |
| Dataset files | 28 |
| Total records | 27,260 |
| Unique `TOT_ID` values | 19,618 |
| Sector labels | 15 |
| Published-date coverage | 17 April 2022 to 27 December 2025 |
| Latest file snapshot | 27 December 2025 |
| JSON validation | All rows parsed successfully during the June 2026 review |

The difference between total records and unique IDs is expected: the repository contains multiple snapshots, so some tenders appear more than once.

## Repository layout

All datasets are stored on the default branch. Files follow this convention:

```text
dataset_ES_<sector-index>_<YYYYMMDD_HHMMSS>.jsonl
```

For example:

```text
dataset_ES_1_20251227_110149.jsonl
```

The timestamp identifies when that snapshot was generated. Use the `Sector` field inside each record as the authoritative category label; the numeric sector index is an internal file grouping.

The collection covers agriculture and food, IT, construction, defence and security, education, environment, finance, materials, mining, power and energy, printing and publishing, research, technology and equipment, transport, and other services.

## Record structure

Each line is an independent JSON object with three top-level properties:

```json
{
  "instruction": "Extraction and classification task",
  "input": "Original tender text",
  "output": {
    "TOT_ID": "129740113",
    "Tender_Notice_No": "PLI-01782",
    "Country": "Spain",
    "Purchaser_Name": "ENUSA INDUSTRIAS AVANZADAS S.A., S.M.E.",
    "Summary": "Supply of molybdenum material",
    "Description": "Tender description",
    "Tender_Value": "191667.00",
    "Currency": "EUR",
    "USD_Tender_Value": "233297.26",
    "Published_Date": "2025-11-08",
    "Closing_Date": "2025-11-14",
    "Competition": "ICB",
    "Financier_Name": "Self Financed",
    "CPV": "14741000",
    "Sector": "Mining and Ores",
    "Sub_Sector": "Mining and Basic Metal",
    "More_Details": "https://tendersontime.com/register/"
  }
}
```

`Tender_Value`, `Currency`, and `USD_Tender_Value` are optional, although value coverage is stronger here than in the other country repositories reviewed. Their absence should not be interpreted as zero.

## Quick start

The following Python example loads every snapshot and keeps the latest occurrence of each `TOT_ID`:

```python
from pathlib import Path
import json


def snapshot_time(path: Path) -> tuple[str, str]:
    return tuple(path.stem.rsplit("_", 2)[1:])


latest_by_id = {}

for path in sorted(Path(".").glob("dataset_ES_*.jsonl"), key=snapshot_time):
    with path.open(encoding="utf-8") as source:
        for line in source:
            row = json.loads(line)
            tender = row["output"]
            tender["_source_file"] = path.name
            latest_by_id[tender["TOT_ID"]] = tender

print(f"{len(latest_by_id):,} unique tenders loaded")
```

To inspect normalized records with `jq`:

```bash
jq -c '.output' dataset_ES_*.jsonl | head
```

## Data quality notes

- All 27,260 non-empty rows are valid JSON.
- Snapshot overlap creates 7,642 repeated `TOT_ID` occurrences.
- `TOT_ID` is the recommended deduplication key.
- Tender values are present in roughly half of the stored rows; currencies and converted values remain optional.
- Descriptions may contain HTML entities inherited from source notices.
- Dates are stored as `YYYY-MM-DD` strings.
- A `More_Details` link may lead to a registration or access page rather than directly to the contracting authority.

For production use, normalize HTML entities, parse dates explicitly, retain the source filename, and verify important records against an official source.

## About TendersOnTime

[TendersOnTime](https://www.tendersontime.com) helps organizations discover public procurement opportunities across markets and sectors. This repository offers a structured sample of that work for Spain.

- [Browse current Spain tenders](https://www.tendersontime.com/spain-tenders/)
- [Explore tenders by country](https://www.tendersontime.com/tendersby/country/)
- [Subscription options](https://www.tendersontime.com/subscribe/)
- [Contact the team](https://www.tendersontime.com/contact/)

## Usage and responsibility

Before redistributing or using this dataset commercially, review the [TendersOnTime Terms & Conditions](https://www.tendersontime.com/terms/) and [Privacy Policy](https://www.tendersontime.com/privacy/). This repository does not grant rights beyond those terms.

Tender information changes. Confirm eligibility, deadlines, values, documents, and submission instructions with the relevant official authority.

## Contributing

If you find malformed JSON, an incorrect classification, a duplicate that cannot be explained by snapshot overlap, or a documentation issue, open a GitHub issue with the filename and `TOT_ID`. Avoid posting confidential or account-specific information.

---

Maintained by [TendersOnTime](https://www.tendersontime.com) · Documentation refreshed 22 June 2026
