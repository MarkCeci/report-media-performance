---
name: report-media-performance
description: Optimize images, icons, screenshots, and video in static HTML research reports. Use when adding or replacing report media, diagnosing slow report loading, implementing lazy loading or responsive thumbnails, preparing GitHub Pages assets, or validating that media does not block the first screen.
---

# Report Media Performance

Keep the first screen fast while retaining high-quality evidence media on demand. Treat the report HTML and its deployed asset directory as one release unit.

## Classify every asset before adding it

1. Confirm that the page actually references the asset. Do not publish source exports, old encodes, duplicate clips, temporary files, or unused originals.
2. Classify images as either **thumbnail/list media** or **full-detail media**.
   - Generate a compact WebP thumbnail for cards, galleries, lists, and video posters.
   - Request the original only after an explicit click opens a modal, lightbox, or detail view.
3. Keep app icons local and compressed. Do not make a report depend on App Store, CDN, or other third-party icon requests.
4. Treat video as opt-in media: a still poster first, then request the video only after a user plays it.

## Convert report images to WebP before wiring them in

1. Inventory every image requested at runtime, including Hero/backgrounds, card thumbnails, full-size case screenshots, case icons, catalog icons, and system icons.
2. Generate WebP counterparts for every runtime JPG/PNG and update all direct and dynamic paths together. For dynamic filename maps, change both the base directory and extension transform; do not leave a hidden PNG/JPG fallback path.
3. Preserve native dimensions for full-detail screenshots and use high visual quality (typically WebP quality 88–92). WebP reduces transfer size; it does not add detail to the original.
4. Keep source JPG/PNG only as local backup when needed. Do not include them in the release copy or request them from the report once WebP is live.
5. Validate that every runtime image path resolves to WebP, every generated WebP decodes, and its dimensions match the source where fidelity matters.

## Implement loading behavior

Use these defaults unless the user explicitly prioritizes immediate fidelity over performance:

```html
<img src="thumbnail.webp" loading="lazy" decoding="async" alt="…">
<video preload="none" playsinline poster="poster.webp">…</video>
```

- Exclude only genuine above-the-fold images from lazy loading.
- Use exactly one deferral mechanism for each image. If `IntersectionObserver` controls `data-src`, do not leave that image under native `loading="lazy"` after the observer fires: set `loading = 'eager'`, then set `src`. Layering both mechanisms can leave visible cards blank on some browsers.
- For a bounded key case-study section (for example, 12 cases with compact WebP thumbnails), observe the section container—not individual cards—and load all section thumbnails plus video posters once it is near the viewport. This prevents blank media during fast scrolling while keeping the first screen free of those requests. Keep originals and videos click-to-load.
- Paginate or virtualize large catalogs. Render only the current page; ten cards per page is a sound default for image-heavy report grids. Current-page images may load eagerly because off-page cards do not exist in the DOM; do not apply native lazy loading to them again.
- Keep original-resolution URLs out of card markup. Use them only inside the modal/detail renderer.
- Avoid eager video poster requests in repeated cards. Assign posters together with deferred thumbnails when the card approaches the viewport.
- Preserve accessibility: give meaningful images an appropriate `alt`, expose visible controls for detail views, and retain keyboard close/previous/next behavior when modifying a gallery.

### Full-size screenshot galleries

- Do not make a user wait for each slide to start loading. When a bounded gallery opens (for example, five screenshots), place all full-size WebP images in the dialog DOM at once with `loading="eager"` and `fetchpriority="high"`.
- Keep a lightweight thumbnail image underneath each full image. Start the full image request immediately, wait for `load` and `decode()`, then fade the full image in. The thumbnail must remain visible if a user switches slides before the high-resolution request finishes.
- Do not rely only on detached `new Image()` preloads for a fast gallery: browsers may deprioritize those requests. Limit simultaneous high-priority loading to the opened gallery, never all cases on the page.

## Prepare video safely

1. Verify each candidate before wiring it into HTML:

```bash
ffmpeg -v error -xerror -i 'video-file' -frames:v 1 -f null -
```

2. If the file is corrupt, not browser-friendly, or unnecessarily large, encode a web version with H.264/AAC and fast start. Keep the source outside the published asset set unless it is needed.

```bash
ffmpeg -i 'input' -c:v libx264 -pix_fmt yuv420p -c:a aac -movflags +faststart 'output_web.mp4'
```

3. Keep `preload="none"`; do not silently replace it with `metadata` or `auto` in a media-heavy report.

## Validate before publishing

Run the checks against the exact release copy, not merely the authoring source:

1. Parse inline JavaScript and check all media paths are relative and present.
2. Search for `file://`, local absolute paths, and unintended remote media hosts.
3. Confirm every published video decodes with the command above.
4. Confirm thumbnail assets exist and that originals are reachable from their click handler only.
5. Confirm every runtime image path is WebP and no released HTML, CSS, or dynamic filename map still requests a JPG/PNG.
6. Inspect initial network behavior in a browser: the first screen must not request offscreen galleries, all catalog pages, or video files.
7. Open one full-size gallery and verify that its bounded image set begins loading together, its thumbnail fallback stays visible, and each high-resolution image fades in only after decode.
8. Scroll to each deferred media section and assert every expected thumbnail has `naturalWidth > 0`, no `data-src` remains pending, and no loaded image has `complete === true && naturalWidth === 0`.
9. After deployment, repeat the gallery assertion and smoke-test one catalog page plus one video interaction on the public URL; a `200` asset response alone does not prove that the page requested or rendered it.
10. Review the repository diff for unreferenced binaries and remove them from the release scope.

## Release discipline

- Change the designated source report first, then copy only its required HTML and referenced assets into a clean publication worktree.
- Do not publish until the user explicitly requests it.
- After deployment, open the public URL and smoke-test the first screen, one paginated catalog page, one case gallery, and one video interaction.
- If a project has a handoff document, treat its current override section as authoritative over older historical notes.
