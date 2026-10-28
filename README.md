const STORAGE_KEY = 'todo-flow-items';
const FILTERS = ['all', 'active', 'completed'];

const todoForm = document.querySelector('#todoForm');
const todoInput = document.querySelector('#todoInput');
const todoList = document.querySelector('#todoList');
const taskCounter = document.querySelector('#taskCounter');
const remainingCount = document.querySelector('#remainingCount');
const completedCount = document.querySelector('#completedCount');
const clearCompletedBtn = document.querySelector('#clearCompleted');
const filterButtons = document.querySelectorAll('.filter-btn');

let todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
let activeFilter = 'all';

function saveTodos() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
}

function updateCounts() {
  const total = todos.length;
  const remaining = todos.filter((todo) => !todo.completed).length;
  const completed = total - remaining;

  taskCounter.textContent = `${total} task${total === 1 ? '' : 's'}`;
  remainingCount.textContent = `${remaining} remaining`;
  completedCount.textContent = `${completed} completed`;
}

function getFilteredTodos() {
  if (activeFilter === 'active') {
    return todos.filter((todo) => !todo.completed);
  }

  if (activeFilter === 'completed') {
    return todos.filter((todo) => todo.completed);
  }

  return todos;
}

function renderTodos() {
  const filteredTodos = getFilteredTodos();

  if (!filteredTodos.length) {
    todoList.innerHTML = `
      <li class="empty-state">
        <div>
          <h3>No tasks here</h3>
          <p>Add a task to get started.</p>
        </div>
      </li>
    `;
    updateCounts();
    return;
  }

  todoList.innerHTML = filteredTodos
    .map(
      (todo) => `
        <li class="todo-item ${todo.completed ? 'completed' : ''}" data-id="${todo.id}">
          <input class="checkbox" type="checkbox" ${todo.completed ? 'checked' : ''} aria-label="Mark ${todo.text} completed" />
          <span class="todo-text">${escapeHtml(todo.text)}</span>
          <button class="delete-btn" type="button" aria-label="Delete ${escapeHtml(todo.text)}">×</button>
        </li>
      `
    )
    .join('');

  updateCounts();
}

function escapeHtml(text) {
  return text
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

function addTodo(text) {
  const cleanText = text.trim();

  if (!cleanText) {
    todoInput.focus();
    return;
  }

  todos.unshift({
    id: crypto.randomUUID(),
    text: cleanText,
    completed: false,
  });

  saveTodos();
  renderTodos();
  todoForm.reset();
  todoInput.focus();
}

function toggleTodo(id) {
  todos = todos.map((todo) =>
    todo.id === id ? { ...todo, completed: !todo.completed } : todo
  );

  saveTodos();
  renderTodos();
}

function deleteTodo(id) {
  todos = todos.filter((todo) => todo.id !== id);
  saveTodos();
  renderTodos();
}

function setFilter(filter) {
  activeFilter = FILTERS.includes(filter) ? filter : 'all';

  filterButtons.forEach((button) => {
    const isActive = button.dataset.filter === activeFilter;
    button.classList.toggle('active', isActive);
  });

  renderTodos();
}

function clearCompleted() {
  todos = todos.filter((todo) => !todo.completed);
  saveTodos();
  renderTodos();
}

todoForm.addEventListener('submit', (event) => {
  event.preventDefault();
  addTodo(todoInput.value);
});

todoList.addEventListener('click', (event) => {
  const deleteButton = event.target.closest('.delete-btn');
  const item = event.target.closest('.todo-item');

  if (deleteButton && item) {
    deleteTodo(item.dataset.id);
    return;
  }
});

todoList.addEventListener('change', (event) => {
  const checkbox = event.target.closest('.checkbox');
  const item = event.target.closest('.todo-item');

  if (checkbox && item) {
    toggleTodo(item.dataset.id);
  }
});

filterButtons.forEach((button) => {
  button.addEventListener('click', () => setFilter(button.dataset.filter));
});

clearCompletedBtn.addEventListener('click', clearCompleted);

renderTodos();
