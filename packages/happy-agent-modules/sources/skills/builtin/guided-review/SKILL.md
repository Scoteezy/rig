---
name: guided-review
description: Use whenever someone asks to review, tour, walk through, or explain a change (a PR, a branch, commits, the work in this worktree), in any language and however it is phrased ("review this", "ревью", "что мы тут поменяли"), or asks to have a subsystem explained with a picture rather than prose. Writes a Happy UI block the transcript draws in place; a review puts its verdict and findings inside it.
---

# Guided review in the chat

A big diff in file order makes a reader jump between files for half an hour
to learn what the change even does. A tour does that work for them in a
minute: one sentence of context, before and after, then the ideas of the
change in chapters, most important first.

The tour is a ` ```happy-ui ` fence in your reply. Happy draws it with its own
components, in the conversation, node by node as you write it. Nothing else is
needed: no panel, no file, no HTML.

## Short is the point

A tour is read in place, in one or two screens of a narrow chat column.
Everything in it competes for that space, so every word earns it or goes.

- **Budget.** Every sentence is at most twenty words. Context is one
  sentence. One before/after, one sentence per side. Two or three chapters,
  each a title of a few words and one sentence. At most two `diff` nodes in
  the whole tour; a finding draws its own lines, so its chapter needs no diff
  of them. Aim for under 150 words of text in the block; a review with
  findings may take 200.
- **Steps only when order matters.** Leave `steps` out unless the change
  really happens in an order the reader needs; then two or three short items.
  Never restate the chapters as steps.
- **Ideas, not files.** Never list files and never put a path in a sentence.
  Say what the code does in ordinary words; a path appears only inside a
  `diff` or `finding` node, where Happy draws it.
- **Show code only when it says what words cannot:** the one function that is
  the idea, the line that is wrong. A diff range is the few lines that
  matter, eight at most; a finding's range is the wrong lines themselves,
  one to five. Happy shows no more than twelve.
- **Skip the routine.** Imports, plumbing, renames, formatting, lockfiles, and
  tests that just follow the code get no chapter of their own. A `tests`
  chapter is worth writing only when the tests say something about the change,
  and then in one sentence without a diff.
- **Write in reading order,** each node's `type` first and the block's
  `version`, `kind`, and `base` or `range` before `children`, as in the
  examples. Happy draws each node as it arrives, its words as you write them.

## A review is a tour with its findings inside

When someone asks to review a change, write the tour and put the review in
it, so each problem sits beside the code it is about:

- A `verdict` right after `context`: one sentence saying whether the
  change can go in and what must happen first. Happy counts the findings
  by severity above it, so never count them yourself.
- Each problem is a `finding` inside the chapter whose code it concerns,
  with `path` and `lines` on the exact lines that are wrong; Happy draws those
  real lines in the finding, so that chapter needs no separate diff of them. A
  problem with the change as a whole is a `finding` after the chapters,
  without `path` or `lines`; the two always come together.
- `severity` is `high` (must be fixed before it goes in), `medium` (should
  be fixed), or `low` (worth knowing). `title` says what is wrong, `text`
  why it matters, and `fix` what to do, a sentence each.
- Only real problems, the ones worth a reader's time; four at most, and
  only one with lines per chapter. No nitpicks, no praise in findings.
- In a review the findings are the code. Add a `diff` node only for the core
  idea of the change, and only when no finding already shows it. What is good goes in the verdict, briefly.

```
{ "type": "verdict", "text": "Safe to merge once the retry loop stops on errors." },
...
{
  "type": "finding",
  "severity": "high",
  "title": "A failed read is retried on every render",
  "text": "The error is not remembered, so the requests never stop.",
  "path": "packages/state/src/store.ts",
  "lines": { "startLine": 210, "endLine": 218 },
  "fix": "Keep the error with the revision it failed at."
}
```

After the fence, write at most one sentence, only for what you could not
check (tests you did not run, a screen you did not open). Nothing the block
already says is repeated outside it. A narrow question about one bug or one
file is answered directly, without a tour.

## Rules that make a tour trustworthy

- **Read the real diff first.** For this branch's work list every changed
  file with `git diff --name-only <merge base>` and `git status --porcelain
-uall` (new files count) and read them with `git diff <merge base>` and the
  new files themselves; for other commits use `git diff <base> <head>`. Read
  every file before writing a word. Every statement in the tour must be
  visible in that diff, and nothing important in the diff may be left out.
- **Chapters are ideas, not folders.** "Where a session opens" is a chapter;
  "packages/state" is not.
- **Every chapter has a role, and the order follows it.** `core` is an idea
  of the change; `supporting` is what an idea rests on (a store, a schema, a
  migration). The first chapter is `core`; core and supporting chapters may
  alternate. `tests` and `generated` come last and are drawn quieter. Happy
  refuses a tour that breaks this order.
- **Core and supporting chapters say what changed** in a `text` of one plain
  sentence.
- **Before/after describes behaviour,** not code. What a person saw or could
  do before, and what they see or can do now.
- **Point, do not reprint.** A diff node names a path and a line range in the
  file as it now stands; Happy shows the real lines from the change. Never
  write code or diff text yourself.
- **Line numbers are current-file numbers.** Check them against the file as it
  is now, not against the old version or the hunk header.
- **Plain text only.** Every string is plain words. No Markdown, no code
  spans, no backticks, no links.

## The format

```happy-ui
{
  "version": 1,
  "kind": "tour",
  "base": "<full merge-base hash>",
  "children": [
    { "type": "context", "text": "One sentence saying what this change does." },
    { "type": "beforeAfter", "before": "What happened before.", "after": "What happens now." },
    {
      "type": "chapter",
      "role": "core",
      "title": "The idea in ordinary words",
      "text": "One sentence on what changed and why.",
      "children": [
        { "type": "diff", "path": "packages/state/src/store.ts", "lines": { "startLine": 40, "endLine": 52 } }
      ]
    },
    {
      "type": "chapter",
      "role": "supporting",
      "title": "What it rests on",
      "text": "One sentence."
    }
  ]
}
```

Nodes, and nothing else:

| Node          | Fields                                                 | Use                                                     |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------- |
| `context`     | `text`                                                 | One sentence. First node of every tour.                 |
| `steps`       | `items`                                                | Optional: two or three steps, when their order matters. |
| `beforeAfter` | `before`, `after`, `beforeImage?`, `afterImage?`       | Behaviour before and after. Images are screenshots.     |
| `chapter`     | `role`, `title`, `text`, `children?`                   | One idea. `children` holds diff, mermaid, finding.      |
| `diff`        | `path`, `lines: { startLine, endLine }`                | Real lines from the change at that place.               |
| `mermaid`     | `source`                                               | A diagram, when the shape of a thing matters.           |
| `verdict`     | `text`                                                 | A review's conclusion, after `context`. At most one.    |
| `finding`     | `severity`, `title`, `text`, `path?`, `lines?`, `fix?` | One problem a review found, where it is.                |

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

The words stay and still describe behaviour: they caption the picture. A side
may have a picture while the other has none — a new dialog has no before. A
change nobody can see, such as a store or a migration, gets no pictures.

## Which change the tour shows

Every tour names the change it describes, beside `kind`: either `base` or
`range`, never both.

**This branch's work — the usual case — names `base`.** Happy reads the
checkout's live change: the working tree against the merge base with the
default branch. That is the whole branch, committed and uncommitted alike, so
committing more does not break the tour.

Take the base from Git, never from memory: `git merge-base origin/main HEAD`
(use the repository's default branch; without an `origin` copy of it, the
local one). Read the change with `git diff <base>` plus every untracked file
from `git status --porcelain -uall`, and give line numbers in the files as
they are now. Once the branch is merged, rebased, or synced, the tour says it
is no longer checked instead of showing a different change.

**Any other change — past commits, another branch, a merged PR — names a
`range`** of two full commit hashes:

```
"version": 1,
"kind": "tour",
"range": { "base": "<full hash>", "head": "<full hash>" },
"children": [ ... ]
```

Take both from `git rev-parse`, read the diff with `git diff <base> <head>`,
and give line numbers in the file as it is at `head`.

Under the tour Happy says how many of the changed files it shows code from.
That is a fact for the reader, not a target: do not add diffs to raise it.

A `tour` must start with `context`, include a `beforeAfter`, and have at
least one `chapter`; every chapter has a `role`, the first is `core`,
and `tests` and `generated` come last.

Use `"kind": "explanation"` for anything that is not a change: a subsystem, a
data flow, a decision. An explanation may use any nodes in any order, and its
chapters need no role. The same budget applies.

## Before you send it

- Does it fit in one or two screens: under 150 words (200 with findings),
  two or three chapters, two diffs at most, no steps unless order matters, no file lists,
  no paths in sentences?
- Is this a review? Then is there a `verdict` after `context`, and is every
  real problem a `finding` in the chapter it concerns, pointing at its lines?
- Is it this branch's work? Then is there a `base` from `git merge-base`?
  Is it any other change? Then is there a `range` from `git rev-parse`?
- Does each before/after describe behaviour you can point to in the diff?
- Is the change visible? Then does its before/after carry real screenshots,
  published as image artifacts and named by id and version?
- Does every chapter have a role, does the first one say `core`, and are
  tests and generated files last?
- Is every line range checked against the current file?

Then write one short sentence before the fence and send the fence. After it,
only what you could not check, if anything. Happy draws the rest.
