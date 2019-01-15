# CanvasTools librarary for VoTT

CanvasTools is one of the UI modules used in the [VoTT project](https://github.com/Microsoft/VoTT/). The library impelements the following core features:

* Region (box) selection & manipulation
* Filters pipeline for underlaying canvas element
* Toolbar for all available tools

## Dependencies

* CanvasTools heavily uses the [Snap.Svg](https://github.com/adobe-webplatform/Snap.svg) library. In the webpack-eged version it is bundled with CanvasTools into one `ct.js` file, including also styles.
* Current version of the library depends on some features (e.g., masks-support in SVG) that are not fully cross-browser, but targeting Electron (Chromium).

## How to use

### Install npm package

Install package from npm:

```node
npm i vott-ct
```

The package structure:

```txt
dist/
    ct.d.ts - bundled typings
    ct.js - webpack bundle for production ({tsc->commonjs, snapsvg, styles} -> umd)
    ct.js.map - source map for ct.js
    ct.min.js -- webpack minimized bundle for production
    ct.min.js.map - source map for ct.min.js
    ct.dev.js -- webpack bundle for development (incl source map)
lib/
    css/
        canvastools.css
    icons/
        {collection of svg icons for toolbar}
    ct.d.ts - typings generated by tcs
    ct.js - AMD module generated by tcs
    ct.js.map - map file generated by tcs
```

### Add library to the app

Add the `ct.js` file to your web-app (e.g., an Electron-based app).

```html
<script src="ct.js"></script>
<!-- OR -->
<script src="ct.min.js"></script>

```

Copy toolbar icons from the [`src` folder](https://github.com/kichinsky/CanvasTools-for-VOTT/tree/master/src/canvastools/icons) to your project.

Create a reference to the CanvasTools.

```js
let ct = CanvasTools.CanvasTools;
```

### Add Editor to the page

Add container elements to host SVG elements for the toolbar and the editor.

```html
<div id="ctZone">
    <div id="toolbarzone"></div>
    <div id="selectionzone">
        <div id="editorzone"></div>
    </div>
</div>
```

Initiate Editor-object from the CanvasTools.

```js
var sz = document.getElementById("editorzone");
var tz = document.getElementById("toolbarzone");

var editor = new ct.Editor(sz);
editor.addToolbar(tz, ct.Editor.FullToolbarSet, "./images/icons/");
```

The editor will auto-adjust to available space in provided container block.
`FullToolbarSet` icons set is used by default and exposes all available tools. The `RectToolbarSet` set contains only box-creation tools.
Correct the path to toolbar icons based on the structure of your project.

### Add callbacks to the Editor

Add a callback for `onSelectionEnd` event to define what should happen when a new region is selected. Usually at the end of processing new region you want to add it actuall to the screen. Use `.RM.addPointRegion` to register point-based regions, and `.RM.addRectRegion` to register box-based regions.

```js
// Callback for onSelectionEnd
editor.onSelectionEnd = (commit) => {
    let r = commit.boundRect;
  
    // Build a random tags collection
    let tags = generateTagDescriptor();

    // Add new region to the Editor based on selection type
    if (commit.meta !== undefined && commit.meta.point !== undefined) {
        let point = commit.meta.point;
        editor.RM.addPointRegion((incrementalRegionID++).toString(), new ct.Core.Point2D(point.x, point.y), tags);
    } else {
        editor.RM.addRectRegion((incrementalRegionID++).toString(), new ct.Core.Point2D(r.x1, r.y1), new ct.Core.Point2D(r.x2, r.y2), tags);
    }
}

// Generate random tags
let primaryTag = new ct.Core.Tag(
                        (Math.random() > 0.5) ? "Awesome" : "Brilliante",
                        Math.floor(Math.random() * 360.0));
let secondaryTag = new ct.Core.Tag(
                        (Math.random() > 0.5) ? "Yes" : "No",
                        Math.floor(Math.random() * 360.0));
let ternaryTag = new ct.Core.Tag(
                        (Math.random() > 0.5) ? "one" : "two",
                        Math.floor(Math.random() * 360.0));

function generateTagDescriptor() {
    let tags = (Math.random() < 0.3) ?
                    new ct.Core.TagsDescriptor(primaryTag, [secondaryTag, ternaryTag]):
                ((Math.random() > 0.5) ?
                    new ct.Core.TagsDescriptor(secondaryTag, [ternaryTag, primaryTag]):
                    new ct.Core.TagsDescriptor(ternaryTag, [primaryTag, secondaryTag]));

    return tags
}
```

### Update background

Once the background image for tagging task is loaded (or a video element is ready, or a canvas element), pass it to the editor as a new content source.

```js
let imagePath = "./../images/background-forest-v.jpg";
let image = new Image();
image.addEventListener("load", (e) => {
    editor.addContentSource(e.target);
});
image.src = imagePath;
```

## Changelog

### 2.1.4

1. Added a new `api` proxy to the `Editor` class. It wraps accessing to all the public methods of `Editor`, `RegionsManager`, `AreaSelector` and `FilterPipeline`. So instead of writing `editor.RM.addRegion(...)`, you can use the following approach:
    ```js
    var editor = new ct.Editor(editorDiv).api;
    editor.addRegion(...)
    editor.setSelectionMode(...)
    ```

2. Removed from the `Editor` class itself the `setSelectionMode` method. Use instead the approach above or `editor.AS.setSelectionMode(...)`.

3. Added new overloads for the `Editor` class `constructor`. You can now also provide custom components (`AreaSelector`, `RegionsManager` or `FilterPipeline`). E.g., to create `Editor` with custom `RegionsManager`:
    ```js
    let editor = new ct.Editor(sz, null, regionsManager);
    ```
    Note: editor will override the `callbacks` properties for `AreaSelector` and `RegionsManager` to ensure they crossreference and can work together.  