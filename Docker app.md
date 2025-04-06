

- **Volumes**
- **Bind Mounts**
- **`--mount`**
- A visual **architecture diagram**
- Docker Compose for advanced usage

---

## 🛠️ Project: Simple Node.js App with Persistent Storage

We'll create:
- A simple Node.js web server that logs requests
- Store the logs in a persistent location using:
  - Docker Volume (`/app/logs`)
  - Bind Mount (for hot reload during development)

---

## 🧾 Project Structure

```
docker-storage-app/
│
├── app/
│   ├── server.js
│   └── logs/               ← This will be mounted (volume)
│
├── Dockerfile
├── docker-compose.yml     ← For multi-container setup
```

---

### 📄 `app/server.js`

```js
const fs = require('fs');
const http = require('http');
const PORT = 3000;

const server = http.createServer((req, res) => {
  const log = `Request: ${req.method} ${req.url}\n`;
  fs.appendFileSync('/app/logs/requests.log', log);
  res.end("Hello from Docker!");
});

server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});
```

---

### 🐳 `Dockerfile`

```Dockerfile
FROM node:18
WORKDIR /app
COPY app /app
RUN npm install
CMD ["node", "server.js"]
```

---

## ✅ Option 1: Using Docker Volume

### 🚀 Run with Docker CLI

```bash
docker volume create app-logs

docker run -d \
  -p 3000:3000 \
  --name myapp \
  -v app-logs:/app/logs \
  my-node-app
```

### ✅ Result:
- Log file saved in volume `app-logs`
- Data persists even if container stops

---

## ✅ Option 2: Using Bind Mount for Development

### 🔄 Live Mount Code Folder

```bash
docker run -d \
  -p 3000:3000 \
  --name devapp \
  -v $(pwd)/app:/app \
  -v $(pwd)/app/logs:/app/logs \
  node:18 node /app/server.js
```

This maps your **local app folder** to the container for live coding (hot reload via nodemon if needed).

---

## ✅ Option 3: Using `--mount`

### Clearer and safer:

```bash
docker run -d \
  -p 3000:3000 \
  --name mountapp \
  --mount type=volume,source=app-logs,target=/app/logs \
  my-node-app
```

---

## 🔧 Docker Compose Example

### 📄 `docker-compose.yml`

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - app-logs:/app/logs
      - ./app:/app  # bind mount for local code

volumes:
  app-logs:
```

### ➕ Run it:

```bash
docker compose up --build
```

---

## 🖼️ Architecture Diagram

Here’s a visual breakdown:

```
                ┌────────────────────────────────────┐
                │            Host Machine            │
                │                                    │
                │  ./app ───────────────┐            │
                │                       ▼            │
                │     Bind Mount   →  /app (Code)    │
                │     Docker Vol  →  /app/logs       │
                │                                    │
                │    Docker Container (myapp)        │
                │    ┌────────────────────────────┐  │
                │    │ Node.js App logging to     │  │
                │    │ /app/logs/requests.log     │  │
                │    └────────────────────────────┘  │
                └────────────────────────────────────┘
```

---

## 🧠 Summary

| Feature         | Method            | Purpose                            |
|----------------|-------------------|------------------------------------|
| Persistent logs| Docker Volume     | Keeps logs safe & reusable         |
| Live code edit | Bind Mount        | Hot-reload development             |
| Structured config | `--mount`      | Safer, more readable               |
| Multi-service   | Docker Compose   | Manages volumes + services easily  |

---
