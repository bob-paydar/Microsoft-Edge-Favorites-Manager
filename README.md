# Edge Favorites Manager — Audit Friendly (ES5)

A single-file HTML app to **view, clean, edit, and export** Microsoft Edge / Chromium favorites (bookmarks).  
It is **ES5-only** (no `let/const`, no arrow functions, no promises), uses **triple-wired event handlers** for broad browser compatibility, and is designed to be **audit-friendly & accessible**.

---

## Features

- **Open HTML**: Load a Netscape-format bookmarks HTML using the file picker or drag-and-drop.
- **Load Demo**: Populate with demo data for quick testing.
- **Search**: Filter by title or URL (instant client-side filter).
- **Select links**: Per-row checkboxes + “Select All / Clear” controls.
- **Edit link (dialog)**: Change title, URL, folder path, add-date (UNIX), and icon (data URL or URL).
- **Add Folder**: Create any nested path (e.g., `Favorites bar/Work/2025`).
- **Add Link**: Insert a link into the active folder or any path you provide.
- **Remove Duplicates**: Deduplicate by normalized URL (case-insensitive).
- **Delete Selected Links**: Remove the currently checked rows.
- **Rename Folder**: Rename the active folder (updates descendants and link paths safely).
- **Delete Folder**: Delete the active folder (and all descendants).
- **Delete Checked Folders**: Multi-select folders (checkboxes in the left list) and delete them in bulk.
- **Save As…**: Export **generic Netscape bookmarks HTML** (works in most browsers).
- **Save As (Edge)**: Export **Edge-friendly** HTML with:
  - `Favorites bar` declared once at top with `PERSONAL_TOOLBAR_FOLDER="true"`
  - `Other favorites` section
  - Links at the root of your data are emitted at the **root** of the exported file (not nested under an extra folder)
  - CRLF line breaks to mimic typical exports

Everything runs locally in your browser—no servers and no external libraries.

---

## How to Use

1. **Open the app** (double-click the HTML file).
2. Click **📂 Open HTML** and choose your exported favorites HTML *or* drag the file onto the **Drop zone** on the left.
3. Browse folders in the left panel. The right table shows links for the active folder.
4. Use **Search** to filter. Use **Select All / Clear** to manage selection quickly.
5. **Edit** a link with the “Edit” button in the row. Save or cancel in the dialog.
6. **Add Folder / Add Link** to build out your collection.
7. **Remove Duplicates** to dedupe by URL (case-insensitive).
8. **Delete Selected Links** to remove checked rows in the table.
9. **Rename Folder**, **Delete Folder**, or **Delete Checked Folders** for folder management.
10. Export via **💾 Save As…** (generic) or **💾 Save As (Edge)** for importing back into Edge.

> Tip: The log pane (bottom-right) shows what the app is doing.

---

## Importing into Microsoft Edge

Use the **Save As (Edge)** export when you plan to import via **Edge → Settings → Profiles → Import browser data → Favorites or bookmarks HTML file**.

This exporter:
- Writes a single **`Favorites bar`** section at the top with `PERSONAL_TOOLBAR_FOLDER="true"` and timestamps.
- Emits **root links** at the **root** (not under a duplicate “Favorites bar”).  
- Groups the rest under **`Other favorites`**, preserving your nested folders.
- Uses UTF-8 and CRLF line endings.
  
If Edge rejects the file:
- Make sure the HTML begins with `<!DOCTYPE NETSCAPE-Bookmark-file-1>`.
- Ensure the first `<DL><p>` contains the top-level structure as in the generated file.
- Try importing the **Edge** variant rather than the **generic** one.
- If you edited the exported HTML by hand, check for broken tag order: `DL/DT/H3 + DL` pairs must be correct.

---

## Accessibility & Audit Notes

- **Keyboard support**: All buttons respond to **Enter/Space**; dialog closes with **Esc**; folder list items are focusable.
- **ARIA**: Buttons have `aria-label`s; folder tree uses `role="tree"`/`treeitem` and label.
- **Visible focus**: Active folder is outlined and indicated with `aria-selected="true"`.
- **No inline styles** for controls; styles are in the document `<style>` block (audit-friendly single file).
- **UTF-8** meta headers set (`<meta charset="UTF-8">` and HTTP-equivalent).

---

## Browser Compatibility

- Microsoft Edge (Chromium) ✅
- Chrome ✅
- IE11 / Legacy modes: The app is ES5-only and includes **triple-wired** events (`addEventListener` / `attachEvent` / `on*`).  
  Notes:
  - The edit dialog uses a **backdrop** that becomes visible via `.modal-backdrop.show { display: flex; }`  
    As a fallback, JS also sets `style.display='flex'` when opening (and `'none'` when closing).

---

## Data Model

The app stores a flattened list of bookmarks plus a folder graph.

```js
{
  flat: [ { id, title, url, folder, addDate, icon } ],
  folders: {
    "": { name: "(root)", path:"", parent:null, childrenFolders:[] },
    "Favorites bar": { name:"Favorites bar", path:"Favorites bar", parent:"", childrenFolders:[...] },
    ...
  },
  itemsByFolder: {
    "": ["id1","id2",...],
    "Favorites bar": ["id3", ...]
  },
  activeFolder: "Favorites bar",
  selection: { "id3": true, ... },
  folderChecks: { "Work/2025": true }
}
```

Data persists in `localStorage` under the key **`efmData`**.

---

## File Format (Netscape Bookmarks)

The app both parses and emits the classic Netscape bookmarks format used by many browsers.

- Folders: `DT > H3` followed by a sibling `DL > p` list.
- Links: `DT > A` with optional attributes `HREF`, `ADD_DATE`, `ICON`, etc.
- Important attributes for Edge import:
  - `PERSONAL_TOOLBAR_FOLDER="true"` on the **H3** that represents the **Favorites bar**.
  - `ADD_DATE` and `LAST_MODIFIED` timestamps are allowed (seconds since epoch).
- Line endings: **CRLF** are used in the Edge export function to match typical exports.

---

## Troubleshooting

**Q: Edit dialog doesn’t show but the log says “Opening edit dialog …”.**  
A: Ensure CSS contains:
```css
.modal-backdrop.show { display: flex; }
```
The code also forces visibility using `m.style.display='flex'` when opening and `'none'` when closing.

**Q: Button actions execute twice.**  
A: Event wiring is guarded (`wiredOnce`) and the Open file dialog is rate-limited (`_openGuardAt`), preventing duplicate actions.

**Q: “Open HTML” shows a picker but nothing loads.**  
A: The app uses a **token** per file-read to ignore duplicated `input/change` storms. Check the log for “File read OK” and “Loaded N link(s).”

**Q: Edge won’t import the saved file.**  
A: Use **Save As (Edge)**. Confirm the top structure (Favorites bar + Other favorites) and try again.

---

## Keyboard Shortcuts

- **Enter / Space** on focused buttons and folder items
- **Esc** closes the Edit dialog
- **Tab** moves focus through actionable controls

---

## Development Notes

- ES5 only: all code uses `var`/function declarations and avoids modern APIs.
- Triple-wired events: `addEventListener`, `attachEvent`, and `on*` fallback.  
- DOM-only; no external libraries; no network requests.
- Robust parser walks `DL/DT/H3` + `DL/DT/A` trees and tolerates minor case differences.

---

## License

MIT — do whatever you need; attribution appreciated.

---

## Changelog (high-level)

- Fix: Prevent duplicate “Favorites bar” in Edge export; place root links at file root.
- Fix: Dialog visibility selector changed to `.modal-backdrop.show` and JS display fallback.
- Fix: Guard against duplicate event wiring and duplicate file-input events.
- Feature: Multi-select & bulk delete folders with checkboxes.
- Feature: Remove Duplicates (URL-based).

