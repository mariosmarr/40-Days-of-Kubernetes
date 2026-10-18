# Day 1: Docker Architecture & Workflow

## 1. Docker Architecture
![Docker Architecture](./docker-architecture.png)

- **Docker Client:** This is the tool we use to send commands to Docker.
- **Docker Daemon:** This is the "brain" running in the background that listens to our commands and manages the containers.
- **Docker Registry:** The digital repository on the internet, like Docker Hub, where ready-made images are stored and shared.

## 2. Simple Docker Flow
![Simple Docker Flow](./simple-docker-flow.png)

- **Build:** Converts our code (Dockerfile) into a ready-to-use package (Docker Image).
- **Push:** Uploads our Image to the Registry.
- **Pull:** Downloads the Image from the Registry to whatever environment we need it in.
- **Run:** Takes the Image and executes it, creating a live Container.