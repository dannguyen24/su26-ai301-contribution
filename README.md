# Contribution [#27956]: [React Doctor] rerender-memo-with-default-value: Default prop value [] creates a new array reference every render — extract to... (20 occurrences)

**Contribution Number:** 1  
**Student:** Dan Nguyen 
**Issue:** https://github.com/wso2/product-is/issues/27956
**Status:** Phase I

---

## Why I Chose This Issue
---

I'm a computer science student with hands-on experience building React applications, and I'm interested in contributing to WSO2 Identity Server as a way to explore the identity and authentication domain. This issue caught my attention because it targets the `react-doctor/rerender-memo-with-default-value` rule, a performance pattern I'm familiar with, where inline default prop values like [] or {} create new object references on every render, defeating the purpose of React.memo. Fixing these 20 occurrences is a focused, well-scoped task that lets me get comfortable with the frontend codebase before moving on to more complex authentication-related features. 

## Understanding the Issue

### Problem Description

Default prop values like items = [] create a new array reference on every render, breaking React.memo, the component re-renders even when props haven't semantically changed.

### Expected Behavior

Components wrapped in React.memo should only re-render when props actually change. An empty array default like items = [] used repeatedly should be the same reference, so if the parent re-renders but passes no new items, the memoized child stays unchanged.

### Current Behavior

Every render creates a brand-new [] array reference as the default value. Since React.memo uses shallow (reference) equality, it sees a "new" array each time and re-renders the child component unnecessarily, even though the data is identical. Memoization is silently broken.


### Affected Components

No runtime error or crash. The only signal is the ESLint warning from the react-doctor plugin:
react-doctor/rerender-memo-with-default-value: default prop value `[]` creates a new array reference every render

---

## Reproduction Process

### Environment Setup

Clone the repository: 
`git clone https://github.com/wso2/identity-apps.git`
`cd identity-apps`
Install dependencies:
`pnpm install`


### Steps to Reproduce

1. Clone `wso2/identity-apps`
2. Run: `grep -rn "\w\+ = \[\]" features --include="*.tsx" | grep -v "const \|let \|var \|//"`
3. Observe 20+ matches where empty array literals are used as default prop values inside destructured parameters



### Reproduction Evidence

- **Commit showing reproduction:**
- https://github.com/dannguyen24/identity-apps/tree/fix-issue-27956
- **Screenshots/logs:**
 <img width="936" height="664" alt="Screenshot 2026-06-17 at 1 11 35 AM" src="https://github.com/user-attachments/assets/e7861cf1-30ec-49cd-959f-9b1d9909d989" />
- **My findings:**
  
- 255 files across features/ and modules/ contain the pattern — useMemo or useCallback combined with inline default values like ?? [] or ?? {}.

Heaviest concentrations:

admin.roles.v2 — multiple edit/wizard components
admin.registration-flow-builder.v1 — provider, core, extended-properties components
admin.ask-password-flow-builder.v1 — same pattern mirrored
admin.copilot.v1 — hooks and components
admin.organization-discovery.v1
The pattern being flagged — returning a new object/array reference as a fallback inside a memo:


// Bad — ?? [] creates a new array reference every render when left side is nullish
const items = useMemo(() => data?.items ?? [], [data]);

// Good — stable reference, no unnecessary rerenders
const EMPTY_ARRAY: MyType[] = [];
const items = useMemo(() => data?.items ?? EMPTY_ARRAY, [data]);
What the rule catches specifically — the issue isn't that useMemo itself rerenders, but that consumers of items (child components, other hooks) will see a new reference on every render even when the data hasn't changed, defeating the purpose of memoization.

---

## Solution Approach

### Analysis

The codebase pattern is module-level const EMPTY_ARRAY or const EMPTY_X: Type[] = [] — common in admin.roles.v2 and other features to stabilize default props.

### Proposed Solution

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Inline = [] defaults in prop destructuring defeat memoization by creating new references on every render.

**Match:** The codebase pattern is module-level const EMPTY_ARRAY or const EMPTY_X: Type[] = [] — common in admin.roles.v2 and other features to stabilize default props.

**Plan:** [Step-by-step implementation plan]
1. Add three module-level constants near the top of rule-conditions.tsx (after imports):
`const EMPTY_STRING_ARRAY: string[] = [];`

2. Replace all four = [] default values in the inner components with = EMPTY_STRING_ARRAY:

Line 213: hiddenResources = [] in ValueInputAutocomplete
Line 406: hiddenResources = [] in ResourceListSelect
Lines 669–670: hiddenResources = [], hiddenValues = [] in ConditionValueInput
Lines 783–785: hiddenConditions = [], hiddenResources = [], hiddenValues = [] in RuleExpression

3. Move `ValueInputAutocomplete`, `ResourceListSelect`, `ConditionValueInput`, and `RuleExpression` outside `RuleConditions` and pass h`andleExpressionChangeDebounced` / context values as explicit props (or consume useRulesContext directly inside each).

No test changes needed — existing tests verify behavior, not referential identity of default props.

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

Review checklist:

 [ ] No any types introduced
 [ ] All props explicitly typed
 [ ] FunctionComponent<...> + ReactElement return signature maintained
 [ ] No new inline [] or {} in JSX or destructuring defaults

 [ ] TypeScript: no any, explicit type annotations on all new constants (const EMPTY_STRING_ARRAY: string[] = [])
 [ ] Styling: no new styled components involved — not applicable
 [ ] No scope creep: change is limited to hoisting default values; does not refactor component structure unless step 3 is explicitly taken
 [ ] Comments: hoisted constants need no comments — names are self-explanatory
 [ ] Changeset: this repo uses changesets — run pnpm changeset and add a patch entry for @wso2is/admin.rules.v1
 
Commit message convention (from git log): plain imperative sentence, no ticket prefix, no conventional-commits prefix, e.g.:


Fix unstable array references in rule-conditions default props

**Evaluate:** [How will you verify it works?]

Existing test infrastructure: `features/admin.rules.v1/__tests__/` exists but currently contains only `__mocks__/data.ts`, no component tests exist yet.

Per `docs/testing/UNIT_TESTING.md`, writing unit tests when working on existing features is mandatory. The fix therefore requires a new test file.

What to create: `features/admin.rules.v1/components/__tests__/rule-conditions.test.tsx`

Minimum test cases required:

`<RuleConditions />` renders without exploding (smoke test, uses data-componentid="rules-component-condition")
Renders condition expressions when rule.rules is populated (uses the existing sampleRuleExecuteInstance mock from __mocks__/data.ts)
Renders without errors when hiddenConditions, hiddenResources, hiddenValues are not passed — this is the regression test that directly validates the fix (previously these would silently create new [] references; a stable render confirms the constant defaults are used)
Run with:


`npx vitest run features/admin.rules.v1/components/__tests__/rule-conditions.test.tsx`


---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
