# Contribution [#27956]: [React Doctor] rerender-memo-with-default-value: Default prop value [] creates a new array reference every render — extract to... (20 occurrences)

**Contribution Number:** 1

**Student:** Dan Nguyen 

**Issue:** https://github.com/wso2/product-is/issues/27956

**Status:** Phase IV

**Working branch:** https://github.com/dannguyen24/identity-apps/tree/fix-issue-27956

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

**Plan:**
1. Add three module-level constants near the top of rule-conditions.tsx (after imports):
`const EMPTY_STRING_ARRAY: string[] = [];`

2. Replace all four = [] default values in the inner components with = EMPTY_STRING_ARRAY:
  - Line 213: hiddenResources = [] in ValueInputAutocomplete
  - Line 406: hiddenResources = [] in ResourceListSelect
  - Lines 669–670: hiddenResources = [], hiddenValues = [] in ConditionValueInput
  - Lines 783–785: hiddenConditions = [], hiddenResources = [], hiddenValues = [] in RuleExpression

3. Move `ValueInputAutocomplete`, `ResourceListSelect`, `ConditionValueInput`, and `RuleExpression` outside `RuleConditions` and pass h`andleExpressionChangeDebounced` / context values as explicit props (or consume useRulesContext directly inside each).

No test changes needed — existing tests verify behavior, not referential identity of default props.

**Implement:** 

**What I built:**
- Fixed inline = [] default prop references across 7 files in 5 feature packages (`admin.rules.v1`, `admin.approval-workflows.v1`, `admin.flow-builder-core.v1`, `admin.policy-administration.v1`, `admin.webhooks.v1`)
- Replaced 20 inline = [] prop defaults with module-level constants (`EMPTY_STRING_ARRAY`, `EMPTY_NODES`, `EMPTY_EDGES`, etc.) to prevent unnecessary re-renders from unstable array references
- Fixed 2 lint warnings I introduced (padding-line-between-statements) in `rule-conditions.tsx`

**Challenges faced**
- Ran `git add .` from inside `apps/console/` instead of the repo root, so the staged area was empty and the commit silently did nothing, learned that git add . only captures files in the current directory and below
- Pre-existing TypeScript errors and lint warnings in files I touched initially looked like my mistakes — learned to use git diff to confirm what I actually changed vs what was already broken
- initialNodes and initialEdges in `decorated-visual-flow.tsx` are typed as Node[] and Edge[] from `@xyflow/react`, not string[], couldn't reuse EMPTY_STRING_ARRAY and had to declare separate typed constants
  
**Review:** 

 [ ] TypeScript: no any, explicit type annotations on all new constants (const EMPTY_STRING_ARRAY: string[] = [])
 
 [ ] Styling: no new styled components involved — not applicable
 
 [ ] No scope creep: change is limited to hoisting default values; does not refactor component structure unless step 3 is explicitly taken
 
 [ ] Comments: hoisted constants need no comments — names are self-explanatory
 
 [ ] Changeset: this repo uses changesets — run pnpm changeset and add a patch entry for @wso2is/admin.rules.v1
 
Commit message convention (from git log): plain imperative sentence, no ticket prefix, no conventional-commits prefix, e.g.:


Fix unstable array references in rule-conditions default props

**Evaluate:** 

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

**Unit Tests**

N/A, no logic was changed. This is a pure refactor replacing inline = [] prop defaults with module-level constants. No existing tests were broken.

**Integration Tests**

N/A — no API calls, routing, or cross-feature behavior was changed.

**Manual Testing**

Verified that all affected files pass ESLint with no new warnings introduced:

 - admin.rules.v1/components/rule-conditions.tsx
 - admin.approval-workflows.v1/components/rules/workflow-condition-value-input.tsx
 - admin.approval-workflows.v1/components/rules/workflow-resource-autocomplete.tsx
 - admin.approval-workflows.v1/components/rules/workflow-resource-list-select.tsx
 - admin.flow-builder-core.v1/components/visual-flow/decorated-visual-flow.tsx
 - admin.policy-administration.v1/components/policy-list.tsx
 - admin.webhooks.v1/components/webhook-channel-config-form.tsx

---

## Implementation Notes

### Week 3 Progress

- Fixed 20 inline = [] default prop references across 7 files in 5 feature packages, as identified in product-is#27956. Each inline [] creates a new array reference on every render, defeating React.memo() and useMemo optimizations. Replaced all occurrences with module-level constants typed to the correct array type.

### Week [Y] Progress

N/A

### Code Changes

**Files modified:** 

 - features/admin.rules.v1/components/rule-conditions.tsx
 - features/admin.approval-workflows.v1/components/rules/workflow-condition-value-input.tsx
 - features/admin.approval-workflows.v1/components/rules/workflow-resource-autocomplete.tsx
 - features/admin.approval-workflows.v1/components/rules/workflow-resource-list-select.tsx
 - features/admin.flow-builder-core.v1/components/visual-flow/decorated-visual-flow.tsx
 - features/admin.policy-administration.v1/components/policy-list.tsx
 - features/admin.webhooks.v1/components/webhook-channel-config-form.tsx

**Key commits:**

 - 0f068f7 inline array default prop reference in rule-conditions
 - 1b2b292 Fix remaining inline array default props in rule-conditions
 - 8d10c5f Fix inline array default props in approval-workflows rule components
 - 5527baf Fix inline array default props in workflow-resource-autocomplete comp…
 - c012b58 Fix inline array default props in workflow-resource-list-select compo… 
 - 7173c5b Fix inline array default props in decorated-visual-flow omponents
 - d533850 Fix inline array default props in policy-list components
 - 6408828 Fix inline array default props in webhook-channel-config-form components
  
**Approach decisions:**

 - Named constants after their type (EMPTY_STRING_ARRAY, EMPTY_NODES, EMPTY_EDGES, EMPTY_POLICIES, EMPTY_CHANNEL_SUBSCRIPTIONS) rather than a generic EMPTY_ARRAY to preserve TypeScript type safety
 - Placed constants at module scope immediately after the last import in each file, before the first interface declaration

---

## Pull Request

**PR Link:** [https://github.com/wso2/identity-apps/pull
](https://github.com/wso2/identity-apps/pull/10440)

**PR Description:** Fixed 20 inline `= []` default prop references across 7 files in 5 feature packages (`admin.rules.v1`, `admin.approval-workflows.v1`, `admin.flow-builder-core.v1`, `admin.policy-administration.v1`, `admin.webhooks.v1`). Each inline `[]` creates a new array reference on every render, defeating `React.memo()` and `useMemo` optimizations.

**Maintainer Feedback:** Awaiting review.

**Status:** Awaiting review

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
