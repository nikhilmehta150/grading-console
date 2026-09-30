A single-page web app that lets an instructor upload student marks from an Excel file, choose a course, set grade ranges, review the distribution and export final grades as a CSV file.

## How to use

1. Enter the instructor name.
2. Upload the marks file (`.xlsx` or `.xls`) with three columns: BITS ID, Course, Total Marks (0–100, whole numbers). Students with an NC grade are not included.
3. Select a course.
4. Review the analytics and adjust the grade ranges if needed.
5. Click **Finalize & Download** to get the CSV.

Default ranges: A 80–100, A- 70–79, B 60–69, B- 50–59, C 40–49, C- 30–39, D 20–29, E 0–19.

## Stage 1: Bug Fix Log

| # | Bug / Issue Identified | How I Reproduced It | Root Cause | Fix Implemented | How I Tested the Fix |
|---|---|---|---|---|---|
| 1 | Course list showed duplicate entries, and old courses stayed after a second upload | Uploaded an Excel file with several students per course, then uploaded another file | One dropdown option was added per student row, and the dropdown was never cleared on a new upload | Cleared the dropdown on every upload and added only unique, sorted course names | Uploaded two files one after another and confirmed each course appeared once with no leftovers |
| 2 | Min and Max values in the statistics were swapped | Selected a course and compared the Min and Max boxes with the actual marks | HTML element IDs were attached to the wrong labels (`id="max"` under Min and `id="min"` under Max) | Renamed the IDs to `statMin` and `statMax` and attached them to the correct labels | Checked Min and Max against the lowest and highest marks in the file |
| 3 | `.xlsx` files could not be selected | Opened the file picker, where `.xlsx` files were not shown | The file input allowed only `.xls` | Changed it to `accept=".xlsx,.xls"` | Uploaded the sample `.xlsx` file successfully |
| 4 | Sample file rejected with "Missing required column(s)" | Uploaded the sample file, whose headers are "Student’s BITS ID" and "Total Marks (out of 100)" | Column names had to match exactly | Made column detection keyword based, ignoring spaces, punctuation and brackets | Uploaded the sample file and the courses loaded |
| 5 | Timer showed 00:00 for the first second after page load | Opened the page and watched the clock | `setInterval` runs its first update only after one second | Called the update function once immediately, then every second | Reloaded the page and confirmed the clock updates instantly |
| 6 | Histogram bars went outside the canvas for larger classes | Loaded a course with many students in one mark range | Bar height was a fixed value (`count × 12`) | Scaled bar heights to the tallest bin and added count labels | Tested with a larger dataset |
| 7 | Bell curve did not line up with the histogram bars | Compared the red curve with the bars | Spacing mismatch (30 px vs 32 px per bin) and a hardcoded vertical scale | Matched the spacing to bar centres and scaled the curve to the tallest bar | Compared curve and bars on different datasets |
| 8 | Grade ranges could leave marks ungraded | Set the A maximum to 90, which left 91–100 without a grade, while Download stayed enabled | Validation checked gaps between grades but not that ranges cover 0–100 | Added checks that A Max = 100 and E Min = 0, with clear error messages | Changed A Max and E Min and confirmed an error appeared and Download was disabled |
| 9 | Reset Range asked for confirmation twice | Clicked Reset Range | Two consecutive `confirm()` dialogs | Kept a single confirmation | Clicked Reset once and saw a single prompt |
| 10 | CSV export could break or be unsafe | Exported a CSV and opened it in Excel | Cells were not quoted or escaped, and `Course, X` had an extra space | Added proper quoting and escaping, and made text starting with `=`, `+`, `-` or `@` safe | Opened the exported CSV in Excel and checked all columns |
| 11 | Bell curve clipped into a flat box on small or narrow datasets | Uploaded a course with 2 students in the same mark range | Curve height was in raw density units, capped at a fixed pixel value, and the standard deviation was tiny | Curve now scales to the tallest bar, is aligned to bar centres, and uses a minimum width so it stays smooth | Tested with 2 students and with a wider spread |

## Stage 2: Enhancements

| # | Enhancement | User problem it solves | What it does |
|---|---|---|---|
| 1 | **Upload validation report** | Bad data in the Excel file used to fail silently or give wrong grades | Detects missing columns, non-numeric marks, marks outside 0–100, duplicate BITS IDs and decimal marks. Shows a report with row numbers. Invalid rows are skipped and listed, not hidden. Column names are matched flexibly |
| 2 | **Grade impact insight and student lookup** | Instructors could not see how their ranges differ from the defaults, or quickly answer a student's query | Shows how many students get a different grade than with the default ranges, adds a percentage to each grade count, and lets the instructor type a BITS ID to see that student's marks and grade |
| 3 | **Safer, richer CSV export** | The exported file had no context and could break in Excel | The CSV includes instructor, course, date, grade ranges used and a class summary (student count, min, max, average, median, grade distribution). Cells are quoted and escaped, the file name is based on the course, and download requires the instructor name |
| 4 | **Unsaved changes warning** | Custom ranges could be lost by closing or refreshing the tab | The browser asks for confirmation if custom ranges have not been downloaded yet |
| 5 | **Redesigned, accessible and responsive UI** | New instructors did not know the order of steps, the chart was hard to read, and the layout was hard to use on small screens or with a keyboard | Adds a 4-step progress indicator and an empty state. Grade cards, chips and the histogram share one colour system, with grade letters and counts on the bars. Mobile-friendly layout, labelled inputs, keyboard focus outlines and screen-reader error alerts |

## Notes

- Fractional marks are rounded to the nearest whole number and listed in the upload report.
- The core flow (upload, select course, analytics, grade ranges, validation, distribution, CSV export) works as in the original.
- Built with plain HTML, CSS and JavaScript. The SheetJS library is loaded from a CDN to read Excel files.
