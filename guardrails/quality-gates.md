# Quality Gates

Quality gates define the minimum criteria that AI-assisted outputs must satisfy before they are accepted and used. These criteria are explicit to make review consistent across engineers and sessions.

---

## How to use this document

Each output type has a quality gate checklist. Before accepting an AI-assisted output, work through the relevant checklist. If any required criterion fails, the output requires revision or rejection — partial passes are not acceptable.

Quality gates are not suggestions. An output that passes 8 of 10 criteria is not ready if it fails a required one.

---

## Code outputs

**Required:**
- [ ] The code compiles or passes a syntax check
- [ ] The logic is traceable — you can explain what it does line by line without re-reading
- [ ] Edge cases are handled or explicitly noted as out of scope
- [ ] No obvious security issues: injection points, credential exposure, unsafe deserialization, unvalidated input at system boundaries
- [ ] The code follows the project's existing style and conventions
- [ ] If tests are expected: tests are present and cover the happy path and at least one failure path

**Disqualifying conditions (reject immediately):**
- The code references APIs, libraries, or function signatures that don't exist in the codebase
- The logic produces incorrect results on known inputs
- The model hallucinated a dependency or fabricated a method signature

---

## Analysis outputs (code review, architecture review, debugging hypotheses)

**Required:**
- [ ] The analysis addresses the specific question asked — it is not a generic overview
- [ ] Claims are specific: "this loop runs O(n²) because of the nested slice scan at line 47" not "this may be slow"
- [ ] Recommendations are actionable and scoped to the system described
- [ ] The analysis does not assume context that was not provided

**Disqualifying conditions:**
- The analysis contradicts verified facts about the system
- Recommendations reference tooling or patterns incompatible with the system's known constraints
- The output is a generic list applicable to any codebase, not the one under review

---

## Documentation outputs

**Required:**
- [ ] Accurate — every technical claim can be verified against the actual system
- [ ] Complete — covers the scope specified in the output contract
- [ ] No hallucinated features, interfaces, or behaviors
- [ ] Language is precise: no unqualified terms like "handles," "manages," or "processes" without specifics
- [ ] Consistent with existing documentation style and terminology

**Disqualifying conditions:**
- Any factual inaccuracy about the system's behavior
- References to features, endpoints, or behaviors that don't exist
- Contradictions with existing authoritative documentation

---

## Investigation outputs (debugging, incident investigation, DataDog analysis)

**Required:**
- [ ] Hypotheses are grounded in the evidence provided — not speculative beyond what the data supports
- [ ] Timeline reconstruction is consistent with the provided data
- [ ] Root cause claims are qualified: "confirmed by X," "probable based on Y," "ruled out because Z"
- [ ] Impact assessment is scoped, not worst-case assumed without evidence
- [ ] Next steps are concrete and assignable

**Disqualifying conditions:**
- Root cause stated as confirmed without supporting evidence
- Timeline inconsistencies that were not flagged
- Investigation that ignores relevant signals present in the provided data

---

## Communication outputs (async updates, status reports, stakeholder messages)

**Required:**
- [ ] Accurate representation of the actual state — no optimism bias introduced by the model
- [ ] Blockers are clearly identified, not buried or softened
- [ ] Decisions made vs. decisions pending are clearly distinguished
- [ ] Tone matches the audience and situation

**Disqualifying conditions:**
- The output omits a known blocker or risk
- A pending decision is presented as made
- The output reframes the situation more positively than the engineer's input described it

---

## Ticket and requirement outputs

**Required:**
- [ ] Acceptance criteria are testable (binary pass/fail, not subjective)
- [ ] Edge cases are enumerated specifically, not listed as generic warnings
- [ ] Scope is clearly bounded — what is in and what is explicitly out
- [ ] No ambiguous terms in acceptance criteria: "fast," "user-friendly," "handles gracefully" are not testable

**Disqualifying conditions:**
- Acceptance criteria that cannot be tested objectively
- Scope that contradicts known product or technical constraints
- Missing edge cases the engineer can identify from system knowledge
