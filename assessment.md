# Assessment / Quiz

## Overview

- **Lesson:** Unidirectional Data Flow and Conditional Rendering / 2.3
- **Format:** 30 questions (mix MCQ / True–False)
- **Time:** ~30 minutes
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

---

### Q11 (True/False)

In React, state is private to the component that owns it. Sibling components cannot directly read each other's state.

A - True

B - False

---

### Q12

Which pattern does `CustomerDetail` use when `customer` is `null`?

```jsx
function CustomerDetail({ customer }) {
  if (!customer) return <p>Select a customer.</p>;
  return <div><h2>{customer.firstName}</h2></div>;
}
```

A - Ternary expression

B - Logical `&&` operator

C - Early return

D - Nullish coalescing

---

### Q13

You are building a `NotificationBadge` component that should only render when `count > 0`. Which implementation is correct?

A - `{count > 0 ? <span>{count}</span> : ""}`

B - `{count > 0 && <span>{count}</span>}`

C - `{count && <span>{count}</span>}`

D - Both A and B are correct

---

### Q14 (True/False)

When a `SearchBar` component owns its own `searchTerm` state, a sibling `CustomerList` component can read that state directly through the component tree.

A - True

B - False

---

### Q15

In the CRM app, `CustomerCard` receives `onSelect` and calls `onSelect(customer)` when the View button is clicked. What does this pattern demonstrate?

A - Derived state

B - Two-way data binding

C - Child-to-parent communication via callback props

D - Direct state mutation

---

### Q16

A `CustomerCard` is updated to receive an `isSelected` prop. Which JSX correctly applies a CSS class when the card is selected?

A - `<div className="customer-card" isSelected={isSelected}>`

B - `<div className={isSelected ? "customer-card selected" : "customer-card"}>`

C - `<div className="customer-card" style={isSelected}>`

D - `<div class={isSelected && "selected"}>`

---

### Q17

In `App.jsx`, `handleDeleteCustomer` is updated to also clear the selected customer. Which implementation is correct?

```jsx
const handleDeleteCustomer = (customerId) => {
  setCustomers(customers.filter((c) => c.id !== customerId));
  // What goes here?
};
```

A - `setSelectedCustomer(customerId)`

B - `if (selectedCustomer.id === customerId) setSelectedCustomer(null)`

C - `if (selectedCustomer?.id === customerId) setSelectedCustomer(null)`

D - `selectedCustomer = null`

---

### Q18 (True/False)

Derived state (values computed from existing state during render) can go out of sync with the state it depends on.

A - True

B - False

---

### Q19

`CustomerDetail` uses `{customer.yearOfBirth && <p>Year of Birth: {customer.yearOfBirth}</p>}`. What happens when `yearOfBirth` is `0`?

A - The `<p>` renders correctly with the value `0`

B - The `<p>` is hidden because `0` is falsy

C - React throws an error

D - The `<p>` renders as an empty element

---

### Q20

A learner writes:

```jsx
const [filteredCustomers, setFilteredCustomers] = useState([]);

useEffect(() => {
  setFilteredCustomers(customers.filter(...));
}, [customers, searchTerm]);
```

What is the main problem with this approach compared to computing `filteredCustomers` during render?

A - `useEffect` is not allowed to call state setters

B - There are now two sources of truth that must be kept in sync, and there is an extra render each time the effect fires

C - The filter logic runs on every keystroke regardless

D - `filteredCustomers` will always be an empty array on the first render

---

### Q21

Which of the following shows the correct way to pass `setSearchTerm` to `SearchBar` as an `onSearch` callback?

A - `<SearchBar onSearch={() => setSearchTerm} />`

B - `<SearchBar onSearch={setSearchTerm()} />`

C - `<SearchBar onSearch={setSearchTerm} />`

D - `<SearchBar onSearch="setSearchTerm" />`

---

### Q22 (True/False)

The `||` operator used in JSX (e.g., `{customer.contactNo || "N/A"}`) is a form of conditional rendering.

A - True

B - False

---

### Q23

In `App.jsx`, after adding `selectedCustomer` state, a customer is deleted from the list. What must the developer also handle?

A - Re-fetch the customer list from the server

B - Clear `selectedCustomer` if the deleted customer is currently selected

C - Reset `searchTerm` to empty string

D - Force a full page reload

---

### Q24

A `ToggleFormButton` component receives `showForm` and `onToggle` as props. Which JSX correctly renders the button label?

A - `<button onClick={onToggle}>{showForm === true ? "Cancel" : "Add Customer"}</button>`

B - `<button onClick={() => onToggle}>{showForm ? "Cancel" : "Add Customer"}</button>`

C - `<button onClick={onToggle}>{showForm ? "Cancel" : "Add Customer"}</button>`

D - `<button onClick={onToggle()}>{showForm && "Cancel"}</button>`

---

### Q25

Consider the following component tree:

```
App
├── SearchBar
├── CustomerDetail
└── CustomerList
    └── CustomerCard (×N)
```

A developer wants `CustomerCard` to update the `selectedCustomer` state in `App`. Which is the minimal, correct approach?

A - Lift `selectedCustomer` state to `App` and pass `setSelectedCustomer` directly as a prop through `CustomerList` to `CustomerCard`

B - Lift `selectedCustomer` state to `App`, define an `handleSelectCustomer` handler, and pass it as a prop through `CustomerList` to `CustomerCard`

C - Move `selectedCustomer` state into `CustomerList` so it is closer to `CustomerCard`

D - Use a global variable outside of React to store the selected customer

---

### Q26

Which statement best explains why `CustomerCard` receives `onDelete` with only the `id` parameter, while `onSelect` receives the full `customer` object?

A - `onDelete` is a built-in React event handler and must receive a primitive value

B - The delete handler only needs the identifier to filter the array; the select handler needs all fields to populate the detail panel

C - Passing the full object to `onDelete` would mutate state

D - `id` is the only property guaranteed to exist on a customer

---

### Q27 (True/False)

Two sibling components can share state by storing it in a common parent, passing the value down as a prop, and passing a callback for updates, without using any global state library.

A - True

B - False

---

### Q28

A developer writes a `CustomerList` component that renders a card for each customer. The `customers` array is empty. Which of the following most accurately describes the output of `{customers.map(c => <CustomerCard key={c.id} customer={c} />)}`?

A - React throws an error because you cannot call `.map()` on an empty array

B - Nothing is rendered; an empty array produces no output in JSX

C - A single empty `<div>` is rendered

D - React renders `undefined`

---

### Q29

Examine these two implementations of an empty-state message:

```jsx
// A
{filteredCustomers.length === 0 && (
  <p>{searchTerm ? "No results." : "No customers yet."}</p>
)}

// B
{filteredCustomers.length === 0 ? (
  <p>{searchTerm ? "No results." : "No customers yet."}</p>
) : (
  <div className="customers">...</div>
)}
```

When would you prefer implementation B over A?

A - When you want to avoid rendering the customer grid at all if the list is empty

B - When the empty-state message contains a number

C - When `filteredCustomers` might be `undefined`

D - There is no practical difference between A and B

---

### Q30

A learner refactors the CRM so that `CustomerCard` now renders its own local `isSelected` state instead of receiving `isSelected` as a prop. What problem does this introduce?

A - The card can no longer be deleted

B - The `isSelected` state in `CustomerCard` will be out of sync with `selectedCustomer` in `App`, causing incorrect visual highlighting

C - React will throw an error because only one component can manage selection state

D - The `onSelect` callback will stop working

---
