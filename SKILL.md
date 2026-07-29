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

## Implement loading behavior

Use these defaults unless the user explicitly prioritizes immediate fidelity over performance:

```html
<img src="thumbnail.webp" loading="lazy" decoding="async" alt="…">
<video preload="none" playsinline poster="poster.webp">…</video>
```

- Exclude only genuine above-the-fold images from lazy loading.
- For dense case-study sections, leave thumbnail `src` unset and store it in `data-src`; use `IntersectionObserver` with a modest prefetch margin (roughly 300–500px) to populate it near the viewport.
- Paginate or virtualize large catalogs. Render and load only the current page; ten cards per page is a sound default for image-heavy report grids.
- Keep original-resolution URLs out of card markup. Use them only inside the modal/detail renderer.
- Avoid eager video poster requests in repeated cards. Assign posters together with deferred thumbnails when the card approaches the viewport.
- Preserve accessibility: give meaningful images an appropriate `alt`, expose visible controls for detail views, and retain keyboard close/previous/next behavior when modifying a gallery.

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
5. Inspect initial network behavior in a browser: the first screen must not request offscreen galleries, all catalog pages, or video files.
6. Review the repository diff for unreferenced binaries and remove them from the release scope.

## Release discipline

- Change the designated source report first, then copy only its required HTML and referenced assets into a clean publication worktree.
- Do not publish until the user explicitly requests it.
- After deployment, open the public URL and smoke-test the first screen, one paginated catalog page, one case gallery, and one video interaction.
- If a project has a handoff document, treat its current override section as authoritative over older historical notes.
