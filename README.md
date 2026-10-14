# IR.21 Prefix Extractor

A single-file, fully offline web app for roaming engineers. It reads the MSISDN number
ranges from a GSMA RAEX IR.21 document, validates them, removes duplicates, expands them into
the minimal set of decimal-aligned routing prefixes, and exports the result as Excel or CSV.

Everything runs in the browser. No server, no build step, no internet, no external libraries,
no fonts, no CDNs. Nothing is uploaded anywhere.

## Usage

Open `index.html` in any modern browser (Chrome, Edge, Firefox, Safari).

1. Drop one or more files onto the page, or paste a table into the "Paste a table" box.
2. Review the prefix preview, the detection report and the source ranges.
3. Click **Download .xlsx** (or **CSV** / **Copy**). The workbook is named `<input name> Prefix.xlsx`.

## Input formats

| Format | How it is detected |
|---|---|
| IR.21 XML | Starts with `<`. Namespace-agnostic, local-name matching. Only `MSISDN_NumberRanges` blocks are read; GT, MSRN and NP ranges are ignored. If no such block exists, any element with `CC` and `NDC` children (or attributes) is used and the report says so. |
| Fixed-width table (copied from a PDF) | Header line with two or more spaces between labels and an IR.21 marker such as "Network Owner" or "SN Range Start". Columns are sliced at the header positions, so blank DPC cells do not shift the Network Owner column. |
| CSV / TSV / semicolon | Delimiter in the first line. Header labels are recognised by aliases ("Country Code", "SN Start", "Range End", "Owner", …). Without a header the columns are positional: CC, NDC, start, stop, length, owner. |
| Free lines | Anything else, e.g. `33 64162 0000–9599 MNO`. |

## Rules

**Prefix derivation.** For each range, `prefixStart = CC + NDC + SN_RangeStart` and
`prefixEnd = CC + NDC + SN_RangeStop`. The range is expanded into the minimal set of covering
prefixes: at each step the largest block `10^e` is taken such that the current SN is a multiple
of `10^e` and the block fits inside the range; the prefix is `CC + NDC` plus the first `W − e`
digits of the SN (where `W` is the SN width). Example: `33 64162 0000–9599` gives 15 prefixes,
`336416 20` … `28` and `336416 290` … `295`. The "Prefix level" option can switch to plain
`CC + NDC` output instead.

**Validation.** Six checks per range: CC is 1–3 digits; CC matches the CC derived from the
header MCC (built-in MCC ↔ CC table); NDC is 2–5 digits; SN start and stop are present and the
same length; prefix length equals the declared Range Length; SN start ≤ SN stop. Failures of
checks 1, 3 or 4 mark the row `err` and it produces no prefixes. Other failures mark it `warn`.

**Duplicates.** Three reversible toggles, re-applied from the full in-memory set each time:
remove exact duplicates (on), remove ranges fully contained in a wider one on the same CC + NDC
(on), merge adjacent ranges (off). Removed rows stay visible, greyed, with the reason and the
parent row. A prefix claimed by several ranges is listed once with a ×N badge and a
`shared_count` column in the CSV.

**Owner.** `NetworkOwner` values MNO / HPMN / VPMN map to MNO, MVNO to MVNO, anything else to
OTHER, missing to UNKNOWN. If the element is not next to the range, up to six ancestor levels are
searched and the row is marked inherited. `NetworkType` is never used. Unknown rows can be
assigned an owner from the report line.

## Output

`Sheet1` with two columns: `PrefixNumber` (numeric) and `NetworkOwner` (`MNO`, `MVNO` or
`OTHER`; blank when the IR.21 gives no owner). The CSV export (UTF-8 with BOM) carries prefix, owner, CC, NDC, SN block, parent
range, CC check, status, notes and shared_count.

## Files

| File | Purpose |
|---|---|
| `index.html` | Standalone page for local use. |
| `ir21-prefix-extractor.html` | Same page without the document wrapper, used as the claude.ai artifact source. |

## Note on sample data

IR.21 documents are confidential operator documents. `.xml` and `.xlsx` files are ignored by
git so that samples kept next to the app are never committed by accident.
