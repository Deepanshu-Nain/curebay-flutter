# CureBay Flutter - Offline-First AI Health Assistant

[![Flutter](https://img.shields.io/badge/Flutter-3.2+-blue.svg)](https://flutter.dev/)
[![Dart](https://img.shields.io/badge/Dart-3.2+-blue.svg)](https://dart.dev/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen.svg)](#)

> **CureBay**: Empowering rural ASHA workers with AI-powered health assessments that work **100% offline**.

---

## 🎯 Overview

CureBay is a cutting-edge mobile health assistant designed for **ASHA (Accredited Social Health Activists)** workers in rural India. It provides **AI-driven medical assessments** without requiring internet connectivity, enabling healthcare professionals to deliver better care in underserved regions.

### Key Features

✅ **100% Offline Operation** - No internet required after initial setup  
✅ **Multimodal Input** - Text, voice, and image-based symptom capture  
✅ **Local AI Inference** - MedGemma quantized LLM runs on-device  
✅ **Vector Search** - Disease knowledge base retrieval with embeddings  
✅ **Multi-turn Assessments** - Follow-up questions based on initial responses  
✅ **Vital Signs Capture** - Heart rate, respiratory rate, SpO2 via camera (rPPG)  
✅ **Patient Management** - Track patient history and assessments locally  
✅ **Privacy-First** - All data stays on device; no cloud dependency  
✅ **Multi-Language Support** - English + 7 Indian languages (roadmap)

---

## 🏗️ Architecture

### System Components

```
User Interface Layer
        ↓
State Management (Provider)
        ↓
Service Layer (Auth, Assessment, Database, LLM, RAG, etc.)
        ↓
ML/Data Layer (SQLite, MedGemma, Chroma, TFLite, ONNX)
```

**For detailed architecture documentation**, see [ARCHITECTURE.md](ARCHITECTURE.md)

### Key Services

| Service | Purpose | Technology |
|---------|---------|------------|
| **AssessmentEngine** | Orchestrates full assessment workflow | Dart |
| **LLMService** | MedGemma GGUF inference | llama_cpp_dart |
| **RAGService** | Retrieval-augmented generation | Chroma + MiniLM |
| **EmbeddingService** | Text embeddings generation | ONNX/TFLite |
| **VoiceService** | Speech-to-text conversion | speech_to_text |
| **ImageService** | Image classification & processing | TFLite EfficientNet |
| **rPPGService** | Vital signs extraction from camera | camera + FFT |
| **DatabaseHelper** | Patient & assessment persistence | SQLite |

---

## 📦 Installation & Setup

### Prerequisites

- **Flutter** >= 3.2.0
- **Dart** >= 3.2.0
- **Android SDK** (for Android builds)
- **Git**

### Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/Deepanshu-Nain/curebay-flutter.git
   cd curebay_flutter
   ```

2. **Install dependencies**
   ```bash
   flutter pub get
   ```

3. **Download required model files**
   
   Place these in `assets/models/`:
   - `medgemma-1.5-4b-it-Q4_K_M.gguf` (LLM)
   - `minilm/model.onnx` (Embeddings)
   - `efficientnet_v2_s.tflite` (Image classification)
   
   See [MODEL_FILES_REQUIRED.md](MODEL_FILES_REQUIRED.md) for download links and setup.

4. **Run the app**
   ```bash
   flutter run
   ```

### Windows Setup (Automated)
```bash
setup_flutter.bat
```

---

## 🚀 Usage

### For ASHA Workers

1. **Login** - Enter credentials
2. **Select Patient** - Pick existing or create new patient
3. **Start Assessment**
   - Enter symptoms (text or voice)
   - Optional: Capture vital signs with camera
   - Optional: Upload relevant medical images
4. **Receive Assessment**
   - AI-generated diagnosis with confidence score
   - Recommended actions (referral, home remedies, follow-up)
   - Follow-up questions if confidence is low
5. **Review History** - Track all past assessments

---

## 📊 Data Flow

```
Patient Intake
    ↓
Symptom Embedding (MiniLM)
    ↓
Disease KB Retrieval (Chroma Vector DB)
    ↓
Prompt Construction (Context + Patient Profile + Vitals)
    ↓
LLM Inference (MedGemma GGUF)
    ↓
Assessment Parsing & Confidence Check
    ↓
If Low Confidence → Generate Follow-up Questions
    ↓
Final Assessment Storage (SQLite)
    ↓
Display Results to ASHA Worker
```

---

## 🛠️ Development

### Project Structure

```
curebay_flutter/
├── lib/
│   ├── main.dart
│   ├── models/              # Data models (User, Patient, Assessment)
│   ├── services/            # Business logic layer
│   ├── screens/             # UI pages
│   └── theme/               # Material Design theme
├── assets/
│   ├── models/              # ML models (GGUF, ONNX, TFLite)
│   ├── disease_kb/          # Disease knowledge base JSON
│   └── chroma_data/         # Vector store data
├── android/                 # Android-specific code
├── pubspec.yaml             # Dependencies
└── analysis_options.yaml    # Lint rules
```

### Code Quality

```bash
# Run analyzer
flutter analyze

# Run tests
flutter test

# Format code
flutter format lib/
```

### Adding a New Feature

1. Create a new service in `lib/services/`
2. Add models if needed in `lib/models/`
3. Create UI screens in `lib/screens/`
4. Wire services via Provider in `main.dart`
5. Write tests in `test/`

---

## 🌍 Multi-Language Support (Roadmap)

### Phase 1: Core Languages (MVP)
- [x] English
- [ ] Hindi

### Phase 2: Regional Expansion
- [ ] Marathi, Tamil, Telugu
- [ ] Regional disease knowledge bases

### Phase 3: Complete Coverage
- [ ] Gujarati, Bengali, Kannada
- [ ] Full i18n infrastructure

**See [ARCHITECTURE.md#internationalization-i18n-plan](ARCHITECTURE.md#internationalization-i18n-plan) for i18n roadmap**

---

## 📈 Performance

| Operation | Time | Device |
|-----------|------|--------|
| App Startup | 2-3s | Pixel 4 |
| Embedding Generation | 200-300ms | Pixel 4 |
| RAG Retrieval | 50-100ms | Pixel 4 |
| LLM Inference | 5-10s | Pixel 4 |
| Image Classification | 500-800ms | Pixel 4 |
| **Total Assessment** | **~15-20s** | **Pixel 4** |

---

## 🔒 Security & Privacy

- ✅ **Zero Internet Dependency** - All data stays on device
- ✅ **End-to-End Privacy** - No cloud storage
- ✅ **Encrypted Database** - Optional SQLCipher encryption
- ✅ **Secure Authentication** - JWT tokens + crypto hashing
- ✅ **HIPAA-Ready** - Can be extended with HIPAA compliance

---

## 🐛 Troubleshooting

### Common Issues

**Issue**: "Model files not found"
```
Solution: Ensure MODEL_FILES_REQUIRED.md is followed
Place files in assets/models/ before building
```

**Issue**: "Dart error with llama_cpp_dart"
```
Solution: Run flutter clean && flutter pub get
Ensure NDK is installed for Android
```

**Issue**: "Out of memory on inference"
```
Solution: Reduce batch size in LLMService
Use quantized model (already in place)
Ensure at least 2GB free RAM
```

**Issue**: "Voice recognition not working"
```
Solution: Check microphone permissions in AndroidManifest.xml
Test permission_handler package separately
```

---

## 📚 Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Detailed system design, scalability roadmap
- **[MODEL_FILES_REQUIRED.md](MODEL_FILES_REQUIRED.md)** - ML model setup instructions
- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines
- **Code Comments** - Inline documentation for complex logic

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for:
- Development setup
- Code standards
- Pull request process
- Testing guidelines

### Areas for Contribution

- Multi-language support
- Model fine-tuning for Indian health data
- UI/UX improvements
- Integration with local healthcare systems
- Performance optimization

---

## 📋 Requirements & Dependencies

### Core Dependencies
- `flutter` - UI framework
- `provider` - State management
- `sqflite` - SQLite database
- `llama_cpp_dart` - LLM inference
- `tflite_flutter` - Image classification
- `onnxruntime` - Model inference
- `speech_to_text` - Voice input
- `image_picker` - Image capture

**See [pubspec.yaml](pubspec.yaml) for complete list**

---

## 📱 Supported Platforms

- **Android** 7.0+ (Primary target)
- iOS (Future)
- Web (Future)

---

## 🎓 Learning Resources

### Flutter
- [Flutter Official Docs](https://flutter.dev/docs)
- [Dart Language Tour](https://dart.dev/guides/language/language-tour)

### Machine Learning
- [MedGemma Model](https://huggingface.co/google/medgemma-1.5-4b-it)
- [Chroma Vector Store](https://www.trychroma.com/)
- [ONNX Runtime](https://onnxruntime.ai/)

### Healthcare
- [NITI Aayog AI Guidelines](https://niti.gov.in/)
- [WHO Digital Health Standards](https://www.who.int/)

---

## 📄 License

This project is licensed under the **MIT License** - see [LICENSE](LICENSE) file for details.

---

## 👥 Team & Credits

**CureBay** is developed by the CureBay team with support from:
- Flutter community
- MedGemma model creators (Google)
- Open-source ML libraries

---

## 📞 Support & Contact

- **Issues**: Open on GitHub for bug reports and feature requests
- **Discussions**: Use GitHub Discussions for questions
- **Security**: Report vulnerabilities to maintainers directly

---

## 🗺️ Roadmap

### Q1 2025
- [x] Core assessment engine
- [x] Offline model bundling
- [ ] Multi-language UI strings (Phase 1)

### Q2 2025
- [ ] Enhanced rPPG algorithm
- [ ] Integration with state health systems
- [ ] Analytics dashboard

### Q3 2025
- [ ] Multi-language complete (8 languages)
- [ ] Telemedicine integration
- [ ] Prescription generation

### Q4 2025+
- [ ] Wearable device sync
- [ ] Insurance claims filing
- [ ] Regional model fine-tuning
- [ ] Federated learning deployment

---

## 🌟 Star History

Your support means a lot! If CureBay helps you, please ⭐ this repository.

---

**Last Updated**: 2025-04-15 | **Version**: 2.0.0

---

<div align="center">

### 🚀 Ready to get started?

[📖 Read the Architecture Guide](ARCHITECTURE.md) | [🛠️ Setup Instructions](MODEL_FILES_REQUIRED.md) | [🤝 Contribute](CONTRIBUTING.md)

</div>
#   c u r e b a y - f l u t t e r  
 