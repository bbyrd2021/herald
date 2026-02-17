# Herald

**Private, on-device AI agent for iOS. Local LLM. Device-bound authentication. It calls you.**

---

## Overview

Herald is a local AI agent for iOS that runs entirely on your own hardware — no cloud APIs, no third-party infrastructure, no data leaving your personal network.

The agent is accessible through Siri, iMessage, and a native iOS app. It handles academic tasks like generating study guides, summarizing lectures, drafting emails, and managing assignment reminders. The differentiating feature is that Herald is proactive — it monitors context in the background and initiates contact with you when it detects something worth surfacing. Your phone rings. You pick up. Herald briefs you and waits for your response.

The authentication architecture is inspired by serial part tracking in military aircraft maintenance, where specific components are bound to a specific airframe — traceable, accountable, and non-transferable. Herald applies the same mental model to device identity: a P256 key pair is generated in the iPhone's Secure Enclave and bound to that specific device. The server recognizes only that device. No credentials, no tokens — just cryptographic proof that the request came from your phone.

---

## Architecture

Full system architecture diagram → [herald architecture](https://bbyrd2021.github.io/herald) · [architecture doc](./ARCHITECTURE.md)

**Three layers:**

**iPhone (Client)**

- CallKit for agent-initiated calls — Herald rings you, not the other way around
- Siri / App Intents for voice-triggered requests
- iMessage Extension for async text interaction
- Secure Enclave P256 key pair for device-bound authentication
- mTLS URLSession with server certificate pinning

**Secure Transport**

- Tailscale WireGuard mesh — private tunnel between iPhone and Mac Mini, no public IP exposed
- Nginx reverse proxy with mutual TLS — requests without a valid device certificate are rejected at the handshake, before they reach the application layer
- APNs outbound for server-to-device push (triggers CallKit)
- One-time QR-based pairing ceremony with TOTP revocation and 30-day certificate rotation

**Mac Mini M4 (Server)**

- FastAPI agent layer with tool-calling orchestration
- MLX + local LLM (Llama 3.1 8B / Mistral 7B) — Apple Silicon optimized, sub-2 second latency
- Proactive event engine — monitors academic context and fires APNs push on trigger conditions
- Student tool registry (see below)

---

## Student Tool Registry

| Tool                                     | Description                                                                |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| `generate_study_guide(notes, topic)`     | Builds a structured study guide from uploaded notes or lecture transcripts |
| `schedule_study_session(exam, deadline)` | Creates a study plan and adds sessions to calendar                         |
| `summarize_lecture(transcript)`          | Condenses lecture content into key points                                  |
| `draft_professor_email(context)`         | Drafts a professional email given context about the situation              |
| `create_assignment_reminder(due_date)`   | Sets tiered reminders for upcoming deadlines                               |
| `track_grades(course)`                   | Monitors grade input and surfaces performance trends                       |

Tool registry is extensible — new tools can be added without changes to the core agent loop.

---

## Privacy Model

Herald sits at an intentional position in the privacy hierarchy:

| Tier | Approach                                      | Tradeoff                                        |
| ---- | --------------------------------------------- | ----------------------------------------------- |
| 1    | On-device inference only                      | Most private, least capable                     |
| 2    | **Herald — your hardware, device-bound auth** | **Private + capable**                           |
| 3    | Apple Private Cloud Compute                   | Strong guarantees, still Apple's infrastructure |
| 4    | Cloud APIs (OpenAI, Anthropic, etc.)          | Least private — data leaves your control        |

The privacy guarantee in Herald is not policy-based — it's architectural. There is no third party to trust because there is no third party in the chain.

---

## Tech Stack

| Layer          | Technology                                                           |
| -------------- | -------------------------------------------------------------------- |
| iOS Client     | Swift, SwiftUI, CallKit, App Intents, CryptoKit, AVSpeechSynthesizer |
| Authentication | Secure Enclave P256, mTLS, Certificate Pinning                       |
| Transport      | Tailscale (WireGuard), Nginx                                         |
| Agent          | Python, FastAPI                                                      |
| Inference      | MLX, Llama 3.1 8B / Mistral 7B                                       |
| Push           | Apple Push Notification Service (APNs)                               |

---

## Status

Currently in active development as a systems course project at North Carolina A&T State University.

- [x] Architecture finalized
- [ ] mTLS pairing ceremony
- [ ] FastAPI agent + tool registry
- [ ] MLX inference integration
- [ ] CallKit proactive call flow
- [ ] Siri / App Intents integration
- [ ] iMessage Extension
- [ ] iOS app UI

---

## Background

The device-bound authentication model in Herald is directly inspired by serial part tracking in military aircraft maintenance — a system where specific components are tied to a specific airframe in a way that is traceable, accountable, and non-transferable. The same principle applied to personal AI authentication: the device and the server recognize only each other, with no intermediary in the chain.

---

## Team

Built by a graduate CS team at NC A&T · Specs & Design Coursework · Spring 2025
