<!DOCTYPE html>
<html lang="uk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Гра Skull King</title>
<style>
body {
background-image: url('https://igrokot.com.ua/image/cache/catalog/products/Nastilni-igry/Nastilna-gra-Skull-King-Cherep-Korolya-EN-1000x1000.jpeg');
background-size: cover;
background-position: center;
height: 100vh;
margin: 0;
display: flex;
justify-content: center;
align-items: center;
color: white;
font-family: Arial, sans-serif;
}
.container {
text-align: center;
background: rgba(0, 0, 0, 0.6);
padding: 20px;
border-radius: 10px;
width: 300px;
}
input {
padding: 10px;
margin: 10px;
font-size: 16px;
}
button {
padding: 10px 20px;
font-size: 16px;
background-color: #4CAF50;
color: white;
border: none;
border-radius: 5px;
cursor: pointer;
}
button:hover {
background-color: #45a049;
}
table {
width: 100%;
margin-top: 20px;
border-collapse: collapse;
}
table, th, td {
border: 1px solid white;
}
th, td {
padding: 10px;
text-align: center;
}
th {
background-color: #4CAF50;
}
</style>
<!-- Firebase SDK -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-app.js";
import { getDatabase, ref, set, get, child } from "https://www.gstatic.com/firebasejs/11.1.0/firebase-database.js";

const firebaseConfig = {
apiKey: "AIzaSyAd0YKE8MCpE3H9z8eChoAdcL2lUuTjcqM",
authDomain: "skull-king-82a78.firebaseapp.com",
databaseURL: "https://skull-king-82a78-default-rtdb.firebaseio.com",
projectId: "skull-king-82a78",
storageBucket: "skull-king-82a78.appspot.com",
messagingSenderId: "22652291023",
appId: "1:22652291023:web:bb874ed135e53cfb87b4ba",
measurementId: "G-EM6QCWHWJV"
};

const app = initializeApp(firebaseConfig);
const database = getDatabase(app);
</script>
</head>
<body>

<div class="container" id="gameChoice">
<h1>Skull King</h1>
<button id="startGameBtn">Почати гру</button>
<br><br>
<button id="joinGameBtn">Доєднатися до гри</button>
</div>

<div class="container" id="adminRoom" style="display: none;">
<h1>Створення кімнати</h1>
<label for="adminName">Ім'я адміністратора:</label><br>
<input type="text" id="adminName" placeholder="Введіть ім'я адміністратора" /><br>
<button id="generateCodeBtn">Згенерувати</button>
<br>
<span id="roomCodeDisplay" style="display: none;">Код кімнати: <span id="roomCodeText"></span></span>
<br>
<button id="startGameBtnAfterGenerate" style="display: none;">Почати гру</button>
</div>

<div class="container" id="joinRoom" style="display: none;">
<h1>Приєднатися до кімнати гри</h1>
<label for="playerName">Ім'я гравця:</label><br>
<input type="text" id="playerName" placeholder="Введіть ім'я гравця" /><br>
<label for="roomCode">Код кімнати:</label><br>
<input type="text" id="roomCode" placeholder="Введіть код кімнати" /><br>
<button id="joinBtn">Доєднатися</button>
</div>

<div class="container" id="leaderboard" style="display: none;">
<h1>Рейтинговий список</h1>
<table id="scoreTable">
<thead>
<tr>
<th>Ім'я</th>
<th>Бали</th>
</tr>
</thead>
<tbody id="scoreBody">
<!-- Дані будуть додані через JavaScript -->
</tbody>
</table>
</div>

<script>
document.getElementById("startGameBtn").addEventListener("click", function() {
document.getElementById("gameChoice").style.display = "none";
document.getElementById("adminRoom").style.display = "block";
});

document.getElementById("joinGameBtn").addEventListener("click", function() {
document.getElementById("gameChoice").style.display = "none";
document.getElementById("joinRoom").style.display = "block";
});

document.getElementById("generateCodeBtn").addEventListener("click", function() {
const adminName = document.getElementById("adminName").value;
if (adminName) {
const roomCode = Math.floor(Math.random() * 900) + 100;
localStorage.setItem("adminName", adminName);
localStorage.setItem("roomCode", roomCode);
document.getElementById("roomCodeText").innerText = roomCode;
document.getElementById("roomCodeDisplay").style.display = "block";
document.getElementById("startGameBtnAfterGenerate").style.display = "block";

// Save room data to Firebase
const roomRef = ref(database, 'rooms/' + roomCode);
set(roomRef, {
admin: adminName,
players: []
});
} else {
alert("Будь ласка, введіть ім'я адміністратора");
}
});

document.getElementById("startGameBtnAfterGenerate").addEventListener("click", function() {
const roomCode = localStorage.getItem("roomCode");
document.getElementById("adminRoom").style.display = "none";
document.getElementById("leaderboard").style.display = "block";

const tableBody = document.getElementById("scoreBody");
const row = tableBody.insertRow();
const nameCell = row.insertCell(0);
const scoreCell = row.insertCell(1);

nameCell.innerText = localStorage.getItem("adminName");
scoreCell.innerHTML = `<input type="number" value="0" />`;
});

document.getElementById("joinBtn").addEventListener("click", function() {
const playerName = document.getElementById("playerName").value;
const roomCode = document.getElementById("roomCode").value;

if (playerName && roomCode) {
const savedRoomCode = localStorage.getItem("roomCode");
if (roomCode === savedRoomCode) {
document.getElementById("joinRoom").style.display = "none";
document.getElementById("leaderboard").style.display = "block";

const tableBody = document.getElementById("scoreBody");
const row = tableBody.insertRow();
const nameCell = row.insertCell(0);
const scoreCell = row.insertCell(1);

nameCell.innerText = playerName;
scoreCell.innerHTML = `<input type="number" value="0" readonly />`;

// Add player to Firebase
const roomRef = ref(database, 'rooms/' + roomCode);
get(roomRef).then((snapshot) => {
if (snapshot.exists()) {
const roomData = snapshot.val();
roomData.players.push(playerName);
set(roomRef, roomData);
}
});

} else {
alert("Невірний код кімнати");
}
} else {
alert("Будь ласка, введіть ім'я гравця та код кімнати");
}
});
</script>

</body>
</html>
