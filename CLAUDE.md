## Project Overview

This is a fishing jig inventory management app with Python backend (main.py, db.py) and JavaScript/HTML frontend. Key domains: BOM (Bill of Materials), order parsing, supply tracking, hook/eyeball/split ring components with sizes and colors.

## Development Rules

When adding new features that span backend and frontend, always verify file extensions match their content (e.g., files with JSX must use .jsx extension, not .js). Run `npx vite build` or equivalent to catch parse errors before considering a task complete.

When working with the preview server, always kill any existing process on the target port before starting a new one. Use `lsof -i :<port> | grep LISTEN` to check, then `kill -9 <pid>` if needed.

## Netlify Usage & Costs

The Claude API is called via a Netlify serverless function (`netlify/functions/claude-proxy.js`). Every PDF parse, Gmail import, and image parse burns a Netlify function invocation. The free tier allows 125k invocations/month — heavy testing can exhaust this quickly.

Before adding any new feature that calls the Claude API, consider:
- Is this call happening in a loop or on every keystroke? Batch it.
- Could this be cached? Don't re-parse the same document twice.
- Is this triggered by the user explicitly, or automatically? Automatic calls are dangerous.

If the site hits the Netlify free tier limit, deploys and functions are blocked but the hosted site stays up. Upgrade at app.netlify.com → Team Settings → Billing, or consider moving API calls directly to the frontend (acceptable for this internal-only tool).

## Testing

When implementing parsers for PO/PDF imports, test against real document samples early. Multi-word values (e.g., color names like 'Pearl White') and quantity calculations (pack sizes) are common edge cases in this codebase.
