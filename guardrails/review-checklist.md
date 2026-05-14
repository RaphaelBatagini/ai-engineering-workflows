# Review Checklist

A rapid checklist for reviewing AI-assisted outputs before use. This checklist applies across all output types. For type-specific criteria, see [quality-gates.md](quality-gates.md).

---

## Before reviewing: establish context

- [ ] I have read the prompt that produced this output
- [ ] I know what context was provided to the model
- [ ] I know what the output contract was — what structure and scope was expected

If you are reviewing an output without knowing what prompt produced it or what context was provided, stop. Request that context before continuing. Reviewing an output without its input is not a valid review.

---

## Accuracy

- [ ] Every technical claim can be verified against available evidence (logs, code, documentation, tickets)
- [ ] No claims contradict what you know about the system
- [ ] References to external systems, APIs, or tools are accurate
- [ ] Numbers, metrics, thresholds, and timelines are correct — do not assume they are

---

## Completeness

- [ ] The output addresses the full scope of what was asked
- [ ] Nothing important from the input was silently dropped
- [ ] Edge cases were addressed or explicitly flagged as out of scope

---

## Scope

- [ ] The output stays within the scope defined in the prompt
- [ ] The model did not add unrequested recommendations or analysis
- [ ] No implicit assumptions were introduced that weren't in the input

---

## Precision

- [ ] Technical claims are specific, not vague
- [ ] Uncertainty is expressed where it exists — not hidden or glossed over
- [ ] Confidence language matches the actual evidence: watch for "will," "always," "definitely" applied to uncertain outcomes

---

## Human judgment gate

Before accepting the output:

- [ ] I can stand behind this output as my own professional judgment
- [ ] If challenged on any claim in this output, I can defend it from first principles
- [ ] I have not accepted the output because it "seems right" — I have actively validated each material claim

This gate is not a formality. An engineer whose name is on the final artifact is accountable for its accuracy regardless of how it was generated.

---

## After review

- [ ] If the output required significant corrections, the prompt or workflow should be updated to prevent the same issues
- [ ] If a new failure mode was encountered, document it in the relevant prompt file's failure modes section
- [ ] If the output was rejected after multiple revision attempts, note whether the workflow or prompt needs structural change
