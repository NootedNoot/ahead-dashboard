# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static GitHub Pages dashboard that shows the user's real-time glucose data and the backend's trend detection output. Two pages: `index.html` (main dashboard) and `viewer.html` (a simpler glucose-now display). No build step — deployed as-is via `.github/workflows/static.yml`, which pushes anything on `main` straight to GitHub Pages.

## Commands

None — no package.json, no build/lint/test tooling. Open the HTML files directly in a browser to preview, or push to `main` to deploy (the Actions workflow handles the rest automatically).

## Architecture

`index.html` talks to two separate Railway deployments — don't confuse them:

- `NS_URL` (`https://web-production-5e0b.up.railway.app`) — the Nightscout instance, fetched directly for raw glucose entries (`/api/v1/entries.json`).
- `BACKEND_URL` (`https://ahead-backend-production-ee80.up.railway.app`) — `ahead-backend`'s Express API, fetched for `/api/latest-trend` (the server-side trend-detection result; see `../ahead-backend/CLAUDE.md`).

As a static, publicly-served page, this fetches `BACKEND_URL` without sending an API key — there's nowhere safe to hold one client-side. If `ahead-backend`'s auth middleware is ever actually enforced in production (currently it isn't — see `../CLAUDE.md`), this dashboard's `/api/latest-trend` call breaks and needs a real fix, not just adding a hardcoded key to this file.

`viewer.html` is a separate, simpler standalone page (its own inline `<style>`/`<script>`, not sharing code with `index.html`).

### Removed: "Ask Ahead" AI-analysis card

An earlier version of `index.html` had an "Ask Ahead" card that POSTed to the backend's `/analyze` Gemini endpoint for on-demand AI trend analysis. It was deliberately removed (premature without insulin-on-board/food-intake context — see `../ahead-backend/CLAUDE.md`) along with the corresponding backend route. Don't re-add a card calling `/analyze` — that endpoint no longer exists. The separate "Ahead Trend Check" card (`fetchLatestTrend()`, calling `/api/latest-trend`) is unrelated and was kept.
