# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal, single-page gardening checklist for a Dallas, PA (Zone 6a) vegetable/flower garden, covering March through harvest. There is no build system, package manager, server, or test suite — this is a static HTML page plus its Markdown source, published via GitHub Pages.

## Files

- `gardening-checklist.md` — the source of truth for content (crop list, weekly tasks, seasonal reference tables, soil notes). Edit this first when checklist *content* changes (dates, tasks, crops, notes).
- `index.html` and `gardening-checklist.html` — **identical files**, both the rendered/interactive version of the Markdown content plus embedded CSS and JavaScript. `index.html` is what GitHub Pages serves as the site root.

## Critical workflow: keeping files in sync

There is no generator/build step — the HTML is hand-authored and must be kept in sync manually:

1. When checklist content changes, update `gardening-checklist.md` first.
2. Apply the same content change to `index.html`, translating Markdown into the existing HTML structure (see below).
3. **Copy `index.html` over `gardening-checklist.html` (or make identical edits to both)** — verify with `diff index.html gardening-checklist.html` before committing. These two files must always be byte-identical.

## HTML structure and conventions

Each checklist item in `index.html` is a checkbox with a stable, unique `id` following the pattern `w{weekNumber}-{itemIndex}`, e.g. `id="w1-1"`, `id="w12-4"`. This ID is load-bearing:

- The embedded JS persists checked state to `localStorage` keyed by `garden-{id}`.
- Week headers use `id="week-{weekNumber}"` (e.g. `id="week-1"`) so the "current week" highlighter can find and scroll to them.
- The `WEEK_MAP` / `WEEKS` arrays in the `<script>` block (near the bottom of `index.html`) hardcode each week's date range (`[weekNum, startMonth, startDay, endMonth, endDay]`, months 0-indexed). **If you add, remove, or renumber weeks, update these arrays and the `getMonthForCheckbox` month-boundary logic (also in the script) to match** — they are not derived from the DOM.

When adding a new week or checklist item:
- Give every new `<input type="checkbox">` a unique, sequential `id` matching the `w{week}-{item}` convention.
- Keep the visual structure per week consistent: an `<h4 id="week-N">` heading, an optional `<blockquote>` context note, then a `<ul>` of checkbox `<li>` items (nested sub-bullets use plain `<li>` without checkboxes, per existing examples).
- Update the "Seasonal Quick Reference" table in both the Markdown and HTML if start/sow/harvest dates change.

## No tooling

There is no linter, formatter, test runner, or build command in this repo — validate changes by opening `index.html` in a browser and checking that checkboxes persist, the progress bar updates, and the current-week highlight appears correctly for today's date.
