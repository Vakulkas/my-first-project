<!DOCTYPE html>
<html lang="uk">
<head>
  <meta charset="UTF-8" />
  <title>HelpShare MVP</title>
  <style>
    body { font-family: Arial; max-width: 800px; margin: auto; }
    input, textarea { width: 100%; margin: 5px 0; }
    .card { border: 1px solid #ccc; padding: 10px; margin: 10px 0; }
    button { margin-top: 5px; }
  </style>
</head>
<body>

<h1>HelpShare</h1>

<div id="auth">
  <h3>Реєстрація / Вхід</h3>
  <input id="username" placeholder="Логін" />
  <input id="password" type="password" placeholder="Пароль" />
  <button onclick="register()">Реєстрація</button>
  <button onclick="login()">Вхід</button>
</div>

<div id="app" style="display:none;">
  <p>Ви: <b id="user"></b> | Бали: <span id="points">0</span></p>

  <h3>Нове звернення</h3>
  <input id="title" placeholder="Заголовок" />
  <textarea id="text" placeholder="Опис проблеми"></textarea>
  <button onclick="addRequest()">Опублікувати</button>

  <h3>Звернення</h3>
  <div id="requests"></div>
</div>

<script>
let users = JSON.parse(localStorage.getItem("users")) || [];
let requests = JSON.parse(localStorage.getItem("requests")) || [];
let currentUser = null;

function save() {
  localStorage.setItem("users", JSON.stringify(users));
  localStorage.setItem("requests", JSON.stringify(requests));
}

function register() {
  const u = username.value, p = password.value;
  if (users.find(x => x.username === u)) return alert("Користувач існує");
  users.push({ id: Date.now(), username: u, password: p, points: 0 });
  save();
  alert("Зареєстровано");
}

function login() {
  const u = username.value, p = password.value;
  currentUser = users.find(x => x.username === u && x.password === p);
  if (!currentUser) return alert("Помилка");
  auth.style.display = "none";
  app.style.display = "block";
  user.innerText = currentUser.username;
  render();
}

function addRequest() {
  requests.push({
    id: Date.now(),
    title: title.value,
    text: text.value,
    authorId: currentUser.id,
    answers: []
  });
  save();
  render();
}

function addAnswer(id) {
  const req = requests.find(r => r.id === id);
  const input = document.getElementById("a"+id);
  if (req.authorId === currentUser.id) return alert("Не можна відповідати собі");
  req.answers.push({ text: input.value, authorId: currentUser.id, rating: null });
  save();
  render();
}

function rate(rid, i, val) {
  const req = requests.find(r => r.id === rid);
  if (req.authorId !== currentUser.id) return alert("Тільки автор оцінює");
  const ans = req.answers[i];
  if (ans.rating !== null) return;
  ans.rating = val;
  const u = users.find(x => x.id === ans.authorId);
  u.points += val;
  save();
  render();
}

function render() {
  points.innerText = currentUser.points;
  requestsDiv = "";
  requests.forEach(r => {
    requestsDiv += `<div class="card">
      <h4>${r.title}</h4>
      <p>${r.text}</p>
      <input id="a${r.id}" placeholder="Відповідь" />
      <button onclick="addAnswer(${r.id})">Відповісти</button>
    `;
    r.answers.forEach((a,i) => {
      requestsDiv += `<p>💬 ${a.text}</p>`;
      if (a.rating === null)
        requestsDiv += `<button onclick="rate(${r.id},${i},10)">10</button>`;
      else
        requestsDiv += `<p>⭐ ${a.rating}</p>`;
    });
    requestsDiv += "</div>";
  });
  requests.innerHTML = requestsDiv;
}
</script>

</body>
</html>
