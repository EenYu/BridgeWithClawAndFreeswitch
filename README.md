# BridgeWithClawAndFreeswitch

BridgeWithClawAndFreeswitch is a split frontend/backend voice bridge that connects FreeSWITCH calls to an AI conversation pipeline:

FreeSWITCH -> Bridge Service -> STT -> OpenClaw -> TTS -> FreeSWITCH

The repository contains:

- a Go backend that accepts FreeSWITCH media over WebSocket, manages call sessions, orchestrates provider calls, and exposes REST/WebSocket APIs
- a React + Vite frontend console for health checks, live session observation, provider settings, and manual playback interruption
- deployment examples and runbooks for the real FreeSWITCH integration path

## What It Does

- receives caller PCM audio from FreeSWITCH over `/ws/freeswitch/stream`
- streams transcripts into the bridge session state
- forwards final user turns to OpenClaw
- synthesizes assistant replies through TTS
- plays synthesized audio back into the active FreeSWITCH call
- exposes a dashboard over REST and WebSocket for operations and debugging

## Repository Layout

```text
backend/   Go bridge server and tests
frontend/  React/Vite operations console
deploy/    example environment files
docs/      API contract, FreeSWITCH notes, frontend runbooks, plans
scripts/   helper scripts for local protocol testing
```

## Runtime Architecture

```text
FreeSWITCH
  -> /ws/freeswitch/stream
  -> backend session manager + orchestrator
  -> STT / OpenClaw / TTS providers
  -> native FreeSWITCH playback for synthesized audio

Browser console
  -> /api/health
  -> /api/sessions
  -> /api/settings/providers
  -> /ws
```

## Requirements

- Go 1.22+
- Node.js 20+ and npm
- FreeSWITCH with WebSocket audio streaming support for real telephony integration

## Quick Start

### 1. Start the backend

```powershell
cd backend
go test ./...
go run ./cmd/bridge-server
```

Default backend listen address:

- `:8080`

### 2. Start the frontend

```powershell
cd frontend
npm install
npm run dev
```

Default frontend dev URL:

- `http://127.0.0.1:5173`

The sample frontend env file is:

- `frontend/.env.example`

Example local frontend settings:

```env
VITE_API_BASE_URL=http://127.0.0.1:8080
VITE_WS_URL=ws://127.0.0.1:8080/ws
```

## Backend Configuration

The backend reads runtime configuration from environment variables. A staging-oriented example is provided at:

- `deploy/env/backend.staging.env.example`

Important groups:

- bridge identity and HTTP bind address
- public base URL and WebSocket ingress URL
- STT provider settings
- OpenClaw gateway settings
- TTS provider settings

The project intentionally keeps secrets out of the repository. Use local environment files or your secret manager for real credentials.

## Main Endpoints

REST:

- `GET /api/health`
- `GET /api/sessions`
- `GET /api/sessions/{id}`
- `POST /api/sessions/{id}/interrupt`
- `POST /api/settings/providers`

WebSocket:

- `GET /ws`
- `GET /ws/freeswitch/stream`

The frozen API contract lives in:

- `docs/api/openapi.yaml`

## FreeSWITCH Integration Notes

The bridge is designed around a WebSocket media ingress contract. Repository references:

- `docs/freeswitch/integration.md`
- `docs/freeswitch/runbook.md`
- `docs/freeswitch/examples/inbound_bridge.xml`

The current production-oriented notes document a working path where:

- FreeSWITCH streams inbound audio into the bridge over WebSocket
- the bridge uses native FreeSWITCH playback for synthesized WAV output when needed

## Frontend Console

The frontend includes:

- dashboard health cards
- active session table
- live event log
- session detail view
- provider settings page

Frontend runbooks:

- `docs/frontend/README.md`
- `docs/frontend/live-call-checklist.md`

## Development Notes

- local-only files such as `workflow.md`, logs, temp files, local binaries, and `frontend/node_modules/` are ignored by Git
- the repository already excludes sensitive local operational data from version control
- use `go test ./...` for backend verification
- use `npm run build` in `frontend/` for frontend verification

## Status

This repository contains the bridge skeleton plus the real integration path that was validated end-to-end for:

- FreeSWITCH media ingress
- session orchestration
- OpenClaw request/response flow
- TTS playback return path
- frontend operational visibility

