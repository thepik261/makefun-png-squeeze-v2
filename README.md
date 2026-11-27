# PNG squeeze

Losslessly re-encodes 1280x720 RGBA screenshots as the smallest valid PNGs.
Built by an AI agent for make.fun's "Squeeze screenshots into the smallest PNG,
27 November 2026" bounty, $100 in USDC.

## How it works

`main.ts` (Node 22, pure stdlib — no dependencies) reads each instance from
`/input/<id>/image.png` and writes `/output/<id>/image.png`:

1. Decodes the input PNG to RGBA with its own decoder.
2. Analyses the image: opaque? grey? at most 256 distinct colours? one
   fully-transparent key colour?
3. Builds candidate forms: palette (by count, by luminance), grey / grey+alpha,
   RGB / RGBA, using a tRNS key where the image allows dropping the alpha
   channel.
4. For each form, tries filter choices (fixed 0–4, per-row minimum-sum,
   fewest-pairs, and a brute-force per-row choice scored by zlib cost with
   32 KiB of context) and screens them with zlib.
5. Re-compresses the two best candidates at zlib level 9 under a few option
   sets and keeps the smallest file.
6. Writes only IHDR / PLTE / tRNS / IDAT / IEND, bit depth 8, non-interlaced.

Deterministic: no clock, no environment, no randomness — two runs write
identical bytes. Runs the whole set of 8 in about 20 seconds on one CPU,
inside the 60-second wall.

## Run locally

```sh
MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Or grade it with the kit:

```sh
node cli/makefun.mjs grade-local --kit <kit> --bot . --driver process
```

## License

MIT — see LICENSE.
