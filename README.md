# Retro Pixel Converter

A browser-based image converter for classic 8-bit and 16-bit machines. Converts JPG/PNG images to native graphics formats for the ZX Spectrum, Timex/Sinclair 2068, Commodore 64, Atari 800, and Sinclair QL.

**[Try it live →](https://factus10.github.io/retro-pixel-converter/)**

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

**[Try it live ->](https://factus10.github.io/retro-pixel-converter/)**

## Supported Display Modes

| Machine | Mode | Pixels | Attribute / block size | Colors | Visible border | File |
|---|---:|---:|---:|---|---|---|
| ZX Spectrum | Standard | 256x192 | 8x8 | 2 of 16, ZX bright-black behavior | 32/32/24/24 | `.scr` / `.tap` |
| TS 2068 | Standard | 256x192 | 8x8 | 2 of 16, Timex bright black is dark gray | 32/32/24/24 | `.scr` / `.tap` |
| TS 2068 | Extended Color Mode | 256x192 | 8x1 | 2 of 16 per strip | 32/32/24/24 | `.scr` / `.tap` |
| TS 2068 | 64-column hi-res | 512x192 | global | 8 hardware ink/paper pairs | 64/64/24/24 | `.scr` / `.tap` |
| C64 | Hi-res bitmap NTSC/PAL | 320x200 | 8x8 | 2 of 16 | mode-specific | `.prg` |
| C64 | Multicolor / Koala NTSC/PAL | 160x200 | 4x8 | 4 of 16 + auto global background | mode-specific | `.kla` |
| Atari 800 | GR.15 / ANTIC E | 160x192 | global | 4 of 128 | 8/8/24/24 | `.mic` |
| Atari 800 | GR.8 | 320x192 | global | 2 user-picked colors | 16/16/24/24 | `.gr8` |
| Atari 800 | GR.9 | 80x192 | per-pixel | 16 luma shades of one hue | 4/4/24/24 | `.gr9` |
| QL | Mode 8 / Low res | 256x256 | per-pixel | 8 | none | `_scr` |
| QL | Mode 4 / Hi-res | 512x256 | per-pixel | 4 fixed colors | none | `_scr` |
| Pico-8 | Standard palette | 128x128 | per-pixel | 16 fixed | 4:3 side border | `.bin` + optional hex `.txt` |

Visible borders are listed as left/right/top/bottom in mode pixels. Border color palettes are mode-aware: ZX and most Timex modes use the 8 basic non-bright colors, TS 2068 hi-res follows the selected paper color, C64 modes use the active C64 palette, and modes without known border color control default to black.

Custom modes can also be imported from JSON at runtime. PNG/JPG export works for custom modes; binary export requires mode-specific exporter code.

## Rendering And Color Pipeline

- Images are loaded and processed in linear-light color space. Source adjustment, resampling, preview rendering, block-selection blur, output visualization, and linear-to-sRGB display conversion are handled through WebGL2 where appropriate.
- Dithering and quantization operate on linear pixels, then final converted pixels are resolved back to sRGB for display and export.
- Output geometry is derived from each mode's addressable pixel area plus visible border, then mapped to a 4:3 display. Pixel aspect is calculated from that full visible frame rather than stored as a separate mode constant.
- The output starts with a 320x240 TV test pattern so scale, border, and CRT sizing are visible before an image is loaded.
- A modern browser with WebGL2 support is required. There is no build step and no runtime dependency download.

## Workflow Features

- Drag-and-drop or browse for an image, then crop interactively in the input panel.
- Resize the input/output split manually with the center divider.
- Lock crop aspect ratio, stretch the selected crop to the target resolution, or turn Stretch off and fill side bars with the configured RGB crop fill color. Filled bars are part of the target image and can be dithered like any other pixels.
- Use brightness, contrast, saturation, gamma, block pre-blur, dithering, palette, and search controls to tune the conversion.
- Inspect available colors with the mode-aware palette strip and disable individual colors when needed.
- Show an attribute grid overlay for block-based modes.
- Snapshot the current output and switch the output panel between `Active` and `Saved` using the header radio toggle. The saved snapshot reuses the normal output canvas, CRT, border, scale, grid, and fullscreen paths.
- Export/import profiles containing current controls, crop state, disabled colors, crop fill color, CRT settings, and border selection.

## CRT, Borders, And Output Display

The output panel uses a `Scale` control, mode-specific pre-scaling where required, and a shader-based CRT simulation. CRT controls include scanlines, brightness, contrast, saturation, RF noise, horizontal bleed, bloom, bloom spread/threshold, and vignette. Range controls are live: a value of `0` disables effects such as noise, bleed, bloom, and vignette.

The border selector is available in the output header for modes with visible borders. The app starts with Scale `1.5` and a black border selected. Border selection persists when switching through borderless modes such as QL.

Fullscreen output renders the selected `Active` or `Saved` image at fullscreen canvas resolution. The decorative CRT frame is not shown in fullscreen, but any enabled border remains because it is rendered inside the output canvas.

## Dithering And Color Search

Dithering options include Floyd-Steinberg, Atkinson, Stucki, Jarvis-Judice-Ninke, Burkes, Sierra Lite, Hilbert-curve Floyd-Steinberg, Bayer 4x4, Bayer 8x8, blue-noise-style interleaved gradient noise, Yliluoma-style projection, halftone, and no dither.

Color search strategies are mode-aware:

- **Weighted average pair fit** is the default for many two-color block modes. It scores candidate color pairs by how well a blend can match the block average.
- **Per-block best-fit** exhaustively evaluates palette combinations for block modes where that is practical.
- **Global pre-quantize** is used for modes such as Atari GR.15 where exhaustive search is too large.
- **Pixel-direct** is used for per-pixel modes such as QL and Atari GR.9.
- **User-picked** is used for hardware-constrained monochrome modes such as TS 2068 64-column and Atari GR.8.

Threshold-based dithers nudge non-uniform blocks toward distinct color pairs so gradients can dither instead of collapsing into flat color.

## Display Geometry

The app displays each mode as part of a 4:3 visible screen, using border dimensions to derive the pixel aspect ratio:

| Mode | Full visible frame | Derived pixel aspect |
|---|---:|---:|
| ZX / TS standard / ECM | 320x240 | 1.000 |
| TS 2068 64-column | 640x240 | 0.500 |
| C64 hi-res NTSC | 418x235 | 0.750 |
| C64 multicolor NTSC | 209x235 | 1.499 |
| C64 hi-res PAL | 362x246 | 0.906 |
| C64 multicolor PAL | 181x246 | 1.812 |
| Atari GR.15 | 176x240 | 1.818 |
| Atari GR.8 | 352x240 | 0.909 |
| Atari GR.9 | 88x240 | 3.636 |
| QL Mode 8 | 256x256 | 1.333 |
| QL Mode 4 | 512x256 | 0.667 |
| Pico-8 | 170.667x128 | 1.000 |

Pico-8 also uses an output pre-scale of `2` before output effects, so CRT-style processing works on a larger image without changing the exported 128x128 pixel data.

## Export Formats

Image export produces sharp PNG/JPG files at 1x, 2x, or 4x with the active conversion's pixel aspect correction. Binary export data is built lazily when a download is requested, so routine preview updates do not rebuild `.scr`, `.tap`, `.prg`, `.kla`, and other binary payloads.

| Format | Output |
|---|---|
| ZX Spectrum SCREEN$ | `.scr`, 6912 bytes |
| TS 2068 ECM | `.scr`, 12288 bytes |
| TS 2068 64-column | `.scr`, 12288 bytes |
| C64 hi-res bitmap | `.prg`, load address + bitmap + screen RAM |
| C64 Koala | `.kla`, load address + bitmap + screen RAM + color RAM + background |
| Atari MicroPainter | `.mic`, bitmap + 4 color register values |
| Atari GR.8 | `.gr8`, raw 1 bit/pixel bitmap |
| Atari GR.9 | `.gr9`, raw 4 bits/pixel luma bitmap |
| QL Mode 4 / Mode 8 | `_scr`, direct QL screen memory dump |
| Pico-8 | `.bin` and optional hex `.txt` |

ZX/Timex modes additionally support `.tap` tape-image export with correctly addressed CODE blocks.

## Hardware Notes

ZX/Timex: the `.tap` export includes CODE blocks at the correct addresses. For ECM/64-column modes, set the video mode first (`OUT 255,2` for ECM, `OUT 255,6+(ink<<3)` for 64-column) before loading.

C64: `.prg` and `.kla` files use standard bitmap/Koala-style layouts and can be loaded by compatible viewers or tools.

Atari 800: `.mic` loads in MicroPainter-compatible tools. `.gr8` and `.gr9` are raw display-memory images.

QL: `_scr` is a direct dump of screen RAM at base `$20000`, loadable in common QL emulators with commands such as `LBYTES file,131072`.

## Running Locally

Open `index.html` directly in a browser, or serve the repository with any static file server:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

There is no package install, bundler, build step, generated output, or server-side processing.

## Credits And References

Special thanks to **Josef Jelinek** for feedback and technical guidance, including palette and color-distance fixes, Timex bright-black behavior, TS 2068 64-column constraints, generalized mode configuration, CRT/output workflow, linear-space processing, and retro display geometry.

References:

- **ZX Spectrum / TS 2068**: Sinclair and Timex original hardware manuals, including TS 2068 Technical Manual section 5.2
- **C64 palette**: Philip "Pepto" Timmermann's Pepto / Colodore references
- **Atari palette**: NTSC-style approximation inspired by Altirra and related references
- **QL screen layout**: RWAP Software QL Technical Guide and Dilwyn Jones QL references
- **Dithering kernels**: Floyd-Steinberg, Atkinson, Stucki, Jarvis-Judice-Ninke, Sierra, Burkes
- **Yliluoma algorithm**: Joel Yliluoma's color mixing article
- **Hilbert curve**: David Hilbert's space-filling curve, applied here as an error-diffusion traversal
- **Blue-noise approximation**: Jorge Jimenez, "Interleaved Gradient Noise"

## License

GPL v3 - see [LICENSE](LICENSE).
