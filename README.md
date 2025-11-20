<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>TrynaWinnin</title>
  <style>
    /* Base e reset */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #1a1a1a, #0b0b0b);
      color: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    /* Colonna centrale mobile‑first */
    .container {
      width: 90%;
      max-width: 360px;
      text-align: center;
      position: relative;
    }

    h1 {
      font-size: 2.5rem;
      margin-bottom: 10px;
      color: #f39c12;
      text-shadow: 1px 1px 5px #000;
    }

    #rules {
      font-size: 1rem;
      color: #ccc;
      margin-bottom: 30px;
    }

    /* Moneta */
    #coin {
      font-size: 100px;
      margin-bottom: 20px;
      transition: transform 1s ease;
    }

    /* Streak counter */
    #counter {
      font-size: 1.5rem;
      margin-bottom: 20px;
    }

    /* Bottone Flip */
    #launchBtn {
      padding: 12px 30px;
      font-size: 1.2rem;
      background: linear-gradient(45deg, #e74c3c, #c0392b);
      border: none;
      border-radius: 12px;
      color: #fff;
      cursor: pointer;
      box-shadow: 0 4px 12px rgba(0,0,0,0.5);
      transition: transform 0.2s ease;
    }
    #launchBtn:hover {
      transform: scale(1.1);
    }

    /* Messaggio provocation / feedback */
    #message {
      margin-top: 20px;
      min-height: 24px;
      font-size: 1.1rem;
      color: #ff4d4d;
    }

    /* End screen quando perdi o vinci */
    #endScreen {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.9);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      color: #fff;
      font-size: 1.8rem;
      padding: 20px;
      display: none;
      z-index: 10;
    }
    #retryBtn {
      margin-top: 30px;
      padding: 10px 25px;
      font-size: 1.1rem;
      background: #27ae60;
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
    }

    /* Popup Ad su mobile */
    .popup-ad {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 80%;
      max-width: 300px;
      background: #333;
      color: #fff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0,0,0,0.7);
      text-align: center;
      z-index: 20;
      display: none;
    }
    .popup-ad .close-popup {
      position: absolute;
      top: 8px;
      right: 12px;
      background: transparent;
      border: none;
      color: #fff;
      font-size: 1.2rem;
      cursor: pointer;
    }

    /* Ads laterali per desktop / schermi grandi */
    .ad-box {
      position: fixed;
      width: 80px;
      height: 250px;
      background: #444;
      color: #ccc;
      display: flex;
      justify-content: center;
      align-items: center;
      border: 2px dashed #666;
      font-size: 14px;
      z-index: 5;
    }
    .ad-left { left: 10px; top: 50%; transform: translateY(-50%); }
    .ad-right { right: 10px; top: 50%; transform: translateY(-50%); }

    @media (max-width: 768px) {
      .ad-box { display: none; }
    }

  </style>
</head>
<body>
  <div class="container">
    <h1>TrynaWinnin</h1>
    <p id="rules">Flip the coin 10 times in a row without messing up. One mistake and you're out.</p>
    <div id="coin">🪙</div>
    <div id="counter">0 / 10</div>
    <button id="launchBtn">Flip</button>
    <div id="message"></div>
  </div>

  <div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
  </div>

  <div class="ad-box ad-left">AD</div>
  <div class="ad-box ad-right">AD</div>

  <div class="popup-ad" id="popupAd">
    <button class="close-popup" id="closePopupBtn">✖</button>
    <div id="popupText">Ad Promo</div>
  </div>

  <script>
    (function(){
      const coin = document.getElementById('coin');
      const btn = document.getElementById('launchBtn');
      const counterDiv = document.getElementById('counter');
      const messageDiv = document.getElementById('message');
      const endScreen = document.getElementById('endScreen');
      const endMessage = document.getElementById('endMessage');
      const retryBtn = document.getElementById('retryBtn');

      const popupAd = document.getElementById('popupAd');
      const closePopupBtn = document.getElementById('closePopupBtn');

      let streak = 0;
      let last = null;

      const losePhrases = [
        "Oh wow… you actually lost. Again.",
        "Seriously? That was your best flip?",
        "Epic fail! The coin laughs at you.",
        "Try harder next time… or don’t.",
        "You're about as lucky as a rock.",
        "Congrats, you messed up pretty badly.",
        "Is this even fun for you? Because it's brutal.",
        "That streak is dead. R.I.P.",
        "No prize for you, just embarrassment.",
        "Maybe the coin is better than you. Fact."
      ];

      function showPopupAd(){
        if(window.innerWidth <= 768){
          popupAd.style.display = 'block';
          btn.disabled = true; // blocca il gioco finché il popup è su
        }
      }

      function hidePopupAd(){
        popupAd.style.display = 'none';
        btn.disabled = false;
        // riprogramma il popup dopo 60 secondi
        setTimeout(showPopupAd, 60000);
      }

      closePopupBtn.addEventListener('click', hidePopupAd);

      // Mostra il primo popup dopo qualche secondo
      setTimeout(showPopupAd, 5000);

      function flipOnce(){
        coin.style.transform = "rotateY(720deg)";
        setTimeout(() => {
          coin.style.transform = "rotateY(0deg)";
          const r = Math.random() < 0.5 ? "H" : "T";
          if(last === null || last === r){
            streak++;
            last = r;
            counterDiv.textContent = streak + " / 10";
            if(streak === 10){
              endMessage.textContent = "You're a legend! 10 flips in a row!";
              endScreen.style.display = 'flex';
              btn.disabled = true;
            } else {
              btn.disabled = false;
            }
          } else {
            const phrase = losePhrases[Math.floor(Math.random()*losePhrases.length)];
            endMessage.textContent = phrase;
            endScreen.style.display = 'flex';
            btn.disabled = true;
          }
        }, 1000);
      }

      btn.addEventListener('click', () => {
        btn.disabled = true;
        flipOnce();
      });

      retryBtn.addEventListener('click', () => {
        streak = 0;
        last = null;
        counterDiv.textContent = "0 / 10";
        endScreen.style.display = 'none';
        btn.disabled = false;
      });

      // Se cambio dimensione schermo, gestisco ad / popup dinamicamente
      window.addEventListener('resize', () => {
        if(window.innerWidth > 768){
          popupAd.style.display = 'none';
        }
      });

    })();
  </script>
</body>
</html>
