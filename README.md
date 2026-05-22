# 2.3 Unidirectional Data Flow and Conditional Rendering

## Lesson Overview

This lesson refactors the CRM app built in Lesson 2.2 into focused, composable components wired together through React's unidirectional data model. Learners lift shared state to a common ancestor, pass callbacks for child-to-parent communication, and add a search bar and a customer detail panel. The lesson also covers the three main patterns for conditional rendering and the distinction between state and derived state.

## Dependencies

- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assignment](./assignment.md)

## Lesson Objectives

- Explain unidirectional data flow and lift state to a common ancestor to share it across components
- Pass callbacks as props to enable child-to-parent communication without breaking the one-way data model
- Render UI conditionally using early returns, ternary expressions, and logical `&&`, and distinguish state from derived state

## Lesson Plan

| Duration  | What                                       | How or Why                                                                                                    |
| --------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| 10 min    | Warm up and recap                          | Recap Lesson 2.2: useState and event handling; show the monolithic App.jsx as motivation for today's refactor |
| 20 min    | Unidirectional data flow and lifting state | Slides: one-way data model diagram, why props are read-only, lifting state to a common ancestor               |
| 15 min    | Child-to-parent communication              | Slides: passing callbacks as props, handler vs setter pattern, state vs derived state                         |
| 10 min    | Conditional rendering patterns             | Slides: early return, ternary expression, logical `&&`, and the falsy-number gotcha                           |
| 5 min     | Break                                      |                                                                                                               |
| 10 min    | Code-along: starting point review          | Walk through the Lesson 2.2 end state; map out today's target component tree                                  |
| 20 min    | Code-along: extract CustomerCard           | Lift customer data up, pass it down as props, wire a delete callback from child to parent                     |
| 25 min    | Code-along: search bar with lifted state   | Add a search input to App, lift query state, derive the filtered list without duplicating state               |
| 5 min     | Break                                      |                                                                                                               |
| 25 min    | Code-along: customer detail panel          | Add selection state to App; conditionally render a detail panel using the three learned patterns              |
| 15 min    | Wrap up and Q&A                            | Recap learning objectives, address common pitfalls, preview Lesson 2.5                                        |
| **Total** |                                            | **160 min — allows ~20 min buffer for questions and pacing**                                                  |
