太棒了，你成功找到你的專屬金鑰了！

因為我們的網頁是直接放在 GitHub 上（不使用複雜的程式打包工具），所以 Firebase 預設給的 import 開頭寫法我們用不到，只要取用裡面那段 firebaseConfig 就可以了。

為了避免貼錯位置或漏掉字元，我直接幫你把你的金鑰整合進我們剛剛寫好的完整程式碼中了。

請直接複製以下這整段程式碼，然後把你的 index.html 裡面的內容全部覆蓋掉、存檔：

HTML
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小大人的理財撲滿</title>
    <style>
        :root {
            --bg-color: #FFF8E7;
            --text-color: #5C4033;
            --btn-color: #FFB6C1;
            --btn-hover: #FF69B4;
            --spend-color: #ADD8E6;
            --save-color: #98FB98;
            --invest-color: #FFD700;
        }

        body { font-family: 'Comic Sans MS', '微軟正黑體', sans-serif; background-color: var(--bg-color); color: var(--text-color); text-align: center; padding: 20px; margin: 0; }
        h1 { color: #d2691e; font-size: 2.5em; text-shadow: 2px 2px 4px rgba(0,0,0,0.1); }
        .banks-container { display: flex; justify-content: space-around; margin: 30px 0; flex-wrap: wrap; }
        .piggy-bank { background: white; border-radius: 20px; padding: 20px; width: 25%; min-width: 120px; box-shadow: 0 10px 15px rgba(0,0,0,0.1); position: relative; border: 3px dashed #ccc; }
        .piggy-bank.spend { border-color: var(--spend-color); }
        .piggy-bank.save { border-color: var(--save-color); }
        .piggy-bank.invest { border-color: var(--invest-color); }
        .bank-icon { font-size: 3em; margin-bottom: 10px; }
        .amount { font-size: 1.5em; font-weight: bold; color: #d2691e; }
        .action-panel { background: rgba(255, 255, 255, 0.7); border-radius: 20px; padding: 20px; margin: 20px auto; max-width: 600px; box-shadow: 0 5px 10px rgba(0,0,0,0.05); }
        button { background-color: var(--btn-color); border: none; border-radius: 10px; padding: 10px 20px; font-size: 1.2em; color: white; cursor: pointer; font-weight: bold; transition: 0.3s; margin: 5px; }
        button:hover { background-color: var(--btn-hover); }
        .dice-area { margin: 20px 0; }
        #dice-result { font-size: 1.5em; font-weight: bold; color: #e84118; margin-top: 10px; }
        .dice-emoji { font-size: 3em; display: inline-block; }
        .spin { animation: spin 0.5s linear infinite; }
        @keyframes spin { 100% { transform: rotate(360deg); } }
        .allocation-area input { width: 50px; font-size: 1.2em; text-align: center; border: 2px solid #ccc; border-radius: 5px; margin: 0 5px; }
        .admin-trigger { position: fixed; bottom: 10px; right: 10px; width: 20px; height: 20px; background: transparent; border: none; cursor: pointer; }
    </style>
</head>
<body>

    <h1>🎈 小大人的理財撲滿 🎈</h1>

    <div class="banks-container">
        <div class="piggy-bank spend">
            <div class="bank-icon">🛍️</div>
            <h3>消費帳</h3>
            <div class="amount" id="amt-spend">讀取中...</div>
        </div>
        <div class="piggy-bank save">
            <div class="bank-icon">🏦</div>
            <h3>儲蓄帳</h3>
            <div class="amount" id="amt-save">讀取中...</div>
            <button id="btn-interest" style="font-size: 0.8em; padding: 5px;">月初領息</button>
        </div>
        <div class="piggy-bank invest">
            <div class="bank-icon">📈</div>
            <h3>投資帳</h3>
            <div class="amount" id="amt-invest">讀取中...</div>
        </div>
    </div>

    <div class="action-panel">
        <h2>🎲 每週六投資挑戰</h2>
        <div class="dice-area">
            <div class="dice-emoji" id="dice-display">🎲</div>
            <br>
            <button id="roll-btn">擲骰子</button>
            <div id="dice-result"></div>
        </div>
    </div>

    <div class="action-panel">
        <h2>💰 發放零用錢 (15元)</h2>
        <p>請將 15 元分配到三個帳戶中：</p>
        <div class="allocation-area">
            🛍️ 消費: <input type="number" id="alloc-spend" min="0" max="15" value="5">
            🏦 儲蓄: <input type="number" id="alloc-save" min="0" max="15" value="5">
            📈 投資: <input type="number" id="alloc-invest" min="0" max="15" value="5">
        </div>
        <button id="btn-deposit" style="margin-top: 15px; background-color: #32CD32;">家長確認存入</button>
    </div>

    <button class="admin-trigger" id="btn-admin"></button>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.1/firebase-app.js";
        import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/10.8.1/firebase-database.js";

        // 你專屬的 Firebase 設定
        const firebaseConfig = {
            apiKey: "AIzaSyBkd-KKJZ-lQjWmg1b3TumwOJ8OEl21N5w",
            authDomain: "xavier-s-bank.firebaseapp.com",
            databaseURL: "https://xavier-s-bank-default-rtdb.asia-southeast1.firebasedatabase.app",
            projectId: "xavier-s-bank",
            storageBucket: "xavier-s-bank.firebasestorage.app",
            messagingSenderId: "804735399947",
            appId: "1:804735399947:web:fe1a82631a6bc40846ef0e"
        };

        // 初始化 Firebase 與資料庫
        const app = initializeApp(firebaseConfig);
        const db = getDatabase(app);
        const dbRef = ref(db, 'childFinance/');

        // 預設資料結構
        let balances = { spend: 0, save: 0, invest: 0, diceRolledThisWeek: false };

        // 監聽雲端資料變動 (達到即時連動效果)
        onValue(dbRef, (snapshot) => {
            const data = snapshot.val();
            if (data) {
                balances = data;
            }
            updateUI();
        });

        // 將新資料同步到雲端
        function syncToCloud() {
            set(dbRef, balances);
        }

        // 更新畫面
        function updateUI() {
            document.getElementById('amt-spend').innerText = `${balances.spend} 元`;
            document.getElementById('amt-save').innerText = `${balances.save} 元`;
            document.getElementById('amt-invest').innerText = `${balances.invest} 元`;
        }

        // 綁定按鈕事件
        document.getElementById('roll-btn').addEventListener('click', () => {
            if (balances.diceRolledThisWeek) {
                alert("這週已經擲過骰子囉！");
                return;
            }

            const diceDisplay = document.getElementById('dice-display');
            const resultText = document.getElementById('dice-result');
            const rollBtn = document.getElementById('roll-btn');
            
            diceDisplay.classList.add('spin');
            rollBtn.disabled = true;
            resultText.innerText = "轉動中...";

            setTimeout(() => {
                diceDisplay.classList.remove('spin');
                const roll = Math.floor(Math.random() * 6) + 1;
                let outcomeMsg = "";
                let change = 0;

                if (roll === 6) { 
                    diceDisplay.innerText = "🌺"; 
                    change = -Math.ceil(balances.invest / 5);
                    outcomeMsg = `遇到食人花！每 5 元損失 1 元，共損失 ${Math.abs(change)} 元。`;
                } else {
                    diceDisplay.innerText = ["⚀","⚁","⚂","⚃","⚄","⚅"][roll-1];
                    if (roll === 4) {
                        change = Math.ceil(balances.invest / 5);
                        outcomeMsg = `點數 4！每 5 元多 1 元，共增加 ${change} 元。`;
                    } else if (roll === 5) {
                        change = Math.ceil(balances.invest / 2);
                        outcomeMsg = `點數 5！每 2 元多 1 元，共增加 ${change} 元！大豐收！`;
                    } else {
                        outcomeMsg = `點數 ${roll}，本週無事發生，平平安安。`;
                    }
                }

                balances.invest += change;
                if (balances.invest < 0) balances.invest = 0; 
                
                balances.diceRolledThisWeek = true;
                resultText.innerText = outcomeMsg;
                
                syncToCloud(); // 存回雲端
                rollBtn.disabled = false;
            }, 1000);
        });

        document.getElementById('btn-interest').addEventListener('click', () => {
            let password = prompt("請輸入家長密碼以發放利息 (預設: 0000):");
            if (password === "0000") {
                let interest = Math.ceil(balances.save / 5);
                balances.save += interest;
                alert(`發放利息！每 5 元增加 1 元，共獲得 ${interest} 元利息。`);
                syncToCloud(); // 存回雲端
            } else if (password !== null) {
                alert("密碼錯誤！");
            }
        });

        document.getElementById('btn-deposit').addEventListener('click', () => {
            let s = parseInt(document.getElementById('alloc-spend').value) || 0;
            let sa = parseInt(document.getElementById('alloc-save').value) || 0;
            let i = parseInt(document.getElementById('alloc-invest').value) || 0;

            if (s + sa + i !== 15) {
                alert("三個帳戶的總和必須剛好是 15 元喔！");
                return;
            }

            let password = prompt("請家長確認存入，請輸入密碼 (預設: 0000):");
            if (password === "0000") {
                balances.spend += s;
                balances.save += sa;
                balances.invest += i;
                balances.diceRolledThisWeek = false; // 存入零用錢時，重置骰子狀態
                
                document.getElementById('dice-display').innerText = "🎲";
                document.getElementById('dice-result').innerText = "";
                
                alert("✨ 存入成功！");
                syncToCloud(); // 存回雲端
            } else if (password !== null) {
                alert("密碼錯誤！無法存入。");
            }
        });

        document.getElementById('btn-admin').addEventListener('click', () => {
            let pwd = prompt("進入後台管理，請輸入家長密碼 (預設: 0000):");
            if (pwd === "0000") {
                let newSpend = prompt("設定 [消費帳] 金額:", balances.spend);
                let newSave = prompt("設定 [儲蓄帳] 金額:", balances.save);
                let newInvest = prompt("設定 [投資帳] 金額:", balances.invest);
                
                if(newSpend !== null) balances.spend = parseInt(newSpend) || 0;
                if(newSave !== null) balances.save = parseInt(newSave) || 0;
                if(newInvest !== null) balances.invest = parseInt(newInvest) || 0;
                
                alert("金額已強制更新至雲端！");
                syncToCloud(); // 存回雲端
            }
        });
    </script>
</body>
</html>
