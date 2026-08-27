# KiCad Template

A shared project template, drawing sheet, color theme and project layout for
hardware projects. It is meant to be pulled into a board repository as a
submodule, so every project ends up with the same first sheets, the same title
block, the same look and the same folder structure.

```
kicad-template/
├── .gitignore                         # KiCad ignore rules, copy into the board repo
├── board-template/                    # KiCad project template, see below
├── colors/
│   └── altium.json                    # KiCad color theme, Altium-like
└── worksheet/
    └── sleman_template.kicad_wks      # drawing sheet / title block
```

## Project template

`board-template/` is a KiCad project template holding the three sheets every
board starts with:

| Page | Sheet | Content |
| --- | --- | --- |
| 1 | root | cover page: 40-entry index, state legend, notes, design considerations |
| 2 | `sheet2.kicad_sch` | block diagram, six placeholder blocks |
| 3 | `sheet3.kicad_sch` | power budget, power tree with the load on each branch |

Both diagrams use the same palette. Block fill says what the block is:

| Fill | Block |
| --- | --- |
| pale yellow `255 255 194` | connector or off-board interface |
| dark gray `72 72 72` | circuit drawn on this board, white text |
| light gray `194 194 194` | main device |

Line color says what the line carries: dark red `132 0 0` unregulated input,
red `255 0 0` regulated rail, blue `0 0 132` data. Stroke `0.5` on blocks,
`1.5` on lines.

Starting a board:

1. Create the repository with an empty `hardware/` folder and add this repo as a
   submodule under `external/kicad-template`.
2. **File > New Project from Template**, click **Select Templates Directory**
   and pick the clone of this repo. Choose **board-template**. KiCad lists every
   subfolder of the directory it is given, so `colors/` and `worksheet/` show up
   next to it; ignore them.
3. In the file dialog, navigate into `hardware/`, type the board name and
   **untick "Create a new folder for the project"**. The project files land
   straight in `hardware/`, renamed after the board.
4. Fill in Page Settings and the text variables below, then edit the index on
   the cover page as sheets are added.

The template ships `sym-lib-table` and `fp-lib-table` pointing at
`${KICAD_MYLIB}`, so define that path once per machine under
**Preferences > Configure Paths**.

The index on the cover page is plain text: page numbers and dotted lines are not
linked to the real sheets, so both have to be typed by hand.

## Drawing sheet

A project started from `board-template/` already has step 1 and 3 done and the
variables filled with placeholders; the steps below are the full setup, for
projects that did not start from the template.

1. **Point the project at the sheet.** In the schematic editor go to
   **File > Page Settings** and set **Page Layout Description File** to:

   ```
   ${KIPRJMOD}/../external/kicad-template/worksheet/sleman_template.kicad_wks
   ```

   `${KIPRJMOD}` is the folder holding the `.kicad_pro`, so the path above
   assumes the KiCad files live in `hardware/` and the submodule in `external/`.
   Adjust it if the layout differs. Repeat the same step in the PCB editor under
   **File > Page Settings**.

2. **Fill in the page variables.** Still in **Page Settings**:

   | Field | Content |
   | --- | --- |
   | Issue Date | Date the project started |
   | Revision | Current board revision |
   | Title | Board name |
   | Company | Company or organization |
   | Comment 1 | `For more designs, visit` |
   | Comment 2 | Personal website or portfolio URL |
   | Comment 3 | Company URL |

3. **Add the text variables.** Go to **Schematic Setup > Project > Text
   Variables** and create these five:

   | Variable | Content | Used by |
   | --- | --- | --- |
   | `DESIGNER` | Who drew the board | drawing sheet |
   | `RELEASE_DATE` | Date of the current state, `DD-MMM-YYYY` | cover page |
   | `STATE` | `DRAFT`, `PRELIMINARY`, `CHECKED` or `RELEASED` | cover page |
   | `VARIANT` | Assembly variant, or `NO VARIANT` | drawing sheet, cover page |
   | `YEAR` | Copyright year | drawing sheet |

   Text variables are stored per project, not per file, so adding them once in
   the schematic editor makes them available to the PCB editor too.

## Color theme

`colors/altium.json` gives the schematic editor, PCB editor and 3D viewer an
Altium-like appearance: pale yellow symbol bodies with maroon outlines on a white
sheet, and a black PCB canvas with a red top layer, blue bottom layer and yellow
silkscreen. Themes are stored per user instead of per project, so the file cannot
travel inside the project and has to be copied once per machine into:

| OS | Destination |
| --- | --- |
| Linux | `~/.config/kicad/9.0/colors/` |
| Windows | `%APPDATA%\kicad\9.0\colors\` |

Restart KiCad and select **altium** under **Preferences > Schematic Editor >
Colors** and again under **Preferences > PCB Editor > Colors**. The pale yellow
body fill only renders on shapes whose fill is set to `Background`, so any symbol
drawn with fill `None`, which includes most of the stock `Device` library, stays
transparent until it is changed in the Symbol Editor.

## Project layout

The structure used on every board.

```
board-name/
├── .gitmodules
├── .gitignore
├── README.md                  # what the board is, current rev, how to build it
├── LICENSE
├── CHANGELOG.md               # one entry per revision
├── docs/
│   ├── datasheets/            # parts used on the board
│   └── design-notes.md        # decisions, calculations, derating
├── hardware/
│   ├── board-name.kicad_pro
│   ├── board-name.kicad_sch   # cover page
│   ├── board-name.kicad_pcb
│   ├── sheet2.kicad_sch       # block diagram
│   ├── sheet3.kicad_sch       # power budget
│   ├── sym-lib-table          # points at the library submodule
│   ├── fp-lib-table
│   ├── schematic/             # exported schematic PDFs, versioned
│   ├── simulation/
│   └── production/
│       └── v1.0/              # gerbers, drill, BOM, CPL, PDF, STEP
├── mechanical/                # enclosure, assembly STEP
└── external/                  # every submodule lives here
    ├── kicad-lib/
    ├── kicad-template/
    └── bom-formatter/
```

`hardware/production/` gets one folder per released revision, frozen once the
files are sent to the fab.

### Boards with firmware

When the board carries a microcontroller, the firmware lives in the same
repository. Add a `firmware/` folder next to `hardware/`:

```
board-name/
├── hardware/
├── firmware/
│   ├── src/
│   ├── include/
│   ├── Makefile
│   └── README.md              # toolchain, how to build and flash
└── ...
```

Releases are tagged per side, `v1.0-hw` and `v1.2-fw`, and `CHANGELOG.md` notes
which firmware version each board revision was tested with.
