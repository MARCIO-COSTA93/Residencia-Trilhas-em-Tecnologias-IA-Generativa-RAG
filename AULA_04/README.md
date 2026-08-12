# 🎙️ Speech Transcription & Audio Analysis

A complete pipeline for **speech-to-text transcription** with audio preprocessing, data augmentation, and visual analysis — built as a foundation for Keyword Spotting (KWS) and Automatic Speech Recognition (ASR) systems.

[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://python.org)
[![TensorFlow](https://img.shields.io/badge/TensorFlow-2.x-orange.svg)](https://tensorflow.org)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Overview

This project demonstrates a production-oriented audio preprocessing and transcription pipeline using the `SpeechRecognition` library, with techniques directly applicable to real-world voice assistant and keyword detection systems.

### Key Features

- **Multi-format support**: WAV, OGG, OPUS, MP3, FLAC (including WhatsApp voice messages)
- **Ambient noise adjustment**: Calibrates energy threshold for robust recognition in noisy environments
- **Data augmentation**: Gaussian noise injection, time stretching, and pitch shifting
- **Visual analysis**: Waveform and spectrogram visualization before/after preprocessing
- **Engine comparison**: Online (Google Web Speech API) vs offline (CMU Sphinx) recognition
- **Batch processing**: Transcribe entire directories of audio files automatically

---

## Why This Matters for KWS Systems

Understanding how raw audio is preprocessed and transcribed is foundational for building **Keyword Spotting (KWS)** pipelines. Key concepts demonstrated here:

| Concept | KWS Application |
|---|---|
| 16kHz resampling | Standard input for most KWS/ASR neural networks |
| Noise calibration | Reduces false positives in always-on detection |
| Spectrogram generation | Visual representation processed by CNNs/RNNs |
| Data augmentation | Improves model robustness to real-world conditions |
| Offline vs online | Edge deployment trade-offs for embedded KWS |

---

## Technical Stack

- **Speech Recognition**: SpeechRecognition (Google Web Speech API, CMU Sphinx)
- **Audio Processing**: librosa, soundfile, pydub
- **Data Augmentation**: audiomentations
- **Visualization**: matplotlib, librosa.display
- **Format Conversion**: ffmpeg, pydub

---

## Installation

```bash
git clone https://github.com/MARCIO-COSTA93/speech-transcription.git
cd speech-transcription

pip install -r requirements.txt
apt-get install -y ffmpeg
```

---

## Usage

Open the notebook in Google Colab:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com)

Or run locally:

```bash
jupyter notebook speech_transcription.ipynb
```

### Quick Example

```python
# Transcribe any audio file
result = process_audio_file('audio.ogg', language='pt-BR')

# Batch transcribe WhatsApp voice messages
results = transcribe_whatsapp_messages('/path/to/audios/', language='pt-BR')

# Compare recognition engines
google_result, sphinx_result = compare_engines('audio.wav')

# Analyze effect of noise adjustment
noise_adjustment_comparison('audio.wav', language='pt-BR')
```

---

## Project Structure

```
speech-transcription/
├── speech_transcription.ipynb   # Main notebook
├── requirements.txt             # Dependencies
└── README.md                    # This file
```

---

## Key Observations

**Noise adjustment is critical**: Ambient noise calibration significantly reduces false positives — the same principle applies to always-on KWS systems where background noise triggers are a key challenge.

**Online vs Offline trade-offs**:
- Google Web Speech API: high accuracy, requires internet, ~500ms latency
- CMU Sphinx: lower accuracy, fully offline, suitable for embedded devices

**Data augmentation improves robustness**: Models and pipelines trained/tested with augmented data (noise, speed variations) generalize better to unseen real-world conditions.

---

## Related Projects

- [Audio Keyword Detector](https://github.com/MARCIO-COSTA93/audio-keyword-detector) — YAMNet-based acoustic event classification with sliding window analysis

---

## Author

**Marcio da Costa Oliveira**  
- LinkedIn: [marcio-costa-oliveira](https://linkedin.com/in/marcio-costa-oliveira)  
- GitHub: [MARCIO-COSTA93](https://github.com/MARCIO-COSTA93)

---

## License

MIT License - Free for educational and commercial use.
