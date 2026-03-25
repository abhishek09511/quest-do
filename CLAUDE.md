# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build system. Open `index.html` directly in a browser. No install, compilation, or server needed. No tests or linting configured.

## Architecture

Single-file SPA (`index.html`, ~1300 lines) — all HTML, CSS, and JS inline. No frameworks. Data persists in `localStorage` under key `questdo`.

**External CDN dependencies:** canvas-confetti 1.9.3, Google Fonts (DM Sans).

### CSS (lines ~9–450)

Multi-theme system using two data attributes on `<html>`: `data-theme` (visual personality) + `data-mode` (color palette).

**Four combinations:**
- `data-theme="glow" data-mode="dark"` — Arc/Raycast-inspired: deep surfaces `#0f0f1a`, purple/cyan/coral/gold accents, soft neon glow effects (default)
- `data-theme="glow" data-mode="light"` — Todoist-inspired: white bg, multicolor accents, flat task rows, no glow
- `data-theme="linear" data-mode="dark"` — Linear.app-inspired: flat dark `#1a1a1a`, monochrome blue `#5e6ad2`, no glow
- `data-theme="linear" data-mode="light"` — Notion-inspired: clean white, single blue accent, minimal borders

**Variable architecture:**
- `:root` fallback = glow-dark (safe default if no attributes set)
- Each theme block defines color vars + personality vars (card shape, shadows, glow effects)
- Personality vars (`--card-bg`, `--card-radius`, `--sidebar-shadow`, `--toast-bg`, etc.) control visual personality via CSS variables
- Structural overrides (`[data-theme="linear"]`, `[data-theme="glow"][data-mode="light"]`) handle things that can't be expressed as variables (border-bottom-only cards, plain text brand)
- RGB channel variables (`--accent-rgb`, etc.) enable `rgba(var(--accent-rgb), 0.1)` patterns
- All colors must use `var()` — no hardcoded color values outside variable definitions

**localStorage:** `questdo-theme-name` (`"glow"` | `"linear"`) + `questdo-theme-mode` (`"dark"` | `"light"`). Flash-prevention `<script>` in `<head>` reads both keys before paint and migrates old `questdo-theme` key.

**Theme picker:** Segmented toggle (Glow/Linear) + mode button in sidebar bottom. Header toggle button is a dark/light shortcut.

### JS (lines ~470–1315)

Global `state` object (saved to localStorage via `save()`). No modules, no classes — all functions are global.

**Key state shape:**
```
state.tasks[]    — {id, text, done, folder, deadline, subtasks[], pomoCount, description, comments, expanded}
state.folders[]  — string array
state.xp, state.totalDone, state.streakDays[], state.activeFolder, state.sortByDeadline, state.hideDone
```

**Render cycle:** `renderAll()` calls `renderFolderNav()`, `renderFolderSelect()`, `renderStats()`, `renderViewTitle()`, `renderTasks()`, `setupFolderDropTargets()`, plus conditionally `renderPomo()` and `renderDetail()`. Task list is rebuilt via DOM manipulation (innerHTML + addEventListener) each render.

**Gamification:** XP awards on task completion and pomodoro sessions. 10 levels defined in `LEVELS[]`. Streaks tracked by date strings. Confetti and sound effects on milestones.

**Pomodoro:** Global `pomo` object tracks timer state. Uses `setInterval` for countdown. Ring SVG progress updated in `renderPomo()` using computed CSS variable colors.

### HTML (lines ~380–470)

Layout: fixed `.sidebar` (left) + `.main` (center) + sliding `.pomo-panel` (right) + sliding `.detail-panel` (right). Modal overlay for task editing. Mobile: sidebar collapses, panels go full-width at 700px breakpoint.

## Key Conventions

- Task completion auto-repositions: done tasks sink to bottom of array
- Subtask completion can auto-complete parent task (when all subtasks done)
- Drag-and-drop: tasks reorder in list, or drop onto folder nav items to move
- Theme system: `setTheme(name, mode)` sets both `data-theme` and `data-mode` on `<html>`, saves to localStorage, and updates UI. `toggleMode()` flips dark/light within current theme.
- All user-facing text uses `escapeHtml()`/`escapeAttr()` for XSS safety
- Hyperlink support: `[text](url)` markdown in task/subtask text, rendered as clickable `<a class="task-link">` via `renderLinkedText()`. Cmd+K (or Ctrl+K) opens a link popup when text is selected in allowlisted inputs. Links restricted to `https?://` for security.
- Sound via Web Audio API (`playSound()`) — no audio files
