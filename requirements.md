# SevaSetu AI - Requirements Document

## 1. Problem Statement

Rural and semi-urban citizens in India face significant barriers in accessing essential healthcare and government services due to:

- **Language Barriers**: Most government portals and healthcare systems operate in English or Hindi, excluding speakers of regional languages (Tamil, Telugu, Kannada, Bengali, etc.)
- **Low Digital Literacy**: Complex forms, navigation flows, and technical terminology create insurmountable obstacles for citizens with limited education
- **Poor Connectivity**: Intermittent or absent internet connectivity in rural areas prevents access to online services and AI-powered assistants that require cloud connectivity
- **Information Asymmetry**: Citizens lack awareness of available schemes, eligibility criteria, required documents, and procedural steps
- **Communication Gaps**: Inability to articulate medical symptoms or service requirements in structured formats expected by service providers

These barriers result in:
- Delayed or denied access to critical healthcare
- Missed opportunities for government welfare schemes
- Increased dependency on intermediaries (often exploitative)
- Perpetuation of socioeconomic inequalities

**SevaSetu AI addresses this gap by providing an offline, voice-first AI assistant that understands natural speech in local languages, reasons about user intent, retrieves relevant information from local knowledge bases, and generates structured, actionable outputs that bridge the communication gap between citizens and service providers.**

---

## 2. Target Users & Stakeholders

### 2.1 Primary Users
- **Rural Citizens** (age 25-65): Far
mers, daily wage workers, homemakers with limited digital literacy and primary language proficiency in regional languages
- **Semi-Urban Citizens**: Small town residents seeking government services but facing connectivity issues
- **Low-Literacy Users**: Citizens with basic or no formal education who rely on voice-based interaction

### 2.2 Secondary Users
- **Healthcare Workers**: ASHA workers, ANMs, PHC doctors who receive structured symptom summaries
- **Government Service Providers**: Gram Panchayat officials, block development officers, service center operators
- **Community Facilitators**: NGO workers, self-help group leaders who assist citizens in service access

### 2.3 Deployment Environments
- **Rural Health Centers**: Primary Health Centers (PHCs), Community Health Centers (CHCs)
- **Common Service Centers (CSCs)**: Jan Seva Kendras in villages and small towns
- **Gram Panchayat Offices**: Village administrative centers
- **Mobile Outreach**: Field workers carrying tablets/smartphones to remote locations
- **Home Usage**: Citizens with basic Android smartphones (optional future scope)

---

## 3. User Stories (EARS Notation)

### 3.1 Voice Interaction
**US-01**: WHEN a user speaks in their local language (Hindi/Tamil/Telugu), THE system SHALL convert speech to text with >85% accuracy for common healthcare and government service vocabulary.

**US-02**: WHEN the system generates a response, THE system SHALL convert text to speech in the user's selected language with natural prosody and clarity.

**US-03**: WHEN background noise is detected during voice input, THE system SHALL prompt the user to repeat their input or provide visual feedback about audio quality.

### 3.2 Language Handling
**US-04**: WHEN a user initiates interaction, THE system SHALL allow language selection from Hindi, Tamil, and Telugu through voice command or visual interface.

**US-05**: WHEN a user switches language mid-conversation, THE system SHALL maintain conversation context and continue in the newly selected language.

**US-06**: WHEN processing user input, THE system SHALL handle code-mixed speech (e.g., Hindi-English) commonly used in rural India.

### 3.3 Intent Understanding
**US-07**: WHEN a user describes symptoms in natural conversational language, THE system SHALL identify the primary health concern and extract relevant symptom details using the on-device SLM.

**US-08**: WHEN a user asks about a government scheme using informal or incomplete information, THE system SHALL reason about the likely intent and retrieve relevant scheme information from the local knowledge base.

**US-09**: WHEN user intent is ambiguous, THE system SHALL ask clarifying questions in the user's language to narrow down the service requirement.

**US-10**: WHEN a user provides information across multiple conversational turns, THE system SHALL maintain context and build a coherent understanding of the complete request.

### 3.4 Offline Operation
**US-11**: WHEN the device has no internet connectivity, THE system SHALL perform all core functions (speech recognition, intent understanding, knowledge retrieval, response generation) using only on-device resources.

**US-12**: WHEN internet connectivity becomes available, THE system SHALL optionally sync usage analytics (anonymized) and check for knowledge pack updates without interrupting user interaction.

**US-13**: WHEN the system is deployed in a new district, THE system SHALL function with the pre-loaded district-specific knowledge pack without requiring internet access.

### 3.5 Structured Output Generation
**US-14**: WHEN a user completes a health consultation, THE system SHALL generate a structured symptom summary in English/Hindi suitable for presentation to a doctor, including: chief complaint, symptom duration, severity indicators, and relevant medical history mentioned.

**US-15**: WHEN a user inquires about a government scheme, THE system SHALL generate a structured eligibility checklist, required documents list, and step-by-step application process in the user's language.

**US-16**: WHEN a user needs to visit a government office, THE system SHALL generate a visit preparation guide including: office location, operating hours, required documents, expected process flow, and estimated time.

### 3.6 Knowledge Retrieval
**US-17**: WHEN the system needs to answer a query, THE system SHALL retrieve relevant information from the local vector database using semantic similarity search with <2 second latency.

**US-18**: WHEN retrieved information is outdated (based on metadata timestamp), THE system SHALL indicate the information age to the user and suggest checking for updates when online.

### 3.7 Privacy & Trust
**US-19**: WHEN a user shares personal health or financial information, THE system SHALL process all data on-device without transmitting to external servers, and SHALL inform the user of this privacy protection.

**US-20**: WHEN the system cannot confidently answer a query, THE system SHALL explicitly state its uncertainty and suggest consulting a human expert rather than providing potentially incorrect information.

---

## 4. Functional Requirements

### 4.1 Voice Input Processing
**FR-01**: The system SHALL support on-device speech-to-text conversion for Hindi, Tamil, and Telugu using a quantized ASR model optimized for Indian accents.

**FR-02**: The system SHALL support continuous listening mode with voice activity detection to enable natural conversation flow.

**FR-03**: The system SHALL provide visual feedback (waveform, transcription preview) during voice input to build user confidence.

**FR-04**: The system SHALL support manual text input as a fallback for users in noisy environments or with speech difficulties.

### 4.2 Intent Understanding Using SLM
**FR-05**: The system SHALL use a quantized Small Language Model (Phi-3-mini 3.8B or Llama-3.2 3B) running on-device to understand user intent from natural language input.

**FR-06**: The system SHALL classify user intent into predefined categories: health_consultation, scheme_inquiry, document_guidance, service_navigation, general_query.

**FR-07**: The system SHALL extract structured entities from user input (symptoms, dates, family size, income level, document types) using the SLM's reasoning capabilities.

**FR-08**: The system SHALL maintain conversation context across multiple turns using a sliding window approach (last 5 turns) to enable follow-up questions and clarifications.

**FR-09**: The system SHALL handle multi-intent queries by decomposing them into sequential sub-tasks.

### 4.3 Local Knowledge Retrieval (RAG)
**FR-10**: The system SHALL maintain a local vector database containing embeddings of:
- Common health conditions, symptoms, and first-aid guidance
- Government schemes (eligibility, benefits, application process)
- Required documents for various services
- Service provider locations and contact information
- Frequently asked questions and answers

**FR-11**: The system SHALL use semantic similarity search to retrieve the top-3 most relevant knowledge chunks for a given user query.

**FR-12**: The system SHALL support district-specific knowledge packs that can be loaded and updated independently.

**FR-13**: The system SHALL use a lightweight embedding model (e.g., all-MiniLM-L6-v2 quantized) for generating query and document embeddings on-device.

### 4.4 Multilingual Response Generation
**FR-14**: The system SHALL generate responses in the user's selected language by:
- Using the SLM to generate responses in English/Hindi
- Applying neural machine translation for Tamil/Telugu if needed
- Ensuring cultural and contextual appropriateness

**FR-15**: The system SHALL adapt language complexity based on user literacy level (inferred from interaction patterns).

**FR-16**: The system SHALL use simple, jargon-free language and provide explanations for technical terms when necessary.

### 4.5 Structured Output Generation
**FR-17**: The system SHALL generate structured outputs in predefined templates:
- **Health Summary Template**: Chief complaint, symptoms list, duration, severity, relevant history, red flags
- **Scheme Eligibility Template**: Scheme name, eligibility criteria, user's match status, required documents, application steps
- **Document Checklist Template**: Service name, document list with descriptions, where to obtain each document
- **Service Navigation Template**: Service provider details, location, timings, process steps, estimated duration

**FR-18**: The system SHALL support output export in multiple formats: plain text, PDF, and shareable image for WhatsApp/SMS.

**FR-19**: The system SHALL provide both voice readback and visual display of generated outputs.

### 4.6 Offline Operation
**FR-20**: The system SHALL function completely offline for all core capabilities with no degradation in primary functionality.

**FR-21**: The system SHALL store all AI models, knowledge bases, and language resources locally on the device.

**FR-22**: The system SHALL operate within 4GB RAM and 2GB storage constraints typical of mid-range Android devices.

**FR-23**: The system SHALL provide graceful degradation if optional online features (updates, analytics) are unavailable.

### 4.7 Model and Knowledge Pack Management
**FR-24**: The system SHALL support over-the-air updates for:
- Knowledge base content (new schemes, updated guidelines)
- District-specific information packs
- Model improvements (when compatible with device constraints)

**FR-25**: The system SHALL allow manual knowledge pack installation via SD card or USB transfer for completely offline deployment scenarios.

**FR-26**: The system SHALL version all knowledge packs and models to ensure compatibility and enable rollback if needed.

**FR-27**: The system SHALL display current knowledge pack version, coverage area, and last update date to users.

---

## 5. Non-Functional Requirements

### 5.1 Performance
**NFR-01**: The system SHALL respond to user queries with <3 seconds end-to-end latency (speech input to speech output) for 90% of queries.

**NFR-02**: The system SHALL perform intent classification and entity extraction within <500ms using the on-device SLM.

**NFR-03**: The system SHALL retrieve relevant knowledge from the local vector database within <1 second.

**NFR-04**: The system SHALL generate structured outputs within <2 seconds after information gathering is complete.

### 5.2 Resource Constraints
**NFR-05**: The system SHALL operate on devices with minimum specifications:
- 4GB RAM
- Quad-core processor (2.0 GHz)
- 2GB available storage
- Android 10 or higher

**NFR-06**: The system SHALL maintain peak memory usage below 2.5GB during active operation.

**NFR-07**: The system SHALL optimize battery consumption to enable 4+ hours of continuous usage on a typical 4000mAh battery.

**NFR-08**: The system SHALL use quantized models (4-bit or 8-bit) to reduce model size and inference latency.

### 5.3 Privacy & Security
**NFR-09**: The system SHALL process all user data (voice, text, personal information) exclusively on-device without cloud transmission.

**NFR-10**: The system SHALL not store personally identifiable information (PII) beyond the current session unless explicitly requested by the user.

**NFR-11**: The system SHALL encrypt any stored user data using AES-256 encryption.

**NFR-12**: The system SHALL provide clear privacy notices explaining data handling practices in the user's language.

**NFR-13**: The system SHALL comply with India's Digital Personal Data Protection Act (DPDPA) 2023 requirements.

### 5.4 Usability
**NFR-14**: The system SHALL be operable by users with zero prior smartphone experience through voice-only interaction.

**NFR-15**: The system SHALL provide large, high-contrast visual elements for users with limited vision.

**NFR-16**: The system SHALL use culturally appropriate icons, colors, and metaphors familiar to rural Indian users.

**NFR-17**: The system SHALL complete 80% of common tasks within 3 conversational turns.

**NFR-18**: The system SHALL achieve >70% task completion rate in user testing with target demographic.

### 5.5 Reliability
**NFR-19**: The system SHALL maintain 99% uptime during operating hours (no crashes or freezes).

**NFR-20**: The system SHALL gracefully handle edge cases (unintelligible speech, out-of-scope queries, corrupted data) without crashing.

**NFR-21**: The system SHALL provide meaningful error messages in the user's language when failures occur.

**NFR-22**: The system SHALL recover from errors and allow users to retry or rephrase their queries.

### 5.6 Scalability
**NFR-23**: The system SHALL support deployment across 100+ districts with district-specific knowledge packs.

**NFR-24**: The system SHALL support addition of new languages through modular language pack architecture.

**NFR-25**: The system SHALL support knowledge base expansion to 10,000+ documents without significant performance degradation.

### 5.7 Maintainability
**NFR-26**: The system SHALL use modular architecture to enable independent updates of ASR, SLM, TTS, and knowledge base components.

**NFR-27**: The system SHALL provide logging and diagnostics capabilities for troubleshooting deployment issues.

**NFR-28**: The system SHALL support A/B testing of different model configurations in field deployments.

---

## 6. AI Justification

### 6.1 Why AI is Required

**Problem Complexity**: The challenge of understanding natural, unstructured speech in multiple languages and mapping it to appropriate services cannot be solved with rule-based systems because:

1. **Linguistic Variability**: Users express the same intent in countless ways. A user seeking healthcare might say:
   - "मेरे पेट में दर्द है" (My stomach hurts)
   - "खाना खाने के बाद तकलीफ होती है" (I feel discomfort after eating)
   - "पेट खराब है, दवा चाहिए" (Stomach is upset, need medicine)
   
   Rule-based pattern matching would require thousands of hand-crafted rules and would still fail on novel phrasings.

2. **Contextual Understanding**: Users often provide information across multiple conversational turns, reference previous statements implicitly, and mix multiple concerns. Example:
   - User: "मुझे बुखार है" (I have fever)
   - System: "कब से?" (Since when?)
   - User: "तीन दिन, और खांसी भी" (Three days, and also cough)
   
   Understanding that "तीन दिन" refers to fever duration and "खांसी" is an additional symptom requires contextual reasoning impossible with simple rules.

3. **Intent Ambiguity**: The same surface form can have different intents based on context:
   - "मुझे पैसा चाहिए" could mean:
     - Seeking financial assistance scheme
     - Asking about pension eligibility
     - Inquiring about loan programs
   
   Disambiguating requires reasoning about user profile, conversation history, and domain knowledge.

4. **Knowledge Grounding**: Answering questions requires retrieving and synthesizing information from large knowledge bases. Example:
   - User: "क्या मुझे आयुष्मान योजना मिल सकती है?" (Can I get Ayushman scheme?)
   - System must:
     - Understand this refers to Ayushman Bharat health insurance
     - Retrieve eligibility criteria from knowledge base
     - Ask relevant questions (income, family size, existing coverage)
     - Reason about eligibility based on user responses
     - Explain decision in user's language

### 6.2 Why Rule-Based Logic is Insufficient

**Rule-based systems fail because:**

1. **Exponential Rule Growth**: Covering even 100 common intents across 3 languages with 10 variations each requires 3,000 rules. Adding context-dependent rules explodes this combinatorially.

2. **Brittleness**: Rules break on slight variations. "पेट में दर्द" vs "पेट दुख रहा है" vs "पेट में तकलीफ" would need separate rules despite identical meaning.

3. **No Generalization**: Rules cannot handle novel phrasings or new domains without manual updates.

4. **Poor Multilingual Support**: Maintaining parallel rule sets for multiple languages is impractical and error-prone.

5. **No Semantic Understanding**: Rules match surface patterns but don't understand meaning. "I don't have fever" and "I have fever" might both match a "fever" rule.

### 6.3 How SLM Reasoning Enables Flexible Understanding

**Small Language Models (SLMs) provide:**

1. **Semantic Understanding**: SLMs trained on large text corpora learn semantic representations that capture meaning beyond surface patterns. They understand that "पेट में दर्द", "stomach pain", and "abdominal discomfort" are semantically equivalent.

2. **Contextual Reasoning**: Transformer-based SLMs maintain conversation context through attention mechanisms, enabling them to:
   - Resolve pronouns and references ("it", "that", "the same")
   - Track entities across turns (symptoms, dates, family members)
   - Understand implicit information ("still hurts" implies continuation)

3. **Few-Shot Adaptation**: SLMs can be prompted with examples to perform new tasks without retraining:
   ```
   Example: User says "बुखार है" → Extract: {symptom: "fever", severity: "unknown"}
   Now extract from: "सिर बहुत दर्द कर रहा है"
   ```

4. **Multilingual Transfer**: Modern SLMs trained on multilingual data can transfer understanding across languages, reducing the need for language-specific engineering.

5. **Structured Output Generation**: SLMs can be prompted to generate outputs in specific formats (JSON, templates) enabling structured information extraction and document generation.

### 6.4 How Local RAG Provides Contextual Responses

**Retrieval Augmented Generation (RAG) addresses SLM limitations:**

1. **Knowledge Grounding**: SLMs have limited parametric knowledge and can hallucinate. RAG grounds responses in retrieved factual documents:
   - User asks about "PM-KISAN scheme"
   - System retrieves official scheme document from local vector DB
   - SLM generates response based on retrieved facts, not memorized (potentially incorrect) information

2. **Up-to-date Information**: Knowledge base can be updated independently of the model, ensuring current information about schemes, guidelines, and services.

3. **Verifiable Responses**: Retrieved documents provide source attribution, enabling users and service providers to verify information.

4. **Domain Specialization**: Local knowledge base contains domain-specific information (government schemes, local health protocols) not present in general SLM training data.

5. **Efficient On-Device Operation**: RAG enables using smaller SLMs (3-4B parameters) by offloading factual knowledge to the retrieval system, making on-device deployment feasible.

**RAG Pipeline:**
```
User Query → Embed Query → Search Vector DB → Retrieve Top-K Docs → 
Augment SLM Prompt with Retrieved Docs → Generate Response
```

### 6.5 Why This Approach is Deployment-Ready

1. **Quantized SLMs** (4-bit) reduce model size from 12GB to 2-3GB, enabling deployment on mid-range devices.

2. **Efficient Vector Search** using HNSW or IVF indices enables <1s retrieval on mobile devices.

3. **Proven Technology Stack**: Llama.cpp, ONNX Runtime, and Faiss provide production-ready on-device inference.

4. **Incremental Improvement**: System can be deployed with basic capabilities and improved through knowledge base updates without model retraining.

---

## 7. Inclusion & Public Impact

### 7.1 Who Benefits

**Direct Beneficiaries (Estimated 50M+ citizens):**

1. **Rural Women**: Often primary healthcare seekers for family but face language barriers and low digital literacy. SevaSetu enables them to:
   - Describe children's symptoms and get structured summaries for doctors
   - Learn about maternal health schemes in their language
   - Navigate government services independently

2. **Elderly Citizens**: Struggle with digital interfaces but can speak naturally. SevaSetu provides:
   - Voice-first interaction requiring no typing or navigation
   - Access to pension and healthcare schemes
   - Medication guidance and health monitoring support

3. **Migrant Workers**: Often lack awareness of services in new locations. SevaSetu offers:
   - Information about local health facilities and government offices
   - Guidance on accessing services without permanent address
   - Multilingual support for workers from different states

4. **Small Farmers**: Need information about agricultural schemes, subsidies, and documentation. SevaSetu provides:
   - Scheme eligibility checking
   - Document preparation guidance
   - Connection to agricultural extension services

**Indirect Beneficiaries:**

1. **Healthcare Workers**: Receive structured patient information, reducing consultation time and improving diagnosis accuracy.

2. **Government Officials**: Spend less time on basic information queries, can focus on service delivery.

3. **Community Organizations**: Can use SevaSetu as a tool for outreach and service facilitation.

### 7.2 Measurable Impact

**Access Metrics:**
- **Target**: Enable 100,000+ service interactions in first year across 10 pilot districts
- **Metric**: Number of structured outputs generated (symptom summaries, scheme applications, document checklists)

**Inclusion Metrics:**
- **Target**: 60% of users are women, 40% are elderly (>60 years)
- **Metric**: User demographic data (anonymized, aggregated)

**Efficiency Metrics:**
- **Target**: Reduce average time to access service information from 2+ hours (travel + waiting + inquiry) to <10 minutes
- **Metric**: User-reported time savings

**Outcome Metrics:**
- **Target**: 70% of users successfully complete intended service access (doctor visit, scheme application, document submission)
- **Metric**: Follow-up surveys and service provider confirmation

**Language Inclusion:**
- **Target**: 80% of interactions in regional languages (Tamil, Telugu) vs Hindi/English
- **Metric**: Language selection and usage analytics

### 7.3 Deployment Feasibility

**Pilot Deployment (Months 1-6):**
- Deploy in 10 districts across 3 states (Tamil Nadu, Telangana, Uttar Pradesh)
- Partner with existing CSCs and PHCs for device placement
- Train 100 community facilitators on system usage
- Gather user feedback and iterate on knowledge base

**Scale-Up (Months 7-12):**
- Expand to 50 districts based on pilot learnings
- Add 2 more languages (Kannada, Bengali)
- Develop district-specific knowledge packs
- Establish update and maintenance processes

**Sustainability Model:**
- Government partnership for CSC deployment and maintenance
- NGO partnerships for community facilitation
- Minimal operational cost due to offline operation (no cloud fees)
- Knowledge base updates through volunteer and government contributions

**Infrastructure Requirements:**
- Mid-range Android tablets (₹15,000-20,000) at CSCs and PHCs
- One-time knowledge pack installation
- Optional periodic updates via mobile data or WiFi when available
- No ongoing connectivity or cloud service costs

### 7.4 Societal Value

**Equity**: Levels the playing field for citizens regardless of language, literacy, or location. A farmer in rural Tamil Nadu gets the same quality of service guidance as an urban English speaker.

**Empowerment**: Reduces dependency on intermediaries who may charge fees or provide biased information. Citizens can independently explore options and make informed decisions.

**Efficiency**: Reduces burden on healthcare workers and government officials by providing pre-structured information, enabling them to serve more citizens effectively.

**Trust**: Transparent, on-device processing builds trust among users concerned about privacy. Clear explanations of system limitations prevent over-reliance.

**Scalability**: Offline-first architecture enables deployment in areas where cloud-based solutions are infeasible, reaching the most underserved populations.

---

## 8. Responsible AI Considerations

### 8.1 Bias Awareness

**Potential Biases:**

1. **Language Bias**: ASR and SLM models may perform better on standard dialects than regional variations or accents. Rural speakers may face higher error rates.

2. **Gender Bias**: Training data may contain gender stereotypes (e.g., associating certain health conditions or schemes with specific genders).

3. **Socioeconomic Bias**: Knowledge base may inadvertently prioritize schemes or services more relevant to certain economic groups.

4. **Geographic Bias**: Initial deployment in select districts may create knowledge gaps for other regions.

**Mitigation Strategies:**

1. **Diverse Training Data**: Include speech samples from multiple dialects, accents, age groups, and genders in ASR fine-tuning.

2. **Bias Testing**: Regularly test system performance across demographic groups and measure disparities.

3. **Inclusive Knowledge Base**: Ensure knowledge base covers schemes and services relevant to all socioeconomic groups, genders, and age groups.

4. **Community Feedback**: Establish channels for users to report biased or inappropriate responses.

5. **Transparent Limitations**: Clearly communicate which languages, dialects, and regions are currently supported.

### 8.2 Explainability

**System Transparency:**

1. **Intent Confirmation**: System explicitly confirms understood intent before proceeding:
   - "आप स्वास्थ्य योजना के बारे में जानना चाहते हैं?" (You want to know about health scheme?)

2. **Source Attribution**: When providing information, system indicates source:
   - "आयुष्मान भारत की आधिकारिक जानकारी के अनुसार..." (According to official Ayushman Bharat information...)

3. **Reasoning Steps**: For eligibility decisions, system explains criteria:
   - "आपकी आय सीमा से अधिक है, इसलिए आप इस योजना के लिए पात्र नहीं हैं" (Your income exceeds the limit, so you are not eligible for this scheme)

4. **Confidence Indicators**: System indicates uncertainty when appropriate:
   - "मुझे पूरा यकीन नहीं है, कृपया अधिकारी से पुष्टि करें" (I'm not completely sure, please confirm with an official)

### 8.3 System Limitations

**Explicit Limitations Communicated to Users:**

1. **Not a Replacement for Professionals**: System clearly states it provides guidance, not medical diagnosis or legal advice:
   - "यह जानकारी केवल मार्गदर्शन के लिए है। कृपया डॉक्टर से परामर्श लें।" (This information is for guidance only. Please consult a doctor.)

2. **Knowledge Boundaries**: System acknowledges when queries are outside its knowledge base:
   - "मेरे पास इस विषय की जानकारी नहीं है" (I don't have information on this topic)

3. **Temporal Limitations**: System indicates knowledge base version and last update date, warning users that information may be outdated.

4. **Language Limitations**: System clearly states which languages and dialects are supported and may suggest alternatives if user's language is not available.

5. **No Emergency Services**: System explicitly warns it cannot handle medical emergencies and provides emergency contact numbers.

### 8.4 User Transparency

**Privacy Notices:**

1. **On-Device Processing**: System prominently displays that all data is processed locally and not sent to servers.

2. **Data Retention**: System explains what data is stored (conversation history for current session only) and when it's deleted (on app close).

3. **Optional Analytics**: If anonymized usage analytics are collected (when online), system requests explicit consent with clear explanation.

**Consent Mechanisms:**

1. **First-Time Setup**: Users are walked through privacy settings and data handling practices in their language.

2. **Ongoing Reminders**: System periodically reminds users about privacy protections, especially when sensitive information is shared.

3. **Opt-Out Options**: Users can disable any optional features (analytics, updates) without losing core functionality.

**Accountability:**

1. **Feedback Mechanism**: Users can report incorrect information, biased responses, or system failures through simple voice or visual interface.

2. **Human Escalation**: System provides contact information for human support when it cannot adequately address user needs.

3. **Audit Trail**: System maintains anonymized logs of system failures and user-reported issues for continuous improvement.

### 8.5 Safety Considerations

**Harm Prevention:**

1. **Medical Safety**: System never provides specific medical advice (drug dosages, treatment recommendations) that could cause harm if incorrect.

2. **Financial Safety**: System warns users about potential scams and emphasizes that government services are free or low-cost.

3. **Misinformation Prevention**: System only provides information from verified sources (government websites, official guidelines) and clearly marks any uncertain information.

4. **Vulnerable User Protection**: System includes special safeguards for queries related to vulnerable populations (children, elderly, pregnant women) and escalates to human support when appropriate.

**Content Moderation:**

1. **Inappropriate Content**: System detects and refuses to engage with abusive, discriminatory, or harmful queries.

2. **Out-of-Scope Queries**: System politely redirects users when queries are outside its intended domain (healthcare and government services).

---

## 9. Success Criteria

### 9.1 Technical Success Criteria

1. **Accuracy**: Intent classification accuracy >85% on test set of 1000+ diverse queries
2. **Latency**: End-to-end response time <3 seconds for 90% of queries
3. **Reliability**: System uptime >99% during operating hours
4. **Resource Efficiency**: Peak memory usage <2.5GB, storage footprint <2GB
5. **Offline Operation**: 100% of core features functional without internet

### 9.2 User Success Criteria

1. **Task Completion**: 70% of users successfully complete intended task
2. **User Satisfaction**: >4/5 average rating on ease of use
3. **Adoption**: 60% of users return for second interaction within 30 days
4. **Inclusion**: 60% of users are women, 40% are elderly, 80% use regional languages

### 9.3 Impact Success Criteria

1. **Reach**: 100,000+ service interactions in first year
2. **Time Savings**: Average 2+ hours saved per service access
3. **Service Completion**: 70% of users successfully complete service access (verified by service providers)
4. **Awareness**: 50% increase in awareness of available schemes among users

---

## 10. Out of Scope (V1)

The following capabilities are explicitly out of scope for the initial version:

1. **Real-time Translation**: Live translation during phone calls or in-person conversations
2. **Image/Document Processing**: OCR or analysis of uploaded documents
3. **Appointment Booking**: Direct integration with government appointment systems
4. **Payment Processing**: Handling of financial transactions
5. **Video Consultation**: Video call facilitation with service providers
6. **Personalized Recommendations**: AI-driven proactive suggestions based on user profile
7. **Multi-user Profiles**: Support for multiple family members on single device
8. **Advanced Medical Diagnosis**: Symptom-based disease prediction or diagnosis
9. **Legal Advice**: Interpretation of laws or legal document preparation
10. **Languages Beyond Initial 3**: Support for languages other than Hindi, Tamil, Telugu

These features may be considered for future versions based on user feedback and deployment learnings.

---

## 11. Assumptions and Dependencies

### 11.1 Assumptions

1. **Device Availability**: Target users have access to mid-range Android devices through CSCs, PHCs, or personal ownership
2. **Basic Digital Literacy**: Users can perform basic operations (power on device, tap screen) with minimal training
3. **Quiet Environment**: Voice interaction occurs in relatively quiet environments (not on busy roads or in crowds)
4. **Knowledge Base Accuracy**: Government scheme information and health guidance provided by authorities is accurate and up-to-date
5. **User Trust**: Users are willing to share personal information with the system after privacy assurances

### 11.2 Dependencies

1. **Government Partnership**: Access to official scheme information, guidelines, and service provider data
2. **Language Resources**: Availability of ASR, TTS, and translation models for target languages
3. **Hardware Availability**: Procurement of suitable Android devices for deployment
4. **Community Facilitators**: Availability of trained facilitators to assist users initially
5. **Maintenance Support**: Ongoing support for device maintenance and knowledge base updates

### 11.3 Risks

1. **Model Performance**: Quantized models may not achieve desired accuracy on diverse accents and dialects
2. **User Adoption**: Low digital literacy users may be hesitant to use voice-based AI system
3. **Knowledge Staleness**: Rapidly changing government schemes may outpace knowledge base update cycles
4. **Device Constraints**: Older or lower-spec devices may not support on-device inference
5. **Sustainability**: Long-term funding and maintenance may be challenging without clear ownership

---

## Document Version

- **Version**: 1.0
- **Date**: 2026-02-14
- **Status**: Draft for Hackathon Submission
- **Authors**: AI for Bharat Hackathon Team - SevaSetu AI

---

**End of Requirements Document**
