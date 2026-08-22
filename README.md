# KiCad Template

A shared drawing sheet, color theme and project layout for hardware projects. It
is meant to be pulled into a board repository as a submodule, so every project
ends up with the same title block, the same look and the same folder structure.

```
kicad-template/
├── .gitignore                         # KiCad ignore rules, copy into the board repo
├── colors/
│   └── altium.json                    # KiCad color theme, Altium-like
└── worksheet/
    └── sleman_template.kicad_wks      # drawing sheet / title block
```

## Drawing sheet

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
   Variables** and create these three:

   - `DESIGNER`
   - `VARIANT`
   - `YEAR`

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
│   ├── board-name.kicad_sch
│   ├── board-name.kicad_pcb
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
