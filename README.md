# SMACK! (standalone version)

This ZIP holds the working single-file version of SMACK!. It is NOT a React/Vite project, so there is no package.json, no npm install and no build step.

## Run it
1. Extract the ZIP.
2. Double-click `index.html` to open it in Chrome or Edge.
   Voice input works best from a local server, since some browsers restrict the mic on file:// pages. If the mic button is blocked, run `python3 -m http.server 8000` in this folder (or `npx serve`) and open http://localhost:8000.

## Microphone
- Use Chrome or Edge. Firefox has no speech recognition, and Safari support is limited.
- The browser asks for mic permission only when you press the mic button. Speech recognition needs internet, since the browser's speech service handles the audio. The game stores nothing.
- Without recognition, the typing box works the same way.

## Turning it into the React/Vite app
`SMACK_PORT_SPEC.md` describes the logic for a full port. Give it, `index.html` and your original prompt to Claude Code in an empty folder.
