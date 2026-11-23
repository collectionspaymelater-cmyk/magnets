# Photo-to-video WebAR viewer

A small static A-Frame + MindAR image-tracking viewer. Each URL loads one catalog entry, for example `https://your-site.example/?id=001`. There is no app to install, backend, login, or dashboard.

## Add your photos and videos

1. For each printed photo, create a MindAR image target using the [MindAR image-target compiler](https://hiukim.github.io/mind-ar-js-doc/tools/compile). Add the photo, compile the target, and upload the resulting `.mind` file to public object storage.
2. Upload the matching `.mp4` video. Use distinct, matching IDs for target and video filenames, such as `001.mind` and `001.mp4`.
3. Edit `catalog.json` (or use the helper below) and replace the example URLs with the public HTTPS URLs for your files. Add entries under `items`; use the ID in the viewer URL, such as `?id=001`.
4. Set `aspectRatio` to **image height / image width**. The video plane is one unit wide and uses this ratio for its height. Prefer matching the video crop to the tracked photo.

## Generate/update catalog entries from MP4s

Install [FFmpeg](https://ffmpeg.org/) locally so `ffprobe` is on your PATH. Put files named by ID in a folder, for example `videos/001.mp4`, then run:

```sh
node scripts/generate-catalog.mjs --video-dir ./videos --video-base-url https://assets.example.com/videos --mind-base-url https://assets.example.com/targets
```

The script reads MP4 dimensions with `ffprobe`, writes `aspectRatio`, and maps each `001.mp4` to `001.mind`. It merges into the existing catalog (keeping unrelated entries) and overwrites generated URLs for scanned IDs. Upload each corresponding `.mind` target to the configured target URL.

## Storage CORS

For an R2/S3 bucket served from a public HTTPS domain, allow browser `GET` requests from your deployed site origin (and your local test origin). Allow the `GET` method and the `Range` request header; expose `Content-Length` and `Content-Range` if your storage setup requires exposed response headers. Video and `.mind` responses must have valid CORS headers; MP4s should be served as `video/mp4`. Never put storage credentials in this static project.

## Test over HTTPS

Camera access requires a secure context: use HTTPS in deployment or `localhost` during development. For example, with Python installed:

```sh
python -m http.server 8000
```

Open `http://localhost:8000/?id=001`, allow camera access, and point the camera at the matching photo. Replace the sample catalog URLs first. Test on a real phone as well as desktop; mobile browsers may require a tap before audio can play.

## Deploy for free

Push these static files to a Git repository and connect it to Cloudflare Pages or Vercel using the static-site/no-build setup (project root as the output directory). No build command is needed. The `.mind` and `.mp4` assets can remain in public R2/S3 storage, separate from the site. After deployment, share links such as `https://your-site.example/?id=001`.

## Tracking and playback

Only the selected item's target is mounted. Tracking starts video playback muted when the image is found and pauses it when the image is lost. Tap **Tap for sound** to unmute. In `index.html`, `filterMinCF` and `filterBeta` tune tracking stability: lower `filterMinCF` smooths idle jitter but adds lag; higher `filterBeta` makes tracking respond more quickly to motion, potentially allowing more jitter. Adjust gradually and test on target devices.
