# Psychic Petals

> **Psychic Petals** is an open-source **magical realism** and **slice of life** novel. While the world features a corporate-regulated hero ecosystem and psionic societies, the narrative focuses on the quiet, personal moments of its characters rather than superhero action or monster-slaying mechanics.
>
> The story follows Damien, a young man with the rare ability to manifest his imagination into physical constructs, as he navigates the complexities of school, social isolation, and the weight of his unspoken dreams.

<br>

> [!WARNING]
> **Spoiler Alert:** If you want to experience the story as intended, avoid the `outlines/` and `story-bible/` directories. These contain plot spoilers, character secrets, and deep lore that may reveal future events.

<br>

## 📖 How to Read
The novel is organized chronologically. To read the story properly:
1. Navigate to the `main/` directory.
2. Follow the episodes and chapters in numerical order.
3. Read the scene files (e.g., `01-...`, `02-...`) sequentially.

---

## 🛠 File Structure
```
├── main/               # THE STORY: Finalized prose (Start here!)
│   └── episode-01/
│       └── chapter-01/
│           └── 01-the-classroom-intro.md
├── outlines/           # THE PLAN: Mirrored structure for beat sheets
│   └── episode-01/
│       ├── episode-01.md
│       └── chapter-01/
│           ├── chapter-01.md
│           └── 01-the-classroom-intro.md
└── story-bible/        # THE LORE: Character profiles and world mechanics
    ├── characters/     # Individual profiles (e.g., damien.md)
    ├── lore/           # Systems, history, and biology
    └── settings/       # Locations and regional data
```

---

## 🤝 Contributing
**Everyone is welcome to contribute!** Whether you want to fix a typo, suggest a plot point, or write a new scene, your input helps build this world.

### How to Help:
*   **Open Issues:** Found a continuity error or have a suggestion? Open an issue with the `[bug]` or `[suggestion]` label.
*   **Pull Requests:** Want to write? Submit a PR! 
    *   Only **one scene** (a single markdown file) per commit/PR.
    *   Ensure new scenes align with the existing `outlines/` or propose an outline change first.

### Engineering Standard:
To maintain the "hacker-writer" aesthetic of this project, all edits to the story and structure must be performed via **terminal tools** (e.g., `vim`, `neovim`, or CLI-based agents).

---
*Psychic Petals is a collaborative experiment in open-source storytelling.*
