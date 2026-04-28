# Retro Pixel Converter

A browser-based image converter for classic 8-bit and 16-bit machines. Converts JPG/PNG images to native graphics formats for the ZX Spectrum, Timex/Sinclair 2068, Commodore 64, Atari 800, and Sinclair QL.

_(The GitHub repo is still named `spectrum-image-maker` for URL stability, since the project started as a ZX Spectrum tool and grew from there.)_

**[Try it live →](https://factus10.github.io/spectrum-image-maker/)**

## Supported display modes

| Machine | Mode | Resolution | Attributes | Colors | File |
|---|---|---|---|---|---|
| ZX Spectrum | Standard | 256×192 | 8×8 | 2 of 16 (Normal + Bright, bright black = pure black) | `.scr` / `.tap` |
| TS 2068 | Extended Color (ECM) | 256×192 | **8×1** | 2 of 16 per strip (bright black = dark grey) | `.scr` / `.tap` |
| TS 2068 | 64-column hi-res | 512×192 | global | 8 hardware ink/paper combos (paper = complement) | `.scr` / `.tap` |
| C64 | Hi-res bitmap | 320×200 | 8×8 | 2 of 16 (Pepto) | `.prg` |
| C64 | Multicolor / Koala | 160×200 | 4×8 | 4 of 16 + auto-chosen global bg | `.kla` |
| Atari 800 | GR.15 / ANTIC E (MicroPainter) | 160×192 | global | 4 of 128 | `.mic` |
| Atari 800 | GR.8 | 320×192 | global | 2 user-picked (mono) | `.gr8` |
| Atari 800 | GR.9 | 80×192 | per-pixel | 16 luma shades of one hue | `.gr9` |
| QL | Mode 8 / Low res | 256×256 | per-pixel | 8 | `_scr` |
| QL | Mode 4 / Hi-res | 512×256 | per-pixel | 4 fixed (black/red/green/white) | `_scr` |
| Pico-8 | Standard palette | 128×128 | per-pixel | 16 fixed | `.bin` (+ optional hex `.txt`) |

Plus user-defined **custom modes** loaded via JSON profile import.

Each mode's palette and attribute constraints are honored natively:
- C64 modes use the Pepto/Colodore palette
- QL Mode 4 is locked to its 4 hardwired colors
- Atari GR.15 picks 4 global colors from 128 via frequency-quantize
- TS 2068 64-col hi-res exposes only the 8 valid hardware ink/paper pairs
- ZX Spectrum standard renders bright-black as pure black (matching real hardware)
- TS 2068 modes render bright-black as the distinctive dark grey (~#606060)

## Dithering algorithms (12)

- **Error-diffusion**: Floyd-Steinberg / Atkinson / Stucki / Jarvis-Judice-Ninke / Burkes / Sierra Lite
- **Hilbert-curve FS** — error diffusion that traverses pixels along a Hilbert space-filling curve instead of scanlines, hiding directional artifacts
- **Threshold / pattern**: Ordered (Bayer 4×4) / Ordered (Bayer 8×8) / Blue noise (interleaved gradient noise) / Yliluoma (palette-aware projection) / Halftone (clustered-dot)
- **None** — plain nearest-color quantization

Error-diffusion dithers propagate across block boundaries globally (critical for ECM's 8×1 strips and QL's per-pixel modes). Error does not bleed into letterbox bars outside the scaled image.

## Color search strategies

- **Weighted average pair fit** (default, contributed by Josef Jelinek) — for each candidate ink/paper pair, find the blend ratio whose result is closest to the block's average color, then pick the pair whose best blend matches best. Skips the expensive dither-and-measure step while still picking excellent pairs. Josef's own evaluation:
  > For colorful pictures, in normal mode, it's neck-and-neck with per-block best-fit. For grayscale, it produces much better results. All this while being 5–10× faster. In TS 2068 high-color (ECM) mode, there is no comparison — the per-block best-fit is mostly broken there, and WAPF wins cleanly.
- **Per-block best-fit** — try all N-combinations of palette colors per block, score each by actually running the chosen dither and measuring per-pixel error. Highest-effort search, sometimes preferred for colorful images in standard ZX mode — but not recommended for ECM (see above).
- **Global pre-quantize** — frequency-based counting for modes where exhaustive is infeasible (Atari GR.15 picks 4 of 128), or as a fast fallback on other modes. Can produce minor artifacts on gradients.
- **Pixel-direct** — 1×1 blocks with no search; dither each pixel to the full palette (QL, Atari GR.9).
- **User-picked** — monochrome and hardware-constrained modes (TS 2068 64-col, Atari GR.8).

For threshold-based dithers (Bayer, blue noise, halftone, yliluoma), the per-block search is nudged toward distinct ink/paper pairs on non-uniform blocks so gradients actually dither instead of quantizing flat.

## Features

- **Interactive cropping** with a 4:3 aspect lock that stays correct even when dragging corners past the image boundary
- **Live side-by-side preview** with correct pixel aspect ratio per mode
- **Adjustments**: brightness, contrast, saturation, and gamma (all reflected in the source preview in real time)
- **Block pre-blur** slider — a Gaussian radius applied only to the block-color-selection pass, making neighbouring blocks pick more cohesive colors without blurring the final dither
- **Dither in linear color space** (experimental) — perform quantization and error diffusion in sRGB-linear rather than gamma-encoded sRGB, for smoother gradients
- **Mode-aware palette strip** — shows the actual colors available for the current mode (8 for QL Mode 8, 4 for QL Mode 4, 16 for Spectrum/C64, a 16×4 sampling for Atari's 128-color palette, etc.)
- **Click palette colors to disable/enable** them on the fly (the color search skips disabled entries; useful for reducing jarring outlier blocks)
- **Attribute grid overlay** — visualise color-clash boundaries
- **Image export**: PNG/JPG at 1×, 2×, 4× with pixel aspect correction and sharp pixelated scaling
- **Binary export** — native file format per mode (`.scr`, `.prg`, `.kla`, `.mic`, `.gr8`, `.gr9`, `_scr`)
- **Tape export** — `.tap` file with correctly-addressed CODE blocks, for ZX Spectrum / TS 2068 modes
- **CRT frame** — optional retro monitor bezel around the converted output, with an optional scanline overlay
- **Zoom output** — 1×, 2×, 4×, 8×, or "fit" sizing, plus a **Fullscreen** button that opens the converted image on a black overlay at maximum fit size
- **A/B comparison** — snapshot the current conversion, then adjust settings and toggle a side-by-side view to compare
- **Profile import / export** — save every setting (mode, dither, adjustments, disabled colors, crop rect, CRT frame, A/B) as a JSON file for later restoration
- **Custom mode import** — merge user-defined modes into the dropdown at runtime via JSON (PNG export works for custom modes; binary export is mode-specific code-wise)

## Pixel aspect ratios

The app tracks pixel aspect ratio (PAR = width/height) per mode and applies the correct stretch when rendering previews and PNG/JPG exports, so output looks physically correct on a modern square-pixel display:

| Mode | PAR | Example render |
|---|---|---|
| ZX Spectrum / ECM / C64 Hi-res | ~1.0 (~0.94 PAL C64) | square pixels |
| TS 2068 64-col hi-res | 0.5 | pixels 2× taller than wide |
| C64 Multicolor | 1.87 | fat pixels (2× wider) |
| Atari GR.15 | 2.0 | fat pixels (PAL) |
| Atari GR.8 | 1.0 | square |
| Atari GR.9 | 4.0 | very wide pixels |
| QL Mode 8 | 1.47 | slightly wide pixels |
| QL Mode 4 | 0.74 | slightly tall pixels |

The crop-fit math also accounts for PAR, so a 4:3 crop fills the entire target pixel grid for modes like 64-col where the pixel grid's raw 8:3 aspect corresponds to a 4:3 display.

## Binary formats

The "Download binary" button emits the native format for each mode:

- **ZX Spectrum SCREEN$** (`.scr`) — 6912 bytes, interleaved screen memory + 768 byte attribute file
- **TS 2068 ECM** (`.scr`) — 12288 bytes (pixels at 0x4000 + attributes at 0x6000, same interleaved layout)
- **TS 2068 64-col** (`.scr`) — 12288 bytes (DF1 even columns + DF2 odd columns)
- **C64 Hi-res PRG** (`.prg`) — 2-byte load addr + 8000 byte bitmap + 1000 byte screen RAM (fg/bg nibbles)
- **C64 Koala** (`.kla`) — 10003 bytes: 2-byte load addr + 8000 byte bitmap + 1000 screen RAM + 1000 color RAM + 1 byte background
- **Atari MicroPainter** (`.mic`) — 7684 bytes: 7680 byte bitmap + 4 color register values
- **Atari GR.8** (`.gr8`) — 7680 bytes, 1 bit/pixel, MSB-first
- **Atari GR.9** (`.gr9`) — 7680 bytes, 4 bits/pixel (16 luma levels)
- **QL Mode 4 / Mode 8** (`_scr`) — 32768 bytes, 128 bytes/line, bitplane-packed per QL screen memory layout

ZX/Timex modes additionally support `.tap` (tape image) export with correctly-addressed CODE blocks, ready to load in an emulator like Fuse.

## How the ZX Spectrum block dithering works

The ZX Spectrum has only 2 colors (ink + paper) per 8×8 attribute block, drawn from a 16-color palette (8 hues × 2 brightness levels). This creates the classic "color clash" effect.

For each attribute block, the converter finds the optimal color combination by dithering the block with each candidate pair and measuring total perceptual error (weighted RGB distance: `3·dR² + 4·dG² + 2·dB²`, matching BT.601 luma contributions). The chosen attributes and pixel pattern are then written to a Spectrum-layout screen buffer using the correct interleaved memory addressing:

```
addr = ((y & 0xC0) << 5) | ((y & 0x07) << 8) | ((y & 0x38) << 2) | (x >> 3)
```

### Timex 2068 advanced modes

Per the TS 2068 Technical Manual section 5.2, set via Port 0xFF:

- **Extended Color Mode (ECM, port=2)** enables 8×1 attribute blocks — 24× finer vertical color resolution. Pixel data at 0x4000 (same layout as standard Spectrum), attribute data at 0x6000 (same interleaved layout, one attribute per pixel byte). Dramatically reduces color clash and is ideal for photo conversion.
- **64-column mode (port=6|(ink<<3))** is 512×192 monochrome. Even character columns (0,2,4…62) come from DF1 at 0x4000, odd columns (1,3,5…63) come from DF2 at 0x6000. Only 8 ink/paper combinations are hardware-realizable (paper = RGB complement of ink); the UI exposes these directly rather than letting users pick invalid pairs. BRIGHT and FLASH are fixed at 0.

### Loading on real hardware or emulators

ZX/Timex: the `.tap` export includes CODE blocks at the correct addresses. For ECM/64-col modes, set the video mode first (`OUT 255, 2` for ECM, `OUT 255, 6+(ink<<3)` for 64-col) before loading.

C64: the `.prg` / `.kla` files load at the standard bitmap area ($2000) and screen RAM ($4000 area); use Koala Painter or a simple viewer.

Atari 800: `.mic` loads in MicroPainter or modern viewer tools; `.gr8` / `.gr9` are raw bitmaps (load at the proper display memory address, e.g. via a short BASIC loader).

QL: the `_scr` file is a direct dump of screen RAM (base $20000), loadable by standard QL emulators (QPC2, SMSQ/E, uQLx, QLAY) using `LBYTES scr_name,131072` or equivalent.

## Running locally

Single self-contained HTML file with no dependencies. Open `index.html` in a browser, or serve it with any static file server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Credits and references

Special thanks to **Josef Jelinek** for invaluable feedback and technical guidance throughout development — including identifying a weighted color-distance bug, the TS 2068 bright-black-as-dark-grey subtlety, the 64-column mode's 2:1 pixel aspect ratio, the ECM per-block error-diffusion limitation, the 8 hardware-only ink/paper combos in 64-col mode, the generalized mode-config schema that made Commodore 64, Atari, and QL support straightforward to add, and the long list of quality-of-life features (CRT frame, zoom, A/B compare, profile I/O, linear-space dithering, block pre-blur, Hilbert-FS dither).

References:

- **ZX Spectrum / TS 2068**: Sinclair / Timex original hardware manuals (TS 2068 Technical Manual §5.2)
- **C64 Pepto palette**: Philip "Pepto" Timmermann — pepto.de/projects/colorvic, colodore.com
- **Atari palette**: computed NTSC palette approximating Altirra / Olivier references
- **QL screen layout**: RWAP Software QL Technical Guide, Dilwyn Jones QL pages
- **Dithering kernels**: classic Floyd-Steinberg (1976), Atkinson (1984), Stucki (1981), Jarvis-Judice-Ninke (1976), Sierra, Burkes
- **Yliluoma algorithm**: Joel Yliluoma's color mixing article
- **Hilbert curve**: David Hilbert (1891); space-filling-curve dithering is a classic dithering alternative that minimises directional artifacts
- **Blue noise approximation**: Jorge Jimenez, "Interleaved Gradient Noise" (2014)

## License

GPL v3 — see [LICENSE](LICENSE).
