# Requirements Document

## Introduction

HireNexus is a virtual hiring committee platform that replaces single-model AI screening with a transparent, multi-agent debate system. Multiple AI models (Llama, Gemini, Mistral) cross-examine candidate data including voice recordings, code repositories, and resumes to provide balanced, explainable hiring recommendations through simulated panel discussions.

## Glossary

- **System**: The HireNexus platform
- **Recruiter**: A human user who uploads candidate data and reviews AI recommendations
- **Candidate**: A job applicant whose materials are being evaluated
- **AI_Agent**: An individual AI model instance with a specific evaluation role (e.g., Code Reviewer, Soft Skills Coach)
- **Debate_Session**: A structured interaction where multiple AI_Agents analyze and discuss a candidate
- **Consensus_Report**: The final synthesized evaluation containing pros, cons, and reasoning
- **Job_Description**: The JD document that defines role requirements and evaluation criteria
- **Candidate_Asset**: Any uploaded file (resume PDF, audio recording, code repository link)
- **Vector_Database**: OpenSearch instance storing JD embeddings for RAG queries
- **Transcript**: The complete record of AI_Agent discussions and reasoning

## Requirements

### Requirement 1: Candidate Data Ingestion

**User Story:** As a Recruiter, I want to upload candidate materials through a dashboard, so that the AI agents can evaluate multiple data sources.

#### Acceptance Criteria

1. WHEN a Recruiter uploads a resume PDF, THEN THE System SHALL store it in Amazon S3 and create a Candidate profile
2. WHEN a Recruiter uploads an audio recording, THEN THE System SHALL store it in Amazon S3 and queue it for transcription
3. WHEN a Recruiter provides a GitHub repository URL, THEN THE System SHALL validate the URL format and store the reference
4. WHEN multiple Candidate_Assets are uploaded for one Candidate, THEN THE System SHALL associate all assets with the same Candidate profile
5. IF an uploaded file exceeds 50MB, THEN THE System SHALL reject the upload and display an error message

### Requirement 2: Audio Transcription

**User Story:** As a Recruiter, I want audio interviews automatically transcribed, so that AI agents can analyze spoken responses.

#### Acceptance Criteria

1. WHEN an audio file is uploaded, THEN THE System SHALL submit it to Amazon Transcribe for processing
2. WHEN transcription completes successfully, THEN THE System SHALL store the text transcript and associate it with the Candidate
3. IF transcription fails, THEN THE System SHALL log the error and notify the Recruiter
4. WHEN a transcript is generated, THEN THE System SHALL preserve timestamps for each spoken segment
5. THE System SHALL support audio formats including MP3, WAV, and M4A

### Requirement 3: Job Description Management

**User Story:** As a Recruiter, I want to define job requirements, so that AI agents can evaluate candidates against specific criteria.

#### Acceptance Criteria

1. WHEN a Recruiter creates a Job_Description, THEN THE System SHALL store it in Amazon RDS with a unique identifier
2. WHEN a Job_Description is saved, THEN THE System SHALL generate vector embeddings and store them in the Vector_Database
3. WHEN a Recruiter updates a Job_Description, THEN THE System SHALL regenerate embeddings and update the Vector_Database
4. THE Job_Description SHALL include required fields for role title, technical skills, soft skills, and experience level
5. WHEN a Job_Description is deleted, THEN THE System SHALL remove associated embeddings from the Vector_Database

### Requirement 4: Multi-Agent Orchestration

**User Story:** As a System Administrator, I want AI agents assigned specific evaluation roles, so that candidate assessment covers multiple perspectives.

#### Acceptance Criteria

1. WHEN a Debate_Session starts, THEN THE System SHALL instantiate at least three AI_Agents with distinct roles
2. THE System SHALL assign roles including Code Reviewer, Soft Skills Evaluator, and Cultural Fit Assessor
3. WHEN AI_Agents are instantiated, THEN THE System SHALL provide each agent with the relevant Job_Description context from the Vector_Database
4. THE System SHALL execute AI_Agent analysis tasks in parallel using Python asyncio
5. WHEN all AI_Agents complete initial analysis, THEN THE System SHALL initiate the debate phase

### Requirement 5: AI Agent Debate Protocol

**User Story:** As a Recruiter, I want AI agents to debate candidate qualifications, so that I can see balanced reasoning from multiple perspectives.

#### Acceptance Criteria

1. WHEN the debate phase begins, THEN THE System SHALL allow each AI_Agent to present initial findings
2. WHEN an AI_Agent presents findings, THEN THE System SHALL enable other AI_Agents to challenge or support those findings
3. THE System SHALL limit debate rounds to a maximum of five exchanges per agent
4. WHEN AI_Agents disagree, THEN THE System SHALL require each agent to provide specific evidence from Candidate_Assets
5. THE System SHALL record all AI_Agent statements in the Transcript with timestamps and agent identifiers

### Requirement 6: Consensus Generation

**User Story:** As a Recruiter, I want a synthesized evaluation report, so that I can understand the collective AI recommendation.

#### Acceptance Criteria

1. WHEN the debate concludes, THEN THE System SHALL generate a Consensus_Report summarizing all AI_Agent perspectives
2. THE Consensus_Report SHALL include distinct sections for strengths, weaknesses, and overall recommendation
3. WHEN generating the Consensus_Report, THEN THE System SHALL cite specific evidence from the Transcript
4. THE Consensus_Report SHALL include a confidence score between 0 and 100
5. WHEN AI_Agents reach unanimous agreement, THEN THE System SHALL flag the decision as high-confidence

### Requirement 7: Evaluation Persistence

**User Story:** As a Recruiter, I want evaluation history stored, so that I can review past decisions and audit the process.

#### Acceptance Criteria

1. WHEN a Debate_Session completes, THEN THE System SHALL store the complete Transcript in Amazon RDS
2. WHEN a Consensus_Report is generated, THEN THE System SHALL persist it with a reference to the Candidate and Job_Description
3. THE System SHALL maintain associations between Candidate_Assets, Transcripts, and Consensus_Reports
4. WHEN a Recruiter queries evaluation history, THEN THE System SHALL retrieve all related records within 2 seconds
5. THE System SHALL retain evaluation data for a minimum of 2 years

### Requirement 8: Dashboard Visualization

**User Story:** As a Recruiter, I want to view live AI debates and final reports, so that I can understand the reasoning behind recommendations.

#### Acceptance Criteria

1. WHEN a Debate_Session is active, THEN THE Dashboard SHALL display real-time AI_Agent statements as they occur
2. THE Dashboard SHALL visually distinguish different AI_Agents using unique colors and icons
3. WHEN displaying the Consensus_Report, THEN THE Dashboard SHALL present strengths and weaknesses in separate sections
4. THE Dashboard SHALL provide a timeline view showing the progression of the debate
5. WHEN a Recruiter selects an AI_Agent statement, THEN THE Dashboard SHALL highlight the supporting evidence from Candidate_Assets

### Requirement 9: Code Repository Analysis

**User Story:** As a Recruiter, I want AI agents to analyze candidate code repositories, so that technical skills can be objectively evaluated.

#### Acceptance Criteria

1. WHEN a GitHub URL is provided, THEN THE System SHALL fetch repository metadata including language distribution and commit history
2. WHEN analyzing code, THEN THE Code Reviewer AI_Agent SHALL evaluate code quality, documentation, and testing practices
3. THE System SHALL support analysis of repositories in Python, JavaScript, TypeScript, Java, and Go
4. IF a repository is private or inaccessible, THEN THE System SHALL notify the Recruiter and skip code analysis
5. WHEN code analysis completes, THEN THE System SHALL include specific file examples in the AI_Agent findings

### Requirement 10: Human Override and Final Decision

**User Story:** As a Recruiter, I want to make the final hiring decision, so that human judgment remains the ultimate authority.

#### Acceptance Criteria

1. WHEN viewing a Consensus_Report, THEN THE Dashboard SHALL provide options to Accept, Reject, or Request Re-evaluation
2. WHEN a Recruiter makes a final decision, THEN THE System SHALL record the decision with timestamp and Recruiter identifier
3. IF a Recruiter decision contradicts the AI recommendation, THEN THE System SHALL prompt for justification notes
4. THE System SHALL allow Recruiters to add comments to any Consensus_Report
5. WHEN a final decision is recorded, THEN THE System SHALL update the Candidate status in Amazon RDS

### Requirement 11: API Request Handling

**User Story:** As a System Administrator, I want the backend to handle concurrent requests efficiently, so that multiple recruiters can use the platform simultaneously.

#### Acceptance Criteria

1. THE FastAPI backend SHALL support at least 50 concurrent API requests without degradation
2. WHEN multiple Debate_Sessions run simultaneously, THEN THE System SHALL isolate each session's AI_Agent instances
3. THE System SHALL implement request rate limiting of 100 requests per minute per Recruiter
4. WHEN API errors occur, THEN THE System SHALL return structured error responses with HTTP status codes
5. THE System SHALL log all API requests with timestamps, endpoints, and response times

### Requirement 12: Security and Access Control

**User Story:** As a System Administrator, I want candidate data protected, so that privacy regulations are satisfied.

#### Acceptance Criteria

1. THE System SHALL encrypt all Candidate_Assets at rest in Amazon S3 using AES-256
2. THE System SHALL encrypt all data in transit using TLS 1.3
3. WHEN a Recruiter authenticates, THEN THE System SHALL verify credentials and issue a time-limited session token
4. THE System SHALL restrict access to Candidate data based on Recruiter permissions
5. WHEN a Candidate requests data deletion, THEN THE System SHALL remove all associated records within 30 days
