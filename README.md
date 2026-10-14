# IR.21 Prefix Extractor

A single-page web app that reads a GSMA RAEX IR.21 XML file and exports the operator's
MSISDN prefixes to an Excel workbook.

## Usage

Open `index.html` in any modern browser. No server or install is needed.

1. Drop one or more IR.21 `.xml` files onto the page, or click to choose them.
2. Review the prefix preview and the source ranges.
3. Click **Download .xlsx**. The file is named `<input name> Prefix.xlsx`.

## Extraction rule

- Every `NumberRange` under `MSISDN_NumberRanges` yields one prefix: `CC` + `NDC`
  (for example `57` + `302` = `57302`).
- Duplicates are removed and the list is sorted ascending.
- `GT_NumberRanges` and `MSRN_NumberRanges` are ignored.
- If a file contains several `<Network>` blocks (an MNO plus its MVNO, for instance), all of
  their MSISDN ranges are merged into one list.

## Output

`Sheet1` with a `PrefixNumber` column of numeric values.

When at least one range has `<NetworkOwner>MVNO</NetworkOwner>`, a second column
`NetworkOwner` is added. Rows for MVNO prefixes contain `MVNO`; the rest are blank.
A prefix that appears under both MNO and MVNO ranges is marked `MVNO`.

## Files

| File | Purpose |
|---|---|
| `index.html` | Standalone page for local use. |
| `ir21-prefix-extractor.html` | Same page without the document wrapper, used as the claude.ai artifact source. |

The page loads SheetJS 0.18.5 from cdnjs and IBM Plex fonts from Google Fonts, so it needs
internet access the first time it is opened.

## Note on sample data

IR.21 documents are confidential operator documents. `.xml` and `.xlsx` files are ignored by
git so that samples kept next to the app are never committed by accident.
