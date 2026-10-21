# Style Review

A fitting-room tool for LOEWE product development. During a fitting the team takes or uploads photos into ready-made slide templates and dictates comments next to each photo. Afterwards they review and edit the finished deck, then export it to **PowerPoint** or **PDF** to share within 24 hours.

Works on iPad, iPhone/Android phones and laptops, in the browser. No install.

---

## What it does

**Capture (during the fitting)**
- One "look" per slide, numbered like the line-up (Look 04, 07, 11…), with style name, style code, sample stage (Proto 1/2/3, SMS, PP sample, Production), fit model/size and status (To review, Approved, Approved with comments, Re-proto, Dropped).
- Four slide templates:
  - **Full look**: front, side, back
  - **Detail focus**: full look + two close-ups
  - **Proto vs proto**: previous proto next to this one
  - **Single garment**: one photo, room for long notes
- Every photo box has **Camera** (opens the device camera directly) and **Upload** (library or files; drag-and-drop on laptop). Photos are resized to 1600 px so decks stay light.
- Next to every photo is a notes box with **Dictate**: live speech-to-text, editable while you talk. Language: English, Spanish, French, Italian.
- Every look also has **Overall comments** and **Actions** boxes.
- **Duplicate look** creates the next proto round of the same style with empty photos and notes (for re-fits).
- Autosaves on the device (IndexedDB), so a refresh or a dropped connection loses nothing.

**Review & edit (after the fitting)**
- Shows the actual slides, exactly as they'll export, including an auto-generated cover with a status summary.
- Tap any text to edit it in a side panel (dictation works there too); tap any photo to replace it. Slides re-fit text live and flag boxes that are too long.
- Reorder looks with up/down arrows.
- A "Check before sharing" list flags missing photos, missing style names, looks still "To review" and overflowing text.

**Export**
- **PowerPoint (.pptx)**: native, editable text boxes and images, 16:9. All notes are also copied into the speaker notes.
- **PDF**: same layout, for people who just need to read it.
- Both are generated from one layout definition, so the PPTX and PDF always match the Review screen.
- On iPad/iPhone the file goes to the share sheet (Mail, Teams, OneDrive, Files…); on laptops it downloads.

---

## Run it

It's one file: `index.html`. No build step needed to run.

**Locally:** open `index.html` in Chrome, Edge or Safari. Dictation needs a secure origin, so for voice use `npx serve .` (or any static server) and open `http://localhost:3000`.

**GitHub Pages (prototype hosting):**
1. Create a repo (e.g. `style-review`), add `index.html` and this README.
2. Settings → Pages → Deploy from branch → `main` / root.
3. Open `https://<user>.github.io/style-review/` on the iPad and tap Share → **Add to Home Screen** for an app-like icon.

> Use the GitHub prototype with **example or non-confidential samples only**. Unreleased collection photos should not go through a personal GitHub account; that's what the LVMH-hosted version is for.

**Editing the source:** edit `app.html`, then run `./build.sh` to regenerate `index.html`. (`app.html` is the page body; `build.sh` wraps it in a full HTML document.)

---

## Browser support

| | Photos | Dictation | Export |
|---|---|---|---|
| Safari, iPad / iPhone (iOS 16+) | ✓ | ✓ | ✓ (share sheet) |
| Chrome / Edge, laptop | ✓ | ✓ | ✓ |
| Chrome, Android | ✓ | ✓ | ✓ |
| Firefox | ✓ | ✗ (type instead) | ✓ |

---

## Handover notes for IT: moving to LVMH hosting

The prototype is deliberately simple: a static front end, no server. Everything below is what changes for production. Items marked **decision** need IT/security input.

### 1. Speech-to-text (**decision**, most important)
The prototype uses the browser's built-in Web Speech API. It's free and fast, but **audio is processed by Apple (Safari) or Google (Chrome) cloud services**, outside LVMH control. For unreleased collections this will likely not pass a security review.

Options for production:
- **Cloud speech service under the group's own tenant** (e.g. Azure AI Speech, Google Cloud Speech-to-Text or AWS Transcribe, whichever the group already contracts). Supports streaming, custom vocabularies and speaker labels.
- **Self-hosted Whisper** (or similar open model) on internal GPU infrastructure, if audio must not leave the network.

Either way: add a **custom vocabulary** for fashion/PD terms (proto, SMS, toile, yoke, placket, gauge, 7gg, double-face, bias-cut, nappa…) and style codes. This is the biggest accuracy gain.

The swap is isolated in one place: the `toggleDictation` / `startRec` functions. Replace them with a client that streams microphone audio (via `getUserMedia` + `MediaRecorder` or `AudioWorklet`) to the backend and writes transcripts back the same way (`onText(text)`).

### 2. Sign-in and storage (**decision**)
- Single sign-on through the group identity provider (Entra ID / Okta / whatever IT uses).
- Store sessions centrally instead of only on the device, so a fitting started on the iPad can be reviewed on the laptop and several people can contribute. Suggested model: one **session** (season, fitting, date, attendees) → many **looks** → photos stored as files (SharePoint/OneDrive or object storage), notes as JSON.
- Keep the on-device autosave as an offline cache: fitting rooms often have poor Wi-Fi.

### 3. Export
- Swap the generic slide styling for the official LOEWE master template (fonts, logo, footer). All slide geometry is in `layoutLook()` / `layoutCover()`, in inches, shared by both exports.
- Option: upload the export straight to the team's SharePoint/Teams folder instead of downloading.

### 4. Branding
- The LOEWE logo is embedded in the page (header, cover slide, slide footers) from the file supplied by the team. Ask the brand team for a vector (SVG) or high-resolution version and swap the `LOGO` constants in `app.html`.
- The official LOEWE typeface is proprietary and isn't on public font services. The app uses **Cormorant Garamond** as a stand-in. To use the real font, add its web font files and one rule; everything picks it up automatically:
  ```css
  @font-face { font-family: "LOEWE Display"; src: url("fonts/loewe-display.woff2") format("woff2"); }
  ```
- Exported slides use Georgia/Arial so they look right on any computer. The official master template (point 3) is where the brand font belongs in PowerPoint.

### 5. Libraries
Two open-source libraries are loaded from public CDNs on first export: **PptxGenJS 3.12.0** (MIT) and **jsPDF 2.5.1** (MIT). For production, vendor them into the repo so nothing loads from outside the network.

### 6. Data model (current, JSON)
```json
{
  "season": "SS27 Women's Ready-to-Wear",
  "fitting": "Fitting 3 · Proto review",
  "date": "2026-10-21", "time": "10:00",
  "attendees": "Design, Product Development, Pattern, Merchandising",
  "lang": "en-GB",
  "slides": [{
    "id": "…", "template": "look3", "look": "04",
    "code": "W27-CT-0412", "name": "Double-face wool coat",
    "proto": "Proto 2", "model": "Size 36", "status": "comments",
    "photos": { "front": { "src": "data:image/jpeg…", "w": 1200, "h": 1600 } },
    "notes": { "front": "…", "side": "…", "back": "…", "overall": "…", "actions": "…" }
  }]
}
```

---

## Ideas for phase 2 (to validate with the team)

- **Draft actions with AI**: turn the rambling dictated notes into a clean action list by owner (Design / Pattern / Product Dev / Factory / Merch), keeping every measurement. A working version of this button already appears in the Claude preview; production would call the group's approved LLM endpoint.
- **Speaker labels**: "who said it" on each comment (Design vs. Pattern vs. Merch).
- **Measurement capture**: pull "add 4 cm", "reduce 0.5 cm each side" into a structured table that can feed the spec sheet / tech pack.
- **PLM link**: look up the style by code and pre-fill name, colour and previous proto photos; push actions back to the PLM.
- **Voice commands**: "next look", "take front photo", "mark re-proto" so hands stay on the garment.
- **Hands-free mode**: one person shoots on the iPad, another dictates on a phone into the same live session.
- **Annotate photos**: draw arrows/circles on a photo (e.g. where the hem dips).

### Questions for the merch / PD team
1. Do the four templates match how you shoot today? Any template missing (e.g. accessories, leather goods, colour-way comparison)?
2. Who owns the deck after the fitting, and who edits it before it's sent?
3. What happens to the actions afterwards (email, Excel tracker, PLM)? That's where most of the remaining time goes.
4. Which languages are spoken in fittings, and do people mix them?
5. Is there an official fitting-deck template we must match?
