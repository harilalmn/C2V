# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Code2Viz Web (C2V) is a browser-based 2D geometry visualization app built with vanilla JavaScript (ES Modules) and HTML5 Canvas. Users write JavaScript to create geometric shapes rendered on an interactive canvas. No build step or bundler — modules are loaded directly by the browser.

## Development Commands

```bash
npm run dev       # Start dev server on port 3000 (npx serve . -p 3000 -s)
npm start         # Same as dev
```

There is no build, lint, or test configuration. The app runs directly from source files served statically.

## Architecture

**Entry point**: `index.html` loads `app.js` as an ES module. CodeMirror 5.65.18 is loaded via CDN for the code editor.

**Module graph** (all ES modules, no bundler):

- `app.js` — Main application: project/file management, editor setup (CodeMirror), code execution, keyboard shortcuts, localStorage persistence (`code2viz_project` key)
- `geometry/index.js` — Core geometry library: `VXYZ` (immutable 3D vector), `BoundingBox`, 15 shape classes (`VPoint`, `VLine`, `VCircle`, `VArc`, `VRectangle`, `VEllipse`, `VPolygon`, `VPolyline`, `VSpline`, `VBezier`, `VArrow`, `VText`, `VDimension`, `VRadialDimension`, `VGroup`, `VGrid`), styling (`VColor`, `VFont`, `LineTypes`, `ShapeDefaults`), and a global shape registry (`_shapeRegistry`)
- `geometry/boolean.js` — Polygon boolean operations (union, intersection, difference) via Sutherland-Hodgman clipping
- `geometry/arrays.js` — Pattern operations: linear, rectangular, polar, path arrays, and mirror
- `renderer.js` — `CanvasRenderer` class: world↔screen coordinate transforms, pan/zoom, grid/axis drawing, layer-based shape rendering
- `selection.js` — `SelectionManager`: hit testing, single/multi-select, rubber-band selection, drag-to-move
- `tools.js` — `ToolManager` + `SnapEngine`: drawing tools (pointer, point, line, circle, rectangle, measure) with grid/endpoint/midpoint/center/intersection snapping
- `animation.js` — Timeline-based animation system: `DrawAnimation`, `MoveAnimation`, `FadeAnimation`, `ScaleAnimation`, `RotateAnimation`, `ColorAnimation`, 13+ easing functions, global `Animator`
- `exporter.js` — Export to SVG, PNG, PDF, DXF with coordinate transformation
- `intellisense.js` — CodeMirror autocomplete provider with parameter hints for all geometry APIs
- `properties.js` — Properties panel for editing selected shape attributes
- `layers.js` — Layer management (visibility, locking, shape assignment)
- `minimap.js` — Scaled overview widget with click-to-navigate
- `help.js` — Searchable help topics database with API reference and examples

**Key design patterns**:
- Shapes auto-register in a global `_shapeRegistry` array on construction; `clearRegistry()` resets it before each code run
- Coordinate system is mathematical: Y-up, origin at canvas center
- All vector operations on `VXYZ` are immutable (return new instances)
- User code runs via `eval()` in `app.js` with geometry classes injected into scope

## Browser Requirements

File System Access API (Chrome/Edge) for folder-based projects. Falls back to localStorage for single-file use.
