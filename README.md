# Cybershield

Live document scanner. Frontend is static HTML/CSS/JS; the backend is a small
Express server that keeps your Anthropic API key private.

## Structure
```
cybershield/
├── server.js          # backend (Express, POST /api/analyze)
├── package.json
├── .env.example       # copy to .env and add your key
└── public/
    ├── index.html     # markup
    ├── styles.css     # styling
    └── app.js         # frontend logic
```

## Run
1. Install Node 18+
2. `npm install`
3. `cp .env.example .env` and set `ANTHROPIC_API_KEY`
4. `npm start`
5. Open http://localhost:3000

## What's real and what isn't
- **Documents (PDF, DOCX, PPTX, TXT/MD):** text is extracted in the browser and analysed through the backend.
  Results show a score, a confidence level, and specific notes. Scores of 36-64 are reported as *inconclusive*.
- **Images (JPG, PNG, WebP, GIF):** analysed by Claude's vision (anatomy, lighting, textures, text rendering, artifacts). This is a visual assessment, **not** forensic analysis: it can't read metadata or run pixel-level tests, and good fakes can pass.
- **Links:** a rule-based check (look-alike brands, hidden destinations, raw IPs, punycode, shorteners, risky endings, disguised "name@" tricks, redirects, and more) plus a Claude second opinion. It reads only the text of the link and never opens it, so "no red flags" does not mean safe. Query values are removed before the link goes to Claude. Optionally set `SAFE_BROWSING_KEY` to add a Google Safe Browsing lookup (this sends the full link to Google).
- **Video:** placeholder only; it shows a clearly labelled "demo" result, never a fake score.

## Accounts & login
- Users sign up / sign in at `/login.html`; the scanner and `/api/analyze` require a session.
- Passwords are never stored: only salted **scrypt** hashes, in `data/users.json` (git-ignored, file mode 600).
- Sessions are signed, HttpOnly, SameSite=Lax cookies lasting 7 days. Set `SESSION_SECRET` in `.env`.
- Login/sign-up are limited to 8 attempts per minute per IP. Wrong-username and wrong-password look identical.
- Set `ALLOW_SIGNUP=false` once your accounts exist to make the site invite-only.
- Use HTTPS in production (`NODE_ENV=production` marks cookies Secure). No password reset or email verification yet.

## Notes
- Your API key never reaches the browser.
- Text is capped at 20,000 characters per scan; files at 15 MB; PDFs at 40 pages.
- Built-in per-IP rate limit (`RATE_LIMIT_PER_MIN`, default 20). Set `TRUST_PROXY=1` behind a proxy.
- Document text is treated as data, so instructions hidden inside a document don't steer the score.
- AI-text detection is probabilistic. Don't use a score alone to accuse anyone.
- For production, consider self-hosting the three CDN libraries (or adding SRI hashes) and a CSP.
