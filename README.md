# 🎙️ VoiceChat AI – Intelligent AI Voice Cloning & Realtime Messaging

[![React Native](https://img.shields.io/badge/React%20Native-0.74-blue.svg)](https://reactnative.dev)
[![Expo SDK](https://img.shields.io/badge/Expo%20SDK-51.0-black.svg)](https://expo.dev)
[![Supabase](https://img.shields.io/badge/Supabase-Backend%20%26%20Realtime-3ECF8E.svg)](https://supabase.com)
[![ElevenLabs](https://img.shields.io/badge/ElevenLabs-AI%20Voice%20Cloning%20%26%20TTS-orange.svg)](https://elevenlabs.io)
[![Android](https://img.shields.io/badge/Platform-Android%20(Release%20APK)-green.svg)](https://android.com)

**VoiceChat AI** is a next-generation mobile voice messaging platform built with **React Native (Expo SDK 51)**, **Supabase**, and **ElevenLabs**. It allows users to record their voice, create a high-definition AI voice clone profile, and exchange text messages that are automatically synthesized into realistic voice messages spoken in their cloned AI voice in realtime.

---

## 📱 Features

- **🎙️ Instant Voice Cloning Profile**: Record a short 8-second voice sample, upload it securely to private cloud storage, and activate an AI voice clone powered by ElevenLabs.
- **🗣️ AI Text-to-Speech Voice Messaging**: Send text messages that are dynamically converted into voice notes using the sender's cloned voice.
- **⚡ Realtime Chat Experience**: Powered by Supabase Realtime subscriptions for instant bidirectional messaging between users.
- **🔒 Private Cloud Storage & Strict RLS**:
  - `voice-samples`: Private audio bucket restricted strictly to the user who uploaded it (`auth.uid()`).
  - `voice-messages`: Private audio bucket accessible only by authenticated conversation participants.
- **🛡️ Secure Backend Edge Architecture**:
  - Sensitive API keys (`ELEVENLABS_API_KEY`, Supabase Service Role) are kept securely in Supabase backend Secrets.
  - Zero sensitive keys are exposed to the client or Android APK.
- **📱 Standalone Signed Android Release APK**: Fully standalone release APK bundled with Hermes JavaScript engine and native C++ shared libraries for `arm64-v8a` and `armeabi-v7a`.

---

## 🏗️ System Architecture

```text
┌────────────────────────────────────────────────────────┐
│             VoiceChat AI Mobile Application            │
│                 (React Native + Hermes)                │
└──────────────┬──────────────────────────┬──────────────┘
               │                          │
       Supabase Auth & RLS        Realtime & REST
               │                          │
               ▼                          ▼
┌────────────────────────────────────────────────────────┐
│                   Supabase Cloud                       │
│  ┌──────────────────┐            ┌──────────────────┐  │
│  │ PostgreSQL DB    │            │ Storage Buckets  │  │
│  │ • profiles       │            │ • voice-samples  │  │
│  │ • voice_profiles │            │   (Private)      │  │
│  │ • conversations  │            │ • voice-messages │  │
│  │ • messages       │            │   (Private)      │  │
│  └─────────┬────────┘            └────────┬─────────┘  │
│            │                              │            │
│            └──────────────┬───────────────┘            │
│                           ▼                            │
│                 Supabase Edge Functions                │
│                 • clone-voice                          │
│                 • generate-voice                       │
│                 • verify-elevenlabs                    │
└───────────────────────────┬────────────────────────────┘
                            │
              Secure Server-to-Server API
                            ▼
┌────────────────────────────────────────────────────────┐
│                   ElevenLabs API                       │
│  • Instant Voice Cloning (/v1/voices/add)              │
│  • Multilingual TTS (/v1/text-to-speech)               │
└────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Schema & Security (Supabase)

### Tables
1. **`public.profiles`**: User details (full name, email, avatar). Auto-created on signup via trigger.
2. **`public.voice_profiles`**: Linked ElevenLabs voice IDs, clone status (`training`, `ready`, `failed`), and metadata.
3. **`public.conversations`**: Conversation threads.
4. **`public.conversation_members`**: Membership mapping for multi-user chat threads.
5. **`public.messages`**: Message history containing text, generated `audio_url`, and duration.

### Row Level Security (RLS)
- Strict non-recursive RLS policies prevent unauthorized access.
- Non-members cannot read conversations, messages, or listen to private audio notes.
- Helper security definer function `public.is_member_of_conversation(c_id uuid)` eliminates recursion and delivers high-performance query execution.

### Auto-Confirm Trigger
- Signups are auto-confirmed via Postgres trigger `trigger_auto_confirm_user`, eliminating email rate-limit blocks and SMTP dependencies during onboarding.

---

## 🚀 Getting Started

### Prerequisites
- **Node.js** (v18 or v20 recommended)
- **npm** or **yarn**
- **Android Studio** & **Android SDK** (for local Android build)
- **Supabase Account** with CLI installed (`npx supabase`)
- **ElevenLabs Account** with API Key

### 1. Clone the Repository
```bash
git clone https://github.com/your-username/VoiceChatAI.git
cd VoiceChatAI
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Configure Environment Variables
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```
Update `.env` with your Supabase credentials:
```env
EXPO_PUBLIC_SUPABASE_URL=https://your-supabase-project.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=your-supabase-publishable-anon-key
```

### 4. Deploy Supabase Backend
Push migrations and storage policies:
```bash
npx supabase db push
```

Set your ElevenLabs API Key in Supabase Secrets:
```bash
npx supabase secrets set ELEVENLABS_API_KEY=your_elevenlabs_api_key
```

Deploy Edge Functions:
```bash
npx supabase functions deploy clone-voice --no-verify-jwt
npx supabase functions deploy generate-voice --no-verify-jwt
npx supabase functions deploy verify-elevenlabs --no-verify-jwt
```

---

## 🧪 Automated End-to-End Testing

VoiceChat AI includes an automated verification test suite:

```bash
node scripts/test-e2e.mjs
```

### Test Coverage:
- `[1/8]` Database Schema and Tables (RLS enabled on all 5 tables)
- `[2/8]` Two-User Authentication (`alice@voicechat.ai` & `bob@voicechat.ai`)
- `[3/8]` Conversation Creation and Membership Verification
- `[4/8]` Private Storage Bucket Security (Alice upload, Bob blocked by RLS)
- `[5/8]` Voice Cloning Edge Function & Database Persistence
- `[6/8]` ElevenLabs Text-to-Speech Generation (`generate-voice`)
- `[7/8]` Realtime Message Transmission & Delivery (Alice -> Bob)
- `[8/8]` Bidirectional Reply & Thread Integrity (Bob -> Alice)

---

## 📦 Android Release APK

The project includes a production-ready, signed release APK.

- **APK File**: `release/VoiceChatAI_FINAL.apk`
- **Package Name**: `com.voicechatai.app`
- **Size**: ~65.4 MB
- **Engine**: Hermes JS Engine
- **ABIs**: `arm64-v8a`, `armeabi-v7a`

### How to Install on Android
1. Transfer `VoiceChatAI_FINAL.apk` to your Android device via USB, Google Drive, or WhatsApp.
2. Tap the APK file on your device.
3. If prompted, enable **"Install unknown apps"** for your file manager.
4. Tap **Install** and open **VoiceChat AI**.

### Building the APK from Source
```bash
cd android
./gradlew assembleRelease
```
The output APK will be generated at:
`android/app/build/outputs/apk/release/app-release.apk`

---

## 📂 Project Directory Structure

```text
VoiceChatAI/
├── android/                     # Native Android project configuration
│   ├── app/                     # Android app module (manifest, gradle)
│   └── build.gradle             # Top-level Gradle configuration
├── assets/                      # Application icons and splash screens
├── release/                     # Production signed release APK
│   └── VoiceChatAI_FINAL.apk
├── scripts/                     # Verification and test scripts
│   ├── test-e2e.mjs             # Complete 8-step end-to-end automated test
│   ├── setup-test-users.mjs     # Provisions Alice and Bob test accounts
│   └── check-status.mjs         # Edge Function & ElevenLabs diagnostics
├── src/                         # React Native Application Source
│   ├── api/                     # Supabase client initialization
│   ├── components/              # UI components (AudioPlayer, Icons)
│   ├── constants/               # Colors, themes, styling constants
│   ├── context/                 # AuthContext and VoiceContext state
│   ├── navigation/              # Navigation container & screens routing
│   ├── screens/                 # AuthScreen, HomeScreen, ChatDetailScreen, etc.
│   └── types/                   # TypeScript interfaces
├── supabase/                    # Supabase Backend Configuration
│   ├── functions/               # Edge Functions (clone-voice, generate-voice)
│   └── migrations/              # SQL migrations for schema, RLS, storage
├── .env.example                 # Template for environment variables
├── app.json                     # Expo configuration
├── package.json                 # Project dependencies & scripts
├── tsconfig.json                # TypeScript configuration
└── README.md                    # Project documentation
```

---

## 📄 License
MIT License. Created for the VoiceChat AI project.
