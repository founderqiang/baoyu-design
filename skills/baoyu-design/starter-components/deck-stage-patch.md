# deck-stage.js local patches

`deck-stage.js` is wholesale-overwritten whenever Claude Design ships an upgrade. This file records the local patches we layer on top of it, so they can be reapplied after each upgrade.

Per-upgrade flow:
1. Overwrite `deck-stage.js` with the new upstream version.
2. Reapply each patch below, locating it by its "anchor" string (so it still works even if upstream shifts line numbers).
3. Run `node --check deck-stage.js` to confirm there are no syntax errors.

---

## Patch 1: native fullscreen auto-hides the thumbnail rail

**Motivation**: the component only hides the rail when the host enters presentation mode via `postMessage({__omelette_presenting:true})`. It does **not** listen for native browser fullscreen. So when the deck is deployed standalone — or when fullscreen is entered with F11 / `element.requestFullscreen()` — the rail does not auto-hide.

**Approach**: add an independent `_fullscreen` flag and listen for `fullscreenchange`. Use a separate flag rather than reusing `_presenting`, so it doesn't clobber the host's presentation-mode messages (both paths can coexist).

Four edits.

### 1.1 `connectedCallback` — register the fullscreenchange listener

**Anchor** (immediately after the beforeprint/afterprint registration):

```js
      window.addEventListener('beforeprint', this._onBeforePrint);
      window.addEventListener('afterprint', this._onAfterPrint);
```

**Insert after it**:

```js
      // Native browser fullscreen (F11 / element.requestFullscreen) hides the
      // rail the same way host-driven presenting does. Independent flag so it
      // doesn't clobber _presenting when both paths are in play.
      this._onFsChange = () => {
        this._fullscreen = !!document.fullscreenElement;
        this._syncRailHidden();
        this._fit();
        this._scaleThumbs();
      };
      document.addEventListener('fullscreenchange', this._onFsChange);
```

### 1.2 `disconnectedCallback` — unbind the listener

**Anchor**:

```js
      window.removeEventListener('afterprint', this._onAfterPrint);
```

**Insert after it**:

```js
      if (this._onFsChange) document.removeEventListener('fullscreenchange', this._onFsChange);
```

### 1.3 `_railWidth()` — return 0 in fullscreen (let the canvas fill)

**Anchor / before**:

```js
      if (!this._railEnabled || !this._railVisible || this.hasAttribute('no-rail')
          || this.hasAttribute('noscale') || this._presenting || this._previewMode
          || NARROW_MQ.matches) return 0;
```

**After** (add `|| this._fullscreen`):

```js
      if (!this._railEnabled || !this._railVisible || this.hasAttribute('no-rail')
          || this.hasAttribute('noscale') || this._presenting || this._previewMode
          || this._fullscreen || NARROW_MQ.matches) return 0;
```

### 1.4 `_syncRailHidden()` — count fullscreen as a hard hide (display:none)

**Anchor / before**:

```js
      const hard = !this._railEnabled || this._presenting || this._previewMode;
```

**After** (add `|| this._fullscreen`):

```js
      const hard = !this._railEnabled || this._presenting || this._previewMode || this._fullscreen;
```

---

## Verification

- `node --check deck-stage.js` passes.
- Open any deck in the browser and enter fullscreen via the **Fullscreen API** — e.g. `document.documentElement.requestFullscreen()` from a user gesture (button/keypress). The rail and its right-edge resize handle both disappear (`.rail[data-presenting]{display:none}` plus the adjacent-sibling selector that hides the resize handle), and the canvas re-fits to fill the viewport; exiting fullscreen restores the rail.
  - Note: the browser's own F11 fullscreen does **not** fire `fullscreenchange` or set `document.fullscreenElement`, so it won't hide the rail — only the Fullscreen API does. This matches how a "present" button (which calls `requestFullscreen()`) behaves.
  - Quick check without a real gesture: in devtools, `const d = document.querySelector('deck-stage'); d._fullscreen = true; d._syncRailHidden(); d._fit(); d._scaleThumbs();` should hide the rail; set `d._fullscreen = false` and rerun to restore.
- Host presentation mode (`__omelette_presenting`) is unaffected — the two flags are independent.
