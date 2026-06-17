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

Default prop values like items = [] create a new array reference on every render, breaking React.memo — the component re-renders even when props haven't semantically changed.

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
