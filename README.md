# too-enough-dudasova-hra
Interaktívna anglická hra na precvičenie too a enough
<!DOCTYPE html>
<html lang="sk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agenti: Misia Too & Enough</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #1a1a2e; color: #fff; margin: 0; padding: 20px; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        .game-container { background: #16213e; width: 100%; max-width: 650px; padding: 30px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.5); border: 2px solid #0f3460; }
        h1, h2 { color: #e94560; text-align: center; }
        .btn { background: #e94560; color: white; border: none; padding: 12px 25px; font-size: 16px; border-radius: 8px; cursor: pointer; display: block; width: 100%; margin-top: 20px; font-weight: bold; transition: 0.3s; }
        .btn:hover { background: #c73650; }
        input[type="text"] { width: 100%; padding: 12px; margin: 15px 0; border-radius: 8px; border: 1px solid #0f3460; background: #1a1a2e; color: white; font-size: 16px; box-sizing: border-box; }
        .grammar-box { background: #0f3460; border-left: 5px solid #e94560; padding: 15px; margin: 15px 0; border-radius: 5px; font-size: 14px; }
        .question-card { background: #1a1a2e; padding: 20px; border-radius: 10px; margin-bottom: 15px; border: 1px solid #0f3460; }
        .option { background: #16213e; border: 1px solid #e94560; padding: 10px 15px; margin: 8px 0; border-radius: 5px; cursor: pointer; transition: 0.2s; }
        .option:hover { background: #e94560; color: white; }
        .status-bar { display: flex; justify-content: space-between; background: #0f3460; padding: 10px 15px; border-radius: 8px; margin-bottom: 20px; font-weight: bold; }
        .hidden { display: none; }
        .feedback { margin-top: 10px; font-weight: bold; }
    </style>
</head>
<body>

<div class="game-container">
    <!-- FÁZA 0: LOGIN -->
    <div id="screen-login">
        <h1>🕵️‍♂️ Operácia: TOO & ENOUGH</h1>
        <p style="text-align: center;">Vitajte, agenti! Odomknite tajný trezor plný peňazí správnym použitím gramatiky z obchodu s oblečením[cite: 1, 2].</p>
        <label>Zadajte svoje krycie meno (meno agenta):</label>
        <input type="text" id="agent-name" placeholder="Napr. Agent 007">
        <button class="btn" onclick="startGame()">ŠTARTOVAŤ MISIU</button>
    </div>

    <!-- HRA - MISIE -->
    <div id="screen-game" class="hidden">
        <div class="status-bar">
            <span id="stat-name">Agent: </span>
            <span id="stat-score">Body: 0</span>
            <span id="stat-time">Čas: 0s</span>
        </div>

        <div id="mission-container"></div>
    </div>

    <!-- FINÁLE & TREZOR -->
    <div id="screen-finale" class="hidden">
        <h1>🔒 Trezor plný peňazí</h1>
        <p style="text-align: center;">Zozbierali ste písmená: <strong id="collected-letters" style="color: #e94560; font-size: 20px;"></strong></p>
        <p>Zoradťe písmená do správneho anglického slova (súvisí so školou), aby sa trezor otvoril:</p>
        <input type="text" id="final-code-input" placeholder="Zadajte tajné slovo...">
        <button class="btn" onclick="checkFinalCode()">Odomknúť trezor</button>
        <p id="code-feedback" class="feedback"></p>
    </div>

    <!-- SEBAHODNOTENIE A REBRÍČEK -->
    <div id="screen-feedback" class="hidden">
        <h2>📊 Hlásenie z misie</h2>
        <p>Úspešne ste prešli hrou!</p>
        
        <label>1. Ako hodnotíš svoje zvládnutie tejto misie?</label>
        <select id="self-eval-1" style="width:100%; padding:10px; margin:10px 0; background:#1a1a2e; color:white; border-radius:5px;">
            <option>Zvládol/a som to ľavou zadnou, gramatiku ovládam na 100%.</option>
            <option>Zvládol/a som to dobre, občas mi pomohlo gramatické okienko.</option>
            <option>Mala/m som s tým trochu problém, musím si to zopakovať.</option>
        </select>

        <label>2. Ako hodnotíš náročnosť úloh?</label>
        <select id="self-eval-2" style="width:100%; padding:10px; margin:10px 0; background:#1a1a2e; color:white; border-radius:5px;">
            <option>Príliš ľahké</option>
            <option>Takurčite akurát (vyvážené)</option>
            <option>Príliš ťažké</option>
        </select>

        <button class="btn" onclick="showLeaderboardTab()">Zobraziť rebríček agentov</button>
    </div>

    <!-- REBRÍČEK -->
    <div id="screen-leaderboard" class="hidden">
        <h2>🏆 Rebríček najlepších agentov</h2>
        <div id="leaderboard-list" style="background: #1a1a2e; padding: 15px; border-radius: 8px; margin: 15px 0;"></div>
        <button class="btn" onclick="location.reload()">Hrať znova</button>
    </div>
</div>

<script>
    let playerName = "";
    let score = 0;
    let currentMission = 0;
    let seconds = 0;
    let timer;
    let letters = ["S", "K", "O", "L", "E"];
    let shuffledLetters = "";

    const missions = [
        {
            title: "Misia 1: Nákupná horúčka v obchode",
            text: "Mike skúša nohavice, ale sú príliš drahé[cite: 1]. Vyberte správnu možnosť:",
            options: [
                { text: "a) They are too expensive[cite: 1].", correct: true },
                { text: "b) They are enough expensive.", correct: false },
                { text: "c) They aren't expensive enough.", correct: false }
            ],
            grammar: "💡 <strong>Gramatické okienko:</strong> TOO sa používa pred prídavným menom a znamená 'príliš' (It's too expensive)[cite: 1]."
        },
        {
            title: "Misia 2: Veľkosť nohavíc",
            text: "Mike zistí, že nohavice nie sú dostatočne veľké[cite: 1]. Vyberte správnu možnosť:",
            options: [
                { text: "a) They are too big.", correct: false },
                { text: "b) They aren't big enough[cite: 1].", correct: true },
                { text: "c) They are big enough.", correct: false }
            ],
            grammar: "💡 <strong>Gramatické okienko:</strong> ENOUGH sa používa za prídavným menom a znamená 'dostatočne' (It isn't big enough)[cite: 1]."
        },
        {
            title: "Misia 3: Príbeh o peniazoch (Levi Strauss)",
            text: "V texte o rifliach sa píše, že Davis nemal dostatok peňazí na patent[cite: 2]. Ako to povieme správne?",
            options: [
                { text: "a) He didn't have enough money[cite: 2].", correct: true },
                { text: "b) He didn't have money enough.", correct: false }
            ],
            grammar: "💡 <strong>Gramatické okienko:</strong> S podstatnými menami (money) dávame ENOUGH PRED podstatné meno (enough money)[cite: 2]."
        }
    ];

    function startGame() {
        const input = document.getElementById("agent-name").value.trim();
        if(!input) {
            alert("Prosím, zadajte svoje krycie meno!");
            return;
        }
        playerName = input;
        document.getElementById("stat-name").innerText = "Agent: " + playerName;
        document.getElementById("screen-login").classList.add("hidden");
        document.getElementById("screen-game").classList.remove("hidden");
        
        // Spustenie stopiek
        timer = setInterval(() => {
            seconds++;
            document.getElementById("stat-time").innerText = "Čas: " + seconds + "s";
        }, 1000);

        loadMission();
    }

    function loadMission() {
        if(currentMission >= missions.length) {
            clearInterval(timer);
            showFinale();
            return;
        }

        let m = missions[currentMission];
        let html = `<h2>${m.title}</h2>`;
        html += `<p>${m.text}</p>`;
        html += `<div class="grammar-box">${m.grammar}</div>`;
        
        m.options.forEach((opt, idx) => {
            html += `<div class="option" onclick="checkAnswer(${opt.correct})">${opt.text}</div>`;
        });

        document.getElementById("mission-container").innerHTML = html;
    }

    function checkAnswer(isCorrect) {
        if(isCorrect) {
            score += 2;
            document.getElementById("stat-score").innerText = "Body: " + score;
            alert("Správne, agent! Postupujete ďalej.");
            currentMission++;
            loadMission();
        } else {
            alert("❌ Nesprávna odpoveď! Pozrite si gramatické okienko a skúste to znova.");
        }
    }

    function showFinale() {
        document.getElementById("screen-game").classList.add("hidden");
        document.getElementById("screen-finale").classList.remove("hidden");
        // Zamiešame písmená pre tajničku
        shuffledLetters = [...letters].sort(() => Math.random() - 0.5).join(" ");
        document.getElementById("collected-letters").innerText = shuffledLetters;
    }

    function checkFinalCode() {
        let val = document.getElementById("final-code-input").value.trim().toUpperCase();
        if(val === "SCHOOL") {
            document.getElementById("screen-finale").classList.add("hidden");
            document.getElementById("screen-feedback").classList.remove("hidden");
        } else {
            document.getElementById("code-feedback").style.color = "#e94560";
            document.getElementById("code-feedback").innerText = "❌ Nesprávny kód! Správne slovo zo zozbieraných písmen je SCHOOL.";
        }
    }

    function showLeaderboardTab() {
        document.getElementById("screen-feedback").classList.add("hidden");
        document.getElementById("screen-leaderboard").classList.remove("hidden");
        
        let listHtml = `
            1. Agent Alex — 6 bodov — 01:20s<br>
            2. <strong>${playerName} — ${score} bodov — ${seconds}s</strong><br>
            3. Agent Nina — 4 body — 02:10s
        `;
        document.getElementById("leaderboard-list").innerHTML = listHtml;
    }
</script>

</body>
</html>
