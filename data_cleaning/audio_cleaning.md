# Audio Cleaning

> **TL;DR** — Convert everything to a single canonical (sample rate, channels, bit depth, dtype) once, on load. After that, trim silence, fix clipping, remove DC offset, and normalize loudness — then deduplicate and audit labels.

## 1. Canonicalize on load

```python
import librosa
y, sr = librosa.load(path, sr=16000, mono=True)  # always 16 kHz mono
y = y.astype('float32')                          # range [-1, 1]
```

Choose one sample rate (16 kHz for speech, 22.05 kHz for general audio, 44.1 kHz for music) and resample everything to it.

## 2. Trim silence (VAD)

```python
y_trimmed, _ = librosa.effects.trim(y, top_db=30)
```

For speech, prefer a true voice-activity detector (WebRTC VAD, Silero VAD) which is robust to background noise.

## 3. Detect clipping

A signal that hit the dynamic range maximum has been truncated and introduces harmonic distortion.

```python
clipped_fraction = ((y == 1.0) | (y == -1.0)).mean()
```

If a non-trivial fraction is clipped, the file is damaged — either drop it or use a model trained with clipped data.

## 4. Remove DC offset

```python
y = y - y.mean()
```

Cheap consumer mics often introduce a constant offset that biases spectral features.

## 5. Loudness normalization

Three common levels:

- **Peak normalization** — scale so the max absolute amplitude is 1. Simple but doesn't account for perceived loudness.
- **RMS normalization** — scale so the root-mean-square equals a target. Better.
- **LUFS normalization** (ITU-R BS.1770) — perceptual standard used in broadcast and streaming. Best.

```python
import pyloudnorm as pyln
meter = pyln.Meter(sr)
loudness = meter.integrated_loudness(y)
y_norm = pyln.normalize.loudness(y, loudness, -23.0)  # target -23 LUFS
```

## 6. Resample carefully

```python
y_16k = librosa.resample(y_44k, orig_sr=44100, target_sr=16000)
```

Use a proper resampler (librosa, soxr, torchaudio). Naïve decimation introduces aliasing.

## 7. Channel handling

```python
if y.ndim == 2:
    y_mono = y.mean(axis=0)   # downmix
```

If you need stereo features later, save them; if not, downmix to mono early.

## 8. Deduplicate

Acoustic fingerprinting (Chromaprint via `pyacoustid`) groups perceptually identical audio even if files differ. Especially important for music and broadcast corpora.

## 9. Duration distribution

Plot:

```python
durations = [librosa.get_duration(filename=p) for p in paths]
```

Set min and max thresholds appropriate for your task — too-short files often lack content; too-long files OOM your batches.

## 10. Augmentation (training-time, not cleaning)

- Add background noise (MUSAN, DEMAND).
- Reverb (RIR convolution).
- Time stretching, pitch shifting.
- SpecAugment (mask time/frequency bins on the spectrogram).

Library: `audiomentations`, `torch-audiomentations`.

## 11. Checklist

- [ ] Resampled to a canonical rate.
- [ ] Mono (or explicit stereo policy).
- [ ] float32 in `[-1, 1]`.
- [ ] Trimmed silence (or applied VAD).
- [ ] Removed DC offset.
- [ ] Loudness-normalized to a chosen standard.
- [ ] Detected and handled clipping.
- [ ] Deduplicated by fingerprint.
- [ ] Audited a sample of labels.
