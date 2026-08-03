# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page, static quiz app that tests ROS 2 (Robot Operating System 2) concepts — specifically the mechanics of a minimal C++ publisher/subscriber node (`SimpleSubscriber`, topic `chatter`, `std_msgs::msg::String`). There is no backend, no build step, and no package manager: the whole app is `index.html` plus `questions.json`.

## Running it

There is no build/lint/test tooling in this repo. To run the app locally, serve the directory over HTTP (opening `index.html` directly via `file://` will fail because the `fetch("questions.json")` call is blocked by CORS on most browsers):

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

Verify changes manually in a browser — there is no automated test suite.

## Architecture

- `index.html` — the entire application: CSS in a `<style>` block, markup, and vanilla JS in a `<script>` block at the bottom. There is no separate JS/CSS file and no framework/bundler.
- `questions.json` — the quiz question bank, fetched at runtime via `fetch("questions.json")`. Must stay in the same directory as `index.html`.

### Data flow

1. On load, `index.html` fetches `questions.json` and normalizes each entry: it accepts either `{q, answers, correct}` or `{question, options, correctAnswer}` shapes (see the `.then(data => ...)` mapping), so `correct` can be supplied as a numeric index or as a letter (`"A"`-`"D"`) via `correctAnswer.charCodeAt(0)-65`.
2. Questions are shuffled client-side (`shuffle()`, Fisher-Yates) and held in the `questions` array along with mutable state (`current`, `correct`, `wrong`, `answered`) as top-level script variables — there is no framework state management.
3. `loadQuestion()` renders the current question and answer buttons; `checkAnswer(index)` locks in an answer, applies `.correct`/`.wrong` classes, and updates the score; `nextQuestion()` advances or calls `showFinalScreen()`; `restartQuiz()` reshuffles and resets counters.
4. Keyboard shortcuts are wired via a single `document.addEventListener("keydown", ...)`: digits `1`-`4` answer the current question, `Enter` advances to the next one.

### `questions.json` schema

Each entry needs: `q` (question text), `answers` (array of 4 option strings), `correct` (0-based index of the correct answer), and `explanation` (unused by the current UI, but present on every entry for context/future use). When adding questions, follow this exact shape — the alternate `question`/`options`/`correctAnswer` shape is only supported for backward-compatible parsing, not the format new entries should use.
