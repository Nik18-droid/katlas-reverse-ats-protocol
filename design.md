# KAtlas Technical Architecture
## Reverse-ATS Protocol - System Design

---

## Architecture Overview

KAtlas is designed as a **three-phase evolution** from service → agent → network protocol. The architecture supports this progression while maintaining data continuity and learning feedback loops.

```
┌─────────────────────────────────────────────────────────────────┐
│                    KATLAS ARCHITECTURE                          │
│                                                                 │
│  Phase 1: Human-in-Loop  →  Phase 2: Agent  →  Phase 3: API     │
│         (2026)                   (2027)            (2028+)      │
└─────────────────────────────────────────────────────────────────┘

         INPUT LAYER              INTELLIGENCE LAYER           OUTPUT LAYER
    ┌──────────────────┐      ┌────────────────────┐      ┌─────────────────┐
    │                  │      │                    │      │                 │
    │  • Raw Resume    │──────│  Audit Engine      │──────│  • Audit Report │
    │  • Job Desc      │      │  • NLP Analysis    │      │  • Pivot Plan   │
    │  • Career Data   │      │  • Pattern Match   │      │  • Dossier PDF  │
    │  • User Input    │      │  • Risk Detection  │      │  • JSON Output  │
    │                  │      │                    │      │                 │
    └──────────────────┘      └────────────────────┘      └─────────────────┘
                                      │
                                      ↓
                          ┌──────────────────────────┐
                          │   GOLDEN DATASET DB      │
                          │   • Labeled Transforms   │
                          │   • Outcome Tracking     │
                          │   • Vector Embeddings    │
                          └──────────────────────────┘
```

---

## Technical Stack

### Phase 1: Human-in-Loop Service (Current - 2026)

**Frontend:**
- Platform: Web-based (React/Next.js)
- Design System: Custom "Cyber-Noir" aesthetic
  - Colors: Black (#000000), Gold (#F59E0B)
  - Typography: Space Grotesk (headers), Outfit (body)
- Mobile-responsive for candidate input

**Backend:**
- Language: Python (FastAPI)
- Authentication: JWT-based
- File Processing: PyPDF2 for resume parsing
- Document Generation: LaTeX → PDF (ATS-optimized)

**AI/ML Layer:**
- Primary LLM: Claude Opus 4.5 (semantic reasoning)
- Advanced Reasoning: OpenAI o1 (strategic pivots)
- Prompt Engineering:
  - Context: Job description + candidate data
  - Task: Identify fatal errors, generate governance language
  - Output: Structured JSON with classifications

**Database:**
- PostgreSQL (candidate profiles, transformation history)
- Simple vector storage for early semantic matching

**Infrastructure:**
- Cloud: AWS (India region for latency)
- Storage: S3 for documents
- Compute: Lambda for processing workflows

---

### Phase 2: Automated Agent (2027)

**Enhancements to Phase 1:**

**Vector Database Integration:**
- Platform: Pinecone or Weaviate
- Purpose: Semantic search across Golden Dataset
- Use cases:
  - Match candidate skills to JD requirements
  - Find similar successful transformations
  - Identify optimal pivot strategies

**Custom Fine-Tuned Model:**
- Base: Claude 4.5 or Gemini 3 Pro
- Training Data: Golden Dataset (labeled transformations)
- Training Objective: 
  - Input: Original resume text + JD
  - Output: Specific semantic pivots with confidence scores
  - Loss function: Weighted by interview outcome (success = higher weight)

**Automated Pipeline:**
```
Resume Upload → PDF Parse → Semantic Analysis → Pattern Matching → 
Strategic Diagnosis → Transformation Generation → Quality Check → 
PDF + JSON Output → Outcome Tracking
```

**Quality Assurance System:**
- Automated checks:
  - ATS compatibility score
  - Keyword density analysis
  - Governance language detection
  - Formatting validation
- Human review threshold: <80% confidence score

---

### Phase 3: API Protocol Network (2028+)

**KAtlas Protocol Standard:**

**Data Schema (JSON):**
```json
{
  "katlas_id": "unique_identifier",
  "version": "1.0",
  "candidate": {
    "name": "encrypted",
    "contact": "encrypted",
    "verified": true
  },
  "professional_profile": {
    "vertical": "BFSI_Governance",
    "seniority": "Senior_Analyst",
    "risk_profile": {
      "scope_control": 8.5,
      "compliance_depth": 9.2,
      "stakeholder_management": 7.8
    },
    "semantic_tags": ["regulatory_compliance", "process_optimization", "risk_mitigation"],
    "experience_blocks": [
      {
        "company": "redacted",
        "role_pivot": "Governance Lead → Strategic Risk Analyst",
        "governance_signals": ["orchestrated", "mitigated", "audited"],
        "impact_metrics": ["23% churn reduction", "44-country rollout"],
        "duration_months": 36
      }
    ]
  },
  "verification": {
    "katlas_audit_date": "2026-01-25",
    "interview_conversion_rate": 0.23,
    "last_updated": "2026-01-25"
  },
  "match_score": {
    "jd_alignment": 0.87,
    "ats_compatibility": 0.92,
    "hiring_manager_appeal": 0.85
  }
}
```

**API Endpoints:**

**For Candidates:**
- `POST /api/v1/candidate/upload` - Submit resume for processing
- `GET /api/v1/candidate/{katlas_id}` - Retrieve profile
- `PUT /api/v1/candidate/{katlas_id}` - Update profile
- `POST /api/v1/candidate/match` - Match against job description

**For Recruiters:**
- `GET /api/v1/recruiter/search` - Search candidates by criteria
- `POST /api/v1/recruiter/verify/{katlas_id}` - Verify candidate data
- `GET /api/v1/recruiter/match` - Get top matches for JD

**For Institutions (Universities/Bootcamps):**
- `POST /api/v1/institution/batch_upload` - Process student cohort
- `GET /api/v1/institution/analytics` - Placement success metrics
- `PUT /api/v1/institution/curriculum_align` - Curriculum recommendations

**Authentication & Security:**
- OAuth 2.0 for third-party integrations
- API keys for institutional access
- Rate limiting: 1000 requests/hour (standard), custom for enterprise
- Data encryption: TLS 1.3, AES-256 at rest
- Privacy: Candidate data encrypted, only shared with explicit consent

---

## Core Components Deep Dive

### 1. Audit Engine

**Purpose:** Identify fatal errors preventing interview success

**Technical Implementation:**

**Error Detection Algorithms:**

```python
def detect_generalist_trap(resume_text, jd_text):
    """
    Identifies lack of vertical specialization
    Returns: Boolean + specificity_score (0-1)
    """
    # Extract role keywords from JD
    jd_keywords = extract_domain_keywords(jd_text)
    
    # Calculate overlap with resume
    resume_keywords = extract_domain_keywords(resume_text)
    overlap_score = semantic_similarity(jd_keywords, resume_keywords)
    
    # Check for generic vs. specific language
    generic_phrases = ["managed team", "coordinated", "handled"]
    specific_count = count_domain_specific_terms(resume_text)
    
    if overlap_score < 0.4 and specific_count < 5:
        return True, overlap_score
    return False, overlap_score

def detect_governance_gap(resume_text):
    """
    Checks for risk mitigation and control language
    Returns: governance_score (0-10)
    """
    governance_keywords = [
        "mitigated", "orchestrated", "audited", "compliance",
        "risk framework", "regulatory", "governance", "controls",
        "scope management", "stakeholder alignment"
    ]
    
    # Count governance signals
    score = 0
    for keyword in governance_keywords:
        if keyword.lower() in resume_text.lower():
            score += 1
    
    # Weight by positioning (header vs. buried in text)
    header_bonus = check_governance_in_headers(resume_text)
    
    return min(10, score + header_bonus)
```

**Output Format:**
```json
{
  "audit_results": {
    "fatal_errors": [
      {
        "type": "GENERALIST_TRAP",
        "severity": "HIGH",
        "description": "Resume lacks vertical specialization in BFSI domain",
        "fix": "Pivot role from 'Business Analyst' to 'BFSI Governance Lead'"
      }
    ],
    "governance_score": 3.5,
    "ats_compatibility": 45,
    "recommended_pivots": ["governance_specialist", "risk_analyst"]
  }
}
```

---

### 2. Semantic Pivot Engine

**Purpose:** Transform "Doer" language → "Governance" language

**Technical Implementation:**

**Transformation Pipeline:**

```python
def semantic_pivot(original_text, target_domain, seniority_level):
    """
    Transforms generic language to governance-focused language
    Uses Claude Opus 4.5 with few-shot examples from Golden Dataset
    """
    # Retrieve similar successful transformations from vector DB
    similar_examples = vector_db.search(
        query=original_text,
        filter={"domain": target_domain, "outcome": "interview"},
        limit=5
    )
    
    # Construct prompt with examples
    prompt = f"""
    You are the KAtlas Semantic Pivot Engine. Transform the following text
    from generic "Doer" language to high-impact "Governance" language.
    
    Domain: {target_domain}
    Seniority: {seniority_level}
    
    Successful examples from our Golden Dataset:
    {format_examples(similar_examples)}
    
    Original text: {original_text}
    
    Transform this text following the pattern of successful examples.
    Focus on: risk mitigation, scope control, governance, regulatory compliance.
    Include specific metrics and scale indicators.
    """
    
    # Call LLM
    transformed = openai.chat.completions.create(
        model="Gemini 3 Pro",
        messages=[
            {"role": "system", "content": "You are KAtlas Pivot Engine"},
            {"role": "user", "content": prompt}
        ],
        temperature=0.3  # Low temperature for consistency
    )
    
    return transformed.choices[0].message.content

def validate_transformation(original, transformed):
    """
    Ensures transformation maintains factual accuracy
    while upgrading language quality
    """
    # Check semantic similarity (should be high)
    similarity = semantic_similarity_score(original, transformed)
    if similarity < 0.75:
        return False, "Transformation too divergent from original"
    
    # Check governance language upgrade
    governance_improvement = (
        detect_governance_gap(transformed) - 
        detect_governance_gap(original)
    )
    if governance_improvement < 2:
        return False, "Insufficient governance language upgrade"
    
    return True, "Transformation valid"
```

**Example Transformations:**

| Original (Doer Language) | Transformed (Governance Language) |
|--------------------------|-----------------------------------|
| "Managed a team of 5 analysts" | "Orchestrated cross-functional governance team (5 senior analysts) across 3 regulatory jurisdictions" |
| "Coordinated project deliverables" | "Architected milestone-driven delivery framework mitigating scope creep risk for ₹12Cr portfolio" |
| "Analyzed customer data" | "Implemented predictive churn framework reducing customer attrition by 23% across 2.4M user base" |

---

### 3. ATS Firewall Penetration System

**Purpose:** Ensure document passes ATS parsing + appeals to hiring managers

**Technical Implementation:**

**ATS Optimization Algorithm:**

```python
def optimize_for_ats(resume_data, jd_text):
    """
    Optimizes resume for both ATS parsing and human appeal
    """
    # Extract JD requirements
    jd_analysis = {
        "must_have_keywords": extract_must_have(jd_text),
        "nice_to_have_keywords": extract_nice_to_have(jd_text),
        "hidden_requirements": infer_implicit_requirements(jd_text),
        "company_culture_signals": detect_culture_keywords(jd_text)
    }
    
    # Calculate keyword coverage
    current_coverage = calculate_keyword_coverage(resume_data, jd_analysis)
    
    # Optimize without "keyword stuffing"
    optimized = {
        "summary": inject_keywords_naturally(
            resume_data["summary"],
            jd_analysis["must_have_keywords"],
            max_density=0.05  # 5% keyword density
        ),
        "experience": inject_in_context(
            resume_data["experience"],
            jd_analysis,
            strategy="semantic_alignment"
        )
    }
    
    # Generate ATS-friendly PDF
    pdf = generate_latex_pdf(
        optimized,
        template="ats_optimized",  # No tables, graphics, complex layouts
        fonts="standard_sans_serif"  # Arial/Helvetica for parsing
    )
    
    return pdf, optimized

def generate_latex_pdf(content, template, fonts):
    """
    Uses LaTeX for pixel-perfect ATS compatibility
    """
    latex_template = """
    \\documentclass[11pt,a4paper]{article}
    \\usepackage{geometry}
    \\geometry{margin=1in}
    \\usepackage{enumitem}
    \\setlist{nosep}
    \\pagenumbering{gobble}
    
    \\begin{document}
    
    {content_formatted}
    
    \\end{document}
    """
    
    # Compile LaTeX → PDF
    pdf_bytes = compile_latex(latex_template, content)
    
    # Validate ATS compatibility
    ats_score = test_ats_parsing(pdf_bytes)
    
    return pdf_bytes
```

**ATS Compatibility Checklist:**
- ✅ Standard fonts (Arial, Helvetica, Calibri)
- ✅ No headers/footers (ATS blind spots)
- ✅ No tables (parsing issues)
- ✅ No graphics/images
- ✅ Simple bullet points
- ✅ Standard section headers
- ✅ File size <1MB
- ✅ .docx and .pdf versions

---

### 4. Golden Dataset Learning System

**Purpose:** Continuous improvement via outcome-labeled data

**Data Collection Pipeline:**

```python
class GoldenDataset:
    def __init__(self):
        self.db = PostgreSQL()
        self.vector_db = Pinecone()
    
    def record_transformation(self, candidate_id, transformation_data):
        """
        Stores transformation with metadata for future learning
        """
        record = {
            "candidate_id": candidate_id,
            "timestamp": datetime.now(),
            "original_text": transformation_data["original"],
            "transformed_text": transformation_data["transformed"],
            "transformation_type": transformation_data["type"],
            "domain": transformation_data["domain"],
            "seniority": transformation_data["seniority"],
            "ats_score_before": transformation_data["ats_before"],
            "ats_score_after": transformation_data["ats_after"],
            "outcome": None,  # Updated later via feedback
            "interview_secured": None,
            "company": None,
            "role": None
        }
        
        # Store in PostgreSQL
        self.db.insert("transformations", record)
        
        # Generate vector embedding
        embedding = generate_embedding(transformation_data["transformed"])
        
        # Store in vector DB for semantic search
        self.vector_db.upsert(
            id=candidate_id,
            values=embedding,
            metadata=record
        )
    
    def update_outcome(self, candidate_id, outcome_data):
        """
        Updates transformation record with interview/offer outcome
        This creates the "labeled" dataset
        """
        self.db.update(
            "transformations",
            {"candidate_id": candidate_id},
            {
                "outcome": outcome_data["result"],  # "interview", "offer", "rejection"
                "interview_secured": outcome_data["interview_date"],
                "company": outcome_data["company"],
                "role": outcome_data["role"],
                "feedback": outcome_data["candidate_feedback"]
            }
        )
        
        # If successful, increase weight in training data
        if outcome_data["result"] in ["interview", "offer"]:
            self.vector_db.update_metadata(
                id=candidate_id,
                metadata={"training_weight": 2.0}  # Double weight for successful examples
            )
```

**Data Privacy:**
- Candidate PII encrypted at rest
- Anonymization for model training (names, companies redacted)
- Explicit consent for data usage
- GDPR/India Data Protection Act compliant

**Dataset Growth Projection:**
- Phase 1 (2026): 500-1000 labeled transformations
- Phase 2 (2027): 10,000+ labeled transformations
- Phase 3 (2028): 100,000+ labeled transformations

---

### 5. India-Specific Intelligence Layer

**Purpose:** Capture India market nuances generic AI misses

**Implementation:**

**Industry-Specific Ontologies:**

```python
BFSI_ONTOLOGY = {
    "regulatory_bodies": ["RBI", "SEBI", "IRDAI", "PFRDA"],
    "compliance_frameworks": ["Basel III", "Solvency II", "IFRS", "Ind AS"],
    "governance_keywords": [
        "regulatory compliance", "audit trail", "risk framework",
        "capital adequacy", "credit risk", "operational risk"
    ],
    "role_hierarchy": {
        "entry": "Analyst",
        "mid": "Senior Analyst / Lead",
        "senior": "Manager / AVP",
        "leadership": "VP / Director"
    }
}

IT_SERVICES_ONTOLOGY = {
    "company_types": ["Product", "Services", "Consulting", "GCC"],
    "delivery_models": ["Agile", "Waterfall", "Hybrid", "DevOps"],
    "governance_keywords": [
        "project governance", "SDLC compliance", "quality assurance",
        "risk mitigation", "stakeholder management"
    ],
    "skill_translation": {
        "doer": "executed requirements gathering",
        "governance": "architected requirements governance framework ensuring 100% UAT compliance"
    }
}

def apply_india_context(resume_data, target_industry):
    """
    Applies India-specific transformations based on industry
    """
    ontology = load_ontology(target_industry)
    
    # Inject regulatory context
    if "compliance" in resume_data:
        resume_data["compliance"] = add_regulatory_specifics(
            resume_data["compliance"],
            ontology["regulatory_bodies"]
        )
    
    # Translate education system
    if "education" in resume_data:
        resume_data["education"] = translate_indian_education(
            resume_data["education"]
        )
    
    # Add salary benchmarking context
    resume_data["salary_range"] = calculate_india_benchmark(
        resume_data["experience_years"],
        resume_data["role"],
        resume_data["location"]
    )
    
    return resume_data
```

**Education Translation:**
| Indian Credential | Global Equivalent |
|-------------------|-------------------|
| BTech/BE | Bachelor of Engineering |
| MBA | Master of Business Administration |
| CA (Chartered Accountant) | CPA equivalent + additional context |
| Tier-1 IIT/IIM | Stanford/MIT equivalent (reputation signal) |

---

## Scalability & Performance

### Load Handling

**Phase 1 (2026):**
- Expected load: 500 candidates/month
- Processing time: 2-4 hours per candidate (human-in-loop)
- Infrastructure: Single AWS EC2 instance

**Phase 2 (2027):**
- Expected load: 10,000 candidates/month
- Processing time: 5-10 minutes per candidate (automated)
- Infrastructure: Auto-scaling Lambda functions, managed Pinecone

**Phase 3 (2028+):**
- Expected load: 100,000+ candidates/month
- Processing time: <1 minute per API call
- Infrastructure: Kubernetes cluster, distributed vector DB, CDN

### Cost Optimization

**Phase 1 Costs:**
- OpenAI API: ~₹50 per candidate
- AWS infrastructure: ~₹5,000/month
- Total: ~₹30,000/month for 500 candidates

**Phase 2 Costs:**
- Fine-tuned model (self-hosted): ~₹5 per candidate
- Vector DB: ~₹50,000/month
- Total: ~₹100,000/month for 10,000 candidates

**Phase 3 Revenue:**
- B2C: ₹2,499 average per candidate
- B2B API: ₹500-₹2,000 per seat (universities/staffing firms)
- Target margins: 95% gross margin at scale

---

## Security & Compliance

**Data Security:**
- Encryption: TLS 1.3 in transit, AES-256 at rest
- Authentication: OAuth 2.0, JWT tokens, MFA for sensitive operations
- Access Control: Role-based (candidate, recruiter, admin)
- Audit Logging: All data access logged for 7 years

**Privacy Compliance:**
- GDPR-ready: Right to access, deletion, portability
- India Data Protection Act: Local data storage, consent management
- Candidate data anonymization for training
- Third-party data sharing: Explicit opt-in only

**Backup & Disaster Recovery:**
- Database: Daily backups, 30-day retention
- Disaster recovery: <4 hour RPO, <1 hour RTO
- Multi-region failover (AWS Mumbai + Singapore)

---

## Monitoring & Analytics

**System Health Metrics:**
- API response time: <500ms (p95)
- Uptime SLA: 99.9%
- Error rate: <0.1%
- ATS compatibility rate: >95%

**Business Metrics:**
- Interview Conversion Rate (ICR): Target >20%
- Customer Satisfaction (CSAT): Target >4.5/5
- NPS (Net Promoter Score): Target >50
- Revenue per candidate: Track against ₹2,499 target

**Dashboard:**
- Real-time candidate pipeline
- Transformation success rates
- Outcome tracking (interviews/offers)
- Revenue metrics
- System performance

---

## Development Roadmap

**Q1 2026 (Current):**
- ✅ MVP: Manual service with basic automation
- ✅ Branding: Cyber-Noir aesthetic implementation
- ✅ Payment: Razorpay integration
- 🔄 Data collection: First 50 labeled transformations

**Q2 2026:**
- Vector DB integration (Pinecone)
- Automated audit engine (70% automation)
- A/B testing different pivot strategies
- Expansion: 500 candidates

**Q3-Q4 2026:**
- Custom fine-tuned model training
- API beta for select partners (2-3 universities)
- Advanced analytics dashboard
- Team expansion: 1 ML engineer, 1 full-stack developer

**2027:**
- Full automation (90%)
- Public API launch
- B2B partnerships: 10+ institutions
- International expansion (Southeast Asia)

**2028+:**
- KAtlas Protocol standardization
- Industry adoption campaigns
- Enterprise features
- IPO preparation

---

## Technical Risks & Mitigation

**Risk 1: AI hallucination in transformations**
- Mitigation: Human QA for <80% confidence, validation algorithms

**Risk 2: ATS algorithm changes**
- Mitigation: Continuous monitoring, quarterly ATS compatibility audits

**Risk 3: Data privacy concerns**
- Mitigation: Encryption, anonymization, explicit consent, audit trails

**Risk 4: Model bias**
- Mitigation: Diverse training data, bias detection algorithms, regular audits

**Risk 5: Scalability bottlenecks**
- Mitigation: Cloud-native architecture, auto-scaling, performance monitoring

---

## Conclusion

KAtlas technical architecture is designed for:
1. **Immediate revenue generation** (Phase 1 service)
2. **Rapid learning** (Golden Dataset accumulation)
3. **Scalable automation** (Phase 2 agent)
4. **Network effect** (Phase 3 protocol)

The core innovation is not just the AI, but the **outcome-labeled dataset** collected during the manual phase. This proprietary data creates a defensible moat that generic AI cannot replicate.

By 2028, KAtlas aims to be the industry standard for candidate data exchange—replacing the PDF resume with a verified, machine-readable protocol that benefits candidates, recruiters, and institutions equally.
