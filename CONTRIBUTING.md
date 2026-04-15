# Contributing to CureBay Flutter

Thank you for your interest in contributing to CureBay! This document provides guidelines for contributing code, documentation, and improvements.

## Table of Contents
- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Testing](#testing)

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. All contributors must adhere to principles of respect, inclusivity, and professionalism.

## Getting Started

1. **Fork the repository** on GitHub
2. **Clone your fork** locally
   ```bash
   git clone https://github.com/yourusername/curebay-flutter.git
   cd curebay-flutter
   ```
3. **Add upstream remote** for staying updated
   ```bash
   git remote add upstream https://github.com/Deepanshu-Nain/curebay-flutter.git
   ```

## Development Setup

### Prerequisites
- Flutter SDK >= 3.2.0
- Dart >= 3.2.0
- Android SDK (for Android development)
- Git

### Setup Steps

1. **Install Flutter dependencies**
   ```bash
   flutter pub get
   ```

2. **Set up Android environment** (if on Windows)
   ```bash
   setup_flutter.bat
   ```

3. **Place required model files** in `assets/models/`
   ```
   - medgemma-1.5-4b-it-Q4_K_M.gguf
   - minilm/model.onnx
   - efficientnet_v2_s.tflite
   ```
   See `MODEL_FILES_REQUIRED.md` for details.

4. **Run the app**
   ```bash
   flutter run
   ```

## Commit Guidelines

### Commit Message Format
Use clear, descriptive commit messages following this pattern:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `refactor`: Code refactoring without behavior change
- `perf`: Performance improvements
- `test`: Test additions/updates
- `chore`: Build, CI, dependency updates

### Examples
```
feat(assessment): add multi-round follow-up questions

Implement session state tracking for multi-turn assessments.
Adds confidence-based follow-up question generation.

Fixes #42
```

```
fix(llm-service): handle null responses from MedGemma

Add null-safety checks in LLM inference response parsing.
```

## Pull Request Process

1. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes** following coding standards (see below)

3. **Run tests and linting**
   ```bash
   flutter analyze
   flutter test
   ```

4. **Commit with descriptive messages**
   ```bash
   git commit -m "feat(module): description"
   ```

5. **Push to your fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request** on GitHub with:
   - Clear title and description
   - Reference to related issues (Fixes #123)
   - Screenshot/video if UI changes
   - Summary of changes

7. **Address review comments** - be responsive and collaborative

## Coding Standards

### Dart Style Guide
Follow [Effective Dart](https://dart.dev/guides/language/effective-dart) conventions:

- Use `const` constructors where possible
- Prefer final variables
- Use meaningful variable names
- Max line length: 120 characters

### Flutter Best Practices
- Use `Provider` for state management
- Separate UI from business logic
- Create reusable widgets
- Use proper error handling

### Code Structure
```dart
// imports organized: dart, flutter, packages, relative
import 'dart:async';
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../models/user.dart';

// class declaration with documentation
/// A service that handles user authentication
class AuthService {
  // private fields
  final _client = http.Client();
  
  // public methods with documentation
  /// Authenticates user with credentials
  /// 
  /// Throws [AuthException] if credentials invalid
  Future<User> login(String email, String password) async {
    // implementation
  }
}
```

### Documentation
- Add doc comments to public APIs
- Use `///` for doc comments
- Explain complex logic with inline comments

```dart
/// Generates embeddings for disease symptoms
/// 
/// Returns a 384-dimensional vector representation
/// of the input symptom text using MiniLM model.
Future<List<double>> generateEmbedding(String symptom) async {
  // implementation
}
```

## Testing

### Unit Tests
```dart
test('Assessment engine correctly calculates confidence', () {
  final engine = AssessmentEngine();
  final assessment = engine.parseResponse(mockResponse);
  
  expect(assessment.confidence, greaterThan(0.7));
});
```

### Widget Tests
```dart
testWidgets('Assessment screen displays form fields', (WidgetTester tester) async {
  await tester.pumpWidget(MyApp());
  
  expect(find.byType(TextField), findsWidgets);
  expect(find.text('Submit'), findsOneWidget);
});
```

### Running Tests
```bash
# Run all tests
flutter test

# Run specific test file
flutter test test/services/assessment_engine_test.dart

# Run with coverage
flutter test --coverage
```

## Areas for Contribution

### High Priority
- [ ] Multi-language support (Hindi, Marathi, Tamil, etc.)
- [ ] Improved confidence scoring
- [ ] rPPG refinement for vital signs
- [ ] Performance optimization

### Medium Priority
- [ ] Enhanced UI/UX
- [ ] More comprehensive testing
- [ ] Documentation improvements
- [ ] Better error handling

### Community Projects
- Regional disease knowledge bases
- Integration with local healthcare systems
- Model fine-tuning for specific regions
- Educational content & training materials

## Reporting Issues

### Security Issues
**Do not** open public issues for security vulnerabilities. Email the maintainers directly.

### Bugs
Use GitHub issues with:
- Clear title
- Description of expected vs actual behavior
- Steps to reproduce
- Device/OS information
- Screenshots if applicable

### Feature Requests
Use GitHub issues to propose new features. Include:
- Motivation: Why is this needed?
- Use case: How would users benefit?
- Implementation ideas: Any thoughts on how to implement?

## Getting Help

- **Documentation**: See [ARCHITECTURE.md](ARCHITECTURE.md)
- **Issues**: Search existing issues or create new one
- **Discussions**: Use GitHub Discussions for questions

## License

By contributing to CureBay, you agree that your contributions will be licensed under the same license as the project.

---

**Questions?** Open an issue or reach out to the maintainers. We're here to help!
