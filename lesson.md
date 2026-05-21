# Lesson 2.3: Unidirectional Data Flow and Conditional Rendering

## Overview

- **Duration:** ~2 hours (hands-on lab)
- **Prerequisites:** Lesson 2.2: State Management and Event Handling in React

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Explain** unidirectional data flow and lift state to a common ancestor
2. **Pass** callbacks as props for child-to-parent communication
3. **Render** UI conditionally using `if`, ternary expressions, and logical `&&`
4. **Distinguish** between state and derived state

## Introduction

In Lesson 2.2 you built a working CRM app, but all the state and UI live inside a single `App.jsx`. Today you'll refactor it into focused components, wire them together using unidirectional data flow, and add a search bar and a customer detail panel. Along the way you'll see why React's one-way data model makes apps predictable, and you'll learn how to render different UI based on application state.

---

## Part 1: Starting Point and Recap (10 minutes)

### Where We Left Off

At the end of Lesson 2.2, your `simple-crm-web` project has:

- `App.jsx`: all state and UI in a single component
- `mockData.js`: customer data and ID generator
- Add Customer form with controlled inputs
- Delete button on each customer card

Open your project and confirm it runs:

```bash
cd simple-crm-web
npm run dev
```

Navigate to `http://localhost:5173` and verify the customer list and add form are working.

### Today's Goal

We'll refactor `App.jsx` into multiple focused components, add a search bar, and add a customer detail panel, all using unidirectional data flow and conditional rendering.

**Final component tree:**

```
App
├── SearchBar
├── CustomerDetail
└── CustomerList
    └── CustomerCard (×N)
```

---

## Part 2: Extract CustomerCard (20 minutes)

Right now, the customer card JSX is embedded directly inside `App.jsx`. That works, but as the card grows (more fields, more buttons) the file becomes hard to navigate. We'll move the card into its own component.

### Step 1: Create `CustomerCard.jsx`

Create a new file `src/CustomerCard.jsx`. We define the component to accept two props: `customer` (the data) and `onDelete` (a function to call when the Delete button is clicked).

```jsx
// src/CustomerCard.jsx
function CustomerCard({ customer, onDelete }) {
  return (
    <div className="customer-card">
      <h3>
        {customer.firstName} {customer.lastName}
      </h3>
      <p>Email: {customer.email}</p>
      <p>Phone: {customer.contactNo || "N/A"}</p>
      <p>Job: {customer.jobTitle || "N/A"}</p>
      <button onClick={() => onDelete(customer.id)}>Delete</button>
    </div>
  );
}

export default CustomerCard;
```

Notice that `CustomerCard` does **not** call `setCustomers`; it has no access to that state. It only calls `onDelete(customer.id)`, and the parent decides what happens next. This is the core of unidirectional data flow: **data flows down via props, events flow up via callbacks**.

> **Common mistake:** Passing the entire `setCustomers` function down to `CustomerCard`. That gives the child unrestricted write access to state it shouldn't own. Pass a specific handler function instead.

### Step 2: Update `App.jsx`

Import `CustomerCard` and replace the inline card JSX with `<CustomerCard>` inside the `.map()`:

```jsx
// src/App.jsx (updated)
import { useState } from "react";
import { mockCustomers, generateCustomerId } from "./mockData";
import CustomerCard from "./CustomerCard";
import "./App.css";

function App() {
  const [customers, setCustomers] = useState(mockCustomers);
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [email, setEmail] = useState("");

  const handleAddCustomer = (e) => {
    e.preventDefault();
    const newCustomer = {
      id: generateCustomerId(),
      firstName,
      lastName,
      email,
      contactNo: null,
      jobTitle: null,
      yearOfBirth: null,
    };
    setCustomers([...customers, newCustomer]);
    setFirstName("");
    setLastName("");
    setEmail("");
  };

  const handleDeleteCustomer = (customerId) => {
    setCustomers(customers.filter((c) => c.id !== customerId));
  };

  return (
    <div className="simple-crm">
      <h1>Simple CRM</h1>

      <form onSubmit={handleAddCustomer} className="add-customer-form">
        <h3>Add New Customer</h3>
        <input
          type="text"
          placeholder="First Name"
          value={firstName}
          onChange={(e) => setFirstName(e.target.value)}
          required
        />
        <input
          type="text"
          placeholder="Last Name"
          value={lastName}
          onChange={(e) => setLastName(e.target.value)}
          required
        />
        <input
          type="email"
          placeholder="Email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
        <button type="submit">Add Customer</button>
      </form>

      <div className="customer-list">
        <h2>Customers ({customers.length})</h2>
        <div className="customers">
          {customers.map((customer) => (
            <CustomerCard
              key={customer.id}
              customer={customer}
              onDelete={handleDeleteCustomer}
            />
          ))}
        </div>
      </div>
    </div>
  );
}

export default App;
```

**Browser check:** The app should look and behave exactly as before. Add a customer, delete a customer; same behaviour, but the card is now its own component.

---

## Part 3: Add a Search Bar (30 minutes)

We want to filter the customer list by name or email as the user types. Let's build `SearchBar` the most natural-feeling way first, then discover why it doesn't work, and fix it.

### Step 1: Create `SearchBar.jsx` with its own state

Create `src/SearchBar.jsx`. Give it its own `searchTerm` state; that is the obvious first instinct:

```jsx
// src/SearchBar.jsx: first attempt
import { useState } from "react";

function SearchBar() {
  const [searchTerm, setSearchTerm] = useState("");

  return (
    <div className="search-bar">
      <input
        type="text"
        placeholder="Search by name or email..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
    </div>
  );
}

export default SearchBar;
```

### Step 2: Render it in `App.jsx`

Import and render `<SearchBar />` above the customer list:

```jsx
// src/App.jsx
import SearchBar from "./SearchBar";

// In the return:
<SearchBar />

<div className="customer-list">
  <h2>Customers ({customers.length})</h2>
  <div className="customers">
    {customers.map((customer) => (
      <CustomerCard
        key={customer.id}
        customer={customer}
        onDelete={handleDeleteCustomer}
      />
    ))}
  </div>
</div>
```

**Browser check:** The search input renders and you can type into it. So far so good.

### Step 3: Try to filter, and hit the wall

Now try to use `searchTerm` in `App.jsx` to filter the list:

```jsx
// src/App.jsx: this won't work
const filteredCustomers = customers.filter(
  (c) => c.firstName.toLowerCase().includes(searchTerm.toLowerCase())
);
```

`searchTerm` is not defined in `App`. It lives inside `SearchBar`. `App` has no way to read it.

This is the fundamental constraint of React's data model: **state is private to the component that owns it**. `SearchBar` and the customer list are siblings; neither can reach into the other.

### Step 4: Lift the state up to `App`

The fix is to move `searchTerm` to the closest component that needs to share it, which is their common ancestor, `App`. Then `App` passes the value down to `SearchBar` as a prop, and `SearchBar` calls a callback to update it.

First, update `SearchBar` to receive its value and change handler as props instead of owning them:

```jsx
// src/SearchBar.jsx: updated
function SearchBar({ searchTerm, onSearch }) {
  return (
    <div className="search-bar">
      <input
        type="text"
        placeholder="Search by name or email..."
        value={searchTerm}
        onChange={(e) => onSearch(e.target.value)}
      />
    </div>
  );
}

export default SearchBar;
```

Then move the state into `App`, compute `filteredCustomers` from it, and pass both down:

```jsx
// src/App.jsx: add searchTerm state and filteredCustomers
const [searchTerm, setSearchTerm] = useState("");

// Derived: computed on every render, always in sync
const filteredCustomers = customers.filter(
  (c) =>
    c.firstName.toLowerCase().includes(searchTerm.toLowerCase()) ||
    c.lastName.toLowerCase().includes(searchTerm.toLowerCase()) ||
    c.email.toLowerCase().includes(searchTerm.toLowerCase()),
);

// In the return:
<SearchBar searchTerm={searchTerm} onSearch={setSearchTerm} />

<div className="customer-list">
  <h2>Customers ({filteredCustomers.length})</h2>
  <div className="customers">
    {filteredCustomers.map((customer) => (
      <CustomerCard
        key={customer.id}
        customer={customer}
        onDelete={handleDeleteCustomer}
      />
    ))}
  </div>
</div>
```

> **Why not `useState` for `filteredCustomers`?** If you store it separately, you now have two sources of truth. Every time `customers` or `searchTerm` changes you'd have to keep them in sync manually, a common source of bugs. Computing it during render is cheaper than it looks and always correct.

**Browser check:** Typing in the search box now filters the displayed cards in real time. The customer count updates to reflect the filtered results. Deleting a customer still works correctly.

This pattern of moving state to a common ancestor so siblings can share it is called **lifting state up**. You'll use it constantly in React.

---

### Activity 1: Empty State Message (15 minutes)

You've seen how to render a list conditionally. Now apply it yourself.

**Task:** When no customers match the search term, replace the card grid with a helpful message. When the list is entirely empty (all deleted), show a different message.

**Hints:**
1. Check `filteredCustomers.length === 0` before rendering the grid
2. Use a ternary (`condition ? a : b`) to choose between the empty message and the grid
3. To show a different message depending on whether there's a search term active, you can nest another ternary inside the empty message: `{searchTerm ? "..." : "..."}`
4. The empty state and the grid are siblings; wrap them both inside the existing `<div className="customer-list">` element

**Expected result:**
- Searching for something with no matches → "No customers match your search."
- Deleting all customers with no search active → "No customers yet. Add one above!"
- Results exist → card grid shows as normal

<details>
<summary>Reference solution</summary>

```jsx
<div className="customer-list">
  <h2>Customers ({filteredCustomers.length})</h2>

  {filteredCustomers.length === 0 ? (
    <p className="empty-state">
      {searchTerm
        ? "No customers match your search."
        : "No customers yet. Add one above!"}
    </p>
  ) : (
    <div className="customers">
      {filteredCustomers.map((customer) => (
        <CustomerCard
          key={customer.id}
          customer={customer}
          onDelete={handleDeleteCustomer}
        />
      ))}
    </div>
  )}
</div>
```

</details>

---

## Part 4: Customer Detail Panel (30 minutes)

Now we'll add a `CustomerDetail` component that shows full information about a selected customer. Clicking a card selects it; the panel updates on the right.

This introduces a new pattern: **selection state**. The selected customer is owned by `App` (it needs to pass it to `CustomerDetail`), and `CustomerCard` raises an event when clicked.

### Step 1: Create `CustomerDetail.jsx`

The component handles two cases: no customer selected (renders a placeholder) and a customer selected (renders its fields).

```jsx
// src/CustomerDetail.jsx
function CustomerDetail({ customer }) {
  if (!customer) {
    return (
      <div className="customer-detail">
        <p className="empty-state">Select a customer to view details.</p>
      </div>
    );
  }

  return (
    <div className="customer-detail">
      <h2>
        {customer.firstName} {customer.lastName}
      </h2>
      <p>
        <strong>Email:</strong> {customer.email}
      </p>
      <p>
        <strong>Phone:</strong> {customer.contactNo || "N/A"}
      </p>
      <p>
        <strong>Job Title:</strong> {customer.jobTitle || "N/A"}
      </p>
      {customer.yearOfBirth && (
        <p>
          <strong>Year of Birth:</strong> {customer.yearOfBirth}
        </p>
      )}
    </div>
  );
}

export default CustomerDetail;
```

Notice we use three different conditional rendering patterns here:
- `if (!customer) return ...`: early return for the null case
- `|| "N/A"`: fallback for optional string fields
- `{customer.yearOfBirth && <p>...}`: only renders when the value exists

### Step 2: Update `CustomerCard` to support selection

Add `onSelect` and `isSelected` props. `onSelect` is called when the View button is clicked; `isSelected` controls a CSS class for visual feedback.

```jsx
// src/CustomerCard.jsx (updated)
function CustomerCard({ customer, onDelete, onSelect, isSelected }) {
  return (
    <div className={`customer-card ${isSelected ? "selected" : ""}`}>
      <h3>
        {customer.firstName} {customer.lastName}
      </h3>
      <p>Email: {customer.email}</p>
      <p>Phone: {customer.contactNo || "N/A"}</p>
      <p>Job: {customer.jobTitle || "N/A"}</p>
      <button onClick={() => onSelect(customer)}>View</button>
      <button onClick={() => onDelete(customer.id)}>Delete</button>
    </div>
  );
}

export default CustomerCard;
```

> **Why does `onSelect` receive the whole `customer` object, while `onDelete` only receives the `id`?** The detail panel needs all the fields, so we pass the full object. The delete handler only needs to identify which one to remove, so the `id` is enough. Pass the minimum needed.

### Step 3: Add `selectedCustomer` state to `App` and wire it up

```jsx
// src/App.jsx: add selectedCustomer state
import CustomerDetail from "./CustomerDetail";

const [selectedCustomer, setSelectedCustomer] = useState(null);
```

Update `handleDeleteCustomer` to clear the selection if the deleted customer is currently selected:

```jsx
const handleDeleteCustomer = (customerId) => {
  setCustomers(customers.filter((c) => c.id !== customerId));
  if (selectedCustomer?.id === customerId) {
    setSelectedCustomer(null);
  }
};
```

### Step 4: Update the layout

Wrap the customer panel and detail panel in a two-column flex layout:

```jsx
// src/App.jsx: updated return
import CustomerDetail from "./CustomerDetail";

return (
  <div className="simple-crm">
    <h1>Simple CRM</h1>

    <form onSubmit={handleAddCustomer} className="add-customer-form">
      {/* ... unchanged ... */}
    </form>

    <div className="crm-layout">
      <div className="customer-panel">
        <SearchBar searchTerm={searchTerm} onSearch={setSearchTerm} />

        <div className="customer-list">
          <h2>Customers ({filteredCustomers.length})</h2>

          {filteredCustomers.length === 0 ? (
            <p className="empty-state">
              {searchTerm
                ? "No customers match your search."
                : "No customers yet. Add one above!"}
            </p>
          ) : (
            <div className="customers">
              {filteredCustomers.map((customer) => (
                <CustomerCard
                  key={customer.id}
                  customer={customer}
                  onDelete={handleDeleteCustomer}
                  onSelect={setSelectedCustomer}
                  isSelected={selectedCustomer?.id === customer.id}
                />
              ))}
            </div>
          )}
        </div>
      </div>

      <CustomerDetail customer={selectedCustomer} />
    </div>
  </div>
);
```

Add the layout styles to `App.css`:

```css
/* App.css */
.crm-layout {
  display: flex;
  gap: 24px;
}

.customer-panel {
  flex: 1;
}

.customer-detail {
  width: 320px;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  background: #fff;
  align-self: flex-start;
}

.customer-card.selected {
  border: 2px solid #0070f3;
}
```

**Browser check:**
- The detail panel appears on the right showing "Select a customer to view details."
- Clicking "View" on a card populates the detail panel with that customer's information
- The selected card is visually highlighted
- Deleting the selected customer clears the detail panel

---

### Activity 2: Show/Hide the Add Customer Form (15 minutes)

Now apply what you know about boolean state and conditional rendering.

**Task:** Add a toggle button that shows or hides the Add Customer form. The button label should change based on the current state.

**Hints:**
1. Add `const [showForm, setShowForm] = useState(false)` to `App`
2. Use `{showForm && <form>...</form>}` to conditionally render the form
3. Add a button outside the form that calls `setShowForm(!showForm)`
4. Change the button label dynamically: `{showForm ? "Cancel" : "Add Customer"}`

**Expected result:**
- Page loads with the form hidden and an "Add Customer" button visible
- Clicking "Add Customer" reveals the form and changes the button label to "Cancel"
- Clicking "Cancel" hides the form again

<details>
<summary>Reference solution</summary>

```jsx
// In App.jsx: add showForm state
const [showForm, setShowForm] = useState(false);

// In the return, replace the form with:
<button
  className="toggle-form-btn"
  onClick={() => setShowForm(!showForm)}
>
  {showForm ? "Cancel" : "Add Customer"}
</button>

{showForm && (
  <form onSubmit={handleAddCustomer} className="add-customer-form">
    <h3>Add New Customer</h3>
    <input
      type="text"
      placeholder="First Name"
      value={firstName}
      onChange={(e) => setFirstName(e.target.value)}
      required
    />
    <input
      type="text"
      placeholder="Last Name"
      value={lastName}
      onChange={(e) => setLastName(e.target.value)}
      required
    />
    <input
      type="email"
      placeholder="Email"
      value={email}
      onChange={(e) => setEmail(e.target.value)}
      required
    />
    <button type="submit">Add Customer</button>
  </form>
)}
```

</details>

---

## Part 5: Bonus Challenges

Work on as many as you can; they are listed in order of difficulty.

### Challenge 1: Customer Count Badge

Display a badge next to the "Customers" heading that shows the filtered count vs. total when a search is active.

Example: `Customers (showing 2 of 5)`

**Hints:**
- Use a ternary: `{searchTerm ? \`showing ${filteredCustomers.length} of ${customers.length}\` : customers.length}`

---

### Challenge 2: Highlight Search Term

In each `CustomerCard`, highlight the matching part of the name or email when a search is active.

**Hints:**
- Pass `searchTerm` down to `CustomerCard`
- Write a helper that splits a string around the matching substring and wraps the match in `<mark>`
- Only apply highlighting when `searchTerm` is non-empty

---

### Challenge 3: Sort Customers

Add sort buttons (by first name, last name, or email) above the customer list.

**Hints:**
- Add `sortField` state in `App` (e.g., `"firstName"`)
- Compute `sortedFilteredCustomers` by sorting `filteredCustomers` on a copy (`[...filteredCustomers].sort(...)`)
- Map over `sortedFilteredCustomers` instead

---

### Challenge 4: Edit Customer (Advanced)

Allow editing a customer's details directly in the `CustomerDetail` panel.

**Hints:**
- Add an `isEditing` state to `CustomerDetail` (or manage it in `App`)
- When editing, render inputs pre-filled with the current values
- Pass an `onUpdate` callback from `App` to update the customer in the array using `.map()`
- Show Save / Cancel buttons
