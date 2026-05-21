# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify the implementation is well-built — clean, tested, maintainable.

**Only dispatch after spec compliance review passes.** If spec compliance failed, the implementer is fixing spec gaps — code quality review on that state is wasted work.

**READ-ONLY ROLE:** The reviewer must not modify files, run `--fix` commands, or create commits. Findings only — the implementer fixes any issues in a follow-up dispatch. Reinforce this explicitly in the dispatched prompt.

```
Task tool (general-purpose):
  description: "Code quality review for Task N"

  Use template at requesting-code-review/code-reviewer.md, with these inputs:
    DESCRIPTION: [task summary, from implementer's report]
    PLAN_OR_REQUIREMENTS: Task N from [plan-file]
    BASE_SHA: [commit before task]
    HEAD_SHA: [current commit]

  Then append the following to the dispatched prompt:

  <role>
  You are a code quality reviewer. You are READ-ONLY: do not modify files, run --fix commands, or create commits. Findings only.
  </role>

  <additional_checks>
  In addition to standard code quality concerns, also check:
  - Does each file have one clear responsibility with a well-defined interface?
  - Are units decomposed so they can be understood and tested independently?
  - Is the implementation following the file structure from the plan?
  - Did this implementation create new files that are already large, or significantly grow existing files? (Don't flag pre-existing file sizes — focus on what this change contributed.)
  </additional_checks>

  <before_responding>
  Before writing your review, take a moment to think through what could go wrong with this change in production — not just style and structure, but edge cases, error handling, concurrency, security implications, and what happens when assumptions in the code are violated. Use that thinking to inform what you flag as Critical vs. Important vs. Minor.
  </before_responding>

  <verdict_format>
  Return:
  - Strengths
  - Issues, categorized: Critical / Important / Minor
  - Assessment line: "Ready to merge: Yes | No | With fixes"
  </verdict_format>
```

## How the orchestrator interprets the verdict

The reviewer's `Assessment` line determines next steps:

- **`Ready to merge: Yes`** AND zero Critical AND zero Important issues → **approved**. Minor issues may be noted and proceeded past.
- **`Ready to merge: With fixes`** OR any Critical/Important issues → **not approved**. Route the issue list back to the implementer, they fix, then re-dispatch this same reviewer until clean.
- **`Ready to merge: No`** → **not approved**. Same flow as With fixes; if the reviewer is suggesting the change shouldn't ship at all (e.g., wrong approach), surface this to the human before re-dispatching.

Do not advance to the next task while either review has unresolved Critical or Important issues.
