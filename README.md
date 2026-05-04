<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>噗噗的理財撲滿</title>
    <style>
        :root {
            --bg-color: #F7EBE8; 
            --text-color: #6B5B5A; 
            --btn-color: #D9A0A7; 
            --btn-hover: #C2878E; 
            --spend-color: #D9A0A7; 
            --save-color: #D4B2A7; 
            --invest-color: #C69F97; 
            --highlight-color: #A77478; 
        }

        body { font-family: 'Comic Sans MS', '微軟正黑體', sans-serif; background-color: var(--bg-color); color: var(--text-color); text-align: center; padding: 10px; margin: 0; }
        h1 { color: var(--highlight-color); font-size: 2em; text-shadow: 1px 1px 3px rgba(0,0,0,0.05); margin-top: 10px; }
        
        .banks-container { 
            display: flex; 
            justify-content: space-between; 
            margin: 20px 0; 
            flex-wrap: nowrap; 
            gap: 5px;
        }
        
        .piggy-bank { 
            background: white; 
            border-radius: 15px; 
            padding: 10px 5px; 
            width: 32%; 
            box-sizing: border-box;
            box-shadow: 0 5px 15px rgba(167, 116, 120, 0.08); 
            border: 3px dashed #ccc; 
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative; /* 為了讓金幣能準確定位 */
        }

        .piggy-bank.spend { border-color: var(--spend-color); }
        .piggy-bank.save { border-color: var(--save-color); }
        .piggy-bank.invest { border-color: var(--invest-color); }
        
        .bank-icon { font-size: 2.5em; margin-bottom: 5px; }
        .piggy-bank h3 { margin: 5px 0; font-size: 1.1em; color: var(--text-color); }
        .amount { font-size: 1.3em; font-weight: bold; color: var(--highlight-color); }

        .coins-container {
            display: flex;
            flex-wrap: wrap-reverse;
            justify-content: center;
            align-content: flex-start;
            width: 100%;
            min-height: 60px;
            max-height: 120px;
            overflow-y: auto;
            margin-top: 10px;
            gap: 2px;
            padding: 5px;
            background: rgba(107, 91, 90, 0.04); 
            border-radius: 10px;
        }

        .coin-visual {
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 10px;
            font-weight: bold;
            box-shadow: inset 0 0 3px rgba(255,255,255,0.5), 1px 1px 3px rgba(0,0,0,0.2);
        }
        .c-50 { width: 28px; height: 28px; background: radial-gradient(circle, #E8C382, #CBA358); color: #7A5C24; }
        .c-10 { width: 24px; height: 24px; background: radial-gradient(circle, #EAEAEA, #BDBDBD); color: #555; }
        .c-5 { width: 20px; height: 20px; background: radial-gradient(circle, #EAEAEA, #BDBDBD); color: #555; }
        .c-1 { width: 16px; height: 16px; background: radial-gradient(circle, #C89F82, #A67B5E); color: #FFF; font-size: 8px; }

        .action-panel { background: rgba(255, 255, 255, 0.6); border-radius: 20px; padding: 15px; margin: 15px auto; max-width: 600px; box-shadow: 0 5px 15px rgba(167, 116, 120, 0.05); }
        .action-panel h2 { color: var(--highlight-color); }
        
        button { background-color: var(--btn-color); border: none; border-radius: 8px; padding: 8px 15px; font-size: 1.1em; color: white; cursor: pointer; font-weight: bold; transition: 0.3s; margin: 5px; }
        button:hover { background-color: var(--btn-hover); }
        
        .dice-emoji { font-size: 2.5em; display: inline-block; }
        .spin { animation: spin 0.5s linear infinite; }
        @keyframes spin { 100% { transform: rotate(360deg); } }
        
        .input-group { margin: 10px 0; color: var(--text-color); font-weight: bold; }
        input[type="number"], select { font-size: 1em; padding: 5px; border-radius: 5px; border: 1px solid #D4B2A7; text-align: center; color: var(--text-color); background-color: #FFFBF9; }
        .alloc-input { width: 40px; }
        .admin-trigger { position: fixed; bottom: 10px; right: 10px; width: 30px; height: 30px; background: transparent; border: none; cursor: pointer; }

        /* 金幣掉落動畫元素 */
        .falling-coin {
            position: fixed;
            font-size: 35px;
            z-index: 1000;
            top: -50px;
            transition: top 0.8s cubic-bezier(0.5, 0, 0.75, 0), opacity 0.8s;
            pointer-events: none;
        }

        @media (max-width: 400px) {
            .bank-icon { font-size: 2em; }
            .piggy-bank h3 { font-size: 0.9em; }
            .amount { font-size: 1.1em; }
            button { font-size: 1em; padding: 6px 12px; }
        }
    </style>
</head>
<body>

    <h1>🎈 噗噗的理財撲滿 🎈</h1>

    <div class="banks-container">
        <div class="piggy-bank spend" id="bank-spend">
            <div class="bank-icon">🛍️</div>
            <h3>消費帳</h3>
            <div class="amount" id="amt-spend">讀取中</div>
            <div class="coins-container" id="coins-spend"></div>
        </div>
        <div class="piggy-bank save" id="bank-save">
            <div class="bank-icon">🏦</div>
            <h3>儲蓄帳</h3>
            <div class="amount" id="amt-save">讀取中</div>
            <div class="coins-container" id="coins-save"></div>
            <button id="btn-interest" style="font-size: 0.75em; padding: 5px; margin-top:10px;">月初領息</button>
        </div>
        <div class="piggy-bank invest" id="bank-invest">
            <div class="bank-icon">📈</div>
            <h3>投資帳</h3>
            <div class="amount" id="amt-invest">讀取中</div>
            <div class="coins-container" id="coins-invest"></div>
        </div>
    </div>

    <div class="action-panel">
        <h2>💰 發零用錢 (15元)</h2>
        <div class="input-group">
            🛍️ <input type="number" id="alloc-spend" class="alloc-input" min="0" max="15" value="5">
            🏦 <input type="number" id="alloc-save" class="alloc-input" min="0" max="15" value="5">
            📈 <input type="number" id="alloc-invest" class="alloc-input" min="0" max="15" value="5">
        </div>
        <button id="btn-deposit" style="background-color: var(--invest-color);">家長確認存入</button>
    </div>

    <div class="action-panel">
        <h2>🎲 每週六投資挑戰</h2>
        <div>
            <div class="dice-emoji" id="dice-display">🎲</div><br>
            <button id="roll-btn">擲骰子</button>
            <div id="dice-result" style="color: var(--highlight-color); font-weight: bold; margin-top: 5px;"></div>
        </div>
    </div>

    <div class="action-panel">
        <h2>🔄 罐子互轉</h2>
        <div class="input-group">
            從 <select id="transfer-from">
                <option value="spend">消費帳</option>
                <option value="save">儲蓄帳</option>
                <option value="invest">投資帳</option>
            </select>
            轉到 <select id="transfer-to">
                <option value="save">儲蓄帳</option>
                <option value="spend">消費帳</option>
                <option value="invest">投資帳</option>
            </select>
        </div>
        <div class="input-group">
            金額: <input type="number" id="transfer-amt" min="1" style="width: 70px;"> 元
        </div>
        <button id="btn-transfer" style="background-color: var(--save-color);">確認轉出</button>
    </div>

    <button class="admin-trigger" id="btn-admin"></button>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.1/firebase-app.js";
        import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.8.1/firebase-database.js";

        const firebaseConfig = {
            apiKey: "AIzaSyBkd-KKJZ-lQjWmg1b3TumwOJ8OEl21N5w",
            authDomain: "xavier-s-bank.firebaseapp.com",
            databaseURL: "https://xavier-s-bank-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "xavier-s-bank",
            storageBucket: "xavier-s-bank.firebasestorage.app",
            messagingSenderId: "804735399947",
            appId: "1:804735399947:web:fe1a82631a6bc40846ef0e"
        };

        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);
        const dbRef = ref(db, 'childFinance/');

        let balances = { spend: 0, save: 0, invest: 0, diceRolledThisWeek: false };

        onValue(dbRef, (snapshot) => {
            const data = snapshot.val();
            if (data) balances = data;
            updateUI();
        });

        function syncToCloud() {
            set(dbRef, balances);
        }

        // 判斷是否為每個月的第一個星期六
        function isFirstSaturday() {
            const today = new Date();
            // getDay() 0是日, 1是一... 6是六。 且日期必須在 1~7 號之間
            return today.getDay() === 6 && today.getDate() <= 7;
        }

        function renderCoins(containerId, totalAmount) {
            const container = document.getElementById(containerId);
            container.innerHTML = ''; 
            let remaining = totalAmount;
            
            const coinTypes = [50, 10, 5, 1];
            const coinsToDraw = [];

            for (let val of coinTypes) {
                while (remaining >= val) {
                    coinsToDraw.push(val);
                    remaining -= val;
                }
            }

            coinsToDraw.forEach(val => {
                const c = document.createElement('div');
                c.className = `coin-visual c-${val}`;
                c.innerText = val;
                container.appendChild(c);
            });
        }

        function updateUI() {
            document.getElementById('amt-spend').innerText = `${balances.spend} 元`;
            document.getElementById('amt-save').innerText = `${balances.save} 元`;
            document.getElementById('amt-invest').innerText = `${balances.invest} 元`;
            
            renderCoins('coins-spend', balances.spend);
            renderCoins('coins-save', balances.save);
            renderCoins('coins-invest', balances.invest);
        }

        // 存錢動畫邏輯
        function triggerCoinAnimation(s, sa, i, callback) {
            const targets = [
                { val: s, id: 'bank-spend' },
                { val: sa, id: 'bank-save' },
                { val: i, id: 'bank-invest' }
            ];
            
            let animationsRunning = 0;

            targets.forEach(target => {
                if (target.val > 0) {
                    animationsRunning++;
                    const bankElement = document.getElementById(target.id);
                    const rect = bankElement.getBoundingClientRect();
                    
                    const coin = document.createElement('div');
                    coin.className = 'falling-coin';
                    coin.innerText = '💰';
                    // 讓金幣從撲滿的正上方掉下來
                    coin.style.left = (rect.left + rect.width / 2 - 17) + "px"; 
                    document.body.appendChild(coin);

                    // 觸發重繪以啟動動畫
                    void coin.offsetWidth; 

                    // 設定金幣掉落的終點 (撲滿內部) 和逐漸變透明
                    coin.
