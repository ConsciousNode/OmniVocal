# OmniVocal

**Browser-native voice synthesis. Single file. Zero dependencies. Yours.**

OmniVocal is a complete neural text-to-speech system that runs entirely in your browser — no server, no API keys, no installation. Open the HTML file and speak.

Built by [ConsciousNode SoftWorks](https://consciousnode.github.io) on the xinu principle: the browser is bare metal. Constraint is the architecture.

---

## Try it

→ **[consciousnode.github.io/OmniVocal](https://consciousnode.github.io/OmniVocal)**

Or download [`omnivocal.html`](./omnivocal.html) and open it locally. Works offline.

---

## What's inside

| Component | Description |
|---|---|
| **G2P pipeline** | Text → phonemes. English (dictionary + rule fallback), Japanese, Korean, Spanish, German, French, Russian |
| **MVC acoustic model** | Bidirectional Mamba-style SSM, d=128, AdaLN conditioned on 10-dim voice vector |
| **Learned duration model** | 2-layer BitLinear MLP. Phoneme timing is *learned*, not table-based. Cosine interpolation between phoneme boundaries. |
| **Neural vocoder** | HiFi-GAN style, BitLinear ternary weights, 3 upsampling stages, GLU gating, learned snake α per channel |
| **Griffin-Lim fallback** | Always available via toggle. Useful for debugging mel quality independent of vocoder. |
| **Pop Studio** | Record your voice → analyze → train conditioning layers → export `.pop2` voice identity file |
| **`.pop2` format** | Portable voice identity: MVC weights, vocoder conditioning, duration model, snake alphas, SHA-256 provenance hash |
| **postMessage API** | Embed OmniVocal in an iframe and control it from any host application |

---

## Voice Space

10 dimensions. Each axis shapes synthesis directly:

| Axis | Effect |
|---|---|
| F0 Range | Pitch expressiveness (monotone → dramatic) |
| Breathiness | Air in the voice |
| Formant Shift | Timbre darkness ↔ brightness |
| Formant Agility | Articulation clarity |
| Spectral Tilt | Frequency balance |
| Mora Timing | Syllable duration feel |
| Vocal Weight | Body and fullness |
| Register | Head voice ↔ chest voice |
| Age Texture | Smoothness ↔ grain |
| F0 Center | Fundamental pitch range |

---

## Pop Studio workflow

1. **Record** — read any text aloud, 10–30 seconds
2. **Analyze** — YIN F0 estimation + spectral analysis → voice vector
3. **Train** — AdaLN conditioning + duration model adapt to your voice
4. **Export** — download your `.pop2` file
5. **Load** — bring your voice identity into any OmniVocal instance

---

## postMessage API

```js
// Embed in an iframe, then:
iframe.contentWindow.postMessage({
  target: 'omnivocal',
  action: 'speak',
  text: 'Hello from the host page'
}, '*');

// Available actions:
// speak · stop · getStatus · getVoiceVec · setVoiceVec
// loadPop2 · loadPop2Url · exportPop2 · reset

// OmniVocal fires these events to the parent:
// ready · synthStart · synthEnd · error
```

---

## Changelog

### v0.6-phase6 — 2026-06-07 · Vael Interim

- **Tabbed UI** — interface reorganized into tabs. Pop Studio surfaced as a first-class tab rather than a buried panel. Cleaner navigation across synthesis, training, analysis, and `.pop2` I/O workflows.
- **Playback cursor** — cursor tracks position across the mel spectrogram canvas during audio playback. Visual synchronization between audio and the spectrogram it came from.
- **Mel zoom** — zoom in/out controls on the mel canvas (± buttons, zoom label). `drawMelCanvas` patched to store data and redraw at current zoom level on resize or zoom change. The spectrogram is now inspectable at fine resolution.
- **Learned Snake α** — Snake activation parameter α is now learned per-layer rather than fixed at 1. `x + (1/α)·sin²(αx)` with α initialized to 1 and updated by optimizer. Gives the model more expressive control over activation curvature per layer.
- **Learned Duration MLP** — duration prediction now learned via a small MLP rather than rule-based phoneme length assignment. Duration model trained jointly with the acoustic model; produces more natural prosody on novel inputs.
- **Principled base init** — weight initialization tightened across all layers. Near-identity bias for SSM matrices, mel-projection for output layer, sinusoidal phoneme embeddings. More stable early training, faster convergence.
- **FFT voice analysis** — incoming voice recordings are analyzed via FFT before fine-tuning. Provides spectral fingerprint for voice identity conditioning.

## Architecture notes

- **BitLinear / TMAC** — all weight matrices are ternary (-1, 0, +1) with a learned scalar gamma. Packed 4-per-byte. Fast on any hardware, no GPU required.
- **No pre-trained weights** — everything initializes procedurally (near-identity bias for SSM matrices, mel-projection for output layer, sinusoidal phoneme embeddings). The base model ships as pure code.
- **Single file** — HTML + CSS + JS. No build step. No bundler. Fork it, read it, modify it.

---

## Built by

**Kham** (Khamerron Edward Ramsey Kizer) — architect and commissioner  
**Ed Interim** — phases 1, 2, 5, integration  
**Vael Interim** — phases 3, 4, 6  

Part of [ConsciousNode SoftWorks](https://consciousnode.github.io) — computational folk art for the browser age.

---

## License

MIT. Take it, break it, build on it.
