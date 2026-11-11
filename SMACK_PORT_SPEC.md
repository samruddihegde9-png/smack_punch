# SMACK! — Port Spec for Claude Code

Reference implementation: a single-file vanilla JS prototype (`smack.html`, published as an artifact). Port it to React + TypeScript + Vite + Tailwind + Framer Motion + Matter.js, keeping the behavior below. The prototype is the source of truth; open it and read the script.

## Suggested mapping
- `lib/rageEngine.ts` — pure functions, unit-testable
- `lib/dialogueEngine.ts` — categories, `fresh()` no-repeat picker, `reply()`
- `lib/audioEngine.ts` — Web Audio synth (slap, boing, squeak, spring, fall, chime, rage)
- `lib/physics.ts` + `hooks/useGamePhysics.ts` — Matter.js body(ies) replacing the custom springs
- `hooks/useSpeechRecognition.ts`, `hooks/useSpeechSynthesis.ts`
- Components: GameStage, CartoonCharacter (SVG, head as a separate rotating group), RageMeter, VoiceControls, CussBox, ObjectTray, GameHUD, SettingsPanel
- Keep per-frame physics values in refs and write to the DOM in the rAF loop. Use React state only for HUD numbers, and update those on change.

## Rage engine
- Levels: 1 CHILL ≤20, 2 ANNOYED ≤40, 3 PISSED ≤60, 4 FURIOUS ≤85, 5 UNHINGED >85.
- Per message: `gain = 5 + min(len,80)*0.12 + profanityCount*6 + recentMsgsIn8s*1.5 + rand(0,4) + (sameAsLast?4:0) + (strongWord?4:0)`, capped at 22 so one message never hits 100. Strong words: motherfuck, cunt, dickhead, asshole.
- Profanity regex: `fuck|shit|bitch|ass|damn|hell|crap|bastard|dick|piss|cunt|twat|arse|bollocks`.
- Slap while rage > 0 adds `2 + power*2`. A slap at rage ≥ 95 triggers meltdown.
- Decay: 1.1/s normally, 5/s after a meltdown ("cool" flag, until rage ≤ 1).
- Meltdown at 100: lock input rage, start chaos (9 s) + confetti, knock down with a spin, and say 5 rant lines about 2.3 s apart (4 fixed + 1 random). Then set rage to 45 and say a calm-down line. The character can't stand up while melting.

## Dialogue engine
- Level lines for levels 1-4 (level 5 uses rants). Categories are matched by regex on the lowercased text, first match wins with 85% probability. Order, most specific first: shut-up, you suck, bastard, bitch, dumbass, motherfucker, asshole, dickhead, fuck-you, shut-up (simple), hate, ugly, stupid/idiot, annoying, nice/love, greeting.
- Fallback: 50% chance to echo the first profane word ("WORD?! I'LL SHOW YOU WORD!"), else a level line.
- `fresh(arr)` avoids the last 8 spoken lines.
- Extra pools: slap-while-angry lines, falling lines, getting-up lines, post-meltdown lines, generic quips ("EXCUSE ME?!", "NOT AGAIN!", "I'M ON MY BREAK!", "WHO RAISED YOU?!").
- Every line the game shows is also spoken. `say()` is the single entry point for bubble and TTS.

## Slap / physics behavior
- Slap(power 0.4-1.3, side -1..1 from the click's x offset): head angular impulse `-side*680*power`, body tilt/x nudge opposite the hit, squash `0.2*power`, hop if power > 0.9.
- Accumulated force (decays 1.4/s) > 7, power ≥ 1.25, or a combo of 20+ (every 10) triggers a knockdown: fly with spin, floor bounces (0.4 restitution), then lie down for 1.8 s with stars, then get up with a hop.
- Combo window 1.2 s. Milestones (configurable): 5 annoyed, 10 dizzy, 20 fall, 30 chaos.
- Chaos: 8 s, power ×1.5, shake, confetti. Calm mode: power ×0.35, no rage or chaos.
- Drag and release: pointer velocity from the last ~5 samples; speed over 1500 px/s knocks the character down. Tap without a drag (< 9 px) is a slap.
- Idle: blink, breathing, eyes look around, 30 s idle triggers the secret dance.
- Objects: foam hand 1.1, pillow 0.5, chicken 0.8, glove 1.4, tomato 0.7 power. Upgrade the prototype's click-to-throw to drag-and-throw with Matter bodies.

## Voice
- Recognition: `continuous`, `interimResults`, lang from an en-IN/en-US select. Auto-restart in `onend` while voice mode is on (350 ms delay). Clear timers and `abort()` on unmount.
- Ignore results while the character is speaking (avoids hearing itself). Dedupe finals: same normalized text within 2.5 s is dropped.
- Normalize masked profanity (`f***`, `s***`, etc.) before matching.
- Commands only when the whole utterance matches: "(hey) listen", "calm down", "go crazy/wild", "reset (game)", "stop listening". Anything else goes through the normal cuss pipeline.
- Truthful status: MIC OFF until `onstart`, then LISTENING, PROCESSING briefly after a final result. The waveform animates only between `onspeechstart` and `onspeechend`.
- Errors: not-allowed, service-not-allowed, audio-capture, network each show a specific message. Unsupported browsers get a "use Chrome or Edge" notice, and the text cuss box keeps working.
- Privacy notice: browser speech service may send audio online; the game records and stores nothing.
- TTS: voice select (English voices), rate and volume sliders, mute button. `cancel()` before each utterance and on slap. Pitch is higher and rate ×1.15 for ALL-CAPS lines. Say it's a system voice, not AI.

## Controls
Space slap, M mic, R reset, C chaos (ignored inside inputs, and Space is ignored on focused buttons). Buttons for sound, voice mute, random reaction, calm, chaos, fullscreen, reset, reset rage, and clear chat.

## Acceptance checks for Claude Code
Run `npm run build`, type-check and lint. Unit-test `rageEngine` (one message never reaches 100, decay works, meltdown fires) and `dialogueEngine` (no consecutive repeats, category matching inside sentences). Verify no horizontal scroll at 375 px, reduced-motion respected, and listeners and rAF cleaned up on unmount.
