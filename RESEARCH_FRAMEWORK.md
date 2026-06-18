# Research Framework: Privacy-Preserving Multimodal Emotion Detection for Mood-Based Food Recommendation

> **Project Scope:** Embedded system for real-time emotion detection using audio, text, and typing-behavior analysis—with optional, consent-gated image processing—to generate personalized, mood-aware food recommendations without violating user privacy.

---

## Executive Summary

The **Emotion** platform implements a privacy-first, multimodal emotion recognition system that operates entirely on-device. Unlike traditional emotion detection systems that rely solely on facial recognition (requiring constant camera access), our approach combines four complementary modalities:

1. **Audio-Based Emotion Recognition (SER)** - Detects emotional tone from voice
2. **Text Content Sentiment Analysis (NLP)** - Analyzes linguistic sentiment
3. **Typing Behavior & Keystroke Dynamics** - Measures typing patterns and stress signals
4. **Image-Based Facial Emotion Recognition (FER)** - Optional, consent-gated facial analysis

This multi-layered approach provides:
- **Higher accuracy** through cross-modal validation
- **Privacy preservation** via on-device processing
- **Graceful degradation** when modalities are unavailable
- **User control** through consent-gating of sensitive modalities

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Modality 1: Audio-Based Emotion Recognition](#2-audio-based-emotion-recognition)
3. [Modality 2: Text Sentiment Analysis](#3-text-sentiment-analysis)
4. [Modality 3: Typing Behavior Analysis](#4-typing-behavior-analysis)
5. [Modality 4: Facial Emotion Recognition](#5-facial-emotion-recognition-optional)
6. [Multimodal Fusion Strategy](#6-multimodal-fusion-strategy)
7. [Mood-to-Food Recommendation Engine](#7-mood-to-food-recommendation-engine)
8. [Privacy Framework & Legal Compliance](#8-privacy-framework)
9. [Hardware & Deployment](#9-hardware--deployment)
10. [Datasets & Benchmarks](#10-datasets--benchmarks)

---

## 1. System Architecture

### 1.1 High-Level Architecture

```
┌──────────────────────────────────────────────────────────┐
│              📱 USER INTERFACE LAYER                      │
│         (React/Vue/Flutter/React Native)                 │
└────────────────────┬─────────────────────────────────────┘
                     │
┌────────────────────▼──────────────────────────────────────┐
│  🔒 ON-DEVICE PROCESSING (Privacy Boundary)              │
├───────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─ Audio Pipeline ──────────────────┐                   │
│  │ • Noise Suppression (RNNoise)     │                   │
│  │ • MFCC + Prosody Extraction       │                   │
│  │ • SER Model (1D-CNN/DistilHuBERT)│                   │
│  └─────────────────────────────────┘                    │
│                                                            │
│  ┌─ Text Pipeline ───────────────────┐                   │
│  │ • Tokenization                    │                   │
│  │ • Sentiment Analysis              │                   │
│  │ • NLP Model (DistilBERT/TinyBERT) │                   │
│  └─────────────────────────────────┘                    │
│                                                            │
│  ┌─ Typing Behavior Pipeline ────────┐                   │
│  │ • Keystroke Dynamics Capture      │                   │
│  │ • Dwell/Flight Time Features      │                   │
│  │ • Behavioral Classifier (LightGBM)│                   │
│  └─────────────────────────────────┘                    │
│                                                            │
│  ┌─ Image Pipeline (Optional) ───────┐                   │
│  │ • Consent Gate: User Opt-In       │                   │
│  │ • Face Detection (BlazeFace)      │                   │
│  │ • FER Model (MobileNet-v3)        │                   │
│  └─────────────────────────────────┘                    │
│                                                            │
│        ↓ ↓ ↓ ↓                                             │
│                                                            │
│  ┌──────────────────────────────────┐                    │
│  │  Attention-Based Multimodal Fusion│                   │
│  │  (Dynamic Weighting by Confidence)│                   │
│  └─────────────┬────────────────────┘                    │
│                │                                          │
│        ┌───────▼──────────┐                              │
│        │ Mood State Output │                              │
│        └───────┬──────────┘                              │
└────────────────┼───────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│         🍽️ RECOMMENDATION ENGINE                        │
├────────────────────────────────────────────────────────┤
│  • Mood → Food Category Mapping                        │
│  • Nutritional Database Filtering                      │
│  • Preference/Dietary Constraints                      │
│  • Personalized Food Suggestions                       │
└────────────────────────────────────────────────────────┘
```

### 1.2 Key Design Principles

| Principle | Implementation | Benefit |
|:---|:---|:---|
| **Privacy-First** | All inference on-device; no raw data transmission | GDPR/DPDP compliance; user trust |
| **Consent-Gated** | Camera access requires per-session opt-in | User control; no background access |
| **Non-Intrusive** | Audio/text/typing work during normal usage | Passive emotion detection |
| **Graceful Degradation** | System works with any subset of modalities | Flexibility; handles missing signals |
| **Health-Conscious** | Recommendations counter emotional eating | Promotes nutritional wellness |

---

## 2. Audio-Based Emotion Recognition

### 2.1 Overview

**Speech Emotion Recognition (SER)** analyzes vocal characteristics to detect emotional state from **how** the user speaks, not **what** they say.

### 2.2 Feature Extraction

| Feature | Description | Emotional Correlation |
|:---|:---|:---|
| **MFCC** | Mel-frequency cepstral coefficients (13-40 coefficients) | Captures timbral qualities of emotions |
| **Prosody** | Pitch, energy, speaking rate, pauses, jitter, shimmer | High pitch + fast rate = excitement; low pitch + slow = sadness |
| **Spectrograms** | 2D time-frequency representations | Enables CNN-based pattern recognition |
| **Pre-trained Embeddings** | Wav2Vec2, HuBERT representations | Transfer learning from large speech datasets |

### 2.3 Recommended Models

#### Option A: Lightweight (MCU/Mobile)
- **Architecture:** 1D-CNN (3 layers) + BiLSTM (128 units)
- **Input:** 40-dim MFCC + Δ + ΔΔ features
- **Output:** 7 emotions [happy, sad, angry, neutral, fearful, surprised, disgusted]
- **Size:** 2–5 MB
- **Latency:** <50ms on ARM Cortex-A73
- **Use Case:** Mobile phones, embedded systems

#### Option B: Higher Accuracy (Smartphone/SBC)
- **Architecture:** DistilHuBERT (quantized INT8)
- **Fine-tuning:** 2-layer MLP classification head
- **Output:** 7 emotions
- **Size:** 25–40 MB
- **Latency:** <100ms on Snapdragon 8xx
- **Use Case:** Modern smartphones, single-board computers

### 2.4 Pipeline Implementation

```python
┌─────────────────────────────────────────┐
│ Raw Audio Input (16kHz, mono)           │
└────────────────┬────────────────────────┘
                 │
         ┌───────▼────────┐
         │ Voice Activity  │
         │ Detection (VAD) │
         └───────┬────────┘
                 │
         ┌───────▼────────────────┐
         │ Noise Suppression      │
         │ (RNNoise / Spectral)   │
         └───────┬────────────────┘
                 │
         ┌───────▼──────────────────┐
         │ Feature Extraction       │
         │ • MFCC (40 coefficients) │
         │ • Prosodic features      │
         └───────┬──────────────────┘
                 │
         ┌───────▼────────────────┐
         │ SER Model Inference    │
         └───────┬────────────────┘
                 │
    ┌────────────▼────────────────┐
    │ [happy, sad, angry, ...]    │
    │ Confidence: 85%             │
    └─────────────────────────────┘
```

### 2.5 Privacy Considerations

- ✅ **Raw audio never stored** - Only features computed in streaming buffer
- ✅ **Immediate purging** - Waveforms discarded after feature extraction
- ✅ **No voice identification** - Emotional tone only, speaker identity not captured
- ✅ **Cultural awareness** - Train on diverse datasets to handle regional variations

---

## 3. Text Sentiment Analysis

### 3.1 Overview

Analyzes **textual content** from user inputs (search queries, chat messages, notes) to infer emotional state from linguistic sentiment.

### 3.2 Approach Comparison

| Approach | Model | Size | Accuracy | Edge Suitability |
|:---|:---|:---|:---|:---|
| **Lexicon-Based** | VADER, SentiWordNet | <1 MB | 65–72% | ✅ Excellent |
| **Classical ML** | TF-IDF + SVM | 5–15 MB | 72–78% | ✅ Excellent |
| **Distilled Transformer** | DistilBERT (INT8) | 60–80 MB | 82–88% | ⚠️ Mobile/SBC |
| **Tiny Transformer** | TinyBERT / MobileBERT | 15–25 MB | 78–84% | ✅ Good |
| **Cloud LLM** | GPT-4 / Gemini API | Cloud | 90%+ | ❌ Privacy risk |

### 3.3 Two-Tier Strategy

```
TIER 1 (Always-On, <5ms latency):
├─ VADER Sentiment Analyzer
├─ Quick polarity: positive, negative, neutral
└─ Runs on every text input

TIER 2 (On sufficient text, ~20+ words, ~30-50ms latency):
├─ TinyBERT (INT8 quantized)
├─ Fine-grained emotions: joy, sadness, anger, fear, etc.
└─ Weight increases with text length

FUSION:
└─ Weighted combination of Tier 1 + Tier 2
```

### 3.4 Text Sources & Signals

| Source | Example | Signal Strength |
|:---|:---|:---|
| Search queries | "spicy comfort food", "healthy options" | 🟡 Medium |
| Chat/voice-to-text | "I'm feeling so tired today" | 🟢 Strong |
| Food reviews | "This always cheers me up!" | 🟢 Strong |
| Order comments | "Quick, anything warm" | 🟡 Medium |

### 3.5 Privacy Considerations

- ✅ **No text storage** - Only emotion labels retained
- ✅ **Multilingual support** - Language detection + region-specific models
- ✅ **Context window** - Analyze last 3-5 inputs for temporal tracking
- ✅ **Sarcasm handling** - Flag low-confidence contradictions

---

## 4. Typing Behavior Analysis

### 4.1 Overview

The **most privacy-friendly** modality. Analyzes **how** users type (keystroke dynamics) as behavioral biometrics correlated with emotional state. Requires NO additional hardware.

### 4.2 Feature Set

| Feature | Description | Emotional Signal |
|:---|:---|:---|
| **Dwell Time** | Key press duration (ms) | Stress → longer dwell times |
| **Flight Time** | Time between key releases | Anxiety → shorter, erratic flights |
| **Typing Speed (WPM)** | Words per minute | Excitement → faster; sadness → slower |
| **Error Rate** | Backspace/delete frequency | Stress/anger → higher error rate |
| **Pause Patterns** | Pauses between words/sentences | Contemplation → longer pauses |
| **Pressure (mobile)** | Touch screen pressure | Anger → harder presses |
| **Burst Patterns** | Rapid typing + pauses | Anxiety → bursty patterns |

### 4.3 Model Implementation

```
Input Stream: Keystroke events (timestamp, keycode, type)
        │
        ▼
Feature Window: Last 30 seconds / 50 keystrokes
        │
        ├─ Statistical aggregation: mean, std, percentiles
        ├─ Temporal derivatives: acceleration
        └─ Rolling features (~25 dimensions)
        │
        ▼
LightGBM Classifier (or Random Forest)
        │
        ▼
Output: [calm, stressed, excited, fatigued, frustrated]
        │
        ├─ Confidence score: 0-100%
        └─ Latency: <5ms
```

### 4.4 Personalization Strategy

- **Baseline Calibration:** First 1-2 weeks establish user's normal typing patterns
- **Device Adaptation:** Separate baselines for keyboard vs. touchscreen vs. tablet
- **Deviation Detection:** Flags significant deviations from baseline
- **Privacy:** Only timing features—actual keystrokes never logged

### 4.5 Advantages

| Advantage | Details |
|:---|:---|
| **Universal Availability** | Works on any keyboard/touchscreen input |
| **Zero Additional Hardware** | Uses standard keyboard/touch interface |
| **Continuous Passive Monitoring** | No explicit user action required |
| **High Privacy** | Only timing data, no character content |
| **Resistant to Spoofing** | Difficult to fake over extended periods |

---

## 5. Facial Emotion Recognition (Optional)

### 5.1 Consent Flow

```
┌─ App Opened ──────────────────────────────────┐
└────────────────┬─────────────────────────────┘
                 │
    ┌────────────▼────────────┐
    │ Camera FER Enabled?     │
    └────┬────────────────┬───┘
    ┌────▼────┐    ┌──────▼──────┐
    │ NO       │    │ YES         │
    │(Default) │    │(Opt-in)     │
    └────┬────┘    └──────┬──────┘
         │                │
    ┌────▼──────┐    ┌────▼─────────────────┐
    │ Use Audio  │    │ Show Privacy Notice: │
    │ + Text +   │    │ • What's captured   │
    │ Typing     │    │ • How processed     │
    └────────────┘    │ • How to revoke     │
                      └────┬─────────────────┘
                           │
                      ┌────▼─────┐
                      │ Confirm?  │
                      └┬──────────┬┘
                   ┌──▼┐    ┌──▼──┐
                   │YES│    │ NO  │
                   └──┬┘    └──┬──┘
                      │        │
                  ┌───▼──┐  ┌──▼───┐
                  │Start │  │Revert│
                  │FER   │  │to    │
                  │      │  │A+T+T │
                  └───┬──┘  └──────┘
                      │
                  ┌───▼──────────────┐
                  │ Session-Only Use │
                  │ Camera Released  │
                  │ After Session    │
                  └───────────────────┘
```

### 5.2 Model Architecture

When FER is enabled (per-session):

| Component | Technology | Size | Latency |
|:---|:---|:---|:---|
| **Face Detection** | BlazeFace (MediaPipe) | ~1 MB | <10ms |
| **Alignment** | 68-landmark detector | <2 MB | <5ms |
| **Classification** | MobileNet-v3 (INT8) | ~3–5 MB | <30ms |
| **Total** | | **6–8 MB** | **<45ms** |

### 5.3 Output & Emotions

```
Input: Face frame from camera
  ↓
Face Detection (BlazeFace)
  ↓
Facial Landmarks (68 points)
  ↓
Emotion Classification
  ↓
Output: [happy, sad, angry, surprised, fearful, disgusted, neutral]
        + Confidence: 0-100%
```

### 5.4 Privacy Safeguards

| Safeguard | Implementation |
|:---|:---|
| **No Image Storage** | Volatile buffer; zero disk writes |
| **No Transmission** | All inference on-device |
| **Per-Session Consent** | Resets when app closed/backgrounded |
| **Visual Indicator** | LED/icon when camera active |
| **Instant Revocation** | Disable anytime via toggle |
| **No Face Embeddings** | Landmarks extracted then discarded |

---

## 6. Multimodal Fusion Strategy

### 6.1 Fusion Approaches

| Approach | Method | Pros | Cons |
|:---|:---|:---|:---|
| **Early Fusion** | Concatenate all raw features | Simple | Requires all modalities; high-dim |
| **Late Fusion** | Independent predictions → combine | Modular; handles missing | Loses correlations |
| **Attention-Based** ✅ | Learned weights per modality | Best accuracy; robust | More complex |
| **Hybrid** | Combine early + late | Best of both | Most complex |

### 6.2 Recommended: Attention-Based Late Fusion

```
┌────────────┐  ┌────────────┐  ┌──────────┐  ┌────────────┐
│ SER Output │  │ NLP Output │  │Typing    │  │ FER Output │
│Confidence: │  │Confidence: │  │Output    │  │Confidence: │
│85%         │  │80%         │  │Conf: 70% │  │88% (opt)   │
└────┬───────┘  └────┬───────┘  └────┬─────┘  └────┬───────┘
     │               │               │            │
     └───────────────┼───────────────┴────────────┘
                     │
            ┌────────▼─────────┐
            │ Attention Weights │
            │ αᵢ = softmax     │
            │ (weight × conf)   │
            └────────┬─────────┘
                     │
            ┌────────▼──────────────┐
            │ Weighted Emotion Dist │
            │ E = Σ αᵢ × eᵢ        │
            └────────┬──────────────┘
                     │
            ┌────────▼──────────────┐
            │ Final Mood Label +    │
            │ Combined Confidence   │
            │ [happy: 82%]          │
            └───────────────────────┘
```

### 6.3 Handling Missing Modalities

| Scenario | Modalities | Expected Accuracy | Strategy |
|:---|:---|:---|:---|
| **Full** | Audio + Text + Typing + Image | 85–92% | All weights active |
| **Standard** (default) | Audio + Text + Typing | 75–85% | Redistribute; higher weight to audio |
| **Text-Only** | Text + Typing | 65–75% | Heavy NLP + typing reliance |
| **Minimal** | Typing only | 55–65% | Low confidence flag |

---

## 7. Mood-to-Food Recommendation Engine

### 7.1 Emotion-to-Nutrition Mapping

| Mood | Nutritional Goal | Food Categories | Scientific Basis |
|:---|:---|:---|:---|
| **Happy/Excited** | Maintain; celebrate | Fresh fruits, salads, light desserts | Positive mood supports healthier choices |
| **Sad/Low** | Serotonin boost | Bananas, chocolate, oats, nuts, soups | Tryptophan + carbs → serotonin |
| **Stressed/Anxious** | Cortisol reduction | Greens, avocado, berries, fish (omega-3) | Magnesium + omega-3 reduce cortisol |
| **Angry/Frustrated** | Calming; anti-inflammatory | Chamomile tea, turmeric, yogurt, nuts | Reduces physiological arousal |
| **Fatigued/Tired** | Sustained energy | Eggs, whole grains, lean protein | Slow-release carbs prevent crash |
| **Fearful/Uncertain** | Comfort; familiar | Warm oatmeal, soup, grilled cheese | Familiar flavors reduce stress |
| **Neutral** | Balanced; exploratory | Diverse based on history | Ideal state for introducing healthy options |

### 7.2 Recommendation Algorithm

```
Detected Mood + Confidence
        │
        ▼
Mood → Food Category Mapping
        │
        ▼
Candidate Food Pool (100s of items)
        │
    ┌───┴───────────────┬──────────────┐
    │                   │              │
    ▼                   ▼              ▼
User Preferences   Historical    Nutritional
(restrictions,     Patterns      Scoring
allergies,         (past orders, (macros,
cuisines)          ratings)      mood benefits)
    │                   │              │
    └───┬───────────────┴──────────────┘
        │
        ▼
    Ranking Engine
        │
        ▼
    Top-K Recommendations (3-5)
        │
        ▼
Explainability Layer:
"Suggested because you seem stressed—
these are rich in magnesium & omega-3"
```

### 7.3 Personalization

- **Collaborative Filtering:** Similar mood patterns → similar recommendations
- **Reinforcement Learning:** Track acceptance/rejection; adjust weights
- **Time-of-Day:** Morning ≠ Evening; breakfast ≠ dinner
- **Seasonal:** Winter = warm; summer = refreshing

### 7.4 Explainability Examples

> 💡 "You seem a bit tired 😴 — try these energy-boosting snacks rich in B-vitamins!"

> 🎉 "Feeling great? Here are some fresh, vibrant options to match your mood!"

> 🧘 "Noticing some stress signals — these calming foods are packed with magnesium."

---

## 8. Privacy Framework & Legal Compliance

### 8.1 Regulatory Requirements

| Regulation | Jurisdiction | Key Requirements | Our Compliance |
|:---|:---|:---|:---|
| **GDPR** | EU | Explicit consent; right to erasure; data minimization | ✅ On-device; erasure API |
| **EU AI Act** | EU | Prohibited in workplaces; high-risk classification | ✅ Consent-gated; not workplace |
| **DPDP Act 2023** | India | Explicit consent; secure deletion; reasonable safeguards | ✅ On-device; deletion policy |
| **CCPA/CPRA** | California | Opt-out rights; disclosure; no 3rd-party sale | ✅ Opt-in design; no sharing |

### 8.2 Privacy-by-Design Architecture

```
┌─────────────────────────────────────────┐
│ 📱 USER DEVICE (Trust Boundary)         │
├─────────────────────────────────────────┤
│                                         │
│ Raw Data                                │
│ (Audio, Text, Keystrokes, Camera)      │
│        ↓                                │
│ On-Device Processing                  │
│ (Feature Extraction + Inference)       │
│        ↓                                │
│ Abstract Mood Label                    │
│ (happy/sad/stressed/etc.)              │
│        ↓                                │
│ IMMEDIATE PURGE                        │
│ (Raw data discarded)                   │
│                                         │
└─────────────────────────────────────────┘
```

### 8.3 Compliance Checklist

| Requirement | Status | Implementation |
|:---|:---|:---|
| **Explicit Consent** | ✅ Required | Camera: per-session toggle; Audio/Text: onboarding disclosure |
| **Purpose Limitation** | ✅ Required | Emotion data ONLY for food recommendations; no ads/sale |
| **Data Minimization** | ✅ Required | Only mood labels retained; raw data purged immediately |
| **Right to Erasure** | ✅ Required | User can delete all mood history anytime |
| **On-Device Processing** | ✅ Required | All biometric inference runs locally; zero transmission |
| **DPIA (GDPR)** | ✅ Required | Formal Data Protection Impact Assessment for EU |
| **Age Verification** | ✅ Required | Minors (<16 EU) require parental consent |
| **Audit Trail** | ✅ Required | Log consent grants/revocations (not data) |
| **Bias Testing** | ✅ Recommended | Regular demographic bias audits |

### 8.4 What We Explicitly Do NOT Do

❌ No continuous camera surveillance
❌ No raw audio/image/keystroke storage
❌ No transmission of biometric data
❌ No workplace/educational emotion detection (EU AI Act)
❌ No emotion data for advertising/profiling/sale
❌ No inference of race/gender/age/protected characteristics
❌ No surveillance of non-consenting individuals

---

## 9. Hardware & Deployment

### 9.1 Target Platforms

| Platform | CPU | RAM | Storage | Suitability | Cost |
|:---|:---|:---|:---|:---|:---|
| **Smartphone** | Snapdragon/A-series + NPU | 6–12 GB | 128+ GB | ✅ Best | User's device |
| **Raspberry Pi 5** | Cortex-A76 (4-core) | 4–8 GB | 32+ GB SD | ✅ Good | ~$80 |
| **NVIDIA Jetson Nano** | 128-core GPU + A57 | 4 GB | 16+ GB | ✅ Good | ~$150 |
| **Google Coral Board** | Edge TPU + A53 | 1 GB | 8 GB | ✅ Optimized | ~$130 |
| **ESP32-S3** | Xtensa LX7 (240MHz) | 512 KB | 8 MB | ⚠️ Typing-only | ~$5 |

### 9.2 Software Stack

```
┌────────────────────────────┐
│ Frontend Layer             │
├────────────────────────────┤
│ React/Vue/Flutter          │
│ Real-time UI Updates       │
└────────┬───────────────────┘
         │
┌────────▼───────────────────────────┐
│ Emotion Detection Engine           │
├────────────────────────────────────┤
│ TensorFlow Lite / ONNX Runtime     │
│ MediaPipe (face detection)         │
│ libsoxr + RNNoise (audio)         │
└────────┬───────────────────────────┘
         │
┌────────▼───────────────────────────┐
│ Local Data Storage                 │
├────────────────────────────────────┤
│ SQLite (encrypted)                 │
│ AES-256 encryption at rest         │
└────────────────────────────────────┘
```

### 9.3 Model Optimization

| Technique | Reduction | Accuracy Impact | Best For |
|:---|:---|:---|:---|
| **INT8 Quantization** | 4× smaller | <2% loss | All models |
| **Knowledge Distillation** | 5–10× smaller | 3–5% loss | Large models |
| **Pruning** | 2–3× smaller | 1–3% loss | Dense networks |
| **Dynamic Quantization** | 2–3× smaller | <1% loss | Inference |

---

## 10. Datasets & Benchmarks

### 10.1 Speech Emotion Recognition Datasets

| Dataset | Language | Emotions | Samples | Type |
|:---|:---|:---|:---|:---|
| **IEMOCAP** | English | 5 basic + frustrated | ~12K utterances | Acted + Scripted |
| **RAVDESS** | English | 7 emotions | 7,356 files | Acted |
| **CREMA-D** | English | 6 emotions | 7,442 clips | Acted |
| **MSP-IMPROV** | English | 4 basic | ~8.4K utterances | Improvised |
| **EmoDB** | German | 7 emotions | 535 utterances | Acted |

### 10.2 Text Sentiment/Emotion Datasets

| Dataset | Emotions | Samples | Source |
|:---|:---|:---|:---|
| **GoEmotions** | 27 fine-grained | 58K | Reddit comments |
| **SemEval-2018** | 11 emotions | 10K+ | Twitter |
| **ISEAR** | 7 emotions | 7,666 | Self-reports |
| **EmoInt** | 4 emotions | 7K+ | Twitter |

### 10.3 Keystroke Dynamics

| Dataset | Users | Features | Context |
|:---|:---|:---|:---|
| **CMU Keystroke** | 51 | Dwell/flight times | Password typing |
| **Aalto University** | 168K | Full keystroke logs | Free typing |
| **EMAKI** | 40 | Keystrokes + labels | Emotion-labeled |

### 10.4 Facial Emotion Recognition

| Dataset | Images | Emotions | Type |
|:---|:---|:---|:---|
| **FER2013** | 35,887 | 7 classes | Lab setting |
| **AffectNet** | ~1M | 8 classes | In-the-wild |
| **RAF-DB** | 30K | 19 expressions | Real-world |

---

## 11. Expected Performance Metrics

### 11.1 Accuracy by Modality (Standalone)

| Modality | Lab Accuracy | Real-World Accuracy | Confidence Range |
|:---|:---|:---|:---|
| **SER (Audio)** | 75–85% | 65–75% | Medium–High |
| **Text (NLP)** | 80–88% | 70–80% | Medium–High |
| **Typing Behavior** | 70–80% | 60–70% | Low–Medium |
| **FER (Image)** | 85–92% | 75–85% | High |
| **Multimodal Fusion** | 90–95% | 80–90% | High |

### 11.2 Latency Requirements

| Component | Target Latency | Priority | Notes |
|:---|:---|:---|:---|
| **Audio SER** | <100ms | High | Real-time feedback |
| **Text Analysis** | <50ms | High | On every input |
| **Typing Analysis** | <5ms | Critical | Continuous monitoring |
| **FER (Optional)** | <100ms | Medium | Per-frame processing |
| **Fusion** | <50ms | High | Final output |
| **Recommendations** | <500ms | Low | Can be deferred |

### 11.3 Resource Constraints

| Resource | Target | Notes |
|:---|:---|:---|
| **Model Size (Total)** | <150 MB | For smartphone deployment |
| **Memory Footprint** | <100 MB | Peak RAM usage during inference |
| **Battery Impact** | <5% per hour | Continuous monitoring |
| **Storage (History)** | <500 MB | 30 days of emotion logs |

---

## 12. Future Research Directions

1. **Wearable Integration:** HRV, skin conductance from smartwatches
2. **Federated Learning:** Improve models collaboratively without sharing data
3. **On-Device LLMs:** Richer conversation-based mood assessment
4. **Longitudinal Analysis:** Pattern detection over weeks/months
5. **Context-Aware Fusion:** Incorporate time, weather, activity level
6. **Adversarial Robustness:** Defend against spoofed emotional inputs

---

## 13. References

### Multimodal Emotion Recognition
1. M. Chen et al., "Multimodal Emotion Recognition Using Deep Learning," MDPI Sensors, 2024.
2. "Edge-Deployable Multimodal Emotion Recognition with Transformer Fusion," TechRxiv, 2025.

### Speech Emotion Recognition
3. S. Mirsamadi et al., "Automatic Speech Emotion Recognition Using MFCCs and Prosodic Features," IEEE ICASSP, 2024.

### Text Sentiment Analysis
4. D. Demszky et al., "GoEmotions: A Dataset of Fine-Grained Emotions," ACL, 2020.
5. J. Devlin et al., "BERT: Pre-training of Deep Bidirectional Transformers," NAACL, 2019.

### Keystroke Dynamics
6. K. Vizer et al., "Detecting Cognitive and Emotional States from Keystroke Dynamics," Computers in Human Behavior, 2009.

### Privacy & Legal
7. EU AI Act — Regulation (EU) 2024/1689, Official Journal, 2024.
8. India Digital Personal Data Protection Act, 2023 — DPDP Rules, 2025.

---

> **Note:** This framework is a living document. Updates should reflect new research, regulatory changes, and system improvements.
