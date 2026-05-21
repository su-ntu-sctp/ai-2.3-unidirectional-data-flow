# Pre-Reading: Lesson 2.3 — Unidirectional Data Flow and Conditional Rendering

Timebox **2–3 hours** across these resources before the lesson. You don't need to memorise everything — focus on building a mental model so the hands-on lab clicks faster.

---

## 1. Thinking in React

**Read (20 min)**

- [Thinking in React](https://react.dev/learn/thinking-in-react) — The official React guide to breaking UI into components and deciding where state lives; pay special attention to Step 3 (minimal state) and Step 4 (where state should live).

**Key idea to take away:** Before writing any code, ask "which component should own this piece of state?" — the answer is always the closest common ancestor of every component that needs it.

---

## 2. Sharing State Between Components (Lifting State Up)

**Read (15 min)**

- [Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) — Walks through exactly the pattern you'll practise in the lab: moving state up to a parent so two sibling components can share it.

**Quick check:** After reading, can you answer — if `ComponentA` and `ComponentB` are siblings and both need the same piece of state, where should that state live?

---

## 3. Conditional Rendering

**Read (15 min)**

- [Conditional Rendering](https://react.dev/learn/conditional-rendering) — Covers all three patterns used in this lesson: `if` statements, ternary expressions (`? :`), and logical `&&`. Focus on when to use each.

**Key ideas:**

- Use `if` / early return when an entire component should not render at all
- Use a ternary when you need to choose between two JSX outputs
- Use `&&` when you only need to show something — or show nothing

---

## 4. Passing Data Deeply vs. Lifting State Up

**Read (10 min)**

- [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component) — A quick refresher on props; focus on the section about passing functions as props, which is how child components communicate back to parents.

**Key idea to take away:** Props flow down; callbacks flow up. A child never modifies a parent's state directly — it calls a function the parent passed down.

---

## 5. Choosing the State Structure

**Read (15 min)**

- [Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure) — Read the "Avoid redundant state" section in particular; this is the principle behind why `filteredCustomers` should be computed during render rather than stored as a second state variable.

**Quick check:** Given this code, what's the problem?

```jsx
const [items, setItems] = useState(initialItems);
const [filteredItems, setFilteredItems] = useState(initialItems);
const [query, setQuery] = useState("");
```

Could you rewrite it to remove the redundant state?

---

## Reflection (5 min)

Before the lesson, write down answers to these three questions (a notebook or a text file is fine):

1. If a search bar and a results list are sibling components, where should the search term state live — and why?
2. What is the difference between storing `filteredCustomers` as state versus computing it during render?
3. What is one thing you're still unclear about after the pre-reading?

Bring question 3 to class.
