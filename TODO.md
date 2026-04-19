# Speculative improvements — to test

Ideas for further exploration. None of these are blockers; several might not yield improvements once tried.

## Dither / color-pattern quality

- **Blue-noise mask ordered dither** — replace Jorge Jimenez's interleaved gradient noise (current "Blue noise" option) with a pre-computed void-and-cluster blue noise texture, tiled. True blue noise is perceptually better than IGN.
- **Riemersma's original weighted-moving-average dither** — my Hilbert-FS uses FS-style weights along the curve; Riemersma's original propagates errors via a weighted moving average of recent errors along the curve (exponential falloff). Often smoother output.
- **Pattern-based stippling LUT** — pre-compute optimal 8×8 stipple patterns for each blend ratio 0/64, 1/64, …, 63/64. Look up per block based on the desired mix. Highest-quality static dither.

## Global optimization / palette selection

- **Simulated annealing for Atari GR.15 4-of-128** — seed with k-means, then anneal by swapping palette entries and accepting reductions in total image error. Likely marginal improvement over k-means.
- **Joint attribute+dither optimization** — select each block's colors INSIDE the dither loop, accounting for already-diffused error from prior blocks. Theoretical optimum but expensive; needs restructure.

## Specialty / experimental

- **Dizzy dithering** — randomized multi-pass approach with slow convergence. High-quality but slow. Suitable as a "best effort" mode.
- **CIEDE2000** color distance — gold-standard perceptual metric. More complex than OkLab, incrementally better. Worth testing if OkLab doesn't provide enough improvement.
- **Adaptive space-filling-curve error diffusion** — Hilbert variants that adapt to image content.

## UI / UX

- **Multiple simultaneous outputs with different settings for live A/B** — current A/B snapshot freezes one side; a "compare two live configurations" mode would show both updating as settings change.

## Already tried / shipped

See git log and README.
