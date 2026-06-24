
# 📦 K5 - Project-i-Team-TravelPlanner

## 📖 Project Overview
This project is a production-oriented, cloud-ready Fullstack application built for the course *K5 - Projekt i team för kursen Cloud, DevOps & AI-integration*.

The core scenario is an **AI-assisted release note generator**: The application takes a raw text input describing sprint changes from the frontend, processes it via a secure Backend API, and requests an AI-generated summary and quality notes from an external AI service (e.g., Grok/OpenAI) via HTTP.

The project focuses on DevOps principles, cloud-native architecture, secure configuration, and robust error handling.

---

## 🛠️ Technologies
- **Backend:** .NET 9 / ASP.NET Core Web API
- **Frontend:** HTML5 / CSS3 / JavaScript (Minimal UI)
- **AI Integration:** External AI Service via HTTP Clients
- **Containerization:** Docker (Multi-stage builds)
- **CI/CD:** GitHub Actions
- **Cloud Platform:** Azure Container Apps & Azure Container Registry (ACR)
- **Security:** Azure Key Vault & Managed Identity

---

## 🧱 Project Structure
The repository is structured to separate concerns cleanly:
- `TravelPlanner.Api/Controllers/` → Handles incoming HTTP requests (`POST /api/summary`)
- `TravelPlanner.Api/Services/` → Contains `TravelService` handling communication with the AI service
- `TravelPlanner.Api/Middleware/` → Global Error Handling Middleware for 4xx/5xx responses
- `TravelPlanner.Api.Tests/` → Automated integration and unit tests (Executed via CI/CD)
- `.github/workflows/` → CI/CD pipeline automation

---

## 🐳 Dockerization & Multi-Stage Builds
The API has been containerized using a multi-stage `Dockerfile`. 
- **Stage 1 (Build):** Uses the full .NET 9 SDK to restore, build, and publish the app.
- **Stage 2 (Runtime):** Uses the lightweight ASP.NET runtime image, ensuring the final image only contains built artifacts, which drastically improves **performance**, **security**, and deployment speed.

### 🔒 Rootless Container Security
The container runs using a non-root user (`USER app`). This adheres to security best practices by minimizing the attack surface and reducing privileges in the event of a container breach.

---

## ⚙️ CI/CD Pipeline (GitHub Actions)
Our automated pipeline ensures continuous integration and deployment:
- **On Pull Request (PR) to `dev`:** Automatically runs `dotnet restore`, `dotnet build`, and executes the automated backend tests to block broken builds from merging.
- **On Merge to `main`:** Automatically builds the production-ready Docker container, tags it, pushes it to **Azure Container Registry (ACR)**, and deploys it to **Azure Container Apps**.

---

## 🔐 Security & Azure Key Vault Integration
We enforce zero hardcoded secrets throughout the application lifecycle:
- **Local Environment:** The AI API Key is kept in local `.env` files or User Secrets, which are strictly ignored by Git (`.gitignore`).
- **Production Environment (Azure):** The API Key is securely stored in an **Azure Key Vault** (`AI--ApiKey`). 
- **Managed Identity & RBAC:** Instead of saving connection strings, our Azure Container App uses a system-assigned **Managed Identity**. Using Azure RBAC (Role-Based Access Control), the app is granted specific rights to pull secrets from Key Vault at runtime without leaking credentials in code, Dockerfiles, or CI/CD pipelines.

---

## 🧪 Verification & Endpoint Test
The application can be verified locally or via the public Azure URL.

**HTTP Method:** `POST /api/summary`  
**Request Body:**
```json
{
  "text": "Fixed auth bug, introduced global exception handling, and optimized Dockerfile layers."
}

```

**Response Body:**

```json
{
  "summary": "This release fixes an authentication issue, adds global exception handling, and optimizes the container setup for better security and speed.",
  "qualityNotes": [
    "Möjlig hallucination om fakta saknas i input.",
    "Sammanfattningen blir sämre vid blandade språk.",
    "Kort input ger ofta generiska svar."
  ],
  "traceId": "00-4bf92f3577b34da6a3ce929d0e0e4736-00"
}

```

*Note: If a 4xx/5xx error occurs, the frontend displays a clear, user-friendly message without leaking inner technical exceptions.*

---

## 📊 AI Evaluation & Quality Assessment (Skriftlig Utvärdering)

### Fullstack Flow

`Frontend (UI input) ──> Backend API (Validation & Tracing) ──> External AI (HTTP Request) ──> Structured UI Response`

### AI Quality: Test with 3 Different Inputs

1. **Short Input ("fixed buttons"):** The AI creates a generic note due to lack of context. Our system correctly returns a predefined `qualityNote` alerting the user that short inputs yield lower quality summaries.
2. **Normal Input ("Fixed validation on backend and resolved crash on empty inputs"):** The AI performs exceptionally well, translating technical tasks into professional, business-readable release notes.
3. **Messy Input ("uh, did some docker stuff and fixed a bug with c# i think but now it builds fine, also modified button colors"):** The AI successfully filters out conversational noise to extract key changes (Docker updates and UI adjustments), though prompt clarity remains crucial to prevent subtle misunderstandings.

### Risks & Mitigations

* **Hallucinations:** Mitigated by backend validation (requests with empty or short text return a `400 Bad Request` before reaching the AI).
* **Rate Limits (429) & Timeouts:** Managed through custom `ExceptionMiddleware` (Global Error Handling). If the AI provider returns a `429` or network timeout, the backend catches it gracefully, logs metrics securely, and issues a clean `503/504` status code with a user-friendly retry message.
* **Data Leakage:** Our logging policy ensures that latency and `traceId` are logged for troubleshooting, but **never** stores API keys, headers, or raw prompt logs.

---

## 👨‍💻 Authors

* Abdalle Abdulkadir
* Sepideh Shoghirabani

```
