# Spectrum Image Maker

A browser-based image converter that turns JPG/PNG images into ZX Spectrum SCREEN$ format, with export to `.scr`, `.tap`, and scaled PNG/JPG.

**[Try it live →](https://factus10.github.io/spectrum-image-maker/)**

## Features

- **Interactive cropping** — drag to select any region of the source image, with optional 4:3 aspect lock
- **10 dithering algorithms**:
  - Floyd-Steinberg
  - Atkinson
  - Stucki
  - Jarvis-Judice-Ninke
  - Burkes
  - Sierra Lite
  - Ordered (Bayer 4×4)
  - Blue noise (interleaved gradient noise)
  - Yliluoma (palette-aware, projects onto the ink↔paper line)
  - Halftone (clustered-dot)
- **Two color strategies**:
  - Per-block best-fit — exhaustive search across all 128 ink/paper/bright combinations per 8×8 attribute block
  - Global pre-quantize — faster approximation using the dominant colors in each block
- **Live preview** with side-by-side source and converted views
- **Adjustments** for brightness, contrast, and saturation
- **Attribute grid overlay** to visualize the Spectrum's 8×8 color-clash boundaries
- **Spectrum palette** reference strip
- **Exports**:
  - `.SCR` — raw 6912-byte SCREEN$ binary
  - `.TAP` — tape file (loads directly in emulators like Fuse, loads to address 16384 as `CODE`)
  - PNG and JPG at 1×, 2×, or 4× scale, with preserved sharp pixels

## How it works

The ZX Spectrum has a famously constrained display: 256×192 pixels with only 2 colors (ink + paper) per 8×8 attribute block, drawn from a 15-color palette (8 hues × 2 brightness levels, minus duplicated black). This creates the classic "color clash" effect.

For each 8×8 block, Spectrum Image Maker finds the optimal ink/paper/bright combination by dithering the block with each candidate pair and measuring total perceptual error (weighted RGB distance, with green weighted highest to match human vision). The chosen attributes and dithered pixel pattern are then written to a Spectrum-layout screen buffer using the correct interleaved memory addressing:

```
addr = ((y & 0xC0) << 5) | ((y & 0x07) << 8) | ((y & 0x38) << 2) | (x >> 3)
```

The `.TAP` export wraps the screen data in a standard tape file with a CODE header pointing at address 16384, so it can be loaded directly on a real Spectrum or in any emulator.

## Running locally

It's a single self-contained HTML file with no dependencies. Just open `index.html` in a browser, or serve it with any static file server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## License

GPL v3 — see [LICENSE](LICENSE).
