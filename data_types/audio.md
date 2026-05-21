# Audio Data

> **TL;DR** — Audio is a 1-D time series of pressure samples (the waveform), but you almost always work in a 2-D *time-frequency* representation (the spectrogram). The sample rate, bit depth, channel count, and windowing choices for the spectrogram all matter — pick defaults that match what your model was trained on.

## 1. Sub-types

| Sub-type | Description | Example |
|---|---|---|
| Speech | Human language | call recordings, dictation |
| Music | Multi-source, harmonic | streaming catalogs |
| Environmental | Mixed natural / urban | bird ID, urban noise |
| Industrial | Machine sounds | predictive maintenance |
| Bio-acoustic | Animal calls, heartbeat, lung | ecology, medical |
| MIDI / symbolic | Note events, not waveforms | music generation, transcription |

## 2. Core parameters

| Parameter | Typical values | Notes |
|---|---|---|
| Sample rate | 8 kHz (telephony), 16 kHz (speech), 22.05 / 44.1 kHz (music), 48 kHz (video) | Resample once at load |
| Bit depth | 16-bit PCM (CD), 24-bit (studio), 32-bit float | Convert to float32 in `[-1, 1]` for processing |
| Channels | mono, stereo, 5.1, ambisonic | Most models expect mono — downmix early |
| Duration | seconds to hours | Stream long files; don't load whole into RAM |
| Encoding | PCM (uncompressed), MP3, AAC, FLAC, Opus | Avoid lossy for training data |

## 3. Storage formats

| Format | Lossy? | Notes |
|---|---|---|
| WAV | No | PCM container, simple |
| FLAC | No | Lossless compressed, ~50% smaller than WAV |
| MP3 | Yes | Avoid for training; OK for storage |
| AAC / M4A | Yes | Common in video |
| Opus | Yes | Best low-bitrate; speech-friendly |
| OGG Vorbis | Yes | Open-source MP3 alternative |

## 4. Representations

### 4.1 Waveform
Raw amplitude over time. Shape: `(num_samples,)` or `(channels, num_samples)`.
- Use for end-to-end models like WaveNet, Wav2Vec2.
- Memory-heavy at high sample rates.

### 4.2 Spectrogram (STFT magnitude)
2-D matrix: frequency × time.
- Computed via Short-Time Fourier Transform.
- Key parameters: window size (e.g., 25 ms), hop length (e.g., 10 ms), window function (Hann).

### 4.3 Mel-spectrogram
Spectrogram remapped to the mel scale (perceptually uniform).
- Standard input to many ASR and audio classification models.
- Typical: 80 mel bands, 25 ms window, 10 ms hop.

### 4.4 MFCC
Cepstral coefficients on mel-spectrogram.
- Classic speech feature; less common now in deep learning.

### 4.5 Embeddings
- **VGGish, YAMNet** — general audio.
- **wav2vec 2.0, HuBERT, Whisper encoder** — speech, multilingual.
- **CLAP** — audio-text joint embedding.

## 5. Common pitfalls

1. **Sample-rate mismatch.** Model trained at 16 kHz fed 44.1 kHz audio produces garbage. Always resample.
2. **Lossy compression artifacts** that the model latches onto instead of the true signal.
3. **Silent files** counted as positive examples.
4. **Clipping** — peaks above the dynamic range get truncated, introducing harmonics. Check the max amplitude.
5. **Inconsistent mono/stereo handling** — averaging channels vs. taking left vs. taking right gives different results.
6. **DC offset** — non-zero mean amplitude, often from cheap hardware. Subtract before modeling.
7. **Label leakage by speaker / device.** Same speaker in train and test trivially inflates speech model accuracy.
8. **Background noise that correlates with class** (e.g., all "positive" examples recorded in the same room).

## 6. Cleaning checklist

See [`data_cleaning/audio_cleaning.md`](../data_cleaning/audio_cleaning.md).

- [ ] Convert all files to a single canonical (format, sample rate, channels).
- [ ] Trim leading / trailing silence (VAD).
- [ ] Detect and remove clipped segments.
- [ ] Normalize loudness (LUFS or simple peak / RMS normalization).
- [ ] Remove DC offset.
- [ ] Check duration distribution — set min / max duration thresholds.
- [ ] Deduplicate by audio fingerprint.

## 7. Visualizations

| Question | Chart |
|---|---|
| What does the signal look like? | Waveform plot |
| What frequencies are present over time? | Spectrogram, mel-spectrogram |
| What is the loudness distribution? | Histogram of RMS / LUFS per file |
| What is the duration distribution? | Histogram |
| Are clusters of similar audio? | t-SNE / UMAP of embeddings |

## 8. References

- Müller. *Fundamentals of Music Processing*.
- Jurafsky & Martin. *Speech and Language Processing*, Chap. 25–26.
- librosa documentation — https://librosa.org
