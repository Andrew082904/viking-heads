## Project Overview

This is a fishing jig inventory management app with Python backend (main.py, db.py) and JavaScript/HTML frontend. Key domains: BOM (Bill of Materials), order parsing, supply tracking, hook/eyeball/split ring components with sizes and colors.

## Development Rules

When adding new features that span backend and frontend, always verify file extensions match their content (e.g., files with JSX must use .jsx extension, not .js). Run `npx vite build` or equivalent to catch parse errors before considering a task complete.

When working with the preview server, always kill any existing process on the target port before starting a new one. Use `lsof -i :<port> | grep LISTEN` to check, then `kill -9 <pid>` if needed.

## Hosting — GitHub Pages (migrated from Netlify Mar 2026)

The app is hosted at: https://andrew082904.github.io/viking-heads/viking-heads.html
Repo: https://github.com/Andrew082904/viking-heads

To deploy updates: commit changes and `git push origin master` — GitHub Pages auto-deploys within ~60 seconds.

**GitHub tokens:** stored in git remote URL and in user's local memory files (not in code — GitHub secret scanning will block pushes containing tokens).

**The Anthropic API key is stored in the user's browser localStorage (not in code).** If it needs to be reset, call `resetAnthropicKey()` from the browser console. Never hardcode the API key — GitHub's secret scanning will block the push.

**Google OAuth:** The OAuth client must have `https://andrew082904.github.io` in Authorized JavaScript Origins and `https://andrew082904.github.io/viking-heads/viking-heads.html` in Authorized Redirect URIs (console.cloud.google.com → APIs & Services → Credentials).

## Why We Left Netlify

Netlify's free tier has 125k function invocations/month. Heavy testing of the Claude API proxy (PDF parsing, Gmail import, image parsing) exhausted this in one session. The site was disabled mid-session.

The fix: API calls now go directly from the browser to Anthropic — no serverless proxy needed. This works because the app is internal-only (just the Viking Heads team), so exposing the API key in localStorage is acceptable. There are no Netlify function limits to worry about.

**Lesson: Never add automatic or looping Claude API calls.** Every parse must be explicitly triggered by the user. Caching parsed results is strongly preferred over re-parsing.

## Gmail Pull — Known Edge Cases

- **Reply threads (Re: subject):** A customer replying to an invoice email may have the original invoice PDFs still attached. The app flags "Re:" subjects as suspicious (yellow warning badge) — always review before importing.
- **Outgoing invoices:** Viking Heads' own invoices (e.g., invoice_1666.pdf, invoice_1667.pdf) can appear as attachments in customer reply threads. These are NOT purchase orders. The suspicious email filter catches most of these.
- **Product type confusion:** Swingheads (VKS) and VK Heads (VKH) are different products. The parser prompt must include the full SKU catalog with descriptions to avoid misidentification.

## Testing

When implementing parsers for PO/PDF imports, test against real document samples early. Multi-word values (e.g., color names like 'Pearl White') and quantity calculations (pack sizes) are common edge cases in this codebase.
