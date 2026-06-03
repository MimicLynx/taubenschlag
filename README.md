# Taubenschlag

Taubenschlag is a portable single-file HTML tool for creating in-game LARP correspondence and saving it as a DIN A4 PDF through the browser print dialog.

The current version includes letter, newspaper, and wanted-poster templates. Shared fields relabel themselves per template, template-specific controls stay hidden until relevant, layout variants change with the selected template, each template has matching sample content instead of one letter pretending to be every document type, and default font pairs now match the selected document type.

## Use it

Open this file in a browser:

```text
index.html
```

Then:

1. Choose the interface language. German is the default; English and Spanish are also available.
2. Choose the template:
   - letter
   - newspaper
   - wanted poster
3. Fill out the template form. Labels adapt to the selected template.
   - If the current content is still the built-in sample, switching templates swaps in the matching sample text.
   - If you have edited the fields, switching templates preserves your text instead of helpfully vaporizing it.
4. Pick a background, title font, text font, template-specific layout variant, line spacing, and page margins.
   - Letters default to handwriting-style fonts.
   - Newspapers and wanted posters default to print/display faces, not handwriting.
   - If you manually change a font, switching templates keeps your chosen font instead of replacing it with a new default.
5. Optionally upload:
   - a borderless page background image
   - a transparent PNG signature
   - a transparent PNG seal or stamp
   - a portrait or crest for wanted posters
   - an article image for newspapers, placed at the beginning, middle, or end of the article flow
6. Optionally export or import a draft as JSON.
7. Click `Print / save PDF`.
8. In the browser print dialog:
   - choose A4
   - save as PDF
   - enable background graphics if your browser requires it
   - use no margins when available

## Portability

The end-user tool is `index.html`.

It has:

- inline CSS
- inline JavaScript
- no external scripts
- no external stylesheets
- no network requirement
- no server requirement

Uploaded images are read locally in the browser session only. They are not uploaded anywhere. Refreshing the page clears them. Printable in-document images are rendered in black and white; background uploads remain page backgrounds.

## Security model

Taubenschlag is a static browser app. There is no upload endpoint, no server-side storage, and no executable web root. A `.php`, `.jsp`, or similar file selected in the browser cannot become a webshell in this architecture, because there is nothing server-side to execute it. That changes immediately if a backend is added later; then uploads must be revalidated server-side and stored outside any executable path. Reality enjoys changing the threat model when people bolt on features.

Current client-side protections:

- Text field content is rendered with text APIs (`textContent` / text nodes), not HTML injection sinks.
- Text inputs and textareas have `maxlength` limits to reduce accidental browser-side resource abuse.
- Draft import accepts a small JSON schema only: fixed app/version envelope, whitelisted keys, enum checks, numeric range checks, and text length limits. Imported images are ignored.
- The document includes a restrictive static-app CSP and `no-referrer` policy: no network connections, forms, objects, frames, workers, media, or base URI.
- Image uploads are limited to PNG, JPEG, and WebP raster images.
- SVG, HTML, XML, scripts, unknown types, and disguised files are rejected.
- Uploads are capped at 8 MB before the browser reads them as Data URLs.
- File signatures are checked before a file is used as an image.

If this is hosted publicly, prefer sending the same CSP as an HTTP response header in addition to the meta tag. If a backend upload/storage feature is ever added, client-side validation is not enough: validate again server-side, generate safe filenames, store files outside executable paths, serve with safe content headers, and never let uploaded bytes share a path with runnable code. The blast door is effective only while nobody installs a smaller door next to it.

Recommended static-hosting headers:

```text
Content-Security-Policy: default-src 'none'; script-src 'unsafe-inline'; style-src 'unsafe-inline'; img-src data: blob:; font-src data:; connect-src 'none'; form-action 'none'; object-src 'none'; base-uri 'none'; frame-src 'none'; media-src 'none'; worker-src 'none'; manifest-src 'none'
Referrer-Policy: no-referrer
X-Content-Type-Options: nosniff
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), interest-cohort=()
```

The CSP still allows inline script/style because the deliverable is intentionally one self-contained HTML file. If the project stops being single-file later, move script and CSS into separate files and replace `'unsafe-inline'` with hashes or nonces. Elegance is optional. Not weakening the security boundary is less optional.

## Implemented features

- DIN A4 portrait preview
- Print/save-to-PDF button using `window.print()`
- Interface language dropdown:
  - English
  - German
  - Spanish
- Template selector:
  - Letter
  - Newspaper
  - Wanted poster
- Shared content fields and optional upload/control groups relabel or hide per template so irrelevant controls are not shown in the selected mode.
- Template-specific sample content for letters, newspapers, and wanted posters. Sample text updates on template switch while built-in samples are untouched; edited user content is preserved.
- Background presets:
  - Old paper - honey
  - Old paper - stained ochre
  - Old paper - rosy vellum
  - Old paper - blue-grey rag
  - Structure overlay - light stains
  - Structure overlay - heavy stains
- Backgrounds are split between fully colored old paper and mostly untinted structural/stain overlays.
- Uploaded borderless background support
- Draft export/import for text/settings as a bounded JSON file; uploaded images remain session-only and are not included
- Signature image upload
- Seal/stamp image upload
- Seal/stamp position, size, and opacity controls
- Handwritten or medieval-print-style font controls only, with separate title and text font selectors instead of hardcoded template typography. Embedded options include Fleur De Leah, Hurricane, Ruthie, Island Moments, Jacquard 24, Uncial Antiqua, UnifrakturMaguntia, New Rocker, UnifrakturCook, Manufacturing Consent, Almendra Display, Splash, Estonia, Monsieur La Doulaise, Pirata One, and Astloch.
- Template-aware font defaults:
  - Letter: Hurricane title, Fleur De Leah text
  - Newspaper: UnifrakturMaguntia title, Uncial Antiqua text
  - Wanted poster: Manufacturing Consent title, New Rocker text
- Template switching updates font defaults only while the current font still matches the old template default. Manually selected fonts are preserved.
- Template-specific layout variants:
  - Letter: Formal letter, Compact dispatch, Official order, Personal note
  - Newspaper: Broadsheet, Gazette sheet, Public notice
    - Broadsheet keeps the classic two-column paper layout.
    - Gazette sheet is tighter and more compact, with three columns and a smaller masthead.
    - Public notice becomes a larger one-column announcement with centered text.
  - Wanted poster: Official warrant, Rough broadside, Reward poster
    - Official warrant keeps the clean framed warrant style.
    - Rough broadside uses a dashed, left-aligned, slightly irregular notice style.
    - Reward poster emphasizes the title and reward block with stronger framing.
- Newspaper article image upload with placement inside the article text flow:
  - beginning
  - middle
  - end
- Printable in-document image fields render black and white by default
- Line spacing and page margin controls
- Historical ornament dividers for subject/header separation instead of plain modern lines
- Overflow warning when the letter content no longer fits on one A4 page
- Preview text avoids fixed wording such as an uneditable subject prefix; if you want a label like `Betreff:`, put it in the subject field yourself. Apparently users enjoy controlling their own letters. Radical.

## Template behavior

The form uses one shared data model and maps it into each printable layout:

- Letter:
  - `From`, `To`, `Location`, `Date`, `Subject`, `Text`, `Closing`, `Signature name/title`, `Postscript / notes`
- Newspaper:
  - `Newspaper title`, `Location`, `Date`, `Headline`, `Article text`, `Footer / note`
  - optional article image inside the article columns
- Wanted poster:
  - `Location`, `Date`, `Wanted name / title`, `Description / accusation`, `Authority line`, `Responsible name/title`, `Reward / note`

## Planned templates

None currently. Add new templates only when they render, print, and pass validation. Disabled fantasies remain fantasies, even with a dropdown.


Current checks verify that `index.html` remains portable and includes the expected A4, language, template, print, historical font/background, editable-preview, and upload-security behavior.

## Files

```text
index.html                 Portable end-user tool
README.md                  Usage and project notes
```
