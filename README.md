# MedAssociate Parser — George Lab

A browser-based tool for extracting and organizing raw Med-PC / MedAssociate behavioral data files into clean CSV outputs. No installation required — open the HTML file directly in any browser.

## Features

- **Drag & drop** raw Med-PC text files (one file per session day, multiple rats per file)
- **Auto-detects** session type from filename (FR, PR, Shock) — overridable manually
- **Auto-detects** session duration from Start/End Time fields
- **Live reward chart** with adjustable bin size, shown immediately after upload
- **Two CSV outputs:**
  - `timestamps_matrix.csv` — variables as rows, rats as columns (matches GEToperant Excel format)
  - `binned_data.csv` — rats as rows, time-binned counts as columns
- **Session type support:**
  - FR (fixed ratio) — SHA / LGA sessions
  - PR (progressive ratio) — includes PR breakpoint (last ratio)
  - Shock — includes total shocks and shock-at-reward timestamps

## Quick start

1. Download `medassociate_parser.html`
2. Double-click to open in your browser
3. Drop your raw Med-PC file(s) onto the upload area
4. Select session type (auto-detected from filename)
5. Review the live reward chart
6. Click **Generate CSV files** and download

## Array mapping (GWAS programs)

| Variable | Array | Type |
|---|---|---|
| Reward timestamps | V | multi-value |
| Active timestamps | Y | multi-value |
| Inactive timestamps | U | multi-value |
| Total rewards | B[0] | scalar |
| Total active presses | G[0] | scalar |
| Total inactive presses | A[0] | scalar |
| PR breakpoint | S[0] | scalar (PR only) |
| Shock at reward # | M | multi-value (Shock only) |
| Total shocks | T[0] | scalar (Shock only) |

All array assignments are configurable in the interface.

## File naming convention for auto-detection

| Pattern in filename | Session type |
|---|---|
| `SHA`, `LGA` | FR (fixed ratio) |
| `PR`, `COCPR` | Progressive ratio |
| `PRESHOCK`, `SHOCK` | Shock session |

## Contributing

Pull requests welcome. To suggest changes:

1. Fork this repository
2. Create a feature branch: `git checkout -b my-improvement`
3. Commit your changes and open a pull request

For bugs or feature requests, open an [issue](https://github.com/George-LabX/GEToperant/issues).

## Citation

If you use this tool in your research, please cite:

> George Lab (2025). MedAssociate Parser. George-LabX/GEToperant. https://github.com/George-LabX/GEToperant

## Related

- [GEToperant](https://github.com/SKhoo/GEToperant) — original Python/Excel-based tool this replaces
- [MedParse](https://github.com/George-LabX/MedParse) — Python/MATLAB parser

## License

MIT
