# Lucky Bag AI: background removal that runs in your browser

[luckybagai.com](https://luckybagai.com/) puts product photos on a white background, sized for marketplaces such as Poshmark and eBay. The photos never leave your device. This page explains how that works and how you can check it yourself.

## What happens to a photo

1. You choose photos. The browser reads them from your disk. Nothing is sent anywhere.
2. The page loads a background model into the browser and runs it with [ONNX Runtime Web](https://github.com/microsoft/onnxruntime) (WebAssembly, up to 4 threads when the page is cross-origin isolated, as luckybagai.com is).
   - On a computer: BiRefNet_lite at 512 × 512 (MIT license), about 94 MB, downloaded once and kept in the browser's storage.
   - On a phone or tablet: U²-Netp at 320 × 320 (Apache-2.0), about 4.4 MB.
3. The model returns a mask. The page cuts the product out, centers it on white at 85% of the frame, and saves a JPG under 500 KB: 1024 × 1024 square, or 1080 × 1440 (3:4).
4. You download the files one by one or as a ZIP. The ZIP is built in the browser too.

## Why it cannot upload your photos

The site sends this security header on every tool page:

```
Content-Security-Policy: default-src 'self'; script-src 'self' 'wasm-unsafe-eval'; connect-src 'self'; img-src 'self' data: blob:; media-src 'self' blob:; worker-src 'self' blob:; style-src 'self' 'unsafe-inline'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'
```

`connect-src 'self'` means the page can only make network requests to luckybagai.com. No third-party scripts, fonts or trackers load. The one exception is the checkout page (`/pass/`), which loads Paddle's payment window after you press the buy button. The photo tool is not on that page.

What the page does send to luckybagai.com is a short count, for example `tool_done` with the tool name, the number of photos, how long they took and which model ran. It never sends a photo, a file name or anything read from inside a file. The full list is on the [privacy page](https://luckybagai.com/privacy/).

## Check it yourself

**Network tab.** Open the browser's developer tools (F12), go to Network, and process a few photos. You will see the model files load from luckybagai.com and a few small requests to `/api/event`. No request carries image data. You may also see one blocked request to `static.cloudflareinsights.com`: our host, Cloudflare, adds its analytics script to the page, and the security header above stops it from loading.

**Turn off the internet.** Process one photo so the model is loaded. Turn off Wi-Fi. Choose more photos. They still come out. (We last ran this check on 10 November 2026.)

**Read the header.** In the Network tab, click the page itself and look at the response headers for `Content-Security-Policy`.

## How fast is it

On 10 November 2026 we ran a batch of 50 photos through the free tool, made from 4 product photos (1086 × 1448 each) used over and over, in Chrome on a computer with an AMD Ryzen 7 8845HS and 28 GB of memory. Free batches hold 30, so it took two batches: 30 in 128 seconds, then 20 in 65 seconds. That is 3 minutes 13 seconds in all. The second batch, with the tool already running, took about 3.2 seconds a photo; the first photo of the first batch took 13 seconds while the model loaded. In this test the model loaded from our own test server; on a first visit over the internet it also has to download, which depends on your connection.

## Other tools on the site that work the same way

- [Extract frames from a video](https://luckybagai.com/extract-frames-from-video/): plays your video from a local `blob:` address and saves frames as JPG.
- [Remove EXIF data and location](https://luckybagai.com/remove-exif-data/): reads the file's bytes, drops the metadata blocks (GPS, camera, date, XMP, IPTC, comments) and copies the picture data as it is, so nothing is re-compressed.
- [Photo grid maker](https://luckybagai.com/photo-grid-maker/): draws 2 to 9 photos onto one canvas and saves a JPG.
- [Compress JPEG to a size in KB](https://luckybagai.com/compress-jpeg/) and [batch resize](https://luckybagai.com/batch-resize/): canvas only, no model.
- [Signature background remover](https://luckybagai.com/remove-background-from-signature/): canvas only, no model.
- [Receipt scanner](https://luckybagai.com/receipt-scanner/): reads the text with Tesseract.js in the browser.

## What it does not do

No transparent PNG, no API, no account. It is made for product photos.

## Licenses

Models and libraries, with their license texts: [luckybagai.com/licenses/](https://luckybagai.com/licenses/).

Lucky Bag AI is run by NICASIA DIGITAL SOLUTIONS (SSM 202603028341), Malaysia. Questions: hello@luckybagai.com
