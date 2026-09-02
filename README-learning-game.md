# AI-Enhanced Mobile Learning Game

An Android quiz-based learning game that adapts to each student. The app tracks performance across every quiz attempt, adjusts difficulty accordingly, recommends topics the learner is weakest in, and includes an AI chatbot tutor for follow-up questions.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Firebase Setup](#firebase-setup)
- [OpenRouter Setup](#openrouter-setup)
- [Firestore Data Model](#firestore-data-model)
- [Key Classes](#key-classes)
- [Security Notes](#security-notes)
- [Roadmap](#roadmap)
- [Credits](#credits)

---

## Overview

Standard quiz apps give every student the same questions in the same order. This one does not. Each attempt is recorded, and the recommendation logic reads that history to decide what the learner sees next — harder items on topics they have mastered, more practice on topics they have not.

A chatbot powered by an LLM is available alongside the quizzes so students can ask for explanations instead of only seeing a right-or-wrong result.

---

## Features

### Adaptive quizzes
- Difficulty adjusts based on the learner's previous performance
- Topic recommendations surfaced on the dashboard
- Animated countdown before each quiz begins

### Performance tracking
- Login history and quiz results recorded to Firestore
- Dedicated Performance Monitor screen with score history
- Mastery level maintained per learner

### AI chatbot tutor
- Conversational Q&A through the OpenRouter API
- Retrofit + OkHttp networking with request/response logging
- Configurable model

### Student dashboard
- Available quizzes, progress status, and personalized recommendations in one place

---

## System Architecture

```
┌──────────────┐
│     User     │
└──────┬───────┘
       │
┌──────▼────────────┐
│   Mobile App      │  Android · Java
│   (UI + logic)    │
└──────┬────────────┘
       │
       ├──────────────────────┐
       │                      │
┌──────▼────────────┐  ┌──────▼────────────┐
│  Firebase Auth    │  │   AI Engine       │
│  + Firestore      │  │   (OpenRouter)    │
│  (data store)     │  │                   │
└───────────────────┘  └───────────────────┘
```

**Sequential flow:**

1. User opens the app and registers or logs in
2. Credentials validated through Firebase Auth
3. On success, the Student Dashboard loads
4. Dashboard displays available quizzes, progress, and recommendations
5. User selects a quiz or learning module
6. Quiz content retrieved from Firestore
7. Performance history analyzed to adjust difficulty and recommend topics
8. Personalized quiz delivered to the user
9. User answers; the app evaluates and computes the score
10. Results sent for learning analysis
11. Learning profile and mastery level updated
12. Progress and feedback written back to Firestore
13. Results, feedback, and next recommendations displayed
14. User returns to the dashboard, takes another quiz, or logs out

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Platform | Android (Java) |
| IDE | Android Studio |
| Authentication | Firebase Authentication |
| Database | Cloud Firestore |
| Networking | Retrofit 2 + OkHttp 3 |
| JSON | Gson |
| AI | OpenRouter API |
| Animated backgrounds | android-gif-drawable |

---

## Project Structure

```
myproject/
└── app/src/main/
    ├── java/com/example/myproject/
    │   ├── LoginActivity.java
    │   ├── RegisterActivity.java
    │   ├── DashboardActivity.java
    │   ├── QuizActivity.java
    │   ├── RecommendationQuizActivity.java
    │   ├── PerformanceActivity.java
    │   ├── Performancetracker.java
    │   ├── ChatbotActivity.java
    │   ├── api/
    │   │   ├── ApiClient.java
    │   │   └── OpenRouterApi.java
    │   └── model/
    │       ├── Message.java
    │       ├── OpenRouterRequest.java
    │       └── OpenRouterResponse.java
    ├── res/
    │   ├── layout/
    │   │   ├── activity_quiz.xml
    │   │   ├── activity_performance.xml
    │   │   ├── item_login_history.xml
    │   │   └── item_quizz_history.xml
    │   └── drawable/
    └── AndroidManifest.xml
```

> Update this tree to match the actual package contents.

---

## Getting Started

### Prerequisites

- Android Studio (recent stable release)
- JDK 17
- Android SDK, minimum API 24
- A Firebase project
- An OpenRouter API key

### Setup

```bash
git clone <repository-url>
```

1. Open the project in Android Studio and let Gradle sync.
2. Add your `google-services.json` (see [Firebase Setup](#firebase-setup)).
3. Add your OpenRouter key (see [OpenRouter Setup](#openrouter-setup)).
4. Run on an emulator or a physical device.

### Required permission

`AndroidManifest.xml` must declare internet access, otherwise both Firestore and the chatbot fail silently:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

---

## Firebase Setup

1. Create a project in the [Firebase Console](https://console.firebase.google.com).
2. Register an Android app using the package name `com.example.myproject`.
3. Download `google-services.json` and place it in `app/`.
4. Enable **Authentication → Email/Password**.
5. Create a **Cloud Firestore** database.
6. Set security rules so a learner can only read and write their own records — do not ship with test-mode rules open to everyone.

---

## OpenRouter Setup

1. Create a key at [openrouter.ai/settings/keys](https://openrouter.ai/settings/keys).
2. Supply it to the app (see [Security Notes](#security-notes) — it should not be hardcoded).
3. Set the model in the request body.

Base URL and endpoint must combine correctly:

```
Base URL:  https://openrouter.ai/api/
Endpoint:  v1/chat/completions
Resulting: https://openrouter.ai/api/v1/chat/completions
```

A `401` means the key is wrong, revoked, or malformed. A `429` means rate limited — free models have per-minute and per-day caps, so an app that works then stops usually hit the cap rather than broke.

---

## Firestore Data Model

| Collection | Contents |
|-----------|----------|
| `users` | Profile, mastery level per topic |
| `quizzes` | Quiz definitions and questions |
| `quiz_results` | Score, topic, timestamp, per attempt |
| `login_history` | Login timestamps per user |

> Adjust to match the actual collections in use.

---

## Key Classes

| Class | Type | Role |
|-------|------|------|
| `LoginActivity` | Activity | Authentication entry point |
| `DashboardActivity` | Activity | Quizzes, progress, recommendations |
| `QuizActivity` | Activity | Quiz flow with countdown overlay |
| `RecommendationQuizActivity` | Activity | Recommendation-driven quiz flow |
| `PerformanceActivity` | Activity | Performance Monitor UI |
| `Performancetracker` | Helper | Writes login and quiz records to Firestore — no UI, no layout file |
| `ChatbotActivity` | Activity | AI tutor chat interface |
| `ApiClient` | Helper | Retrofit/OkHttp client setup |
| `OpenRouterApi` | Interface | Chat completions endpoint definition |

---

## Security Notes

**The OpenRouter API key should not be hardcoded in an Activity.** Anything compiled into the APK can be extracted — an APK is a zip file, and decompiling it takes minutes. A key pulled out that way can be used by anyone against your account until you revoke it.

For a school submission or demo this is a known and acceptable shortcut, but the key must be revoked and regenerated afterward, and it should never be committed to a public repository. For anything beyond that, route chat requests through a small backend (a Cloud Function works well here) that holds the key and forwards the request.

**Firestore rules matter as much as the app code.** Rules left in test mode allow any client with your project ID to read and write everything, regardless of what the app enforces. Lock reads and writes to the authenticated user's own documents before the app is distributed.

---

## Roadmap

- [ ] Move the OpenRouter key behind a Cloud Function
- [ ] Offline quiz caching
- [ ] Teacher/admin view for class-level progress
- [ ] Achievements and streaks
- [ ] Expanded mastery model per subtopic

---

## Credits

**Developer:** John

---

## License

Specify the applicable license or mark the project as academic use only.
