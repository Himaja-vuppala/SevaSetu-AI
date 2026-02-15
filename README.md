SevaSetu AI

Offline Voice-First AI Assistant for Healthcare & Government Service Access

SevaSetu AI is an offline, edge-AI assistant designed to help rural and semi-urban citizens access healthcare and government welfare services using natural speech in local languages — without internet connectivity.

The system runs entirely on-device and converts voice input into structured, actionable guidance that users can take directly to doctors or government offices.

Problem

Millions of citizens in rural India face barriers in accessing essential services due to:

• Language barriers
• Low digital literacy
• Poor or no internet connectivity
• Lack of awareness of schemes and procedures
• Difficulty explaining medical symptoms or service needs

These challenges lead to delayed care, missed benefits, and dependence on intermediaries.

Solution

SevaSetu AI provides an offline voice-based assistant that:

• Understands natural speech in local languages
• Identifies user intent using on-device AI reasoning
• Retrieves relevant information from local knowledge packs
• Generates structured outputs for real-world use

The system removes connectivity, language, and literacy barriers through edge AI.

Key Features

• Voice interaction in Hindi, Tamil, and Telugu
• Fully offline AI inference on device
• Local knowledge retrieval using vector search
• Structured outputs:

Symptom summary for doctors

Scheme eligibility checklist

Document preparation guide
• Privacy-preserving (no data leaves device)
• Export results as text, PDF, or shareable image

How It Works

User speaks → Speech converted to text → AI understands intent → Local knowledge retrieved → Structured response generated → Output displayed and read aloud

System Architecture
Edge Device (Primary Intelligence)

• On-device Speech Recognition
• Quantized Small Language Model
• Local Vector Knowledge Database
• Text-to-Speech Engine
• Voice-first Mobile UI

Cloud Layer (Support Only — No AI Inference)

• Model and knowledge pack distribution
• Device update management
• Aggregated anonymized analytics

All personal data processing happens exclusively on-device.

Technology Stack

AI Models
• Quantized Small Language Model (Phi-3 / Llama class)
• On-device ASR model
• Local RAG pipeline

Mobile Runtime
• Android platform
• ONNX Runtime / MediaPipe
• SQLite + vector search

Cloud Support
• AWS for distribution and update management

Security
• On-device encryption
• Signed knowledge packs

Public Impact

SevaSetu AI enables inclusive digital access for:

• Rural citizens
• Women and elderly users
• Migrant workers
• Small farmers

Expected outcomes:

• Faster healthcare access
• Increased welfare scheme utilization
• Reduced dependency on intermediaries
• Improved service delivery efficiency

Repository Contents

requirements.md → Functional and non-functional system requirements
design.md → Technical architecture and deployment design
architecture.png → System architecture diagram

Deployment Vision

Deployable at:

• Common Service Centers
• Primary Health Centers
• Gram Panchayat Offices
• Community outreach programs

Works fully offline after installation.

Team

Team Name: SevaSetu AI
Hackathon: AI for Bharat
