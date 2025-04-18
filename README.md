<!DOCTYPE html>
<html lang="pl">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Otwieranie Skrzynek CS2</title>
  <style>
    /* CSS - Style dla strony */
    body {
      background: linear-gradient(45deg, #ff6a00, #ee0979);
      background-size: 200% 200%;
      animation: backgroundAnimation 15s ease infinite;
      color: white;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      padding: 2rem;
    }

    /* Animacja tła */
    @keyframes backgroundAnimation {
      0% { background-position: 0 0; }
      100% { background-position: 100% 100%; }
    }

    h1 {
      color: #FFD700;
    }

    .button {
      margin-top: 1rem;
      padding: 0.5rem 1rem;
      background-color: #FFD700;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
    }

    input, select {
      margin-bottom: 1rem;
      padding: 0.3rem;
      border-radius: 6px;
    }
    
    /* Reszta stylów CSS */
  </style>
</head>

<body>
  <h1>Symulator Otwierania Skrzynek CS2</h1>
  
  <!-- Formularz rejestracji -->
  <div id="registration-form">
    <h2>Rejestracja</h2>
    <input type="text" id="registerUsername" placeholder="Nazwa użytkownika">
    <input type="password" id="registerPassword" placeholder="Hasło">
    <button class="button" onclick="register()">Zarejestruj</button>
  </div>

  <!-- Formularz logowania -->
  <div id="login-form">
    <h2>Logowanie</h2>
    <input type="text" id="loginUsername" placeholder="Nazwa użytkownika">
    <input type="password" id="loginPassword" placeholder="Hasło">
    <button class="button" onclick="login()">Zaloguj</button>
  </div>

  <!-- Informacje o użytkowniku po zalogowaniu -->
  <div id="user-info" style="display:none">
    <p>Witaj, <span id="username-display"></span></p>
    <button class="button" onclick="logout()">Wyloguj</button>
  </div>

  <!-- Sklep z doładowaniem konta -->
  <div class="balance">Saldo: <span id="balance">100 PLN</span></div>
  <div>
    <input type="number" id="topUpAmount" placeholder="Doładuj" min="1">
    <button class="button" onclick="topUpBalance()">Doładuj konto</button>
  </div>

  <!-- Skrzynki i losowanie -->
  <div class="case">
    <label for="caseSelect"><strong>Wybierz skrzynkę:</strong></label>
    <select id="caseSelect">
      <option value="clutch">Clutch Case</option>
      <option value="random">Random Case</option>
      <option value="prisma">Prisma Case</option>
    </select>
    <button class="button" onclick="openCase()">Otwórz skrzynkę (koszt: 20 PLN)</button>
    <div class="carousel" id="carousel"></div>
    <div class="result" id="result"></div>
  </div>

  <div class="history" id="history">
    <h2>Historia losowań</h2>
    <div id="historyList"></div>
  </div>

  <div class="inventory" id="inventory">
    <h2>Ekwipunek</h2>
    <div id="inventoryList"></div>
  </div>

  <audio id="openSound" src="https://www.myinstants.com/media/sounds/csgo-case-open.mp3"></audio>

  <!-- JavaScript - Logika strony -->
  <script>
    // Wstępnie ustawione dane
    let currentUser = null;
    let balance = 100;
    const allCases = {
      clutch: [
        { name: "Skin 1", rarity: "common", price: 5, image: "https://via.placeholder.com/100" },
        { name: "Skin 2", rarity: "rare", price: 10, image: "https://via.placeholder.com/100" }
      ],
      prisma: [
        { name: "Skin A", rarity: "mythical", price: 15, image: "https://via.placeholder.com/100" },
        { name: "Skin B", rarity: "exceedingly-rare", price: 30, image: "https://via.placeholder.com/100" }
      ]
    };

    // Zaktualizowanie wyświetlania salda
    function updateBalanceDisplay() {
      document.getElementById("balance").textContent = balance + " PLN";
    }

    // Powiadomienia na stronie
    function showNotification(message) {
      const notification = document.createElement('div');
      notification.className = 'notification';
      notification.textContent = message;
      document.body.appendChild(notification);
      setTimeout(() => {
        notification.remove();
      }, 3000);
    }

    // Rejestracja
    function register() {
      const username = document.getElementById("registerUsername").value;
      const password = document.getElementById("registerPassword").value;

      if (username && password) {
        localStorage.setItem(username, JSON.stringify({ username, password, balance: 100 }));
        showNotification("Rejestracja zakończona sukcesem!");
      } else {
        showNotification("Proszę podać nazwę użytkownika i hasło!");
      }
    }

    // Logowanie
    function login() {
      const username = document.getElementById("loginUsername").value;
      const password = document.getElementById("loginPassword").value;
      const userData = JSON.parse(localStorage.getItem(username));

      if (userData && userData.password === password) {
        currentUser = userData;
        document.getElementById("username-display").textContent = username;
        document.getElementById("login-form").style.display = "none";
        document.getElementById("user-info").style.display = "block";
        updateBalanceDisplay();
        showNotification("Zalogowano pomyślnie!");
      } else {
        showNotification("Błędna nazwa użytkownika lub hasło!");
      }
    }

    // Wylogowanie
    function logout() {
      currentUser = null;
      document.getElementById("user-info").style.display = "none";
      document.getElementById("login-form").style.display = "block";
      showNotification("Wylogowano pomyślnie!");
    }

    // Doładowanie salda
    function topUpBalance() {
      const amount = parseInt(document.getElementById("topUpAmount").value);
      if (!isNaN(amount) && amount > 0) {
        balance += amount;
        updateBalanceDisplay();
        showNotification("Doładowano konto!");
      } else {
        showNotification("Podaj poprawną liczbę PLN.");
      }
    }

    // Otwarcie skrzynki
    function openCase() {
      const selectedCase = document.getElementById("caseSelect").value;
      const skins = allCases[selectedCase];
      const carousel = document.getElementById('carousel');
      const result = document.getElementById('result');
      const openSound = document.getElementById('openSound');
      
      result.textContent = "";
      openSound.play();

      carousel.innerHTML = '';
      const track = document.createElement('div');
      track.className = 'carousel-track';

      let displaySkins = [];
      for (let i = 0; i < 10; i++) {
        displaySkins.push(skins[Math.floor(Math.random() * skins.length)]);
      }

      displaySkins.forEach(skin => {
        const item = document.createElement('div');
        item.className = `item ${skin.rarity}`;
        item.innerHTML = `<img src="${skin.image}" alt="${skin.name}"><div>${skin.name}</div>`;
        track.appendChild(item);
      });

      carousel.appendChild(track);

      setTimeout(() => {
        const finalSkin = displaySkins[7];
        result.innerHTML = `Wylosowano: <span class="${finalSkin.rarity}">${finalSkin.name}</span>`;
      }, 2600);
    }
  </script>
</body>

</html>
