# Low-level background audio filter latency benchmark

Measured on GitHub-hosted Apple Silicon using the exact candidate `StreamingSpeechActivityGate` implementation.

## Results

- Typical FluidVoice callback after 48 kHz to 16 kHz conversion: 1,365 samples (about 85.312 ms)
- Processing time over 2,000 warmed iterations:
  - median: 102.375 microseconds
  - p95: 138.375 microseconds
  - p95 share of callback budget: 0.16220%
- Audio-delivery activation effect by post-resample packet size:
  - 160 samples / 10 ms: +30 ms
  - 320 samples / 20 ms: +20 ms
  - 640 samples / 40 ms: 0 ms
  - 1,365 samples / about 85.312 ms: 0 ms

The 160 ms pre-roll retains the speech onset even when delivery waits for the second active 20 ms frame.

## Validation

- Strict SwiftLint passed.
- Full macOS/arm64 Xcode suite passed: 540 tests, 0 failures.
- The performance assertions use a deliberately generous 2 ms p95 ceiling to catch pathological regressions without making hosted-runner load a source of flakes.

## Method

The CPU benchmark opens and warms the gate, then times 2,000 calls with a 1,365-sample active packet. The onset benchmark feeds active packets at 10, 20, 40, and about 85 ms packet sizes until the first output is delivered, comparing that callback boundary with the ungated first-packet baseline.

Private real-world recordings were used only for local validation and are not published.
