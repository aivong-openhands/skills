---
name: test-improvement-workflow
description: This skill should be used when the user asks to "improve test quality", "refactor tests", "audit tests and fix them", or wants a systematic workflow for improving test suite quality using Dave Farley's principles. Orchestrates the test-design-reviewer, tdd, and refactoring skills into a complete improvement workflow.
---

# Test Improvement Workflow

A systematic workflow for improving test suite quality using Dave Farley's 8 properties of good tests. This skill orchestrates the `test-design-reviewer`, `tdd`, and `refactoring` skills into a complete improvement cycle.

## Prerequisites

Ensure these skills are available in the workspace:
- `test-design-reviewer` - For auditing test quality using Dave Farley's framework
- `tdd` - For RED-GREEN-MUTATE-KILL MUTANTS-REFACTOR cycle
- `refactoring` - For safe, behavior-preserving code improvements

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  1. AUDIT: Run test-design-reviewer to get Farley Score     │
├─────────────────────────────────────────────────────────────┤
│  2. PRIORITIZE: Determine most efficient order of fixes     │
├─────────────────────────────────────────────────────────────┤
│  3. PLAN: Create task list with skill assignments           │
│     • TDD skill for new code (RED → GREEN)                  │
│     • Refactoring skill for existing code changes           │
├─────────────────────────────────────────────────────────────┤
│  4. EXECUTE: Follow plan, commit after each phase           │
├─────────────────────────────────────────────────────────────┤
│  5. SKIP CHECK: Evaluate if improvements add value          │
├─────────────────────────────────────────────────────────────┤
│  6. VERIFY: Run unbiased re-audit in NEW conversation       │
└─────────────────────────────────────────────────────────────┘
```

**Important**: This workflow proceeds automatically through steps 1-4 without prompting for user input on ordering. After execution, evaluate whether to skip remaining improvements.

---

## Step 1: Initial Audit

Run the test-design-reviewer skill to evaluate current test quality:

```
test-design-reviewer audit test quality of each test file and make a report of results
```

This produces:
- Property scores (Understandable, Maintainable, Repeatable, Atomic, Necessary, Granular, Fast, First)
- Farley Score (1-10 weighted average)
- Top recommendations for improvement

---

## Step 2: Prioritize Recommendations (Using test-design-reviewer)

**Do not prompt the user** - use the `test-design-reviewer` skill to automatically determine the most efficient implementation order.

The test-design-reviewer evaluates priorities based on:

- **Dependencies** between recommendations
- **Foundation-first** approach (fix patterns before consolidating)
- **Effort vs impact** tradeoffs
- **Property weights** from the Farley Score formula (Understandable and Maintainable have 1.5× weight)

### Priority Order Logic (from test-design-reviewer)

Using the Farley Score weights as guidance:

1. **First** (Highest impact): Changes affecting **Understandable** (1.5×) or **Maintainable** (1.5×) properties - these have the highest weight in the Farley Score
2. **Second**: Changes affecting **Repeatable** (1.25×) - reliability is critical for trust
3. **Third**: Changes affecting **Atomic**, **Necessary**, **Granular**, or **First** (1.0×) - core principles
4. **Last**: Changes affecting **Fast** (0.75×) - can be optimized later

Within each tier, order by:
- Design decisions affecting future code first (e.g., reduce implementation coupling)
- Changes that leverage patterns from earlier steps (e.g., consolidate with parameterization)
- Cleanup that's easier after consolidation (e.g., extract common helpers)

### Example Priority Order

| Order | Recommendation | Property Affected | Rationale |
|-------|----------------|-------------------|-----------|
| 1st | Reduce implementation coupling | Maintainable (1.5×) | Design decision affecting future code |
| 2nd | Consolidate with parameterization | Necessary (1.0×) | Uses cleaner patterns from step 1 |
| 3rd | Extract common helpers | Understandable (1.5×) | Fewer places to apply after consolidation |

---

## Step 3: Create Implementation Plan (Automatic)

**Do not prompt the user** - automatically create a detailed task plan with skill assignments.

### Skill Assignment Rules

| Task Type | Skill | When to Use |
|-----------|-------|-------------|
| Adding new code | **TDD** | New helper methods, new test utilities |
| Modifying existing code | **Refactoring** | Consolidating tests, updating assertions |
| Verification | **TDD** | Running tests, checking coverage |

### Plan Structure

Each phase should follow this pattern:

**For new functionality (TDD skill):**
1. 🔴 **RED**: Write failing tests for new behavior
2. 🟢 **GREEN**: Implement minimum code to pass
3. ✅ **VERIFY**: Run tests to confirm GREEN
4. 💾 **COMMIT**: Save working code before refactoring

**For code improvements (Refactoring skill):**
1. 💾 **COMMIT FIRST**: Always commit working code before refactoring
2. 🔄 **REFACTOR**: Make behavior-preserving changes
3. ✅ **VERIFY**: Run tests to confirm no regressions
4. 💾 **COMMIT**: Save refactored code

### Example Plan Structure

```markdown
## Phase 1: Reduce Implementation Coupling [TDD skill]

### 1.1 RED: Write failing tests for is_unchanged() helper
[TDD skill] Write tests expecting is_unchanged(key) method before implementing

### 1.2 GREEN: Implement is_unchanged() helper
[TDD skill] Write minimum code to make tests pass

### 1.3 COMMIT before refactoring
[TDD skill] Save working code as safety net

### 1.4 REFACTOR: Update existing tests to use helper
[Refactoring skill] Replace old assertions with new helper. No behavior change.

### 1.5 VERIFY and COMMIT
[TDD skill] Run all tests, then commit refactoring
```

---

## Step 4: Execute the Plan

Follow the plan systematically:

### TDD Cycle (for new code)

```bash
# 1. Write failing test
# 2. Run test - confirm it fails (RED)
pytest test_file.py::TestClass::test_method -v

# 3. Implement minimum code
# 4. Run test - confirm it passes (GREEN)
pytest test_file.py::TestClass::test_method -v

# 5. Commit before refactoring
git add -A && git commit -m "feat: add helper method"
```

### Refactoring Cycle (for existing code)

```bash
# 1. Commit current working state FIRST
git add -A && git commit -m "checkpoint: before refactoring"

# 2. Make refactoring changes
# 3. Run ALL tests to verify no regressions
pytest test_file.py -v

# 4. Commit refactoring
git add -A && git commit -m "refactor: consolidate tests with parameterization"
```

### Commit Message Conventions

| Type | Format | Example |
|------|--------|---------|
| New feature | `feat: <description>` | `feat: add is_unchanged() helper to UpdateResult` |
| Refactoring | `refactor: <description>` | `refactor: consolidate tests with @pytest.mark.parametrize` |
| Tests only | `test: <description>` | `test: add failing tests for new helper` |

---

## Step 5: Skip Check - Evaluate Improvement Value

After completing the plan (or before executing low-priority items), evaluate whether remaining improvements should be skipped.

### When to Recommend Skipping

Reference the **test-design-reviewer** scoring criteria to justify skipping:

| Property | Skip When | Justification Using Farley's Properties |
|----------|-----------|------------------------------------------|
| Understandable | Score ≥ 8 | Tests already read like specifications; additional clarity adds marginal value |
| Maintainable | Score ≥ 8 | Abstractions are sufficient; more helpers may over-engineer |
| Repeatable | Score ≥ 9 | Tests are deterministic; no flakiness to address |
| Atomic | Score ≥ 9 | Tests are isolated; further isolation is unnecessary |
| Necessary | Score ≥ 8 | No significant redundancy; consolidation may reduce coverage |
| Granular | Score ≥ 8 | Tests are focused; splitting further reduces readability |
| Fast | Score ≥ 8 | Tests are quick enough; micro-optimizations waste effort |
| First (TDD) | Score ≥ 7 | Evidence of test-first; historical changes not worth rewriting |

### Skip Recommendation Format

When recommending to skip an improvement, explain using this format:

```markdown
### 🚫 Recommended Skip: [Improvement Name]

**Current Score**: [Property] = X/10

**Why This Doesn't Add Value**:
Based on Dave Farley's [Property] principle, this test suite already:
- [Specific evidence from the code]
- [How current state meets the principle]

**Cost vs Benefit**:
- Effort required: [Low/Medium/High]
- Expected score improvement: [+0.X]
- Risk of introducing issues: [Description]

**Recommendation**: Skip this improvement. The current implementation 
sufficiently satisfies Farley's [Property] property.
```

### Overall Skip Threshold

If the **Farley Score is ≥ 9.0**, ask the user whether to proceed with improvements:

```markdown
## ⚠️ High-Quality Test Suite Detected

**Current Farley Score: X.X/10 (Excellent)**

This test suite already demonstrates strong adherence to Dave Farley's 
principles. The recommended improvements would yield diminishing returns:

| Improvement | Est. Score Impact | Effort | Recommendation |
|-------------|-------------------|--------|----------------|
| [Name] | +0.X | [Effort] | [Skip/Proceed] |

**Would you like to:**
1. **Skip all** - The test suite is production-ready
2. **Proceed with high-impact only** - Implement only items with Skip=Proceed
3. **Proceed with all** - Implement all recommendations regardless

Please confirm before continuing.
```

---

## Step 6: Unbiased Re-evaluation

**Critical**: Run the workflow again in a **new conversation** to ensure unbiased evaluation.

### Why a New Conversation?

- No prior knowledge of target scores
- No context bias from implementation work
- Objective assessment of current code state only

### Instructions for Re-evaluation

Start a new OpenHands conversation with:

```
test-improvement-workflow
```

**Important**: Do NOT include any target scores or expectations in the prompt.

### Comparing Results

After the new workflow run completes its audit:
1. Compare Farley Scores (before vs after)
2. Review individual property improvements
3. Continue with additional improvement phases if recommended

---

## Common Improvement Patterns

### Pattern 1: Reduce Implementation Coupling

**Problem**: Tests directly access internal data structures

```python
# Before (coupled to internal structure)
assert ("key", "value") in result.unchanged
```

**Solution**: Add helper methods to production code

```python
# After (uses abstraction)
assert result.is_unchanged("key")
```

### Pattern 2: Consolidate with Parameterization

**Problem**: Multiple similar test methods

```python
# Before (redundant)
def test_case_a(self):
    assert func("a") == "A"

def test_case_b(self):
    assert func("b") == "B"
```

**Solution**: Use `@pytest.mark.parametrize`

```python
# After (consolidated)
@pytest.mark.parametrize("input,expected", [
    ("a", "A"),
    ("b", "B"),
])
def test_func_transforms_correctly(self, input, expected):
    assert func(input) == expected
```

### Pattern 3: Extract Common Assertions

**Problem**: Repeated assertion patterns across tests

```python
# Before (duplicated)
content = file.read_text()
assert "key1" in content
assert "key2" in content
```

**Solution**: Create reusable helper in conftest.py

```python
# conftest.py
def assert_file_contains_all(file_path, expected_strings):
    content = file_path.read_text()
    for expected in expected_strings:
        assert expected in content

# Test file
assert_file_contains_all(file, ["key1", "key2"])
```

---

## Workflow Summary

| Step | Action | Skill Used | Prompts User? |
|------|--------|------------|---------------|
| 1 | Audit test quality | test-design-reviewer | No |
| 2 | Prioritize recommendations | test-design-reviewer | No (automatic) |
| 3 | Create implementation plan | TDD + Refactoring | No (automatic) |
| 4 | Execute plan with commits | TDD + Refactoring | No |
| 5 | Evaluate skip vs proceed | test-design-reviewer | **Yes** (if score ≥ 8.0) |
| 6 | Re-run in new conversation | test-improvement-workflow | No |

---

## Quick Start

When this workflow is invoked, automatically:

1. **Audit** all test files using test-design-reviewer
2. **Prioritize** recommendations by dependency order and impact
3. **Plan** implementation with TDD/Refactoring skill assignments
4. **Execute** the plan, committing after each phase
5. **Evaluate** whether to skip low-value improvements (explain using Farley's properties)
6. **Recommend** re-evaluation in a new conversation
