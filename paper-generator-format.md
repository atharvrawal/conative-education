# Question paper format

A complete description of the file this system accepts, written so that an AI
assistant with no other information can produce a valid paper from it. Paste
this whole document into ChatGPT, Claude, or whatever you have, add the
instruction block in [section 6](#6-the-instruction-block-to-paste), and check
the result against [section 8](#8-checklist-before-you-upload).

Everything here is the behaviour of the actual importer, including the exact
wording of its error messages.

---

## 1. What this produces

A **zip file** containing one text file — `test.yaml` (or `test.json`) — that
describes the whole paper, plus any image files that paper refers to. You
upload that zip in the admin panel: create the test first, open it, and use its
upload control. The paper is checked as a whole and either imported completely
or rejected completely, with every problem listed at once. A re-upload replaces
the previous paper for that test entirely.

A paper with no diagrams is a zip containing a single file. That is a perfectly
normal paper, and it is what an AI can produce unaided.

---

## 2. The schema

### The file itself

| | |
| --- | --- |
| Name | `test.yaml`, `test.yml` or `test.json` — nothing else is recognised |
| Encoding | UTF-8. Non-English text is fine |
| Location | At the top of the zip, or one folder down; both work |
| Size | Under 8 MB, in a zip under 64 MB that unpacks to under 128 MB |

Only one such file may be in the archive. Two gives you
`more than one test file in the archive: ...`; none gives you
`no test.yaml, test.yml or test.json found in the archive`.

YAML and JSON are equally valid and describe exactly the same structure. YAML
is easier to read and its errors report a line number, so prefer it.

### Top level

Exactly two keys are recognised, and **any other key is an error**:

```yaml
test:        # optional — settings for the test as a whole
questions:   # required — the list of questions
```

`questions` missing gives `questions is required`. An unrecognised key gives
`unknown field 'title'`, and if it looks like a near-miss for a real one you
get a suggestion: `unknown field 'tolerence' — did you mean 'tolerance'?`

There is no field for an author, a subject, a difficulty label, a section
heading, an instruction sheet, a total-marks figure, or an answer key at the
end. Adding any of those fails the upload. **Do not invent fields.**

### The `test:` block — all optional

You create the test in the admin panel before uploading, so it already has a
name, a duration and a time window. Anything you put here *overwrites* what is
there. Anything you leave out is left alone. The whole block may be omitted.

| Field | Type | Rules and what a mistake says |
| --- | --- | --- |
| `name` | text | Cannot be blank. `name cannot be blank` |
| `duration_minutes` | whole number | At least 1. `duration_minutes must be a whole number of minutes, at least 1` |
| `opens_at` | text | `"YYYY-MM-DD HH:MM"`, Indian Standard Time, quoted. `'next tuesday' is not a date and time — use "YYYY-MM-DD HH:MM" (IST)` |
| `closes_at` | text | Same format, and must be later than `opens_at`. `closes_at must be after opens_at` |
| `groups` | list of text | Class names that **already exist** in the admin panel. `group 'does-not-exist' does not exist — create it before uploading` |

An AI writing a paper usually should not include `groups` at all — it cannot
know which classes exist on your installation, and a wrong guess fails the
whole upload. Assign classes in the admin panel instead.

### A question

`questions:` is a list, and it must not be empty. Order in the file is the
order students see. Every question is a mapping with these fields and no
others:

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `type` | **yes** | text | Exactly `mcq_single` or `numeric` |
| `text` | **yes** | text | The question itself. May contain LaTeX — [section 5](#5-latex) |
| `marks_correct` | **yes** | number | Awarded for a right answer |
| `marks_incorrect` | **yes** | number | For a wrong answer. Normally negative or `0` |
| `marks_unattempted` | **yes** | number | For leaving it blank. Normally `0` |
| `image` | no | text | A path inside the zip — [section 4](#4-images) |
| `time_limit_seconds` | no | whole number | Per-question limit. `null`, `0` or `-1` all mean untimed |
| `options` | mcq only | list | At least two. **Forbidden on numeric** |
| `answer` | **yes** | text or number | An option key for mcq, a number for numeric |
| `tolerance` | numeric only | number | **Forbidden on mcq** |

**There are no defaults.** A field that is not optional above must be written
out on every single question, even when its value is the same every time. In
particular all three marks fields are required on all questions of both types:

```
marks_correct is required on every question
marks_incorrect is required on every question
marks_unattempted is required on every question
```

Marks may be negative or fractional, must be between -999.99 and 999.99, and
may not have more than two decimal places — `4.005` gives
`marks_correct has more than two decimal places`.

`time_limit_seconds: 45` gives that question 45 seconds. Omitting it, or
writing `null`, `0` or `-1`, leaves it untimed. Any other value at or below
zero is treated as a typo: `-30 is not a valid time limit — use null, 0 or -1
for untimed`.

### `mcq_single` — one correct option

```yaml
options:
  - key: a
    text: "$10\\ ms^{-1}$"
  - key: b
    text: "$14\\ ms^{-1}$"
answer: "b"
```

- **At least two options.** Fewer gives `an mcq_single question needs at least
  two options`.
- Each option needs `key` and `text`. `image` is allowed. Nothing else is.
- A missing option label gives `text is required`; a blank one gives
  `text cannot be blank`.
- **Keys must be unique** within a question:
  `option key 'a' is used twice in this question`.
- **`answer` must be quoted text that exactly matches one of the keys.**
  Otherwise: `answer 'c' does not match any option key (options are: a, b)`.
- `tolerance` on an mcq gives `tolerance only applies to numeric questions`.

> **Quote your keys.** Keys are text, but YAML silently turns some bare words
> into other things. `key: on` and `key: 1` both fail with
> `key must be text — quote it if it is a bare number or a word like 'on'`.
> Writing `key: "1"` and `answer: "1"` is fine. Plain lowercase letters
> `a b c d` never have this problem, and are what the examples use.

### `numeric` — a typed-in number

```yaml
answer: 9.81
tolerance: 0.01
```

- **`answer` must be an unquoted number.** `answer: "9.81"` gives
  `a numeric answer must be a number, unquoted`.
- **`tolerance` is required.** Leaving it out gives `tolerance is required on
  numeric questions — use 0 for an exact match`. It must be zero or greater.
  A student's response counts as correct when it is within `tolerance` of
  `answer`. Use `0.01` for a two-decimal answer, `0` for a whole number that
  must match exactly.
- `options` on a numeric question gives
  `a numeric question cannot have options`.

### How rejection looks

Nothing is imported unless everything is valid. You get every problem at once,
each with the line it is on, like:

```
the file was rejected with 2 problems — nothing was imported
  line 3   questions[0].tolerance: tolerance is required on numeric questions — use 0 for an exact match
  line 8   questions[0].answer: a numeric answer must be a number, unquoted
```

`questions[0]` is the first question in the file, counting from zero.

One error is not about your file at all: **this test already has attempts on
it, so its questions are frozen — reset or delete those attempts first.** Once
a student has started a test, its paper cannot be changed.

---

## 3. Worked examples

A complete, valid file. Copy it, replace the questions, and it will upload as
it stands.

```yaml
test:
  name: "JEE Mock 01 — Kinematics & Constants"
  duration_minutes: 60
  opens_at: "2026-09-10 10:00"   # IST
  closes_at: "2026-09-10 13:00"  # IST

questions:
  # --- multiple choice, timed, with a figure -------------------------------
  - type: mcq_single
    text: "A particle moves such that $x = 3t^2 + 2t$. Find $v$ at $t = 2\\ \\mathrm{s}$."
    image: "images/q1.png"
    time_limit_seconds: 120
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: "$10\\ ms^{-1}$"
      - key: b
        text: "$14\\ ms^{-1}$"
      - key: c
        text: "$8\\ ms^{-1}$"
      - key: d
        text: "$12\\ ms^{-1}$"
    answer: "b"

  # --- multiple choice, untimed, no figure ---------------------------------
  - type: mcq_single
    text: "For $\\int_{0}^{\\pi/2} \\sin^2 x\\, dx$, the value is:"
    time_limit_seconds: null     # untimed
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: "$\\dfrac{\\pi}{4}$"
      - key: b
        text: "$\\dfrac{\\pi}{2}$"
      - key: c
        text: "$1$"
      - key: d
        text: "$\\dfrac{\\pi}{8}$"
    answer: "a"

  # --- numeric, tolerant to two decimal places -----------------------------
  - type: numeric
    text: "Acceleration due to gravity, in $ms^{-2}$, to 2 decimal places."
    time_limit_seconds: 0        # 0, -1 and null all mean untimed
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 9.81
    tolerance: 0.01

  # --- numeric, exact whole number, timed ----------------------------------
  - type: numeric
    text: "A body starts from rest with $a = 2\\ ms^{-2}$. Its displacement after $5\\ \\mathrm{s}$, in metres."
    time_limit_seconds: 90
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    answer: 25
    tolerance: 0
```

The same paper in JSON, if you prefer it — save as `test.json`:

```json
{
  "test": { "name": "JEE Mock 01", "duration_minutes": 60 },
  "questions": [
    {
      "type": "mcq_single",
      "text": "For $\\int_{0}^{\\pi/2} \\sin^2 x\\, dx$, the value is:",
      "marks_correct": 4,
      "marks_incorrect": -1,
      "marks_unattempted": 0,
      "options": [
        { "key": "a", "text": "$\\dfrac{\\pi}{4}$" },
        { "key": "b", "text": "$\\dfrac{\\pi}{2}$" }
      ],
      "answer": "a"
    },
    {
      "type": "numeric",
      "text": "Acceleration due to gravity, in $ms^{-2}$, to 2 decimal places.",
      "marks_correct": 4,
      "marks_incorrect": 0,
      "marks_unattempted": 0,
      "answer": 9.81,
      "tolerance": 0.01
    }
  ]
}
```

Note that JSON needs `\\` where YAML also needs `\\`, but JSON has no comments
and no unquoted `null` shorthand for untimed — just leave `time_limit_seconds`
out instead.

---

## 4. Images

An image path is **relative to the folder holding the test file inside the
zip**. Both of these layouts work, and neither is preferred:

```
paper.zip                       paper.zip
├── test.yaml                   ├── test.yaml
└── images/                     └── q1.png
    └── q1.png
```
```yaml
image: "images/q1.png"          image: "q1.png"
```

Allowed types: `.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`. Options may carry an
`image` too, in exactly the same way — useful when the choices are diagrams.

What goes wrong:

- `'images/q1.png' is not in the archive` — the path does not match a file in
  the zip. Check spelling, capitalisation, and that you zipped the images.
- `'x.bmp' is not an allowed image type (allowed: .gif, .jpeg, .jpg, .png, .webp)`
- `'../secret.png' points outside the folder holding the test file`

### An AI cannot draw your diagrams

This is the one thing to be careful about when a paper is generated for you.
An assistant writing a paper has no way to produce a real circuit diagram, ray
diagram, or graph, and **an `image:` line pointing at a file that does not
exist fails the whole upload**. So the rule for generated papers is:

- For text-only questions, **omit `image` entirely.** Do not write
  `image: null`, a placeholder name, or a description in the field.
- Where a question genuinely needs a figure, the assistant should write the
  question without an `image` line and list it separately afterwards, in prose,
  saying what the figure should show and which question number it belongs to.

You then draw or scan those figures, save them into `images/`, and add the
`image:` line yourself. The instruction block in the next section tells the
assistant to work this way.

---

## 5. LaTeX

Maths is written between dollar signs and is typeset when the student sees it.

| Form | Use |
| --- | --- |
| `$ ... $` | Inline, within a sentence. **Must stay on one line** |
| `$$ ... $$` | Displayed on its own line, centred. May span lines |

**Backslashes must be doubled inside double-quoted strings.** This is the
single most common mistake. In YAML, `"\\int"` produces the LaTeX `\int`,
while `"\int"` is a broken escape. Three real examples, exactly as they appear
in a working paper:

```yaml
text: "A particle moves such that $x = 3t^2 + 2t$. Find $v$ at $t = 2\\ \\mathrm{s}$."
text: "For $\\int_{0}^{\\pi/2} \\sin^2 x\\, dx$, the value is:"
text: "$\\dfrac{\\pi}{4}$"
```

Those render as $x = 3t^2 + 2t$, the integral of $\sin^2 x$, and a fraction.

Two ways to avoid the doubling entirely, both valid:

```yaml
# single quotes: backslashes are literal, so write LaTeX exactly as-is
text: 'For $\int_{0}^{\pi/2} \sin^2 x\, dx$, the value is:'

# a block scalar, for a long question
text: |
  A block of mass $m$ slides down a frictionless incline of angle
  $\theta$. Find its acceleration.
```

Maths that cannot be typeset is shown as the raw text you wrote rather than as
an error, so a broken formula will not stop the upload — it will simply look
wrong to the student. Read your questions on screen after uploading.

---

## 6. The instruction block to paste

Paste **this whole document** into the assistant first, then this, with the
four bracketed placeholders filled in:

```text
Using the question paper format document above, write a question paper for me.

Subject:            [e.g. Physics]
Number of questions: [e.g. 20]
Topics:             [e.g. 8 on kinematics, 7 on Newton's laws, 5 on work and energy]
Difficulty:         [e.g. JEE Main level, with the last three harder]

Requirements:

1. Follow the schema in that document exactly. Every question must carry type,
   text, marks_correct, marks_incorrect, marks_unattempted, and — for
   mcq_single — at least two options and an answer matching one of their keys,
   or — for numeric — an unquoted numeric answer and a tolerance.
2. Use only the fields listed in that document. Do not add fields of your own
   such as difficulty, topic, subject, explanation or solution, and do not add
   section headings. Any extra field causes the upload to be rejected.
3. Use marks_correct: 4, marks_incorrect: -1, marks_unattempted: 0 for
   multiple-choice questions, and marks_correct: 4, marks_incorrect: 0,
   marks_unattempted: 0 for numeric questions, unless I have said otherwise.
4. Do not include an image field on any question. If a question genuinely needs
   a diagram, write it without an image field and then, after the code block,
   list those questions by their number and describe what each figure should
   show. Never invent a filename.
5. Do not include a groups field.
6. Write LaTeX between single dollar signs, and double every backslash inside
   double-quoted strings, exactly as the examples in the document do.
7. Vary the position of the correct answer across the options; do not let it
   sit at the same key repeatedly.
8. Check before answering that each mcq answer value is one of that question's
   own option keys, and that every numeric question has a tolerance.

Output only the YAML, in a single fenced code block, with no commentary before
it. Any notes about figures go after the code block.
```

---

## 7. Packaging

Save the assistant's YAML as `test.yaml`. Then, from a terminal in the folder
holding it:

**No images:**

```bash
zip paper.zip test.yaml
```

**With images**, keeping the paths the file refers to:

```bash
mkdir -p images          # put your figures in here first
zip -r paper.zip test.yaml images
```

Check what you built before uploading it:

```bash
unzip -l paper.zip
```

You want to see exactly this shape — `test.yaml` at the top, not buried:

```
    Length      Date    Time    Name
---------  ---------- -----   ----
     1421  2026-09-04 11:02   test.yaml
      765  2026-09-04 11:02   images/q1.png
```

On **Windows**, select `test.yaml` and the `images` folder together, right-click,
and choose *Send to → Compressed (zipped) folder*. Select the files, not the
folder that contains them — though if you do zip the containing folder, that
works too, as long as there is only one test file inside it.

On **macOS**, select both, right-click, *Compress*. The extra `__MACOSX` entries
the Finder adds are ignored.

Then in the admin panel: create the test, open it, and upload `paper.zip`.

---

## 8. Checklist before you upload

Read down the file once. Almost every rejection is one of these.

**Every question, both types**

- [ ] `type` is exactly `mcq_single` or `numeric`
- [ ] `text` is present and not blank
- [ ] All three of `marks_correct`, `marks_incorrect`, `marks_unattempted` are
      written out — on **every** question, with no exceptions
- [ ] No invented fields: no `difficulty`, `topic`, `subject`, `explanation`,
      `solution`, `section`, `id` or `number`

**Multiple-choice questions**

- [ ] At least two options, each with both `key` and `text`
- [ ] No two options in one question share a key
- [ ] `answer` is **quoted** and is character-for-character one of that
      question's own option keys — check each one against its own list, not
      the previous question's
- [ ] No `tolerance`

**Numeric questions**

- [ ] `answer` is a bare number with no quotes: `9.81`, not `"9.81"`
- [ ] `tolerance` is present on every one of them, `0` for an exact match
- [ ] No `options`

**Images**

- [ ] Every `image` path names a file that is actually in the zip
- [ ] No `image` line was invented for a figure that does not exist
- [ ] Paths are relative to `test.yaml`, spelled and capitalised identically

**The file and the zip**

- [ ] Named `test.yaml`, `test.yml` or `test.json`, and only one of them
- [ ] Only `test` and `questions` at the top level
- [ ] `unzip -l` shows the test file at the top of the archive

If the upload is rejected anyway, the list it gives you names the line and the
question number for each problem, and fixing them all and re-uploading is safe
— nothing was written the first time.
