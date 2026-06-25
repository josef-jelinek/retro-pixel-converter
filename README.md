# Retro Pixel Converter

A dependency-free browser image converter for classic 8-bit and 16-bit machines. It converts JPG/PNG images into native graphics formats for the ZX81, ZX Spectrum, Timex/Sinclair 2068, Commodore 64, Atari 800, Sinclair QL, and Pico-8.

**[Try it live ->](https://factus10.github.io/retro-pixel-converter/)**

## Supported Display Modes

| Machine | Mode | Pixels | Attribute / block size | Colors | Visible border | File |
|---|---:|---:|---:|---|---|---|
| ZX Spectrum | Standard | 256x192 | 8x8 | 2 of 16, ZX bright-black behavior | 32/32/24/24 | `.scr` / `.tap` |
| ZX81 | Character graphics equalized / linear | 256x192 | 8x8 character cells | 64 glyphs + inverse video | 32/32/24/24 | `.zx8` |
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

Visible borders are listed as left/right/top/bottom in mode pixels. Border color palettes are mode-aware: ZX81 uses a fixed white border, ZX Spectrum and most Timex modes use the 8 basic non-bright colors, TS 2068 hi-res follows the selected paper color, C64 modes use the active C64 palette, and modes without known border color control default to black.

Custom modes can also be imported from JSON at runtime. PNG/JPG export works for custom modes; binary export requires mode-specific exporter code.

## Rendering And Color Pipeline

- Images are loaded and processed in linear-light color space. Source adjustment, resampling, preview rendering, block-selection blur, output visualization, and linear-to-sRGB display conversion are handled through WebGL2 where appropriate.
- Transparent source pixels keep their RGB values; alpha is ignored instead of compositing the image over black before conversion.
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

Fullscreen output renders the selected `Active` or `Saved` image with devicePixelRatio-aware backing resolution. The decorative CRT frame is not shown in fullscreen, but any enabled border remains because it is rendered inside the output canvas.

## Dithering And Color Search

Dithering options include Floyd-Steinberg, Atkinson, Stucki, Jarvis-Judice-Ninke, Burkes, Sierra Lite, Hilbert-curve Floyd-Steinberg, Bayer 4x4, Bayer 8x8, blue-noise-style interleaved gradient noise, Yliluoma-style projection, halftone, and no dither.

Color search strategies are mode-aware:

- **Weighted average pair fit** is the default for many two-color block modes. It scores candidate color pairs by how well a blend can match the block average.
- **Per-block best-fit** exhaustively evaluates palette combinations for block modes where that is practical.
- **Global pre-quantize** is used for modes such as Atari GR.15 where exhaustive search is too large.
- **Pixel-direct** is used for per-pixel modes such as QL and Atari GR.9.
- **User-picked** is used for hardware-constrained monochrome modes such as TS 2068 64-column and Atari GR.8.
- **ZX81 character fit** converts the image to 32x24 fixed character cells and picks the closest normal or inverse-video glyph using multi-scale grayscale intensity matching. The `equalized` sub-mode applies the sRGB transfer to source luma before matching; the `linear` sub-mode matches directly in linear luma.

Threshold-based dithers nudge non-uniform blocks toward distinct color pairs so gradients can dither instead of collapsing into flat color.

## Display Geometry

The app displays each mode as part of a 4:3 visible screen, using border dimensions to derive the pixel aspect ratio:

| Mode | Full visible frame | Derived pixel aspect |
|---|---:|---:|
| ZX / TS standard / ECM | 320x240 | 1.000 |
| ZX81 | 320x240 | 1.000 |
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

Image export produces sharp PNG/JPG files at 1x, 2x, or 4x with the active conversion's pixel aspect correction. PNG/JPG export reuses a WebGL2 export renderer instead of allocating a new context for each download. Binary export data is built lazily when a download is requested, so routine preview updates do not rebuild `.scr`, `.tap`, `.prg`, `.kla`, and other binary payloads.

| Format | Output |
|---|---|
| ZX81 character screen | `.zx8`, 768 character-code bytes |
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

ZX81: `.zx8` is a raw 32x24 row-major character-code screen using normal codes `0..63` and inverse-video codes `128..191`.

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
