# Software Inc — guides from the source

Two self-contained HTML guides for **Software Inc** (Beta 1.8.42), both reconstructed from the
shipped `Assembly-CSharp.dll` and the game's own data assets.

| File | For | Style |
|---|---|---|
| [`Software Inc - Player's Handbook.html`](Software%20Inc%20-%20Player's%20Handbook.html) | **Playing the game** — what to do, what to prioritise, what to avoid | Plain language, no formulas, ~25 min read |
| [`Software Inc - The Score Book.html`](Software%20Inc%20-%20The%20Score%20Book.html) | **Understanding the mechanics** — every formula, constant and data table | Technical reference, source citations |

Start with the Handbook if you just want to play better. Open the Score Book when you want to know
*why* something works, or to look up exact numbers.

The game installation at `C:\Program Files (x86)\Steam\steamapps\common\Software Inc` was **only read**.
Nothing in it was created, modified or deleted.

---

## Player's Handbook (the non-technical one)

Thirteen chapters, written to be read in one sitting:

| Chapter | Contents |
|---|---|
| 01 | Your first two hours — a safe, boring, profitable first project |
| 02 | The loop: design → alpha → beta → release, and where every screen lives |
| 03 | People: roles, skills, teams, morale, your first office, **training, HR, lead-designer demands** |
| 04 | Money: income, salaries, contracts, **loans, taxes, insurance & safety**, the survival plan |
| 05 | Selling: marketing timing, pricing, subscriptions, release windows, updates/add-ons/sequels, **ports, publishers, support** |
| 06 | Servers and hardware: hosting, bandwidth, SCM, **device economics**, manufacturing checklist |
| 07 | Reviews and reputation — **awards, business reputation**, recovering from a bad release |
| 08 | Technology: staying current without over-spending |
| 09 | Growing: second team, series, platforms, hardware, publishing, stocks |
| 10 | Secrets: quirks, hidden effects and the three-star features that matter |
| 11 | **How to win: software, hardware and subscription playbooks** |
| 12 | Ten mistakes that end studios |
| 13 | Cheat sheet: phase table, first-year plan, glossary |

## The Score Book (the technical one)

A reverse-engineered reference that documents every score, formula and data table the game uses to
judge software — reconstructed from the shipped `Assembly-CSharp.dll` and the game's own data assets.

---

## What the guide contains

| Part | Contents |
|---|---|
| 00 | What "score" means in this game — the five different numbers the UI shows |
| 01 | Spec points: how features convert into score (level impact table, submarket split) |
| 02 | Real quality: design-pass quality ceilings, alpha accrual, `FinalQualityCalc` |
| 03 | Bugs: ceiling, generation, the `bestProgrammerSkill` fix cap, `BugScore`, refunds |
| 04 | Reviews: the in-development peer review vs. the cosmetic press review |
| 05 | Sales: the demand equation, quality share, seasonality, price curves, retention |
| 06 | Reputation, fans, awareness, marketing and the difficulty settings |
| 07 | Data bible: all 12 software types, every category, every feature, add-ons, manufacturing |
| 08 | Calculators (review simulator, quality/bugs builder, team size, sales estimator, price curve) |
| 09 | The playbook and five myths the source code kills |
| A–D | Constants registry, formula index, method, glossary |

The calculators implement the decompiled formulas directly (verified by `tools/test_app.js`).

---

## Workspace layout

```
Software Inc - The Score Book.html   ← the deliverable (open this)
guide/
  shell.html        page shell (head, sidebar, script tags)
  content.html      all written guide content
  style.css         theme
  app.js            data tables + calculators (formulas transcribed from the game)
tools/build_guide.py  assembles everything into the single HTML file
data/
  gamedata.json     all software types / categories / features / add-ons from the game assets
  reviewers.json    the seven press reviewers and their weight tables
  assets_index.json inventory of the 202 extracted game text assets
assets/             202 TextAssets extracted from Software Inc_Data/resources.assets
analysis/           detailed reverse-engineering reports (quality chain, dev pipeline,
                    market & sales, reviews & reputation) with verbatim code and file:line,
                    plus assembly-identity.json (REA artefact identity + SHA-256)
src/                decompiled C# of Assembly-CSharp.dll (902 top-level types)
src_firstpass/      decompiled C# of Assembly-CSharp-firstpass.dll (195 types, incl. FloatInterpolator)
tools/
  Decomp/           offline ILSpy-engine decompiler (ICSharpCode.Decompiler 9.1)
  UnityText/        minimal Unity SerializedFile reader → TextAsset extractor
  tyd.py            parser for the game's TYD data format
  extract_data.py   assets → data/*.json
  build_guide.py    guide template + data → final HTML
  validate_guide.py structural checks (tag balance, required ids, JSON payloads)
  test_app.js       33 assertions over the transcribed formulas
  test_render.js    35 assertions over the data-table/calculator renderers
  feed/ + .nuget/   offline NuGet feed used to build the decompiler without network
```

## Rebuilding

```powershell
# 1. re-extract the game's text assets (needs the installed game)
tools\UnityText\bin\Debug\net10.0\UnityText.exe extract `
  "C:\Program Files (x86)\Steam\steamapps\common\Software Inc\Software Inc_Data\resources.assets" assets

# 2. rebuild the JSON data tables
python tools\extract_data.py

# 3. rebuild + verify the HTML
python tools\build_guide.py
python tools\validate_guide.py
node tools\test_app.js
node tools\test_render.js
```

To re-decompile the game assembly: `tools\Decomp\bin\Debug\net10.0\Decomp.exe dump <Assembly-CSharp.dll> src`.

## Method

1. The build is Unity 2018.4.36f1 **Mono**, so the game logic is managed IL. `Assembly-CSharp.dll`
   (5.5 MB, 3,279 types) was decompiled with the ILSpy engine (`ICSharpCode.Decompiler` 9.1) using a
   local, offline NuGet feed.
2. The numeric game data ships as Unity `TextAsset`s inside `resources.assets`. A small SerializedFile
   parser extracted all 202 text assets and a TYD parser turned them into JSON — so every number in
   the data tables is the shipped value, not a hand transcription.
3. The score chain was traced from `SoftwareWorkItem` / `SoftwareAlpha` (design → alpha → beta) through
   `SoftwareProduct` (quality, decay) into `MarketSimulation` (sales) and `FinalReviewGenerator`
   (press reviews). Every formula in the guide cites its `file:line`.

## Known limits

* Static analysis of one build (Beta 1.8.42); runtime state (market mix, AI quality, per-save
  difficulty) is presented as formulas, not fixed values.
* The sales estimator deliberately ignores competitors, existing userbase and followers, so it is an
  upper bound meant for comparing decisions.
* Decompiled code is reconstructed C#, not original source; names of compiler-generated members and
  some expression shapes differ from the developers' code.
