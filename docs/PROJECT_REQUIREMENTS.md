# EKA Project Requirements

## Project Overview

EKA (Evolving Kognitive Assistant) is an AI-powered mission support system designed for astronauts conducting experiments in space-station and ground-based lab environments. The system provides real-time guidance, monitors procedure compliance, detects safety violations, and answers mission-specific questions.

**Development Context**:
- 6-member student team (B.Tech CSE/AI-ML, 2nd/3rd year)
- Limited development time (~9 weeks)
- Production-style architecture with sustainable practices
- Focus on core functionality over feature breadth

---

## Functional Requirements

### FR1: Vision-Based Object Detection

**Requirement**: The system shall detect astronauts, equipment, and samples in real-time video.

- **FR1.1**: Detect astronauts with >90% accuracy at 15+ FPS
- **FR1.2**: Detect common experiment equipment (beakers, balances, chambers, etc.)
- **FR1.3**: Provide bounding boxes and class labels
- **FR1.4**: Support video input from helmet-mounted or station cameras
- **FR1.5**: Output detection results in standard JSON format (see API_CONTRACTS.md)

**Acceptance Criteria**:
- Vision module correctly identifies 10+ object classes
- Latency <100ms per frame on target hardware
- Confidence scores provided for each detection
- API contract compliance

---

### FR2: Human Activity Recognition

**Requirement**: The system shall recognize human activities from video to understand procedural progress.

- **FR2.1**: Recognize 10+ activity classes (placing_sample, securing_chamber, measuring, etc.)
- **FR2.2**: Achieve >85% accuracy on common astronaut actions
- **FR2.3**: Track activity duration
- **FR2.4**: Handle partial occlusion gracefully
- **FR2.5**: Output activity recognition results in standard JSON format

**Acceptance Criteria**:
- Activity model recognizes all defined classes
- Inference latency <200ms
- Works with helmet-mounted camera perspective
- Includes confidence scores and alternatives

---

### FR3: Experiment Procedure Tracking

**Requirement**: The system shall track multi-step experiment procedures and current progress.

- **FR3.1**: Parse experiment SOPs (Standard Operating Procedures)
- **FR3.2**: Track current step and percentage completion
- **FR3.3**: Associate expected activities with each step
- **FR3.4**: Maintain complete procedure timeline
- **FR3.5**: Output state in standard JSON format

**Acceptance Criteria**:
- Can load and track a 10+ step experiment
- Correctly reports current step and progress
- Provides expected activities for context
- State persists across session restarts

---

### FR4: Safety Deviation Detection

**Requirement**: The system shall detect procedure deviations and safety violations in real-time.

- **FR4.1**: Compare observed activities against expected procedure
- **FR4.2**: Detect critical safety violations (e.g., missing chamber lock before reaction)
- **FR4.3**: Generate alerts with severity levels (info, warning, critical)
- **FR4.4**: Provide recommendations for corrective action
- **FR4.5**: Use deterministic rules, not AI predictions, for safety-critical decisions

**Acceptance Criteria**:
- Detects pre-defined deviations with 100% recall on test cases
- Severity levels are consistent and meaningful
- Recommendations are actionable
- No false positives on normal procedure execution
- Safety engine is independent of LLM inference

---

### FR5: Knowledge-Grounded Q&A

**Requirement**: The system shall answer astronaut questions using only approved mission documents.

- **FR5.1**: Answer questions about procedures, equipment, and safety
- **FR5.2**: Retrieve answers from approved mission/experiment documents only
- **FR5.3**: Provide source attribution (document, page, section)
- **FR5.4**: Include confidence scores
- **FR5.5**: Reject out-of-scope queries gracefully

**Acceptance Criteria**:
- Answers are always sourced from approved documents
- Sources are properly cited with locations
- Confidence scores reflect answer quality
- Out-of-scope queries are rejected with explanation
- System works offline with preloaded documents
- Latency <2 seconds for typical queries

---

### FR6: User Authentication and Authorization

**Requirement**: The system shall authenticate users and enforce role-based access control.

- **FR6.1**: Support astronaut and mission-control roles
- **FR6.2**: Require login for all API endpoints
- **FR6.3**: Issue JWT tokens with expiration
- **FR6.4**: Enforce role-based access to sensitive data
- **FR6.5**: Log all authentication events

**Acceptance Criteria**:
- Users cannot access API without valid token
- Tokens expire after configured interval
- Roles control access to experiment data and safety logs
- Login attempts are logged

---

### FR7: Comprehensive Audit Logging

**Requirement**: The system shall maintain complete audit trail of all decisions and actions.

- **FR7.1**: Log all module outputs with timestamps
- **FR7.2**: Log all safety decisions and alerts
- **FR7.3**: Log all knowledge queries and answers
- **FR7.4**: Log all user actions and API calls
- **FR7.5**: Make audit logs queryable and exportable

**Acceptance Criteria**:
- Audit logs include source, timestamp, action, and result
- Logs are immutable and cannot be deleted
- Query tools allow filtering by time, user, module
- Export supports CSV and JSON formats

---

### FR8: Real-Time Dashboard

**Requirement**: The system shall display mission state and experiment progress to astronaut and mission control.

- **FR8.1**: Display detected objects and activities
- **FR8.2**: Display current experiment step and progress
- **FR8.3**: Display active alerts and warnings
- **FR8.4**: Display Q&A interface
- **FR8.5**: Update in real-time (WebSocket)
- **FR8.6**: Support responsive design (tablet/small screen)

**Acceptance Criteria**:
- Dashboard updates within 1 second of state change
- All core information is visible on standard tablet screen
- Alerts are prominently displayed
- Q&A interface is intuitive and responsive

---

### FR9: Mission Event Timeline

**Requirement**: The system shall maintain queryable timeline of all mission events.

- **FR9.1**: Record activity recognitions, detections, alerts
- **FR9.2**: Support time-range queries
- **FR9.3**: Support filtering by event type or severity
- **FR9.4**: Export timeline for post-mission analysis

**Acceptance Criteria**:
- Timeline events are recorded with millisecond precision
- Queries return results in <500ms
- Timeline can be exported in standard formats
- Events are cross-referenced with experiment steps

---

## Non-Functional Requirements

### NFR1: Performance

- **NFR1.1**: Vision inference: <100ms per frame
- **NFR1.2**: Activity recognition: <200ms per activity
- **NFR1.3**: Safety engine: <50ms per check
- **NFR1.4**: Knowledge query: <2 seconds
- **NFR1.5**: Dashboard updates: <1 second latency

### NFR2: Reliability

- **NFR2.1**: System uptime >99% during missions
- **NFR2.2**: All critical safety checks fail safely (default: "stop")
- **NFR2.3**: Database transactions are ACID-compliant
- **NFR2.4**: Failed module gracefully degrades system (doesn't crash entire system)

### NFR3: Security

- **NFR3.1**: All data in transit is encrypted (TLS 1.2+)
- **NFR3.2**: All data at rest is encrypted
- **NFR3.3**: API endpoints require authentication
- **NFR3.4**: No hardcoded credentials in code
- **NFR3.5**: Regular security audit logs

### NFR4: Scalability

- **NFR4.1**: Support up to 10 concurrent astronauts
- **NFR4.2**: Support concurrent experiment tracking
- **NFR4.3**: Database can handle 1000+ events per mission

### NFR5: Offline Capability

- **NFR5.1**: Core modules work without internet
- **NFR5.2**: Knowledge base is preloaded on device
- **NFR5.3**: Experiment definitions are cached locally
- **NFR5.4**: Sync to mission control when connection restored

### NFR6: Maintainability

- **NFR6.1**: All modules have comprehensive documentation
- **NFR6.2**: Code follows PEP 8 (Python) standards
- **NFR6.3**: >80% test coverage for critical modules
- **NFR6.4**: Models are version-controlled and reproducible
- **NFR6.5**: Dependencies are pinned and documented

---

## Out of Scope (For This Release)

- Multi-language support (English only)
- Mobile app (web dashboard only)
- Computer-vision-based pose estimation (use activity recognition instead)
- Autonomous robotic control
- Integration with actual spacecraft systems
- Voice command processing (voice transcription only)
- Real-time video compression or optimization

---

## Test Strategy

### Unit Tests
- Each module has >80% line coverage
- API contracts are validated with example data
- Safety rules are exhaustively tested

### Integration Tests
- Modules communicate correctly via API contracts
- End-to-end flow: video → vision → activity → safety → alert
- Dashboard receives updates correctly

### System Tests
- Complete experiment scenario (demo scenario, see DEMO_SCENARIO.md)
- Safety violations are detected correctly
- Knowledge queries return expected answers

### Performance Tests
- Latency measurements for all critical paths
- Load testing for dashboard with concurrent updates
- Database query performance

---

## Acceptance Criteria Checklist

- [ ] All 8 FRs have working implementations
- [ ] All NFRs are measured and documented
- [ ] Module API contracts are validated
- [ ] Demo scenario runs successfully end-to-end
- [ ] Documentation is current and complete
- [ ] All critical paths have >80% test coverage
- [ ] Security audit passed
- [ ] Performance benchmarks met

---

## Development Roadmap

### Week 1-2: Foundation
- [ ] Backend skeleton with FastAPI
- [ ] Database schema and migrations
- [ ] Frontend skeleton (React)
- [ ] Module stubs with mock data

### Week 3-4: Vision & Activity
- [ ] Vision engine implementation
- [ ] Activity recognition model training/setup
- [ ] Integration with backend
- [ ] Dashboard visualization

### Week 5-6: Experiment & Safety
- [ ] Experiment engine (SOP parsing, state tracking)
- [ ] Safety rules engine
- [ ] Integration with vision/activity outputs
- [ ] Alert system

### Week 7-8: Knowledge & Polish
- [ ] Knowledge assistant (RAG setup)
- [ ] Q&A interface
- [ ] Comprehensive testing
- [ ] Security hardening

### Week 9+: Deployment & Demo
- [ ] Docker containerization
- [ ] Performance tuning
- [ ] Demo scenario validation
- [ ] Final documentation
- [ ] Team presentation

---

## Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Vision accuracy | >90% | Test on 100+ frames |
| Activity accuracy | >85% | Test on 50+ activity instances |
| Safety detection recall | 100% | Test against 20+ deviation scenarios |
| Knowledge answer relevance | >90% | Manual review of 20+ queries |
| System uptime | >99% | Track during demo |
| API latency | <500ms | Measure p95 response time |
| Test coverage | >80% | Code coverage tools |
| Documentation | 100% | All modules have README + API doc |

---

## Constraints & Assumptions

### Constraints
- Small team (6 developers), limited time (9 weeks)
- No access to real spacewear or experiment hardware
- Limited computational resources (must run on modest hardware)
- Must use open-source or affordable tools (no enterprise licenses)

### Assumptions
- Team members are familiar with Python, Git, REST APIs
- Pretrained models can be downloaded/installed reliably
- Test data (video, documents) will be provided
- No production spacecraft integration required
- Demo is offline (no real-time connection to actual astronauts)

---

## Change Control

Requirements can be modified by team consensus and documented in this file with a change log entry.

### Change Log
- v1.0 (2024-01-15): Initial requirements document
