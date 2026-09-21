# Variable-difficulty bank format

A complete description of the file a **variable-difficulty** test accepts,
written so that an AI assistant with no other information can produce a valid
bank from it. Paste this whole document into ChatGPT, Claude, or whatever you
have, add the instruction block in
[section 7](#7-the-instruction-block-to-paste), check the result against
[section 9](#9-checklist-before-you-upload), and — once it is uploaded — read
the questions on screen as
[section 10](#10-the-one-check-nothing-else-does-read-the-bank-on-screen)
describes.

This document is for variable-difficulty tests only. An ordinary test has its
own document, *Question paper format*, and the two files are not
interchangeable: a bank without difficulties is rejected here, and everything
below about the ladder has no meaning there.

Everything here is the behaviour of the actual importer, including the exact
wording of its error messages.

**If you read only two sections, read [2](#2-how-a-variable-difficulty-test-works)
and [6](#6-mathematics-and-latex).** Section 2 is what makes a bank good rather
than merely valid. Anything the importer can catch, it catches loudly and tells
you the line — but a difficulty label that does not match the question's actual
difficulty, like a badly written formula, uploads perfectly and is discovered by
a student.

---

## 1. What this produces

A **zip file** containing one text file — `test.yaml` (or `test.json`) — that
describes the whole bank, plus any image files it refers to. In the admin
panel, select **Create a test**, choose **Variable difficulty**, set how many
questions each student should answer, continue to **Questions**, and upload the
zip. The bank is checked as a whole and either imported completely or rejected
completely, with every problem listed at once. A re-upload replaces the
previous bank for that test entirely.

A bank with no diagrams is a zip containing a single file. That is a perfectly
normal bank, and it is what an AI can produce unaided.

---

## 2. How a variable-difficulty test works

A variable-difficulty test is not a paper. It is a **bank**, and no student
sees all of it.

* The student is shown **one question at a time**, and cannot go back. A
  question they have moved past is finished — there is no palette, no
  Previous, and no second look.
* Everybody starts on a **`med`** question.
* A **right** answer moves them **up** one rung. A **wrong** one, or one left
  unanswered, moves them **down** one.
* The ladder has five rungs — `easy`, `easy-med`, `med`, `med-hard`, `hard` —
  and **the ends hold**: a wrong answer on `easy` is followed by another
  `easy`, and a right one on `hard` by another `hard`.
* Within a rung the question is picked **at random** from the ones that student
  has not already seen, so two students on the same rung rarely get the same
  question.
* The test ends after the number of questions set in the panel, or when the
  time runs out, whichever comes first. There is no early submit.
* Each student is marked out of the questions they actually saw, not out of the
  whole bank.

Two consequences for how you write the questions:

**The difficulty labels have to be true.** They are not decoration and they are
not shown to anyone — they are the mechanism. A bank whose `easy` questions are
as hard as its `hard` ones produces a test that moves students up and down at
random, which is worse than a fixed paper. An `easy` question should be one
step of recall or one substitution; a `hard` one should need several steps or a
non-obvious idea; the three in between should genuinely sit between them.

**Order in the file means nothing.** Students are served questions in an order
nobody can predict, so do not group by topic in the hope it reads well, and do
not build a question that refers to "the previous question".

### How big the bank should be

Aim for about **five times** the number of questions each student will be
asked, spread evenly across the five difficulties — a 50-question test wants a
bank of about 250, roughly 50 at each rung. That is advice, not a rule, and
nothing checks it:

* A rung with nothing left on it falls back to the nearest rung that is not
  empty, keeping the direction the student was moving. A right answer on `hard`
  with no `hard` questions left gives a `med-hard` one; a wrong answer on
  `easy` with no `easy` left gives an `easy-med` one.
* A bank smaller than the target simply ends the test when it runs dry. A bank
  of 50 serves a 50-question test perfectly well — every student just ends up
  having seen all of it.
* A bank with only one rung filled works too. Every student gets that rung.

A thin bank costs variety, not correctness. The panel shows you the spread
across the five rungs after every upload, so you can see at a glance where the
bank is thin.

---

## 3. The schema

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
questions:   # required — the bank
```

`questions` missing gives `questions is required`. An unrecognised key gives
`unknown field 'title'`, and if it looks like a near-miss for a real one you
get a suggestion: `unknown field 'tolerence' — did you mean 'tolerance'?`

There is no field for an author, a subject, a topic, a section heading, an
instruction sheet, a total-marks figure, or an answer key at the end. There is
no field for how many questions each student answers either — that is set in
the admin panel, not in the file. Adding any of these fails the upload. **Do
not invent fields.**

### The `test:` block — all optional

You enter the test details in the admin panel before uploading, so the test
already has a name, a duration, and a time window. Anything you put here
*overwrites* what is there. Anything you leave out is left alone. The whole
block may be omitted.

| Field | Type | Rules and what a mistake says |
| --- | --- | --- |
| `name` | text | Cannot be blank. `name cannot be blank` |
| `duration_minutes` | whole number | At least 1. `duration_minutes must be a whole number of minutes, at least 1` |
| `opens_at` | text | `"YYYY-MM-DD HH:MM"`, Indian Standard Time, quoted. `'next tuesday' is not a date and time — use "YYYY-MM-DD HH:MM" (IST)` |
| `closes_at` | text | Same format, and must be later than `opens_at`. `closes_at must be after opens_at` |
| `groups` | list of text | Class names that **already exist** in the admin panel. `group 'does-not-exist' does not exist — create it before uploading` |

An AI writing a bank usually should not include `groups` at all — it cannot
know which classes exist on your installation, and a wrong guess fails the
whole upload. Assign classes in the admin panel instead.

`duration_minutes` is the whole sitting, not one question. Set it against the
number of questions each student answers, with room to spare: a student who
runs out of time stops wherever they are.

### A question

`questions:` is a list, and it must not be empty. Every question is a mapping
with these fields and no others:

| Field | Required | Type | Notes |
| --- | --- | --- | --- |
| `type` | **yes** | text | Exactly `mcq_single` or `numeric`. `subjective` is not allowed here |
| `text` | **yes** | text | The question itself. May contain LaTeX — [section 6](#6-mathematics-and-latex) |
| `difficulty` | **yes** | text | Exactly `easy`, `easy-med`, `med`, `med-hard` or `hard` |
| `marks_correct` | **yes** | number | Awarded for a right answer |
| `marks_incorrect` | **yes** | number | For a wrong answer. Normally negative or `0` |
| `marks_unattempted` | **yes** | number | For leaving it blank. Normally `0` |
| `image` | no | text | A path inside the zip — [section 5](#5-images) |
| `time_limit_seconds` | no | whole number | Per-question limit. `null`, `0` or `-1` all mean untimed |
| `options` | mcq only | list | At least two. **Forbidden on numeric** |
| `answer` | **yes** | text or number | An option key for mcq, a number for numeric |
| `tolerance` | numeric only | number | **Forbidden on mcq** |

**There are no defaults.** A field that is not optional above must be written
out on every single question, even when its value is the same every time. In
particular all three marks fields are required on every question:

```
marks_correct is required on every question
marks_incorrect is required on every question
marks_unattempted is required on every question
```

Marks may be negative or fractional, must be between -999.99 and 999.99, and
may not have more than two decimal places — `4.005` gives
`marks_correct has more than two decimal places`.

**Use the same marks on every question of a given type**, across all five
rungs. Students answer different questions, and a bank that pays more for a
hard question makes two scores incomparable in a way nothing corrects for.

`time_limit_seconds: 45` gives that question 45 seconds. Omitting it, or
writing `null`, `0` or `-1`, leaves it untimed. Any other value at or below
zero is treated as a typo: `-30 is not a valid time limit — use null, 0 or -1
for untimed`. A question whose time runs out is committed as it stands, scored
as unattempted, and moves the student **down** a rung, exactly as a wrong
answer does.

### `difficulty` — the field this format exists for

```yaml
difficulty: med-hard
```

Write it exactly as one of `easy`, `easy-med`, `med`, `med-hard`, `hard`. It is
required on every question:

```
difficulty is required on every question of a variable-difficulty test — one of easy, easy-med, med, med-hard, hard
```

Anything else is rejected by name:

```
'tricky' is not a difficulty — use one of easy, easy-med, med, med-hard, hard
```

Do not mention the difficulty in the question text, and do not write it into
the marks. It is invisible to students by design.

### `mcq_single` — one correct option

```yaml
options:
  - key: a
    text: '$10\ \mathrm{m\,s^{-1}}$'
  - key: b
    text: '$14\ \mathrm{m\,s^{-1}}$'
answer: 'b'
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

Vary which key is correct. A bank in which `a` is right far more often than the
rest is guessable, and a student who notices it climbs the ladder on nothing.

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

### No `subjective` questions

A variable-difficulty test cannot hold them. One in the file gives:

```
a variable-difficulty test cannot hold subjective questions — the next question is chosen from whether this one was answered correctly, which nothing can decide for a written answer
```

The next question has to be picked the instant the current one is answered, and
nothing can mark a written answer in that moment. Every question in a bank must
be `mcq_single` or `numeric`.

### How rejection looks

Nothing is imported unless everything is valid. You get every problem at once,
each with the line it is on, like:

```
the file was rejected with 2 problems — nothing was imported
  line 3   questions[0].difficulty: difficulty is required on every question of a variable-difficulty test — one of easy, easy-med, med, med-hard, hard
  line 8   questions[0].answer: a numeric answer must be a number, unquoted
```

`questions[0]` is the first question in the file, counting from zero.

One error is not about your file at all: **this test already has attempts on
it, so its questions are frozen — reset or delete those attempts first.** Once
a student has started a test, its bank cannot be changed.

---

## 4. A worked example

A complete, valid bank — shortened to two questions per rung so it fits here; a
real one is several times longer. Copy it, replace the questions, and it will
upload as it stands. Note the quoting: **single quotes around every `text:`**,
so the LaTeX inside is written exactly as it would be anywhere else, with no
doubled backslashes. Copy that habit along with the structure — see
[section 6](#6-mathematics-and-latex).

```yaml
test:
  name: 'Adaptive Mock 01 — Kinematics & Calculus'
  duration_minutes: 30

questions:
  # --- easy ----------------------------------------------------------------
  - type: mcq_single
    text: 'A car travels $60\ \mathrm{km}$ in $2\ \mathrm{h}$. Its average speed is:'
    difficulty: easy
    time_limit_seconds: null
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: '$15\ \mathrm{km\,h^{-1}}$'
      - key: b
        text: '$30\ \mathrm{km\,h^{-1}}$'
      - key: c
        text: '$60\ \mathrm{km\,h^{-1}}$'
      - key: d
        text: '$120\ \mathrm{km\,h^{-1}}$'
    answer: 'b'

  - type: numeric
    text: 'A body at rest accelerates at $2\ \mathrm{m\,s^{-2}}$. Its speed after $3\ \mathrm{s}$, in $\mathrm{m\,s^{-1}}$.'
    difficulty: easy
    time_limit_seconds: null
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 6
    tolerance: 0

  # --- easy-med ------------------------------------------------------------
  - type: mcq_single
    text: 'If $x = 3t^2 + 2t$, then $v$ at $t = 2\ \mathrm{s}$ is:'
    difficulty: easy-med
    time_limit_seconds: 90
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: '$10\ \mathrm{m\,s^{-1}}$'
      - key: b
        text: '$14\ \mathrm{m\,s^{-1}}$'
      - key: c
        text: '$8\ \mathrm{m\,s^{-1}}$'
      - key: d
        text: '$12\ \mathrm{m\,s^{-1}}$'
    answer: 'b'

  - type: numeric
    text: 'A stone is dropped from rest. Taking $g = 10\ \mathrm{m\,s^{-2}}$, how far does it fall in $2\ \mathrm{s}$, in metres?'
    difficulty: easy-med
    time_limit_seconds: 90
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 20
    tolerance: 0

  # --- med -----------------------------------------------------------------
  - type: mcq_single
    text: 'For $\int_{0}^{\pi/2} \sin^2 x\, dx$, the value is:'
    difficulty: med
    time_limit_seconds: null
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: '$\dfrac{\pi}{4}$'
      - key: b
        text: '$\dfrac{\pi}{2}$'
      - key: c
        text: '$1$'
      - key: d
        text: '$\dfrac{\pi}{8}$'
    answer: 'a'

  - type: numeric
    text: 'A projectile is thrown at $20\ \mathrm{m\,s^{-1}}$ at $30^\circ$ to the horizontal. Its vertical component of velocity, in $\mathrm{m\,s^{-1}}$.'
    difficulty: med
    time_limit_seconds: 120
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 10
    tolerance: 0.1

  # --- med-hard ------------------------------------------------------------
  - type: mcq_single
    text: 'If $y = \ln(\sin x)$, then $\dfrac{dy}{dx}$ equals:'
    difficulty: med-hard
    time_limit_seconds: 150
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: '$\cot x$'
      - key: b
        text: '$\tan x$'
      - key: c
        text: '$\dfrac{1}{\sin x}$'
      - key: d
        text: '$-\cot x$'
    answer: 'a'

  - type: numeric
    text: 'Two bodies of $2\ \mathrm{kg}$ and $3\ \mathrm{kg}$ move head-on at $4\ \mathrm{m\,s^{-1}}$ and $1\ \mathrm{m\,s^{-1}}$ and stick together. Their common speed, in $\mathrm{m\,s^{-1}}$.'
    difficulty: med-hard
    time_limit_seconds: null
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 1
    tolerance: 0.01

  # --- hard ----------------------------------------------------------------
  - type: mcq_single
    text: 'The solution of $\dfrac{dy}{dx} + 2y = 0$ with $y(0) = 3$ is:'
    difficulty: hard
    time_limit_seconds: 180
    marks_correct: 4
    marks_incorrect: -1
    marks_unattempted: 0
    options:
      - key: a
        text: '$3e^{-2x}$'
      - key: b
        text: '$3e^{2x}$'
      - key: c
        text: '$\dfrac{3}{1 + 2x}$'
      - key: d
        text: '$3 - 2x$'
    answer: 'a'

  - type: numeric
    text: 'A uniform rod of mass $M$ and length $L$ rotates about one end. Its moment of inertia is $\dfrac{ML^2}{n}$. Give $n$.'
    difficulty: hard
    time_limit_seconds: 180
    marks_correct: 4
    marks_incorrect: 0
    marks_unattempted: 0
    answer: 3
    tolerance: 0
```

The same shape in JSON, if you prefer it — save as `test.json`:

```json
{
  "test": { "name": "Adaptive Mock 01", "duration_minutes": 30 },
  "questions": [
    {
      "type": "mcq_single",
      "text": "For $\\int_{0}^{\\pi/2} \\sin^2 x\\, dx$, the value is:",
      "difficulty": "med",
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
      "text": "Acceleration due to gravity, in $\\mathrm{m\\,s^{-2}}$, to 2 decimal places.",
      "difficulty": "easy",
      "marks_correct": 4,
      "marks_incorrect": 0,
      "marks_unattempted": 0,
      "answer": 9.81,
      "tolerance": 0.01
    }
  ]
}
```

**Prefer YAML.** JSON has no single-quoted string, so every backslash in it has
to be doubled — exactly the friction YAML's single quotes remove. JSON also has
no comments, and its errors cannot report a line number.

---

## 5. Images

An image path is **relative to the folder holding the test file inside the
zip**. Both of these layouts work, and neither is preferred:

```
bank.zip                        bank.zip
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

This is the one thing to be careful about when a bank is generated for you. An
assistant has no way to produce a real circuit diagram, ray diagram, or graph,
and **an `image:` line pointing at a file that does not exist fails the whole
upload**. So the rule for generated banks is:

- For text-only questions, **omit `image` entirely.** Do not write
  `image: null`, a placeholder name, or a description in the field.
- Where a question genuinely needs a figure, the assistant should write the
  question without an `image` line and list it separately afterwards, in prose,
  saying what the figure should show and which question it belongs to.

You then draw or scan those figures, save them into `images/`, and add the
`image:` line yourself. The instruction block in
[section 7](#7-the-instruction-block-to-paste) tells the assistant to work this
way.

---

## 6. Mathematics and LaTeX

Read this section before writing a single question. Both of the mistakes it
guards against are silent: the upload succeeds, and the damage is only visible
to the student sitting the test.

### Every piece of mathematics must be LaTeX

Not "may be" — must. A bank written with plain-text mathematics is a defective
bank. It reaches students looking like a hastily typed email, and nothing in
the upload will warn you, because as far as the importer is concerned `x^2` is
a perfectly ordinary sentence.

| Never write this | Write this |
| --- | --- |
| `x^2` | `$x^2$` |
| `mu0`, `u0` | `$\mu_0$` |
| `root 3`, `sqrt(3)` | `$\sqrt{3}$` |
| `30 degrees` | `$30^\circ$` |
| `degree C`, `deg C` | `$^\circ\mathrm{C}$` |
| `10^-19 C` | `$10^{-19}\ \mathrm{C}$` |
| `m/s^2` | `$\mathrm{m\,s^{-2}}$` |
| `pi/4` | `$\dfrac{\pi}{4}$` |
| `delta x` | `$\Delta x$` |
| `integral from 0 to pi/2 of sin^2 x dx` | `$\int_{0}^{\pi/2}\sin^2 x\,dx$` |
| `H2SO4` | `$\mathrm{H_2SO_4}$` |
| `vector F` | `$\vec{F}$` |
| `2 x 10^8` | `$2\times 10^{8}$` |

This applies inside option text as much as inside question text. An option
reading `10 m/s` next to three others reading `$10\ \mathrm{m\,s^{-1}}$` is
just as wrong.

### Use single quotes, and stop thinking about escaping

Wrap every `text:` value in **single** quotes. Inside single quotes YAML treats
a backslash as an ordinary character, so you write LaTeX exactly as you would
write it anywhere else — nothing to double, nothing to escape:

```yaml
text: 'For $\int_{0}^{\pi/2} \sin^2 x\, dx$, the value is:'
text: '$\dfrac{\pi}{4}$'
text: 'A particle moves such that $x = 3t^2 + 2t$. Find $v$ at $t = 2\ \mathrm{s}$.'
```

The one thing single quotes care about is an apostrophe in your prose, which
must be doubled:

```yaml
text: 'State Newton''s second law.'      # two apostrophes, not one
```

If a question is long or full of apostrophes, a block scalar needs no escaping
of any kind:

```yaml
text: |-
  A block of mass $m$ slides down a frictionless incline of angle $\theta$.
  Using Newton's second law, find its acceleration.
```

Double quotes also work, but inside them every backslash must be written twice
(`"\\int"` to mean `\int`). There is no reason to take that on. Use single
quotes.

> **Do not let escaping put you off using LaTeX.** Getting the quoting wrong is
> the *safe* failure: YAML rejects the file, you are told the line, and you fix
> it in thirty seconds. Writing `x^2` instead of `$x^2$` is the *dangerous*
> one: it uploads cleanly and nobody finds out until a student is looking at it.
> Faced with any doubt, write the LaTeX.

### Delimiters

| Form | Use |
| --- | --- |
| `$ ... $` | Inline, within a sentence. **Must stay on one line** |
| `$$ ... $$` | Displayed on its own line, centred. May span lines |

### A cookbook of forms that render

Start here. These are known to typeset correctly, so reaching for one of them
is always cheaper than working something out. The table cannot cover every
formula a real bank needs, though — when you do compose something new, check it
against [the syntax rules](#checking-something-the-cookbook-does-not-cover)
below before you use it.

| | |
| --- | --- |
| **Greek** | `\alpha` `\theta` `\lambda` `\omega` `\mu_0` `\varepsilon_0` `\Delta x` `\pi` |
| **Powers, indices** | `x^2` `v^{-1}` `10^{-19}` `x_1` `\mathrm{H_2SO_4}` |
| **Fractions** | `\dfrac{\pi}{4}` `\frac{dv}{dt}` `\dfrac{a}{b}` |
| **Roots** | `\sqrt{3}` `\sqrt{2gh}` |
| **Trig, logs** | `\sin^2 x` `\sin^{-1}x` `\log_{10} x` `\tan\theta` |
| **Calculus** | `\int_{0}^{\pi/2}\sin^2 x\,dx` `\frac{dv}{dt}` |
| **Vectors** | `\vec{F}` `\hat{n}` |
| **Multiplication** | `2\times 10^{8}` `5\cdot 3` |
| **Brackets that grow** | `\left(\dfrac{a}{b}\right)^{2}` |
| **Degrees** | `30^\circ` and, for temperature, `^\circ\mathrm{C}` |
| **Units** | `\mathrm{m\,s^{-1}}` `\mathrm{m\,s^{-2}}` `\mathrm{J\,kg^{-1}\,K^{-1}}` `\mathrm{kg\,m^{-3}}` `\mathrm{N\,m}` |
| **A number with a unit** | `1.6\times10^{-19}\ \mathrm{C}` `4.18\ \mathrm{J\,g^{-1}\,K^{-1}}` |

Units go inside `\mathrm{...}` so they stand upright instead of leaning over
like variables, and `\,` is the thin space between them. A single `\ ` separates
the number from its unit.

### The four ways it breaks

**1. A superscript or subscript straight after a spacing command.** This is the
one that looks right and is not:

```
BROKEN   $\mathrm{cal}\,g^{-1}\,^{\circ}C^{-1}$
```

`^` needs something to sit on, and `\,` is a space, not something. The same
goes for `\;` and `\quad`. Three ways to fix it, all fine:

```
$\mathrm{cal\,g^{-1}\,{}^{\circ}C^{-1}}$      an empty {} gives it a base
$\mathrm{cal\,g^{-1}}\ ^{\circ}\mathrm{C^{-1}}$   a plain \ space instead of \,
$4.18\ \mathrm{J\,g^{-1}\,K^{-1}}$                sidestep it: use J and K
```

**2. Unmatched braces.** `\dfrac{\pi}{4` — count every `{` and `}`.

**3. A command that does not exist here.** `\vecc{F}`, `\ce{H2O}`. Chemistry
notation is **not** available: write `$\mathrm{H_2SO_4}$`, never `$\ce{H2SO4}$`.
If a command is not in the cookbook above, do not assume it works.

**4. A stray `$`.** An odd number of dollar signs in one line leaves maths
running into your prose, or prose being typeset as maths.

### Checking something the cookbook does not cover

When you write an expression that is not in the table, put it through these
three rules before using it:

1. **Braces balance.** Every `{` has a matching `}`, counting left to right.
2. **Every `^` and `_` has something to attach to** immediately before it — a
   letter, a digit, or a closing `}`. A spacing command (`\,` `\;` `\!`
   `\quad` `\qquad`) is not something; that is the failure described above.
3. **Every command is one you know exists here**, not one that resembles a
   command you know. `\vec` yes, `\vecc` no. `\mathrm` yes, `\ce` no.

Rules 1 and 2 are mechanical, and the
[script in section 9](#a-mechanical-check-before-you-upload) applies them to a
whole file in a second. Rule 3 needs judgment: if you are not certain a command
exists, rewrite the expression using ones from the cookbook.

### Broken mathematics does not stop the upload

The file imports, the test opens, and the question reaches students with the
expression not typeset as you intended. Nothing in the upload will tell you.
The mechanical check below, and reading the questions on screen afterwards, are
what catch it.

---

## 7. The instruction block to paste

Paste **this whole document** into the assistant first, then this, with the
bracketed placeholders filled in:

```text
Using the variable-difficulty bank format document above, write a question bank
for me.

Subject:                [e.g. Physics]
Questions per student:  [e.g. 10 — what the test is set to]
Size of the bank:       [e.g. 50, ten at each difficulty]
Topics:                 [e.g. kinematics, Newton's laws, work and energy]
Standard:               [e.g. JEE Main, so that 'med' is a typical Main question]

THIS IS A BANK, NOT A FIXED PAPER
1. Every question must carry a difficulty field whose value is exactly one of
   easy, easy-med, med, med-hard, hard. There are no other values and none may
   be left out.
2. Write the same number of questions at each of the five difficulties, and
   make them genuinely different in difficulty. An easy question should be one
   step of recall or one substitution; a hard one should need several steps or
   a non-obvious idea; the ones between should sit between them. The labels
   drive which question each student is served next, so a bank whose
   difficulties are only labels defeats the whole test.
3. Do not write any subjective questions. Every question must be mcq_single or
   numeric.
4. Do not mention the difficulty in the question text, and do not refer to any
   other question — students are served them one at a time, in an order nobody
   can predict, and see only a few of them.
5. Cover the topics at every difficulty rather than making one topic the easy
   one and another the hard one.

SCHEMA
6. Follow the schema in that document exactly. Every question must carry type,
   text, difficulty, marks_correct, marks_incorrect, marks_unattempted, and —
   for mcq_single — at least two options and an answer matching one of their
   keys, or — for numeric — an unquoted numeric answer and a tolerance.
7. Use only the fields listed in that document. Do not add fields of your own
   such as topic, subject, explanation or solution, and do not add section
   headings. Any extra field causes the upload to be rejected.
8. Use marks_correct: 4, marks_incorrect: -1, marks_unattempted: 0 for
   multiple-choice questions, and marks_correct: 4, marks_incorrect: 0,
   marks_unattempted: 0 for numeric questions, unless I have said otherwise.
   Use the same marks at every difficulty.
9. Do not include a groups field.

MATHEMATICS — the part most likely to go wrong
10. Every mathematical symbol, variable, power, unit, Greek letter and formula
    must be LaTeX between dollar signs. Plain-text mathematics is not an
    acceptable fallback: never write x^2, mu0, sqrt(3), pi/4, m/s^2, 10^-19 or
    "degree C" as bare text. This applies to option text as much as to question
    text.
11. Put single quotes around every text: value, so you can write LaTeX exactly
    as it is without doubling any backslashes. If the prose contains an
    apostrophe, double it: 'Newton''s second law'. Do not use double quotes for
    anything containing LaTeX.
12. Do not avoid LaTeX because you are unsure of the quoting. A quoting mistake
    is caught and reported the moment I upload the file; plain-text mathematics
    is accepted silently and reaches students looking wrong. If in doubt, write
    the LaTeX.
13. Use the forms in that document's cookbook wherever they cover what you need.
    Units go inside \mathrm{...} with \, between them, like $\mathrm{m\,s^{-2}}$
    and $4.18\ \mathrm{J\,g^{-1}\,K^{-1}}$. Where the cookbook does not cover
    something, compose it and then check it against the three syntax rules in
    that document: braces balance, every ^ and _ has a base immediately before
    it, and every command is one you are certain exists here. If you are not
    certain a command exists, rewrite the expression with ones you are.
14. Never place ^ or _ immediately after a spacing command. $\,^{\circ}C$ is
    invalid LaTeX; write $\,{}^{\circ}\mathrm{C}$ or restructure the expression.
    Chemistry notation such as \ce{...} is not available — write
    $\mathrm{H_2SO_4}$.

BEFORE YOU ANSWER — check your own output mechanically, not by re-reading
15. Go through the finished YAML looking for these five things specifically,
    scanning for the pattern rather than reading for sense. A careful re-read
    misses them; a character-by-character pass does not.
      a. every question has a difficulty, and the five values are used evenly
      b. every { has a matching }, per string
      c. no ^ or _ immediately follows \, \; \! \quad or \qquad
      d. an even number of $ on every line
      e. no mathematics left as plain text
    Then confirm, question by question, that each mcq answer value is one of
    that question's own option keys and each numeric question has a tolerance.

FIGURES
16. Do not include an image field on any question. If a question genuinely
    needs a diagram, write it without an image field and then, after the code
    block, list those questions and describe what each figure should show.
    Never invent a filename.

17. Vary the position of the correct answer across the options; do not let it
    sit at the same key repeatedly.

Output only the YAML, in a single fenced code block, with no commentary before
it. Any notes about figures go after the code block.
```

If you are asking an assistant to extend a bank it wrote earlier — the usual
way to reach 250 questions — add: *"Keep every rule above, especially the
mathematics and difficulty rules. Add [N] more questions at [difficulty],
covering [topics], and do not repeat any question already written."*

---

## 8. Packaging

Save the assistant's YAML as `test.yaml`. Then, from a terminal in the folder
holding it:

**No images:**

```bash
zip bank.zip test.yaml
```

**With images**, keeping the paths the file refers to:

```bash
mkdir -p images          # put your figures in here first
zip -r bank.zip test.yaml images
```

Check what you built before uploading it:

```bash
unzip -l bank.zip
```

You want to see exactly this shape — `test.yaml` at the top, not buried:

```
    Length      Date    Time    Name
---------  ---------- -----   ----
     4821  2026-09-04 11:02   test.yaml
      765  2026-09-04 11:02   images/q1.png
```

On **Windows**, select `test.yaml` and the `images` folder together, right-click,
and choose *Send to → Compressed (zipped) folder*. Select the files, not the
folder that contains them — though if you do zip the containing folder, that
works too, as long as there is only one test file inside it.

On **macOS**, select both, right-click, *Compress*. The extra `__MACOSX` entries
the Finder adds are ignored.

Then select **Create a test** in the admin panel, choose **Variable
difficulty**, set how many questions each student answers, continue to
**Questions**, and upload `bank.zip`.

---

## 9. Checklist before you upload

Read down the file once. Almost every rejection is one of these.

**The difficulty ladder**

- [ ] Every question has a `difficulty`, spelled exactly `easy`, `easy-med`,
      `med`, `med-hard` or `hard`
- [ ] All five rungs are used, and roughly evenly
- [ ] The labels are honest — an `easy` question really is easier than a
      `med-hard` one
- [ ] The bank is several times the number of questions each student will be
      asked
- [ ] No question refers to another question, and none names its own difficulty

**Every question**

- [ ] `type` is exactly `mcq_single` or `numeric` — **no `subjective`**
- [ ] `text` is present and not blank
- [ ] All three of `marks_correct`, `marks_incorrect`, `marks_unattempted` are
      written out — on **every** question, with no exceptions, and the same at
      every difficulty
- [ ] No invented fields: no `topic`, `subject`, `explanation`, `solution`,
      `section`, `id` or `number`

**Multiple-choice questions**

- [ ] At least two options, each with both `key` and `text`
- [ ] No two options in one question share a key
- [ ] `answer` is **quoted** and is character-for-character one of that
      question's own option keys — check each one against its own list, not
      the previous question's
- [ ] The correct key varies across the bank
- [ ] No `tolerance`

**Numeric questions**

- [ ] `answer` is a bare number with no quotes: `9.81`, not `"9.81"`
- [ ] `tolerance` is present on every one of them, `0` for an exact match
- [ ] No `options`

**Mathematics** — nothing here will fail the upload, so this pass is the only
thing standing between a bad expression and a student

- [ ] **No plain-text mathematics anywhere**, in question text or option text.
      Scan for the tell-tales: a bare `^`, `sqrt`, `pi`, `mu`, `deg`, `/s`,
      `^-1`, a number followed by a unit with no `$` around it
- [ ] Every `text:` with mathematics in it is in **single** quotes, and any
      apostrophe inside is doubled
- [ ] Dollar signs are paired — an even number on every line
- [ ] Braces balance — every `{` has its `}`
- [ ] No `^` or `_` directly after `\,`, `\;` or `\quad`
- [ ] No `\ce{...}`, and no command that is not in the
      [cookbook](#a-cookbook-of-forms-that-render)
- [ ] Units are inside `\mathrm{...}`

**Images**

- [ ] Every `image` path names a file that is actually in the zip
- [ ] No `image` line was invented for a figure that does not exist
- [ ] Paths are relative to `test.yaml`, spelled and capitalised identically

**The file and the zip**

- [ ] Named `test.yaml`, `test.yml` or `test.json`, and only one of them
- [ ] Only `test` and `questions` at the top level
- [ ] `unzip -l` shows the test file at the top of the archive

If the upload is rejected, the list it gives you names the line and the
question number for each problem, and fixing them all and re-uploading is safe
— nothing was written the first time.

### A mechanical check before you upload

The three items above about braces, spacing commands and dollar signs are the
ones a careful re-read reliably misses — they are pattern faults, and reading
for sense skips straight over them. Save this as `check.py` next to your
`test.yaml` and run it instead. It counts the difficulties for you as well, so
you can see the spread before the panel does:

```python
import re, sys
from collections import Counter

spread = Counter()
for n, line in enumerate(open(sys.argv[1], encoding="utf-8"), 1):
    found = re.match(r"\s*-?\s*difficulty:\s*'?\"?([\w-]+)", line)
    if found:
        spread[found.group(1)] += 1
    if line.count("{") != line.count("}"):
        print(f"line {n}: braces do not balance -- {line.strip()}")
    if re.search(r"\\(?:,|;|!|quad|qquad)\s*[\^_]", line):
        print(f"line {n}: ^ or _ straight after a spacing command -- {line.strip()}")
    if line.count("$") % 2:
        print(f"line {n}: odd number of $ -- {line.strip()}")

print("difficulties:", dict(spread) or "none found -- this bank will be rejected")
```

```bash
python3 check.py test.yaml
```

Silence, apart from the difficulty count, means those three faults are absent.
Anything it prints is worth looking at, though not every hit is a fault: it
reads one line at a time, so a question written as a block scalar over several
lines, or `$$` display maths split across lines, can show up as unbalanced.
Judge each one.

It does not know which commands exist, it does not know whether your formula
says what you meant, and it certainly cannot tell whether a question labelled
`hard` is hard. Those still need the cookbook and your eyes.

If you are having an AI write the bank, tell it to run these same checks over
its output before answering — the instruction block in
[section 7](#7-the-instruction-block-to-paste) already does.

---

## 10. The one check nothing else does: read the bank on screen

**Do this every time.** It takes a few minutes, and it is the last thing
between a broken formula and a student, because broken mathematics uploads
perfectly happily. The checks before this one are mechanical and catch pattern
faults; this one catches everything else.

After the upload succeeds, stay on the same page. Below the upload control is
the **Questions** panel, showing every question in the bank with its difficulty
beside it, and above them the spread across the five rungs. Read down it and
look for:

1. **A lopsided spread.** If one rung has three questions and another has
   forty, students will spend the test bouncing off the thin one.
2. **Difficulties that do not match the questions.** Read a few from each rung
   in turn. This is the judgment nothing else in the system can make, and it is
   the difference between an adaptive test and a random one.
3. **Any expression that did not typeset.** A formula that failed shows up as
   something other than laid-out mathematics — backslashes, braces and command
   names sitting in the middle of the sentence.
4. **Stray dollar signs** on screen. A `$` you can actually see means its pair
   is missing and the maths around it was never typeset.
5. **Mathematics that is still plain text.** `x^2` and `m/s^2` will sit there
   looking like ordinary typing.
6. **Formulae that render but say the wrong thing** — a missing minus sign, a
   subscript that swallowed the next character.

Anything wrong: fix the file and upload again. A re-upload replaces the bank
completely, so there is no cleaning up to do and no cost to doing it twice.

Do this **before** the test opens, and before any student has started — once an
attempt exists the bank is frozen.
