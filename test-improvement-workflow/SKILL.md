---
name: test-improvement-workflow
description: This skill should be used when the user asks to "improve test quality", "refactor tests", "audit tests and fix them", or wants a systematic workflow for improving test suite quality using Dave Farley's principles. Orchestrates the test-design-reviewer, tdd, testing, and refactoring skills into a complete improvement workflow.
---

# Test Improvement Workflow

A systematic workflow for improving test suite quality using Dave Farley's 8 properties of good tests. This skill orchestrates the `test-design-reviewer`, `tdd`, `testing`, and `refactoring` skills into a complete improvement cycle.

## Prerequisites

Ensure these skills are available in the workspace:
- `test-design-reviewer` - For auditing test quality using Dave Farley's framework
- `tdd` - For RED-GREEN-MUTATE-KILL MUTANTS-REFACTOR cycle
- `refactoring` - For safe, behavior-preserving code improvements
- `testing` - For behavior-driven testing patterns and edge case documentation

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────┐
│  1. AUDIT: Run test-design-reviewer to get Farley Score     │
├─────────────────────────────────────────────────────────────┤
│  2. PRIORITIZE: Determine most efficient order of fixes     │
│     (Always show prioritized improvements to user)          │
├─────────────────────────────────────────────────────────────┤
│  3. PRESENT: Show improvements table with effort/impact     │
│     • Even for exemplary suites (≥9.0), show the list       │
│     • Let user decide whether to proceed                    │
├─────────────────────────────────────────────────────────────┤
│  4. PLAN: Create task list with skill assignments           │
│     • TDD skill for new code (RED → GREEN)                  │
│     • Refactoring skill for existing code changes           │
│     • testing skill for edge case comments                  │
│     • test-design-reviewer skill for other test comments    │
├─────────────────────────────────────────────────────────────┤
│  5. EXECUTE: Follow plan, commit after each phase           │
├─────────────────────────────────────────────────────────────┤
│  6. VERIFY: Run unbiased re-audit in NEW conversation       │
└─────────────────────────────────────────────────────────────┘
```

**Important**: Always show the prioritized improvements table to the user, regardless of the Farley Score. Even exemplary test suites (≥9.0) benefit from seeing what minor improvements are possible.

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

## Step 2: Calculate Efficiency Scores and Prioritize

**CRITICAL**: You MUST calculate an Efficiency Score for each improvement and sort by efficiency (highest first). Do not skip this calculation.

### Efficiency Score Formula

For each identified improvement, calculate:

```
Efficiency Score = (Property Weight × Score Improvement) ÷ Effort Value

Where:
- Property Weights: Understandable=1.5, Maintainable=1.5, Repeatable=1.25, others=1.0, Fast=0.75
- Score Improvement: Estimated points gained (e.g., improving a property from 8→9 = +1.0)
- Effort Values: Low=1, Medium=2, High=3
```

### Efficiency Score Calculation Example

| Improvement | Property | Weight | Δ Score | Effort | Effort Val | Efficiency |
|-------------|----------|--------|---------|--------|------------|------------|
| Add docstrings | Understandable | 1.5 | +0.5 | Low | 1 | **0.75** |
| Extract helpers | Maintainable | 1.5 | +0.3 | Medium | 2 | **0.23** |
| Speed up tests | Fast | 0.75 | +1.0 | High | 3 | **0.25** |

**Sort order**: Add docstrings (0.75) → Speed up tests (0.25) → Extract helpers (0.23)

### Dependency Adjustments

After calculating raw efficiency, adjust for dependencies:

1. **Foundation-first**: If improvement B depends on A, A goes first regardless of efficiency
2. **Compound benefits**: If A makes B easier, boost A's effective efficiency by 25%
3. **Consolidation order**: Design changes → Consolidation → Cleanup

### Final Priority Order Logic

1. **Sort by Efficiency Score** (highest first)
2. **Apply dependency adjustments** (move prerequisites earlier)
3. **Break ties using property weight** (higher weight wins)

### Worked Example

Given these raw calculations:
| Improvement | Efficiency | Depends On |
|-------------|------------|------------|
| Consolidate with parameterization | 0.50 | Reduce coupling |
| Reduce implementation coupling | 0.45 | - |
| Extract common helpers | 0.30 | Consolidation |

**Final order** (after dependency adjustment):
1. Reduce implementation coupling (0.45, but prerequisite)
2. Consolidate with parameterization (0.50, depends on #1)
3. Extract common helpers (0.30, depends on #2)

---

## Step 3: Present Prioritized Improvements with Efficiency Scores

**Always show the prioritized improvements table**, regardless of the Farley Score. This helps users understand what's possible and make informed decisions.

### Improvements Table Format

Present improvements in this format, **sorted by Efficiency Score (highest first)**:

```markdown
## 📋 Prioritized Improvements (Sorted by Efficiency)

**Current Farley Score: X.X/10 ([Rating])**

| # | Improvement | Property | Δ Score | Effort | Efficiency | Skill |
|---|-------------|----------|---------|--------|------------|-------|
| 1 | [Description] | [Property] (1.5×) | +0.X | Low | **0.XX** | [skill] |
| 2 | [Description] | [Property] (1.0×) | +0.X | Medium | **0.XX** | [skill] |
| 3 | [Description] | [Property] (0.75×) | +0.X | High | **0.XX** | [skill] |

**Legend**: Efficiency = (Weight × Δ Score) ÷ Effort. Higher = better ROI.
```

**IMPORTANT**: The Efficiency column MUST be populated with calculated values. Do not leave it blank or use placeholders.

### For Exemplary Test Suites (≥ 9.0)

Even when the test suite scores ≥ 9.0 (Exemplary), **still show the improvements table with efficiency scores**:

```markdown
## 🏆 Exemplary Test Suite - Optional Improvements (Sorted by Efficiency)

**Current Farley Score: X.X/10 (Exemplary)**

This test suite already demonstrates exemplary adherence to Dave Farley's 
principles. The following improvements are optional but could provide 
marginal gains:

| # | Improvement | Property | Δ Score | Effort | Efficiency | Status |
|---|-------------|----------|---------|--------|------------|--------|
| 1 | [Description] | [Property] (1.5×) | +0.X | Low | **0.XX** | Optional |
| 2 | [Description] | [Property] (1.0×) | +0.X | Medium | **0.XX** | Optional |

**Legend**: Efficiency = (Weight × Δ Score) ÷ Effort. Higher = better ROI.
**Estimated Final Score**: X.X/10 (if all implemented)

**Would you like to:**
1. **Skip all** - The test suite is already production-ready
2. **Implement selected** - Choose specific improvements by number
3. **Implement all** - Proceed with all optional improvements
```

### For Good/Excellent Test Suites (< 9.0)

For test suites below the exemplary threshold, present improvements as recommendations **sorted by efficiency**:

```markdown
## 📋 Recommended Improvements (Sorted by Efficiency)

**Current Farley Score: X.X/10 ([Rating])**

| # | Improvement | Property | Δ Score | Effort | Efficiency | Skill |
|---|-------------|----------|---------|--------|------------|-------|
| 1 | [Description] | [Property] (1.5×) | +0.X | Low | **0.XX** | [TDD/Refactoring] |
| 2 | [Description] | [Property] (1.0×) | +0.X | Medium | **0.XX** | [TDD/Refactoring] |

**Legend**: Efficiency = (Weight × Δ Score) ÷ Effort. Higher = better ROI.
**Estimated Final Score**: X.X/10 (if all implemented)

Shall I proceed with the implementation plan?
```

### Property Thresholds for Skip Recommendations

Reference these thresholds when marking individual improvements as "Optional" vs "Recommended":

| Property | Optional When | Justification |
|----------|---------------|---------------|
| Understandable | Score ≥ 9 | Tests read like specifications |
| Maintainable | Score ≥ 9 | Proper abstractions in place |
| Repeatable | Score ≥ 9 | Tests are deterministic |
| Atomic | Score ≥ 9 | Tests are completely isolated |
| Necessary | Score ≥ 9 | Every test adds value |
| Granular | Score ≥ 9 | Each test asserts one thing |
| Fast | Score ≥ 8 | Tests execute quickly (0.75× weight) |
| First (TDD) | Score ≥ 8 | Clear test-first evidence (harder to assess) |

---

## Step 4: Create Implementation Plan

**After user confirms** which improvements to implement, create a detailed task plan with skill assignments.

### Skill Assignment Rules

| Task Type | Skill | When to Use |
|-----------|-------|-------------|
| Adding new code | **TDD** | New helper methods, new test utilities |
| Modifying existing code | **Refactoring** | Consolidating tests, updating assertions |
| Adding edge case comments | **testing** | Comments documenting boundary conditions, branch coverage, behavior-driven edge cases |
| Adding other test comments | **test-design-reviewer** | Docstrings, test documentation, clarity improvements using Farley's Understandable property |
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

**For edge case comments (testing skill):**
1. 📖 **ANALYZE**: Identify boundary conditions, branch coverage gaps, and behavior-driven edge cases
2. ✏️ **DOCUMENT**: Add comments explaining edge case rationale and expected behavior
3. ✅ **VERIFY**: Run tests to confirm no syntax errors
4. 💾 **COMMIT**: Save edge case documentation

**For other test comments (test-design-reviewer skill):**
1. 📖 **ANALYZE**: Use Farley's Understandable property to identify clarity needs
2. ✏️ **DOCUMENT**: Add docstrings and comments explaining WHY, not just WHAT
3. ✅ **VERIFY**: Run tests to confirm no syntax errors
4. 💾 **COMMIT**: Save documentation changes

---

## Step 5: Execute the Plan

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

### Pushing Changes

After completing all improvements, push the test changes but **do NOT push the `.agents/skills/` directory**:

```bash
# Verify .agents/skills is untracked (not staged)
git status

# Push only the committed test improvements
git push origin <branch-name>
```

**Important**: The skills added to `.agents/skills/` during this workflow are workspace-local tools. They should NOT be committed to the repository being improved. If accidentally staged, unstage them:

```bash
# If .agents/skills was accidentally staged, unstage it
git reset HEAD .agents/skills/

# Or add to .gitignore if not already present
echo ".agents/skills/" >> .gitignore
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
| 3 | Present improvements table | test-design-reviewer | **Yes** (always) |
| 4 | Create implementation plan | TDD + Refactoring | No (after user confirms) |
| 5 | Execute plan with commits | TDD + Refactoring | No |
| 6 | Re-run in new conversation | test-improvement-workflow | No |

---

## Quick Start

When this workflow is invoked, automatically:

1. **Audit** all test files using test-design-reviewer
2. **Calculate Efficiency Scores** for each improvement using the formula:
   ```
   Efficiency = (Property Weight × Δ Score) ÷ Effort Value
   ```
3. **Sort improvements by Efficiency** (highest first), adjusting for dependencies
4. **Present** the prioritized improvements table with Efficiency column populated
5. **Plan** implementation with TDD/Refactoring skill assignments (after user confirms)
6. **Execute** the plan, committing after each phase
7. **Recommend** re-evaluation in a new conversation

**CRITICAL**: Steps 2-4 ensure the user sees the most efficient fixes first. Do not skip the efficiency calculation.

