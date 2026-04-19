# omarfaruk.bd

Personal portfolio of **Omar Faruk** — product engineer with a security habit.

Single static HTML file. No build step, no framework. The halftone portrait is rendered live via a small canvas routine; the source image is inlined as base64. Terminal/system aesthetic, dark + light themes, responsive, konami-code easter egg.

- **Live:** https://omarfaruk.bd
- **Deploy:** static site on Vercel
- **Entry:** `index.html`
- **Social card:** `og.png`
- **Analytics:** [onedollarstats](https://onedollarstats.com) (`stonks.js`)

## Local preview

Any static server works:

```sh
python3 -m http.server 8080
# or
bunx serve .
```

Then open http://localhost:8080.
