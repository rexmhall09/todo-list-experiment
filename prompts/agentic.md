You are a senior front-end engineer. Write a to-do list web app called **Tasks** as ONE self-contained `index.html` file: inline `<style>`, inline `<script>`, no libraries, no CDNs, no web fonts, no build step. The file is published as a teaching example of clean, human-readable front-end code, so readability matters as much as behaviour.

Follow this specification exactly: same ids, classes, function names, order and values. Do not invent extra features, globals, comments or wrappers. Output ONLY the finished file in a single ```html code block, with nothing before or after it.

# 1. Behaviour

1. **Add.** A text field (placeholder "What needs to be done?", `maxlength="200"`) and an "Add" button inside a `<form>`. Pressing Enter or clicking Add submits. Text is trimmed; blank text adds nothing. New tasks go to the bottom. After submitting, the field is cleared and keeps focus.
2. **Complete.** Each row has a real checkbox. Toggling it flips `completed`; completed text is struck through in the muted colour. Keyboard focus stays on the checkbox after toggling.
3. **Edit.** Double-clicking the task text, or pressing Enter while the text is focused, swaps the text for an inline input with all its text selected. Enter or leaving the field saves. Escape cancels. Saving blank text deletes the task.
4. **Delete.** A small "×" button per row, drawn with CSS (no emoji, no icon font). It is invisible until the row is hovered or something in the row has keyboard focus. Clicking it plays a short exit animation, then removes the task.
5. **Filter.** Buttons All, Active, Completed. The filter lives in the URL hash (`#/`, `#/active`, `#/completed`), so it survives reloads and the back button.
6. **Footer.** "N items left" counting only incomplete tasks, correctly pluralised ("0 items left", "1 item left", "2 items left"), and a "Clear completed" button that is hidden unless something is completed.
7. **Empty state.** When no rows are visible, one muted line: All → "Nothing here yet.", Active → "No active tasks.", Completed → "No completed tasks.". Otherwise the line is empty and hidden.
8. **Persistence.** Tasks are saved to `localStorage` under `"todos.v1"` after every change and loaded on start. Missing, corrupt or wrong-shaped data never crashes the app.
9. **Security.** User text is only ever written with `textContent`, `.value` or `setAttribute`. Never `innerHTML`, never a template string that becomes HTML.

A todo is `{ id: string, text: string, completed: boolean }`. Ids come from `crypto.randomUUID()`.

# 2. HTML

Use exactly this document. Indent with 2 spaces. The `<style>` element holds the CSS from section 3 and the `<script>` element holds the JavaScript from section 4.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="color-scheme" content="light dark">
  <title>Tasks</title>
  <style>
    /* section 3 goes here */
  </style>
</head>
<body>
  <main class="app">
    <h1 class="title">Tasks</h1>

    <form class="composer" id="composer" autocomplete="off">
      <label class="visually-hidden" for="composer-input">New task</label>
      <input class="composer-input" id="composer-input" type="text"
             placeholder="What needs to be done?" maxlength="200">
      <button class="composer-add" type="submit">Add</button>
    </form>

    <section class="card">
      <ul class="todo-list" id="todo-list" aria-label="Tasks"></ul>
      <p class="empty-state" id="empty-state"></p>
    </section>

    <footer class="footer">
      <span class="footer-count" id="item-count" aria-live="polite"></span>
      <div class="filters" role="group" aria-label="Filter tasks">
        <button class="filter" type="button" data-filter="all" aria-pressed="true">All</button>
        <button class="filter" type="button" data-filter="active" aria-pressed="false">Active</button>
        <button class="filter" type="button" data-filter="completed" aria-pressed="false">Completed</button>
      </div>
      <button class="clear-completed" id="clear-completed" type="button" hidden>Clear completed</button>
    </footer>
  </main>

  <script>
    // section 4 goes here
  </script>
</body>
</html>
```

Each task row is built in JavaScript and looks like this:

```html
<li class="todo" data-id="{id}">        <!-- + "is-completed" when completed, + "is-entering" when just added -->
  <input class="todo-toggle" type="checkbox" aria-label="Complete: {text}">
  <span class="todo-text" tabindex="0">{text}</span>
  <button class="todo-delete" type="button" aria-label="Delete task"></button>
</li>
```

While editing, the row also has `is-editing` and the span is replaced by `<input class="todo-edit" type="text" maxlength="200" aria-label="Edit task">`. While being deleted, it has `is-leaving`.

# 3. CSS

The look is calm, minimal and product-quality, in the spirit of Things or Linear. Absolutely NO emoji, gradients, gradient text, glassmorphism, glow, heavy or stacked shadows, bouncing, confetti, red, rainbow colours, badges, coloured side stripes or filled pill buttons.

Write the stylesheet in exactly this order. Every selector starts in column 0 of the stylesheet; declarations are indented one level. Put one short `/* Group name */` comment above each group. Plain CSS only: no nesting, no `&`.

**Tokens**
- `:root`: `--bg: #f7f7f5; --surface: #ffffff; --text: #1d1d1f; --muted: #8a8a8e; --border: #e6e6e3; --accent: #2f6f4f; --radius: 10px; --ease: cubic-bezier(0.2, 0.8, 0.2, 1);`
- `@media (prefers-color-scheme: dark) { :root { --bg: #121212; --surface: #1b1b1b; --text: #ececec; --muted: #7d7d80; --border: #2a2a2a; --accent: #6fbf8f; } }`

**Base**
- `*, *::before, *::after { box-sizing: border-box; }`
- `body`: margin 0; background `var(--bg)`; colour `var(--text)`; `font-family: system-ui, -apple-system, "Segoe UI", sans-serif`; font-size 16px; line-height 1.5; `-webkit-font-smoothing: antialiased`.
- `.app`: max-width 520px; `margin: 12vh auto`; `padding: 0 20px`.
- `.title`: font-size 28px; font-weight 600; letter-spacing -0.01em; `margin: 0 0 24px`.
- `.visually-hidden`: position absolute; width 1px; height 1px; margin -1px; padding 0; border 0; overflow hidden; `clip: rect(0 0 0 0)`; white-space nowrap.

**Composer and card**
- `.composer, .card`: background `var(--surface)`; `border: 1px solid var(--border)`; border-radius `var(--radius)`; `box-shadow: 0 1px 2px rgba(0, 0, 0, 0.04)`.
- `.composer`: display flex; margin-bottom 12px; overflow hidden.
- `.composer:focus-within`: `border-color: color-mix(in srgb, var(--accent) 40%, var(--border))`.
- `.composer-input`: flex 1; min-width 0; border 0; outline none; background transparent; `font: inherit`; colour `var(--text)`; `padding: 14px 16px`.
- `.composer-input::placeholder`: colour `var(--muted)`.
- `.composer-add`: background none; border 0; `font: inherit`; font-weight 500; colour `var(--accent)`; `padding: 0 16px`; cursor pointer.

**Rows**
- `.todo-list`: list-style none; margin 0; padding 0.
- `.todo`: display flex; align-items center; gap 12px; `padding: 14px 16px`; `transition: background-color 120ms`.
- `.todo + .todo`: `border-top: 1px solid var(--border)`.
- `.todo:hover`: `background: color-mix(in srgb, var(--text) 3%, transparent)`.
- `.todo-text`: flex 1; min-width 0; `overflow-wrap: anywhere`; cursor text; border-radius 4px; `transition: color 150ms`.
- `.todo.is-completed .todo-text`: `text-decoration: line-through`; colour `var(--muted)`.

**Checkbox**
- `.todo-toggle`: `appearance: none`; margin 0; width 18px; height 18px; flex-shrink 0; `border: 1.5px solid var(--muted)`; border-radius 50%; display grid; `place-content: center`; cursor pointer; `transition: background-color 150ms, border-color 150ms`.
- `.todo-toggle:checked`: background-color and border-color `var(--accent)`.
- `.todo-toggle::after`: `content: ""`; width 4px; height 8px; `border: solid white`; `border-width: 0 2px 2px 0`; `transform: translateY(-1px) rotate(45deg) scale(0)`; `transition: transform 150ms var(--ease)`. No background.
- `.todo-toggle:checked::after`: `transform: translateY(-1px) rotate(45deg) scale(1)`.

**Delete button**
- `.todo-delete`: position relative; width 24px; height 24px; flex-shrink 0; padding 0; border 0; border-radius 6px; background none; cursor pointer; opacity 0; `transition: opacity 120ms, background-color 120ms`.
- `.todo:hover .todo-delete, .todo:focus-within .todo-delete`: opacity 1.
- `.todo-delete:hover`: `background: color-mix(in srgb, var(--text) 6%, transparent)`.
- `.todo-delete::before, .todo-delete::after`: `content: ""`; position absolute; left 50%; top 50%; width 12px; height 1.5px; background `var(--muted)`; border-radius 1px.
- `.todo-delete::before`: `transform: translate(-50%, -50%) rotate(45deg)`.
- `.todo-delete::after`: `transform: translate(-50%, -50%) rotate(-45deg)`.

**Edit field**
- `.todo-edit`: flex 1; min-width 0; `font: inherit`; colour inherit; background transparent; border 0; outline none; padding 0; `box-shadow: inset 0 -1px var(--accent)`.

**Empty state and footer**
- `.empty-state`: margin 0; `padding: 28px 16px`; text-align center; colour `var(--muted)`; font-size 14px.
- `.empty-state:empty`: display none.
- `.footer`: display flex; flex-wrap wrap; align-items center; justify-content space-between; gap 12px; margin-top 14px; `padding: 0 4px`; font-size 13px; colour `var(--muted)`.
- `.filters`: display flex; gap 14px.
- `.filter, .clear-completed`: background none; border 0; `padding: 4px 2px`; `font: inherit`; colour inherit; cursor pointer; `transition: color 120ms`.
- `.filter:hover, .clear-completed:hover`: colour `var(--text)`.
- `.filter[aria-pressed="true"]`: colour `var(--text)`; `text-decoration: underline`; `text-decoration-thickness: 1px`; `text-underline-offset: 4px`.
- `.clear-completed[hidden]`: display none.

**Focus**
- `:focus-visible`: `outline: 2px solid var(--accent)`; `outline-offset: 2px`.
- `.composer-input:focus-visible, .todo-edit:focus-visible`: outline none. (The composer border and the edit underline are their focus indicators.)

**Motion** (one block, nothing animates outside it)
```css
@media (prefers-reduced-motion: no-preference) {
  .composer {
    transition: border-color 150ms var(--ease);
  }

  @keyframes todo-enter {
    from { opacity: 0; transform: translateY(-6px); }
  }

  .todo.is-entering {
    animation: todo-enter 220ms var(--ease);
  }

  @keyframes todo-leave {
    to { opacity: 0; transform: translateX(8px); }
  }

  .todo.is-leaving {
    animation: todo-leave 180ms var(--ease) forwards;
  }
}
```

# 4. JavaScript

Rules for all of the script: `function` declarations for every function (no `const name = () =>` functions), `const` and `let` only, double quotes, strict equality, 2-space indentation. Above every function put exactly ONE `//` comment line saying WHY it exists (not what it does). No other comments, no JSDoc, no `console.log`, no `alert`, no inline `on…=` attributes.

The script has five sections, in this order, each opened by exactly one header line: `// ---- Data ----`, `// ---- Pure updates ----`, `// ---- Rendering ----`, `// ---- Events ----`, `// ---- Init ----`. No other header lines.

## 4.1 Data

```js
const STORAGE_KEY = "todos.v1";
const MAX_LENGTH = 200;
const LEAVE_DURATION = 180;
const FILTERS = {
  all: () => true,
  active: (todo) => !todo.completed,
  completed: (todo) => todo.completed,
};

let state = { todos: [], filter: "all" };
let newestTodoId = null;
```

- `isValidTodo(value)`: one `return` of `value !== null && typeof value === "object" && typeof value.id === "string" && typeof value.text === "string" && typeof value.completed === "boolean"`, with each `&&` starting a new indented line.
- `loadTodos()`: in `try`, `const parsed = JSON.parse(localStorage.getItem(STORAGE_KEY) ?? "[]");` then `return Array.isArray(parsed) ? parsed.filter(isValidTodo) : [];`. In `catch {`, `return [];`.
- `saveTodos(todos)`: in `try`, `localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));`. An empty `catch {}`; its comment line above the function says storage can be full or disabled and the app should keep working.

## 4.2 Pure updates

These never change the array or any todo they receive: no `todos[i] = …`, no `todo.text = …`, no `push`, no `splice`. They return a NEW array, except `addTodo` with blank text, which returns the same `todos` so callers can tell nothing changed.

- `addTodo(todos, text)`: `const trimmed = text.trim();` `if (!trimmed) return todos;` `return [...todos, { id: crypto.randomUUID(), text: trimmed, completed: false }];`
- `toggleTodo(todos, id)`: `return todos.map((todo) => (todo.id === id ? { ...todo, completed: !todo.completed } : todo));`
- `editTodo(todos, id, text)`: `const trimmed = text.trim();` `if (!trimmed) return deleteTodo(todos, id);` `return todos.map((todo) => (todo.id === id ? { ...todo, text: trimmed } : todo));`
- `deleteTodo(todos, id)`: `return todos.filter((todo) => todo.id !== id);`
- `clearCompleted(todos)`: `return todos.filter((todo) => !todo.completed);`

## 4.3 Rendering

```js
const elements = {
  list: document.getElementById("todo-list"),
  emptyState: document.getElementById("empty-state"),
  itemCount: document.getElementById("item-count"),
  clearCompleted: document.getElementById("clear-completed"),
  filterButtons: document.querySelectorAll(".filter"),
  composerInput: document.getElementById("composer-input"),
};

const EMPTY_MESSAGES = {
  all: "Nothing here yet.",
  active: "No active tasks.",
  completed: "No completed tasks.",
};
```

- `setState(patch)`: `state = { ...state, ...patch };` `saveTodos(state.todos);` `render();` Every change in the app goes through `setState`.
- `createTodoElement(todo)`:
  - `const item = document.createElement("li");` `item.className = "todo";` `item.dataset.id = todo.id;`
  - `item.classList.toggle("is-completed", todo.completed);` and `item.classList.toggle("is-entering", todo.id === newestTodoId);`
  - `const toggle = document.createElement("input");` with `className = "todo-toggle"`, `type = "checkbox"`, `checked = todo.completed`, and `toggle.setAttribute("aria-label", "Complete: " + todo.text);`
  - `const text = document.createElement("span");` with `className = "todo-text"`, `tabIndex = 0`, `textContent = todo.text`.
  - `const remove = document.createElement("button");` with `className = "todo-delete"`, `type = "button"`, and `remove.setAttribute("aria-label", "Delete task");`
  - `item.append(toggle, text, remove);` `return item;`
- `findRow(id)`: `return [...elements.list.children].find((row) => row.dataset.id === id);` (ids are never put into a selector string).
- `renderFooter()`:
  - `const remaining = state.todos.filter(FILTERS.active).length;`
  - `const noun = remaining === 1 ? "item" : "items";`
  - ``elements.itemCount.textContent = `${remaining} ${noun} left`;``
  - `elements.filterButtons.forEach((button) => button.setAttribute("aria-pressed", String(button.dataset.filter === state.filter)));`
  - `elements.clearCompleted.hidden = !state.todos.some(FILTERS.completed);`
- `render()`:
  - `const visibleTodos = state.todos.filter(FILTERS[state.filter]);`
  - `elements.list.replaceChildren(...visibleTodos.map(createTodoElement));`
  - `newestTodoId = null;`
  - `elements.emptyState.textContent = visibleTodos.length === 0 ? EMPTY_MESSAGES[state.filter] : "";`
  - `renderFooter();`

## 4.4 Events

Helpers first:

- `filterFromHash()`: `if (location.hash === "#/active") return "active";` `if (location.hash === "#/completed") return "completed";` `return "all";`
- `prefersReducedMotion()`: `return window.matchMedia("(prefers-reduced-motion: reduce)").matches;`
- `startEditing(item)`:
  - `const todo = state.todos.find((candidate) => candidate.id === item.dataset.id);`
  - `const input = document.createElement("input");` with `type = "text"`, `className = "todo-edit"`, `maxLength = MAX_LENGTH`, `value = todo.text`, and `input.setAttribute("aria-label", "Edit task");`
  - `item.querySelector(".todo-text").replaceWith(input);` `item.classList.add("is-editing");` `input.focus();` `input.select();` (focus only after the input is in the page).
- `finishEditing(item, shouldSave)`:
  - `if (!item.classList.contains("is-editing")) return;` (leaving the field fires after Enter; this stops a second save)
  - `const text = item.querySelector(".todo-edit").value;` `item.classList.remove("is-editing");`
  - if `shouldSave`, `setState({ todos: editTodo(state.todos, item.dataset.id, text) });` otherwise `render();`
- `removeTodo(item)`:
  - `const remove = () => setState({ todos: deleteTodo(state.todos, item.dataset.id) });`
  - `if (prefersReducedMotion()) { remove(); return; }`
  - `item.classList.add("is-leaving");` `setTimeout(remove, LEAVE_DURATION);` (a timer, not `animationend`: a re-render during the animation would detach the row and the event would never fire)

Then the handlers. Every list handler begins with `const item = event.target.closest(".todo");` and `if (!item) return;`.

- `handleSubmit(event)`: `event.preventDefault();` `const todos = addTodo(state.todos, elements.composerInput.value);` `if (todos !== state.todos) { newestTodoId = todos[todos.length - 1].id; setState({ todos }); }` then `elements.composerInput.value = "";` `elements.composerInput.focus();`
- `handleListChange(event)`: if `event.target.matches(".todo-toggle")`: `setState({ todos: toggleTodo(state.todos, item.dataset.id) });` then `findRow(item.dataset.id)?.querySelector(".todo-toggle").focus();` (the re-render replaced the checkbox, so focus is put back).
- `handleListClick(event)`: if `event.target.closest(".todo-delete")`, `removeTodo(item);`
- `handleListDoubleClick(event)`: if `event.target.matches(".todo-text")`, `startEditing(item);`
- `handleListKeydown(event)`: if `event.target.matches(".todo-edit")`: Enter → `finishEditing(item, true);`, Escape → `finishEditing(item, false);`. Else if `event.target.matches(".todo-text")` and `event.key === "Enter"` → `startEditing(item);`
- `handleListFocusOut(event)`: if `event.target.matches(".todo-edit")`, `finishEditing(item, true);`
- `handleFilterClick(event)`: `const button = event.target.closest(".filter");` `if (!button) return;` `location.hash = button.dataset.filter === "all" ? "#/" : "#/" + button.dataset.filter;`
- `handleHashChange()`: `setState({ filter: filterFromHash() });`
- `handleClearCompleted()`: `setState({ todos: clearCompleted(state.todos) });`

Then the registrations, one per line, no comments between them:

```js
document.getElementById("composer").addEventListener("submit", handleSubmit);
elements.list.addEventListener("change", handleListChange);
elements.list.addEventListener("click", handleListClick);
elements.list.addEventListener("dblclick", handleListDoubleClick);
elements.list.addEventListener("keydown", handleListKeydown);
elements.list.addEventListener("focusout", handleListFocusOut);
document.querySelector(".filters").addEventListener("click", handleFilterClick);
window.addEventListener("hashchange", handleHashChange);
elements.clearCompleted.addEventListener("click", handleClearCompleted);
```

## 4.5 Init

```js
state = { todos: loadTodos(), filter: filterFromHash() };
render();
```

# 5. Before you answer

Check silently: every function above exists once with the exact name; no `innerHTML`; no `var`; nothing mutates `state.todos` in place; no listener is attached to an individual row; every CSS selector starts in column 0; exactly five `// ---- … ----` headers in the script; the file contains the full CSS and the full JavaScript, not placeholders. Then output the file.
