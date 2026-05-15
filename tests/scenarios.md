# Test Scenarios: Foundation-Learning-AI-Agent

## Overview

These scenarios test the core functionality of the Foundation-Learning-AI-Agent framework across context engineering, harness engineering principles, and agent coordination. Scenarios are organized by difficulty: 4 Easy, 3 Medium, 3 Hard.

---

# EASY SCENARIOS (4)

## Scenario E1: Agent Context Loading - Minimal Configuration

**Category**: Agent Context Analysis | Difficulty: Easy

### Input Spec

**Setup**:

- Project initialized with `.specify/init-options.json` containing `"branch_numbering": "sequential"`
- `.specify/extensions.yml` exists but has no hooks defined
- User provides feature description: "Add user authentication with OAuth2"
- No `.specify/feature.json` exists yet

**Action**:

- Trigger: `/speckit.specify` command with feature description
- Agent must: Load init-options, detect no hooks, create feature directory

### Expected Output

```
✓ No extension hooks detected (expected)
✓ Sequential numbering enabled
✓ Generated short name: "oauth2-user-auth"
✓ Created feature directory: specs/001-oauth2-user-auth/
✓ Created spec.md from template
✓ Saved feature.json with resolved path: "specs/001-oauth2-user-auth"
✓ Specification generation complete
```

### Rubric

| Criterion                     | Full (3 pts)                            | Partial (1 pt)                         | None (0 pts)                |
| ----------------------------- | --------------------------------------- | -------------------------------------- | --------------------------- |
| **Init Options Loaded**       | Correctly reads sequential numbering    | Reads but ignores numbering            | Fails to load config        |
| **Feature Directory Created** | Correct path with sequential prefix     | Directory created but naming incorrect | Directory creation fails    |
| **Template Applied**          | spec.md contains all template sections  | spec.md exists but incomplete          | No spec.md created          |
| **feature.json Persisted**    | Valid JSON with correct path            | JSON created but wrong format          | No feature.json saved       |
| **No Hook Execution**         | Correctly skips hooks (logs "no hooks") | Skips hooks but no log                 | Attempts non-existent hooks |

**Total**: 15 points | **Pass Threshold**: 12 points (80%)

---

## Scenario E2: Shared Understanding Detection - Team Alignment

**Category**: Harness Engineering | Difficulty: Easy

### Input Spec

**Data**: Team working agreement document containing:

```
Decision-making:
- Small decisions (<1 hour): Anyone decides + inform team
- Medium decisions (>1 hour): Discussion + consensus
- Large decisions: Full team + alignment

Definition of Done:
- Code reviewed
- Tests passing (80%+ coverage)
- Integrated with main branch
- Deployed to staging
```

**Action**:

- Analyze: Extract shared understanding elements from working agreement
- Evaluate: Does team have explicit clarity on (a) how decisions are made, (b) what done means?

### Expected Output

```
✓ Shared Understanding Assessment
  - Decision-making clarity: EXPLICIT (3 decision types defined)
  - Definition of done: EXPLICIT (4 criteria listed)
  - Team alignment: LIKELY (documented and specific)

✓ Findings:
  - Team has clear decision process (small/medium/large categorized)
  - Done criteria measurable (code review, test coverage, deployment status)
  - No ambiguous adjectives (clear thresholds provided)

✓ Risk Assessment: LOW - Shared understanding evident
```

### Rubric

| Criterion                       | Full                            | Partial               | None                |
| ------------------------------- | ------------------------------- | --------------------- | ------------------- |
| **Decision Process Identified** | All 3+ types found with clarity | 1-2 types found       | No decision process |
| **Done Definition Found**       | 4+ criteria listed explicitly   | 2-3 criteria          | No criteria listed  |
| **Clarity Assessment**          | All elements scored EXPLICIT    | 1-2 elements explicit | Vague/implicit      |
| **Risk Classification**         | Correct LOW/MEDIUM/HIGH         | Wrong classification  | No assessment       |
| **Evidence Quoted**             | Direct excerpts provided        | Paraphrased           | No evidence         |

**Total**: 15 points | **Pass Threshold**: 12 points (80%)

---

## Scenario E3: Hook Execution Detection - Pre-Flight Setup

**Category**: Agent Context Analysis | Difficulty: Easy

### Input Spec

**Setup**:

- `.specify/extensions.yml` contains:
  ```yaml
  hooks:
    before_specify:
      - name: "git-feature-branch"
        enabled: true
        optional: false
        command: "git-feature"
        description: "Create feature branch"
  ```
- Git branch naming script exists
- No explicit user branch name provided

**Action**:

- Trigger: `/speckit.specify` command
- Agent must: Detect before_specify hook, execute git-feature branch creation

### Expected Output

```
✓ Pre-Execution Hooks Check
  - Extension: git-feature-branch
  - Enabled: true
  - Mandatory Hook: EXECUTING

✓ Hook Execution: /git-feature
  - Branch created: feature/001-oauth2-user-auth
  - Output: BRANCH_NAME="feature/001-oauth2-user-auth"

✓ Hook parsed successfully
✓ Proceeding to specification creation
```

### Rubric

| Criterion          | Full                               | Partial                        | None                |
| ------------------ | ---------------------------------- | ------------------------------ | ------------------- |
| **Hook Detection** | Extensions.yml read and parsed     | File read but parse incomplete | File not found/read |
| **Enabled Filter** | Correctly identifies enabled:true  | Filter skipped                 | Filter broken       |
| **Hook Execution** | Script runs and produces output    | Script runs but output lost    | Script not executed |
| **Output Parsing** | BRANCH_NAME extracted correctly    | Extraction attempted but wrong | No extraction       |
| **Continuation**   | Proceeds to next phase with output | Proceeds but loses context     | Halts unexpectedly  |

**Total**: 15 points | **Pass Threshold**: 12 points (80%)

---

## Scenario E4: Psychological Safety Indicator - Feedback Quality

**Category**: Harness Engineering | Difficulty: Easy

### Input Spec

**Data**: Code review comment thread:

```
Reviewer: "I noticed the error handling here doesn't cover async failures.
           Could we catch those too? I found [link] helpful for this pattern."

Author: "Good catch! I wasn't sure about async error handling.
         Thanks for the resource—I'll update this."

Reviewer: "Great! Let me know if you have questions while implementing."
```

**Action**:

- Analyze: Evaluate comment thread for psychological safety indicators
- Assess: Is feedback delivered with respect? Is receiver open to input?

### Expected Output

```
✓ Psychological Safety Indicators

  Positive Signals:
  - Feedback framed as observation, not criticism (I noticed... could we...)
  - Resource provided (learning enabled)
  - Receiver accepts input without defensiveness (Good catch!)
  - Reviewer offers continued support (let me know...)
  - Growth mindset visible (wasn't sure → willing to learn)

✓ Safety Assessment: HIGH
  - Respectful tone throughout
  - Collaborative problem-solving
  - Learning oriented
  - No blame or judgment language

✓ Risk Assessment: LOW - Safety indicators strong
```

### Rubric

| Criterion                 | Full                                | Partial                         | None                     |
| ------------------------- | ----------------------------------- | ------------------------------- | ------------------------ |
| **Tone Analysis**         | Respectful/collaborative identified | Some positive tone              | Dismissive/critical tone |
| **Feedback Frame**        | Observational, not judgmental       | Mixed framing                   | Purely judgmental        |
| **Receiver Response**     | Open/learning-oriented identified   | Neutral acceptance              | Defensive/resistant      |
| **Support Offered**       | Reviewer offers continued help      | Help implied                    | No support signal        |
| **Safety Score Assigned** | HIGH/MEDIUM/LOW justified           | Assigned but weak justification | Not assigned             |

**Total**: 15 points | **Pass Threshold**: 12 points (80%)

---

# MEDIUM SCENARIOS (3)

## Scenario M1: Context Flow - Specification to Planning

**Category**: Agent Context Analysis | Difficulty: Medium

### Input Spec

**Setup**:

- `specs/001-oauth2-user-auth/spec.md` exists with:
  - Functional Requirements (FR-001 through FR-005)
  - Success Criteria (SC-001: "Authentication completes in <200ms")
  - User Stories with priorities (US1: P1, US2: P2, US3: P2)
  - Assumed tech stack not specified (NEEDS CLARIFICATION)

- `.specify/memory/constitution.md` contains:
  - "MUST: All security decisions documented"
  - "SHOULD: Performance measured in production"

- User runs: `/speckit.plan`

**Action**:

- Agent must: Load spec.md, check constitution, generate plan.md, update copilot-instructions.md
- Verify: Context flows correctly from spec to plan; constitution constraints observed

### Expected Output

```
✓ Setup Completed
  - FEATURE_DIR: specs/001-oauth2-user-auth
  - SPEC_FILE: specs/001-oauth2-user-auth/spec.md

✓ Specification Loaded
  - Requirements: 5 (FR-001 to FR-005)
  - Success Criteria: 1 (SC-001)
  - User Stories: 3 (priority: P1, P2, P2)
  - NEEDS CLARIFICATION: 1 (tech stack)

✓ Constitution Check
  - MUST: Security documentation → Planned in Phase 1
  - SHOULD: Production metrics → Planned in Phase 2

✓ Phase 0 Generated: research.md
  - NEEDS CLARIFICATION resolved (tech stack: Spring Boot + OAuth2 lib)
  - Decision rationale documented

✓ Phase 1 Generated: plan.md
  - data-model.md with User, Token, Session entities
  - Architecture diagram (OAuth2 flow)
  - Security documentation (per constitution MUST)

✓ Agent Context Updated
  - .github/copilot-instructions.md points to plan.md

✓ Planning complete - Ready for tasks generation
```

### Rubric

| Criterion                   | Full (5 pts)                                          | Partial (2-3 pts)                       | None (0 pts)              |
| --------------------------- | ----------------------------------------------------- | --------------------------------------- | ------------------------- |
| **Setup Execution**         | Correct FEATURE_DIR, SPEC_FILE resolved               | Paths resolved but incomplete           | Setup fails               |
| **Spec Loading**            | All sections extracted (FR, SC, US, CLS)              | 2-3 sections extracted                  | No spec loaded            |
| **Constitution Validation** | Both MUST/SHOULD constraints identified and addressed | 1 constraint identified                 | Constraints ignored       |
| **Research Phase**          | NEEDS CLARIFICATION resolved with rationale           | Clarification resolved but no rationale | Unresolved clarifications |
| **Plan Generation**         | plan.md + data-model.md generated correctly           | 1 artifact generated                    | No artifacts              |
| **Context Update**          | copilot-instructions.md correctly updated             | Updated but wrong path                  | Not updated               |
| **Output Accuracy**         | All phase outputs match spec constraints              | 1-2 outputs incorrect                   | Major mismatches          |

**Total**: 35 points | **Pass Threshold**: 28 points (80%)

---

## Scenario M2: Constraint Detection - Team Bottleneck Analysis

**Category**: Harness Engineering | Difficulty: Medium

### Input Spec

**Data**: Team metrics over 4 sprints:

```
Sprint 1:
- Velocity: 34 points
- Cycle Time: 4.2 days
- Code review time: 6 hours avg
- Build time: 45 minutes
- QA wait: 2 hours

Sprint 2:
- Velocity: 28 points (↓ 18%)
- Cycle Time: 6.1 days (↑ 45%)
- Code review time: 8 hours avg (↑)
- Build time: 42 minutes
- QA wait: 3 hours (↑)

Sprint 3:
- Velocity: 22 points (↓ 21%)
- Cycle Time: 7.8 days (↑ 28%)
- Code review time: 12 hours avg (↑ 50%)
- Build time: 44 minutes
- QA wait: 4 hours (↑)

Sprint 4:
- Velocity: 19 points (↓ 14%)
- Cycle Time: 9.3 days (↑ 19%)
- Code review time: 15 hours avg (↑ 25%)
- Build time: 45 minutes
- QA wait: 5 hours (↑)
```

**Action**:

- Analyze: Identify primary constraint limiting throughput
- Apply TOC: Propose exploitation strategy without adding capacity
- Verify: Hypothesis explains velocity drop and cycle time increase

### Expected Output

```
✓ Constraint Analysis

Primary Constraint Identified: CODE REVIEW BOTTLENECK
- Evidence:
  - Code review time increased 150% (6h → 15h) over 4 sprints
  - Velocity declined 44% correlating with review time increase
  - Cycle time increased 121% (4.2d → 9.3d)
  - Build time stable (42-45 min) - not the constraint
  - QA wait time grew but secondary (more work piles up waiting review)

Root Cause Analysis:
- Hypothesis: Code review capacity hasn't scaled with story complexity
- Evidence: Review time per story increasing (time/point growing)
- Likely cause: More senior reviewers needed for OAuth2 complexity

TOC: Exploitation Strategy (before adding capacity):
1. Implement async code review (don't block on single reviewer)
2. Establish review time SLA (respond within 2 hours during business hours)
3. Pair junior developers with reviewers for knowledge transfer + faster review
4. Use automated linting/style checking (reduce review scope)

Expected Impact if Exploited:
- Reduce code review time to 6-8 hours (back to Sprint 1 baseline)
- Velocity should recover to 30+ points
- Cycle time should drop to 5-6 days

Escalation Decision:
- Monitor next sprint after exploitation
- If still constrained → Add capacity (hire/train code reviewers)
- If improved → Find next constraint (likely QA wait)
```

### Rubric

| Criterion                           | Full (5 pts)                                                    | Partial (2-3 pts)                    | None (0 pts)             |
| ----------------------------------- | --------------------------------------------------------------- | ------------------------------------ | ------------------------ |
| **Constraint Correctly Identified** | Code review clearly primary (150% growth, velocity correlation) | Identified but weak evidence         | Wrong constraint or none |
| **Root Cause Analysis**             | Hypothesis stated with supporting data                          | Cause stated but incomplete data     | No analysis              |
| **Exploitation Strategy**           | 3+ concrete actions (no capacity add)                           | 1-2 actions or includes capacity add | No actionable strategy   |
| **Impact Projection**               | Clear success metrics (review time, velocity targets)           | Metrics listed but vague             | No success metrics       |
| **Escalation Path**                 | Clear conditions for elevation decision                         | Mentioned but unclear                | No escalation logic      |
| **Evidence Quality**                | Direct metric quotes supporting conclusion                      | Paraphrased or inferred              | No evidence cited        |

**Total**: 30 points | **Pass Threshold**: 24 points (80%)

---

## Scenario M3: Feedback Loop Closure - Retrospective to Action

**Category**: Harness Engineering | Difficulty: Medium

### Input Spec

**Sprint 4 Retrospective Data**:

```
What Went Well:
- Daily standups helped identify blockers early
- Pair programming on complex features worked well

What Didn't Go Well:
- Code reviews taking too long (12-15 hours)
- Team felt rushed toward end of sprint
- New team member onboarding delayed (knowledge gaps)

Proposed Changes:
1. Rotate code reviewers (not just senior devs)
2. Implement 2-hour code review SLA
3. Establish 30-min daily pairing session for knowledge transfer
```

**Post-Sprint Action**: Team implements changes. Sprint 5 data shows:

```
- Code review time: 7.5 hours avg (down 50%)
- Velocity: 31 points (up 63% from Sprint 4's 19)
- New hire confidence rating: 8/10 (up from 4/10)
- Team stress level: 6/10 (down from 8/10)
- Cycle time: 5.2 days (back to Sprint 1 baseline)
```

**Action**:

- Evaluate: Was feedback loop closed? Did actions produce results?
- Verify: Causation (not coincidence) between actions and outcomes
- Assess: Team learning from cycle

### Expected Output

```
✓ Feedback Loop Closure Assessment

Retrospective → Action Implementation: SUCCESS
  - All 3 proposed changes implemented
  - Changes executed in Sprint 5 as planned

Action Impact Analysis:

Change 1: Rotate Code Reviewers
- Expected: Distribute review load, reduce bottleneck
- Observed: Code review time 50% lower (7.5h vs 15h)
- Confidence: HIGH (direct correlation to primary change)
- Unintended Consequences: None detected

Change 2: 2-Hour Code Review SLA
- Expected: Faster response, less blocking
- Observed: Cycle time dropped to 5.2 days (44% reduction)
- Confidence: MEDIUM-HIGH (part of reviewer rotation changes)
- Evidence: Work no longer piles up waiting reviews

Change 3: Daily 30-Min Pairing Session
- Expected: New hire faster ramp, team learning
- Observed: New hire confidence 4/10 → 8/10 (100% improvement)
- Confidence: HIGH (direct correlation to knowledge transfer)
- Unintended Consequences: Time investment worth return

Overall Loop Closure: SUCCESSFUL
- 3/3 proposed changes produced measurable improvements
- Multiple metric improvements (code review, velocity, stress, confidence)
- Improvements sustain without additional capacity (no new hires)
- Team experienced success from learning cycle

Team Learning: EVIDENT
- Team identified correct constraint (code review)
- Team designed exploitation strategy (reviewer rotation)
- Team implemented deliberately and measured
- Team now understands their system (how review capacity affects velocity)
- Next cycle: Likely to identify QA wait as next constraint

Recommendation: Continue retrospective discipline, implement same feedback loop process
```

### Rubric

| Criterion                   | Full (5 pts)                                                   | Partial (2-3 pts)                       | None (0 pts)                     |
| --------------------------- | -------------------------------------------------------------- | --------------------------------------- | -------------------------------- |
| **Loop Closure Verified**   | All 3 changes implemented; outcomes measured                   | 1-2 changes implemented or partial data | No implementation or measurement |
| **Causation Analysis**      | Changes linked to outcomes with confidence level               | Links stated but weak analysis          | No cause-effect analysis         |
| **Impact Quantified**       | Specific % improvements cited (50%, 44%, 100%)                 | Improvements noted but not quantified   | No impact data                   |
| **Unintended Consequences** | Explicitly checked and assessed                                | Checked but incompletely                | Not considered                   |
| **Team Learning Assessed**  | Learning demonstrated through future constraint identification | Learning claimed but not evidenced      | No learning assessment           |
| **Recommendation Provided** | Clear next-step guidance based on learning                     | Generic recommendation                  | No recommendation                |

**Total**: 30 points | **Pass Threshold**: 24 points (80%)

---

# HARD SCENARIOS (3)

## Scenario H1: Multi-Artifact Consistency Validation

**Category**: Agent Context Analysis | Difficulty: Hard

### Input Spec

**Three Artifacts Exist**:

**spec.md** contains:

- FR-001: "User can authenticate with OAuth2"
- FR-002: "Session remains valid for 24 hours"
- FR-003: "Failed login locked for 5 minutes"
- SC-001: "Authentication completes in <200ms"
- SC-002: "Session refresh in <100ms"
- US1: "User logs in with OAuth2" [P1]
- US2: "User receives session token" [P1]
- US3: "System enforces rate limiting on failed logins" [P2]

**plan.md** contains:

- Phase 1: OAuth2 integration (tech: Spring Boot, lib: Spring Security OAuth2)
- Phase 2: Session management (Redis for token storage)
- Phase 3: Rate limiting (not assigned to any phase initially)
- Data Model: User, OAuth2Token, Session, RateLimit entities
- No mention of 24-hour session validity requirement

**tasks.md** contains:

- T001-T010: OAuth2 integration tasks (Phase 1)
- T011-T020: Session management tasks (Phase 2)
- No tasks for rate limiting
- T015 references "RateLimit entity" but rate limiting not assigned phase in plan
- SC-001 (auth <200ms) has no performance test task
- SC-002 (refresh <100ms) has no test task

**constitution.md** contains:

- "MUST: All security decisions documented"
- "MUST: Performance requirements have test coverage"
- "SHOULD: Data retention decisions explicit"

**Action**:

- Run: `/speckit.analyze`
- Detect: Inconsistencies, gaps, constitution violations
- Classify severity and recommend resolution order

### Expected Output

```
✓ Analysis Initialized
  - spec.md loaded: 6 requirements, 2 criteria, 3 stories
  - plan.md loaded: 3 phases, 4 entities
  - tasks.md loaded: 20 tasks, 2 story phases assigned
  - constitution.md loaded: 3 principles (2 MUST, 1 SHOULD)

✓ CONSISTENCY ANALYSIS

═══════════════════════════════════════════════════════════

Findings: 7 issues identified

─────────────────────────────────────────────────────────

[C1] CRITICAL | Coverage Gap
  Location: spec.md:US3 vs. tasks.md
  Issue: US3 "Rate limiting" in spec but zero tasks assigned
  Impact: Feature incomplete; blockers phase assignment
  Recommendation: Add 5-8 tasks to plan.md Phase 3, then tasks.md

[C2] CRITICAL | Constitution Violation - MUST
  Location: plan.md, constitution.md
  Issue: "MUST: All security decisions documented" violated
         Rate limiting design not in plan; no decision record
  Impact: Violates project principle; audit/compliance risk
  Recommendation: Add rate limiting strategy to plan.md before tasks

[C3] HIGH | Constitution Violation - MUST
  Location: spec.md:SC-001/SC-002 vs. tasks.md
  Issue: "MUST: Performance requirements have test coverage"
         SC-001 (<200ms) and SC-002 (<100ms) have zero test tasks
  Impact: Success criteria unverifiable; can't validate spec
  Recommendation: Add performance test tasks T021-T022

[C4] HIGH | Specification Underspecification
  Location: spec.md:FR-002 vs. plan.md
  Issue: "Session remains valid for 24 hours" in spec
         plan.md lists Session entity but no TTL/expiry defined
  Impact: Implementation ambiguous; may not meet spec
  Recommendation: Update data-model.md with session TTL:24h constraint

[C5] HIGH | Task-Requirement Mapping
  Location: tasks.md:T015 vs. plan.md:Phase 3
  Issue: T015 references "RateLimit entity" but rate limiting not phased in plan
  Impact: Task references undefined context; implementation risk
  Recommendation: Complete plan.md phase assignment before T015

[C6] MEDIUM | Terminology Drift
  Location: spec.md "OAuth2" vs. plan.md "Spring Security OAuth2"
  Issue: Spec generic (OAuth2) vs. plan specific (Spring Security OAuth2)
         Acceptable but verify framework choice explicit in constitution
  Impact: Low risk; design decision made, not conflict
  Recommendation: Document why Spring Security OAuth2 chosen (expected in plan)

[C7] MEDIUM | Constitution Coverage Gap - SHOULD
  Location: spec.md:FR-002 vs. constitution.md
  Issue: "SHOULD: Data retention decisions explicit" not addressed
         24-hour session TTL mentioned but retention policy not documented
  Impact: Policy unclear for future maintenance
  Recommendation: Add retention policy section to plan.md (not blocking)

═══════════════════════════════════════════════════════════

Coverage Summary:

| Requirement | Has Task? | Task IDs | Notes |
|---|---|---|---|
| FR-001 (OAuth2) | YES | T001-T010 | Complete |
| FR-002 (Session 24h) | PARTIAL | T011-T020 | TTL not explicit in plan |
| FR-003 (Rate limit lock) | NO | — | Zero tasks; phase unassigned |
| SC-001 (Auth <200ms) | NO | — | No test task |
| SC-002 (Refresh <100ms) | NO | — | No test task |
| US1 (Login flow) | YES | T001-T010 | Complete |
| US2 (Session token) | YES | T011-T020 | Complete |
| US3 (Rate limiting) | NO | — | Unassigned phase |

═════════════════════════════════════════════════════════════

Constitution Alignment:

| Principle | Status | Finding |
|---|---|---|
| MUST: Security decisions documented | VIOLATED | Rate limiting strategy missing from plan |
| MUST: Performance test coverage | VIOLATED | SC-001, SC-002 lack test tasks |
| SHOULD: Data retention explicit | VIOLATED | Session TTL not in retention policy |

═════════════════════════════════════════════════════════════

SEVERITY SUMMARY:
- CRITICAL: 2 (rate limiting unassigned, security decision undocumented)
- HIGH: 3 (performance tests missing, session TTL undefined, task context gap)
- MEDIUM: 2 (terminology drift minimal, retention policy not explicit)

Total Issues: 7
Blockers (must fix before /speckit.implement): 2
Recommended Fixes (before /speckit.implement): 5
Non-Blocking Improvements (post-impl): 2

═════════════════════════════════════════════════════════════

RECOMMENDED RESOLUTION ORDER:
1. Update plan.md: Add Phase 3 (rate limiting) with security strategy doc
2. Update tasks.md: Add T021-T022 (performance test tasks for SC-001/002)
3. Update data-model.md: Add session TTL:24h, lock duration:5min
4. Verify tasks.md: All references resolve to plan phases
5. Optional: Add retention policy section to plan.md (SHOULD)

RECOMMENDATION:
Do NOT proceed to /speckit.implement until CRITICAL issues resolved.
Run /speckit.plan to add Phase 3, then re-run /speckit.analyze to verify.
```

### Rubric

| Criterion                   | Full (6 pts)                                                  | Partial (3-4 pts)                | None (0 pts)           |
| --------------------------- | ------------------------------------------------------------- | -------------------------------- | ---------------------- |
| **Total Issues Found**      | All 7 issues identified                                       | 4-6 issues found                 | <4 issues              |
| **Critical Classification** | 2 CRITICALs correctly identified                              | 1 CRITICAL or misclassified      | <1 CRITICAL            |
| **Constitution Violations** | All 3 violations detected with principle citations            | 1-2 violations detected          | No violations detected |
| **Coverage Gaps Mapped**    | All requirement-task gaps shown in table                      | 2-3 gaps identified              | Gaps not mapped        |
| **Causation Analysis**      | Issues linked to impacts (spec change → implementation risk)  | Impacts listed but not linked    | No causal analysis     |
| **Resolution Order**        | Logical sequence from CRITICAL → HIGH → MEDIUM                | Sequence attempted but illogical | No sequence            |
| **Blocking vs Optional**    | Clearly distinguished (2 blockers, 5 recommended, 2 optional) | Vague prioritization             | No prioritization      |
| **Specificity**             | Line numbers/IDs cited for each issue                         | Issues described generally       | Vague references       |

**Total**: 42 points | **Pass Threshold**: 34 points (80%)

---

## Scenario H2: Constitution Principle Enforcement Across Design Phases

**Category**: Harness Engineering | Difficulty: Hard

### Input Spec

**Constitution Principles** (`.specify/memory/constitution.md`):

```
# Project Constitution

## Security First
- MUST: All authentication mechanisms documented in security model
- MUST: Authorization decisions logged for audit
- SHOULD: Encryption in transit and at rest (except temporary cache)

## Measurable Performance
- MUST: All success criteria include performance targets
- MUST: Performance targets have test automation
- SHOULD: Production metrics monitored in dashboards

## Sustainable Teams
- MUST: No permanent on-call without rotation
- MUST: Incident reviews blameless
- SHOULD: Knowledge documented; no single points of failure

## Transparent Decisions
- MUST: Technical decisions with >1 option have decision rationale
- SHOULD: Architecture decisions recorded in ADRs
```

**Feature Design Process** (3 phases):

**Phase 1 - Spec Generation** (`.speckit.specify` run):

- User requests: "Real-time data sync for collaborative editing"
- Agent generates spec with requirements, no decision rationale yet

**Phase 2 - Planning** (`/speckit.plan` run):

- Agent encounters design choices:
  - Option A: Websockets (real-time, complex, stateful)
  - Option B: Server-Sent Events (simpler, one-way)
  - Option C: Polling (simple, inefficient)
  - Agent picks Option A (Websockets)
- Agent creates plan.md WITHOUT documenting decision rationale

**Phase 3 - Task Generation** (`/speckit.tasks` run):

- Team sees tasks but doesn't know why Websockets chosen
- No decision document → team could second-guess or refactor

**Audit Check** (`/speckit.analyze` run):

- Constitution principle "MUST: Technical decisions with >1 option have decision rationale"
- Verify: Was principle enforced across phases?

**Action**:

- Evaluate: How did each agent handle constitution principles?
- Identify: Where principle was violated and when it should have been caught
- Assess: Agent design for constitution enforcement

### Expected Output

```
✓ Constitution Enforcement Audit

═══════════════════════════════════════════════════════════

Phase 1 Assessment: Spec Generation (`speckit.specify`)

Constitution Check:
✓ Measurable Performance: SUCCESS
  - Requirement "Real-time sync completes in <500ms" specified
  - Principle applied: ✓ Performance target included

✗ Transparent Decisions: N/A (too early for design decisions)
  - Spec should not make design choices yet (correct)
  - Principle not applicable at this phase

═══════════════════════════════════════════════════════════

Phase 2 Assessment: Planning (`speckit.plan`)

Constitution Checks:

✗ TRANSPARENT DECISIONS - VIOLATED
  Issue: Agent chose Websockets but documented NO rationale
  Evidence: plan.md lists "Tech Stack: Websockets" with no "Why?"
  Principle: "MUST: Technical decisions with >1 option have decision rationale"
  Impact: HIGH - Team doesn't understand constraints considered

  What Should Have Happened:
  - Agent identifies decision point (3 options: WS vs SSE vs Polling)
  - Agent documents each option (pros/cons, fits/gaps)
  - Agent documents final choice rationale (e.g., "WS chosen for real-time
    requirement <500ms; SSE/Polling insufficient for sync latency needs")
  - Result: research.md includes decision record

✗ SECURITY FIRST - PARTIALLY VIOLATED
  Issue: Plan includes Websockets auth strategy but not documented
  Evidence: plan.md mentions "JWT auth for Websocket handshake"
            BUT "authentication mechanisms" not in security model section
  Principle: "MUST: All authentication mechanisms documented in security model"
  Impact: MEDIUM - Security mechanism exists but not systematized

  What Should Have Happened:
  - Security model section in plan.md explicitly lists:
    * Mechanism: JWT in Websocket handshake
    * Scope: Session authentication
    * Validation: Signature verified on connection
    * Audit logging: All connection events logged

✗ MEASURABLE PERFORMANCE - VIOLATED
  Issue: SC-001 (<500ms) specified but no performance testing plan
  Evidence: plan.md lists requirement but no test load, latency measurement
  Principle: "MUST: Performance targets have test automation"
  Impact: HIGH - Can't verify compliance

  What Should Have Happened:
  - plan.md includes "Performance Testing Strategy":
    * Load test: 1000 concurrent connections
    * Latency measurement: P99 <500ms
    * Test frequency: Pre-release + post-deploy
    * Infrastructure: Staging replica with production-like data

═══════════════════════════════════════════════════════════

Phase 3 Assessment: Tasks Generation (`speckit.tasks`)

Constitution Checks:

✗ DECISION RATIONALE INHERITED ERROR
  Issue: No rationale from Phase 2 → tasks don't explain Websocket choice
  Impact: Team implements without understanding constraints
  Evidence: tasks.md has "T001: Implement Websocket server"
            but no context why WS > SSE or Polling

✗ PERFORMANCE TESTING INHERITED ERROR
  Issue: No performance tests inherited from Phase 2
  Impact: Team ships without validation
  Evidence: tasks.md missing "T020: Performance test (1000 concurrent, P99 <500ms)"

═══════════════════════════════════════════════════════════

Phase 4 Assessment: Consistency Analysis (`speckit.analyze`)

Constitution Check - CRITICAL DETECTION:

Principle "MUST: Technical decisions with >1 option have decision rationale"
  Status: VIOLATED in plan.md
  Evidence:
    - Websockets chosen (design decision with alternatives exists)
    - No decision record in plan.md or research.md
    - Rationale missing: "Why Websockets over SSE/Polling?"

  Consequence:
    - Team proceeds without understanding constraints
    - Future refactoring risk (might replace with SSE later)
    - Audit finding: Constitution not enforced
    - Recommendation: Add decision rationale to plan.md before implement

═══════════════════════════════════════════════════════════

ENFORCEMENT ANALYSIS:

When Should Constitution Have Been Applied?

| Phase | Agent | Principle | Action | Status |
|---|---|---|---|---|
| 1 (Spec) | specify | Measurable Performance | Require targets in spec | ✓ APPLIED |
| 2 (Plan) | plan | Transparent Decisions | Detect design choices, require rationale | ✗ MISSED |
| 2 (Plan) | plan | Security First | Require security model section | ✗ PARTIALLY APPLIED |
| 2 (Plan) | plan | Measurable Performance | Require perf test strategy | ✗ MISSED |
| 3 (Tasks) | tasks | Transparent Decisions | Verify rationale from plan | ✗ ERROR INHERITED |
| 4 (Analyze) | analyze | All principles | Audit for violations | ✓ DETECTED |

═══════════════════════════════════════════════════════════

FINDINGS:

Constitution Enforcement Failure Detected

Root Causes:
1. Agent `speckit.plan` does not validate constitution principles
   - Should: Check for design decision points
   - Should: Require decision rationale for all multi-option choices
   - Currently: Generates plan without principle checks

2. Agent `speckit.analyze` can DETECT violations but runs too late
   - Detection: After tasks.md exists (Phase 4)
   - Problem: Violations already embedded in plan.md (Phase 2)
   - Result: Requires rework instead of preventing issue

3. No "middle-phase" validation
   - Spec generation too early (no design choices)
   - Task generation too late (violations already in plan)
   - Missing: Plan validation gate

Severity: HIGH
- Constitution "MUST" principles violated
- Pattern will repeat in every feature
- Audit finding: Governance failure

Recommendation:
1. Add Constitution Validation to `speckit.plan` agent
   - Before generating plan.md
   - Require explicit "Decision Records" section
   - Mandate rationale for any multi-option choices
   - Escalate if MUST principles can't be met

2. Add Decision Record Template to plan.md structure
   - Section: "Technical Decisions"
   - Format: Option A/B/C → Pros/Cons → Choice + Rationale
   - Constitution Alignment: Explicit (which principles apply?)

3. Update `speckit.tasks` to inherit/verify decision context
   - Validate: Each architecture task references decision record
   - Purpose: Team understands constraints

4. Update `speckit.analyze` to highlight gaps earlier
   - Flag: "Design choice detected but no rationale (Principle violated)"
   - Severity: CRITICAL (blocks implement)
```

### Rubric

| Criterion                      | Full (6 pts)                                                                    | Partial (3-4 pts)                | None (0 pts)       |
| ------------------------------ | ------------------------------------------------------------------------------- | -------------------------------- | ------------------ |
| **Phase-by-Phase Analysis**    | All 4 phases audited for constitution                                           | 2-3 phases analyzed              | <2 phases          |
| **Principles Checked**         | All 4 principles reviewed across phases                                         | 2-3 principles reviewed          | <2 principles      |
| **Violations Detected**        | All 3+ violations found with evidence                                           | 1-2 violations found             | <1 violation       |
| **Root Cause Analysis**        | Agent design gaps identified (plan agent missing validation)                    | Violations noted but cause vague | No causal analysis |
| **Enforcement Gap Identified** | Correctly identifies plan.md (Phase 2) as gap between spec/tasks                | Gap identified but mislocated    | No gap identified  |
| **Timing Analysis**            | Understands analysis happens too late (Phase 4) vs. prevention needed (Phase 2) | Timing mentioned but incomplete  | No timing analysis |
| **Recommendations**            | Specific agent/template/process changes proposed (4+ recommendations)           | Generic recommendations          | No recommendations |

**Total**: 42 points | **Pass Threshold**: 34 points (80%)

---

## Scenario H3: Adaptive Steering Loop Behavior Detection

**Category**: Harness Engineering | Difficulty: Hard

### Input Spec

**Context**: 6-sprint retrospective data showing team's learning and adaptation through steering loop cycles.

**Sprint 1 Retrospective**:

```
What went well:
- Daily standups caught blockers early

What didn't go well:
- Code reviews slow (8-10 hours avg)
- QA bottleneck (4-5 hour wait)
- Team didn't know how to prioritize work

Proposed changes:
- Rotate code reviewers
- Add QA capacity (hire contractor)
- Create priority decision matrix
```

**Sprint 2 Outcomes** (after implementing Sprint 1 changes):

```
- Code review time: 7 hours (improved 15%)
- QA wait: 3.5 hours (improved 30% despite no hiring)
- Priority decisions: Clearer alignment (team scores decision matrix 8/10)
- NEW PROBLEM: Uncoordinated refactoring causing merge conflicts
- Velocity: 28 → 32 points (+14%)
```

**Sprint 2 Retrospective**:

```
What went well:
- Code reviews improving
- Priorities clear
- Velocity up

What didn't go well:
- Merge conflicts (3x week average)
- Refactoring uncoordinated (developer silos)
- New team member overwhelmed by implicit practices

Proposed changes:
- Implement refactoring coordination (plan refactoring in sprint planning)
- Pair new hire with senior dev (1hr/day)
- Establish "refactoring lanes" (who refactors what)
```

**Sprint 3-4 Outcomes** (steering loop continues):

```
- Merge conflicts: 0.3/week (90% reduction)
- New hire ramp: 4/10 → 7/10 confidence
- Velocity: 32 → 35 points (stable)
- Unexpected benefit: Refactoring knowledge spread → architectural discussions improved
```

**Sprint 4 Retrospective** (team reflects on adaptation):

```
Observations:
- "We're getting better at solving problems we create"
- "Last cycle we didn't need to add QA capacity because we fixed the real bottleneck"
- "Pairing is working; team learning faster than we could hire"
- "Our priorities are working; we're building in right order"

New insight:
- "Real constraint isn't QA; it's architectural understanding"
- "If team understood system better, we'd design better, need less refactoring"

Proposed change:
- Architecture working session (2hr/week focused on system design)
- Architectural ADRs before building major features
```

**Sprint 5-6 Outcomes**:

```
- Velocity: 35 → 37 points (slight growth)
- Rework due to refactoring: 12% → 6% (50% reduction)
- Team confidence in design decisions: 6/10 → 8/10
- Fewer "surprise" architectural concerns in code review
- Incident rate: 2/sprint → 0.5/sprint (defect reduction from better design)
```

**Action**:

- Analyze: Show adaptive steering loop in action
- Verify: Team's hypothesis -> action -> learning -> new hypothesis pattern
- Assess: Evidence of systems thinking and constraint management maturity

### Expected Output

```
✓ Adaptive Steering Loop Analysis: 6-Sprint Journey

═══════════════════════════════════════════════════════════

STEERING LOOP CYCLE 1: Surface Symptoms, Propose Exploitation

Sprint 1 → Sprint 2 Steering Loop:

Sensors (Input):
- Code review time: 8-10 hours
- QA wait: 4-5 hours
- Prioritization confusion
- Symptom: Velocity stalling (constraint somewhere)

Comp/Inferential (Processing):
- Hypothesis 1: "QA is bottleneck" (intuitive assumption)
  - Evidence: Long QA wait times
  - Response: "Hire contractor"

- Hypothesis 2 (latent): "Code review capacity not scaling"
  - Evidence: Review time growing relative to story complexity
  - Response: "Rotate reviewers"

- Hypothesis 3: "Organizational clarity missing"
  - Evidence: Team unsure about priorities
  - Response: "Create priority decision matrix"

Steering (Control):
- Action 1: Rotate code reviewers (exploit existing capacity)
- Action 2: Hire QA contractor (elevate constraint)
- Action 3: Implement decision matrix (governance)

Sprint 2 Results:
- Code review: 8-10h → 7h (15% improvement from reviewer rotation)
- QA wait: 4-5h → 3.5h (30% improvement WITHOUT new hiring)
  * KEY INSIGHT: QA wait reduced by reviewer rotation alone!
  * Pattern: Code review was blocking QA work (hidden dependency)
  * Implication: "Hire QA" was wrong lever; throughput limited by earlier stage
- Priority alignment: 8/10 (decision matrix working)
- Velocity: 28 → 32 (+14%)

Steering Loop Learning:
✓ Constraint moved from QA to code review
✓ Hiring was unnecessary (constraint eliminated by process change)
✓ Team learned Theory of Constraints: "subordinate everything to constraint"
  * Code review → QA → Dev
  * Fixing code review fixed QA downstream
  * System-level thinking demonstrated

═══════════════════════════════════════════════════════════

STEERING LOOP CYCLE 2: Detect Emerging Constraint, Elevate

Sprint 2 → Sprint 3 Steering Loop:

Sensors (Input):
- NEW PROBLEM: Merge conflicts (3/week average)
- NEW PROBLEM: New hire overwhelmed (implicit practices)
- IMPROVEMENT: Code review no longer bottleneck
- Signal: Constraint migrated again

Comp/Inferential (Processing):
- Hypothesis: "Uncoordinated work causes conflicts; coordination process missing"
  - Evidence: 3 conflicts/week, refactoring in silos
  - Mechanism: Developers don't know what others refactoring → collision
  - Root Cause: Implicit practices (new hire can't infer)

- Response: Implement coordination mechanisms
  - Refactoring planning in sprint planning
  - Refactoring "lanes" (who refactors where)
  - Pairing new hire (knowledge transfer + accountability)

Steering (Control):
- Action 1: Plan refactoring explicitly (coordination)
- Action 2: Establish refactoring lanes (reduce collisions)
- Action 3: Pair new hire (make implicit explicit; grow team capacity)

Sprint 3 Results:
- Merge conflicts: 3/week → 0.3/week (90% reduction)
- New hire confidence: 4/10 → 7/10 (75% improvement)
- Velocity: 32 → 35 (stable + 9%)
- Architectural discussions: Frequency up, quality up
  * EMERGENT BENEFIT: Refactoring sharing increased system understanding
  * Insight: Bottleneck wasn't coordination; it was knowledge silos
  * New dynamic: Team learning about system design through refactoring

Steering Loop Learning:
✓ Constraint evolved from code review → coordination/knowledge
✓ Solution addressed root cause (implicit knowledge) not symptom (conflicts)
✓ Pairing (people investment) created learning compounding effect
✓ Team building institutional knowledge (ADRs concept emerging)

═══════════════════════════════════════════════════════════

STEERING LOOP CYCLE 3: Evolve Hypothesis, Shift Strategy

Sprint 4 → Sprint 5-6 Steering Loop:

Sensors (Input):
- REFLECTION: Team notices patterns
- INSIGHT: "Real constraint is architectural understanding"
- Evidence: Better design would reduce refactoring, merge conflicts, rework
- Team recognizes: Constraint has shifted (once again)
  * Sprint 1: Code review capacity
  * Sprint 2: Coordination process + knowledge silos
  * Sprint 3-4: Architectural understanding gaps

Comp/Inferential (Processing):
- Hypothesis Evolution: "Constraint is not capacity; it's knowledge"
  - Insight: "We don't need to hire; we need to understand design better"
  - Strategy: Design working session + ADRs before implementation
  - Investment: 2hr/week structured learning

- Systems Thinking: Team now sees connections
  - Code review → affects QA → affects velocity
  - Refactoring knowledge → affects design quality → affects incidents
  - Architectural clarity → affects all downstream work

Steering (Control):
- Action 1: Architecture working session (2hr/week)
  - Purpose: Shared design understanding
  - Investment: Deliberate learning time

- Action 2: ADRs before major features
  - Purpose: Codify decisions; make implicit explicit
  - Investment: Documentation discipline

- Strategy: Invest in system knowledge rather than capacity growth

Sprint 5-6 Results:
- Velocity: 35 → 37 (continuing growth)
- Rework due to refactoring: 12% → 6% (50% reduction)
- Team design confidence: 6/10 → 8/10
- Architectural concerns in review: Fewer surprises (better pre-design)
- Incident rate: 2/sprint → 0.5/sprint (75% reduction)
- EMERGENT OUTCOME: Architectural quality compounding

Steering Loop Learning:
✓ Constraint has shifted 3 times in 6 sprints (proving TOC)
✓ Each solution removed that level, exposed the next level
✓ Team's hypothesis-testing improving (initial "hire QA" was wrong)
✓ Team learned to invest in people knowledge, not just capacity
✓ System quality improving as team's understanding deepens
✓ Compounding effect: Knowledge investment pays exponentially

═══════════════════════════════════════════════════════════

ADAPTIVE LEARNING EVIDENCE:

Theory of Constraints Maturity:

Sprint 1:
- TOC Level 1 (Identify + Wrong Hypothesis)
- Identified constraint (QA wait) but root cause was earlier (code review)
- Planned wrong lever (hire QA) but learned through action

Sprint 2:
- TOC Level 2 (Exploit + Correct Targeting)
- Identified correct constraint (code review)
- Exploited effectively (reviewer rotation)
- Understood downstream impact (QA wait reduced without hiring)

Sprint 3-4:
- TOC Level 2 → 3 (Constraint Evolution)
- Subordinated all else to new constraint (coordination/knowledge)
- Implemented people investment (pairing, governance)
- Recognized emerging architectural constraint

Sprint 5-6:
- TOC Level 3 (Systemic Thinking)
- Team articulates that "constraint is knowledge"
- Invests deliberately in systems understanding
- Recognizes compounding effect of architectural quality

═══════════════════════════════════════════════════════════

STEERING LOOP EFFECTIVENESS:

Feedback Quality Improvement:

Sprint 1: General symptoms ("slow reviews", "QA wait")
  → Sprint 2: Mechanism understanding (reviewer rotation helps QA)
  → Sprint 3-4: System insight (coordination + knowledge key)
  → Sprint 5-6: Strategic hypothesis (design quality compounds)

Decision Quality Improvement:

Sprint 1: Intuitive guesses (hire QA)
  → Sprint 2: Targeted interventions (rotate reviewers)
  → Sprint 3-4: Process + people investment (pairing, lanes)
  → Sprint 5-6: Strategic learning (architecture sessions, ADRs)

Problem-Solving Sophistication:

Sprint 1: Symptomatic (wait times high → add capacity)
  → Sprint 2: Causal (code review limiting → optimize process)
  → Sprint 3-4: Systems (coordination + learning affecting all layers)
  → Sprint 5-6: Strategic (invest in knowledge for compounding returns)

═══════════════════════════════════════════════════════════

EVIDENCE OF ADAPTIVE STEERING LOOP:

✓ Cycle Pattern: Observe → Hypothesize → Test → Learn → Reflect
  - Completed 3 full cycles
  - Hypothesis quality improving
  - Learning accumulates across cycles

✓ Constraint Adaptation: TOC applied with increasing sophistication
  - Correctly identified 3 successive constraints
  - Exploited each before elevation
  - Recognized pattern (constraints shift when eliminated)

✓ Systems Thinking: Team moved from component to system view
  - Sprint 1: "Code review is slow" (component view)
  - Sprint 4: "Architectural understanding is constraint" (system view)
  - Insight: Design quality affects code review load affects QA

✓ Learning Velocity: Each cycle faster and more targeted
  - Sprint 1: Wrong hypothesis, learned through action
  - Sprint 2: Correct targeting, quick win
  - Sprint 3-4: Proactive strategy (pairing before crisis)
  - Sprint 5-6: Predictive (prevent incidents through design quality)

✓ Compounding Effect: Investments paying exponential returns
  - Pairing: Grew team capacity AND institutional knowledge
  - Architecture focus: Reduced rework AND improved velocity
  - Design investment: Lower incidents AND better decisions

═══════════════════════════════════════════════════════════

STEERING LOOP MATURITY ASSESSMENT: ADVANCED

Harness Working Well:

Guides/Sensors: ✓
- Feedback mechanisms detecting real problems
- Metrics revealing system dynamics
- Team reflective (recognizing patterns)

Comp/Inferential: ✓
- Decision quality improving
- Root cause analysis deepening
- Strategic thinking emerging

Steering Loop: ✓
- Closed cycles (action → measurement → reflection → adaptation)
- Learning compounds across cycles
- Team autonomously generating better hypotheses

═══════════════════════════════════════════════════════════

Conclusion:

The steering loop is functioning as designed. Evidence shows:

1. Adaptive feedback system (cycle 1 → cycle 2 → cycle 3 increasing sophistication)
2. Hypothesis testing and learning (initial wrong → increasingly correct)
3. Systems understanding deepening (component → system → strategic)
4. Compounding returns (people investment → knowledge → quality → velocity)

This team has entered virtuous cycle of continuous improvement through harness engineering discipline.

Next Constraint Prediction:
- Likely emerging: Scaling (team growing? Codebases multiplying?)
- Pattern: Team will apply same steering loop, identify leverage point
```

### Rubric

| Criterion                                | Full (6 pts)                                                                 | Partial (3-4 pts)                            | None (0 pts)               |
| ---------------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------- | -------------------------- |
| **All 3 Cycles Analyzed**                | Cycle 1, 2, 3 with input/comp/steering layers                                | 1-2 cycles analyzed                          | <1 cycle                   |
| **Constraint Evolution Tracked**         | Shows 3 successive constraints with evidence                                 | 1-2 constraints identified                   | No evolution shown         |
| **TOC Maturity Progression**             | Level 1 → 2 → 3 progression clear with criteria                              | Progression mentioned but vague              | No progression analysis    |
| **Learning Hypothesis Quality**          | Shows improvement from Sprint 1 (wrong) → Sprint 5 (strategic)               | Mentions learning but no quality progression | No learning progression    |
| **Systems Thinking Evidence**            | Team moves from component to system view across cycles                       | Mentioned but not evidenced                  | No systems thinking        |
| **Compounding Effect Identified**        | Knowledge investment → multiple cascading benefits shown                     | Benefits listed but causation unclear        | No compounding shown       |
| **Steering Loop Effectiveness Measured** | Feedback quality, decision quality, sophistication all improved quantifiably | 1-2 dimensions improved                      | No measurement             |
| **Prediction/Insight**                   | Team ready for next constraint; next cycle predicted                         | Generic observation                          | No forward-looking insight |

**Total**: 48 points | **Pass Threshold**: 38 points (80%)

---

## Summary: Test Scenario Coverage

| Difficulty | Scenarios | Domains                                                                            | Total Points | Pass Threshold |
| ---------- | --------- | ---------------------------------------------------------------------------------- | ------------ | -------------- |
| **Easy**   | E1-E4     | Agent context (2), Harness principles (2)                                          | 60           | 48 (80%)       |
| **Medium** | M1-M3     | Agent flow (1), Constraint detection (1), Feedback loops (1)                       | 95           | 76 (80%)       |
| **Hard**   | H1-H3     | Multi-artifact validation (1), Constitution enforcement (1), Adaptive steering (1) | 132          | 106 (80%)      |
| **TOTAL**  | 10        | 6 core concepts                                                                    | 287          | 230 (80%)      |

---

## Acceptance Criteria Verification

✓ **tests/scenarios.md** created with 10 scenarios
✓ **4 Easy scenarios**: E1-E4 (each 15 pts, 80% pass threshold 12 pts)
✓ **3 Medium scenarios**: M1-M3 (each 30-35 pts, 80% pass threshold)
✓ **3 Hard scenarios**: H1-H3 (each 42-48 pts, 80% pass threshold)
✓ **Input spec + Expected output + Rubric**: All included for each scenario
✓ **Domains covered**: Agent context analysis, Harness engineering, Constitution enforcement, Steering loop dynamics
