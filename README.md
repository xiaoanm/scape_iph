# Interactive Programme Handbook (IPH)
## School of Chemical and Process Engineering — University of Leeds

An interactive, student-facing programme/module catalogue, adapted from the School of Civil Engineering IPH (https://mohsenhiwa.github.io/handbook-/). Students can browse programmes, search modules across all degrees, model their optional-module selections against pathway rules and credit limits, and print a PDF record of their choices.

**Key difference from the Civil version:** all content and rules now live in one data file, `data/programme_data.json`. The Civil site hard-coded programme specifications, staff titles, and List A/B/C option rules inside the HTML; here, you control all of it by editing the JSON — you should never need to touch `index.html`.

---

## 1. What's in this folder

```
scape-handbook/
├── index.html                 The entire application (HTML + CSS + JS). Don't edit for content changes.
├── data/
│   └── programme_data.json    ALL content: school name, programmes, modules, rules. This is YOUR file.
├── leeds-logo.png             (you add this) University logo shown in the header. If absent, header simply shows no logo.
└── README.md                  This file.
```

> **⚠ The supplied `programme_data.json` contains PLACEHOLDER modules** (marked "PLACEHOLDER" throughout). They exist only to demonstrate every feature — compulsory/optional modules, assessments, List A/B unlock rules, MSc structure. Replace them with the official SCAPE catalogue data before sharing with students.

---

## 2. Getting it online with full control (GitHub Pages)

1. Create a free GitHub account (or use your existing one).
2. Create a new **public** repository, e.g. `scape-handbook`.
3. Upload `index.html`, `README.md`, the `data/` folder, and your `leeds-logo.png` (drag-and-drop on the repo page → "Add file" → "Upload files").
4. Repo **Settings → Pages → Source: Deploy from a branch → Branch: main, folder: / (root) → Save**.
5. After ~1 minute the site is live at `https://<your-username>.github.io/scape-handbook/`.

**To change anything afterwards:** open `data/programme_data.json` on GitHub, click the pencil (✏️ Edit), make your change, and commit. The site updates within a minute or two (the app adds a cache-busting query string, so students always get the latest data on reload — it also auto-refreshes itself at midnight).

**Testing locally:** browsers block `fetch()` of local files, so double-clicking `index.html` shows a CORS error. Instead, run a tiny server in the folder:
```
python -m http.server 8000
```
then open http://localhost:8000.

**Tip:** after editing, paste the JSON into https://jsonlint.com (or any JSON validator) — a single missing comma will stop the whole site loading.

---

## 3. The data file, top to bottom

### 3.1 Site-level settings

```json
{
  "school_display_name": "School of Chemical and Process Engineering",
  "external_link": {                       ← optional sidebar button; delete to hide it
    "label": "View Assessment Details",
    "url": "https://..."
  },
  "staff_titles": {                        ← optional; maps names to titles everywhere they appear
    "Jane Example": "Dr",
    "John Sample": "Prof"
  },
  "programmes": [ ... ]
}
```

If a module leader's name isn't in `staff_titles`, their name is shown without a title.

### 3.2 A programme

```json
{
  "prog_id": 2001,                          ← any unique number
  "programme_title": "MEng, BEng Chemical Engineering",
  "programme_short_title": "ChemEng",       ← shown on global-search result cards
  "category": "Chemical Engineering Programmes (Undergraduate)",
  "programme_description": "…",
  "spec": {                                 ← optional; omit to hide the spec panel
    "code": "MEBE-CAPE",
    "ucas": "H810",
    "duration": "4 Years",
    "attendance": "Full Time",
    "total_credits": "480",
    "manager": "Jane Example",
    "email": "ugchemical@leeds.ac.uk"
  },
  "default_target_credits": 120,            ← credit cap per year unless a level overrides it
  "levels": [ ... ]
}
```

`category` controls the grouping in the programme dropdown. Programmes with the same category string appear under one heading, in the order they appear in the file. Use any headings you like, e.g. `"MSc Programmes (Postgraduate)"`.

### 3.3 A level (year)

```json
{
  "level": 3,                               ← unique number within the programme
  "year_label": "Year 3",                   ← what students see ("Year 1" for an MSc level-5, etc.)
  "target_credits": 120,                    ← selection cap & progress-bar target (180 for MSc)
  "option_groups": [ ... ],                 ← optional; see 3.5
  "modules": [ ... ]
}
```

### 3.4 A module

```json
{
  "module_code": "CAPE3000",
  "module_title": "Chemical Engineering Design Project",
  "module_leader": "John Sample",
  "module_description": "…",
  "credits": 40,
  "semester": 3,                            ← 1 = Semester 1, 2 = Semester 2, 3 = Full Year
  "compulsory": true,                       ← compulsory modules are auto-selected and locked
  "pass_for_progression": true,
  "notes": "Group design project.",         ← shown in the detail popup and on the PDF
  "other_school": "",
  "assessments": [
    {
      "description": "Final Design Report and Presentation",
      "short_description": "Report + presentation",
      "percent_of_module": 0.6,             ← 0–1; use 0 for formative work
      "credits": 24,
      "is_exam": false,
      "is_multi_submission": false,
      "submission_date": "05/05/2027"
    }
  ]
}
```

To **add** a module: copy an existing block, edit it, mind the commas. To **delete**: remove the block. To **modify**: edit in place. The same applies to whole levels and programmes.

### 3.5 Optional-module pathway rules (`option_groups`)

This replaces the Civil site's hard-coded List A/B/C logic. Add to any level:

```json
"option_groups": [
  {
    "id": "A",
    "name": "List A Optional Modules",
    "description": "You must select between 1 and 2 modules from this list.",
    "codes": ["CAPE3100", "CAPE3110", "CAPE3120"],
    "min": 1,
    "max": 2
  },
  {
    "id": "B",
    "name": "List B Optional Modules",
    "description": "You may choose up to 2 modules here, unlocked only after selecting a List A option.",
    "codes": ["CAPE3200", "CAPE3210"],
    "max": 2,
    "unlock": { "groups": ["A"], "min_total": 1 }
  }
]
```

How each field behaves:

| Field | Effect |
|---|---|
| `codes` | Module codes belonging to the list. These modules are rendered under their own section heading. |
| `name`, `description` | Section heading and explanatory text shown to students. |
| `max` | Hard cap: selecting beyond it is blocked with an alert. |
| `min` | Soft rule: shown as a red warning in the credit tracker, and **blocks printing the PDF** until satisfied. |
| `unlock` | The list stays locked until at least `min_total` modules are selected across the listed `groups`. Deselections that would break an unlock condition of an active list are blocked. Combined prerequisites work too, e.g. `{"groups": ["A","B"], "min_total": 2}` reproduces the Civil site's List C rule. |

Optional modules not in any group appear under a generic "Optional Modules" heading with no special rules. Levels without `option_groups` behave exactly like the simple levels on the Civil site.

---

## 4. Built-in behaviour (no configuration needed)

- **Global search** across all programmes (code, title, description, leader, assessment keywords like "Exam" or "Dissertation"), with "Jump to Programme Layout".
- **Credit tracker** with per-level targets, lock at the cap, and pathway-rule warnings.
- **Print to PDF** producing a clean selection record with the school name, date, group context per module, and the non-enrolment disclaimer.
- **Semester filter** (Full Year / Semester 1 / Semester 2) and within-year module search.
- **Enrolment disclaimer popup** on load, making clear this is a planning tool, not official enrolment.
- **Auto-refresh at midnight** and cache-busted data fetch, so edits propagate quickly.

## 5. Logo

Place the University of Leeds logo in the root as `leeds-logo.png` (a white-on-transparent PNG works best against the dark header). If the file is missing the header simply omits it — the site still works.

---

## 6. Excel workflow (edit modules in a spreadsheet)

You can maintain all content in `programme_data.xlsx` instead of editing JSON directly. The bundle includes:

```
programme_data.xlsx     Editable workbook holding all handbook content (pre-filled with the current data)
convert.html            Browser-based converter — no installation, nothing leaves your computer
```

### The cycle

1. Edit `programme_data.xlsx` in Excel.
2. Open `convert.html` in any browser (double-clicking the file works — no server needed) and drop the workbook onto it.
3. It validates the workbook, lists any **errors** (must fix — e.g. duplicate module codes, references to unknown programmes) and **warnings** (advisory — e.g. assessment weights not summing to 100%), then offers `programme_data.json` for download.
4. Replace `data/programme_data.json` in your GitHub repository. The live site updates within a minute.

The converter also works **in reverse**: drop a `programme_data.json` onto it to regenerate the Excel workbook. So whichever file is current, you can always rebuild the other — nothing gets out of sync permanently.

### The workbook's seven sheets

| Sheet | One row per | Key columns |
|---|---|---|
| **Settings** | site setting | `Key` / `Value`: school_display_name, external_link_label, external_link_url, year_begin, year_end |
| **StaffTitles** | staff member | `Name`, `Title` (Dr/Prof/…) — applied wherever that name appears |
| **Programmes** | programme | `prog_id` (unique number), `programme_title`, `category` (dropdown grouping), `default_target_credits`, `spec_*` columns (code, ucas, duration, attendance, total_credits, manager, email), `programme_description` |
| **Levels** | year of a programme | `prog_id`, `level`, `year_label` ("Year 1"…), `target_credits` (120 UG / 180 MSc) |
| **OptionGroups** | optional-module list | `prog_id`, `level`, `group_id` (A/B/…), `name`, `description`, `min`, `max`, `unlock_groups` (e.g. "A" or "A, B"), `unlock_min_total` |
| **Modules** | module | `prog_id`, `level`, `module_code`, `module_title`, `module_leader`, `credits`, `semester` (1, 2, or 3 = full year), `compulsory` (TRUE/FALSE), `pass_for_progression`, `option_group` (blank, or a group_id from OptionGroups), `notes`, `other_school`, `module_description` |
| **Assessments** | assessment task | `prog_id`, `module_code`, `description`, `short_description`, `percent_of_module` (0.3 or 30% both work), `credits`, `is_exam`, `is_multi_submission`, `submission_date` (dd/mm/yyyy) |

Notes on how the sheets connect:

- A module belongs to an optional list simply by putting the list's `group_id` in its `option_group` cell — you never maintain code lists by hand. The list's rules (min/max/unlock) live on the OptionGroups sheet.
- To **add** a module: add a row on Modules (and its assessment rows on Assessments). To **delete**: delete the rows. To **move** it between years or lists: change its `level` or `option_group` cell.
- If a level used on the Modules sheet has no row on the Levels sheet, it's created automatically with defaults (you'll get a warning so you can add labels/targets).
- Booleans accept TRUE/FALSE, yes/no, or 1/0. Dates are safest typed as text `dd/mm/yyyy`, but real Excel dates are converted correctly too.
- The converter runs entirely in your browser (it loads one open-source library, SheetJS, from a CDN) — module data is never uploaded anywhere.

`convert.html` can also be hosted in the same GitHub repository, so you can run the conversion from any machine at `https://<your-username>.github.io/scape-handbook/convert.html`.
