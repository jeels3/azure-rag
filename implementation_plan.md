# Azure AI Foundry — Employee Knowledge Assistant
### RAG-based Q&A Platform with .NET 10 & Microsoft Entra ID

A comprehensive plan to build an intelligent, enterprise-grade employee Q&A system backed by company documents, Azure AI Search, and Azure AI Foundry — with a secure Entra ID login and a clear roadmap for future AI capabilities.

---

## 🗂️ Project Overview

| Property | Value |
|---|---|
| **Framework** | .NET 10 (ASP.NET Core Web API + Blazor / React frontend) |
| **AI Platform** | Azure AI Foundry (Azure OpenAI + AI Search) |
| **Auth** | Microsoft Entra ID (formerly Azure AD) |
| **Document Store** | Azure Blob Storage + Azure AI Search Index |
| **Deployment** | Azure App Service / Azure Container Apps |

---

## User Review Required

> [!IMPORTANT]
> **Frontend Choice**: The plan proposes a **Blazor WebAssembly** frontend (to stay fully in the .NET ecosystem). If you prefer a separate **React/Next.js** SPA, please confirm before execution so the architecture can be adjusted.

> [!IMPORTANT]
> **Azure OpenAI Model**: The plan assumes **GPT-4o** deployed via Azure AI Foundry. Please confirm the model tier / region availability for your Azure subscription.

> [!WARNING]
> **Entra ID Tenant**: You will need an **Entra ID tenant** with permissions to register App Registrations (for both the API and the frontend). Make sure you have the required admin consent rights.

> [!NOTE]
> **Phase-Based Delivery**: The project is split into phases. Phase 1 (Core RAG Q&A + Auth) is the primary deliverable. Phases 2 & 3 (Tools, TTS, Translation) are future additions designed so that Phase 1 already lays the architectural groundwork.

---

## Open Questions

> [!IMPORTANT]
> 1. **Document types**: What formats will company documents be in? (PDF, DOCX, XLSX, HTML pages?) This affects the document ingestion pipeline.
> 2. **Scale**: Approximate number of employees who will use the system and volume of documents (GB of data)?
> 3. **Frontend**: Blazor WebAssembly vs React? (Default plan: Blazor)
> 4. **Roles**: Do different employee roles need access to different document sets (e.g., HR docs only for HR team)?
> 5. **On-premise documents**: Are any documents stored on-prem (SharePoint, file servers) that need to be synced?
> 6. **Chat history**: Should the system maintain multi-turn conversation history per user, or treat each question as standalone?
> 7. **Deployment target**: Azure App Service, Azure Container Apps, or AKS?

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        EMPLOYEE BROWSER / CLIENT                         │
│                    Blazor WASM / React Frontend App                      │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │ HTTPS + Entra ID Token (MSAL)
                            ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    ASP.NET Core 10 Web API (Backend)                     │
│  ┌──────────────┐  ┌───────────────┐  ┌────────────────────────────┐   │
│  │ Auth Middleware│  │  Chat / RAG   │  │   Admin / Document Upload  │   │
│  │ (Entra ID JWT)│  │  Controller   │  │      Controller            │   │
│  └──────────────┘  └───────┬───────┘  └───────────┬────────────────┘   │
└──────────────────────────────────────────────────────────────────────────┘
                      │                              │
          ┌───────────▼──────────┐    ┌─────────────▼──────────────┐
          │  Azure AI Foundry    │    │   Document Ingestion        │
          │  ┌────────────────┐  │    │   Pipeline (Azure Function  │
          │  │  Azure OpenAI  │  │    │   or background service)    │
          │  │  GPT-4o Model  │  │    └─────────────┬──────────────┘
          │  └────────────────┘  │                  │
          │  ┌────────────────┐  │    ┌─────────────▼──────────────┐
          │  │  AI Search     │◄─┼────│   Azure Blob Storage        │
          │  │  (RAG Index)   │  │    │   (Raw Documents)           │
          │  └────────────────┘  │    └────────────────────────────┘
          └──────────────────────┘
```

---

## Phase 1 — Core Platform (MVP)

### 1.1 Azure Infrastructure Setup

#### Azure Resources to Provision
| Resource | Purpose |
|---|---|
| **Azure AI Foundry Hub** | Central AI management, model deployments |
| **Azure OpenAI Service** | GPT-4o for Q&A generation |
| **Azure AI Search** | Vector + keyword hybrid search index |
| **Azure Blob Storage** | Raw document storage |
| **Azure Entra ID** | Identity & auth provider |
| **Azure App Service / Container App** | Host the .NET 10 backend |
| **Azure Static Web App** | Host Blazor WASM frontend |
| **Azure Key Vault** | Secure secrets management |
| **Azure Application Insights** | Telemetry & logging |

---

### 1.2 Solution Structure

```
AzureKnowledgeAssistant/
├── src/
│   ├── KnowledgeAssistant.Api/            # ASP.NET Core 10 Web API
│   │   ├── Controllers/
│   │   │   ├── ChatController.cs          # Q&A endpoints
│   │   │   ├── DocumentsController.cs     # Upload/manage docs
│   │   │   └── AdminController.cs         # Admin operations
│   │   ├── Services/
│   │   │   ├── RagService.cs              # RAG orchestration
│   │   │   ├── AzureSearchService.cs      # AI Search integration
│   │   │   ├── AzureOpenAIService.cs      # OpenAI calls
│   │   │   ├── DocumentIngestionService.cs# Chunking + indexing
│   │   │   └── TranslationService.cs      # (Phase 3) Azure Translator
│   │   ├── Models/
│   │   ├── Middleware/
│   │   │   └── EntraIdMiddleware.cs       # JWT validation
│   │   └── Program.cs
│   │
│   ├── KnowledgeAssistant.Web/            # Blazor WebAssembly Frontend
│   │   ├── Pages/
│   │   │   ├── Chat.razor                 # Main Q&A interface
│   │   │   ├── Documents.razor            # Document browser
│   │   │   └── Admin.razor                # Admin dashboard
│   │   ├── Components/
│   │   │   ├── ChatWindow.razor
│   │   │   ├── MessageBubble.razor
│   │   │   ├── DocumentUploader.razor
│   │   │   └── AudioPlayer.razor          # (Phase 2) TTS output
│   │   └── Services/
│   │       └── ApiClient.cs
│   │
│   ├── KnowledgeAssistant.Ingestion/      # Document Ingestion Worker
│   │   ├── DocumentChunker.cs
│   │   ├── EmbeddingGenerator.cs
│   │   └── IndexUpserter.cs
│   │
│   └── KnowledgeAssistant.Shared/        # Shared DTOs & Models
│       ├── ChatRequest.cs
│       ├── ChatResponse.cs
│       └── DocumentMetadata.cs
│
├── infra/                                 # IaC (Bicep / Terraform)
│   ├── main.bicep
│   ├── ai-search.bicep
│   ├── openai.bicep
│   ├── storage.bicep
│   └── entra-app-registration.ps1
│
├── tests/
│   ├── KnowledgeAssistant.Api.Tests/
│   └── KnowledgeAssistant.Ingestion.Tests/
│
└── .github/workflows/
    └── deploy.yml                         # CI/CD pipeline
```

---

### 1.3 Document Ingestion Pipeline

```
Document Upload (Admin/HR)
        │
        ▼
Azure Blob Storage (raw files)
        │
        ▼
Ingestion Worker Service
   ├── Parse document (PDF, DOCX, etc.) — using iTextSharp / DocumentFormat.OpenXml
   ├── Chunk text (sliding window: 512 tokens, 10% overlap)
   ├── Generate embeddings via Azure OpenAI (text-embedding-3-large)
   ├── Attach metadata (filename, dept, upload date, page number)
   └── Upsert chunks into Azure AI Search Index (vector + text fields)
```

**Azure AI Search Index Schema:**
```json
{
  "fields": [
    { "name": "id", "type": "Edm.String", "key": true },
    { "name": "content", "type": "Edm.String", "searchable": true },
    { "name": "contentVector", "type": "Collection(Edm.Single)", "vectorDimensions": 3072 },
    { "name": "sourceFile", "type": "Edm.String", "filterable": true },
    { "name": "department", "type": "Edm.String", "filterable": true },
    { "name": "pageNumber", "type": "Edm.Int32" },
    { "name": "uploadDate", "type": "Edm.DateTimeOffset", "filterable": true }
  ],
  "vectorSearch": { "algorithm": "hnsw" },
  "semantic": { "configurations": [{ "name": "default-semantic" }] }
}
```

---

### 1.4 RAG Q&A Flow

```
Employee asks a question
        │
        ▼
Backend API — ChatController
        │
        ▼
1. Generate query embedding (Azure OpenAI — text-embedding-3-large)
        │
        ▼
2. Hybrid Search: Azure AI Search
   ├── Vector similarity search (top-K chunks)
   └── Semantic ranker (rerank results)
        │
        ▼
3. Build prompt with retrieved context
   ├── System: "You are a helpful company knowledge assistant..."
   ├── Context: [Top-K document chunks]
   └── User: [Employee's question]
        │
        ▼
4. Call Azure OpenAI GPT-4o (streaming response)
        │
        ▼
5. Return answer + source citations to frontend
```

---

### 1.5 Microsoft Entra ID Integration

#### App Registrations
| App | Type | Purpose |
|---|---|---|
| `KnowledgeAssistant-API` | Web API | Exposes API scopes |
| `KnowledgeAssistant-Frontend` | SPA / Blazor WASM | MSAL login flow |

#### Authentication Flow
```
User visits app → Redirect to Entra ID login →
User authenticates (MFA optional) →
Entra issues JWT token →
Frontend attaches Bearer token to all API requests →
ASP.NET Core validates token (Microsoft.Identity.Web) →
User claims extracted (name, email, roles, department)
```

#### .NET 10 Configuration
```csharp
// Program.cs
builder.Services.AddMicrosoftIdentityWebApiAuthentication(builder.Configuration, "AzureAd");
builder.Services.AddAuthorization(options => {
    options.AddPolicy("EmployeePolicy", policy => policy.RequireRole("Employee", "Admin"));
    options.AddPolicy("AdminPolicy", policy => policy.RequireRole("Admin"));
});
```

#### Entra ID App Roles
| Role | Access |
|---|---|
| `Employee` | Ask questions, view answers |
| `HR` | Upload HR documents, view HR-only docs |
| `IT` | Upload IT docs, manage index |
| `Admin` | Full access, user management |

---

## Phase 2 — Tools & Extensibility

> **Planned after Phase 1 is stable**

### 2.1 AI Foundry Tool Calling (Function Calling)
- Define tools as `.NET` delegates/functions registered in the AI Foundry project
- Examples:
  - `GetEmployeeLeaveBalance(employeeId)` → calls HR API
  - `GetITPolicyDocument(policyName)` → fetches specific doc
  - `CreateSupportTicket(issue)` → creates JIRA/ServiceNow ticket
- Orchestration via **Azure AI Agent Service** (built into Foundry)

### 2.2 Agentic Workflows
- Multi-step reasoning using **Azure AI Foundry Agents SDK** for .NET
- Chain-of-thought for complex HR / IT queries
- Approval workflows for sensitive actions

---

## Phase 3 — Text-to-Speech & Language Translation

> **Planned after Phase 2 is stable**

### 3.1 Text-to-Audio (Azure AI Speech)
```
GPT-4o Answer Text
        │
        ▼
Azure AI Speech Service (TTS)
   ├── Neural voice (en-US-JennyNeural or custom)
   ├── SSML markup for natural pauses
   └── Stream audio back to Blazor AudioPlayer component
```
- REST endpoint: `POST /api/chat/speak`
- Returns audio stream (WAV/MP3)
- Frontend: HTML5 `<audio>` element auto-plays answer

### 3.2 Language Translation (Azure AI Translator)
```
Employee question (any language)
        │
        ▼
Azure AI Translator → Detect language → Translate to English
        │
        ▼
RAG Pipeline (runs in English)
        │
        ▼
Answer in English → Azure AI Translator → Translate back to employee's language
        │
        ▼
Display translated answer + TTS in detected language
```

**Supported workflow:**
- Auto language detection from input text
- Answer returned in the user's preferred language
- Voice output in the user's language via Azure Speech multilingual voices

---

## Proposed File Changes

### Infrastructure (Bicep IaC)

#### [NEW] `infra/main.bicep`
Main orchestration — deploys all Azure resources in one command via `az deployment group create`

#### [NEW] `infra/ai-foundry.bicep`
AI Foundry Hub + Project, OpenAI deployment (GPT-4o + text-embedding-3-large), AI Search

#### [NEW] `infra/entra-app-registration.ps1`
PowerShell script using `az ad app create` to set up App Registrations and App Roles

---

### Backend — ASP.NET Core 10

#### [NEW] `src/KnowledgeAssistant.Api/Program.cs`
Entry point — registers services, Entra ID auth, Swagger, CORS, Application Insights

#### [NEW] `src/KnowledgeAssistant.Api/Controllers/ChatController.cs`
- `POST /api/chat` — main Q&A endpoint (streaming SSE)
- `GET /api/chat/history` — conversation history per user

#### [NEW] `src/KnowledgeAssistant.Api/Controllers/DocumentsController.cs`
- `POST /api/documents/upload` — admin upload endpoint
- `GET /api/documents` — list indexed documents
- `DELETE /api/documents/{id}` — remove a document

#### [NEW] `src/KnowledgeAssistant.Api/Services/RagService.cs`
Core RAG orchestration — embedding → search → prompt build → OpenAI call

#### [NEW] `src/KnowledgeAssistant.Api/Services/AzureSearchService.cs`
Azure AI Search SDK wrapper — hybrid + semantic search

#### [NEW] `src/KnowledgeAssistant.Api/Services/AzureOpenAIService.cs`
Azure OpenAI SDK calls — completions + embeddings + streaming

#### [NEW] `src/KnowledgeAssistant.Api/Services/DocumentIngestionService.cs`
Document parsing (PDF/DOCX), chunking, embedding, index upsert

---

### Frontend — Blazor WebAssembly

#### [NEW] `src/KnowledgeAssistant.Web/Pages/Chat.razor`
Main chat interface with streaming answer display, source citation panel

#### [NEW] `src/KnowledgeAssistant.Web/Pages/Admin.razor`
Document upload dashboard, index management, user role assignment

#### [NEW] `src/KnowledgeAssistant.Web/Components/ChatWindow.razor`
Reusable chat UI with message bubbles, typing indicator, copy-to-clipboard

---

### Shared

#### [NEW] `src/KnowledgeAssistant.Shared/ChatRequest.cs`
DTO: `{ Question, ConversationId, Language?, UseDepartmentFilter? }`

#### [NEW] `src/KnowledgeAssistant.Shared/ChatResponse.cs`
DTO: `{ Answer, Citations[], ConversationId, DetectedLanguage }`

---

## Key NuGet Packages

| Package | Purpose |
|---|---|
| `Azure.AI.OpenAI` | Azure OpenAI SDK |
| `Azure.Search.Documents` | Azure AI Search SDK |
| `Azure.AI.Translation.Text` | (Phase 3) Azure Translator |
| `Azure.CognitiveServices.Speech` | (Phase 3) Azure TTS |
| `Microsoft.Identity.Web` | Entra ID / JWT auth |
| `Microsoft.Identity.Web.MicrosoftGraph` | Graph API for user info |
| `Microsoft.SemanticKernel` | Optional: SK orchestration layer |
| `Azure.Storage.Blobs` | Blob Storage SDK |
| `iTextSharp.LGPLv2.Core` | PDF parsing |
| `DocumentFormat.OpenXml` | DOCX parsing |
| `Serilog.AspNetCore` | Structured logging |

---

## CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
Steps:
  1. Checkout code
  2. Setup .NET 10
  3. Run unit tests
  4. Build API + Web project
  5. az login (OIDC — no secrets stored)
  6. Deploy Bicep infra (only on infra changes)
  7. Deploy API to Azure App Service / Container App
  8. Deploy Blazor WASM to Azure Static Web App
```

---

## Verification Plan

### Automated Tests
```bash
dotnet test tests/KnowledgeAssistant.Api.Tests/
dotnet test tests/KnowledgeAssistant.Ingestion.Tests/
```

### Manual Verification
- [ ] Entra ID login works (employee + admin roles)
- [ ] Upload a PDF document and verify it appears in AI Search index
- [ ] Ask a question and verify the answer cites the correct document
- [ ] Verify unauthorized users cannot call API endpoints
- [ ] Verify streaming response displays correctly in Blazor UI
- [ ] (Phase 3) Test TTS audio playback in browser
- [ ] (Phase 3) Ask a question in a non-English language and verify translated answer

---

## 🗓️ Phased Delivery Timeline

| Phase | Scope | Estimated Effort |
|---|---|---|
| **Phase 1** | Infra setup, Document ingestion, RAG Q&A, Entra ID login, Blazor UI | 3–4 weeks |
| **Phase 2** | Tool calling, AI Agent workflows | 2 weeks |
| **Phase 3** | TTS (text-to-audio) + Language Translation | 1–2 weeks |

---

> **Ready to start Phase 1?** Approve this plan and we'll begin with infrastructure provisioning (Bicep), followed by the .NET 10 solution scaffold.
