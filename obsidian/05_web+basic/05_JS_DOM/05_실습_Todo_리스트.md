# 05. 실습: Todo 리스트 만들기

#dom #실습 #todo-list

지금까지 배운 DOM 조작과 이벤트 처리 기술을 종합하여 동적인 Todo 리스트 웹 애플리케이션을 만들어 봅니다. 이 실습을 통해 실제 웹 개발에서 DOM API가 어떻게 활용되는지 이해할 수 있습니다.

---

## 1. 최종 목표

- 사용자가 입력창에 할 일을 입력하고 '추가' 버튼을 누르면 목록에 추가됩니다.
- 각 할 일 항목 옆의 '삭제' 버튼을 누르면 해당 항목이 목록에서 삭제됩니다.
- 페이지 새로고침 없이 모든 동작이 이루어집니다.
- `localStorage`를 이용하여 브라우저를 닫았다 열어도 목록이 유지됩니다.

---

## 2. 기본 구조 (HTML, CSS)

### 가. HTML (`index.html`)

기본적인 구조를 정의합니다. `<h1>` 제목, 할 일을 입력할 `<form>`, 그리고 목록이 표시될 `<ul>`로 구성됩니다.

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <title>Todo List 실습</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div class="container">
      <h1>Todo List</h1>
      <form id="todo-form">
        <input
          type="text"
          id="todo-input"
          placeholder="새로운 할 일을 입력하세요..."
          required
        />
        <button type="submit">추가</button>
      </form>
      <ul id="todo-list">
        <!-- 할 일 목록이 여기에 동적으로 추가됩니다. -->
      </ul>
    </div>
    <script src="app.js"></script>
  </body>
</html>
```

### 나. CSS (`style.css`)

가독성을 위한 최소한의 스타일을 적용합니다.

```css
body {
  font-family: sans-serif;
  background-color: #f4f4f4;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
}

.container {
  background: white;
  padding: 25px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  width: 400px;
}

#todo-form {
  display: flex;
  margin-bottom: 20px;
}

#todo-input {
  flex-grow: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

#todo-form button {
  padding: 10px 15px;
  border: none;
  background-color: #007bff;
  color: white;
  cursor: pointer;
  border-radius: 4px;
  margin-left: 10px;
}

#todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

#todo-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border-bottom: 1px solid #eee;
}

#todo-list li .delete-btn {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
}
```

---

## 3. 기능 구현 (JavaScript)

### 다. JavaScript (`app.js`)

```javascript
// 1. 필요한 DOM 요소 선택
const todoForm = document.getElementById("todo-form");
const todoInput = document.getElementById("todo-input");
const todoList = document.getElementById("todo-list");

const TODOS_KEY = "todos"; // localStorage의 키

// 2. 할 일 목록을 저장할 배열
let todos = [];

// 3. localStorage에 저장된 할 일 목록 불러오기
function loadTodos() {
  const savedTodos = localStorage.getItem(TODOS_KEY);
  if (savedTodos) {
    todos = JSON.parse(savedTodos);
    todos.forEach(addTodoToList); // 화면에 목록 표시
  }
}

// 4. 할 일 목록을 localStorage에 저장하기
function saveTodos() {
  localStorage.setItem(TODOS_KEY, JSON.stringify(todos));
}

// 5. 할 일을 목록(ul)에 추가하는 함수
function addTodoToList(newTodo) {
  const li = document.createElement("li");
  li.id = newTodo.id; // 각 li에 고유 id 부여

  const span = document.createElement("span");
  span.textContent = newTodo.text;

  const deleteButton = document.createElement("button");
  deleteButton.textContent = "삭제";
  deleteButton.className = "delete-btn";

  li.appendChild(span);
  li.appendChild(deleteButton);
  todoList.appendChild(li);
}

// 6. 폼 제출(submit) 이벤트 처리
todoForm.addEventListener("submit", function (event) {
  event.preventDefault(); // 새로고침 방지

  const newTodoText = todoInput.value.trim();
  if (newTodoText === "") return;

  const newTodoObj = {
    text: newTodoText,
    id: Date.now(), // 고유 ID로 현재 시간 사용
  };

  todos.push(newTodoObj); // 배열에 추가
  addTodoToList(newTodoObj); // 화면에 추가
  saveTodos(); // localStorage에 저장

  todoInput.value = ""; // 입력창 비우기
});

// 7. 삭제 버튼 클릭 이벤트 처리 (이벤트 위임 사용)
todoList.addEventListener("click", function (event) {
  if (event.target.classList.contains("delete-btn")) {
    const liToDelete = event.target.parentElement;

    // 배열에서 해당 할 일 제거
    todos = todos.filter((todo) => todo.id !== parseInt(liToDelete.id));

    liToDelete.remove(); // 화면에서 제거
    saveTodos(); // 변경된 목록을 localStorage에 저장
  }
});

// 8. 페이지 로드 시 저장된 목록 불러오기
loadTodos();
```

### 코드 해설

1.  **요소 선택**: `getElementById`로 필요한 DOM 요소들을 미리 찾아 변수에 저장합니다.
2.  **데이터 관리**: 할 일 목록을 `todos` 배열로 관리합니다. 화면과 데이터의 상태를 일치시키는 것이 중요합니다.
3.  **`loadTodos`**: 페이지가 로드될 때 `localStorage`에서 `TODOS_KEY`로 저장된 데이터를 가져옵니다. 데이터가 있으면 `JSON.parse`로 다시 배열로 변환하고, 각 항목을 화면에 표시합니다.
4.  **`saveTodos`**: `todos` 배열이 변경될 때마다 `JSON.stringify`를 이용해 문자열로 변환하여 `localStorage`에 저장합니다.
5.  **`addTodoToList`**: 새로운 할 일 객체를 받아 `<li>`, `<span>`, `<button>` 요소를 동적으로 생성하고 `<ul>`에 추가합니다.
6.  **폼 `submit` 이벤트**: 사용자가 할 일을 입력하고 제출하면, `preventDefault`로 새로고침을 막습니다. 고유 `id`를 가진 객체를 생성하여 `todos` 배열과 화면에 추가하고, `localStorage`에 저장합니다.
7.  **`click` 이벤트 (이벤트 위임)**: `<ul>`에 이벤트 리스너를 하나만 등록합니다. 클릭된 요소가 삭제 버튼(`delete-btn`)일 경우에만 동작합니다. `filter`를 사용해 `todos` 배열에서 해당 `id`를 가진 항목을 제외한 새 배열을 만들고, 화면에서도 해당 `<li>`를 제거한 후, 변경된 상태를 `localStorage`에 저장합니다.
8.  **초기화**: 스크립트가 실행될 때 `loadTodos()`를 호출하여 저장된 데이터를 불러옵니다.
