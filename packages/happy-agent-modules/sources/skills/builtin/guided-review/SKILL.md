---
name: guided-review
description: Use whenever someone asks to review, tour, walk through, or explain a change (a PR, a branch, commits, the work in this worktree), in any language and however it is phrased ("review this", "ревью", "что мы тут поменяли"), or asks to have a subsystem explained with a picture rather than prose. Writes a Happy UI block the transcript draws in place; a review puts its verdict and findings inside it.
---

# Guided review in the chat

A big diff in file order makes a reader jump between files for half an hour
to learn what the change even does. A tour does that work for them: one
sentence of context, the steps the change takes, before and after as "show
me", then the files gathered into chapters by idea, most important first.

The tour is a ` ```happy-ui ` fence in your reply. Happy draws it with its own
components, in the conversation, once the fence closes. Nothing else is
needed: no panel, no file, no HTML.

## A review is a tour with its findings inside

When someone asks to review a change, write the tour and put the review in
it, so each problem sits beside the code it is about:

- A `verdict` right after `context`: one or two sentences saying whether
  the change can go in and what must happen first. Happy counts the findings
  by severity above it, so never count them yourself.
- Each problem is a `finding` inside the chapter whose code it concerns,
  after that chapter's diffs, with `path` and `lines` on the exact lines
  that are wrong; Happy draws those real lines in the finding. A problem with
  the change as a whole, not one place, is a `finding` after the chapters,
  without lines.
- `severity` is `high` (must be fixed before it goes in), `medium` (should
  be fixed), or `low` (worth knowing). `title` says what is wrong, `text`
  why it matters, and `fix` what to do, each in a sentence or two.
- What is good goes in the verdict, briefly. A chapter with nothing wrong
  simply has no finding.

```
{ "type": "verdict", "text": "Safe to merge once the retry loop stops on errors." },
...
{
  "type": "finding",
  "severity": "high",
  "title": "A failed read is retried on every render",
  "text": "The error is not remembered, so each render starts the same read again and the requests never stop.",
  "path": "packages/state/src/store.ts",
  "lines": { "startLine": 200, "endLine": 226 },
  "fix": "Keep the error with the revision it failed at and read again only when that moves."
}
```

After the fence, write at most a sentence or two, only for what you could not
check (tests you did not run, a screen you did not open). Nothing the block
already says is repeated outside it. A narrow question about one bug or one
file is answered directly, without a tour.

## Rules that make a tour trustworthy

- **Read the real diff first.** For this branch's work list every changed
  file with `git diff --name-only <merge base>` and `git status --porcelain
-uall` (new files count) and read them with `git diff <merge base>` and the
  new files themselves; for other commits use `git diff <base> <head>`. Read
  every file before writing a word. Every statement in the tour must be
  visible in that diff.
- **Cover every file.** Each changed file is named in exactly one chapter.
  Happy checks this: under a `base` tour it lists every changed file the tour
  left out, so a forgotten file is visible to the reader.
- **Chapters are ideas, not folders.** "Where a session opens" is a chapter;
  "packages/state" is not. One chapter should be readable without jumping.
- **Every chapter has a role, and the order follows it.** `core` is an idea
  of the change; `supporting` is what an idea rests on (a store, plumbing,
  styles, a schema or migration). The first chapter is `core`; core and
  supporting chapters may alternate, so each idea sits beside what it rests
  on. `tests` and `generated` (lockfiles, snapshots, generated code, frozen
  fixtures) come last and are drawn quieter. Happy refuses a tour that breaks
  this order. Put database schema and migration chapters after the core
  ideas, just before tests, unless the schema is itself the idea.
- **Core and supporting chapters say what their files do,** in a `text` of
  one or two plain sentences. Tests and generated chapters may be a bare list.
- **Before/after describes behaviour,** not file names. What a person saw or
  could do before, and what they see or can do now.
- **Point, do not reprint.** A diff node names a path and a line range in the
  file as it now stands; Happy shows the real lines from the change. Never
  write diff text yourself. Keep a range short enough to read in place.
- **Line numbers are current-file numbers.** Check them against the file as it
  is now, not against the old version or the hunk header.
- **Plain text only.** Every string is plain words. No Markdown, no code
  spans, no links; a file is a file entry, a picture is a mermaid node.

## The format

```happy-ui
{
  "version": 1,
  "kind": "tour",
  "base": "<full merge-base hash>",
  "children": [
    { "type": "context", "text": "One sentence saying what this change does." },
    { "type": "steps", "items": ["First thing it does.", "Second thing."] },
    { "type": "beforeAfter", "before": "What happened before.", "after": "What happens now." },
    {
      "type": "chapter",
      "role": "core",
      "title": "The idea in ordinary words",
      "text": "A sentence or two on how the files below carry it.",
      "files": [
        { "path": "packages/state/src/store.ts", "lines": { "startLine": 40, "endLine": 58 }, "note": "what this file contributes" }
      ],
      "children": [
        { "type": "diff", "path": "packages/state/src/store.ts", "lines": { "startLine": 40, "endLine": 58 } }
      ]
    },
    {
      "type": "chapter",
      "role": "tests",
      "title": "Tests",
      "files": [{ "path": "packages/state/src/store.test.ts" }]
    }
  ]
}
```

Nodes, and nothing else:

| Node          | Fields                                                 | Use                                                               |
| ------------- | ------------------------------------------------------ | ----------------------------------------------------------------- |
| `context`     | `text`                                                 | One sentence. First node of every tour.                           |
| `steps`       | `items`                                                | The steps the change takes, in order.                             |
| `beforeAfter` | `before`, `after`, `beforeImage?`, `afterImage?`       | Behaviour before and after. At least one. Images are screenshots. |
| `chapter`     | `role`, `title`, `text`, `files`, `children?`          | One idea and its files. `children` holds diff, mermaid, finding.  |
| `diff`        | `path`, `lines: { startLine, endLine }`                | Real lines from the change at that place.                         |
| `mermaid`     | `source`                                               | A diagram, when the shape of a thing matters.                     |
| `verdict`     | `text`                                                 | A review's conclusion, after `context`. At most one.              |
| `finding`     | `severity`, `title`, `text`, `path?`, `lines?`, `fix?` | One problem a review found, where it is.                          |

## Show a visible change

When the change is something a person can see — a component, a page, a
dialog, a layout — put a screenshot of each side on the `beforeAfter` node.
Happy draws each in a frame above its words, and a click opens it full size.

1. **Capture both sides of the same place,** at the same window size, framed
   on the part that changed. Take the after side from the change as it now
   runs. Take the before side from the code before it: a Git worktree at the
   base commit, or the app before your edit. Never draw, mock, or generate a
   picture of a UI; a screenshot of the real one is the whole point.
2. **Publish each as an image artifact** (one PNG, JPEG, or WebP), titled for
   what it shows, for example "Profile settings before cropping".
3. **Name the artifact and its version,** never a URL or a file path:

```
{
  "type": "beforeAfter",
  "before": "Upload photo sent the picked file as it was.",
  "after": "A crop dialog opens first, then the photo is saved.",
  "beforeImage": { "artifact": "<artifact id>", "version": 1 },
  "afterImage": { "artifact": "<artifact id>", "version": 1 }
}
```

The words stay and still describe behaviour: they caption the picture and
carry the change wherever a picture cannot be shown. A side may have a
picture while the other has none — a new dialog has no before. A change
nobody can see, such as a store or a migration, gets no pictures.

## Which change the tour shows

Every tour names the change it describes, beside `kind`: either `base` or
`range`, never both.

**This branch's work — the usual case — names `base`.** Happy reads the
checkout's live change: the working tree against the merge base with the
default branch. That is the whole branch, committed and uncommitted alike, so
committing more does not break the tour, and Happy checks coverage against
every file the branch changes.

```
"version": 1,
"kind": "tour",
"base": "<full hash>",
"children": [ ... ]
```

Take the base from Git, never from memory: `git merge-base origin/main HEAD`
(use the repository's default branch; without an `origin` copy of it, the
local one). Read the change with `git diff <base>` plus every untracked file
from `git status --porcelain -uall`, and give line numbers in the files as
they are now. Once the branch is merged, rebased, or synced, the checkout
measures from another base; the tour then says it is no longer checked
instead of showing a different change.

**Any other change — past commits, another branch, a merged PR — names a
`range`** of two full commit hashes:

```
"version": 1,
"kind": "tour",
"range": { "base": "<full hash>", "head": "<full hash>" },
"children": [ ... ]
```

Take both from `git rev-parse`, read the diff with `git diff <base> <head>`,
and give line numbers in the file as it is at `head`. Happy lists the files
between the two commits and checks coverage against them too. A Happy Agent
too old to list them says coverage was not checked: covering every file is
then on you alone.

A `tour` must start with `context`, include `steps` and a `beforeAfter`, and
have at least one `chapter`; every chapter names at least one file and has a
`role`, the first is `core`, and `tests` and `generated` come last.

Use `"kind": "explanation"` for anything that is not a change: a subsystem, a
data flow, a decision. An explanation may use any nodes in any order, needs no
files, and its chapters need no role.

## Before you send it

- Is this a review? Then is there a `verdict` after `context`, and is every
  problem a `finding` in the chapter it concerns, pointing at its lines?

- Is it this branch's work? Then is there a `base` from `git merge-base`?
  Is it any other change? Then is there a `range` from `git rev-parse`?
- Does every changed file — `git diff --name-only <merge base>` plus
  untracked files, or `git diff --name-only <base> <head>` — appear in a
  chapter?
- Does each before/after describe behaviour you can point to in the diff?
- Is the change visible? Then does its before/after carry real screenshots,
  published as image artifacts and named by id and version?
- Can each chapter be read top to bottom without opening another chapter?
- Does every chapter have a role, does the first one say `core`, and are
  tests and generated files last?
- Does every core and supporting chapter have its `text`?
- Is every line range checked against the current file?

Then write one short sentence before the fence and send the fence. After it,
only what you could not check, if anything. Happy draws the rest.
