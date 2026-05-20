# Docker for Data Engineers — From Zero to Docker Compose
### Pre-Training Primer | Code With Z

---

> **Who this is for:** You've heard the word "Docker." Maybe you've nodded along when someone said it. That's perfectly fine. By the end of this guide, you'll not only understand Docker — you'll be comfortable using Docker Compose to spin up the entire training stack we'll use across all 5 days.

---

## Part 1 — The Problem Docker Solves

Before we talk about Docker, let's talk about a problem every developer has lived:

> *"It works on my machine."*

You install a data pipeline on your laptop. It runs perfectly. You share it with a colleague. They spend three hours debugging why it crashes. The reason? Their Java version is different. Or their Python path is wrong. Or they're on Windows and you're on Mac.

This is called the **"works on my machine" problem**, and it has caused more lost weekends than anything else in software.

**Docker is the solution.** It lets you say:

> *"Here is not just my code — here is my entire machine, pre-configured, ready to run."*

---

## Part 2 — The Restaurant Kitchen Analogy

Let's use a restaurant kitchen to understand Docker from the ground up.

---

### 🏗️ The Traditional Way (Without Docker)

Imagine you're a chef. Every time you open a new restaurant branch, you have to:

- Renovate the kitchen from scratch
- Buy all new equipment
- Train each kitchen on your exact setup
- Hope nothing is missing

This is how software used to be deployed — install the OS, install dependencies, configure everything manually. Every "kitchen" (server) is slightly different. Chaos.

---

### 📦 What is a Docker Image? — The Recipe Card

A **Docker Image** is like a **complete recipe card** that includes:

- The dish you want to make (your application/service)
- Every ingredient, down to the exact brand and quantity (dependencies, libraries, configs)
- Step-by-step cooking instructions (startup commands)

The recipe card itself is **not the food** — it's the instructions. You can print this card 1,000 times and hand it to 1,000 kitchens. Every kitchen will produce the **exact same dish**.

> **Real world:** The Apache NiFi Docker image is a recipe card that says: "Install Java 11, configure NiFi 1.23, expose port 8443, start with these settings." Everyone who uses this image gets the same NiFi.

---

### 🍱 What is a Docker Container? — The Actual Dish

A **Docker Container** is the **running instance** made from the image — it's the actual dish served at the table.

| Concept | Restaurant | Docker |
|---|---|---|
| Recipe Card | Image | `docker image` |
| Actual Dish | Container | `docker container` |
| Kitchen | Host Machine | Your laptop / server |
| Restaurant Branch | Environment | dev / staging / prod |

You can make **many dishes** from one recipe card. Similarly, you can run **many containers** from one image.

```
Image (Recipe Card)  →  run  →  Container 1
                     →  run  →  Container 2
                     →  run  →  Container 3
```

---

### 🏠 What is the Docker Engine? — The Kitchen Itself

The **Docker Engine** is the kitchen. It's the software running on your machine that knows how to read recipe cards (images) and actually cook the food (run containers).

When you install Docker on your laptop, you're installing the kitchen.

---

### 📚 What is Docker Hub? — The Recipe Library

**Docker Hub** is a giant public library of recipe cards. Companies like Apache, Cloudera, and Confluent publish their official images here.

Instead of building a NiFi image from scratch, we just pull the official one:

```bash
docker pull apache/nifi:1.23.2
```

That's like saying: *"Get me Apache's official NiFi recipe from the library."*

---

## Part 3 — Docker in Practice

Let's make this concrete with commands you'll actually use.

### Installing Docker

Download Docker Desktop from [https://www.docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop).

After installation, verify it works:

```bash
docker --version
# Docker version 24.x.x
```

---

### Your First Container — Hello World

```bash
docker run hello-world
```

What just happened?
1. Docker looked for an image called `hello-world` on your machine — didn't find it
2. It went to Docker Hub and downloaded it
3. It created a container from that image and ran it
4. The container printed a message and exited

---

### Running a Real Service — Apache NiFi

```bash
docker run -d -p 8443:8443 --name my-nifi apache/nifi:1.23.2
```

Breaking this down:

| Flag | Meaning | Restaurant Analogy |
|---|---|---|
| `run` | Create and start a container | "Open for business" |
| `-d` | Run in background (detached) | Kitchen runs without you watching |
| `-p 8443:8443` | Map port on your machine to port inside container | Front door number → Kitchen door number |
| `--name my-nifi` | Give the container a friendly name | Name the branch "Downtown NiFi" |
| `apache/nifi:1.23.2` | Which image to use (and version) | Which exact recipe version |

---

### The Port Mapping Explained

The `-p` flag is critically important. Think of it like this:

```
Your Laptop (Port 8443)  ←→  Container (Port 8443)
```

Your laptop's port 8443 is the **front door** of the restaurant. The container's port 8443 is the **kitchen window** where food comes out. You connect them with `-p`.

Without port mapping, the container runs but nobody can reach it from outside.

---

### Essential Docker Commands

```bash
# See all running containers
docker ps

# See all containers (including stopped ones)
docker ps -a

# Stop a container
docker stop my-nifi

# Start a stopped container
docker start my-nifi

# Remove a container (must be stopped first)
docker rm my-nifi

# See logs from a container
docker logs my-nifi

# See logs in real-time (follow)
docker logs -f my-nifi

# Go inside a running container (like SSH-ing in)
docker exec -it my-nifi bash

# See all images on your machine
docker images

# Remove an image
docker rmi apache/nifi:1.23.2
```

---

### Volumes — Persisting Data (The Fridge Analogy)

Containers are **ephemeral** by default. When you delete a container, everything inside it disappears. 

> **Analogy:** The kitchen is destroyed every night. Any food left in the kitchen at closing time is gone. But the **fridge** (volume) survives — it's outside the kitchen.

A **volume** is storage that lives outside the container. Data written to a volume persists even after the container is deleted.

```bash
docker run -d \
  -p 8443:8443 \
  -v /my/local/nifi-data:/opt/nifi/nifi-current/data \
  apache/nifi:1.23.2
```

The `-v` flag maps a folder on your machine (`/my/local/nifi-data`) to a folder inside the container (`/opt/nifi/.../data`). Data written to the container path lands on your machine.

---

### Environment Variables — The Kitchen Configuration Board

Most Docker containers are configured through **environment variables** — key-value settings passed at startup.

```bash
docker run -d \
  -p 8443:8443 \
  -e NIFI_WEB_HTTPS_PORT=8443 \
  -e NIFI_CLUSTER_IS_NODE=false \
  apache/nifi:1.23.2
```

The `-e` flag sets environment variables. Think of it as writing instructions on a whiteboard for the kitchen before it opens.

---

## Part 4 — Networks (How Containers Talk to Each Other)

Now things get interesting. In our training, we're not running just one service. We're running:

- Apache Hudi (on Spark)
- Apache NiFi
- Apache Kafka
- HDFS / Hive
- Zookeeper
- Debezium

All these services need to **talk to each other**. How?

### The Apartment Building Analogy

Docker creates a **virtual network** — like a private apartment building.

- Each container is an **apartment** with a **name** (like "kafka-broker" or "nifi-server")
- Containers on the same network can call each other by name: `kafka-broker:9092`
- The outside world (your laptop) can only get in through mapped ports — the building's **front door**

```bash
# Create a custom network
docker network create my-training-network

# Run containers on the same network
docker run -d --network my-training-network --name zookeeper zookeeper:3.8
docker run -d --network my-training-network --name kafka \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  confluentinc/cp-kafka:7.5.0
```

Notice: Kafka references Zookeeper by **name** (`zookeeper:2181`), not by IP. That's the power of Docker networking — containers find each other by name.

---

## Part 5 — Docker Compose (Orchestrating the Full Stack)

Now we get to the tool we'll actually use every day in training: **Docker Compose**.

### The Problem With Running Containers One by One

Managing our 8-service stack with individual `docker run` commands would look like:

```bash
docker network create training-net
docker run -d --network training-net --name zookeeper ...
docker run -d --network training-net --name kafka ...
docker run -d --network training-net --name hive-metastore ...
docker run -d --network training-net --name namenode ...
docker run -d --network training-net --name datanode ...
docker run -d --network training-net --name nifi ...
docker run -d --network training-net --name spark-master ...
docker run -d --network training-net --name spark-worker ...
```

That's error-prone, hard to share, and a nightmare to manage. Every time you want to start the lab environment, you'd run 9+ commands manually.

### Docker Compose — The Catering Order Form

**Docker Compose** is like a **single catering order form** that describes your entire event:

> *"I need 1 Zookeeper kitchen, 1 Kafka kitchen, 2 Spark kitchens, 1 NiFi kitchen — all connected through the same network, all starting in the right order."*

Everything is described in one file: `docker-compose.yml`. One command brings the whole stack up. One command tears it down.

```bash
# Start everything (all services defined in docker-compose.yml)
docker compose up -d

# Stop everything
docker compose down

# Stop and delete all data volumes too
docker compose down -v
```

---

### Anatomy of a docker-compose.yml File

Let's break down a real `docker-compose.yml` piece by piece.

```yaml
version: "3.8"

services:

  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    container_name: zookeeper
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"
    networks:
      - training-net

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    container_name: kafka-broker
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker:9092
    ports:
      - "9092:9092"
    networks:
      - training-net

  nifi:
    image: apache/nifi:1.23.2
    container_name: nifi
    depends_on:
      - kafka
    environment:
      NIFI_WEB_HTTPS_PORT: 8443
      SINGLE_USER_CREDENTIALS_USERNAME: admin
      SINGLE_USER_CREDENTIALS_PASSWORD: adminpassword
    ports:
      - "8443:8443"
    volumes:
      - nifi-data:/opt/nifi/nifi-current/data
    networks:
      - training-net

networks:
  training-net:
    driver: bridge

volumes:
  nifi-data:
```

---

### The docker-compose.yml Sections Explained

#### `services:`
Each entry under `services` is one container. The name you give it (e.g., `kafka`, `nifi`) becomes its **hostname** on the Docker network.

```yaml
services:
  kafka:       # ← This name becomes reachable as "kafka" inside the network
    image: ...
```

#### `image:`
Which Docker image to use. Matches what you'd type in `docker pull`.

```yaml
image: apache/nifi:1.23.2
#      ↑ image name : ↑ version tag
```

Always pin to a **specific version** (like `1.23.2`), never use `latest`. This prevents unexpected upgrades from breaking your setup.

#### `container_name:`
The actual name of the running container. Useful for `docker logs`, `docker exec`, etc.

#### `environment:`
Configuration variables — the kitchen's instruction whiteboard.

```yaml
environment:
  KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
  # ↑ Variable name          ↑ Value
```

Notice `zookeeper:2181` — Kafka is referencing Zookeeper by its **service name**, which works because both are on the same Docker network.

#### `ports:`
Maps your machine's ports to the container's ports.

```yaml
ports:
  - "9092:9092"
  # "YOUR_MACHINE:CONTAINER"
```

You access this service from your browser/terminal at `localhost:9092`.

#### `volumes:`
Persistent storage. Data written here survives container restarts.

```yaml
volumes:
  - nifi-data:/opt/nifi/nifi-current/data
  # named-volume : path inside container
```

#### `depends_on:`
Tells Compose the startup order. Kafka depends on Zookeeper, so Compose starts Zookeeper first.

```yaml
depends_on:
  - zookeeper
```

> ⚠️ **Important:** `depends_on` only controls *start order*, not whether the service is fully *ready*. Kafka might start before Zookeeper is actually accepting connections. In production systems, you'd add health checks. For training, we handle this with startup delays.

#### `networks:`
All services sharing a network can reach each other by service name.

```yaml
networks:
  - training-net
```

At the bottom of the file, you declare the network:

```yaml
networks:
  training-net:
    driver: bridge
```

---

### Essential Docker Compose Commands

```bash
# Start all services in background
docker compose up -d

# Start and show all logs in terminal
docker compose up

# Stop all services (keeps data)
docker compose down

# Stop all services and delete volumes (wipes data)
docker compose down -v

# See status of all services
docker compose ps

# View logs for all services
docker compose logs

# View logs for a specific service (follow mode)
docker compose logs -f nifi

# Restart a single service
docker compose restart nifi

# Go inside a specific running container
docker compose exec nifi bash

# Pull latest images for all services
docker compose pull

# Build images (when using custom Dockerfiles)
docker compose build
```

---

### Checking If Services Are Healthy

When you run `docker compose ps`, you'll see output like:

```
NAME            IMAGE                   STATUS          PORTS
zookeeper       cp-zookeeper:7.5.0      Up 2 minutes    0.0.0.0:2181->2181/tcp
kafka-broker    cp-kafka:7.5.0          Up 1 minute     0.0.0.0:9092->9092/tcp
nifi            apache/nifi:1.23.2      Up 45 seconds   0.0.0.0:8443->8443/tcp
```

**What to look for:**

| Status | Meaning | Action |
|---|---|---|
| `Up X minutes` | Healthy and running | ✅ Good |
| `Up X seconds` | Just started, may still be initializing | ⏳ Wait a moment |
| `Exited (1)` | Crashed | Check logs: `docker compose logs <service>` |
| `Restarting` | Keeps crashing | Check logs urgently |

---

## Part 6 — The Full Training Stack

Here's the complete picture of what we're running and why:

```
┌─────────────────────────────────────────────────────┐
│                  Docker Network                      │
│                                                     │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐  │
│  │Zookeeper │───▶│  Kafka   │───▶│   Debezium   │  │
│  │ :2181    │    │  :9092   │    │  (CDC layer) │  │
│  └──────────┘    └──────────┘    └──────┬───────┘  │
│                                         │           │
│  ┌──────────┐                    ┌──────▼───────┐  │
│  │  NiFi    │───────────────────▶│    Hudi      │  │
│  │  :8443   │   (ingest/route)   │  (on Spark)  │  │
│  └──────────┘                    └──────┬───────┘  │
│                                         │           │
│  ┌──────────┐    ┌──────────┐    ┌──────▼───────┐  │
│  │   Hive   │◀───│   HDFS   │◀───│   Spark      │  │
│  │Metastore │    │NameNode  │    │ Master+Worker│  │
│  └──────────┘    └──────────┘    └──────────────┘  │
└─────────────────────────────────────────────────────┘
        ▲ All of this starts with: docker compose up -d
```

| Service | Role | Why We Need It |
|---|---|---|
| **Zookeeper** | Coordination service | Kafka cannot run without it |
| **Kafka** | Event streaming | Source of real-time data for Hudi |
| **Debezium** | Change Data Capture | Captures database changes as Kafka events |
| **NiFi** | Ingestion & routing | Moves data from sources into the pipeline |
| **Spark Master/Worker** | Processing engine | Hudi uses Spark to write data |
| **Hudi** | Storage layer | Manages upserts, deletes, time travel on data lake |
| **HDFS NameNode/DataNode** | Distributed storage | Where Hudi actually stores the data files |
| **Hive Metastore** | Schema registry | Stores table definitions for querying |

---

## Part 7 — Troubleshooting Common Issues

### Container Exits Immediately

```bash
docker compose logs <service-name>
```

Look for error messages. Common causes:
- Wrong environment variable (e.g., Kafka can't reach Zookeeper)
- Port already in use on your machine
- Not enough memory (we need 16GB+ RAM)

### Port Already in Use

```
Error: bind: address already in use
```

Something on your laptop is already using that port. Find and stop it:

```bash
# Mac/Linux: find what's using port 9092
lsof -i :9092

# Windows (PowerShell)
netstat -ano | findstr :9092
```

### Out of Memory

Docker Desktop has a memory limit. For our stack, you need at minimum **16GB allocated to Docker**.

1. Open Docker Desktop
2. Go to Settings → Resources
3. Set Memory to at least 16GB
4. Apply & Restart

### Service Shows "Restarting" Loop

The service is crashing and Docker keeps trying to restart it. Check logs immediately:

```bash
docker compose logs --tail=50 <service-name>
```

### Clean Slate — Nuclear Option

If everything is broken and you want to start completely fresh:

```bash
# Stop everything and remove volumes
docker compose down -v

# Remove all stopped containers
docker container prune -f

# Remove unused images
docker image prune -f

# Start fresh
docker compose up -d
```

> ⚠️ `down -v` deletes all data in named volumes. Don't use this if you have data you need to keep.

---

## Part 8 — Checklist Before Day 1

Complete these steps before the first session:

- [ ] Docker Desktop installed and running
- [ ] Docker Desktop memory set to **16GB minimum** (32GB preferred)
- [ ] Run `docker --version` — confirms Docker is working
- [ ] Run `docker compose version` — confirms Compose is available
- [ ] Pull the training stack: `docker compose pull` (from the provided `docker-compose.yml`)
- [ ] Run `docker compose up -d` and confirm all services show `Up` in `docker compose ps`
- [ ] Access NiFi UI at `https://localhost:8443` in your browser
- [ ] Run `docker compose down` to stop everything cleanly before Day 1

If any step fails, contact the trainer **at least 3 days before** the first session.

---

## Quick Reference Card

```bash
# DAILY COMMANDS
docker compose up -d          # Start the whole stack
docker compose down           # Stop the whole stack
docker compose ps             # Check status
docker compose logs -f nifi   # Watch NiFi logs

# DEBUGGING
docker compose logs <service> # See what went wrong
docker compose exec <svc> bash # Get a shell inside container
docker compose restart <svc>  # Restart one service

# NUCLEAR OPTIONS
docker compose down -v        # Stop + wipe all data
docker system prune -a        # Remove everything Docker-related
```

---

## Glossary

| Term | Plain English |
|---|---|
| **Image** | A recipe card — the blueprint for a container |
| **Container** | A running instance made from an image |
| **Docker Engine** | The software that runs containers on your machine |
| **Docker Hub** | Public library of official images |
| **Volume** | Persistent storage outside the container |
| **Port Mapping** | Connecting your machine's port to a container's port |
| **Network** | A private channel containers use to talk to each other |
| **Docker Compose** | A tool to define and run multi-container applications |
| `docker-compose.yml` | The configuration file for Docker Compose |
| **Service** | One container definition in a Compose file |
| `depends_on` | Tells Compose which services to start first |
| **Environment Variable** | A configuration value passed to a container at startup |

---

