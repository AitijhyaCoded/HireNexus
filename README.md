# HireNexus

A multi-agent AI hiring evaluation platform that simulates a virtual interview panel through transparent debate protocols.

## Overview

HireNexus replaces opaque single-model AI screening with a transparent multi-agent debate system. Multiple specialized AI models (Llama 3, Gemini, Mistral) analyze candidate materials—resumes, code repositories, and voice recordings—then debate their findings to produce balanced, auditable hiring recommendations.

The core innovation: instead of a black-box decision, recruiters see the full deliberation process where AI agents present evidence, challenge each other's conclusions, and synthesize a consensus report that can be audited and overridden.

## Key Features

- **Multi-Agent Debate System**: Three specialized AI agents (Code Reviewer, Soft Skills Evaluator, Cultural Fit Assessor) independently analyze candidates then debate findings
- **Multi-Modal Analysis**: Evaluates resumes (PDF), voice interviews (audio transcription), and GitHub repositories
- **Real-Time Debate Viewer**: Watch AI agents present evidence and challenge each other via WebSocket streaming
- **Transparent Reasoning**: Every recommendation includes cited evidence from candidate materials
- **Human-in-the-Loop**: Recruiters make final decisions and can override AI recommendations with justification
- **RAG-Enhanced Context**: Job descriptions stored as vector embeddings for semantic matching

## Architecture

### Technology Stack

**Frontend:**
- Next.js 14 (App Router)
- React 18
- Tailwind CSS
- WebSocket client for real-time updates

**Backend:**
- FastAPI (Python 3.11+)
- Clerk Authentication
- CrewAI for agent orchestration
- LiteLLM for unified model interface
- SQLAlchemy 2.0 ORM

**AWS Infrastructure:**
- Amazon S3 (asset storage)
- Amazon Transcribe (speech-to-text)
- Amazon RDS PostgreSQL 15 (relational data)
- Amazon OpenSearch Serverless (vector embeddings)

**AI Models:**
- Llama 3 (code analysis)
- Gemini Pro (soft skills evaluation)
- Mistral Medium (cultural fit assessment)
- Claude via Bedrock (consensus synthesis)

### System Flow

```
1. Recruiter uploads candidate assets (resume, audio, GitHub URL)
2. System stores files in S3, transcribes audio, fetches repo metadata
3. Recruiter initiates evaluation against a job description
4. Three AI agents perform parallel independent analysis
5. Agents enter structured debate: presentation → cross-examination → consensus
6. Supervisor agent synthesizes final report with confidence score
7. Recruiter reviews live debate transcript and makes final decision
```

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- AWS account with configured credentials
- Docker (optional, for local development)

### Installation

```bash
# Clone repository
git clone https://github.com/AitijhyaCoded/HireNexus.git
cd HireNexus

# Backend setup
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt

# Frontend setup
cd ../frontend
npm install
```

### Configuration

Create `.env` files:

**Backend (.env):**
```bash
AWS_REGION=us-east-1
S3_BUCKET_NAME=candidate-assets-prod
RDS_CONNECTION_STRING=postgresql://user:pass@host:5432/hirenexus
OPENSEARCH_ENDPOINT=https://your-opensearch-domain.region.es.amazonaws.com
OLLAMA_BASE_URL=http://localhost:11434
GOOGLE_API_KEY=your_gemini_key
MISTRAL_API_KEY=your_mistral_key
```

**Frontend (.env.local):**
```bash
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_WS_URL=ws://localhost:8000/ws
```

### Running Locally

```bash
# Start backend
cd backend
uvicorn app.main:app --reload

# Start frontend (separate terminal)
cd frontend
npm run dev
```

Access the dashboard at `http://localhost:3000`

## API Endpoints

### Candidate Management
- `POST /api/candidates` - Create candidate with assets
- `GET /api/candidates/{id}` - Retrieve candidate details
- `PUT /api/candidates/{id}` - Update candidate info
- `DELETE /api/candidates/{id}` - Remove candidate

### Job Descriptions
- `POST /api/job-descriptions` - Create JD with embeddings
- `GET /api/job-descriptions/{id}` - Retrieve JD
- `PUT /api/job-descriptions/{id}` - Update JD and re-embed

### Evaluation
- `POST /api/evaluate` - Start debate session
- `GET /api/evaluate/{session_id}` - Get session status
- `WS /ws/debate/{session_id}` - Real-time debate stream

### Reports
- `GET /api/reports/{report_id}` - Get consensus report
- `POST /api/reports/{report_id}/decision` - Record final decision
- `GET /api/evaluations` - List evaluation history

## Debate Protocol

The multi-agent evaluation follows a structured four-phase process:

1. **Independent Analysis (Parallel)**: Each agent analyzes candidate materials independently (30-60s)
2. **Presentation Round**: Agents present findings sequentially with evidence citations (2 min each)
3. **Cross-Examination**: Agents challenge each other's conclusions (max 5 exchanges per agent)
4. **Consensus Synthesis**: Supervisor agent reviews all statements and generates balanced report

## Testing

```bash
# Run unit tests
pytest tests/unit

# Run property-based tests
pytest tests/property -m property_test

# Check coverage
pytest --cov=app --cov-report=html
```

The project uses both traditional unit tests and property-based testing (Hypothesis) to verify correctness properties across all inputs.

## Deployment

### Backend (AWS App Runner)
```bash
docker build -t hirenexus-api .
aws ecr push ...
aws apprunner update-service ...
```

### Frontend (Vercel)
```bash
vercel deploy --prod
```

See `design.md` for complete deployment architecture and CI/CD pipeline configuration.

## Project Structure

```
HireNexus/
├── backend/
│   ├── app/
│   │   ├── api/          # FastAPI routes
│   │   ├── services/     # Business logic
│   │   ├── models/       # SQLAlchemy models
│   │   ├── agents/       # CrewAI agent definitions
│   │   └── main.py       # Application entry point
│   ├── tests/
│   │   ├── unit/         # Unit tests
│   │   └── property/     # Property-based tests
│   └── requirements.txt
├── frontend/
│   ├── app/              # Next.js pages
│   ├── components/       # React components
│   └── package.json
├── .kiro/
│   └── specs/            # Design and requirements docs
└── README.md
```

## Documentation

- [Design Document](/.kiro/specs/hire-nexus/design.md) - Complete architecture and technical specifications
- [Requirements](/.kiro/specs/hire-nexus/requirements.md) - Functional requirements and acceptance criteria

## Security

- All candidate assets encrypted at rest (AES-256) and in transit (TLS 1.3)
- JWT-based authentication with 24-hour token expiration
- Rate limiting: 100 requests/minute per recruiter
- IAM role-based access to AWS services

## Contributing

Contributions welcome! Please ensure:
- All tests pass (`pytest`)
- Code coverage remains above 80%
- Property tests included for new correctness properties
- API changes documented in OpenAPI spec

## License

MIT License - see LICENSE file for details

## Support

For issues or questions, please open a GitHub issue or contact the development team.
