# nicojones.ca

Personal site. Hosted on GitHub Pages.

## Structure

```
index.html     main page — template + render loop
posts.js       the feed data (all posts live here)
styles.css     styles — matches the old inline styles exactly
images/        photos
videos/        videos
CNAME          custom domain config
```

## Adding a new post manually

Open `posts.js` and add a line to the end of the `posts` array:

```js
{ type: "image", src: "images/YourPhoto.jpg", caption: "your caption" },
```

or for a video:

```js
{ type: "video", src: "videos/YourClip.mp4", caption: "your caption" },
```

Then commit and push — GitHub Pages rebuilds in ~30 seconds.

## What changed in the cleanup (visual output is identical)

- The 50+ duplicated inline-styled post blocks got replaced by a single
  template in `index.html` that renders the `posts` array from `posts.js`.
  Adding a new post is one line instead of copying seven.
- Inline styles moved into `styles.css` with the same values. No visual
  change — the site renders pixel-identical to the old version.
- Fixed broken HTML hygiene: added `<!DOCTYPE html>`, `<meta charset>`,
  `<meta name="viewport">` (so mobile and Japanese characters render
  correctly), and removed the ~30 orphan `</div>` tags that had
  accumulated from copy-pasting.
- Removed `id="myVideo"` from every video — you can't legally have the
  same id on multiple elements, and nothing was using it.
- Added `loading="lazy"` to images and `preload="metadata"` to videos,
  so the page doesn't try to download every asset before it's visible.
  This is the single biggest perceived-speed win.
- Fixed the `CNAME` file (it previously contained `nicojones.canicojones.ca`
  run together — probably worked by accident).
- Deleted the unused `/docs/` folder.

## Known issues not yet addressed

**Repo size (286MB).** This is the real perf bottleneck. 62 images average
~1.7MB each and 15 videos average ~12MB. GitHub Pages has a soft 1GB repo
cap and a 100GB/month bandwidth cap, and files over 100MB get rejected. You
won't hit those today, but you'll get there. When you're ready:

```bash
# Resize+recompress all images (requires imagemagick)
# Run from repo root.
for f in images/*.{jpg,jpeg,JPG,JPEG,png,PNG}; do
  [ -e "$f" ] || continue
  convert "$f" -resize '1600x1600>' -quality 82 "$f"
done

# Videos: re-encode to a reasonable bitrate (requires ffmpeg)
for f in videos/*.{mp4,mov,MP4,MOV}; do
  [ -e "$f" ] || continue
  ffmpeg -i "$f" -c:v libx264 -crf 23 -preset slow -c:a aac -b:a 128k "${f%.*}_compressed.mp4"
done
```

Expect 5-10× size reduction on images and 2-4× on videos with no visible
quality loss.

**Filename weirdness to eventually clean up:**
- `images/Hong Kong.jpg` has a space
- `images/Nico and Jessie Casino.JPG.jpg` has a double extension
- Mixed casing: `.JPG`, `.jpg`, `.JPEG`, `.jpeg`, `.PNG`, `.png`

These currently work but are the kind of thing that breaks silently later.
Rename to lowercase, no-spaces, single-extension and update `posts.js` to
match.

## Coming next: post.nicojones.ca

Plan is Decap CMS + a tiny Cloudflare Workers OAuth proxy. You log in with
GitHub, fill out a form (photo/video + caption), and it commits a new entry
to `posts.js` automatically. Not built yet — this cleanup had to come first
so the CMS has a predictable data shape to write to.
