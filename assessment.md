# Assessment / Quiz

## Overview

- **Lesson:** Unidirectional Data Flow and Conditional Rendering / 2.3
- **Format:** 10 questions (mix MCQ / True–False)
- **Time:** ~10–15 minutes
- **Scoring:** 1 point each (unless stated)

## Questions

### Q1 (True/False)

In React, data flows in one direction: from parent components down to child components via props.

A - True

B - False

---

### Q2

Which term describes the React pattern where state is moved to the closest common ancestor so that sibling components can share it?

A - Prop drilling

B - Lifting state up

C - Context injection

D - State delegation

---

### Q3

What does "unidirectional data flow" mean in React?

A - Components can only receive data from a single prop

B - Data flows down from parent to child via props; child components communicate up by calling callback functions

C - State can only be updated in one place per component

D - Components render in a single pass from top to bottom

---

### Q4 (True/False)

A child component can directly modify the props it receives, for example: `props.customer.name = "Alice"`.

A - True

B - False

---

### Q5

Which conditional rendering pattern is best suited to showing a completely different UI tree when a condition is true vs. false?

A - Logical `&&` operator

B - Ternary expression (`condition ? a : b`)

C - `if` statement / early return

D - Template literals

---

### Q6

What is the main advantage of computing `filteredCustomers` directly during render (as derived state) rather than storing it in a separate `useState`?

A - It avoids re-rendering the component

B - It eliminates the need to pass props to child components

C - It ensures the filtered list is always in sync with the source array and search term without manual updates

D - It makes the filter logic run faster

---

### Q7 (True/False)

Passing the entire `setCustomers` state setter function as a prop to a child component is a recommended React practice.

A - True

B - False

---

### Q8

In the CRM app, `SearchBar` and `CustomerList` are sibling components. Where should the `searchTerm` state live so both can access it?

A - Inside `SearchBar`, since it owns the input

B - Inside `CustomerList`, since it uses the value

C - In the closest common ancestor component (e.g., `App`)

D - In a separate utility module outside the component tree

---

### Q9

Which of the following is the correct way to pass a delete handler to `CustomerCard` so the child can trigger deletion?

A - Call `handleDelete()` directly inside `CustomerCard`'s function body

B - Export `handleDelete` from `App.jsx` and import it inside `CustomerCard`

C - Pass `onDelete={handleDeleteCustomer}` as a prop and call `props.onDelete(customer.id)` inside the child

D - Store `handleDelete` in a `useState` and pass it down

---

### Q10

What will the following JSX render when `count` is `0`?

```jsx
<div>{count && <p>Items found</p>}</div>
```

A - Nothing: the `<p>` is hidden

B - An empty `<div>`

C - `0` rendered as text inside the `<div>`

D - An error: `&&` cannot be used with numbers
