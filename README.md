[README.md](https://github.com/user-attachments/files/32039299/README.md)
# SpaceRover — Gesture-Controlled Rover Game

An IBM-sponsored software engineering capstone project (Ontario Tech
University): a full microservices game where players control a rover on a
mission board using **real-time hand gestures**, captured via webcam and
classified by a CNN, with live scoring, a leaderboard, and full system
monitoring.

## What it does

A player raises their hand in front of a webcam. A gesture-recognition
pipeline classifies the gesture (movement direction, capture, etc.) and
streams the command over WebSockets to a game service, which updates the
rover's position/score on a mission board in real time and pushes state to
a React client. Game results are persisted to a leaderboard service backed
by MongoDB, and the whole stack is instrumented with Prometheus + Grafana.

## Architecture

```
┌─────────────────┐      WebSocket       ┌──────────────────┐
│  Gesture Control │ ───────────────────▶ │   Game Service    │
│  (OpenCV + CNN)  │                      │ (Java/OpenLiberty) │
└─────────────────┘                      └─────────┬──────────┘
                                                     │ REST
┌─────────────────┐      WebSocket       ┌──────────▼──────────┐    ┌──────────┐
│   React Client    │ ◀──────────────────│   (game state)      │───▶│ Leaderboard │──▶ MongoDB
│  (TS, Tailwind)   │                    └──────────────────────┘    │ (Java)     │
└─────────────────┘                                                  └──────────┘

Prometheus + Grafana monitor all services throughout.
```

- **Client** — React + TypeScript + Tailwind CSS. Home, signup, in-game, and
  leaderboard views, connected to the game service over WebSockets and the
  leaderboard service over HTTP.
- **Game Service** — Java, Open Liberty, Jakarta EE 9.1 + MicroProfile 5.0
  (WebSockets, Health, Config, Metrics, Fault Tolerance, REST Client,
  OpenAPI). Holds live game state and is the hub connecting hardware,
  gesture input, and the client.
- **Leaderboard Service** — Java, Open Liberty, Jakarta EE + MicroProfile.
  REST API backed by MongoDB for persisting and querying past game results.
- **Gesture Recognition** — a CNN built on **VGG19** (transfer learning,
  see `gestures/cnn_architecture/src/Space-Rover-Mission-vgg19.ipynb`) for
  classifying hand gestures from webcam frames, deployed in a real-time
  OpenCV pipeline (`gestures/openCV_implementation/src/GestureRecognitionCVv2.py`)
  that streams recognized gestures to the game service.
- **Monitoring** — Prometheus scrapes service metrics; Grafana dashboards
  visualize them.
- **Mock services** — a keyboard-controlled mock rover/board for testing the
  full game loop without physical hardware.
- Everything is containerized and orchestrated with Docker Compose.

## My role

Contributed across most of the system as part of the capstone team —
spanning the microservices architecture, service integration, and the
gesture-control pipeline.

## Stack

Java (Open Liberty, Jakarta EE, MicroProfile), React, TypeScript, Tailwind
CSS, Python (OpenCV, TensorFlow/Keras for the CNN), MongoDB, Docker Compose,
Prometheus, Grafana

## Running it

```bash
./start_demo.sh
```

This brings up the full Docker Compose stack (client, game service,
leaderboard service, MongoDB, Prometheus, Grafana), sets up a Python venv,
and starts the gesture-recognition control loop once the game service is
healthy. See `services/mock/README.md` for running with a mock rover/board
(keyboard-controlled) instead of physical hardware.

Individual service setup details are in their own READMEs:
[`services/client`](services/client/README.md) ·
[`services/game`](services/game/README.md) ·
[`services/leaderboard`](services/leaderboard/README.md) ·
[`services/mock`](services/mock/README.md)
