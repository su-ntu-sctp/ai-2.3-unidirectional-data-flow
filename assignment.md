# Optional Assignment: Task Board

## Overview

- **Lesson:** Unidirectional Data Flow and Conditional Rendering / 3
- **Type:** Optional Take-Home Assignment
- **Estimated Time:** 2–3 hours
- **Due:** Before next lesson
- **Submission:** GitHub repository link or ZIP file

## Learning Objectives Covered

This assignment reinforces:

- Lifting state up to a common ancestor
- Passing callbacks as props for child-to-parent communication
- Conditional rendering using `if`, ternary, and logical `&&`
- Computing derived values from state instead of duplicating state
- Structuring an app into focused, reusable components

## Assignment Description

Build a **Task Board**: a simple Kanban-style board where users can add tasks, move them between columns (To Do, In Progress, Done), and view task details. This project exercises unidirectional data flow and conditional rendering across multiple components.

### What You'll Build

A single-page application that:

- Displays tasks in three columns: **To Do**, **In Progress**, **Done**
- Has a form to add new tasks (title, description, priority)
- Allows moving a task to the next column via a button
- Shows a task detail panel when a task is clicked
- Conditionally renders an empty state message when a column has no tasks

## Requirements

### Core Requirements

#### 1. Project Setup

- [ ] Create a new React app using Vite
- [ ] Name your project `task-board`
- [ ] Pre-populate the board with at least 5 sample tasks across the three columns

#### 2. Task Data Structure

Each task should have the following shape:

```js
{
  id: 1,
  title: "Design login page",
  description: "Create wireframes and mockups for the login screen.",
  priority: "high",      // "low", "medium", or "high"
  status: "todo",        // "todo", "inprogress", or "done"
}
```

#### 3. Components to Create

**a) `App` Component**

- [ ] Holds the `tasks` array in state
- [ ] Holds `selectedTask` state (the currently viewed task, or `null`)
- [ ] Implements `handleAddTask`, `handleMoveTask`, `handleSelectTask`
- [ ] Passes state and callbacks down to child components
- [ ] Computes the three column arrays as derived state (do not store them separately)

**b) `AddTaskForm` Component**

- [ ] Accepts an `onAdd` callback prop
- [ ] Has controlled inputs for title (text), description (textarea), and priority (select: low/medium/high)
- [ ] Calls `onAdd` with the new task on submission
- [ ] Clears inputs after successful submission
- [ ] Only renders if a "Add Task" toggle button has been clicked (show/hide with state)

**c) `TaskColumn` Component**

- [ ] Accepts `title` (string), `tasks` (array), `onMove` and `onSelect` as props
- [ ] Renders a column header and a list of `TaskCard` components
- [ ] Shows an empty state message when `tasks` is empty

**d) `TaskCard` Component**

- [ ] Accepts `task`, `onMove`, and `onSelect` as props
- [ ] Displays the task title and a priority badge
- [ ] Has a "Move →" button that calls `onMove(task.id)` (disabled/hidden when status is "done")
- [ ] Calls `onSelect(task)` when the card is clicked

**e) `TaskDetail` Component**

- [ ] Accepts `task` (object or `null`) as a prop
- [ ] If `task` is `null`, renders a placeholder: "Click a task to view details."
- [ ] Otherwise, renders the full task: title, description, priority, and status

#### 4. Move Task Logic

`handleMoveTask` in `App` should advance a task to the next status:

```
todo → inprogress → done
```

Use `.map()` to update the specific task immutably:

```js
const handleMoveTask = (taskId) => {
  const nextStatus = { todo: "inprogress", inprogress: "done" };
  setTasks(tasks.map((t) =>
    t.id === taskId ? { ...t, status: nextStatus[t.status] } : t
  ));
};
```

#### 5. Derived Column Arrays

Compute the column arrays during render; do not store them as state:

```js
const todoTasks = tasks.filter((t) => t.status === "todo");
const inProgressTasks = tasks.filter((t) => t.status === "inprogress");
const doneTasks = tasks.filter((t) => t.status === "done");
```

### Stretch Goals

- [ ] Add a "Delete" button to each `TaskCard` that removes it from the board (clear `selectedTask` if the deleted task was selected)
- [ ] Display a task count badge on each column header: e.g., "To Do (3)"
- [ ] Color-code the priority badge: green for low, amber for medium, red for high
- [ ] Add a search/filter input in `App` that filters tasks across all columns by title keyword

## Example Initial Data

```js
const initialTasks = [
  { id: 1, title: "Design login page", description: "Create wireframes for the login screen.", priority: "high", status: "todo" },
  { id: 2, title: "Set up project repo", description: "Initialise Git repo and push initial commit.", priority: "medium", status: "done" },
  { id: 3, title: "Write API docs", description: "Document all REST endpoints.", priority: "low", status: "todo" },
  { id: 4, title: "Implement auth flow", description: "Build login and registration with JWT.", priority: "high", status: "inprogress" },
  { id: 5, title: "Fix navigation bug", description: "Sidebar collapses unexpectedly on mobile.", priority: "medium", status: "inprogress" },
];
```

## Resources

- [React docs: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
- [React docs: Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [React docs: Choosing the State Structure](https://react.dev/learn/choosing-the-state-structure)
