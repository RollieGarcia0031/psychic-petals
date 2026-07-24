# Psychic Petals — Novel-to-Database Sync Plan

## Overview

Automatically syncs new/updated chapters from `psychic-petals-novel` into Firestore whenever they're pushed to the novel repo.

## Architecture

```
[Push to novel repo main branch]
        │
        ▼
[GitHub Action triggers]
        │
        ├── Checks out novel repo (with git history for diff)
        ├── Checks out platform repo (for sync script + firebase-admin)
        ├── npm install in platform/backend
        ├── git diff --name-only to find changed .md files
        └── Runs sync-novel.js with changed file list
                │
                ▼
        [Firestore: novels/psychic_petals]
```

### Repo Responsibilities

| Repo | What it provides |
|------|-----------------|
| `psychic-petals-novel` | GitHub Action workflow + chapter `.md` files |
| `psychic-petals-platform` | Sync script + `firebase-admin` dependency |

---

## 1. Novel File Structure (expected)

Each `.md` file inside `main/episode-{NN}/` represents **one full chapter**. Chapter ordering comes from the numeric filename prefix.

```
psychic-petals-novel/
└── main/
    ├── episode-01/              ← episode 1
    │   ├── 01-classroom-intro.md  ← chapter 1
    │   ├── 02-new-friend.md       ← chapter 2
    │   └── 03-unknown-dreams.md   ← chapter 3
    ├── episode-02/
    │   └── 01-first-chapter.md
    └── ...
```

### Path → Episode/Chapter Mapping

| File path | Episode | Chapter |
|-----------|---------|---------|
| `main/episode-01/01-classroom-intro.md` | `01` → 1 | Extracted from `episode-{NN}` dir name |
| (chapter number) | | `01` → 1 | Extracted from `{NN}-*.md` filename prefix |

**Rule:**
- **Episode number** = first digits from parent directory name: `episode-01` → `01`
- **Chapter number** = first digits from filename (before first `-`): `01-classroom-intro.md` → `01`

---

## 2. Database Schema (Firestore)

**Collection:** `novels`
**Document ID:** `psychic_petals` (single document for the whole novel)

### Document Structure

```json
{
  "_id": "psychic_petals",
  "title": "Psychic Petals",
  "description": "A magical realism and slice of life novel...",
  "author": "RollieGarcia0031",
  "status": "draft",
  "createdAt": "2026-07-17T16:26:12Z",
  "updatedAt": "2026-07-24T...",
  "episodes": [
    {
      "episodeNumber": 1,
      "title": "The Girl Who Inspired the Songs",
      "summary": "Damien navigates high school...",
      "published": false,
      "chapters": [
        {
          "chapterNumber": 1,
          "title": "Chapter 1",
          "slug": "classroom-intro",
          "content": "Full markdown content here...",
          "wordCount": 1250,
          "lastEdited": "2026-07-24T..."
        },
        {
          "chapterNumber": 2,
          "title": "Chapter 2",
          "slug": "new-friend",
          "content": "...",
          "wordCount": 2100,
          "lastEdited": "2026-07-24T..."
        }
      ]
    }
  ],
  "metadata": {
    "tags": ["magical-realism", "slice-of-life"],
    "totalWords": 3350
  }
}
```

### Update Strategy

**Idempotent upsert** — the sync script:
1. Reads the current Firestore doc
2. Finds or creates the episode by `episodeNumber`
3. Finds or creates the chapter by `chapterNumber`
4. Updates `content`, `title`, `wordCount`, `lastEdited`
5. Recalculates episode and novel `totalWords`
6. Writes back with `set({ ... }, { merge: true })`

Safe to re-run. No duplicate data.

---

## 3. Sync Script

### File: `psychic-petals-platform/backend/scripts/sync-novel.js`

**Purpose:** Node.js script that reads changed chapter markdown files and updates Firestore.

**Usage:**
```bash
node scripts/sync-novel.js \
  --novel-dir ../../novel \
  --changed "main/episode-01/01-classroom-intro.md,main/episode-01/02-new-friend.md"
```

**Input:**
- `--novel-dir` — path to the novel repo root
- `--changed` — comma-separated list of relative file paths that changed

**Logic (pseudocode):**

```
for each changed file path:
  // Parse path
  match path = /main\/episode-(\d+)\/(\d+)-(.+)\.md$/
  episodeNumber = parseInt(capture[1])
  chapterNumber = parseInt(capture[2])
  slug = capture[3]

  // Read markdown
  content = readFile(path)
  title = extract first heading (# <center> ... </center> or # ...)
  wordCount = content.split(/\s+/).length

  // Build Firestore update
  findOrCreate episode in doc.episodes where episodeNumber matches
  findOrCreate chapter in episode.chapters where chapterNumber matches
  chapter.title = title
  chapter.content = content
  chapter.wordCount = wordCount
  chapter.lastEdited = now()
  chapter.slug = slug

recalculate totalWords for each episode and novel
write doc back to Firestore
```

**Dependencies used:**
- `firebase-admin` (already in `psychic-petals-platform/backend/package.json`)
- Node.js built-ins: `fs`, `path`

**Firebase credentials:** Read from `GOOGLE_APPLICATION_CREDENTIALS` env var (file path) or accept `GOOGLE_APPLICATION_CREDENTIALS_JSON` env var (inline JSON).

---

## 4. GitHub Action Workflow

### File: `psychic-petals-novel/.github/workflows/sync-to-db.yml`

```yaml
name: Sync Novel to Database

on:
  push:
    branches: [main]
    paths: ['main/**/*.md']

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout novel repo
        uses: actions/checkout@v4
        with:
          path: novel
          fetch-depth: 2

      - name: Checkout platform repo
        uses: actions/checkout@v4
        with:
          repository: RollieGarcia0031/psychic-petals-platform
          path: platform

      - name: Setup Node.js
        uses: actions/setup-node@v4

      - name: Install backend dependencies
        working-directory: platform/backend
        run: npm install

      - name: Get changed chapter files
        id: changed
        run: |
          cd novel
          CHANGED=$(git diff --name-only ${{ github.event.before }} ${{ github.sha }} \
            -- 'main/**/*.md' | tr '\n' ',' | sed 's/,$//')
          echo "files=$CHANGED" >> $GITHUB_OUTPUT

      - name: Sync to Firestore
        env:
          GOOGLE_APPLICATION_CREDENTIALS_JSON: ${{ secrets.FIREBASE_SERVICE_ACCOUNT_KEY }}
        run: |
          echo "$GOOGLE_APPLICATION_CREDENTIALS_JSON" > /tmp/service-account.json
          export GOOGLE_APPLICATION_CREDENTIALS=/tmp/service-account.json
          cd platform/backend
          node scripts/sync-novel.js \
            --novel-dir ../../novel \
            --changed "${{ steps.changed.outputs.files }}"
```

### Trigger Condition

Only runs when `.md` files under `main/` change on the `main` branch. Outline and story-bible changes are ignored.

---

## 5. GitHub Secrets Required

| Secret Name | Description |
|-------------|-------------|
| `FIREBASE_SERVICE_ACCOUNT_KEY` | Full JSON content of Firebase Admin SDK service account key |

---

## 6. Files to Create

| File | Location | Purpose |
|------|----------|---------|
| `scripts/sync-novel.js` | `psychic-petals-platform/backend/scripts/` | The sync script |
| `sync-to-db.yml` | `psychic-petals-novel/.github/workflows/` | The GitHub Action |

---

## 7. Edge Cases & Safety

| Scenario | Handling |
|----------|----------|
| **First run (no existing doc)** | Script creates the novel document from scratch |
| **Re-running same chapter** | Idempotent — overwrites same chapter, no duplicates |
| **Multiple chapters in one push** | Iterates over all changed files in one run |
| **Chapter renamed** | Old chapter stays until a full re-sync is triggered |
| **Empty diff** | Script exits early if no `.md` files changed |
| **Firestore quota** | Single document writes are well within Firestore free tier limits |
| **Submodule sync** | If novel submodule updates in parent repo, action still triggers from novel repo |

---

## 8. Future Considerations

- **Full re-sync:** A workflow dispatch trigger to rebuild the entire Firestore document from all current `.md` files
- **Outline sync:** Optionally sync outline files to a separate `outlines` collection
- **Story bible sync:** Optionally sync character/lore files for a wiki/documentation site
- **Frontend integration:** The Next.js frontend reads from the same Firestore to display chapters
