# MediConnect — Healthcare + AI Assistant (Android)

Kotlin · Jetpack Compose · MVVM · Clean Architecture · Hilt · Coroutines/Flow · Retrofit · Room · Firebase Auth · FCM

## What's in this scaffold

A working Clean Architecture skeleton covering all 8 requested features:

| Layer | Contents |
|---|---|
| `domain/` | Models, repository interfaces, use cases — pure Kotlin, no Android/Firebase imports |
| `data/` | Retrofit DTOs + API service, Room entities/DAOs/database, repository implementations |
| `di/` | Hilt modules (Network, Database, Firebase, Repository bindings) |
| `presentation/` | Compose screens + ViewModels, one package per feature (auth, doctor, appointment, chat, prescription, video, common, navigation) |
| `notification/` | FCM service for push notifications |

Features wired end-to-end (UI → ViewModel → UseCase → Repository → Room/Retrofit):
- Patient registration/login (Firebase Auth)
- Doctor listing (offline-first: Room cache + Retrofit refresh)
- Appointment booking (slot picker, video/in-person toggle)
- Appointment history (with cancel)
- **AI Health Assistant chat** (see below)
- Prescription management (read-only list/detail)
- Push notifications (FCM service + channel)
- Mock video consultation screen (timer, mute/camera/end-call UI)

## AI integration — what you need

**Don't put an AI API key in the Android app.** This scaffold calls `POST /chat` on
**your own backend**, which then calls the AI provider server-side. A working sample
backend is included in `backend-sample/functions/src/chat.js` (Firebase Cloud
Function → Anthropic Claude API, model `claude-sonnet-4-6`).

To go live:
1. Get an Anthropic API key from https://console.anthropic.com
2. `cd backend-sample/functions && npm install`
3. `firebase functions:secrets:set ANTHROPIC_API_KEY`
4. `firebase deploy --only functions`
5. Update `BASE_URL` in `app/build.gradle.kts` to your deployed function's base URL.

Alternative: swap the backend call for Firebase's AI Logic SDK (Gemini) if you'd
rather not run a custom backend — same `ChatRepository` interface, different
implementation.

The system prompt in `chat.js` enforces basic medical-safety guardrails (no
diagnosis, emergency-symptom escalation, no dosage advice) — read and adjust it
before shipping.

## Setup

1. **Firebase**: create a project at https://console.firebase.google.com, add an
   Android app with applicationId `com.mediconnect.app`, download
   `google-services.json` into `app/`, enable Email/Password auth and Cloud
   Messaging.
2. **Backend**: deploy `backend-sample/functions` (or your own backend) and set
   `BASE_URL` in `app/build.gradle.kts`.
3. **Open in Android Studio** (Koala+ recommended), let Gradle sync, run on a
   device/emulator with API 24+.

## What's stubbed / left for you

- `backend-sample` is a minimal reference, not production-hardened (add logging,
  better rate limiting, retries).
- Doctor/appointment/prescription endpoints in `MediConnectApiService` assume a
  REST backend with the shapes in `data/remote/dto/ApiDtos.kt` — adapt to your
  actual backend or build matching Cloud Functions/Firestore rules.
- Video consultation is **UI-only** (no real call). Swap in Agora, Twilio, or
  WebRTC behind the same `VideoConsultationScreen` composable signature.
- App icon is a placeholder vector — replace with your real launcher icon.
- No unit/UI tests included yet — the use-case layer is the easiest place to
  start (pure Kotlin, fully mockable repositories).

## Architecture flow (AI chat example)

```
ChatScreen (Compose)
   -> ChatViewModel (StateFlow<List<ChatMessage>>)
      -> SendChatMessageUseCase
         -> ChatRepository (interface, domain layer)
            -> ChatRepositoryImpl (data layer)
               -> Room (caches user + assistant messages for offline history)
               -> Retrofit -> your backend /chat -> Claude API (claude-sonnet-4-6)
```

