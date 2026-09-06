# Conative Education

Online JEE-style testing: timed, auto-scored, run from your own machine.

## Setup

You need **Docker Desktop** (Windows or Mac) or **Docker Engine** (Linux) on the
computer that will run it. Nothing else — no source code, no build tools, no
configuration file to fill in.

**1. Download it into a folder of its own.**

```bash
git clone https://github.com/atharvrawal/conative-education.git conative
cd conative
```

Cloning rather than saving one file is what makes [updating](#updating) a
single command later. If you do not have `git`, use the green **Code** button
above and **Download ZIP** instead — everything works the same, with one extra
step when you update.

**2. Start it.**

```bash
docker compose up -d
```

That is the whole installation. The first run downloads the images and takes a
few minutes; after that it starts in seconds, runs in the background, and comes
back on its own when the computer restarts. A `backups` folder appears next to
the compose file — [see below](#backups), it matters.

---

## Open it

On the computer running it:

**<http://localhost:8080>**

On any other device on the same Wi-Fi — student laptops, tablets — you need
that computer's address on the network:

| | |
| --- | --- |
| **Windows** | `ipconfig` — look for *IPv4 Address*, e.g. `192.168.1.42` |
| **Mac** | `ipconfig getifaddr en0` |
| **Linux** | `hostname -I` |

Students then open **`http://192.168.1.42:8080`** (with your number in place
of that one). Everyone has to be on the same network — this is not published
to the internet.

## Log in

| Username | Password |
| --- | --- |
| `admin` | `change-me-too` |

**Change the password immediately.** Until you do, a yellow banner sits across
the top of every admin page, and it is telling the truth: anyone who can open
the address above can sign in as you. Click **Change it now** in the banner, or
your username in the top right.

The new password is stored in the database. Restarting, updating, or shutting
down for the night will not undo it.

Everything after this happens in the admin panel — creating groups, approving
students, uploading test papers, watching a test run, downloading results.

There are two ways to get questions into a test, and they write the same thing.
Upload a zip, or type questions one at a time on the test's Questions tab — the
editor shows you the mathematics rendered as you type, so a broken formula is
caught there rather than by a student mid-test. Uploading a zip replaces the
whole paper, including anything typed in.

**Download paper** on the same tab gives you the paper as it stands now, as a
zip you can upload into another test, keep as a file, or hand to someone else.
It is built from the questions currently in the test, so it carries anything you
corrected in the editor — not whatever file was first uploaded, if there ever
was one.

To write a paper as a file, see
**[paper-generator-format.md](paper-generator-format.md)** — the upload format in
full, including a prompt you can hand to an AI assistant to draft one for you.

A test's name, duration and time window can all be changed afterwards, on the
Settings tab. The window controls who may *start* the test; a student already
sitting it keeps the deadline they were given.

---

## Stopping and starting

```bash
docker compose stop      # stop, keep everything
docker compose start     # start it again
docker compose restart   # stop and start, e.g. if something looks stuck
```

Use `stop`, not `down`. Both keep your data, but `stop` is the one you want
day to day.

Check on it:

```bash
docker compose ps        # all four should say "healthy"
docker compose logs -f   # live log, Ctrl-C to stop watching
```

## Where the data lives

Your live data is in two Docker volumes on the machine, not in the folder with
the compose file:

| Volume | Holds |
| --- | --- |
| `conative_pgdata` | everything — students, groups, tests, answers, scores |
| `conative_uploads` | the images from uploaded test papers |

They survive `stop`, `start`, `restart`, and updating to a new version. The one
command that deletes them is `docker compose down -v`. Do not run that.

The `backups` folder *is* in the folder with the compose file, deliberately —
it is the one thing you can see, copy and carry away, and the only thing that
survives `down -v`.

Copying the compose file to another computer does **not** copy your data. The
backups are what moves it.

---

## Backups

Backups run on their own. There is nothing to remember and nothing to type.

A fourth container writes one **every night at 02:30**, and one **every time the
stack starts**, so there is always at least one backup from the current run.
They land in a `backups` folder next to your compose file:

```
conative/
  docker-compose.yml
  backups/
    2026-09-03_0230/
      db.dump          students, groups, tests, answers, scores
      uploads.tar.gz   the images from your test papers
    2026-09-04_0230/
      ...
```

The two files in a dated folder belong together: the database refers to the
images by name, so one without the other is half a backup. A folder only gets
its dated name once both files are finished, so the newest one is always a
complete, matching pair — there is no such thing as a half-written backup here.

The **last 7** are kept. Older ones are deleted automatically, so the folder
does not grow without limit.

Want one right now — before an update, or at the end of a test day?

```bash
docker compose restart backup
```

To check that backups are actually happening, `docker compose ps` reports
`backup` as **healthy** only while there is a backup less than 26 hours old. If
it ever reads unhealthy, backups have stopped and `docker compose logs backup`
will say why.

On Linux the files belong to `root`. You can read and copy them as yourself;
deleting one by hand needs `sudo`, which is why the pruning above is automatic.

### Copy them off this computer

**A backup that only exists on this laptop is not a backup.** The disk that
fails takes it with it, and that is the exact day you need it.

The simplest thing that keeps working without anyone remembering: put the whole
`conative` folder — compose file, `backups` and all — **inside a synced cloud
folder**, the one the Google Drive, Dropbox or OneDrive desktop app already
watches. Every night's backup is then uploaded on its own. Nobody has to do
anything, which is the only kind of routine that survives a busy term.

If you would rather not use cloud storage: keep a USB drive for this and drag
the newest dated folder onto it after every test day. Use two drives and
alternate, so a corrupt one is never the only copy.

## Restore from backup

On a fresh machine, or on this one after something went badly wrong. Install
Docker and do [step 1 of Setup](#setup) again to get the compose file, then put
your dated backup folder inside a `backups` folder next to it, as it was.

**1. Start the database on its own** and wait for it to be ready:

```bash
docker compose up -d --wait db
```

**2. Restore the database.** Use your own dated folder in place of this one:

```bash
docker compose exec -T db pg_restore -U conative -d conative --clean --if-exists \
  < backups/2026-09-04_0230/db.dump
```

**3. Restore the images:**

```bash
docker run -i --rm -v conative_uploads:/data alpine \
  sh -c 'rm -rf /data/* && tar xzf - -C /data' \
  < backups/2026-09-04_0230/uploads.tar.gz
```

**4. Start everything:**

```bash
docker compose up -d --wait
```

Then open <http://localhost:8080> and sign in. Everything comes back: students,
groups, papers, submitted answers and scores, the figures in the questions, and
the admin password you had set — that is in the backup too, so the one printed
further up this page will *not* work.

The database is started alone in step 1 on purpose. Bringing the whole stack up
first would have the backup container take a snapshot of the still-empty
database before you have restored anything.

---

## Updating

> ⚠️ **Never run `docker compose down -v` as part of an update.** The `-v`
> deletes the database and uploads volumes permanently — every student, every
> paper, every result, gone with no way back. An update needs no `down` at all.

In the folder with the compose file:

```bash
git pull
docker compose up -d
```

That is the whole update. **It never touches your data.**

**Why there is no separate download step.** `git pull` brings you the new
compose file, and that file points at a new version of the software — say
`1.0.2` where it used to say `1.0.1`. Your machine does not have that version
yet, so `docker compose up -d` sees an image it is missing and fetches it
before starting the container, exactly as it did on your very first run.
Nothing else needs asking for.

**Why so little seems to happen.** Only the containers whose version actually
changed get recreated; anything already on the right version is left running
untouched. Your database and uploaded images live in volumes that are not part
of any container, so they are not involved either way. Some updates change only
the compose file and recreate nothing at all.

**Do it outside a live test.** Recreating a container costs a few seconds of
downtime. Students mid-test are not in danger — the app retries saving on its
own and nothing is lost — but there is no reason to make anyone watch a
spinner. Between sessions, or at the end of the day.

**Take a backup first anyway.** `docker compose restart backup`, then check
that a new dated folder appeared in `backups`.

**If you downloaded the ZIP instead of cloning**, there is no `git pull` to
run. Download the ZIP again, replace your old `docker-compose.yml` with the new
one, and run `docker compose up -d`. Leave the `backups` folder where it is.

**Do not edit `docker-compose.yml`.** `git pull` will refuse to overwrite your
changes and you will be stuck. Everything worth changing can be set in a `.env`
file next to it instead — see [Settings you can change](#settings-you-can-change).

**Downgrading is not supported.** Going back to an older compose file after a
newer one has run is not something to attempt: a newer version may have changed
the shape of the database, and the older software will not understand it. If an
update has gone wrong, do not roll back on your own — ask first.

## If something is wrong

```bash
docker compose ps        # is anything not "healthy"?
docker compose logs api  # the last few hundred lines usually say why
docker compose restart   # fixes most transient trouble
```

Two specific cases:

- **Students cannot reach it.** They are probably on a different Wi-Fi, or the
  computer's firewall is blocking port 8080. Check <http://localhost:8080>
  works on the machine itself first — that separates the two problems.
- **`backup` says unhealthy.** No backup has been written in the last 26
  hours. `docker compose logs backup` gives the reason — nearly always no room
  left on the disk. `docker compose restart backup` retries immediately.
- **You forgot the admin password.** Recoverable without losing anything. Make
  a file called `.env` next to the compose file containing one line —
  `ADMIN_PASSWORD=your-new-password` — then run `docker compose up -d`. The
  account is reset to that password and signed out everywhere. Everything else
  is untouched.

---

## Settings you can change

You do not need any of these to run it. If you do want to change something,
make a file called `.env` next to the compose file with only the lines you
want to change:

```bash
WEB_PORT=9000            # if something else is already using 8080
ADMIN_USERNAME=priya     # a different admin username
BACKUP_TIME=23:45        # when the nightly backup runs, 24-hour clock
BACKUP_KEEP=14           # how many nightly backups to keep
TZ=Asia/Kolkata          # the clock BACKUP_TIME is read against
```

Then `docker compose up -d`. Every value in the compose file can be set this
way.
