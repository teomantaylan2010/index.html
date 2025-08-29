index.html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Calculatrice Galaxie Vocale</title>
  <style>
    body {
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      display: flex;
      align-items: center;
      justify-content: center;
      height: 100vh;
      color: #fff;
      background: radial-gradient(circle at 20% 20%, #0f0c29, #302b63, #24243e);
      background-size: 200% 200%;
      animation: galaxy 20s ease infinite;
      overflow: hidden;
    }

    @keyframes galaxy {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }

    body::before {
      content: "";
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background-image: radial-gradient(1px 1px at 1px 1px, #fff, rgba(255,255,255,0));
      background-size: 2px 2px;
      animation: stars 60s linear infinite;
      z-index: -1;
    }

    @keyframes stars {
      from { transform: translateY(0); }
      to { transform: translateY(-1000px); }
    }

    .calculator {
      width: 90%;
      max-width: 320px;
      background: rgba(0, 0, 0, 0.6);
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.5);
    }

    .display {
      background: #222;
      color: #0f0;
      font-size: 2rem;
      padding: 10px;
      border-radius: 5px;
      text-align: right;
      overflow-x: auto;
      margin-bottom: 15px;
    }

    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
    }

    button {
      padding: 15px;
      font-size: 1.2rem;
      border: none;
      border-radius: 5px;
      background: #444;
      color: #fff;
      cursor: pointer;
      transition: background 0.3s;
    }

    button:hover { background: #666; }
    button:active { background: #888; }
    #micro { background: #1e90ff; }
    #micro:hover { background: #3aa3ff; }
    #clear { background: #ff4d4d; }
    #clear:hover { background: #ff6666; }
    #equals { background: #28a745; }
    #equals:hover { background: #42b85c; }
  </style>
</head>
<body>
  <div class="calculator">
    <div id="display" class="display">0</div>
    <div class="buttons">
      <button data-value="7">7</button>
      <button data-value="8">8</button>
      <button data-value="9">9</button>
      <button data-value="/">/</button>
      <button data-value="4">4</button>
      <button data-value="5">5</button>
      <button data-value="6">6</button>
      <button data-value="*">*</button>
      <button data-value="1">1</button>
      <button data-value="2">2</button>
      <button data-value="3">3</button>
      <button data-value="-">-</button>
      <button data-value="0">0</button>
      <button data-value=".">.</button>
      <button id="micro">🎤</button>
      <button data-value="+">+</button>
      <button id="clear">C</button>
      <button id="equals" style="grid-column: span 3;">=</button>
    </div>
  </div>

  <script>
    const display = document.getElementById('display');
    let expression = '';

    function updateDisplay() {
      display.textContent = expression || '0';
    }

    document.querySelectorAll('.buttons button').forEach(btn => {
      const value = btn.getAttribute('data-value');
      if (value) {
        btn.addEventListener('click', () => {
          expression += value;
          updateDisplay();
        });
      }
    });

    document.getElementById('clear').addEventListener('click', () => {
      expression = '';
      updateDisplay();
    });

    document.getElementById('equals').addEventListener('click', calculate);

    function calculate() {
      try {
        const result = eval(expression);
        if (!isFinite(result)) throw new Error('Erreur');
        expression = result.toString();
      } catch (e) {
        expression = 'Erreur';
      }
      updateDisplay();
      speak(expression);
    }

    const numberWords = {
      'zéro':0,'zero':0,'un':1,'deux':2,'trois':3,'quatre':4,'cinq':5,'six':6,'sept':7,'huit':8,'neuf':9,
      'dix':10,'onze':11,'douze':12,'treize':13,'quatorze':14,'quinze':15,'seize':16,'dix-sept':17,'dix-huit':18,'dix-neuf':19,'vingt':20,
      'trente':30,'quarante':40,'cinquante':50,'soixante':60,'soixante-dix':70,'quatre-vingt':80,'quatre-vingt-dix':90,'cent':100
    };
    const operations = {
      'plus':'+','moins':'-','fois':'*','x':'*','multiplié':'*','multiplie':'*','divisé':'/','divise':'/'
    };

    function wordsToExpression(text) {
      return text.toLowerCase().replace(/-/g,' ').split(/\s+/).map(w => {
        if (operations[w]) return operations[w];
        if (!isNaN(parseFloat(w))) return w;
        if (numberWords[w] !== undefined) return numberWords[w];
        return '';
      }).join('');
    }

    let recognition;
    if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
      const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
      recognition = new SpeechRecognition();
      recognition.lang = 'fr-FR';
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;

      recognition.onresult = event => {
        const spoken = event.results[0][0].transcript;
        expression = wordsToExpression(spoken);
        updateDisplay();
        calculate();
      };
    }

    document.getElementById('micro').addEventListener('click', () => {
      if (recognition) recognition.start();
    });

    function speak(text) {
      if ('speechSynthesis' in window) {
        const utter = new SpeechSynthesisUtterance(text);
        utter.lang = 'fr-FR';
        speechSynthesis.speak(utter);
      }
    }
  </script>
</body>
</html>
