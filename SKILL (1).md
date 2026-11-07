---
name: blocky-3d-story-shorts
description: Build vertical (9:16) funny 3D "blocky character" story shorts entirely from code, in the style of "In This World, You Can Only Travel by Being Thrown". Use when asked to make a short, animated, narrated, word-by-word captioned comedy video with Roblox-like characters, fast cuts, whip-pan cameras, and an escalating absurd premise. Output is an MP4 rendered with Three.js (headless Chromium) or Blender (bpy), TTS narration, ASS captions and ffmpeg.
---

# Blocky 3D Story Short (all-code pipeline)

## 1. What the reference video is (analysis)

Format
- 720x1280 (9:16), 30 fps, H.264 + AAC, about 150 s long. Works as TikTok / Reels / Shorts.
- Single continuous voice-over narration (storyteller voice, fast, energetic). No silence gaps. Audio is voice-dominant (energy mostly 500-4000 Hz), mean about -17 dB, peaks about -1 dB. Treat sound effects (whoosh, thud, boom, crowd roar, confetti pop) as optional accents under the voice.
- No on-screen UI other than captions. No logos, no intro card, the first frame is already the hook scene.

Visual style
- Stylised 3D render that looks like Roblox / low-poly toy world, rendered with soft real-time-quality lighting (Blender Eevee-like or Three.js with soft shadows).
- Characters: blocky bodies. Head is a slightly rounded box (a bit oversized), torso is a box, arms and legs are separate boxes, simple flat-colour clothes. Faces are minimal: two dot eyes, brows, a mouth that changes shape (open scream, smile, deadpan). Accessories are simple geometry (round glasses, spiky hair blocks, cap, tracksuit stripes, lightning-bolt chest logo).
- Cast archetypes: a small hero kid (blue shirt with yellow lightning bolt, spiky brown hair, round glasses), Dad (orange/red polo, brown hair), Grandma (purple tracksuit, white hair, glasses), a huge muscular rival ("Titan Tom", black outfit, red billboard branding). Reuse the same few characters across the whole video.
- World: pastel town street (brick school, pink/blue/yellow shop fronts with awnings and readable signs: SUPERMARKET, BARBER, GYM), low-poly trees (flat green icosphere on a thin trunk), bright saturated blue sky with soft fluffy clouds, green grass, sand throwing field, huge stadium with a dense crowd made of thousands of tiny coloured blocky people, big billboards ("FIRST PRIZE $1 BILLION" on a purple board with gold text, "WORLD THROWING CHAMPIONSHIP" banners), confetti pieces falling.
- Escalation of scale is the visual joke: backyard, street, stadium, sky, clouds, space station, full planet Earth on black starfield, then the fall back.
- Palette: saturated primaries and pastels, high key light, light blue sky gradient, strong sun glare in some shots, black space for the climax.

Camera and editing
- Vertical framing, characters centred or lower third (leave the bottom 20% for captions).
- Very fast cutting: roughly 1 cut every 1.0-1.3 s (about 130+ cuts in 150 s). Each shot lasts 0.5-2 s. Almost no shot is static.
- Camera moves: low-angle wide FOV (60-90 deg) hero shots, whip pans, quick push-ins to faces for jokes, tracking/follow cam on a flying character with a long white motion streak behind it, top-down shots of the stadium, POV shots from the thrown character, slow orbit around Earth/ISS for the epic beat.
- Motion blur and radial speed lines on throws. Camera shake (2-6 frames) on impacts. Dust puff (white sprite particles) on landings. Confetti particles in stadium shots.
- Comedy timing: setup shot, reaction close-up (big eyes, open mouth), payoff wide shot.

Captions
- One word at a time, synced to the narration (about 3-4 caption changes per second).
- Position: horizontally centred, baseline around y = 1040-1110 of 1280 (about 82-86% down the frame).
- Style: ALL CAPS, very heavy rounded display font, white fill (#FFFFFF), thick black outline (about 8-10 px at 720 wide) plus a subtle black drop shadow, cap height about 50 px. A small "pop" scale animation (110% to 100% over about 3 frames) on each new word is welcome.
- Emphasis words may be tinted (yellow for money, red for danger) but default is white.

Story structure (copy this skeleton, not the content)
1. Hook line in the first second that states a weird rule of the world ("In this world, you can only travel by being thrown").
2. Quick world tour showing the rule applies to everyone (school, supermarket, street) for 10-15 s.
3. A competition or social structure built around the rule (World Throwing Championship, VIP throws, $1 billion prize).
4. Introduce the small underdog hero + family (Dad, Grandma) + giant rival.
5. Escalating throws: each one goes further than the last (street, sky, clouds, orbit).
6. Climax at maximum scale (space station, Earth) with a joke about how far it went.
7. Return / fall back toward the stadium with panic screaming, crowd shots, family reaction.
8. Short punchline or open ending.
Narration is third person, simple present or past tense, short sentences, absurd facts delivered deadpan.

## 2. Tech stack (everything is code)

Preferred (fast, no GPU needed): Three.js scene driven by Playwright/Puppeteer headless Chromium, deterministic frame stepping, screenshot every frame, ffmpeg to encode.
Alternative (prettier, slower): Blender headless with `bpy` (Eevee, 720x1280). Use if Blender is installed. The same shot list and timing data applies.

Tools
- Python 3.10+, `playwright` (or Node + `puppeteer`), `ffmpeg`, `edge-tts` (neural TTS with word timestamps; fallback `piper-tts` or `espeak-ng` + forced alignment), `Pillow`, `numpy`.
- Fonts: use a heavy rounded display font, in order of preference: "Luckiest Guy", "Lilita One", "Baloo 2 ExtraBold", "Nunito Black", "Montserrat Black". Download the TTF once and reference it locally.

Project layout
```
project/
  script.txt           # narration text
  shots.json           # shot list (see section 4)
  build_audio.py       # TTS + word timings -> narration.mp3, words.json
  scene/index.html     # three.js scene, exposes window.renderFrame(t, shotIndex)
  scene/characters.js  # blocky character factory
  scene/world.js       # buildings, trees, stadium, crowd, sky, space
  render.py            # steps frames headless -> frames/%05d.png
  captions.py          # words.json -> captions.ass
  assemble.sh          # ffmpeg: frames + narration + sfx + captions -> out.mp4
```

## 3. Workflow (follow in order)

1. Write the script (about 330-420 words for 150 s; narration runs at roughly 2.5-3 words per second). Split into beats of 1-2 sentences.
2. Generate narration with word-level timing (section 5). The audio length defines the video length. Never time video first.
3. Create `shots.json`: for each beat, 3-6 shots of 0.5-2 s. Every shot has a camera move, a set, characters and an action. Total shot durations must equal the audio duration.
4. Build the scene and characters (section 6), then each shot as a function `shot_N(t)` with `t` from 0 to duration. Everything is a pure function of time (no real-time clock), so rendering is deterministic.
5. Render frames headless at 720x1280, 30 fps.
6. Make captions (section 7), then assemble (section 8).
7. Check: extract a contact sheet with ffmpeg and look for dull static shots, caption collisions, and shots with no motion. Fix and re-render only the changed shots (cache frames per shot).

## 4. shots.json format

```json
[
  {
    "id": 1, "start": 0.00, "dur": 1.6, "set": "street",
    "camera": {"type": "dolly_low", "from": [0,1.2,8], "to": [0,1.6,3], "look": [0,1.5,0], "fov": 80},
    "actors": [{"who": "kevin", "pos": [0,0,0], "anim": "idle_wave"}],
    "fx": ["clouds_drift"], "shake": 0.0, "sfx": null
  },
  {
    "id": 2, "start": 1.60, "dur": 1.0, "set": "street",
    "camera": {"type": "follow", "target": "kevin", "offset": [0,0.5,-4], "fov": 85},
    "actors": [{"who": "kevin", "anim": "thrown_arc", "from": [0,1,0], "to": [0,30,60]}],
    "fx": ["motion_streak", "speed_lines"], "shake": 0.3, "sfx": "whoosh"
  }
]
```
Camera types to implement: `dolly_low`, `whip_pan`, `follow`, `orbit`, `topdown`, `pov`, `push_in_face`, `static_wide` (use rarely).

## 5. Narration with word timestamps (edge-tts)

```python
# build_audio.py
import asyncio, json, edge_tts

VOICE = "en-US-GuyNeural"      # energetic male storyteller; also try en-US-ChristopherNeural, en-US-AriaNeural
RATE  = "+12%"                 # a bit fast, like the reference
TEXT  = open("script.txt", encoding="utf-8").read()

async def main():
    comm = edge_tts.Communicate(TEXT, VOICE, rate=RATE, boundary="WordBoundary")
    words, audio = [], bytearray()
    async for ch in comm.stream():
        if ch["type"] == "audio":
            audio += ch["data"]
        elif ch["type"] == "WordBoundary":
            words.append({
                "word": ch["text"],
                "start": ch["offset"] / 1e7,                       # 100-ns ticks -> s
                "end": (ch["offset"] + ch["duration"]) / 1e7,
            })
    open("narration.mp3", "wb").write(audio)
    json.dump(words, open("words.json", "w"), indent=1)

asyncio.run(main())
```
If edge-tts is unavailable (no network): use `piper` or `espeak-ng` to make a WAV, then compute word timings by proportional split on character count across detected speech segments (`ffmpeg -af silencedetect`). It is less exact but fine.

## 6. The 3D look (Three.js)

Renderer
- `WebGLRenderer({antialias:true, preserveDrawingBuffer:true})`, size 720x1280, pixel ratio 1, `toneMapping = ACESFilmicToneMapping`, `outputColorSpace = SRGBColorSpace`, soft shadows (`PCFSoftShadowMap`).
- Lights: one warm directional sun (castShadow, shadow map 2048) + `HemisphereLight(sky 0xbfe3ff, ground 0x9ccc65, 0.9)`. Keep the image bright and saturated.
- Sky: large gradient sphere (top #3a7bd5, horizon #cfe9ff). Clouds: 15-30 soft white billboards (canvas-generated radial-gradient puffs) or grouped white spheres with `MeshStandardMaterial({color:0xffffff, roughness:1})`, drifting slowly.
- Space shot: black background, 2000 star points, Earth = sphere with a procedural blue/green canvas texture + a slightly larger transparent light-blue atmosphere sphere + rim light. ISS = grey boxes with 2 blue solar-panel planes.
- Post: radial blur / speed lines as a full-screen quad (shader or overlay PNG drawn on canvas) enabled only during throws; subtle vignette; optional 2-3 frame motion blur by accumulating sub-frames (render 3 sub-frames at t, t+dt/3, t+2dt/3 and average) for fast shots.

Blocky character factory (reuse for every character)
```js
function makeBlocky({skin=0xf1c27d, shirt=0x3b82f6, pants=0x1d4ed8, hair=0x6b3f1d, scale=1, glasses=false, hairStyle='spiky'}) {
  const g = new THREE.Group();
  const mat = c => new THREE.MeshStandardMaterial({color:c, roughness:0.7});
  const box = (w,h,d,c)=> { const m=new THREE.Mesh(new THREE.BoxGeometry(w,h,d,2,2,2), mat(c)); m.castShadow=true; return m; };
  const torso = box(1.0,1.2,0.5,shirt); torso.position.y=1.7;
  const head  = box(0.9,0.9,0.9,skin);  head.position.y=2.75;
  const armL=box(0.35,1.1,0.35,skin), armR=box(0.35,1.1,0.35,skin);
  const legL=box(0.45,1.1,0.45,pants), legR=box(0.45,1.1,0.45,pants);
  // pivots at shoulder/hip so rotation swings from the joint
  const pivot=(m,x,y,dy)=>{const p=new THREE.Group();p.position.set(x,y,0);m.position.y=-dy;p.add(m);return p;};
  const aL=pivot(armL,-0.68,2.2,0.5), aR=pivot(armR,0.68,2.2,0.5);
  const lL=pivot(legL,-0.25,1.1,0.55), lR=pivot(legR,0.25,1.1,0.55);
  // face: eyes (small black boxes), brows, mouth (scaleY animates open/close)
  const eye=()=>box(0.1,0.14,0.02,0x111111);
  const eL=eye(), eR=eye(); eL.position.set(-0.2,2.8,0.46); eR.position.set(0.2,2.8,0.46);
  const mouth=box(0.3,0.08,0.02,0x5a1a1a); mouth.position.set(0,2.5,0.46);
  g.add(torso,head,aL,aR,lL,lR,eL,eR,mouth);
  g.userData={aL,aR,lL,lR,mouth,head,torso};
  g.scale.setScalar(scale);
  return g;
}
```
Add hair (several small boxes on top, tilted for the spiky look), glasses (two thin `TorusGeometry` rings), chest logo (yellow lightning shape as a flat `ShapeGeometry`), cap, tracksuit stripes as needed. Muscular rival: scale arms x2.2 and torso x1.4.

Animation vocabulary (all functions of time)
- `idle`: torso bob sin(t*3)*0.03, arms swing slightly.
- `wave`: one arm rotates 140 deg and oscillates.
- `walk`: legs and arms opposite-phase swing at 6 rad/s.
- `wind_up_throw` (for the thrower): spin the body 360-720 deg over 0.6 s, then release.
- `thrown_arc`: ballistic path (position = lerp + parabola), body tumbling with all limbs flailing (random sinusoids), mouth wide open.
- `scream`: mouth scaleY 3, eyes scaled 1.4, body shake.
- `land_roll`: bounce twice with decreasing height, dust sprites at each contact.
- `crowd_cheer`: instanced boxes bob with random phase.

Crowd: use `InstancedMesh` of 3000-8000 boxes in stadium stands, random colours from a bright palette, per-instance bobbing in the vertex shader or by updating matrices every frame.

World set pieces to build (procedural boxes, no assets): school (brick box + red SCHOOL sign), 3-4 row-house shop fronts in pastel colours with striped awnings and canvas-texture signs, low-poly trees, house with porch, fence, sand throwing pit, stadium bowl (cylinder segments + stands + big field with white circle markings), billboards (canvas texture with text), goal-like throwing ring, confetti (200 small coloured planes with gravity and spin).

Signs and billboards: draw text onto a `CanvasTexture` using the same heavy font, bold outlines, gold gradient for money text.

## 7. Captions (ASS, one word at a time, burned in)

```python
# captions.py
import json
W = json.load(open("words.json"))
H = """[Script Info]
ScriptType: v4.00+
PlayResX: 720
PlayResY: 1280
WrapStyle: 2

[V4+ Styles]
Format: Name,Fontname,Fontsize,PrimaryColour,SecondaryColour,OutlineColour,BackColour,Bold,Italic,Underline,StrikeOut,ScaleX,ScaleY,Spacing,Angle,BorderStyle,Outline,Shadow,Alignment,MarginL,MarginR,MarginV,Encoding
Style: Word,Luckiest Guy,74,&H00FFFFFF,&H00FFFFFF,&H00000000,&H80000000,0,0,0,0,100,100,0,0,1,9,3,2,40,40,170,1

[Events]
Format: Layer,Start,End,Style,Name,MarginL,MarginR,MarginV,Effect,Text
"""
def t(s):
    h=int(s//3600); m=int(s%3600//60); sec=s%60
    return f"{h}:{m:02d}:{sec:05.2f}"
lines=[]
for i,w in enumerate(W):
    start=w["start"]
    end=W[i+1]["start"] if i+1<len(W) else w["end"]     # hold until next word, no flicker
    txt=w["word"].upper().strip(".,!?;:\"")
    pop=r"{\fscx112\fscy112\t(0,100,\fscx100\fscy100)}"   # pop-in
    lines.append(f"Dialogue: 0,{t(start)},{t(end)},Word,,0,0,0,,{pop}{txt}")
open("captions.ass","w",encoding="utf-8").write(H+"\n".join(lines))
```
Alignment 2 + MarginV 170 puts the baseline about 85% down the frame like the reference. Make sure the font is installed (`fc-list | grep -i luckiest`) or pass `fontsdir=` in the ffmpeg filter.

## 8. Render and assemble

Frame capture (Python + Playwright)
```python
# render.py
import asyncio, json
from playwright.async_api import async_playwright
FPS, W, H = 30, 720, 1280
DUR = float(open("duration.txt").read())          # seconds, from narration.mp3 (ffprobe)
async def main():
    async with async_playwright() as p:
        b = await p.chromium.launch(args=["--use-gl=swiftshader","--enable-webgl","--ignore-gpu-blocklist"])
        pg = await b.new_page(viewport={"width":W,"height":H})
        await pg.goto("file:///abs/path/scene/index.html")
        await pg.wait_for_function("window.sceneReady===true")
        n = int(DUR*FPS)
        for i in range(n):
            await pg.evaluate(f"window.renderFrame({i/FPS})")
            await pg.screenshot(path=f"frames/{i:05d}.png", clip={"x":0,"y":0,"width":W,"height":H})
        await b.close()
asyncio.run(main())
```
Tip: render shots in parallel processes (split the frame range) to cut render time. Cache frames per shot so edits re-render only what changed.

ffmpeg
```bash
# duration.txt first:  ffprobe -v error -show_entries format=duration -of csv=p=0 narration.mp3 > duration.txt
ffmpeg -y -framerate 30 -i frames/%05d.png -i narration.mp3 \
  -vf "ass=captions.ass:fontsdir=./fonts,format=yuv420p" \
  -c:v libx264 -preset slow -crf 18 -r 30 -c:a aac -b:a 192k -ar 48000 -shortest out.mp4
```
Optional SFX bed: generate whooshes, thuds and booms with numpy (filtered noise sweeps, sine drops), place them at shot starts from `shots.json` (`sfx` field), mix at about -18 dB under the voice with `amix`/`adelay`, and apply `loudnorm` to the final mix. Optional very low-volume light background music; the reference is voice-led.

## 9. Quality checklist (verify before delivering)

- Resolution 720x1280 (or 1080x1920), 30 fps, H.264 yuv420p, AAC 48 kHz, plays on phones.
- First frame already shows the hook scene; first spoken words within 0.3 s.
- Average shot length between 0.8 and 1.6 s; no shot over 2.5 s; every shot has camera motion.
- Captions: one word at a time, uppercase, white with thick black outline, centred near the lower third, never covering the main character's face, in sync within 80 ms of the voice.
- Escalating scale joke is visible (small world, then huge, then planet scale), with a deadpan narration line on the biggest beat.
- Same 3-5 recurring characters, same colours and accessories throughout.
- Bright, saturated image, soft shadows, sky always visible in outdoor shots.
- Export a contact sheet (`ffmpeg -i out.mp4 -vf "fps=1/5,scale=240:-1,tile=6x5" sheet.png`) and compare with the style described in section 1.

## 10. Adapting to a new video idea

Keep: format, character look, cut rhythm, captions, escalation structure, narration tone.
Change: the one weird rule (e.g. "In this world you can only travel by being launched from a cannon / sliding down a slide / being flushed"), the cast names, the competition, the prize, the final location, the punchline. Write the new script first, then re-do the shot list. Never copy the original's exact lines, characters' names or signs; the style is the template, the story must be original.
