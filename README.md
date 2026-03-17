<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>あなたにぴったりの勉強法診断</title>
    <link href="https://fonts.googleapis.com/css2?family=Zen+Maru+Gothic:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #4A90E2;
            --bg: #E3F2FD;
            --text: #333;
        }
        body {
            font-family: 'Zen Maru Gothic', sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container {
            background: white;
            padding: 2rem;
            border-radius: 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            width: 90%;
            max-width: 500px;
            text-align: center;
        }
        h1 { color: var(--primary); font-size: 1.5rem; }
        .question { display: none; margin-top: 20px; }
        .question.active { display: block; }
        button {
            background-color: var(--primary);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 25px;
            font-size: 1rem;
            cursor: pointer;
            transition: 0.3s;
            margin: 5px;
        }
        button:hover { opacity: 0.8; transform: translateY(-2px); }
        .result-card { display: none; text-align: left; background: #f9f9f9; padding: 20px; border-radius: 15px; }
        .icon { font-size: 3rem; display: block; text-align: center; }
        ul { padding-left: 20px; }
        li { margin-bottom: 8px; }
    </style>
</head>
<body>

<div class="container">
    <div id="start-screen">
        <h1>🔍 あなたにぴったりの勉強法診断</h1>
        <p>あなたの性格やスタイルから、最適な勉強法を見つけます。</p>
        <button onclick="startDiag()">診断スタート！</button>
    </div>

    <div id="quiz-container"></div>

    <div id="result-screen" class="result-card">
        <span class="icon" id="res-icon"></span>
        <h2 id="res-title" style="text-align:center;"></h2>
        <p id="res-desc"></p>
        <h3>💡 おすすめ勉強法</h3>
        <ul id="res-methods"></ul>
        <button onclick="location.reload()" style="width:100%; margin-top: 20px;">もう一度診断する</button>
    </div>
</div>

<script>
    // --- GASのURL設定 ---
    const GAS_URL = 'https://script.google.com/macros/s/AKfycbxq91Xv7OsDJFq8t8PoXNEiEMBoO9ikL37CHrvM2AUdEcCKTzk2O8GH4I8X5Tdrdn4qFg/exec'; 

    // --- 診断データ ---
    const types = [
        { id: 0, name: "視覚型学習者", icon: "👁️", desc: "図やグラフで理解するのが得意。カラフルなノートを作るのがおすすめ！", methods: ["マインドマップを使って情報を整理しよう", "カラーペンで重要な箇所を色分けするのが効果的", "YouTube動画や図解を積極的に活用しよう"] },
        { id: 1, name: "聴覚型学習者", icon: "🎧", desc: "聞いて理解するのが得意。音読や講義形式がおすすめ！", methods: ["声に出して読む「音読勉強法」が効果的", "友達に説明することで理解が深まる", "軽いBGMをかけながら勉強すると集中しやすい"] },
        { id: 2, name: "体験型学習者", icon: "✍️", desc: "実際に書いたり体を動かしたりすることで理解が深まるタイプ。反復練習や実験が向いています！", methods: ["何度も書いて覚える「反復書き取り」が効果的", "実験や体験型の学習に積極的に参加しよう", "ポモドーロ法（25分集中+5分休憩）で集中力UP"] },
        { id: 3, name: "読み書き型学習者", icon: "📖", desc: "文章を読んで整理することが得意。ノートにまとめることで理解が深まるタイプです。", methods: ["丁寧にノートを取り、自分の言葉でまとめ直そう", "箇条書きやリストで情報を整理するのが効果的", "問題を自分で作って解く「自作問題法」がおすすめ"] },
        { id: 4, name: "論理型学習者", icon: "🧩", desc: "理由や仕組みから理解するのが得意。「なぜ？」を追求することで力を発揮するタイプです。", methods: ["「なぜそうなるのか」を徹底的に調べてみよう", "原理・原則から覚えることで応用力がつく", "問題を解く前に「解法の流れ」を考える習慣をつけよう"] }
    ];

    const questions = [
        { q: "教科書の図やイラストに目が止まることが多い", type: 0 },
        { q: "カラーペンやマーカーを使ってノートを取るのが好き", type: 0 },
        { q: "授業を聞いているだけで内容がスッと入ってくることが多い", type: 1 },
        { q: "音読すると内容を覚えやすいと感じる", type: 1 },
        { q: "実際に手を動かしてやってみることで理解できることが多い", type: 2 },
        { q: "体験したことや実験の内容はよく覚えている", type: 2 },
        { q: "教科書や参考書を読んで自分でまとめるのが好き", type: 3 },
        { q: "文章を箇条書きやリストに整理するのが得意", type: 3 },
        { q: "「なぜそうなるのか」理由がわかると納得できる", type: 4 },
        { q: "仕組みや原理から理解しようとする方だ", type: 4 }
    ];

    let currentIdx = 0;
    let scores = [0, 0, 0, 0, 0];

    // --- 処理用関数 ---
    function startDiag() {
        const startScreen = document.getElementById('start-screen');
        if (startScreen) {
            startScreen.style.display = 'none';
            showQuestion();
        }
    }

    function showQuestion() {
        const container = document.getElementById('quiz-container');
        if (!container) return;
        container.innerHTML = `
            <div class="question active">
                <p>質問 ${currentIdx + 1} / ${questions.length}</p>
                <h3>${questions[currentIdx].q}</h3>
                <button onclick="answer(true)">あてはまる</button>
                <button onclick="answer(false)">あてはまらない</button>
            </div>
        `;
    }

    function answer(isYes) {
        if(isYes) scores[questions[currentIdx].type]++;
        currentIdx++;
        if(currentIdx < questions.length) {
            showQuestion();
        } else {
            showResult();
        }
    }

    async function showResult() {
        document.getElementById('quiz-container').style.display = 'none';
        const maxScore = Math.max(...scores);
        const resultType = types[scores.indexOf(maxScore)];

        document.getElementById('res-icon').innerText = resultType.icon;
        document.getElementById('res-title').innerText = resultType.name;
        document.getElementById('res-desc').innerText = resultType.desc;
        const list = document.getElementById('res-methods');
        list.innerHTML = ""; 
        resultType.methods.forEach(m => {
            const li = document.createElement('li');
            li.innerText = m;
            list.appendChild(li);
        });
        document.getElementById('result-screen').style.display = 'block';

        sendData(resultType.name);
    }

    async function sendData(typeName) {
        try {
            // 地域情報の取得（エラー回避のためtry-catch内）
            let locationStr = "Unknown";
            try {
                const geoRes = await fetch('https://ipapi.co/json/');
                const geoData = await geoRes.json();
                locationStr = `${geoData.region}, ${geoData.city}`;
            } catch(e) { console.log("Geo location failed"); }

            const payload = {
                timestamp: new Date().toLocaleString('ja-JP'),
                result: typeName,
                scores: scores.join(','),
                location: locationStr,
                device: navigator.userAgent
            };

            const formData = new URLSearchParams();
            formData.append('data', JSON.stringify(payload));

            await fetch(GAS_URL, {
                method: 'POST',
                mode: 'no-cors',
                body: formData,
            });
        } catch (e) {
            console.error("Data send error:", e);
        }
    }
</script>
</body>
</html>
