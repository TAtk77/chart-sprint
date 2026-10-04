# Chart Sprint

An untimed, self-paced training activity for Holter/ECG technicians. Each sprint gives the tech a random set of patients. They look each patient up in Cardiologs, then answer five dropdown questions about that study. Scoring and the answer key are automatic.

Everything runs in the browser. There is no server and no build step.

## Files

| File | What it does | Who edits it |
|---|---|---|
| `index.html` | The activity itself (layout, logic, styling) | Rarely |
| `patients_db.csv` | The patient library: one row per patient, raw chart numbers | You, often |
| `question_templates.csv` | Which questions exist, their wording, and their priority | You, occasionally |

All three files must sit in the same folder. `index.html` loads the two CSVs by filename.

## How it works

1. The page reads `patients_db.csv` and `question_templates.csv`.
2. Each time a tech clicks **Start Sprint**, it draws a random set of patients from the library.
3. For each patient it builds five questions from that patient's row. The correct answer is the value in the sheet, and the wrong answers are generated automatically.
4. Answers save in the tech's browser as they go, so they can close the tab and resume later.
5. **Submit Sprint** shows a score and a full review.

## Adding patients

Add a row to `patients_db.csv`. Do not change the header row.

- **`patient`**: the de-identified ID the tech will search for. Each ID should appear once.
- **Blank vs. 0**: a blank cell means "not measured / not applicable," so that question is skipped. A `0` means the count really was zero and can still be asked.
- **`rhythm`**: type it how it should appear in the dropdown (e.g. `Sinus Bradycardia`). `sinus`, `afib`, and `svt` are cleaned up automatically. Anything else displays exactly as typed.
- **`misc`**: notes for yourself (signal loss, etc.). It is never used in questions.
- Save as **CSV (UTF-8)** with the same filename, then push to GitHub.

## Which questions get asked

Every patient gets five questions:

1. Maximum HR
2. Minimum HR
3. Average HR
4. Underlying rhythm
5. One "How many ___ were there?" question

The 5th question is the **first** count in the `count_pick` list (in `question_templates.csv`) that has a value for that patient. Currently PSVC is first, then PVC, SVT, VT, and so on.

### Overriding the 5th question for one patient

Add a column named `focus` to `patients_db.csv`. For that patient, type the column name of the count you want asked instead (e.g. `PVC count`, `VT`, `v bigeminy`). Capitalization doesn't matter. If that patient has no value for the field you named, it falls back to the default.

## Changing questions (`question_templates.csv`)

| Column | Meaning |
|---|---|
| `field_key` | Matches a `patients_db.csv` column header, lowercased, with spaces and punctuation turned into underscores (`# of morph` becomes `of_morph`) |
| `prompt` | The question text. Keep `___` where the dropdown goes. In `count_pick` rows, keep `{label}` too |
| `type` | `numeric_hr`, `numeric_count`, `categorical`, or `yesno`. Controls how wrong answers are generated |
| `tier` | `core` = always asked, in file order. `count_pick` = candidates for the flexible 5th question, in file order. `extra` = never asked automatically |
| `label` | Text swapped into `{label}` for `count_pick` rows (e.g. `PVCs`) |

**To add a new trackable field:** add a column to `patients_db.csv`, then add a matching row here.

## Settings in `index.html`

Near the top of the script:

- `PATIENTS_PER_SPRINT`: how many patients each sprint draws (default 10)
- `QUESTIONS_PER_PATIENT`: questions per patient (default 5)

## Deploying (GitHub Pages)

1. Put the three files in a repo. The activity file must be named `index.html`.
2. In the repo, go to **Settings > Pages**. Set Source to "Deploy from a branch," branch `main`, folder `/ (root)`.
3. After a minute or two it is live at `https://<username>.github.io/<repo-name>/`.
4. To update patients or questions later, edit the CSV and push. No other change is needed.

## Things to know

- **Scores and progress are stored in each tech's own browser** (localStorage). The leaderboard is per-device, and nothing is sent anywhere. Clearing browser data clears it.
- **Opening `index.html` by double-clicking it won't load your CSVs.** Browsers block that, so it falls back to a small built-in sample set. Test through GitHub Pages or a local server (`python -m http.server`).
- **Rhythm wrong answers come from a short fixed list** in the script (`RHYTHM_VOCAB`). If you start using rhythms outside that list, add them there.
- **Dark mode** follows the device setting by default, and techs can switch it with the toggle in the header.
- Wrong answers for heart rates and counts are generated randomly each time, so two techs will see different distractors for the same patient.

## Credits

Built by T. Atkinson, CCT. Pivotal Health CCT Training.
