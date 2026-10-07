# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

「三子輪替井字棋」: a tic-tac-toe variant where each player may have at most 3 pieces on the board. Placing a 4th removes that player's oldest piece, so there are no draws. The whole game is the single file `tic-tac-toe.html`. There is no build, lint, test, or package setup. To run it, open the file in a browser.

The page is published as a claude.ai Artifact: https://claude.ai/artifact/5F5A9nkhDFLMFWQLb5zUUh (linked from `01-小專案說明.docx`). Because the Artifact host wraps the page in a document skeleton, `tic-tac-toe.html` deliberately has no `<!doctype>`, `<html>`, `<head>` or `<body>` tags. Don't add them. To update the live page, republish this file to that URL.

All user-facing text, including UI strings, aria-labels, and code comments, is in Traditional Chinese. Keep it that way.

## Git workflow

Commit and push to GitHub (`origin` → https://github.com/Steven-Shih-0730/01-tic-tac-toe---, branch `main`) regularly as you work, so no progress is ever lost. You don't need to ask before doing this.

- Commit each logical unit of work as soon as it is complete and working (one feature, one fix, one doc update). Don't batch unrelated changes into one commit, and don't let work sit uncommitted at the end of a task.
- Push right after committing (`git push`).
- Write clean commit messages: a short imperative summary line (≤ 72 chars, e.g. `Add hover preview for CPU turn`), then a blank line and a brief body explaining *why* when it isn't obvious.
- If a push fails (auth, rejected, network), tell the user instead of forcing it. Never `git push --force` unless the user explicitly asks.

## Docs that must stay in sync

`說明文件_遊戲規則與設計巧思.txt` and `01-小專案說明.docx` describe the rules, the CPU strategy, the accessibility features, and timings such as the 0.45s CPU delay. The in-page `<aside class="rules">` repeats the rules too. If you change game behavior, update all three.

## Architecture (inside the `<script>` IIFE)

- **State**: `board` (9 cells holding `'O'`, `'X'` or `null`), `queues` (per-player FIFO of cell indices, in placement order), `turn`, `winner`/`winLine`, `busy` (the CPU is thinking), `score`, and `starter` (whoever wins, `starter` flips so the next game alternates the first player).
- **`place(i)`**: the core rule. If the player's queue already holds `MAX` (3) pieces, it shifts out the oldest piece and clears that cell before placing the new one, then checks for a win. Order matters, because the removed piece must not count toward a line.
- **`render(justPlaced, removed)`**: one pass that syncs the whole DOM to the state. It computes the "doomed" cell (`queues[turn][0]` when the queue is full) for the blink hint. It only re-injects the SVG (which replays the draw animation) for the cell that was just placed. The removed cell keeps its `.vanish` class for 350ms before it is cleared. That number is tied to the CSS `vanish` animation duration.
- **CPU** (`cpuPick`, `simulate`, `maybeCpu`): the CPU is always X. It plays a winning move first, then blocks, then follows the preference order center, corners, edges, filtered to avoid moves that let O win immediately. `simulate` runs a move on copies of the board and queue so the rotation rule is taken into account. The safety filter in `cpuPick` temporarily mutates the global `board`/`queues` and then restores them. Watch for this if you refactor it. `maybeCpu` sets `busy` and moves after 450ms, and it re-checks its state in case the user toggled the CPU off or restarted in the meantime.
- **Styling**: color tokens are defined on `:root`. They are redefined for dark mode both under `prefers-color-scheme: dark` (guarded by `:not([data-theme="light"])`) and under `[data-theme="dark"]`, so a dark-mode color change must be made in both blocks. `prefers-reduced-motion` turns off the animations but keeps the doomed piece faded.
- **Accessibility**: each cell is a `<button>` with an aria-label (`第 N 格：圈/叉/空`), and the status line uses `aria-live`. Keep this when you change the cell markup.
