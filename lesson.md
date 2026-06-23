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

In Lesson 2.2 you built a working CRM app and extracted `CustomerCard` into its own component. Today you will continue from that end state and add two new features: a search bar that filters the customer list, and a detail panel that shows information about the selected customer. Along the way you will see why React's one-way data model makes apps predictable, and you will learn how to render different UI based on application state.

---

## Part 1: Starting Point and Recap (10 minutes)

### Where We Left Off

At the end of Lesson 2.2, your `simple-crm-web` project looks like this:

```
simple-crm-web/
├── src/
│   ├── components/
│   │   ├── CustomerCard.jsx
│   │   └── CustomerCard.module.css
│   ├── App.jsx
│   ├── App.css
│   ├── index.css
│   ├── mockData.js
│   └── main.jsx
```

Open the project and confirm it runs:

```bash
cd simple-crm-web
npm run dev
```

Navigate to `http://localhost:5173` and verify the customer list and Add Customer form are working.

### What We Are Adding Today

We will add two new features to the app:

1. A **search bar** that filters the customer list by name or email
2. A **customer detail panel** that shows full information about the selected customer

**Final component tree:**

```
App
├── SearchBar        (new)
├── CustomerDetail   (new)
└── CustomerCard (×N)
```

`App` will remain the single component that owns all shared state. `SearchBar` and `CustomerDetail` will be purely presentational: they receive props and render UI.

---

## Part 2: Smart Components and Presentational Components (5 minutes)

Before writing any code, take a moment to look at the current `App.jsx`. It does two very different things at once:

- It **manages state**: `customers`, `form`, and the logic that updates them
- It **renders UI**: the form inputs, the customer cards, the layout

As an app grows, mixing these two concerns in one file makes it harder to understand and change. A useful way to think about components is to ask: does this component *own data and logic*, or does it *just render what it receives*?

In the React community, these two roles are sometimes called **smart components** and **presentational components**:

| | Smart component | Presentational component |
|---|---|---|
| **Also called** | Container component | UI component |
| **Owns state?** | Yes | No |
| **Has logic?** | Yes (handlers, derivations) | Minimal |
| **Receives data via** | State and its own logic | Props only |
| **Example (today)** | `App` | `CustomerCard`, `SearchBar`, `CustomerDetail` |

> **A note on history:** This terminology was popularised by Dan Abramov around 2015, when class components were the norm. He later updated his original article to note that React Hooks make the distinction less rigid; a single function component can manage state and stay clean at the same time. The terms are still useful as a way of thinking about *responsibility*: which components should own decisions, and which should just display what they are given.

You do not need to enforce this as a strict rule. Think of it as a question to ask when a component is getting complicated: should I separate the logic from the display?

In this lesson, `App` acts as the smart component. The components we add (`SearchBar` and `CustomerDetail`) will be purely presentational: they receive props and render UI.

---

## Part 3: Add a Search Bar (30 minutes)

We want to filter the customer list by name or email as the user types. Let's build `SearchBar` the most natural-feeling way first, then discover why it does not work, and fix it.

### Step 1: Create `SearchBar.jsx` with its own state

Create two new files side by side: `src/components/SearchBar.jsx` and `src/components/SearchBar.module.css`.

Start with the CSS module:

```css
/* src/components/SearchBar.module.css */
.searchBar {
  margin-bottom: var(--space-4);
}

.searchBar input {
  width: 100%;
  padding: var(--space-2) var(--space-3);
  border: 1px solid var(--border-default);
  border-radius: var(--radius-md);
  font-family: var(--font-sans);
  font-size: var(--text-md);
  color: var(--text-strong);
}

.searchBar input:focus {
  outline: none;
  border-color: var(--border-focus);
  box-shadow: var(--ring);
}
```

Now create the component. Give it its own `searchTerm` state; that is the obvious first instinct:

```jsx
// src/components/SearchBar.jsx (first attempt)
import { useState } from "react";
import styles from "./SearchBar.module.css";

function SearchBar() {
  const [searchTerm, setSearchTerm] = useState("");

  return (
    <div className={styles.searchBar}>
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

Import and render `<SearchBar />` above the customer list. You only need to add the import and the component itself; nothing else in `App.jsx` changes yet:

```jsx
// src/App.jsx (add this import at the top)
import SearchBar from "./components/SearchBar";

// In the return, add <SearchBar /> just above <div className="customer-list">:
<SearchBar />

<div className="customer-list">
  <h2>Customers ({customers.length})</h2>
  ...
</div>
```

**Browser check:** The search input renders and you can type into it. So far so good.

### Step 3: Try to filter, and hit the wall

Now try to use `searchTerm` in `App.jsx` to filter the list:

```jsx
// src/App.jsx (this will not work)
const filteredCustomers = customers.filter(
  (c) => c.firstName.toLowerCase().includes(searchTerm.toLowerCase())
);
```

`searchTerm` is not defined in `App`. It lives inside `SearchBar`. `App` has no way to read it.

This is the fundamental constraint of React's data model: **state is private to the component that owns it**. `SearchBar` and the customer list are siblings; neither can reach into the other's state.

### Step 4: Lift the state up to `App`

The fix is to move `searchTerm` to the closest component that needs to share it: their common ancestor, `App`. Then `App` passes the value down to `SearchBar` as a prop, and `SearchBar` calls a callback to update it.

First, update `SearchBar` to receive its value and change handler as props instead of owning them:

```jsx
// src/components/SearchBar.jsx (updated)
import styles from "./SearchBar.module.css";

function SearchBar({ searchTerm, onSearch }) {
  return (
    <div className={styles.searchBar}>
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
// src/App.jsx (add searchTerm state)
const [searchTerm, setSearchTerm] = useState("");

// Derived: computed on every render, always in sync
const filteredCustomers = customers.filter(
  (c) =>
    c.firstName.toLowerCase().includes(searchTerm.toLowerCase()) ||
    c.lastName.toLowerCase().includes(searchTerm.toLowerCase()) ||
    c.email.toLowerCase().includes(searchTerm.toLowerCase()),
);
```

Update the return to pass props to `SearchBar` and use `filteredCustomers` in the list:

```jsx
// src/App.jsx (updated return, only the relevant section)
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

> **Why not `useState` for `filteredCustomers`?** If you store it separately, you now have two sources of truth. Every time `customers` or `searchTerm` changes you would have to keep them in sync manually, which is a common source of bugs. Computing it during render is cheaper than it looks and always correct.

**Browser check:** Typing in the search box now filters the displayed cards in real time. The customer count updates to reflect the filtered results. Deleting a customer still works correctly.

This pattern of moving state to a common ancestor so siblings can share it is called **lifting state up**. You will use it constantly in React.

---

### Activity 1: Empty State Message (15 minutes)

You have seen how to render a list conditionally. Now apply it yourself.

**Task:** When no customers match the search term, replace the card grid with a helpful message. When the list is entirely empty (all deleted, no search active), show a different message.

**Hints:**
1. Check `filteredCustomers.length === 0` before rendering the grid
2. Use a ternary (`condition ? a : b`) to choose between the empty message and the grid
3. To distinguish between "no search results" and "truly empty list", check whether `searchTerm` is non-empty inside the message: `{searchTerm ? "..." : "..."}`
4. Both the empty message and the grid should sit inside the existing `<div className="customer-list">` element

**Expected result:**
- Searching for something with no matches: "No customers match your search."
- Deleting all customers with no search active: "No customers yet. Add one above!"
- Results exist: card grid shows as normal

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

Add the style to `App.css`:

```css
/* src/App.css */
.empty-state {
  color: var(--text-muted);
  font-size: var(--text-sm);
  padding: var(--space-6) 0;
}
```

</details>

---

## Part 4: Customer Detail Panel (30 minutes)

Now we will add a `CustomerDetail` component that shows full information about a selected customer. Clicking a card selects it; the detail panel updates on the right.

This introduces a new pattern: **selection state**. The selected customer is owned by `App` (it needs to pass it to `CustomerDetail`), and `CustomerCard` raises an event when clicked.

### Step 1: Create `CustomerDetail.jsx` and `CustomerDetail.module.css`

Following the same pattern as `CustomerCard`, styles should live alongside the component. Create two files side by side: `src/components/CustomerDetail.jsx` and `src/components/CustomerDetail.module.css`.

Start with the CSS module:

```css
/* src/components/CustomerDetail.module.css */
.panel {
  width: 300px;
  flex: none;
  background: var(--surface-card);
  border: 1px solid var(--border-subtle);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  box-shadow: var(--shadow-sm);
  align-self: flex-start;
}

.name {
  font-size: var(--text-md);
  font-weight: var(--weight-semibold);
  color: var(--text-strong);
  margin-bottom: var(--space-1);
}

.company {
  font-size: var(--text-sm);
  color: var(--text-muted);
  margin-bottom: var(--space-4);
}

.contactRow {
  font-size: var(--text-sm);
  color: var(--text-body);
  margin-bottom: var(--space-2);
}

.section {
  margin-top: var(--space-4);
  padding-top: var(--space-4);
  border-top: 1px solid var(--border-subtle);
}

.sectionLabel {
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: var(--space-2);
}

.notes {
  font-size: var(--text-sm);
  color: var(--text-body);
  line-height: var(--leading-normal);
}

.notesEmpty {
  font-size: var(--text-sm);
  color: var(--text-subtle);
  font-style: italic;
}

.tags {
  display: flex;
  flex-wrap: wrap;
  gap: var(--space-2);
}

.badge {
  display: inline-flex;
  align-items: center;
  padding: 2px 9px;
  border-radius: var(--radius-pill);
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
}

.badgeActive {
  background: var(--success-50);
  color: var(--success-700);
}

.badgeInactive {
  background: var(--neutral-100);
  color: var(--text-muted);
}

.tag {
  font-size: var(--text-xs);
  font-weight: var(--weight-medium);
  color: var(--primary-700);
  background: var(--primary-100);
  border-radius: var(--radius-pill);
  padding: 2px 8px;
}

.empty {
  color: var(--text-muted);
  font-size: var(--text-sm);
}
```

Now create the component. It handles two cases: no customer selected (renders a placeholder) and a customer selected (renders its fields).

```jsx
// src/components/CustomerDetail.jsx
import styles from "./CustomerDetail.module.css";

function CustomerDetail({ customer }) {
  if (!customer) {
    return (
      <div className={styles.panel}>
        <p className={styles.empty}>Select a customer to view details.</p>
      </div>
    );
  }

  return (
    <div className={styles.panel}>
      <h2 className={styles.name}>
        {customer.firstName} {customer.lastName}
      </h2>
      {customer.company && (
        <p className={styles.company}>{customer.company}</p>
      )}

      <div>
        <p className={styles.contactRow}>{customer.email}</p>
        {customer.phone && (
          <p className={styles.contactRow}>{customer.phone}</p>
        )}
      </div>

      <div className={styles.section}>
        <p className={styles.sectionLabel}>Status and tags</p>
        <div className={styles.tags}>
          <span
            className={`${styles.badge} ${customer.status === "active" ? styles.badgeActive : styles.badgeInactive}`}
          >
            {customer.status}
          </span>
          {customer.tags.map((tag) => (
            <span key={tag} className={styles.tag}>
              {tag}
            </span>
          ))}
        </div>
      </div>

      <div className={styles.section}>
        <p className={styles.sectionLabel}>Notes</p>
        {customer.notes ? (
          <p className={styles.notes}>{customer.notes}</p>
        ) : (
          <p className={styles.notesEmpty}>No notes yet.</p>
        )}
      </div>

      <div className={styles.section}>
        <p className={styles.sectionLabel}>Customer since</p>
        <p className={styles.contactRow}>{customer.createdAt}</p>
      </div>
    </div>
  );
}

export default CustomerDetail;
```

Notice we use three different conditional rendering patterns here:
- `if (!customer) return ...`: early return for the null case
- `{customer.company && <p>...}` and `{customer.phone && <p>...}`: only render when the value exists
- ternary for `notes`: show the note text when present, or a placeholder when empty
- ternary in the `className`: choose between two badge styles based on status

### Step 2: Update `CustomerCard` to support selection

Add `onSelect` and `isSelected` props. `onSelect` is called when the card is clicked; `isSelected` controls a CSS class for visual feedback.

Open `src/components/CustomerCard.jsx` and update it:

```jsx
// src/components/CustomerCard.jsx (updated)
import { Mail, Phone } from "lucide-react";
import styles from "./CustomerCard.module.css";

function initials(firstName, lastName) {
  return (firstName[0] + lastName[0]).toUpperCase();
}

function CustomerCard({ customer, onDelete, onSelect, isSelected }) {
  const { firstName, lastName, email, phone, status, tags } = customer;

  return (
    <div
      className={`${styles.card} ${isSelected ? styles.cardSelected : ""}`}
      onClick={() => onSelect(customer)}
    >
      <div className={styles.header}>
        <div className={styles.avatar}>{initials(firstName, lastName)}</div>
        <div className={styles.nameBlock}>
          <p className={styles.name}>
            {firstName} {lastName}
          </p>
        </div>
        <span
          className={`${styles.badge} ${status === "active" ? styles.badgeActive : styles.badgeInactive}`}
        >
          {status}
        </span>
      </div>

      <div className={styles.meta}>
        <div className={styles.metaRow}>
          <Mail size={14} />
          {email}
        </div>
        <div className={styles.metaRow}>
          <Phone size={14} />
          {phone}
        </div>
      </div>

      <div className={styles.footer}>
        <div className={styles.tags}>
          {tags.map((tag) => (
            <span key={tag} className={styles.tag}>
              {tag}
            </span>
          ))}
        </div>
        <button
          className={styles.deleteButton}
          onClick={(e) => {
            e.stopPropagation();
            onDelete(customer.id);
          }}
        >
          Delete
        </button>
      </div>
    </div>
  );
}

export default CustomerCard;
```

Two things to notice:

- The `onClick` on the card calls `onSelect(customer)`, passing the full object because the detail panel needs all the fields.
- The delete button calls `e.stopPropagation()` to prevent the click from bubbling up to the card's own `onClick` handler. Without this, clicking Delete would both delete the customer and try to select it at the same time.

> **Why does `onSelect` receive the whole `customer` object, while `onDelete` only receives the `id`?** The detail panel needs all the fields to display, so we pass the full object. The delete handler only needs to identify which one to remove, so the `id` is enough. Pass the minimum needed for the operation.

Now add the `cardSelected` rule to `CustomerCard.module.css`:

```css
/* src/components/CustomerCard.module.css (add at the bottom) */
.cardSelected {
  border-color: var(--primary-500);
  box-shadow: var(--shadow-md), 0 0 0 2px var(--primary-100);
}
```

### Step 3: Add `selectedCustomer` state to `App` and wire it up

```jsx
// src/App.jsx (add this state declaration alongside the others)
const [selectedCustomer, setSelectedCustomer] = useState(null);
```

Update `handleDeleteCustomer` to clear the selection if the deleted customer is currently selected:

```jsx
// src/App.jsx
const handleDeleteCustomer = (customerId) => {
  setCustomers(customers.filter((c) => c.id !== customerId));
  if (selectedCustomer?.id === customerId) {
    setSelectedCustomer(null);
  }
};
```

> **Why the `?.` operator?** `selectedCustomer` may be `null`; optional chaining prevents a runtime error when no customer is selected.

### Step 4: Update the layout

Import `CustomerDetail` and add `selectedCustomer` props to the list. We also want a two-column layout: the customer list on the left and the detail panel on the right.

Update `App.jsx`:

```jsx
// src/App.jsx (add this import)
import CustomerDetail from "./components/CustomerDetail";
```

Update the return to wrap the customer panel and detail panel in a two-column layout:

```jsx
// src/App.jsx (updated return)
return (
  <div className="simple-crm">
    <h1>Simple CRM</h1>

    <form onSubmit={handleAddCustomer} className="add-customer-form">
      {/* unchanged */}
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

Add the two-column layout rules to `App.css`. The `CustomerDetail` panel's own styles already live in `CustomerDetail.module.css`; only the rules that govern how `App` arranges its children belong here:

```css
/* src/App.css */
.crm-layout {
  display: flex;
  gap: var(--space-6);
  align-items: flex-start;
}

.customer-panel {
  flex: 1;
  min-width: 0;
}
```

**Browser check:**
- The detail panel appears on the right showing "Select a customer to view details."
- Clicking a card populates the detail panel with that customer's information
- The selected card is visually highlighted with a blue border
- Deleting the selected customer clears the detail panel
- Clicking Delete does not accidentally select the customer being deleted

---

### Activity 2: Show/Hide the Add Customer Form (15 minutes)

Apply what you know about boolean state and conditional rendering.

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
// src/App.jsx (add showForm state)
const [showForm, setShowForm] = useState(false);

// In the return, replace the form section with:
<button
  className="toggle-form-btn"
  onClick={() => setShowForm(!showForm)}
>
  {showForm ? "Cancel" : "Add Customer"}
</button>

{showForm && (
  <form onSubmit={handleAddCustomer} className="add-customer-form">
    {/* unchanged form contents */}
  </form>
)}
```

Add the button style to `App.css`:

```css
/* src/App.css */
.toggle-form-btn {
  padding: var(--space-2) var(--space-5);
  background: var(--action-primary);
  color: var(--text-on-primary);
  border: none;
  border-radius: var(--radius-md);
  font-family: var(--font-sans);
  font-size: var(--text-sm);
  font-weight: var(--weight-medium);
  cursor: pointer;
  margin-bottom: var(--space-6);
  transition: background var(--duration-fast) var(--ease-standard);
}

.toggle-form-btn:hover {
  background: var(--action-primary-hover);
}
```

</details>

---

## Part 5: Bonus Challenges

Work on as many as you can; they are listed in order of difficulty.

### Challenge 1: Customer Count Badge

Display a badge next to the "Customers" heading that shows the filtered count versus total when a search is active.

Example: `Customers (showing 2 of 5)`

**Hint:** Use a ternary: `{searchTerm ? \`showing ${filteredCustomers.length} of ${customers.length}\` : customers.length}`

---

### Challenge 2: Highlight Search Term

In each `CustomerCard`, highlight the matching part of the name or email when a search is active.

**Hints:**
- Pass `searchTerm` down to `CustomerCard` as a prop
- Write a helper that splits a string around the matching substring and wraps the match in `<mark>`
- Only apply highlighting when `searchTerm` is non-empty

---

### Challenge 3: Sort Customers

Add sort buttons (by first name, last name, or email) above the customer list.

**Hints:**
- Add `sortField` state in `App` (e.g., `"firstName"`)
- Compute a sorted list by calling `.sort()` on a copy of `filteredCustomers` (`[...filteredCustomers].sort(...)`)
- Map over the sorted list instead

---

### Challenge 4: Edit Customer (Advanced)

Allow editing a customer's details directly in the `CustomerDetail` panel.

**Hints:**
- Add an `isEditing` state in `App`
- When editing, render inputs pre-filled with the current values inside `CustomerDetail`
- Pass an `onUpdate` callback from `App` that updates the customer in the array using `.map()`
- Show Save and Cancel buttons; Cancel should restore the original values
