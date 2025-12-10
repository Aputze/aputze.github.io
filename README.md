# Sergei Lerner Portfolio

Single-page portfolio for showcasing AI automation, ERP integration, and cloud projects. The site lives at https://aputze.github.io and highlights hands-on work spanning Priority ERP, AI/ML, IoT, and automation tooling.

## What you’ll find
- Hero header with marquee animation, KPI badges, and calls-to-action for projects and LinkedIn.
- Skills grid covering AI/ML, ERP & business systems, cloud/DevOps, and automation tools.
- Highlighted projects (Priority ERP MCP server, ERP chatbot, MedOp OCR/NLP pipeline, SmartThings automations, IoT + ERP workflows, dashboards, AI research/assistant demos, and more).
- Freelance services overview, About section with selected certificates, and recent certification cards.
- Blog/Insights placeholders for future posts.
- Contact section with mailto/LinkedIn/GitHub buttons, Formspree-powered form, embedded AI career assistant (Gradio), plus a modal demo for the Whispr voice assistant.

## Tech stack
- Static HTML, CSS, and vanilla JavaScript (no build step required).
- Media assets under `pics/`; supporting docs in `docs/` (CVs) and `projects/priority-mcp-server-read-more.md`.
- External services: Formspree form endpoint (`https://formspree.io/f/xpwleyez`), Google Analytics (G-5B9B8J4PK9), Gradio web components for the embedded assistants, optional background videos in `pics/`.

## Run locally
1. Clone the repo and open the root directory.
2. Serve the static files (examples):
   - `python -m http.server 4173`
   - `npx serve .`
3. Open `http://localhost:4173` (or the port you choose). The contact form needs network access to reach Formspree; local-only viewing will still render the page, but submissions won’t send.

## Editing guidance
- Main page content, styling, and scripts live in `index.html`. Update hero text, skills, or project cards directly there.
- Project details: adjust links and descriptions in the “Highlighted Projects” section; long-form MCP writeup sits in `projects/priority-mcp-server-read-more.md`.
- Media: place new images/videos in `pics/` and update the corresponding `src` attributes.
- Contact form: change the Formspree `action` URL if you want submissions to route elsewhere.
- Analytics: swap the GA measurement ID in the `<head>` if needed.

## Deployment
- GitHub Pages can serve the root of this repo directly (no build step). Push changes to the published branch and Pages will reflect the update.
- If you prefer another static host (S3, Netlify, Cloudflare Pages, Vercel), deploy the repository root or the `dist/` folder as static assets.

