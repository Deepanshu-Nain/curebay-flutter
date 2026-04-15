# CureBay Flutter - Architecture & Design

## Table of Contents
1. [System Architecture](#system-architecture)
2. [Project Structure](#project-structure)
3. [Core Modules](#core-modules)
4. [Data Flow](#data-flow)
5. [Offline-First Design](#offline-first-design)
6. [Future Scalability](#future-scalability)
7. [Internationalization (i18n) Plan](#internationalization-i18n-plan)

---

## System Architecture

### High-Level Overview
CureBay is an **offline-first, AI-powered health assessment assistant** designed for ASHA (Accredited Social Health Activists) workers in rural areas. It operates with **zero internet dependency** after initial setup and provides multimodal input (text, voice, images) with local LLM inference.

```
┌─────────────────────────────────────────────────────────────────┐
│                      User Interface Layer                        │
│  (Auth → Dashboard → Assessment → History → Patient Management) │
└────────────────────┬────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────┐
│                   State Management (Provider)                    │
└────────────────────┬────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────┐
│                    Service Layer                                 │
│  ┌──────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │ Auth     │  │ Assessment │  │ Database   │  │ Embedding  │  │
│  │ Service  │  │ Engine     │  │ Helper     │  │ Service    │  │
│  └──────────┘  └────────────┘  └────────────┘  └────────────┘  │
│  ┌──────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │ LLM      │  │ RAG        │  │ Vector DB  │  │ Voice      │  │
│  │ Service  │  │ Service    │  │            │  │ Service    │  │
│  └──────────┘  └────────────┘  └────────────┘  └────────────┘  │
│  ┌──────────┐  ┌────────────┐                                    │
│  │ Image    │  │ rPPG       │                                    │
│  │ Service  │  │ Service    │                                    │
│  └──────────┘  └────────────┘                                    │
└────────────────────┬────────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────────┐
│                    ML/Data Layer                                 │
│  ┌──────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │ SQLite   │  │ MedGemma   │  │ EfficientNet│ │ Chroma     │  │
│  │ Database │  │ GGUF Model │  │ Image Model │ │ Vector DB  │  │
│  └──────────┘  └────────────┘  └────────────┘  └────────────┘  │
│  ┌──────────┐  ┌────────────┐                                    │
│  │ Disease  │  │ MiniLM     │                                    │
│  │ KB       │  │ Embedder   │                                    │
│  └──────────┘  └────────────┘                                    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

```
curebay_flutter/
├── lib/
│   ├── main.dart                          # App entry point, initialization
│   ├── theme/
│   │   └── app_theme.dart                # Material Design theme configuration
│   │
│   ├── models/                            # Data Models
│   │   ├── user.dart                     # User/ASHA Worker profile
│   │   ├── patient.dart                  # Patient data structure
│   │   └── assessment.dart               # Assessment results
│   │
│   ├── services/                          # Business Logic & Integration
│   │   ├── auth_service.dart             # Authentication & authorization
│   │   ├── database_helper.dart          # SQLite operations
│   │   ├── patient_service.dart          # Patient CRUD operations
│   │   ├── assessment_engine.dart        # Assessment workflow orchestration
│   │   ├── llm_service.dart              # MedGemma inference interface
│   │   ├── embedding_service.dart        # Text embedding generation
│   │   ├── rag_service_mobile.dart       # RAG pipeline (retrieval + generation)
│   │   ├── vector_db.dart                # Chroma vector store interface
│   │   ├── voice_service_mobile.dart     # STT/TTS operations
│   │   ├── image_service_mobile.dart     # Image processing & classification
│   │   ├── rppg_service_mobile.dart      # Remote photoplethysmography (vital signs)
│   │   ├── api_service.dart              # Optional remote API calls
│   │   └── offline_model_installer.dart  # Asset bundling & model extraction
│   │
│   └── screens/                           # UI Pages
│       ├── auth_screen.dart              # Login/Registration
│       ├── main_shell.dart               # Bottom navigation container
│       ├── dashboard_screen.dart         # Home/Dashboard
│       ├── assessment_screen.dart        # Multi-step assessment intake
│       ├── history_screen.dart           # Past assessments
│       └── patients_screen.dart          # Patient list & management
│
├── assets/
│   ├── models/
│   │   ├── medgemma-1.5-4b-it-Q4_K_M.gguf  # Quantized LLM
│   │   ├── efficientnet_v2_s.tflite        # Image classification model
│   │   ├── imagenet_labels.txt
│   │   ├── minilm/
│   │   │   ├── model.onnx                # Embedding model
│   │   │   ├── tokenizer.json
│   │   │   ├── tokenizer_config.json
│   │   │   └── vocab.txt
│   │   ├── whisper/
│   │   │   ├── encoder_model.onnx        # Speech-to-text model
│   │   │   └── decoder_model.onnx
│   │   └── rag_index.json                # Pre-built vector index
│   │
│   ├── disease_kb/
│   │   └── diseases_kb.json              # Disease knowledge base
│   │
│   ├── chroma_data/
│   │   └── chroma.sqlite3                # Persistent vector store
│   │
│   └── database/
│       └── (Initial database schemas if any)
│
├── android/
│   └── app/
│       └── src/main/AndroidManifest.xml  # Android permissions & config
│
├── pubspec.yaml                           # Flutter dependencies
├── analysis_options.yaml                  # Lint rules
├── MODEL_FILES_REQUIRED.md                # Model setup instructions
├── setup_flutter.bat                      # Windows setup script
└── README.md                              # Project overview
```

---

## Core Modules

### 1. **Authentication Service** (`auth_service.dart`)
- User login/registration (ASHA worker credentials)
- JWT token management
- Password hashing with crypto
- Session persistence

### 2. **Database Layer** (`database_helper.dart`)
- SQLite schema management
- Patient records, assessment history
- Session state persistence
- Migration handling

### 3. **Assessment Engine** (`assessment_engine.dart`)
- Orchestrates the full assessment workflow
- Coordinates embedding, RAG retrieval, LLM inference
- Manages multi-round follow-up Q&A
- Confidence scoring and validation
- Fallback to deterministic triage if LLM unavailable

### 4. **LLM Service** (`llm_service.dart`)
- MedGemma GGUF model loading & inference
- Context window management
- JSON response parsing
- Prompt templating
- Quantization-aware inference

### 5. **RAG Pipeline** (`rag_service_mobile.dart`)
- Query embedding generation
- Vector similarity search
- Retrieved context ranking
- Prompt building with patient context

### 6. **Vector DB** (`vector_db.dart`)
- Chroma vector store integration
- Disease KB embedding storage
- Similarity search interface
- Persistent storage

### 7. **Embedding Service** (`embedding_service.dart`)
- MiniLM model for 384-dim embeddings
- Query and document embedding generation
- ONNX/TFLite model inference

### 8. **Voice Service** (`voice_service_mobile.dart`)
- Speech-to-Text conversion
- Text-to-Speech output
- Permission handling
- Optional Whisper model integration

### 9. **Image Service** (`image_service_mobile.dart`)
- Camera/gallery image capture
- Image preprocessing
- EfficientNet classification
- Feature extraction

### 10. **rPPG Service** (`rppg_service_mobile.dart`)
- Camera-based vital sign extraction
- Heart rate, respiratory rate, SpO2 estimation
- Signal processing and filtering

---

## Data Flow

### Assessment Workflow

```
1. User Input (Auth Screen)
   └─> Credentials validated by AuthService
       └─> Token stored in SharedPreferences

2. Assessment Initiation (Dashboard → Assessment Screen)
   └─> PatientService retrieves patient data
       └─> Assessment session created in DatabaseHelper

3. Symptom/Data Capture
   ├─> Text input: symptoms, notes
   ├─> Voice input → STT → TextualSymptoms
   ├─> Image input → ImageService → classification features
   └─> Optional: rPPG → vital signs (HR, RR, SpO2)

4. Embedding Generation (EmbeddingService)
   └─> Query (symptom text) embedded as 384-dim vector
       └─> Stored in local context

5. RAG Retrieval (RAG Service)
   └─> Vector similarity search in Chroma DB
       └─> Top-K disease chunks retrieved from KB
           └─> Context scored and ranked

6. Prompt Construction
   ├─> Patient profile (age, sex, medical history)
   ├─> Vitals (BP, HR, RR, SpO2)
   ├─> Symptoms (text + voice transcript)
   ├─> Retrieved disease context
   └─> Previous assessment history

7. LLM Inference (MedGemma GGUF)
   └─> Quantized model processes prompt
       └─> Returns JSON assessment:
           ├─ Differential diagnoses
           ├─ Confidence scores
           ├─ Recommended actions
           └─ Follow-up questions

8. Post-Processing (Assessment Engine)
   ├─> Parse JSON response
   ├─> Confidence validation
   └─> If confidence < threshold:
       ├─> Generate follow-up questions
       └─> Store session state
           └─> Await follow-up answers
               └─> Merge context & re-assess

9. Persistence (Database)
   └─> Store assessment in SQLite
       ├─ Final diagnosis
       ├─ Confidence scores
       ├─ Recommended actions
       ├─ Session transcript
       └─ Timestamp + worker ID

10. Display Results (History Screen)
    └─> Render assessment summary
        ├─ Diagnosis
        ├─ Confidence
        └─ Recommended referral/actions
```

---

## Offline-First Design

### Why Offline-First?
- **Rural Connectivity**: Unreliable internet in ASHA worker deployment areas
- **Latency**: Real-time assessment without network round-trips
- **Privacy**: Patient data never leaves device
- **Resilience**: App works 100% without connectivity after setup

### Model Bundling Strategy
All ML models are **pre-bundled in APK**:
- MedGemma GGUF (quantized LLM)
- MiniLM (embedding model)
- EfficientNet v2 (image classification)
- Whisper (optional STT)
- Disease knowledge base JSON
- Pre-computed vector index (Chroma)

### Initialization Flow
1. App detects missing models at startup
2. `offline_model_installer.dart` extracts bundled assets
3. Models copied to app-local storage (not cloud)
4. On subsequent launches, models loaded from local storage
5. Optional: Sync with backend server (if connectivity available)

### Storage Layer
- **SQLite**: Patient records, assessments, session state
- **SharedPreferences**: Auth tokens, user settings
- **File system**: Large ML models, embeddings cache
- **Chroma**: Vector DB (persisted as SQLite3)

---

## Future Scalability

### 1. **Horizontal Scaling**
- **Worker deployment**: Scale to 1000s of ASHA workers
- **Distributed sync**: Aggregate anonymous assessments to backend
- **Analytics**: Track disease patterns, model drift
- **Federated learning**: Improve models without centralizing data

### 2. **Model Improvements**
- Larger quantized LLMs (7B, 13B parameter models)
- Domain-specific fine-tuning on Indian health data
- Multi-modal fusion (text + image + vitals jointly)
- Confidence calibration for better follow-up triggering

### 3. **Feature Expansion**
- **Prescription generation**: Integration with e-pharmacy
- **Telemedicine**: Video consultation with doctors
- **Insurance integration**: Direct claims filing
- **Lab integration**: Remote lab booking + results
- **Wearable sync**: Connect to health tracking devices

### 4. **Regional Deployment**
- **State-specific disease prevalence**: Adapt models per region
- **Local healthcare guidelines**: Customize recommendations
- **Government integration**: MOHFW, state health systems
- **Clinic/Hospital backends**: Connect to primary care centers

---

## Internationalization (i18n) Plan

### Current Status
- App currently in **English only**

### Multi-Language Support (Phase 1)

#### Supported Languages (MVP)
1. **English** (en)
2. **Hindi** (hi)
3. **Marathi** (mr)
4. **Tamil** (ta)
5. **Telugu** (te)
6. **Gujarati** (gu)
7. **Bengali** (bn)
8. **Kannada** (kn)

### Implementation Strategy

#### 1. **String Localization**
```
Use Flutter's `intl` package (already in pubspec.yaml)

Structure:
lib/l10n/
├── app_en.arb        # English translations
├── app_hi.arb        # Hindi translations
├── app_mr.arb        # Marathi translations
├── app_ta.arb        # Tamil translations
├── app_te.arb        # Telugu translations
├── app_gu.arb        # Gujarati translations
├── app_bn.arb        # Bengali translations
├── app_kn.arb        # Kannada translations
└── localizer.dart    # AppLocalizations delegate

Usage in code:
String message = AppLocalizations.of(context)!.assessmentTitle;
```

#### 2. **Voice & Text Processing**
- **STT**: Use language-specific speech recognition
  - Google Speech-to-Text supports all 8 languages
  - AI4Bharat IndicSpeech (open-source alternative)
  
- **TTS**: Language-specific text-to-speech
  - Google TTS supports all 8 languages
  - Offline options via MariTTS, Indic TTS

- **LLM Prompting**: Generate disease context in user's language
  - Extend MedGemma prompting to support language selection

#### 3. **Model & Knowledge Base Adaptation**
```
Current: diseases_kb.json (English)

Multi-lang structure:
assets/disease_kb/
├── diseases_kb_en.json    # English
├── diseases_kb_hi.json    # Hindi
├── diseases_kb_mr.json    # Marathi
├── diseases_kb_ta.json    # Tamil
├── diseases_kb_te.json    # Telugu
├── diseases_kb_gu.json    # Gujarati
├── diseases_kb_bn.json    # Bengali
└── diseases_kb_kn.json    # Kannada

Each KB contains:
{
  "diseases": [
    {
      "name": "मलेरिया",     // Localized name
      "description": "...",   // Localized description
      "symptoms": [...],      // Localized symptoms
      "treatments": [...]     // Localized treatments
    }
  ]
}
```

#### 4. **Vector Embeddings for Non-English**
- Extend `embedding_service.dart` to use language-specific embeddings
- Options:
  - **IndicBERT**: Multilingual embeddings for Indian languages
  - **XLM-RoBERTa**: Pre-trained multilingual model
  - **Language-specific MiniLM variants**

#### 5. **RAG Context Switching**
```dart
// Example: dynamically select KB based on language
String selectedLanguage = // en, hi, mr, ta, te, gu, bn, kn
String kbPath = 'assets/disease_kb/diseases_kb_$selectedLanguage.json';

// Load language-specific vector index
ChromaDB db = await ChromaDB.load(
  locale: selectedLanguage
);
```

#### 6. **User Preference Storage**
```dart
// SharedPreferences to persist language choice
await prefs.setString('app_language', 'hi');

// On app restart, reload with saved language
String savedLanguage = prefs.getString('app_language') ?? 'en';
```

#### 7. **LLM Response Localization**
```dart
// MedGemma inference with language context
String prompt = '''
Language: $selectedLanguage
Patient Symptoms: $symptoms
...
Respond in $selectedLanguage language.
''';

// Parse response, ensure it's in selected language
AssessmentResult result = await llmService.infer(prompt);
```

### Multi-Language Rollout Timeline

**Phase 1 (MVP)**: English + Hindi (covers ~50% of ASHA workers)
- Implement i18n framework
- Translate UI strings
- Hindi knowledge base

**Phase 2**: Marathi + Tamil + Telugu (adds ~25%)
- Regional KB localization
- Language-specific embedding models

**Phase 3**: Gujarati + Bengali + Kannada (adds remaining ~25%)
- Complete 8-language support
- Regional deployment

### Testing i18n

```dart
// Unit tests for language switching
testWidgets('Language switching updates UI', (WidgetTester tester) async {
  await tester.pumpWidget(CureBayApp());
  
  // Switch to Hindi
  await setLanguage(context, 'hi');
  await tester.pumpAndSettle();
  
  expect(find.text('मूल्यांकन'), findsOneWidget);  // "Assessment" in Hindi
});

// Verify all strings are localized
test('All UI strings have translations', () {
  final languages = ['en', 'hi', 'mr', 'ta', 'te', 'gu', 'bn', 'kn'];
  for (String lang in languages) {
    final kb = loadKnowledgeBase(lang);
    expect(kb.isNotEmpty, true);
  }
});
```

### Adding a New Language (Template)

1. **Create `.arb` file**: `lib/l10n/app_xx.arb`
2. **Translate strings**: Copy from `app_en.arb`, translate values
3. **Create KB JSON**: `assets/disease_kb/diseases_kb_xx.json`
4. **Create embedding index**: Generate vector embeddings for new language
5. **Update `pubspec.yaml`**: Add language to `supportedLocales`
6. **Test thoroughly**: Ensure text rendering, RTL (if applicable), encoding

---

## Architecture Decision Records (ADRs)

### ADR 1: Offline-First with Local SQLite + Vector DB
**Decision**: Store all data locally, sync optionally with backend
**Rationale**: Rural connectivity unreliable; privacy-first approach
**Trade-offs**: No real-time sync; eventual consistency model

### ADR 2: Quantized MedGemma GGUF over Full-Size LLM
**Decision**: Use 4-bit quantized model for mobile feasibility
**Rationale**: 4B param model fits in APK; runs on modest hardware
**Trade-offs**: Slightly lower accuracy; acceptable for triage

### ADR 3: Provider for State Management
**Decision**: Use Provider package for reactive state
**Rationale**: Lightweight, sufficient for this domain; proven in Flutter community
**Trade-offs**: Not as powerful as Bloc/Redux; simpler mental model

### ADR 4: Asset Bundling vs. Runtime Download
**Decision**: Bundle models in APK; optional backend sync
**Rationale**: Zero internet dependency; reliable offline performance
**Trade-offs**: Larger APK (~300-500MB); simpler deployment

---

## Performance Considerations

| Operation | Time | Device | Notes |
|-----------|------|--------|-------|
| App startup | 2-3s | Pixel 4 | Model loading + DB init |
| Embedding generation | 200-300ms | Pixel 4 | Query text → vector |
| RAG retrieval | 50-100ms | Pixel 4 | Vector similarity search |
| LLM inference | 5-10s | Pixel 4 | MedGemma GGUF inference |
| Image classification | 500-800ms | Pixel 4 | EfficientNet v2 |
| Voice transcription | 2-5s | Pixel 4 | STT (device model) |
| Total assessment | ~15-20s | Pixel 4 | End-to-end workflow |

---

## Security & Privacy

- **Data**: Never leaves device (except opt-in sync)
- **Auth**: JWT tokens + local crypto
- **Model weights**: Shipped with app; no remote calls
- **KB**: Bundled assets; no external queries
- **Database**: Encrypted SQLite (optional via sqlcipher)

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on extending architecture.

---

**Last Updated**: 2025 | **Maintainer**: CureBay Team
