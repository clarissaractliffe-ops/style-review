# Style Review

A fitting-room tool for LOEWE product development. During a fitting the team takes or uploads photos into ready-made pages that follow the LOEWE "Comments Edit" sheet, and dictates comments next to each photo. Afterwards they review and edit the finished deck, then export it to **PowerPoint** or **PDF** to share within 24 hours.

Works on iPad, iPhone/Android phones and laptops, in the browser. No install.

---

## What it does

**Pages follow the LOEWE "Comments Edit" sheet**
- A3 landscape pages with the sheet's header on every page: blue LOEWE bar with style code, style name and fitting stage, **COMMENTS EDIT**, date and time; a grey row with Line, Season, Pattern Maker and Modelist (highlighted); a sketch thumbnail on the right; "Page X of Y" at the bottom.
- Five page templates:
  - **Line-up**: front, back and side photos, with a column for Fabrics, Pattern, Garment sent, Relaunch and Shipment deadline
  - **Flats**: up to four technical sketches or colour flats
  - **Colorways**: two colorways, each with its fabric and trim spec box, and a headline box
  - **Comments · 3 photos**: a category tag (Fit, Neckline, Hood, Pockets, Trims, Finishing, Fabrics, Lining) with the sheet's colours, an optional note beside it, three photos and a comment box under each
  - **Comments · close-up**: category tag, one large photo, a detail photo and comment boxes
- A style usually runs over several pages (line-up, flats, colorways, then one page per category). Header details (style code, name, pattern maker, modelist, sketch) are shared by every page with the same style code, so you type them once.

**Capture (during the fitting)**
- **Add page** picks a template; the new page copies the current style's header.
- Every photo box has **Camera** (opens the device camera directly) and **Upload** (library or files; drag-and-drop on laptop). Photos are resized to 1600 px so files stay light.
- Every comment box has **Dictate**: live speech-to-text, editable while you talk. Language: English, Spanish, French, Italian.
- **Duplicate page** makes another page of the same template and category (e.g. a second FIT page) with empty photos and comments.
- Autosaves on the device (IndexedDB), so a refresh or a dropped connection loses nothing. Fittings saved with the earlier templates are moved onto the new ones automatically, keeping every photo and note.

**Review & edit (after the fitting)**
- Shows the actual pages, exactly as they'll export. Comment boxes size themselves to their text, like on the sheet.
- Tap any text to edit it in a side panel (dictation works there too); tap the header to change style details, category, line, season or date; tap any photo to replace it.
- Empty spaces show as dashed boxes on screen only. They never appear in the export.
- Reorder pages with up/down arrows. A "Check before sharing" list flags missing photos, missing style details and text that's too long to fit.

**Export**
- **PowerPoint (.pptx)**: A3 landscape, native editable text boxes, shapes and images. All comments are also copied into the speaker notes.
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
- Swap the generic slide styling for the official LOEWE master template (fonts, logo, footer). All page geometry is in `headerEls()` and the `LAYOUTS` object, in inches, shared by the screen and both exports.
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
One fitting = a list of pages. Each page has a template; header fields repeat on every page of the same style.
```json
{
  "line": "Men's Presentation",
  "season": "Spring / Summer 2027",
  "fitting": "Line up",
  "date": "2026-10-21", "time": "10:00",
  "lang": "en-GB",
  "slides": [{
    "id": "…", "template": "comments3",
    "code": "H526Y02WIH", "name": "Bomber jacket",
    "patternMaker": "", "modelist": "",
    "category": "fit",
    "photos": { "thumb": { "src": "data:image/jpeg…", "w": 800, "h": 800 }, "p1": { "src": "…", "w": 1200, "h": 1600 } },
    "notes": { "p1": "ADD 3 CM ON BODY LENGTH ALL AROUND", "headline": "" }
  }]
}
```
Templates: `lineup`, `flats`, `colorways`, `comments3`, `closeup` (defined in `TEMPLATES` in `app.html`).

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
