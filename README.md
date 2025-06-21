# Hhhhhh
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>純前端多益線上測驗</title>
<style>
  body {
    font-family: "微軟正黑體", Arial, sans-serif;
    background: #f0f4f8;
    margin: 0; padding: 0;
    display: flex;
    justify-content: center;
    min-height: 100vh;
    align-items: center;
  }
  #app {
    background: white;
    max-width: 600px;
    width: 90%;
    padding: 30px 40px;
    border-radius: 12px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.15);
  }
  h2 {
    text-align: center;
    color: #1a73e8;
  }
  .input-group {
    margin: 15px 0;
  }
  label {
    display: block;
    margin-bottom: 6px;
    font-weight: 600;
  }
  input[type=text], input[type=password] {
    width: 100%;
    padding: 10px 14px;
    border-radius: 6px;
    border: 1px solid #ccc;
    font-size: 16px;
    box-sizing: border-box;
    transition: border-color 0.3s;
  }
  input[type=text]:focus, input[type=password]:focus {
    border-color: #1a73e8;
    outline: none;
  }
  button {
    width: 100%;
    background: #1a73e8;
    color: white;
    font-weight: 700;
    padding: 12px;
    font-size: 18px;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    margin-top: 20px;
    transition: background-color 0.3s;
  }
  button:hover {
    background: #155db8;
  }
  .error {
    color: #e94b4b;
    margin-top: 8px;
    text-align: center;
  }
  .question {
    margin-top: 20px;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 10px;
  }
  .question h3 {
    margin: 0 0 10px;
    color: #333;
  }
  .options label {
    display: block;
    margin-bottom: 8px;
    cursor: pointer;
    user-select: none;
  }
  .explanation {
    margin-top: 12px;
    background: #eef6ff;
    border-left: 4px solid #1a73e8;
    padding: 10px 14px;
    border-radius: 6px;
    color: #0b3a91;
    font-style: italic;
  }
  .correct {
    background: #d4edda;
    border-color: #c3e6cb;
  }
  .incorrect {
    background: #f8d7da;
    border-color: #f5c6cb;
  }
  .result-summary {
    font-size: 20px;
    font-weight: 700;
    margin-top: 15px;
    color: #1a73e8;
    text-align: center;
  }
  .logout-btn {
    margin-top: 15px;
    background: #e94b4b;
  }
  .stats-section {
    margin-top: 30px;
  }
  .stats-section h3 {
    margin-bottom: 10px;
    text-align: center;
  }
  .stats-list {
    max-height: 150px;
    overflow-y: auto;
    border: 1px solid #ddd;
    padding: 10px;
    border-radius: 8px;
    background: #fafafa;
    font-size: 14px;
  }
  .stats-list div {
    padding: 5px 0;
    border-bottom: 1px solid #eee;
  }
  .btn-inline {
    width: auto;
    display: inline-block;
    margin-left: 10px;
    padding: 6px 14px;
    font-size: 14px;
  }
</style>
</head>
<body>
<div id="app"></div>

<script>
(() => {
  const app = document.getElementById('app');

  // 模擬帳號密碼 (純前端，不安全)
  const USERS = [
    { username: 'testuser', password: '123456' },
    { username: 'admin', password: 'admin123' },
  ];

  // 多益題目
  const QUESTIONS = [
    {
      id: 1,
      question: 'The meeting has been postponed until next Monday. What does "postponed" mean?',
      options: ['Cancelled', 'Delayed', 'Rescheduled earlier', 'Shortened'],
      answer: 'Delayed',
      explanation: 'Postponed means to delay something to a later time.',
    },
    {
      id: 2,
      question: 'Please submit the report by the end of the day. What does "submit" mean?',
      options: ['Ignore', 'Send in', 'Delete', 'Copy'],
      answer: 'Send in',
      explanation: 'Submit means to present or deliver something for consideration.',
    },
    {
      id: 3,
      question: 'The new software update will enhance security features. What does "enhance" mean?',
      options: ['Reduce', 'Improve', 'Remove', 'Ignore'],
      answer: 'Improve',
      explanation: 'Enhance means to improve or increase the quality of something.',
    },
  ];

  // LocalStorage 資料存取
  function loadStats() {
    try {
      const stats = localStorage.getItem('toeic_stats');
      return stats ? JSON.parse(stats) : [];
    } catch {
      return [];
    }
  }

  function saveStats(stats) {
    localStorage.setItem('toeic_stats', JSON.stringify(stats));
  }

  let currentUser = null;

  // 登入畫面
  function renderLogin() {
    app.innerHTML = `
      <h2>登入多益測驗系統</h2>
      <div class="input-group">
        <label>帳號</label>
        <input type="text" id="login-username" placeholder="請輸入帳號" />
      </div>
      <div class="input-group">
        <label>密碼</label>
        <input type="password" id="login-password" placeholder="請輸入密碼" />
      </div>
      <button id="login-btn">登入</button>
      <div class="error" id="login-error"></div>
      <div style="margin-top:20px; font-size:14px; color:#555; text-align:center;">
        預設帳號：testuser 密碼：123456<br/>
        管理員帳號：admin 密碼：admin123
      </div>
    `;

    document.getElementById('login-btn').onclick = () => {
      const username = document.getElementById('login-username').value.trim();
      const password = document.getElementById('login-password').value.trim();
      const errElem = document.getElementById('login-error');

      const user = USERS.find(u => u.username === username && u.password === password);
      if (!user) {
        errElem.textContent = '帳號或密碼錯誤！';
        return;
      }
      errElem.textContent = '';
      currentUser = user;

      if (currentUser.username === 'admin') {
        renderStats(); // 管理員直接進統計頁
      } else {
        renderQuiz();
      }
    };
  }

  // 測驗頁面
  function renderQuiz() {
    app.innerHTML = `
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <div>歡迎，<b>${currentUser.username}</b></div>
        <button id="logout-btn" class="logout-btn" style="width:auto;">登出</button>
      </div>
      <h2>多益線上測驗</h2>
      <form id="quiz-form"></form>
      <button id="submit-btn">提交測驗</button>
      <div id="submit-error" class="error"></div>
    `;

    const form = document.getElementById('quiz-form');
    QUESTIONS.forEach(q => {
      const div = document.createElement('div');
      div.className = 'question';
      div.innerHTML = `<h3>${q.id}. ${q.question}</h3>`;
      const opts = document.createElement('div');
      opts.className = 'options';
      q.options.forEach(opt => {
        const label = document.createElement('label');
        label.innerHTML = `<input type="radio" name="q${q.id}" value="${opt}" /> ${opt}`;
        opts.appendChild(label);
      });
      div.appendChild(opts);
      form.appendChild(div);
    });

    document.getElementById('logout-btn').onclick = () => {
      currentUser = null;
      renderLogin();
    };

    document.getElementById('submit-btn').onclick = e => {
      e.preventDefault();
      const answers = [];
      let allAnswered = true;
      QUESTIONS.forEach(q => {
        const radios = document.querySelectorAll(`input[name="q${q.id}"]`);
        let selected = null;
        radios.forEach(r => {
          if (r.checked) selected = r.value;
        });
        if (!selected) allAnswered = false;
        answers.push({ questionId: q.id, answer: selected });
      });
      if (!allAnswered) {
        document.getElementById('submit-error').textContent = '請完成所有題目再提交！';
        return;
      }
      document.getElementById('submit-error').textContent = '';
      renderResult(answers);
    };
  }

  // 顯示測驗結果
  function renderResult(answers) {
    const form = document.getElementById('quiz-form');
    form.innerHTML = '';

    let score = 0;
    let content = '';

    QUESTIONS.forEach(q => {
      const userAns = answers.find(a => a.questionId === q.id).answer;
      const correct = userAns === q.answer;
      if (correct) score++;
      content += `
        <div class="question ${correct ? 'correct' : 'incorrect'}">
          <h3>${q.id}. ${q.question}</h3>
          <p>你的答案：<b>${userAns}</b></p>
          <p>正確答案：<b>${q.answer}</b></p>
          <div class="explanation">解釋：${q.explanation}</div>
        </div>
      `;
    });

    form.innerHTML = content;
    document.getElementById('submit-btn').style.display = 'none';

    const summary = document.createElement('div');
    summary.className = 'result-summary';
    summary.textContent = `你的得分：${score} / ${QUESTIONS.length}`;
    form.parentNode.insertBefore(summary, form);

    saveUserResult(currentUser.username, score, answers);

    const againBtn = document.createElement('button');
    againBtn.textContent = '再測一次';
    againBtn.className = 'btn-inline';
    againBtn.onclick = () => {
      renderQuiz();
      summary.remove();
      againBtn.remove();
      document.getElementById('submit-btn').style.display = 'block';
    };
    form.parentNode.appendChild(againBtn);

    // 管理員也可由結果頁直接看統計
    if (currentUser.username === 'admin') {
      const statsBtn = document.createElement('button');
      statsBtn.textContent = '查看答題統計';
      statsBtn.className = 'btn-inline';
      statsBtn.onclick = () => {
        renderStats();
        summary.remove();
        againBtn.remove();
        statsBtn.remove();
      };
      form.parentNode.appendChild(statsBtn);
    }
  }

  // 存儲答題紀錄
  function saveUserResult(username, score, answers) {
    const stats = loadStats();
    stats.push({
      username,
      score,
      answers,
      date: new Date().toISOString(),
    });
    saveStats(stats);
  }

  // 答題統計頁面（管理員用）
  function renderStats() {
    app.innerHTML = `
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <h2>答題統計紀錄</h2>
        <button id="back-btn" class="btn-inline" style="background:#1a73e8; color:#fff;">返回</button>
        <button id="logout-btn" class="btn-inline" style="background:#e94b4b; color:#fff;">登出</button>
      </div>
      <div class="stats-section">
        <h3>所有使用者答題記錄</h3>
        <div class="stats-list" id="stats-list"></div>
      </div>
    `;

    document.getElementById('back-btn').onclick = () => {
      if (currentUser.username === 'admin') {
        renderStats(); // 管理員看統計
      } else {
        renderQuiz();
      }
    };

    document.getElementById('logout-btn').onclick = () => {
      currentUser = null;
      renderLogin();
    };

    const listDiv = document.getElementById('stats-list');
    const stats = loadStats();
    if (stats.length === 0) {
      listDiv.textContent = '目前無答題記錄。';
      return;
    }

    stats.slice().reverse().forEach((r, i) => {
      const div = document.createElement('div');
      div.innerHTML = `
        <b>${i + 1}. ${r.username}</b> - 得分: ${r.score} / ${QUESTIONS.length} - 時間: ${new Date(r.date).toLocaleString()}
        <br/>答題詳情:
        <ul>
          ${r.answers.map(a => {
            const q = QUESTIONS.find(q => q.id === a.questionId);
            const isCorrect = a.answer === q.answer;
            return `<li>Q${a.questionId}: 你的答案 <b>${a.answer || '無'}</b> - ${isCorrect ? '✔️' : '❌'} (正確答案: ${q.answer})</li>`;
          }).join('')}
        </ul>
      `;
      listDiv.appendChild(div);
    });
  }

  // 啟動入口
  renderLogin();
})();
</script>
</body>
</html>
