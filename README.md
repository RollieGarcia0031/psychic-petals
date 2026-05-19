# Psychic Petals

> Psychic Petals is a fictional novel written using the markdown
> syntax maintained in Github. This is an open-source novel where
> everyone can work together to build a unique story that's relatable
> with a great depth.

<br>

## Guidelines:
> This repository follows a certain rules for editing and suggesting.

#### File Structure:
```
├── main                        # The finalized prose of the story
│   └── episode-01
│       └── chapter-01
│           └── 01-the-classroom-intro.md
├── outlines                    # Story planning (mirrors main structure)
│   └── episode-01              
│       ├── episode-01.md
│       └── chapter-01         
│           ├── chapter-01.md   # A summary for entire chapter
│           └── 01-the-classroom-intro.md # A scene by scene breakdown
└── story-bible                 # Reference material and world-building
    ├── characters              # Individual character profiles (e.g., damien.md)
    ├── lore                    # World mechanics, history, and systems
    └── settings                # Location details and regional data
```

#### Editing:
* Any tasks such as moving files, writing chapters must be done **only in
the terminal**, with cli tools and *vim/neovim*.
* Only **one scene** (a single markdown file) can be added per commit.
* **Suggestions** can be made through opening issue with label [bug].
