# LivePad

A single-file, password-gated, live collaborative notepad with an Obsidian dark theme. Static site, works on GitHub Pages.

## How it works

- No backend and no accounts. Sync is relayed over encrypted websockets (port 443) via public MQTT brokers, which works on networks that block WebRTC/P2P.
- The password (plus optional note name) derives the room topic and an AES-GCM key (PBKDF2, 600k iterations). All note traffic is end-to-end encrypted; the broker and anyone without the password see only ciphertext.
- The editor unlocks only when at least one other live participant is in the room (mutual presence), and goes back to waiting if everyone leaves.
- The broker retains the last encrypted state, so a late joiner gets the current note even if the other side has since gone offline. Each browser also keeps an encrypted copy in localStorage.
- Paste images straight into the pad (Ctrl+V): they are downscaled, encrypted, and synced as content-addressed blobs on retained subtopics.

## Deploy to GitHub Pages

```bash
git init
git add index.html README.md
git commit -m livepad
git branch -M main
git remote add origin git@github.com:<you>/livepad.git
git push -u origin main
```

Then: repo Settings > Pages > Source: `main` branch, `/ (root)`. Your pad is live at `https://<you>.github.io/livepad/`.

Or with the GitHub CLI:

```bash
gh repo create livepad --public --source=. --push
```

Then enable Pages as above.

## Usage

1. Open the page, pick a note name (optional) and a password.
2. Share the URL, note name, and password with your collaborator.
3. Type. Both sides see edits within a fraction of a second.

## Notes and limits

- Sync rides free public MQTT brokers (broker.emqx.io, test.mosquitto.org). Fine for a notepad; do not treat them as guaranteed infrastructure.
- Not end-to-end audited; use a strong password. Anyone with the password can read and edit.
- One room is one note. Use different note names for multiple pads.
