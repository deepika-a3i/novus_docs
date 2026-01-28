# Report Rendering

This module covers how PDFs and Excel reports are rendered in the reporting stack.

## PDF scorecards (WeasyPrint)

- Class: `WeasyprintScorecardGenerator` (`src/novus_calls/report/scorecard.py`).
- Templates: `src/novus_calls/report/app/` (HTML, CSS, assets).
- Renderer: WeasyPrint via `HTML(...).write_pdf()`.

### Flow

1. Scorecard JSON is prepared by `AgentReportService`.
2. The generator loads Jinja2 templates and renders HTML.
3. WeasyPrint converts HTML/CSS to PDF.
4. Output is written to local or cloud storage via `ArtifactManager`.

### Notes

- WeasyPrint does not require Chromium, which makes it Databricks-friendly.
- If you need custom fonts, ensure they are available in the environment or embed them in templates.

## Excel reports

- Base class: `ExcelReportFormatter` (`src/novus_calls/report/excel_report_base.py`).
- Used by TL and trainer report services to format Excel outputs.

### Formatting features

- Shared color palette and fonts.
- Header styling helpers (`header_fmt_white`, `header_fmt_black`).
- Cell formatting for percentages, integers, floats, and strings.
- Conditional formatting for score columns (traffic-light scale).

### Date ranges

`DateRangeCalculator` provides helpers for computing date windows for daily and weekly reports.

## Customization

- PDF structure: edit HTML templates under `src/novus_calls/report/app/`.
- Excel styles: modify methods in `ExcelReportFormatter`.
- Report logic: update `tl_report_service.py` and `trainer_report_service.py`.
