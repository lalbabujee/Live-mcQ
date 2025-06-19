# Live-mcQ
Live mcQ generator 
<!DOCTYPE html>
<html lang="hi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Live MCQ Test Generator + OCR</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/tesseract.js/4.0.2/tesseract.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(to right, #f2f9ff, #d4e9ff);
      margin: 0;
      padding: 0;
    }
    header {
      background: #007bff;
      color: white;
      padding: 20px;
      text-align: center;
      border-radius: 0 0 20px 20px;
    }
    .container {
      max-width: 800px;
      margin: auto;
      padding: 20px;
    }
    input, button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 16px;
    }
    button {
      background-color: #28a745;
      color: white;
      border: none;
      font-weight: bold;
    }
    button:hover {
      background-color: #218838;
      cursor: pointer;
    }
    .question {
      background: white;
      padding: 15px;
      margin-top: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    .question p {
      margin: 5px 0;
    }
    .result {
      margin-top: 20px;
      padding: 20px;
      background: #e9f7ef;
      border-left: 5px solid #2ecc71;
      border-radius: 10px;
      display: none;
    }
    .leaderboard {
      margin-top: 30px;
      background: white;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    .leaderboard h3 {
      margin-top: 0;
      color: #007bff;
    }
    .score-tag {
      background: #17a2b8;
      color: white;
      padding: 6px 10px;
      border-radius: 5px;
      display: inline-block;
    }
    .motivation {
      font-weight: bold;
      color: #e67e22;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <header>
    <h1>🧠 Live MCQ Test + OCR</h1>
    <p>📸 Photo लो → 📃 Questions बनाओ → ✅ Test दो → 🏆 Rank देखो</p>
  </header>

  <div class="container">
    <input type="file" accept="image/*" capture="environment" id="imageInput" onchange="runOCR()" />
    <div id="questionsArea"></div>

    <div class="result" id="resultBox">
      <h3>📊 आपका रिजल्ट:</h3>
      <p>✅ सही जवाब: <span id="correctCount">0</span></p>
      <p>❌ गलत जवाब: <span id="wrongCount">0</span></p>
      <p>🎯 स्कोर: <span class="score-tag" id="scoreTag">0%</span></p>
      <p class="motivation" id="motivationMsg"></p>
    </div>

    <div class="leaderboard">
      <h3>🏆 लाइव लीडरबोर्ड</h3>
      <ol>
        <li>👨‍🎓 Raju - 100%</li>
        <li>👩‍🎓 Sita - 90%</li>
        <li><b>🧑‍🎓 आप - <span id="yourScore">0%</span></b></li>
      </ol>
    </div>
  </div>

  <script>
    let correctAnswers = ["B", "B"]; // आप इसे dynamic बना सकते हैं

    function runOCR() {
      const file = document.getElementById("imageInput").files[0];
      if (!file) return;

      const questionsArea = document.getElementById("questionsArea");
      questionsArea.innerHTML = "<p>🔍 Photo process हो रहा है, कृपया प्रतीक्षा करें...</p>";

      const reader = new FileReader();
      reader.onload = async function () {
        const result = await Tesseract.recognize(reader.result, 'hin+eng');
        const text = result.data.text.trim();
        renderQuestions(text);
      };
      reader.readAsDataURL(file);
    }

    function renderQuestions(text) {
      const questionsArea = document.getElementById("questionsArea");
      questionsArea.innerHTML = "";

      // Static MCQ data (OCR text ke sath integrate ho sakta hai)
      const questions = [
        {
          q: "1. भारत की राजधानी क्या है?",
          options: ["A) मुंबई", "B) दिल्ली", "C) कोलकाता", "D) चेन्नई"]
        },
        {
          q: "2. 2 + 2 कितना होता है?",
          options: ["A) 3", "B) 4", "C) 5", "D) 6"]
        }
      ];

      questions.forEach((item, index) => {
        const div = document.createElement("div");
        div.className = "question";
        div.innerHTML = `<p><b>${item.q}</b></p>` + item.options.map(opt => `
          <label><input type="radio" name="q${index}" value="${opt.charAt(0)}"> ${opt}</label><br/>
        `).join("") + (index === questions.length - 1 ? `<br/><button onclick="submitTest()">🎯 टेस्ट समाप्त करें</button>` : "");
        questionsArea.appendChild(div);
      });
    }

    function submitTest() {
      let correct = 0;
      let total = correctAnswers.length;

      correctAnswers.forEach((ans, idx) => {
        const selected = document.querySelector(`input[name='q${idx}']:checked`);
        if (selected && selected.value === ans) correct++;
      });

      const wrong = total - correct;
      const percent = Math.round((correct / total) * 100);

      document.getElementById("correctCount").innerText = correct;
      document.getElementById("wrongCount").innerText = wrong;
      document.getElementById("scoreTag").innerText = percent + "%";
      document.getElementById("yourScore").innerText = percent + "%";

      let msg = "🔥 शानदार प्रदर्शन!";
      if (percent < 80) msg = "📈 अच्छा प्रयास! और अभ्यास करें।";
      if (percent < 50) msg = "💪 मेहनत जारी रखें, आप कर सकते हैं!";

      document.getElementById("motivationMsg").innerText = msg;
      document.getElementById("resultBox").style.display = "block";
    }
  </script>
</body>
</html>
