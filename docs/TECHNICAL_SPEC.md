# TR4NSFORM — Technical Specification

> Personal biohacking & optimization platform combining Human Design, biomarker tracking, peptide protocols, and AI-driven recommendations.

**Version:** 1.0.0  
**Date:** 2026-05-03  
**Author:** Hermes Agent for Tobias

---

## Table of Contents

1. [Overview](#1-overview)
2. [Architecture](#2-architecture)
3. [Tech Stack](#3-tech-stack)
4. [Data Models](#4-data-models)
5. [Module Specifications](#5-module-specifications)
6. [API Design](#6-api-design)
7. [AI Integration](#7-ai-integration)
8. [Security & Compliance](#8-security--compliance)
9. [Development Phases](#9-development-phases)
10. [Infrastructure](#10-infrastructure)

---

## 1. Overview

### 1.1 Product Vision

Tr4nsform is a comprehensive personal optimization platform that:
- Integrates Human Design chart analysis with health recommendations
- Tracks blood work and biomarkers with intelligent interpretation
- Provides complete peptide knowledge base and protocol builder
- Uses AI to connect all data points for personalized guidance

### 1.2 Target Users

- Biohackers and self-optimizers
- Longevity-focused individuals
- Athletes and performance seekers
- Practitioners managing client protocols
- Anyone interested in peptide therapy

### 1.3 Core Value Proposition

**One platform that connects the dots** — your genetics (HD), your biomarkers (labs), your protocols (peptides/supplements), and your progress (body metrics) — giving you personalized answers, not generic advice.

---

## 2. Architecture

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                              │
├─────────────────────────────────────────────────────────────────┤
│  Mobile App (React Native)  │  Web App (React)  │  Admin Panel  │
└─────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────┐
│                         API GATEWAY                              │
│                    (FastAPI + Auth Layer)                        │
└─────────────────────────────────────────────────────────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
┌───────────────┐         ┌───────────────┐         ┌───────────────┐
│  Core Services │         │  AI Services   │         │  Data Services │
├───────────────┤         ├───────────────┤         ├───────────────┤
│ User Service  │         │ Chat Coach    │         │ Lab Parser    │
│ HD Engine     │         │ Protocol Gen  │         │ OCR Service   │
│ Lab Tracker   │         │ Biomarker AI  │         │ File Storage  │
│ Peptide DB    │         │ Embeddings    │         │ Analytics     │
│ Protocol Svc  │         └───────────────┘         └───────────────┘
│ Metrics Svc   │                  │
└───────────────┘                  ▼
        │                 ┌───────────────┐
        │                 │  LLM Provider  │
        │                 │ (Claude/GPT)   │
        │                 └───────────────┘
        ▼
┌─────────────────────────────────────────────────────────────────┐
│                        DATA LAYER                                │
├─────────────────────────────────────────────────────────────────┤
│  PostgreSQL (Primary)  │  pgvector (Embeddings)  │  Redis (Cache) │
│  S3/Minio (Files)      │  TimescaleDB (Metrics)                   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Service Breakdown

| Service | Responsibility | Tech |
|---------|---------------|------|
| **API Gateway** | Auth, rate limiting, routing | FastAPI + JWT |
| **User Service** | Profiles, onboarding, settings | FastAPI |
| **HD Engine** | Chart calculation, interpretation | Python + custom HD lib |
| **Lab Tracker** | Biomarker storage, trend analysis | FastAPI + PostgreSQL |
| **Peptide DB** | Peptide encyclopedia, search | FastAPI + pgvector |
| **Protocol Service** | Protocol generation, injection plans | FastAPI + AI |
| **Metrics Service** | Body metrics, progress tracking | TimescaleDB |
| **AI Coach** | Conversational interface, Q&A | FastAPI + LLM |
| **OCR Service** | Lab PDF parsing | Google Vision / Tesseract |

---

## 3. Tech Stack

### 3.1 Frontend

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Mobile | React Native 0.74+ | iOS + Android from one codebase |
| State | Zustand | Lightweight, no boilerplate |
| Navigation | React Navigation 7 | Industry standard |
| UI Kit | Tamagui | Cross-platform, performant |
| Charts | Victory Native | Health data visualization |
| Forms | React Hook Form + Zod | Validation |
| API Client | TanStack Query | Caching, background sync |

### 3.2 Backend

| Layer | Technology | Rationale |
|-------|------------|-----------|
| Framework | FastAPI (Python 3.12) | Async, type-safe, fast |
| ORM | SQLAlchemy 2.0 + Alembic | Migrations, async support |
| Validation | Pydantic v2 | Native FastAPI integration |
| Auth | JWT + OAuth2 | Mobile-friendly |
| Task Queue | Celery + Redis | Background jobs (OCR, AI) |
| Caching | Redis | Session, rate limits |

### 3.3 Database

| Database | Purpose |
|----------|---------|
| PostgreSQL 16 | Primary data store |
| pgvector | Peptide/HD embeddings for semantic search |
| TimescaleDB | Time-series metrics (body stats, HRV) |
| Redis | Cache, sessions, rate limits |
| S3/Minio | File storage (lab PDFs, images) |

### 3.4 AI/ML

| Component | Technology |
|-----------|------------|
| LLM Provider | Claude API (primary), GPT-5.4 (fallback) |
| Embeddings | OpenAI text-embedding-3-large |
| OCR | Google Cloud Vision API |
| Vector Search | pgvector |

### 3.5 Infrastructure

| Component | Technology |
|-----------|------------|
| Container | Docker |
| Orchestration | Docker Compose (MVP) → Kubernetes (scale) |
| CI/CD | GitHub Actions |
| Hosting | AWS (ECS/RDS) or self-hosted VPS |
| CDN | Cloudflare |
| Monitoring | Prometheus + Grafana |

---

## 4. Data Models

### 4.1 Core Entities

#### User
```python
class User:
    id: UUID
    email: str
    password_hash: str
    created_at: datetime
    
    # Profile
    name: str
    birth_date: date
    birth_time: time | None
    birth_location: str | None  # For HD calculation
    sex: Literal["male", "female"]
    
    # Body stats
    height_cm: float
    weight_kg: float
    body_fat_pct: float | None
    
    # Goals
    primary_goal: Literal["performance", "longevity", "healing", "hormonal", "mental", "body_comp"]
    secondary_goals: list[str]
    
    # Lifestyle
    sleep_hours_avg: float
    stress_level: int  # 1-10
    diet_type: str
    training_frequency: int  # days per week
    
    # Settings
    units: Literal["metric", "imperial"]
    timezone: str
    notifications_enabled: bool
```

#### HumanDesignChart
```python
class HumanDesignChart:
    id: UUID
    user_id: UUID
    calculated_at: datetime
    
    # Core
    type: Literal["Manifestor", "Generator", "MG", "Projector", "Reflector"]
    strategy: str
    authority: str
    profile: str  # e.g. "5/1"
    definition: str  # Single, Split, Triple Split, Quadruple Split, None
    incarnation_cross: str
    
    # Centers (9 centers, defined or undefined)
    centers: dict[str, CenterData]
    # e.g. {"head": {"defined": True, "gates": [64, 61]}, ...}
    
    # Channels (36 possible)
    channels: list[ChannelData]
    
    # Gates (64 possible, from planets)
    gates: list[GateData]
    
    # Variables (if advanced)
    variables: dict | None
    
    # Raw planetary positions for recalculation
    planetary_positions: dict
```

#### LabResult
```python
class LabResult:
    id: UUID
    user_id: UUID
    test_date: date
    lab_name: str | None
    source: Literal["manual", "ocr", "api"]
    pdf_url: str | None
    
    markers: list[LabMarker]

class LabMarker:
    id: UUID
    lab_result_id: UUID
    
    name: str  # e.g. "Testosterone, Total"
    category: str  # e.g. "Hormones"
    value: float
    unit: str
    
    # Ranges
    ref_range_low: float | None
    ref_range_high: float | None
    optimal_range_low: float | None
    optimal_range_high: float | None
    
    # Status
    status: Literal["low", "suboptimal", "optimal", "high", "critical"]
    
    # Metadata
    notes: str | None
```

#### Peptide
```python
class Peptide:
    id: UUID
    slug: str  # e.g. "bpc-157"
    name: str  # e.g. "BPC-157"
    full_name: str  # e.g. "Body Protection Compound-157"
    
    # Classification
    category: str  # e.g. "Healing", "GH Secretagogue", "Cognitive"
    subcategory: str | None
    
    # Content (markdown)
    description: str
    mechanism_of_action: str
    benefits: str
    research_summary: str
    side_effects: str
    contraindications: str
    
    # Dosing
    dosing_protocols: list[DosingProtocol]
    reconstitution_guide: str
    injection_type: list[Literal["SubQ", "IM", "IV", "Oral", "Nasal"]]
    
    # Cycles
    typical_cycle_length_weeks: int
    on_off_pattern: str | None  # e.g. "5 on / 2 off"
    
    # Stacking
    synergistic_peptides: list[str]  # slugs
    avoid_with: list[str]  # slugs
    
    # Lab marker interactions
    affects_markers: list[MarkerInteraction]
    
    # Metadata
    legal_status: dict[str, str]  # country -> status
    embedding: list[float]  # for semantic search

class DosingProtocol:
    name: str  # e.g. "Standard", "Loading", "Maintenance"
    dose_mcg: float
    frequency: str  # e.g. "2x daily", "EOD", "weekly"
    timing: str  # e.g. "AM fasted", "pre-sleep"
    duration_weeks: int
    notes: str | None
```

#### Protocol
```python
class Protocol:
    id: UUID
    user_id: UUID
    name: str
    created_at: datetime
    status: Literal["draft", "active", "paused", "completed"]
    
    # Goals this protocol targets
    goals: list[str]
    
    # Based on
    based_on_labs: list[UUID] | None
    based_on_hd: bool
    
    # Items
    items: list[ProtocolItem]
    
    # Schedule
    start_date: date
    end_date: date | None
    
    # Progress
    adherence_pct: float
    notes: str | None

class ProtocolItem:
    id: UUID
    protocol_id: UUID
    
    item_type: Literal["peptide", "supplement"]
    item_id: UUID  # peptide_id or supplement_id
    item_name: str
    
    # Dosing
    dose: float
    dose_unit: str
    frequency: str
    timing: str
    injection_site: str | None  # for peptides
    
    # Notes
    reconstitution_notes: str | None
    special_instructions: str | None
```

#### InjectionLog
```python
class InjectionLog:
    id: UUID
    user_id: UUID
    protocol_item_id: UUID
    
    timestamp: datetime
    peptide_name: str
    dose_mcg: float
    injection_site: str  # e.g. "abdomen_left", "thigh_right"
    
    # Optional
    batch_number: str | None
    notes: str | None
    reaction: str | None  # any immediate reaction
```

#### BodyMetric
```python
class BodyMetric:
    id: UUID
    user_id: UUID
    recorded_at: datetime
    
    # Measurements
    weight_kg: float | None
    body_fat_pct: float | None
    muscle_mass_kg: float | None
    
    # Subjective (1-10)
    energy_level: int | None
    sleep_quality: int | None
    mood: int | None
    libido: int | None
    cognition: int | None
    recovery: int | None
    
    # Wearable data
    hrv_ms: float | None
    resting_hr: int | None
    sleep_hours: float | None
    deep_sleep_pct: float | None
    
    # Notes
    notes: str | None
```

### 4.2 Supporting Entities

```python
# Supplements (similar structure to Peptide, simplified)
class Supplement:
    id: UUID
    slug: str
    name: str
    category: str
    description: str
    benefits: str
    dosing: str
    timing: str
    interactions: str
    embedding: list[float]

# Chat history for AI coach
class ChatMessage:
    id: UUID
    user_id: UUID
    role: Literal["user", "assistant"]
    content: str
    timestamp: datetime
    context_used: dict | None  # what data was pulled for this response

# Notifications / Reminders
class Reminder:
    id: UUID
    user_id: UUID
    type: Literal["injection", "supplement", "lab_test", "checkin"]
    scheduled_at: datetime
    sent_at: datetime | None
    message: str
```

---

## 5. Module Specifications

### 5.1 Module 1: User Profile & Onboarding

**Screens:**
1. Welcome / Sign Up
2. Basic Info (name, email, password)
3. Body Stats (DOB, sex, height, weight)
4. Human Design Input (birth time, birth location)
5. Goals Selection (primary + secondary)
6. Lifestyle Assessment (sleep, stress, diet, training)
7. Onboarding Complete → Dashboard

**API Endpoints:**
```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
GET    /users/me
PATCH  /users/me
POST   /users/me/onboarding
```

**Business Logic:**
- Validate birth time format
- Geocode birth location for HD calculation
- Trigger HD chart calculation after birth data submitted
- Calculate BMI, TDEE estimates

---

### 5.2 Module 2: Human Design Engine

**Features:**
- Full bodygraph visualization (SVG/Canvas)
- Type, Strategy, Authority explanation
- Center-by-center breakdown with health implications
- Gate health correlations
- Personalized recommendations based on chart

**HD Calculation:**
- Use Swiss Ephemeris for planetary positions
- Calculate Design (88° before birth) and Personality (birth moment)
- Map to gates and channels
- Determine type, authority, profile

**Health Correlations Database:**
```python
GATE_HEALTH_CORRELATIONS = {
    20: {
        "center": "Throat",
        "body_system": "Thyroid",
        "themes": ["Metabolism", "Expression", "Timing"],
        "supplements": ["Selenium", "Zinc", "Iodine"],
        "peptides": ["Thymosin Alpha-1"],
        "markers_to_watch": ["TSH", "T3", "T4"],
    },
    57: {
        "center": "Spleen",
        "body_system": "Immune/Lymphatic",
        "themes": ["Intuition", "Immune response", "Survival"],
        "supplements": ["Vitamin C", "Zinc", "Elderberry"],
        "peptides": ["LL-37", "Thymosin Alpha-1"],
        "markers_to_watch": ["WBC", "Lymphocytes", "CRP"],
    },
    # ... all 64 gates
}

CENTER_HEALTH_THEMES = {
    "Spleen": {
        "defined": "Strong immune system, intuitive health awareness",
        "undefined": "Sensitive to environment, may hold onto things too long",
        "health_focus": ["Immune support", "Detox", "Listening to body signals"],
    },
    # ... all 9 centers
}
```

**API Endpoints:**
```
GET    /hd/chart                    # Get user's chart
POST   /hd/chart/calculate          # Trigger calculation
GET    /hd/chart/interpretation     # Full interpretation
GET    /hd/gates/{gate_number}      # Gate details
GET    /hd/centers/{center_name}    # Center details
GET    /hd/recommendations          # Health recs based on chart
```

---

### 5.3 Module 3: Blood & Lab Tracker

**Features:**
- Manual marker entry
- PDF upload with OCR extraction
- 40+ biomarkers supported
- Reference ranges + optimal ranges
- Trend charts over time
- AI interpretation per marker

**Supported Marker Categories:**

| Category | Markers |
|----------|---------|
| **Blood Count** | RBC, WBC, Hemoglobin, Hematocrit, Platelets, MCV, MCH, MCHC |
| **Metabolic** | Glucose, Insulin, HbA1c, HOMA-IR |
| **Hormones** | Total T, Free T, Estradiol, Progesterone, DHEA-S, Cortisol (AM), SHBG |
| **Thyroid** | TSH, Free T3, Free T4, Reverse T3, TPO Antibodies |
| **Inflammatory** | CRP (hs-CRP), ESR, Homocysteine, IL-6, TNF-alpha |
| **Vitamins** | Vitamin D (25-OH), B12, Folate, Vitamin A |
| **Minerals** | Ferritin, Iron, TIBC, Zinc, Magnesium (RBC), Selenium |
| **Liver** | AST, ALT, GGT, Bilirubin, Albumin, ALP |
| **Kidney** | BUN, Creatinine, eGFR, Uric Acid |
| **Lipids** | Total Cholesterol, LDL, HDL, Triglycerides, ApoB, Lp(a) |
| **Growth** | IGF-1, GH (if available) |
| **Gut** | Zonulin, Calprotectin (if available) |

**Optimal Ranges:**
```python
OPTIMAL_RANGES = {
    "testosterone_total": {
        "male": {"low": 600, "high": 900, "unit": "ng/dL"},
        "female": {"low": 30, "high": 70, "unit": "ng/dL"},
    },
    "vitamin_d": {
        "low": 50, "high": 80, "unit": "ng/mL",
        "note": "Most labs say 30-100, but 50-80 is optimal"
    },
    "ferritin": {
        "male": {"low": 75, "high": 150, "unit": "ng/mL"},
        "female": {"low": 50, "high": 150, "unit": "ng/mL"},
    },
    # ... all markers
}
```

**API Endpoints:**
```
GET    /labs                        # List all lab results
POST   /labs                        # Create new lab result
GET    /labs/{id}                   # Get specific result
DELETE /labs/{id}                   # Delete result
POST   /labs/upload                 # Upload PDF for OCR
GET    /labs/markers                # List supported markers
GET    /labs/markers/{name}/history # Trend data for marker
GET    /labs/interpretation         # AI interpretation of latest
```

---

### 5.4 Module 4: Peptide Database

**Content per Peptide:**
- Name, full name, aliases
- Category and mechanism
- Benefits (bullet points)
- Research summary with citations
- Dosing protocols (beginner, standard, advanced)
- Reconstitution calculator
- Injection guidance
- Cycle recommendations
- Stacking compatibility
- Side effects and contraindications
- Interactions with lab markers
- Legal status by region

**Initial Peptide List (50+):**

| Category | Peptides |
|----------|----------|
| **Healing/Repair** | BPC-157, TB-500, Thymosin Beta-4, GHK-Cu, Pentadecapeptide |
| **GH Secretagogues** | CJC-1295, Ipamorelin, Tesamorelin, GHRP-2, GHRP-6, MK-677 (oral), Hexarelin |
| **Fat Loss** | AOD-9604, Fragment 176-191, Tirzepatide, Semaglutide |
| **Cognitive** | Semax, Selank, Dihexa, P21, FGL, NSI-189 |
| **Sexual** | PT-141, Melanotan II, Kisspeptin |
| **Longevity** | Epitalon, MOTS-c, SS-31 (Elamipretide), Humanin |
| **Immune** | Thymosin Alpha-1, LL-37, KPV |
| **Gut** | BPC-157, KPV, Larazotide |
| **Skin/Hair** | GHK-Cu, Copper Peptides, Follistatin |
| **Inflammation** | VIP, Alpha-MSH, KPV |
| **Muscle** | Follistatin, ACE-031, YK-11 (SARM) |
| **Sleep** | DSIP (Delta Sleep Inducing Peptide) |

**API Endpoints:**
```
GET    /peptides                    # List all (paginated)
GET    /peptides/{slug}             # Get details
GET    /peptides/search?q=          # Semantic search
GET    /peptides/categories         # List categories
GET    /peptides/by-goal/{goal}     # Filter by goal
GET    /peptides/{slug}/protocols   # Dosing protocols
GET    /peptides/{slug}/stacks      # Compatible stacks
```

---

### 5.5 Module 5: Peptide Protocol Builder

**Input:**
- User goals (from profile)
- Current lab results
- HD chart (optional integration)
- Existing conditions/contraindications
- Budget constraints
- Experience level

**Output:**
- Recommended peptide stack
- Full protocol with:
  - Each peptide name
  - Dose per injection (mcg)
  - Frequency (daily, EOD, 5/2, etc.)
  - Timing (AM fasted, pre-bed, post-workout)
  - Injection site rotation schedule
  - Cycle length (weeks)
  - On/off pattern
  - Reconstitution instructions
- Calendar view
- Shopping list (what to buy)
- Estimated cost

**AI Protocol Generation:**
```python
async def generate_protocol(
    user: User,
    goals: list[str],
    labs: LabResult | None,
    hd_chart: HumanDesignChart | None,
    contraindications: list[str],
    experience_level: Literal["beginner", "intermediate", "advanced"],
    budget: Literal["low", "medium", "high"],
) -> Protocol:
    # 1. Gather context
    context = build_context(user, goals, labs, hd_chart)
    
    # 2. Query peptide DB for candidates
    candidates = await search_peptides_by_goals(goals)
    
    # 3. Filter by contraindications
    candidates = filter_contraindicated(candidates, contraindications)
    
    # 4. Rank by lab marker needs
    if labs:
        candidates = rank_by_lab_gaps(candidates, labs)
    
    # 5. Apply HD insights if available
    if hd_chart:
        candidates = apply_hd_recommendations(candidates, hd_chart)
    
    # 6. Build protocol via LLM
    protocol = await llm_build_protocol(
        candidates=candidates,
        context=context,
        experience_level=experience_level,
        budget=budget,
    )
    
    return protocol
```

**API Endpoints:**
```
POST   /protocols/generate          # AI-generate protocol
GET    /protocols                   # List user's protocols
POST   /protocols                   # Create manual protocol
GET    /protocols/{id}              # Get protocol details
PATCH  /protocols/{id}              # Update protocol
DELETE /protocols/{id}              # Delete protocol
POST   /protocols/{id}/activate     # Start protocol
POST   /protocols/{id}/pause        # Pause protocol
GET    /protocols/{id}/calendar     # Calendar view
GET    /protocols/{id}/shopping     # Shopping list
```

---

### 5.6 Module 6: Supplement Stack Builder

**Same concept as peptide builder but for oral supplements:**
- Vitamins, minerals, adaptogens, nootropics
- Based on lab gaps + HD + goals
- Dosing, timing, cycling
- Interaction checker
- Meal timing recommendations

**Supplement Categories:**
- Foundational (D3, K2, Magnesium, Omega-3, B-complex)
- Hormonal (Tongkat Ali, Fadogia, DIM, Ashwagandha)
- Cognitive (Lion's Mane, Alpha-GPC, Bacopa, Phosphatidylserine)
- Sleep (Magnesium Glycinate, Apigenin, L-Theanine, Glycine)
- Performance (Creatine, Beta-Alanine, Citrulline)
- Longevity (NMN, Resveratrol, Quercetin, Fisetin)
- Gut (Probiotics, L-Glutamine, Digestive Enzymes)

**API Endpoints:**
```
GET    /supplements                 # List all
GET    /supplements/{slug}          # Get details
POST   /supplements/stack/generate  # AI-generate stack
GET    /supplements/interactions    # Check interactions
```

---

### 5.7 Module 7: AI Biomarker Coach

**Features:**
- Conversational interface
- Access to all user data (labs, HD, protocols, metrics)
- Answers questions like:
  - "Why is my testosterone low?"
  - "What peptide helps with high CRP?"
  - "Based on my HD, what should I focus on?"
  - "Is my protocol working? Look at my metrics."
- Remembers conversation context
- Can suggest protocol adjustments

**Implementation:**
```python
async def chat(user_id: UUID, message: str) -> str:
    user = await get_user(user_id)
    
    # Retrieve relevant context
    context = {
        "profile": user.profile,
        "hd_chart": await get_hd_chart(user_id),
        "latest_labs": await get_latest_labs(user_id),
        "active_protocol": await get_active_protocol(user_id),
        "recent_metrics": await get_recent_metrics(user_id, days=14),
        "chat_history": await get_chat_history(user_id, limit=10),
    }
    
    # Semantic search for relevant peptide/supplement info
    relevant_docs = await semantic_search(message, top_k=5)
    
    # Build prompt
    system_prompt = build_coach_system_prompt(context, relevant_docs)
    
    # Call LLM
    response = await llm_chat(
        system=system_prompt,
        messages=context["chat_history"] + [{"role": "user", "content": message}],
    )
    
    # Save to history
    await save_chat_message(user_id, "user", message)
    await save_chat_message(user_id, "assistant", response)
    
    return response
```

**API Endpoints:**
```
POST   /chat                        # Send message
GET    /chat/history                # Get chat history
DELETE /chat/history                # Clear history
```

---

### 5.8 Module 8: Body Metrics Tracker

**Daily Check-in:**
- Weight (optional)
- Body fat % (optional)
- Subjective scores (1-10): Energy, Sleep, Mood, Libido, Cognition, Recovery
- Notes

**Wearable Sync (Phase 2):**
- Oura Ring: Sleep, HRV, Readiness
- Whoop: Strain, Recovery, Sleep
- Apple Health / Google Fit: Steps, HR, Sleep
- CGM: Glucose data

**Visualizations:**
- Line charts per metric over time
- Correlation with protocol phases
- Before/after comparisons

**API Endpoints:**
```
POST   /metrics                     # Log daily metrics
GET    /metrics                     # Get history
GET    /metrics/summary             # Aggregate stats
GET    /metrics/correlations        # Correlate with protocols
```

---

### 5.9 Module 9: Injection Logbook

**Features:**
- Log each injection with one tap
- Pre-filled from active protocol
- Site rotation tracker (visual body map)
- History view
- Export to PDF/CSV

**Injection Sites:**
```python
INJECTION_SITES = [
    "abdomen_left",
    "abdomen_right",
    "abdomen_center",
    "thigh_left_upper",
    "thigh_left_lower",
    "thigh_right_upper",
    "thigh_right_lower",
    "deltoid_left",
    "deltoid_right",
    "glute_left",
    "glute_right",
    "love_handle_left",
    "love_handle_right",
]
```

**API Endpoints:**
```
POST   /injections                  # Log injection
GET    /injections                  # History
GET    /injections/sites            # Site rotation status
GET    /injections/export           # Export PDF/CSV
```

---

### 5.10 Module 10: Education Hub

**Content:**
- Beginner's Guide to Peptides
- How to Reconstitute Peptides (step-by-step)
- Injection Technique (with illustrations/video)
- Syringe & Needle Selection Guide
- Storage & Handling Best Practices
- Understanding Your Labs
- Human Design Basics
- Legal Considerations by Country
- FAQ

**Format:**
- Markdown content stored in DB
- Rich media (images, videos) in S3
- Searchable

**API Endpoints:**
```
GET    /education                   # List articles
GET    /education/{slug}            # Get article
GET    /education/search?q=         # Search
```

---

### 5.11 Module 11: Integrations (Phase 2)

**Wearables:**
- Oura Ring (OAuth2)
- Whoop (OAuth2)
- Apple Health (HealthKit)
- Google Fit (OAuth2)
- Garmin (OAuth2)

**CGM:**
- Levels (API)
- Dexcom (API)
- Libre (via LibreLink or manual)

**Lab Ordering:**
- LabCorp (partner API)
- Quest (partner API)
- Private labs (Marek Health, etc.)

---

### 5.12 Module 12: Practitioner Mode (Phase 2)

**Features:**
- Practitioner account type
- Manage multiple clients
- View client data (with consent)
- Suggest protocols
- Messaging
- Consent & disclaimer management
- White-label option

---

## 6. API Design

### 6.1 Authentication

**JWT-based auth with refresh tokens:**
```
Access Token: 15 min expiry
Refresh Token: 7 day expiry, stored in httpOnly cookie
```

**Endpoints:**
```
POST /auth/register     # Create account
POST /auth/login        # Get tokens
POST /auth/refresh      # Refresh access token
POST /auth/logout       # Invalidate refresh token
POST /auth/password/reset-request
POST /auth/password/reset-confirm
```

### 6.2 API Conventions

- RESTful design
- JSON request/response
- Pagination: `?page=1&per_page=20`
- Filtering: `?category=healing&search=tendon`
- Sorting: `?sort=-created_at` (prefix `-` for desc)
- Error format:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [
      {"field": "email", "message": "Invalid email format"}
    ]
  }
}
```

### 6.3 Rate Limiting

| Endpoint Type | Limit |
|---------------|-------|
| Auth endpoints | 10/min |
| Standard API | 100/min |
| AI endpoints (chat, generate) | 20/min |
| File upload | 10/min |

---

## 7. AI Integration

### 7.1 LLM Provider

**Primary:** Claude API (Anthropic)
- claude-3-5-sonnet for most tasks
- claude-3-opus for complex reasoning

**Fallback:** OpenAI GPT-5.4
- For redundancy if Claude is down

### 7.2 Use Cases

| Feature | Model | Input | Output |
|---------|-------|-------|--------|
| Lab interpretation | Sonnet | Lab results JSON | Markdown analysis |
| Protocol generation | Opus | Goals, labs, HD, peptide DB | Protocol JSON |
| Chat coach | Sonnet | User message + context | Response text |
| Semantic search | text-embedding-3-large | Query text | Vector |
| OCR correction | Sonnet | Raw OCR text | Structured markers |

### 7.3 Prompt Engineering

**System prompts stored in:**
```
/prompts/
  lab_interpretation.md
  protocol_generator.md
  chat_coach.md
  ocr_corrector.md
```

**Context injection pattern:**
```python
def build_chat_prompt(user_context: dict, message: str) -> list:
    return [
        {"role": "system", "content": CHAT_COACH_SYSTEM_PROMPT},
        {"role": "system", "content": f"User context:\n{json.dumps(user_context)}"},
        {"role": "user", "content": message},
    ]
```

### 7.4 Cost Management

- Cache common queries (Redis)
- Truncate context to last N messages
- Use cheaper models for simple tasks
- Batch embeddings generation
- Rate limit AI endpoints

---

## 8. Security & Compliance

### 8.1 Data Security

- All data encrypted at rest (AES-256)
- TLS 1.3 for transit
- Database credentials in secrets manager
- No PII in logs
- Regular security audits

### 8.2 Health Data Compliance

**Considerations:**
- HIPAA (if US users with health data)
- GDPR (EU users)
- Data minimization
- Right to deletion
- Consent management

**Approach for MVP:**
- Clear Terms of Service: "Not medical advice"
- User data ownership: export/delete anytime
- No selling data to third parties
- Optional: host in HIPAA-compliant region

### 8.3 Disclaimers

Every protocol/recommendation must include:
> This is for informational purposes only. Consult a qualified healthcare provider before starting any peptide or supplement protocol.

---

## 9. Development Phases

### Phase 1 — MVP (8-10 weeks)

**Goal:** Core product with manual data entry, working peptide DB, basic protocol builder.

| Week | Deliverable |
|------|-------------|
| 1-2 | Backend setup, auth, user service |
| 3-4 | Lab tracker (manual entry), marker DB |
| 5-6 | Peptide database (top 30), detail screens |
| 7-8 | Basic protocol builder, HD engine v1 |
| 9-10 | Mobile app MVP, testing, polish |

**Features:**
- User registration & onboarding
- Human Design calculation & display
- Manual lab entry & tracking
- Peptide encyclopedia (30 peptides)
- Basic protocol builder (template-based)
- Simple metrics logging

**Tech:**
- Backend deployed on VPS or AWS
- Mobile app on TestFlight / internal testing

---

### Phase 2 — Core Product (8-10 weeks)

**Goal:** AI features, OCR, full peptide library, advanced protocols.

| Week | Deliverable |
|------|-------------|
| 1-2 | AI chat coach integration |
| 3-4 | OCR lab upload, parsing |
| 5-6 | Full peptide library (50+), supplement DB |
| 7-8 | AI protocol generator, injection logbook |
| 9-10 | HD-health correlations, testing |

**Features:**
- AI biomarker coach (conversational)
- Lab PDF upload with OCR
- Expanded peptide DB (50+)
- Supplement stack builder
- AI-powered protocol generation
- Injection logbook with site rotation
- HD health correlations complete
- Progress tracking & correlations

---

### Phase 3 — Scale (6-8 weeks)

**Goal:** Integrations, practitioner mode, polish.

| Week | Deliverable |
|------|-------------|
| 1-2 | Wearable integrations (Oura, Whoop) |
| 3-4 | CGM integration, Apple Health |
| 5-6 | Practitioner mode, client management |
| 7-8 | Lab ordering integration, white-label |

**Features:**
- Oura, Whoop, Garmin sync
- Apple Health / Google Fit
- CGM data integration
- Practitioner accounts
- Client management
- Lab ordering partnerships
- Community features (optional)
- App store launch

---

## 10. Infrastructure

### 10.1 MVP Infrastructure

```
┌─────────────────────────────────────────┐
│            Cloudflare CDN               │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│              VPS / AWS EC2              │
├─────────────────────────────────────────┤
│  Docker Compose                         │
│  ├── fastapi-app (uvicorn)              │
│  ├── celery-worker                      │
│  ├── redis                              │
│  └── postgres                           │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│           S3 / Minio (files)            │
└─────────────────────────────────────────┘
```

### 10.2 Production Infrastructure

```
┌─────────────────────────────────────────┐
│            Cloudflare CDN               │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│          AWS Application LB             │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│              ECS Fargate                │
├─────────────────────────────────────────┤
│  ├── api-service (auto-scaling)         │
│  ├── celery-worker (auto-scaling)       │
│  └── celery-beat                        │
└─────────────────────────────────────────┘
        │                   │
┌───────────────┐   ┌───────────────┐
│  RDS Postgres │   │ ElastiCache   │
│  (Multi-AZ)   │   │ (Redis)       │
└───────────────┘   └───────────────┘
        │
┌───────────────┐
│      S3       │
│   (files)     │
└───────────────┘
```

### 10.3 Estimated Costs (MVP)

| Service | Monthly Cost |
|---------|-------------|
| VPS (4 vCPU, 16GB) | $50-100 |
| PostgreSQL managed | $50-100 |
| Redis managed | $30 |
| S3 storage | $10 |
| Claude API | $100-300 |
| OCR API | $20-50 |
| **Total** | **$260-580/mo** |

---

## Appendix A: Peptide Data Schema Example

```json
{
  "slug": "bpc-157",
  "name": "BPC-157",
  "full_name": "Body Protection Compound-157",
  "category": "Healing & Repair",
  "mechanism_of_action": "BPC-157 is a pentadecapeptide (15 amino acids) derived from human gastric juice. It promotes angiogenesis (new blood vessel formation), upregulates growth hormone receptors, and modulates nitric oxide synthesis. It has been shown to accelerate healing of tendons, ligaments, muscles, and the GI tract.",
  "benefits": [
    "Accelerates tendon and ligament healing",
    "Repairs muscle injuries",
    "Heals gut lining (leaky gut, IBD)",
    "Reduces inflammation",
    "Protects organs (liver, brain)",
    "May counteract NSAID damage"
  ],
  "research_summary": "Extensive animal studies show remarkable healing properties. Limited human data but widespread anecdotal use with positive reports. Most studied peptide for healing applications.",
  "dosing_protocols": [
    {
      "name": "Standard",
      "dose_mcg": 250,
      "frequency": "2x daily",
      "timing": "AM and PM",
      "duration_weeks": 4,
      "notes": "Inject near injury site if possible, or SubQ abdomen for systemic"
    },
    {
      "name": "Aggressive",
      "dose_mcg": 500,
      "frequency": "2x daily",
      "timing": "AM and PM",
      "duration_weeks": 4,
      "notes": "For acute injuries or stubborn healing"
    }
  ],
  "reconstitution_guide": "Add 2mL BAC water to 5mg vial = 2500mcg/mL. 0.1mL = 250mcg.",
  "injection_type": ["SubQ", "IM"],
  "typical_cycle_length_weeks": 4,
  "on_off_pattern": "4 weeks on, 2 weeks off if continuing",
  "synergistic_peptides": ["tb-500", "ghk-cu"],
  "avoid_with": [],
  "side_effects": [
    "Generally very well tolerated",
    "Rare: mild nausea, dizziness",
    "Injection site reactions (minor)"
  ],
  "contraindications": [
    "Active cancer (theoretical concern due to angiogenesis)",
    "Pregnancy/breastfeeding (no data)"
  ],
  "affects_markers": [
    {"marker": "CRP", "effect": "May decrease"},
    {"marker": "GH", "effect": "May increase receptor sensitivity"}
  ],
  "legal_status": {
    "US": "Research chemical, not FDA approved",
    "EU": "Gray area, varies by country",
    "AU": "Prescription only"
  }
}
```

---

## Appendix B: HD Health Correlations (Partial)

```python
GATE_HEALTH_MAP = {
    # Head Center
    64: {"system": "Pineal/Mental", "focus": "Mental pressure, anxiety", "support": ["Magnesium", "L-Theanine"]},
    61: {"system": "Pineal/Mental", "focus": "Inner truth, mental clarity", "support": ["Lion's Mane", "Meditation"]},
    63: {"system": "Pineal/Mental", "focus": "Doubt, questioning", "support": ["Grounding practices"]},
    
    # Ajna Center
    47: {"system": "Pituitary/Mental", "focus": "Realization, mental processing", "support": ["Omega-3", "Sleep"]},
    24: {"system": "Pituitary/Mental", "focus": "Rationalization", "support": ["Stress management"]},
    4: {"system": "Pituitary/Mental", "focus": "Formulation, answers", "support": ["Journaling"]},
    17: {"system": "Pituitary/Mental", "focus": "Opinions, mental organization", "support": ["Structure"]},
    43: {"system": "Pituitary/Mental", "focus": "Insight, breakthrough", "support": ["Semax", "Nootropics"]},
    11: {"system": "Pituitary/Mental", "focus": "Ideas, peace", "support": ["Meditation"]},
    
    # Throat Center
    62: {"system": "Thyroid/Parathyroid", "focus": "Details, expression", "support": ["Selenium"]},
    23: {"system": "Thyroid/Parathyroid", "focus": "Assimilation, explanation", "support": ["Iodine"]},
    56: {"system": "Thyroid/Parathyroid", "focus": "Stimulation, storytelling", "support": ["B vitamins"]},
    35: {"system": "Thyroid/Parathyroid", "focus": "Change, experience", "support": ["Adaptability"]},
    12: {"system": "Thyroid/Parathyroid", "focus": "Caution, articulation", "support": ["Speaking practices"]},
    45: {"system": "Thyroid/Parathyroid", "focus": "Gathering, leadership", "support": ["Voice work"]},
    33: {"system": "Thyroid/Parathyroid", "focus": "Privacy, retreat", "support": ["Alone time"]},
    8: {"system": "Thyroid/Parathyroid", "focus": "Contribution, creativity", "support": ["Creative expression"]},
    31: {"system": "Thyroid/Parathyroid", "focus": "Influence, democracy", "support": ["Leadership"]},
    20: {"system": "Thyroid", "focus": "NOW, presence, metamorphosis", "support": ["Thyroid support", "Selenium", "Zinc"]},
    16: {"system": "Thyroid", "focus": "Skills, enthusiasm", "support": ["Practice"]},
    
    # G Center (Identity)
    1: {"system": "Liver/Blood", "focus": "Self-expression, creativity", "support": ["Liver support"]},
    13: {"system": "Liver/Blood", "focus": "Listening, secrets", "support": ["Boundaries"]},
    25: {"system": "Liver/Blood", "focus": "Universal love, innocence", "support": ["Heart opening"]},
    46: {"system": "Liver/Blood", "focus": "Body, sensuality", "support": ["Embodiment practices"]},
    2: {"system": "Liver/Blood", "focus": "Direction, receptivity", "support": ["Patience"]},
    15: {"system": "Liver/Blood", "focus": "Extremes, rhythm", "support": ["Routine flexibility"]},
    10: {"system": "Liver/Blood", "focus": "Self-love, behavior", "support": ["Self-care"]},
    7: {"system": "Liver/Blood", "focus": "Role of self, leadership", "support": ["Direction"]},
    
    # Heart/Ego Center
    21: {"system": "Heart/Stomach/Gallbladder", "focus": "Control, willpower", "support": ["Digestive enzymes"]},
    26: {"system": "Heart/Thymus", "focus": "Ego, trickster", "support": ["Heart coherence"]},
    51: {"system": "Heart/Gallbladder", "focus": "Shock, initiation", "support": ["Stress resilience"]},
    40: {"system": "Heart/Stomach", "focus": "Aloneness, delivery", "support": ["Alone time", "Stomach support"]},
    
    # Spleen Center
    48: {"system": "Spleen/Immune", "focus": "Depth, talent", "support": ["Immune support"]},
    57: {"system": "Spleen/Immune", "focus": "Intuition, clarity", "support": ["LL-37", "Thymosin Alpha-1", "Zinc"]},
    44: {"system": "Spleen/Immune", "focus": "Alertness, pattern recognition", "support": ["Awareness practices"]},
    50: {"system": "Spleen/Immune", "focus": "Values, responsibility", "support": ["Boundaries"]},
    32: {"system": "Spleen/Immune", "focus": "Continuity, instinct", "support": ["Grounding"]},
    28: {"system": "Spleen/Kidneys", "focus": "Game player, risk", "support": ["Kidney support", "Adrenal support"]},
    18: {"system": "Spleen/Immune", "focus": "Correction, judgment", "support": ["Discernment"]},
    
    # Sacral Center
    5: {"system": "Sacral/Reproductive", "focus": "Rhythm, habits", "support": ["Routine", "Hormonal balance"]},
    14: {"system": "Sacral/Reproductive", "focus": "Power skills, wealth", "support": ["Energy management"]},
    29: {"system": "Sacral/Reproductive", "focus": "Perseverance, commitment", "support": ["Pacing"]},
    59: {"system": "Sacral/Reproductive", "focus": "Sexuality, intimacy", "support": ["Sexual health", "PT-141"]},
    9: {"system": "Sacral/Reproductive", "focus": "Focus, determination", "support": ["Concentration"]},
    3: {"system": "Sacral/Reproductive", "focus": "Ordering, mutation", "support": ["Patience"]},
    42: {"system": "Sacral/Reproductive", "focus": "Growth, finishing", "support": ["Completion practices"]},
    27: {"system": "Sacral/Immune", "focus": "Caring, nourishment", "support": ["Nurturing"]},
    34: {"system": "Sacral/Reproductive", "focus": "Power, individuation", "support": ["Energy outlets"]},
    
    # Solar Plexus
    36: {"system": "Solar Plexus/Pancreas/Kidneys", "focus": "Crisis, experience", "support": ["Emotional awareness"]},
    22: {"system": "Solar Plexus/Kidneys", "focus": "Grace, openness", "support": ["Emotional expression"]},
    37: {"system": "Solar Plexus/Pancreas", "focus": "Friendship, community", "support": ["Blood sugar balance"]},
    6: {"system": "Solar Plexus/Immune", "focus": "Conflict, intimacy", "support": ["Emotional boundaries"]},
    49: {"system": "Solar Plexus/Adrenals", "focus": "Revolution, principles", "support": ["Adrenal support", "Ashwagandha"]},
    55: {"system": "Solar Plexus/Pancreas", "focus": "Spirit, moods", "support": ["Blood sugar", "Emotional awareness"]},
    30: {"system": "Solar Plexus/Kidneys", "focus": "Feelings, desire", "support": ["Kidney/adrenal support"]},
    
    # Root Center
    53: {"system": "Adrenals/Stress", "focus": "Beginnings, pressure", "support": ["Adrenal support"]},
    60: {"system": "Adrenals/Stress", "focus": "Limitation, acceptance", "support": ["Stress management"]},
    52: {"system": "Adrenals/Stress", "focus": "Stillness, concentration", "support": ["Meditation", "Rest"]},
    19: {"system": "Adrenals/Stress", "focus": "Sensitivity, needs", "support": ["Boundaries"]},
    39: {"system": "Adrenals/Stress", "focus": "Provocation, spirit", "support": ["Energy outlets"]},
    41: {"system": "Adrenals/Stress", "focus": "Contraction, fantasy", "support": ["Dreaming", "Vision"]},
    58: {"system": "Adrenals/Stress", "focus": "Joy, vitality", "support": ["Joyful movement"]},
    38: {"system": "Adrenals/Stress", "focus": "Fighter, opposition", "support": ["Purposeful challenge"]},
    54: {"system": "Adrenals/Stress", "focus": "Ambition, drive", "support": ["Pacing", "Rest cycles"]},
}
```

---

## Appendix C: Lab Marker Reference

Full reference data for 50+ biomarkers with:
- Standard reference ranges by lab
- Optimal ranges
- What it measures
- Causes of high/low
- Related symptoms
- Actions to improve
- Related peptides/supplements

*(Full data to be compiled in separate document)*

---

## Next Steps

1. **Review this spec** — confirm scope, adjust priorities
2. **Find development team** — React Native + Python backend
3. **Create detailed task breakdown** for Phase 1
4. **Design UI/UX mockups**
5. **Set up development infrastructure**
6. **Begin Phase 1 build**

---

*Document generated by Hermes Agent — May 3, 2026*
