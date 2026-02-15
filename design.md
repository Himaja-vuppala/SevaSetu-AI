# SevaSetu AI - System Design Document

## 1. System Overview

### 1.1 Architecture Philosophy

SevaSetu AI is an **offline-first, edge-AI system** designed to democratize access to healthcare and government services for rural Indian citizens. The system operates entirely on-device, eliminating dependency on internet connectivity while providing intelligent, context-aware assistance through natural language interaction.

**Core Design Principles:**

1. **Offline-First**: All AI inference, knowledge retrieval, and response generation occur on-device
2. **Voice-First**: Primary interaction mode is natural speech in local languages
3. **Privacy-Preserving**: All personal data processing happens locally without cloud transmission
4. **Resource-Efficient**: Optimized for mid-range Android devices (4GB RAM, 2GB storage)
5. **Structured Output**: Generates actionable, formatted outputs for real-world service access
6. **Inclusion-Focused**: Designed for low-literacy users with culturally appropriate interfaces

**CRITICAL CONSTRAINT:**

> **All AI inference and personal data processing occur exclusively on-device.**
> 
> **Cloud infrastructure must never perform speech recognition, reasoning, embedding generation, retrieval, or response generation.**
> 
> Cloud is used ONLY for distribution, packaging, monitoring, and update management.

### 1.2 Real-World Usage Scenario

**Scenario**: Lakshmi, a 45-year-old farmer's wife in rural Tamil Nadu, visits her local Common Service Center (CSC) with her 8-year-old son who has been sick for three days.

**Interaction Flow:**

1. **Language Selection**: Lakshmi taps the Tamil language icon on the tablet at the CSC
2. **Voice Input**: She speaks naturally: "என் மகனுக்கு மூன்று நாட்களாக காய்ச்சல். இருமல் கூட இருக்கு" 
   (My son has fever for three days. Also has cough)
3. **On-Device Processing**: 
   - System transcribes speech locally (no internet needed)
   - Understands health consultation intent
   - Extracts symptoms: fever (3 days), cough
   - Retrieves relevant health information from local knowledge base
4. **Clarification**: System asks in Tamil: "காய்ச்சல் எவ்வளவு அதிகம்?" (How high is the fever?)
5. **Context Building**: Lakshmi responds, system maintains conversation context across turns
6. **Structured Output Generation**: System creates a symptom summary in both Tamil and English
7. **Delivery**: CSC operator prints the summary, Lakshmi takes it to the Primary Health Center (PHC)

**Result**: 
- Doctor receives structured information (chief complaint, symptom duration, severity)
- Enables faster, more accurate diagnosis
- Lakshmi saved 30+ minutes explaining symptoms through language barriers
- No internet connection was required at any point

**Privacy Guarantee**: All voice data, transcriptions, and personal information remained on the device and were deleted after the session.

---

## 2. Offline AI Pipeline

### 2.1 End-to-End Processing Architecture

**Complete Pipeline:**

```
User Voice Input
      |
      v
[Voice Activity Detection (VAD)]
  - Silero VAD model (~5MB)
  - Detects speech vs silence/noise
  - <10ms latency
      |
      v
[On-Device Speech-to-Text]
  - IndicWav2Vec (quantized INT8, 300MB)
  - Supports Hindi, Tamil, Telugu
  - Optimized for Indian accents
  - 1-2s latency for 10s audio
  - Output: Transcribed text
      |
      v
[Context Manager]
  - Maintains last 5 conversational turns
  - Tracks entities across conversation
  - Builds context window for SLM
      |
      v
[Quantized Small Language Model]
  - Llama-3.2 3B (Q4_K_M, 1.8GB)
  - Intent classification
  - Entity extraction
  - Query formulation
  - <500ms inference time
      |
      v
[Local Embedding Generation]
  - all-MiniLM-L6-v2 (quantized INT8, 23MB)
  - Converts query to 384-dim vector
  - <100ms latency
      |
      v
[Local Vector Database Retrieval]
  - FAISS HNSW index
  - Semantic similarity search
  - Retrieves top-3 relevant documents
  - <500ms search time
      |
      v
[Knowledge Base]
  - Health: symptoms, conditions, first-aid
  - Schemes: eligibility, benefits, process
  - Documents: requirements, sources
  - Services: locations, timings, procedures
      |
      v
[Structured Response Generation]
  - SLM generates response with retrieved context
  - Follows predefined templates
  - Includes source attribution
  - <1.5s generation time
      |
      v
[Text-to-Speech Output]
  - Indic TTS (ONNX, 150MB)
  - Natural prosody in user's language
  - <800ms for typical response
      |
      v
[Printable Service Summary]
  - PDF generation
  - Image export for WhatsApp/SMS
  - Structured format for service providers
```

**Total End-to-End Latency**: <5 seconds (speech input to speech output)


### 2.2 Conversation Memory Handling

**Context Window Management:**

- **Sliding Window**: Maintains last 5 conversational turns (user + assistant)
- **Entity Tracking**: Extracts and tracks entities across turns
  - Symptoms, dates, family members, income levels, document types
- **Coreference Resolution**: SLM resolves pronouns and implicit references
- **Context Limit**: 2048 tokens maximum (fits in SLM context window)

**Example Context State:**
```json
{
  "session_id": "uuid-12345",
  "language": "tamil",
  "intent_history": ["health_consultation"],
  "entities": {
    "symptoms": ["fever", "cough"],
    "duration": "3 days",
    "patient": "son",
    "age": 8,
    "severity": "moderate"
  },
  "turn_history": [
    {"role": "user", "text": "என் மகனுக்கு காய்ச்சல்", "timestamp": "2024-02-14T10:30:00"},
    {"role": "assistant", "text": "எவ்வளவு நாட்களாக?", "timestamp": "2024-02-14T10:30:03"},
    {"role": "user", "text": "மூன்று நாட்கள், இருமல் கூட", "timestamp": "2024-02-14T10:30:10"}
  ]
}
```

**Memory Cleanup**: All conversation data deleted when user closes app or session ends.

### 2.3 Structured Output Generation

**Output Templates:**

1. **Health Symptom Summary**:
   - Chief complaint
   - Symptom list with duration and severity
   - Relevant medical history
   - Red flags requiring immediate attention
   - Recommendations

2. **Scheme Eligibility Report**:
   - Scheme name and description
   - Eligibility criteria
   - User's match status
   - Required documents
   - Application process steps

3. **Document Checklist**:
   - Service name
   - Complete document list
   - Where to obtain each document
   - Validity requirements

4. **Service Navigation Guide**:
   - Service provider details
   - Location and contact information
   - Operating hours
   - Expected process flow
   - Estimated time and cost

**Format Options**: Plain text, PDF, PNG image (for WhatsApp sharing)


### 2.4 Supported Languages

**Primary Languages (V1)**:
- Hindi (हिंदी)
- Tamil (தமிழ்)
- Telugu (తెలుగు)

**Language Handling**:
- User selects language at session start
- Can switch language mid-conversation
- Code-mixed speech support (e.g., Hindi-English)
- Translation via IndicTrans2 (quantized, 400MB per direction)

### 2.5 Knowledge Categories

**Health Knowledge**:
- Common symptoms and conditions (5000+ entries)
- First-aid guidance
- When to seek medical care
- Medication information (general guidance only)
- Maternal and child health
- Preventive care

**Government Schemes**:
- Healthcare schemes (Ayushman Bharat, state programs)
- Agricultural subsidies (PM-KISAN, crop insurance)
- Financial assistance (pension, widow support)
- Education schemes (scholarships, mid-day meals)
- Housing and sanitation programs

**Document Guidance**:
- Aadhar card
- Ration card
- Income certificate
- Caste certificate
- Bank account documents
- Land ownership papers

**Service Provider Information**:
- Primary Health Centers (PHCs)
- Community Health Centers (CHCs)
- Common Service Centers (CSCs)
- Gram Panchayat offices
- Block development offices
- District administration offices

**Knowledge Pack Size**: 
- Base (national): 200MB
- District-specific: 50MB
- Total: 250MB per district

### 2.6 Pipeline Diagram

```
+-----------------------------------------------------------------------+
|                         USER VOICE INPUT                              |
|                    "मेरे बेटे को बुखार है"                            |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 1: VOICE ACTIVITY DETECTION                                    |
| - Silero VAD (~5MB)                                                  |
| - Detects speech vs noise                                            |
| - Latency: <10ms                                                     |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 2: SPEECH-TO-TEXT (On-Device)                                 |
| - IndicWav2Vec ASR (INT8, 300MB)                                    |
| - Hindi/Tamil/Telugu support                                         |
| - Latency: 1-2s for 10s audio                                       |
| - Output: "मेरे बेटे को बुखार है"                                   |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 3: CONTEXT MANAGEMENT                                          |
| - Maintains last 5 turns                                             |
| - Tracks entities across conversation                                |
| - Builds context window (2048 tokens)                                |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 4: SLM REASONING (On-Device)                                   |
| - Llama-3.2 3B (Q4_K_M, 1.8GB)                                      |
| - Intent: "health_consultation"                                      |
| - Entities: {symptom: "fever", patient: "son"}                       |
| - Query: "fever in children causes"                                  |
| - Latency: <500ms                                                    |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 5: LOCAL EMBEDDING GENERATION                                  |
| - all-MiniLM-L6-v2 (INT8, 23MB)                                     |
| - Query -> 384-dim vector                                            |
| - Latency: <100ms                                                    |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 6: VECTOR DATABASE RETRIEVAL (Local RAG)                      |
| - FAISS HNSW index                                                   |
| - Semantic similarity search                                         |
| - Top-3 documents retrieved                                          |
| - Latency: <500ms                                                    |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 7: RESPONSE GENERATION (On-Device SLM)                        |
| - Llama-3.2 3B with retrieved context                               |
| - Structured template-based output                                   |
| - Source attribution included                                        |
| - Latency: <1.5s                                                     |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 8: TEXT-TO-SPEECH OUTPUT                                       |
| - Indic TTS (ONNX, 150MB)                                           |
| - Natural prosody in user's language                                 |
| - Latency: <800ms                                                    |
+--------------------------------+--------------------------------------+
                                 |
                                 v
+-----------------------------------------------------------------------+
| STAGE 9: STRUCTURED OUTPUT DELIVERY                                  |
| - Visual display on screen                                           |
| - PDF generation for printing                                        |
| - Image export for WhatsApp/SMS                                      |
+-----------------------------------------------------------------------+

[LOCK] ALL PROCESSING ON-DEVICE | [LOCK] NO CLOUD INFERENCE | [LOCK] OFFLINE
```

---

## 3. Edge Device Architecture

### 3.1 Runtime Environment

**Target Hardware:**
- **Device Type**: Android smartphone or tablet
- **Minimum Specifications**:
  - RAM: 4GB (6GB recommended)
  - Storage: 3GB free space (5GB recommended)
  - Processor: ARM64 quad-core 2.0GHz+
  - Android Version: 10+ (API level 29+)
  - Battery: 4000mAh minimum
  - Screen: 7-10 inch for tablets, 5.5+ inch for phones

**Compatible Devices** (₹12,000-20,000 range):
- Xiaomi Redmi Note series
- Samsung Galaxy M series
- Realme Narzo series
- Motorola Moto G series

**Software Stack:**

```
+----------------------------------------------------------+
|              User Interface Layer                         |
|  React Native App (Voice-First, Large Touch Targets)     |
+----------------------------------------------------------+
                        |
+----------------------------------------------------------+
|           Application Logic Layer                         |
|  [Conversation Manager] [Intent Classifier] [Output Gen] |
+----------------------------------------------------------+
                        |
+----------------------------------------------------------+
|            AI Inference Layer                             |
|  [ONNX Runtime] [llama.cpp] [FAISS Vector DB]           |
+----------------------------------------------------------+
                        |
+----------------------------------------------------------+
|              Storage Layer                                |
|  [Models 2GB] [Knowledge 250MB] [User Data SQLite]      |
+----------------------------------------------------------+
```

### 3.2 Model Execution Engines

**ONNX Runtime Mobile (v1.16+)**:
- **Purpose**: Execute ASR, embedding, and TTS models
- **Optimizations**:
  - INT8 quantization for activations
  - ARM NEON SIMD instructions
  - Memory-mapped model loading
  - Graph optimization (constant folding, operator fusion)
- **Models**: IndicWav2Vec (ASR), all-MiniLM-L6-v2 (embeddings), Indic TTS

**llama.cpp (via JNI Bridge)**:
- **Purpose**: Execute quantized SLM (Llama-3.2 3B)
- **Format**: GGUF (GPT-Generated Unified Format)
- **Quantization**: Q4_K_M (4-bit with K-quants, medium quality)
- **Features**:
  - KV-cache for multi-turn conversations (50% speedup)
  - Streaming token generation
  - CPU-optimized inference (ARM64)
  - Memory-efficient attention mechanisms

**Integration Example**:
```java
// Native bridge to llama.cpp
public class LlamaInference {
    static {
        System.loadLibrary("llama-android");
    }
    
    public native long loadModel(String modelPath, int nThreads);
    public native String generate(long context, String prompt, 
                                   int maxTokens, float temperature);
    public native void freeModel(long context);
}
```


### 3.3 Model Packaging Format

**Model Files:**

| Model | Format | Size | Quantization | Purpose |
|-------|--------|------|--------------|---------|
| Llama-3.2 3B | GGUF | 1.8GB | Q4_K_M | Intent, reasoning, generation |
| IndicWav2Vec (Hi) | ONNX | 300MB | INT8 | Speech-to-text Hindi |
| IndicWav2Vec (Ta) | ONNX | 300MB | INT8 | Speech-to-text Tamil |
| IndicWav2Vec (Te) | ONNX | 300MB | INT8 | Speech-to-text Telugu |
| IndicTrans2 (Hi-Ta) | ONNX | 400MB | INT8 | Translation |
| IndicTrans2 (Hi-Te) | ONNX | 400MB | INT8 | Translation |
| all-MiniLM-L6-v2 | ONNX | 23MB | INT8 | Embeddings |
| Indic TTS | ONNX | 150MB | INT8 | Text-to-speech |

**Total Model Storage**: ~2GB (only required models loaded per language)

**Model Metadata Format**:
```json
{
  "model_id": "llama-3.2-3b-q4",
  "version": "2024.02.15",
  "format": "gguf",
  "quantization": "Q4_K_M",
  "size_bytes": 1887436800,
  "checksum_sha256": "a3f5b8c9d2e1f4a7...",
  "compatible_versions": ["1.0.0", "1.1.0"],
  "min_ram_mb": 2048,
  "inference_engine": "llama.cpp",
  "license": "Llama-3.2-Community"
}
```

### 3.4 Knowledge Pack Structure

**Directory Layout**:
```
/sdcard/SevaSetu/
├── models/                          (AI Models - 2GB)
│   ├── llama-3.2-3b-q4.gguf        (1.8GB - SLM)
│   ├── indicwav2vec-hi.onnx        (300MB - ASR Hindi)
│   ├── indicwav2vec-ta.onnx        (300MB - ASR Tamil)
│   ├── indicwav2vec-te.onnx        (300MB - ASR Telugu)
│   ├── indictrans2-hi-ta.onnx      (400MB - Translation)
│   ├── indictrans2-hi-te.onnx      (400MB - Translation)
│   ├── embedding-minilm.onnx       (23MB - Embeddings)
│   ├── tts-indic.onnx              (150MB - TTS)
│   └── metadata.json
├── knowledge/                       (Knowledge Packs - 250MB)
│   ├── base/                        (National - 200MB)
│   │   ├── health_symptoms.json
│   │   ├── health_conditions.json
│   │   ├── schemes_national.json
│   │   ├── documents_common.json
│   │   ├── embeddings_base.faiss
│   │   └── metadata.json
│   ├── district/                    (District-specific - 50MB)
│   │   ├── TN_Coimbatore/
│   │   │   ├── services_local.json
│   │   │   ├── schemes_state.json
│   │   │   ├── embeddings_district.faiss
│   │   │   └── metadata.json
│   │   └── version.json
│   └── version.json
└── exports/                         (User outputs)
    └── symptom_summaries/
```

**Knowledge Document Format**:
```json
{
  "id": "health_fever_001",
  "category": "health",
  "subcategory": "symptoms",
  "title": "Fever - Common Causes and When to Seek Care",
  "content": "Fever (बुखार/காய்ச்சல்/జ్వరం) is body temperature above 100.4°F...",
  "keywords": ["fever", "बुखार", "காய்ச்சல்", "జ్వరం", "temperature"],
  "metadata": {
    "source": "WHO Guidelines",
    "last_updated": "2024-01-15",
    "language": "multilingual",
    "verified": true
  }
}
```


### 3.5 Versioning System

**Semantic Versioning**: YYYY.MM.DD format

**Version Compatibility**:
- **Backward Compatible**: New knowledge packs work with older app versions (within 6 months)
- **Forward Compatible**: Older knowledge packs work with newer app versions
- **Breaking Changes**: Major version increment (e.g., 2024.02 -> 2024.08)

**Version Metadata**:
```json
{
  "pack_id": "sevasetu_knowledge_v2024.02.15",
  "pack_type": "base",
  "version": "2024.02.15",
  "district_code": null,
  "size_mb": 200,
  "checksum_sha256": "a3f5b8c9d2e1...",
  "dependencies": [],
  "compatible_app_versions": ["1.0.0", "1.1.0", "1.2.0"],
  "metadata": {
    "created_at": "2024-02-15T10:00:00Z",
    "expires_at": "2025-02-15T10:00:00Z",
    "source": "National Health Authority + MyGov"
  }
}
```

**Rollback Support**: Previous version retained for 30 days after update

### 3.6 Local Storage Layout

**Internal Storage** (/data/data/com.sevasetu.ai/ - 50MB):
```
├── databases/
│   └── user_prefs.db          (User settings, session data)
├── cache/
│   └── temp_audio/            (Temporary voice recordings)
└── files/
    └── logs/                  (Diagnostic logs)
```

**External Storage** (/sdcard/SevaSetu/ - 2.5GB):
- Models: 2GB
- Knowledge packs: 250MB
- User exports: Variable (user-controlled)

**Storage Optimization**:
- Lazy loading: Load models only when needed
- Memory mapping: OS manages model memory
- Compression: Knowledge packs use gzip compression
- Cleanup: Automatic cache cleanup after 7 days

### 3.7 Update Compatibility

**Update Types**:

1. **Model Updates**:
   - Rare (quarterly)
   - Requires app restart
   - Backward compatible quantization formats

2. **Knowledge Pack Updates**:
   - Frequent (monthly)
   - Hot-swappable (no restart needed)
   - Differential updates (only changed documents)

3. **App Updates**:
   - Regular (monthly)
   - Standard Android update process
   - Maintains data compatibility

**Compatibility Matrix**:

| App Version | Knowledge Pack | Model Version |
|-------------|----------------|---------------|
| 1.0.x | 2024.01.x - 2024.06.x | 2024.Q1 |
| 1.1.x | 2024.02.x - 2024.07.x | 2024.Q1 |
| 1.2.x | 2024.03.x - 2024.08.x | 2024.Q2 |

---

## 4. Deployment & Update Architecture (Cloud-Assisted, Offline-First)

### 4.1 Architecture Principle

**CRITICAL DESIGN CONSTRAINT:**

> **All AI inference and personal data processing occur exclusively on-device.**
> 
> **Cloud infrastructure must never perform speech recognition, reasoning, embedding generation, or response generation.**
> 
> Cloud is used ONLY for distribution, packaging, monitoring, and update management.

### 4.2 Cloud Responsibilities (AWS-Based)

**What Cloud DOES**:
1. Store AI model binaries and knowledge packs
2. Deliver updates when connectivity exists
3. Manage secure device registration
4. Provide update version control
5. Distribute updates regionally via CDN
6. Aggregate anonymized impact metrics (no PII)

**What Cloud DOES NOT DO**:
- [X] NO AI inference or model execution
- [X] NO processing of user voice or text input
- [X] NO storage of conversation history or personal data
- [X] NO real-time query handling
- [X] NO access to device-generated outputs

### 4.3 AWS Architecture Design

**Services Used**:

1. **Amazon S3 (Object Storage)**:
   - Store model binaries (2GB per model)
   - Store knowledge packs (250MB per district)
   - Versioned buckets for rollback
   - Lifecycle policies (archive old versions to Glacier after 90 days)

2. **Amazon CloudFront (CDN)**:
   - Regional edge caching (Mumbai, Delhi, Chennai, Hyderabad)
   - Reduces download latency
   - 30-day TTL for models, 7-day TTL for knowledge packs

3. **AWS Lambda (Serverless Functions)**:
   - Update check handler
   - Download URL generator (presigned S3 URLs)
   - Device registration manager
   - Analytics collector (validates no PII)

4. **Amazon API Gateway (REST API)**:
   - `/api/updates/check` - Check for available updates
   - `/api/updates/download/{pack_id}` - Get presigned S3 URL
   - `/api/devices/register` - Register new device
   - `/api/analytics/submit` - Submit anonymized metrics
   - Rate limiting: 10 requests/minute per device

5. **Amazon DynamoDB (NoSQL Database)**:
   - Device registry (device_id, district, version)
   - Update metadata (versions, checksums, compatibility)
   - Aggregated analytics (NO conversation content, NO PII)

6. **Amazon QuickSight (Analytics Dashboard)**:
   - Deployment metrics (devices active, districts covered)
   - Usage metrics (interactions, languages, intents)
   - Impact metrics (time saved, services accessed)


### 4.4 Synchronization Behavior

**Device Operation**:

1. **Default State**: Device operates fully offline
   - All AI inference on-device
   - No network required for core functionality
   - User can complete all tasks without internet

2. **Periodic Connectivity Check**: Every 24 hours (non-blocking background task)
   - Does not interrupt user interactions
   - Runs during idle time

3. **When Internet Detected**:
   ```
   a. Check for updates
      GET /api/updates/check
      Payload: {device_id, current_version, district_code}
      
   b. IF updates available:
      - Notify user (non-intrusive banner)
      - User can defer or accept
      - Download in background
      - Verify checksum
      - Install when device idle
      
   c. Submit anonymized analytics
      POST /api/analytics/submit
      Payload: {aggregated metrics, NO PII}
   ```

4. **Return to Offline**: Updates installed, ready for next session

**Update Download Process**:

1. **Differential Updates**:
   - Only download changed documents
   - Binary diff for model updates (if applicable)
   - Typical update size: 10-50MB (vs 250MB full pack)

2. **Resumable Downloads**:
   - Support for interrupted downloads
   - Range requests to S3
   - Retry with exponential backoff (2s, 4s, 8s, 16s)

3. **Integrity Verification**:
   ```python
   def verify_update(downloaded_file, expected_checksum):
       actual_checksum = sha256(downloaded_file)
       if actual_checksum != expected_checksum:
           raise IntegrityError("Checksum mismatch")
       return True
   ```

4. **Atomic Installation**:
   - Download to temporary directory
   - Verify integrity
   - Atomic swap with current version
   - Keep previous version for 30 days (rollback capability)

**Analytics Submission** (Anonymized Only):
```python
# Device-side analytics aggregation
analytics_payload = {
    "device_id": hash(device_uuid),  # One-way hash
    "date": "2024-02-14",
    "district": "TN_Coimbatore",
    "metrics": {
        "sessions": 25,
        "intents": {
            "health_consultation": 15,
            "scheme_inquiry": 8,
            "document_guidance": 2
        },
        "languages": {
            "tamil": 20,
            "hindi": 5
        },
        "avg_latency_ms": 2800,
        "errors": 2
    }
}
# NO conversation content, NO PII, NO user data
```


### 4.5 Deployment Architecture Diagram

```
+---------------------------------------------------------------------+
|                    EDGE DEVICES (Offline-First)                     |
|                                                                     |
|  +--------------+  +--------------+  +--------------+              |
|  |   CSC #1     |  |   CSC #2     |  |   PHC #1     |              |
|  | Coimbatore   |  |  Warangal    |  |  Varanasi    |              |
|  |              |  |              |  |              |              |
|  | [LOCK] AI    |  | [LOCK] AI    |  | [LOCK] AI    |              |
|  |  Inference   |  |  Inference   |  |  Inference   |              |
|  |  On-Device   |  |  On-Device   |  |  On-Device   |              |
|  |              |  |              |  |              |              |
|  | Models: 2GB  |  | Models: 2GB  |  | Models: 2GB  |              |
|  | Knowledge:   |  | Knowledge:   |  | Knowledge:   |              |
|  |  250MB       |  |  250MB       |  |  250MB       |              |
|  +--------------+  +--------------+  +--------------+              |
|         |                  |                  |                     |
|         +------------------+------------------+                     |
|                            |                                        |
|                   (Optional Sync Layer)                             |
|                   When Internet Available                           |
|                            |                                        |
+----------------------------+----------------------------------------+
                             |
                             | HTTPS (TLS 1.3)
                             | Device Certificate Auth
                             |
                             v
+---------------------------------------------------------------------+
|              AWS DISTRIBUTION LAYER (India Region)                  |
|                                                                     |
|  +------------------------------------------------------------+    |
|  |         CloudFront CDN (Edge Locations)                    |    |
|  |  Mumbai | Delhi | Chennai | Hyderabad                      |    |
|  |  * Cache models and knowledge packs                        |    |
|  |  * Reduce latency for downloads                            |    |
|  +------------------------------------------------------------+    |
|                             |                                       |
|  +------------------------------------------------------------+    |
|  |              API Gateway (REST API)                        |    |
|  |  * /api/updates/check                                      |    |
|  |  * /api/updates/download/{pack_id}                         |    |
|  |  * /api/analytics/submit (anonymized only)                 |    |
|  +------------------------------------------------------------+    |
|         |              |              |              |              |
|         v              v              v              v              |
|  +----------+  +----------+  +----------+  +----------+            |
|  |  Lambda  |  |  Lambda  |  |  Lambda  |  |  Lambda  |            |
|  |  Update  |  | Download |  |  Device  |  |Analytics |            |
|  |  Check   |  | Handler  |  |   Mgmt   |  |Collector |            |
|  +----------+  +----------+  +----------+  +----------+            |
|         |              |              |              |              |
|         +--------------+--------------+--------------+              |
|                             |                                       |
|  +------------------------------------------------------------+    |
|  |              Amazon S3 (Object Storage)                    |    |
|  |  +--------------+  +--------------+  +--------------+      |    |
|  |  |   Models     |  |  Knowledge   |  |   Updates    |      |    |
|  |  |   Bucket     |  |    Packs     |  |   Metadata   |      |    |
|  |  +--------------+  +--------------+  +--------------+      |    |
|  +------------------------------------------------------------+    |
|                             |                                       |
|  +------------------------------------------------------------+    |
|  |         Amazon DynamoDB (NoSQL Database)                   |    |
|  |  * Device registry                                         |    |
|  |  * Update versions and metadata                            |    |
|  |  * Anonymized analytics (aggregated only)                  |    |
|  +------------------------------------------------------------+    |
|                                                                     |
|  +------------------------------------------------------------+    |
|  |         Amazon QuickSight (Analytics Dashboard)            |    |
|  |  * Deployment metrics (devices active, districts covered)  |    |
|  |  * Usage metrics (interactions, languages, intents)        |    |
|  |  * Impact metrics (time saved, services accessed)          |    |
|  +------------------------------------------------------------+    |
|                                                                     |
|  [X] NO AI INFERENCE                                               |
|  [X] NO USER DATA STORAGE                                          |
|  [X] NO CONVERSATION PROCESSING                                    |
|  [CHECK] ONLY DISTRIBUTION & MONITORING                            |
+---------------------------------------------------------------------+
```

---
## 5. Security & Privacy Architecture

### 5.1 Privacy-by-Design Principles

**Core Privacy Guarantees:**

1. **On-Device Processing**: All AI inference, data processing, and storage occur exclusively on the device
2. **No Cloud Transmission**: User voice, text, and personal information never leave the device
3. **Session-Based Storage**: Conversation data deleted when session ends
4. **Minimal Data Collection**: Only essential data for functionality is processed
5. **User Control**: Users can delete all data at any time

### 5.2 Data Flow Security

**Voice Data Handling:**
```
User Voice Input
      |
      v
[Temporary Buffer] (RAM only, <10s audio)
      |
      v
[On-Device ASR] (Transcription in RAM)
      |
      v
[Conversation Context] (Last 5 turns, RAM)
      |
      v
[Session End] -> All data deleted
```

**No Persistent Storage of:**
- Voice recordings
- Transcribed text (beyond current session)
- Personal health information
- Financial details
- Family information


### 5.3 Encryption & Storage Security

**Model and Knowledge Pack Security:**

1. **Integrity Verification**:
   - SHA-256 checksums for all model files
   - Digital signatures for knowledge packs
   - Verification before loading into memory

2. **Tamper Detection**:
   - Hash verification on app startup
   - Alert if model or knowledge pack modified
   - Refuse to load corrupted files

3. **Secure Storage**:
   - Models stored in app-private directory (Android sandbox)
   - Knowledge packs encrypted at rest (AES-256)
   - User exports encrypted if containing sensitive data

**Optional User Data Encryption:**

If users choose to save outputs (symptom summaries, checklists):
- AES-256 encryption with device-specific key
- Key stored in Android Keystore (hardware-backed)
- Automatic deletion after 30 days (configurable)


### 5.4 Network Security (Update Layer Only)

**When Device Connects to Cloud for Updates:**

1. **Transport Security**:
   - TLS 1.3 for all connections
   - Certificate pinning for API endpoints
   - Mutual TLS for device authentication

2. **Device Authentication**:
   - Device-specific certificate issued during registration
   - JWT tokens with short expiry (15 minutes)
   - Rate limiting per device (10 requests/minute)

3. **Update Integrity**:
   - Presigned S3 URLs with 1-hour expiry
   - SHA-256 verification before installation
   - Atomic updates (all-or-nothing)

4. **Analytics Privacy**:
   - One-way hash of device ID (no reverse lookup)
   - Aggregated metrics only (no individual events)
   - No PII in any transmitted data

**Example Analytics Payload** (only data sent to cloud):
```json
{
  "device_hash": "a3f5b8c9...",
  "date": "2024-02-14",
  "district": "TN_Coimbatore",
  "sessions": 25,
  "intents": {"health": 15, "scheme": 8, "document": 2},
  "languages": {"tamil": 20, "hindi": 5},
  "avg_latency_ms": 2800
}
```


### 5.5 Access Control

**Role-Based Access:**

1. **End Users**: Full access to voice interaction, no access to system settings
2. **Facilitators**: Can view usage statistics, cannot access user data
3. **Administrators**: Can update knowledge packs, view diagnostics, no access to conversations
4. **System**: Automated processes have minimal privileges (principle of least privilege)

**Physical Security:**

- Devices deployed in CSCs/PHCs under supervision
- Screen lock after 5 minutes of inactivity
- No USB debugging enabled in production builds
- Tamper-evident seals on device ports (optional)

### 5.6 Compliance

**Regulatory Compliance:**

1. **Digital Personal Data Protection Act (DPDPA) 2023**:
   - Explicit consent for any data processing
   - Right to deletion (user can clear all data)
   - Data minimization (only essential data processed)
   - Purpose limitation (data used only for stated purpose)

2. **IT Act 2000 (Amended 2008)**:
   - Reasonable security practices
   - Sensitive personal data protection
   - Breach notification procedures

3. **Accessibility Standards**:
   - WCAG 2.1 Level AA compliance for visual interface
   - Voice-first design for low-literacy users


---

## 6. Performance & Resource Optimization

### 6.1 Model Quantization Strategy

**Quantization Techniques:**

1. **SLM (Llama-3.2 3B)**:
   - **Format**: GGUF Q4_K_M (4-bit with K-quants)
   - **Size Reduction**: 12GB (FP16) → 1.8GB (Q4_K_M) = 85% reduction
   - **Accuracy Impact**: <3% perplexity increase
   - **Inference Speed**: 2-3x faster than FP16 on CPU

2. **ASR Models (IndicWav2Vec)**:
   - **Format**: ONNX INT8 quantization
   - **Size Reduction**: 1.2GB (FP32) → 300MB (INT8) = 75% reduction
   - **WER Impact**: <2% increase in Word Error Rate
   - **Latency**: 40% faster inference

3. **Embedding Model (all-MiniLM-L6-v2)**:
   - **Format**: ONNX INT8 quantization
   - **Size Reduction**: 90MB (FP32) → 23MB (INT8) = 74% reduction
   - **Similarity Score Impact**: <1% change in cosine similarity
   - **Latency**: 50% faster

4. **TTS Models (Indic TTS)**:
   - **Format**: ONNX INT8 quantization
   - **Size Reduction**: 600MB (FP32) → 150MB (INT8) = 75% reduction
   - **Audio Quality**: Minimal perceptual difference (MOS >4.0)


### 6.2 Memory Footprint Optimization

**Memory Budget (4GB Device):**

| Component | Memory Usage | Optimization |
|-----------|--------------|--------------|
| Android OS | 1.2GB | System reserved |
| App Runtime | 200MB | React Native + JVM |
| SLM (Llama-3.2) | 1.8GB | Memory-mapped, lazy load |
| ASR Model | 300MB | Load on demand, unload after use |
| Embedding Model | 23MB | Persistent in memory |
| TTS Model | 150MB | Load on demand, unload after use |
| Vector DB Index | 100MB | Memory-mapped FAISS |
| Conversation Context | 50MB | Sliding window (5 turns) |
| UI & Buffers | 100MB | Optimized React Native |
| **Total Peak** | **2.5GB** | **1.5GB margin for OS** |

**Memory Management Strategies:**

1. **Lazy Loading**: Load ASR and TTS models only when needed
2. **Memory Mapping**: Use mmap for models (OS manages paging)
3. **KV-Cache Reuse**: Reuse SLM key-value cache across turns (50% speedup)
4. **Garbage Collection**: Aggressive GC after each conversation turn
5. **Buffer Pooling**: Reuse audio buffers instead of allocating new ones


### 6.3 Latency Optimization

**Target Latency Breakdown:**

| Stage | Target | Optimization Techniques |
|-------|--------|------------------------|
| Voice Activity Detection | <10ms | Lightweight Silero VAD |
| Speech-to-Text | 1-2s | INT8 quantization, ONNX Runtime |
| Context Building | <50ms | In-memory operations |
| SLM Intent Classification | <500ms | Q4 quantization, KV-cache |
| Embedding Generation | <100ms | INT8 quantization, batch size 1 |
| Vector Search | <500ms | FAISS HNSW index, top-3 retrieval |
| Response Generation | <1.5s | Streaming tokens, early stopping |
| Text-to-Speech | <800ms | INT8 quantization, streaming audio |
| **Total End-to-End** | **<5s** | **Parallel processing where possible** |

**Optimization Techniques:**

1. **Parallel Processing**:
   - Embed query while SLM is generating
   - Start TTS as soon as first sentence is generated (streaming)

2. **Caching**:
   - Cache frequent queries and responses
   - Cache embeddings for common intents
   - Reuse SLM KV-cache for multi-turn conversations

3. **Early Stopping**:
   - Stop SLM generation when complete sentence detected
   - Stop vector search when confidence threshold reached

4. **Hardware Acceleration**:
   - ARM NEON SIMD for matrix operations
   - Android NNAPI for supported operations (optional)


### 6.4 Battery Optimization

**Power Consumption Targets:**

- **Active Conversation**: 800-1000 mAh/hour
- **Idle (Screen On)**: 200 mAh/hour
- **Background Sync**: <50 mAh per update check
- **Target Usage**: 4+ hours continuous use on 4000mAh battery

**Battery Optimization Strategies:**

1. **CPU Throttling**:
   - Use efficiency cores for non-critical tasks
   - Reduce CPU frequency during idle periods
   - Batch operations to minimize wake-ups

2. **Model Optimization**:
   - Quantized models require fewer CPU cycles
   - Memory-mapped models reduce I/O operations
   - Streaming inference reduces peak power draw

3. **Screen Management**:
   - Auto-dim screen during voice interaction
   - Use dark theme to reduce OLED power consumption
   - Screen timeout after 2 minutes of inactivity

4. **Network Optimization**:
   - Batch analytics uploads (once per day)
   - Use WiFi for updates when available (vs mobile data)
   - Exponential backoff for failed connections

5. **Background Task Management**:
   - Defer non-critical tasks to charging time
   - Use Android WorkManager for efficient scheduling
   - Disable background sync if battery <20%


### 6.5 Storage Optimization

**Storage Budget (2GB Available):**

| Component | Size | Compression |
|-----------|------|-------------|
| SLM (Llama-3.2 Q4) | 1.8GB | GGUF quantization |
| ASR Models (3 langs) | 900MB | INT8 quantization |
| TTS Models | 150MB | INT8 quantization |
| Embedding Model | 23MB | INT8 quantization |
| Translation Models | 800MB | INT8 quantization (optional) |
| Knowledge Base | 200MB | Gzip compression |
| District Pack | 50MB | Gzip compression |
| Vector Index | 100MB | FAISS compressed index |
| App Binary | 50MB | ProGuard + R8 optimization |
| **Total** | **~2GB** | **Fits in budget** |

**Storage Optimization Techniques:**

1. **Selective Language Loading**:
   - Only load ASR/TTS for selected languages
   - Download additional languages on demand

2. **Knowledge Pack Compression**:
   - Gzip compression (60-70% size reduction)
   - Decompress on-the-fly during retrieval

3. **Differential Updates**:
   - Only download changed documents
   - Binary diff for model updates (if applicable)

4. **Cache Management**:
   - Automatic cleanup of temporary files after 7 days
   - User-controlled cleanup of exported outputs


### 6.6 Benchmarking & Performance Monitoring

**Performance Metrics Tracked:**

1. **Latency Metrics**:
   - Per-stage latency (ASR, SLM, retrieval, TTS)
   - End-to-end response time
   - 95th percentile latency

2. **Resource Metrics**:
   - Peak memory usage
   - Average CPU utilization
   - Battery drain rate
   - Storage usage

3. **Quality Metrics**:
   - ASR Word Error Rate (WER)
   - Intent classification accuracy
   - Retrieval relevance (user feedback)
   - TTS naturalness (MOS score)

**Continuous Monitoring:**

- On-device performance logging (anonymized)
- Periodic benchmarking on reference devices
- A/B testing of optimization strategies
- User-reported performance issues

---

## 7. Failure Handling & Resilience

### 7.1 Graceful Degradation

**Failure Scenarios & Responses:**

1. **Low Storage (<500MB Free)**:
   - **Symptom**: Cannot load all models
   - **Response**: 
     - Load only essential models (SLM + one language)
     - Disable TTS (text-only output)
     - Prompt user to free space
     - Provide manual cleanup option

2. **Low Memory (<1GB Available)**:
   - **Symptom**: Risk of OOM crashes
   - **Response**:
     - Unload ASR/TTS models after each use
     - Reduce conversation context window (5 → 3 turns)
     - Disable caching
     - Restart app if memory pressure persists


3. **Corrupted Knowledge Pack**:
   - **Symptom**: Checksum mismatch or parse errors
   - **Response**:
     - Refuse to load corrupted pack
     - Fall back to base knowledge pack (if available)
     - Notify user of corruption
     - Provide re-download option (when online)
     - Log error for diagnostics

4. **Model Loading Failure**:
   - **Symptom**: Model file missing or corrupted
   - **Response**:
     - Display clear error message in user's language
     - Suggest reinstalling app
     - Provide diagnostic information for support
     - Do not crash app (graceful exit)

5. **ASR Failure (Unintelligible Speech)**:
   - **Symptom**: Low confidence transcription
   - **Response**:
     - Prompt user to repeat more clearly
     - Suggest quieter environment
     - Offer text input as fallback
     - Provide visual feedback on audio quality

6. **SLM Inference Timeout**:
   - **Symptom**: Generation takes >10 seconds
   - **Response**:
     - Stop generation
     - Return partial response if coherent
     - Apologize and ask user to rephrase
     - Log timeout for diagnostics


### 7.2 Error Recovery Strategies

**Automatic Recovery:**

1. **Retry with Backoff**:
   - Network requests: 3 retries with exponential backoff (2s, 4s, 8s)
   - Model loading: 2 retries with memory cleanup between attempts
   - Vector search: Retry with relaxed similarity threshold

2. **Fallback Mechanisms**:
   - ASR fails → Text input
   - TTS fails → Text-only output
   - SLM fails → Rule-based intent classification (limited)
   - Vector search fails → Keyword-based search

3. **State Recovery**:
   - Save conversation state every 3 turns
   - Restore state after app crash
   - Offer to resume interrupted conversation

**User-Initiated Recovery:**

1. **Manual Retry**: User can tap "Try Again" button
2. **Rephrase Prompt**: System suggests rephrasing query
3. **Reset Conversation**: User can start fresh conversation
4. **Report Issue**: User can report problem with one tap


### 7.3 Offline-First Guarantees

**Core Offline Capabilities (100% Functional):**

1. **Voice Interaction**: Full speech-to-text and text-to-speech
2. **Intent Understanding**: Complete SLM reasoning
3. **Knowledge Retrieval**: Full access to local knowledge base
4. **Response Generation**: Structured output generation
5. **Export**: PDF and image generation

**Optional Online Features (Gracefully Degrade):**

1. **Knowledge Updates**: Device continues with existing knowledge
2. **Analytics Submission**: Queued for next connection
3. **Model Updates**: Deferred until connectivity available
4. **Feedback Submission**: Stored locally, synced later

**Offline Behavior:**

- No "No Internet" error messages for core features
- Clear indication of last knowledge update date
- Offline badge in UI (non-intrusive)
- Automatic sync when connectivity detected

### 7.4 Data Integrity

**Integrity Checks:**

1. **Model Files**:
   - SHA-256 checksum verification on load
   - Refuse to load if checksum mismatch
   - Alert user to reinstall if corrupted

2. **Knowledge Packs**:
   - Digital signature verification
   - Version compatibility check
   - Atomic updates (all-or-nothing)

3. **User Exports**:
   - Verify PDF generation success
   - Check image encoding integrity
   - Retry generation if failed


### 7.5 Crash Recovery

**Crash Prevention:**

1. **Memory Management**:
   - Monitor memory usage continuously
   - Proactive cleanup before OOM
   - Graceful degradation under memory pressure

2. **Exception Handling**:
   - Catch all exceptions in critical paths
   - Log errors for diagnostics
   - Display user-friendly error messages

3. **Watchdog Timers**:
   - Timeout long-running operations
   - Kill stuck threads
   - Restart components if frozen

**Crash Recovery:**

1. **Automatic Restart**:
   - Detect crash on next launch
   - Offer to restore previous session
   - Send crash report (if user consents)

2. **State Preservation**:
   - Save conversation state every 3 turns
   - Persist user preferences
   - Restore UI state after crash

3. **Diagnostic Logging**:
   - Capture stack traces
   - Log system state before crash
   - Anonymize logs before submission


### 7.6 Update Failure Handling

**Update Scenarios:**

1. **Download Interrupted**:
   - **Response**: Resume from last byte (HTTP range requests)
   - **Retry**: 3 attempts with exponential backoff
   - **Fallback**: Continue with current version

2. **Checksum Mismatch**:
   - **Response**: Delete downloaded file
   - **Retry**: Re-download from scratch
   - **Alert**: Notify user of integrity issue

3. **Installation Failed**:
   - **Response**: Rollback to previous version
   - **Preserve**: Keep previous version for 30 days
   - **Log**: Capture installation error for diagnostics

4. **Incompatible Version**:
   - **Response**: Refuse to install
   - **Notify**: Prompt user to update app first
   - **Guidance**: Provide clear instructions

**Rollback Mechanism:**

- Previous version retained for 30 days
- One-tap rollback if new version has issues
- Automatic rollback if new version crashes repeatedly (3+ times)

---

## 8. Implementation Roadmap

### 8.1 Phase 1: MVP (Months 1-3)

**Scope:**
- Single language (Hindi)
- Basic health consultation intent
- 1000-document knowledge base
- Text-only output (no TTS)
- Single district deployment (pilot)

**Deliverables:**
- Working Android app with on-device SLM
- Basic ASR integration
- Local RAG with FAISS
- Structured symptom summary generation
- 100 user pilot test


### 8.2 Phase 2: Multi-Language & TTS (Months 4-6)

**Scope:**
- Add Tamil and Telugu support
- Integrate TTS for voice output
- Expand to 3 intents (health, schemes, documents)
- 5000-document knowledge base
- 10-district deployment

**Deliverables:**
- Multi-language ASR and TTS
- Intent classification for 3 categories
- PDF export functionality
- Cloud distribution infrastructure (AWS)
- 1000 user pilot across 10 districts

### 8.3 Phase 3: Full Feature Set (Months 7-9)

**Scope:**
- All 5 intent categories
- 10,000-document knowledge base
- District-specific knowledge packs
- Image export for WhatsApp sharing
- 50-district deployment

**Deliverables:**
- Complete intent classification
- District-specific knowledge management
- Update and sync infrastructure
- Analytics dashboard
- 10,000 user deployment

### 8.4 Phase 4: Scale & Optimize (Months 10-12)

**Scope:**
- Performance optimization
- Battery and memory improvements
- Additional languages (Kannada, Bengali)
- 100+ district deployment
- Sustainability planning

**Deliverables:**
- Optimized models and inference
- Extended language support
- Nationwide deployment readiness
- Long-term maintenance plan
- Impact evaluation report


---

## 9. Testing Strategy

### 9.1 Unit Testing

**Components to Test:**

1. **ASR Module**: Test transcription accuracy on diverse accents
2. **SLM Inference**: Test intent classification and entity extraction
3. **Vector Search**: Test retrieval relevance and latency
4. **Response Generation**: Test template filling and formatting
5. **TTS Module**: Test audio quality and naturalness

**Test Data:**
- 1000+ voice samples across languages, accents, ages, genders
- 500+ test queries with ground-truth intents and entities
- 100+ edge cases (ambiguous queries, out-of-scope, errors)

### 9.2 Integration Testing

**End-to-End Scenarios:**

1. **Health Consultation Flow**: Voice input → Intent → Retrieval → Summary generation
2. **Scheme Inquiry Flow**: Multi-turn conversation → Eligibility check → Document list
3. **Language Switching**: Mid-conversation language change
4. **Offline Operation**: All features with no network
5. **Update Flow**: Download → Verify → Install → Rollback

### 9.3 Performance Testing

**Benchmarks:**

1. **Latency**: Measure per-stage and end-to-end latency on target devices
2. **Memory**: Monitor peak memory usage under various scenarios
3. **Battery**: Measure power consumption during active use
4. **Storage**: Verify storage footprint within budget

**Load Testing:**
- Continuous operation for 4+ hours
- 100+ consecutive queries without restart
- Memory leak detection


### 9.4 User Acceptance Testing

**Field Testing:**

1. **Pilot Deployment**: 100 users in 1 district for 2 weeks
2. **Usability Testing**: Observe 50 users completing tasks
3. **Feedback Collection**: Surveys and interviews
4. **Iteration**: Fix issues and improve based on feedback

**Success Criteria:**
- 70% task completion rate
- >4/5 user satisfaction rating
- <5% crash rate
- <3s average response time

### 9.5 Security Testing

**Security Audits:**

1. **Penetration Testing**: Test API endpoints and device security
2. **Privacy Audit**: Verify no PII leakage
3. **Integrity Testing**: Attempt to tamper with models and knowledge packs
4. **Compliance Review**: Verify DPDPA and IT Act compliance

---

## 10. Success Metrics

### 10.1 Technical Metrics

1. **Accuracy**: Intent classification >85%, ASR WER <15%
2. **Latency**: End-to-end <3s for 90% of queries
3. **Reliability**: <1% crash rate, 99% uptime
4. **Resource Efficiency**: Peak memory <2.5GB, storage <2GB
5. **Offline Operation**: 100% core features functional offline

### 10.2 User Metrics

1. **Adoption**: 100,000+ interactions in first year
2. **Retention**: 60% return within 30 days
3. **Satisfaction**: >4/5 average rating
4. **Task Completion**: 70% successfully complete intended task
5. **Inclusion**: 60% women, 40% elderly, 80% regional languages


### 10.3 Impact Metrics

1. **Reach**: 100,000+ citizens served in first year
2. **Time Savings**: Average 2+ hours saved per service access
3. **Service Completion**: 70% successfully access intended service
4. **Awareness**: 50% increase in scheme awareness
5. **Equity**: Equal performance across languages and demographics

---

## 11. Risk Mitigation

### 11.1 Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Quantized models underperform | High | Extensive benchmarking, fallback to larger models if needed |
| Device compatibility issues | Medium | Test on 10+ device models, provide minimum spec requirements |
| Storage constraints | Medium | Selective language loading, compression, cloud fallback |
| Battery drain | Low | Aggressive optimization, power profiling, user guidance |
| Model hallucination | High | RAG grounding, confidence thresholds, human escalation |

### 11.2 Deployment Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Low user adoption | High | Community facilitators, training, simple UX |
| Knowledge staleness | Medium | Monthly updates, version indicators, user feedback |
| Device theft/damage | Low | Cloud backup of knowledge packs, easy reinstall |
| Connectivity for updates | Low | Manual update via SD card, long update cycles |
| Maintenance burden | Medium | Automated monitoring, remote diagnostics, community support |


### 11.3 Privacy & Security Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Data breach | High | On-device processing, no cloud storage of PII |
| Model tampering | Medium | Checksum verification, digital signatures |
| Unauthorized access | Low | Device lock, role-based access control |
| Analytics privacy leak | Medium | Anonymization, aggregation, no PII collection |
| Regulatory non-compliance | High | Legal review, DPDPA compliance, audit trail |

### 11.4 Social & Ethical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Bias in responses | High | Diverse training data, bias testing, feedback mechanism |
| Over-reliance on AI | Medium | Clear limitations, human escalation, disclaimers |
| Misinformation | High | Verified sources only, confidence indicators, fact-checking |
| Digital divide | Medium | Community facilitators, shared devices, training |
| Language exclusion | Medium | Phased language rollout, community input on priorities |

---

## 12. Differentiation & Innovation

### 12.1 Key Differentiators

**vs. Cloud-Based Chatbots (ChatGPT, Gemini):**
- **Offline Operation**: Works without internet, critical for rural India
- **Privacy**: All data processing on-device, no cloud transmission
- **Structured Outputs**: Generates actionable documents, not just text responses
- **Domain-Specific**: Focused on healthcare and government services, not general chat


**vs. Translation Apps (Google Translate):**
- **Intent Understanding**: Understands user goals, not just translates words
- **Knowledge Grounding**: Retrieves relevant information from local database
- **Conversational**: Multi-turn dialogue, not single-shot translation
- **Structured Outputs**: Generates formatted documents, not just translated text

**vs. Rule-Based IVR Systems:**
- **Natural Language**: Speak naturally, no menu navigation
- **Flexible**: Handles variations and novel phrasings
- **Contextual**: Remembers conversation history
- **Multilingual**: Seamless language switching

**vs. Existing Government Apps:**
- **Voice-First**: No typing or navigation required
- **Low Literacy**: Accessible to users with limited education
- **Offline**: No internet dependency
- **Unified**: Single interface for multiple services

### 12.2 Innovation Highlights

1. **On-Device SLM Reasoning**: First deployment of quantized 3B SLM for government service access in India
2. **Offline RAG**: Local vector database with semantic search on mobile devices
3. **Structured Output Generation**: AI-generated, formatted documents for real-world service access
4. **Multilingual Voice-First UX**: Natural speech interaction in regional languages
5. **Privacy-Preserving AI**: Complete on-device processing with no cloud dependency
6. **District-Specific Knowledge**: Localized information packs for regional relevance

---

## 13. Conclusion

SevaSetu AI represents a pragmatic, deployment-ready approach to democratizing access to healthcare and government services in rural India. By combining offline-first architecture, on-device AI inference, local knowledge retrieval, and voice-first interaction, the system addresses the core barriers of connectivity, language, and digital literacy that prevent millions of citizens from accessing essential services.

**Key Strengths:**

1. **Offline-First**: Operates completely without internet, reaching the most underserved populations
2. **Privacy-Preserving**: All AI processing on-device, building trust with users
3. **Voice-First**: Natural speech interaction accessible to low-literacy users
4. **Structured Outputs**: Generates actionable documents that bridge communication gaps
5. **Resource-Efficient**: Runs on mid-range Android devices within 4GB RAM and 2GB storage
6. **Scalable**: District-specific knowledge packs enable nationwide deployment
7. **Sustainable**: Minimal operational costs due to offline operation

**Technical Feasibility:**

- Quantized SLMs (Llama-3.2 3B Q4) enable on-device reasoning
- Local RAG with FAISS provides knowledge grounding
- Proven technology stack (llama.cpp, ONNX Runtime, React Native)
- <5s end-to-end latency on target hardware
- Extensive optimization for memory, battery, and storage

**Social Impact:**

- Empowers 50M+ rural citizens to access services independently
- Reduces dependency on exploitative intermediaries
- Levels the playing field regardless of language, literacy, or location
- Enables healthcare workers and government officials to serve more citizens effectively

**Deployment Readiness:**

- Phased rollout plan (10 → 50 → 100+ districts)
- Partnership model with CSCs, PHCs, and NGOs
- Minimal infrastructure requirements (mid-range tablets)
- Sustainable maintenance through community support and government partnership

SevaSetu AI demonstrates that meaningful AI innovation for inclusion doesn't require cutting-edge hardware or constant connectivity—it requires thoughtful design, appropriate technology choices, and a deep understanding of user needs. This system is ready for real-world deployment and poised to make a measurable impact on service access equity in India.

---

## Document Metadata

- **Version**: 1.0
- **Date**: 2026-02-15
- **Status**: Final Design for Hackathon Submission
- **Track**: Inclusion & Public Impact
- **Hackathon**: AI for Bharat
- **Project**: SevaSetu AI - Offline Edge-AI Assistant for Rural India

---

**End of Design Document**
