# 🦎 Ezekiel's 6th Birthday — Interactive Invite

`index.html` is a self-contained, mobile-friendly birthday invite.

## ✨ What it does
- Reptile theme (neon-green title, animated snake/lizard/turtle/frog/gecko)
- Live countdown to **Fri 3 July 2026, 3:00pm**
- **Text RSVP** button (pre-filled SMS to Ashby) + tap-to-call
- **Add to Calendar** button (downloads an `.ics` event)
- Tap the address to open **Google Maps**
- 🔊 Optional jungle ambience toggle (top-right)

## 📸 Add Ezekiel's photo
Save his photo as **`ezekiel.jpg`** in the same folder as `index.html`
(commit it to the repo root). It appears automatically in the round frame.
Until then, a styled placeholder shows.

## 🌐 Share link
A GitHub Actions workflow (`.github/workflows/pages.yml`) publishes the invite
to GitHub Pages on every push. After the first run, the link is:

**https://myrtllabs.github.io/**

> First time only: in **Settings → Pages**, set the source to **GitHub Actions**
> if it isn't already (the workflow attempts to enable it automatically).

To preview locally, just open `index.html` in a browser.
