# 1001YSquiz
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>你是哪種社工？</title>
    <style>
        body { font-family: 'Arial', 'Microsoft JhengHei', sans-serif; background-color: #e6f0ff; display: flex; justify-content: center; align-items: center; min-height: 100vh; margin: 0; color: #333; }
        .game-container { background-color: #fff; padding: 40px; border-radius: 12px; box-shadow: 0 8px 30px rgba(0, 0, 0, 0.1); width: 90%; max-width: 700px; text-align: center; transition: all 0.5s ease-in-out; }
        h1, h2 { color: #004d99; }
        .progress-bar { width: 100%; height: 8px; background-color: #f0f0f0; border-radius: 4px; overflow: hidden; margin-bottom: 30px; }
        .progress { height: 100%; background-color: #4CAF50; transition: width 0.4s ease; }
        .question-box { margin-bottom: 20px; font-size: 1.1em; line-height: 1.8; min-height: 120px; text-align: left; }
        .options { display: flex; flex-direction: column; gap: 15px; }
        .option-btn { background-color: #f0f8ff; border: 2px solid #b3d1ff; border-radius: 8px; padding: 15px; font-size: 1em; cursor: pointer; transition: all 0.3s ease; text-align: left; }
        .option-btn:hover { background-color: #d1e8ff; border-color: #1a73e8; transform: translateY(-3px); }
        .start-btn, #submit-btn { background-color: #1a73e8; color: white; padding: 15px 30px; border: none; border-radius: 8px; font-size: 1.2em; cursor: pointer; transition: background-color 0.3s ease; }
        #submit-btn:disabled { background-color: #aaa; cursor: not-allowed; }
        #result-screen { text-align: left; }
        #result-box { margin-top: 20px; padding: 25px; background-color: #f7f9fa; border-left: 5px solid #0066cc; border-radius: 8px; }
        #result-box p { margin: 10px 0; font-size: 1em; line-height: 1.6; }
        #data-form { margin-top: 40px; }
        .form-group { margin-bottom: 20px; text-align: left; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: bold; }
        .form-group input { width: 100%; padding: 10px; border: 1px solid #ddd; border-radius: 5px; box-sizing: border-box; }
        .loader { border: 4px solid #f3f3f3; border-top: 4px solid #3498db; border-radius: 50%; width: 20px; height: 20px; animation: spin 2s linear infinite; display: none; margin: 0 auto; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body>

<div class="game-container">
    <div id="intro-screen">
        <h1>你是哪種社工？</h1>
        <p>歡迎來到社工的世界！透過三個情境題，發掘你的潛在特質。</p>
        <button class="start-btn" onclick="startGame()">開始測驗</button>
    </div>

    <div id="quiz-screen" style="display:none;">
        <div class="progress-bar"><div id="progress" class="progress"></div></div>
        <div class="question-box">
            <h2 id="question-text"></h2>
            <div class="options" id="options-container"></div>
        </div>
    </div>

    <div id="result-screen" style="display:none;">
        <h2 id="result-title"></h2>
        <div id="result-box">
            <p><b style="color: #0066cc;">性格特質：</b> <span id="character-trait"></span></p>
            <p><b style="color: #0066cc;">適合出路：</b> <span id="suitable-jobs"></span></p>
        </div>
        
        <div id="data-form">
            <p>請填寫您的資料，以將結果傳送給老師。</p>
            <div class="form-group">
                <label for="name">您的姓名：</label>
                <input type="text" id="name" required>
            </div>
            <div class="form-group">
                <label for="email">您的電子郵件：</label>
                <input type="email" id="email" required>
            </div>
            <button id="submit-btn" onclick="submitResult()">送出結果</button>
            <div id="loader" class="loader"></div>
            <p id="submit-status" style="margin-top: 15px; color: green;"></p>
        </div>
    </div>
</div>

<script>
    const WEB_APP_URL = 'https://script.google.com/macros/s/AKfycbxajPgXERUoH8HtvYO92Ywa5DL8mmClJFh7Ryer0AYCXmsi_hVYHHm07gS_c0Kga4qVHg/exec'; // **請將此處替換為您在第一步複製的網址**

    const questions = [
        { text: '你是社區發展中心的社工。有一天，一位焦慮的家長衝進辦公室，說他的孩子因為校園霸凌而不願意上學，情緒非常崩潰。你會怎麼做？', options: [ { text: '先冷靜下來，仔細詢問家長事情發生的詳細經過，以便後續分析與制定策略。', code: 'I' }, { text: '馬上與家長一起討論並寫下幾項立即能採取的行動，例如尋求法律協助。', code: 'R' }, { text: '輕拍家長的肩膀，讓他坐下，遞上一杯溫水，先同理他的感受。', code: 'S' }, { text: '邀請家長一起畫出或寫下他與孩子此刻的感受，透過非語言的方式幫助情緒釋放。', code: 'A' } ] },
        { text: '你需要為一個經濟困難的家庭尋找外界幫助。你手上有很多資訊，但時間有限，你會選擇哪種方式來有效連結資源？', options: [ { text: '優先打電話給一位長期合作的企業家朋友，直接說明狀況，尋求贊助。', code: 'E' }, { text: '仔細查閱所有政府補助、民間基金會的申請流程與規定，確保每個步驟都合法合規。', code: 'C' }, { text: '舉辦一個小型社區座談會，讓大家一起集思廣益，共同找出解決方案。', code: 'S' }, { text: '深入分析這家人的困境根源，研究類似案例的成功模式，設計個人化資源計畫。', code: 'I' } ] },
        { text: '你發現社區裡許多獨居老人因為缺乏社交而感到孤單。你希望設計一個創新的服務方案來改善這個問題。你的第一個想法是什麼？', options: [ { text: '舉辦一場「回憶故事沙龍」，讓長輩們分享人生故事，創造跨世代的連結。', code: 'A' }, { text: '說服幾個年輕的志工團隊，一起發起一個線上線下結合的陪伴專案。', code: 'E' }, { text: '設計一套簡單的運動或園藝活動，讓長輩們能動手動腳，在活動中自然互動。', code: 'R' }, { text: '規劃一個系統化的陪伴計畫，包括固定的探訪頻率和紀錄表，確保關懷持續。', code: 'C' } ] }
    ];

    let currentQuestionIndex = 0;
    let scores = { 'R': 0, 'I': 0, 'A': 0, 'S': 0, 'E': 0, 'C': 0 };
    let finalResult = {};

    const introScreen = document.getElementById('intro-screen');
    const quizScreen = document.getElementById('quiz-screen');
    const resultScreen = document.getElementById('result-screen');
    const questionText = document.getElementById('question-text');
    const optionsContainer = document.getElementById('options-container');
    const progress = document.getElementById('progress');
    const resultTitle = document.getElementById('result-title');
    const characterTrait = document.getElementById('character-trait');
    const suitableJobs = document.getElementById('suitable-jobs');
    const submitBtn = document.getElementById('submit-btn');
    const loader = document.getElementById('loader');
    const submitStatus = document.getElementById('submit-status');
    
    const resultsData = { /* (您的結果資料，同上一次回覆) */
        'SI': { character: '...', job: '...' }, 'SE': { character: '...', job: '...' }, 'RI': { character: '...', job: '...' }, 'EA': { character: '...', job: '...' }, 'CS': { character: '...', job: '...' }, 'AR': { character: '...', job: '...' }, 'IC': { character: '...', job: '...' }
    };

    function startGame() {
        introScreen.style.display = 'none';
        quizScreen.style.display = 'block';
        showQuestion();
    }

    function showQuestion() {
        progress.style.width = `${(currentQuestionIndex / questions.length) * 100}%`;
        if (currentQuestionIndex < questions.length) {
            const currentQuestion = questions[currentQuestionIndex];
            questionText.textContent = `第 ${currentQuestionIndex + 1} 題：${currentQuestion.text}`;
            optionsContainer.innerHTML = '';
            currentQuestion.options.forEach(option => {
                const button = document.createElement('button');
                button.classList.add('option-btn');
                button.textContent = option.text;
                button.onclick = () => selectOption(option.code);
                optionsContainer.appendChild(button);
            });
        } else {
            progress.style.width = '100%';
            setTimeout(showResult, 500);
        }
    }

    function selectOption(code) {
        scores[code]++;
        currentQuestionIndex++;
        showQuestion();
    }

    function showResult() {
        quizScreen.style.display = 'none';
        resultScreen.style.display = 'block';

        const sortedScores = Object.keys(scores).sort((a, b) => scores[b] - scores[a]);
        const finalCode = sortedScores[0] + sortedScores[1];

        const result = resultsData[finalCode] || { character: '你的特質非常多元！', job: '請多方嘗試，探索你最感興趣的助人方向。' };
        
        finalResult = {
            code: finalCode,
            character: result.character,
            job: result.job
        };

        resultTitle.textContent = `恭喜你！你是 ${finalCode} 型社工！`;
        characterTrait.textContent = result.character;
        suitableJobs.textContent = result.job;
    }

    async function submitResult() {
        const name = document.getElementById('name').value;
        const email = document.getElementById('email').value;

        if (!name || !email) {
            alert('請完整填寫您的姓名和電子郵件！');
            return;
        }

        submitBtn.disabled = true;
        loader.style.display = 'block';
        submitStatus.textContent = '正在送出中...';

        const dataToSend = {
            name: name,
            email: email,
            code: finalResult.code,
            character: finalResult.character,
            job: finalResult.job
        };

        try {
            const response = await fetch(WEB_APP_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(dataToSend)
            });
            const result = await response.json();
            
            if (result.result === 'success') {
                submitStatus.textContent = '送出成功！您的結果已傳送給Haolilife。';
                submitStatus.style.color = 'green';
            } else {
                submitStatus.textContent = '送出失敗，請再試一次。';
                submitStatus.style.color = 'red';
            }
        } catch (error) {
            submitStatus.textContent = '連線錯誤，請檢查網路後再試一次。';
            submitStatus.style.color = 'red';
        } finally {
            loader.style.display = 'none';
            submitBtn.disabled = false;
        }
    }
</script>

</body>
</html>
