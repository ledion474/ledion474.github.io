<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>TrynaWinnin</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #0d0f12, #1a1c22);
      color: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }

    .container {
      width: 90%;
      max-width: 380px;
      text-align: center;
      position: relative;
    }

    h1 {
      font-size: 2.8rem;
      margin-bottom: 15px;
      color: #f1c40f;
      text-shadow: 2px 2px #000;
    }

    #rules {
      font-size: 1rem;
      color: #bbb;
      margin-bottom: 30px;
    }

    #coin {
      font-size: 100px;
      margin-bottom: 25px;
      transition: transform 0.8s ease;
    }

    #counter {
      font-size: 1.5rem;
      margin-bottom: 25px;
    }

    #launchBtn {
      padding: 14px 35px;
      font-size: 1.2rem;
      background: linear-gradient(45deg, #e74c3c, #d35400);
      color: #fff;
      border: none;
      border-radius: 12px;
      cursor: pointer;
      box-shadow: 0 4px 12px rgba(0,0,0,0.6);
      transition: transform 0.2s, background 0.3s;
    }
    #launchBtn:hover {
      transform: scale(1.05);
      background: linear-gradient(45deg, #ff5e57, #e67e22);
    }

    #message {
      margin-top: 20px;
      min-height: 24px;
      font-size: 1.1rem;
      color: #e74c3c;
    }

    #endScreen {
      position: fixed;
      top: 0; left: 0;
      width: 100%; height: 100%;
      background: rgba(0,0,0,0.92);
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      font-size: 1.9rem;
      color: #fff;
      padding: 20px;
      text-align: center;
      z-index: 50;
      display: none;
    }

    #retryBtn {
      margin-top: 30px;
      padding: 12px 30px;
      font-size: 1.1rem;
      background: #27ae60;
      border: none;
      border-radius: 8px;
      color: #fff;
      cursor: pointer;
      box-shadow: 0 3px 10px rgba(0,0,0,0.5);
    }

    .ad-box {
      position: fixed;
      width: 80px;
      height: 250px;
      background: #2c3e50;
      color: #aaa;
      display: flex;
      justify-content: center;
      align-items: center;
      border: 2px dashed #555;
      font-size: 14px;
      z-index: 10;
    }
    .ad-left { left: 8px; top: 50%; transform: translateY(-50%); }
    .ad-right { right: 8px; top: 50%; transform: translateY(-50%); }

    .popup-ad {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 80%;
      max-width: 300px;
      background: #34495e;
      color: #fff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 0 20px rgba(0,0,0,0.8);
      text-align: center;
      z-index: 100;
      display: none;
    }

    .popup-ad button.close-popup {
      position: absolute;
      top: 8px;
      right: 12px;
      background: transparent;
      border: none;
      color: #fff;
      font-size: 1.3rem;
      cursor: pointer;
    }

    @media(max-width: 768px) {
      .ad-box { display: none; }
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>TrynaWinnin</h1>
    <p id="rules">Flip the coin 10 times in a row. One mistake and you’re out. Ready?</p>
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
    <button class="close-popup" id="closePopup">✖</button>
    <div id="popupText">🔥 Special Offer!</div>
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
      const closePopupBtn = document.getElementById('closePopup');

      let streak = 0;
      let last = null;

      const loseMessages = [
        "Oh wow… you actually lost. Again.",
        "Seriously? That was your best flip?",
        "Epic fail! The coin laughs at you.",
        "Try harder next time… or don’t.",
        "You're about as lucky as a rock.",
        "Congrats, you messed up pretty badly.",
        "Is this even fun for you? Because it's brutal.",
        "That streak is dead. R.I.P.",
        "No prize for you, just embarrassment.",
        "Maybe the coin is better than you. Fact.",
        "You call that a flip? Pathetic.",
        "Losing never looked so consistent.",
        "Try flipping with your eyes closed… still fail.",
        "Are you even paying attention? Because you lost.",
        "Your streak died a sad, slow death."
      ];

      // Mostra popup ad, blocca il tasto
      function showPopupAd(){
        if(window.innerWidth <= 768){
          popupAd.style.display = 'block';
          btn.disabled = true;
        }
      }
      // Chiudi popup
      function hidePopupAd(){
        popupAd.style.display = 'none';
        btn.disabled = false;
        setTimeout(showPopupAd, 25000); // ogni 25s
      }

      closePopupBtn.addEventListener('click', hidePopupAd);
      // Primo popup dopo breve attesa
      setTimeout(showPopupAd, 5000);

      function flipOnce(){
        coin.style.transform = "rotateY(720deg)";
        setTimeout(() => {
          coin.style.transform = "rotateY(0deg)";
          let r = Math.random() < 0.5 ? "H" : "T";
          if(last === null || r === last){
            streak++;
            last = r;
            counterDiv.textContent = streak + " / 10";
            if(streak === 10){
              endMessage.textContent = "Legendary! You hit 10 in a row!";
              endScreen.style.display = 'flex';
              btn.disabled = true;
            } else {
              btn.disabled = false;
            }
          } else {
            const msg = loseMessages[Math.floor(Math.random()*loseMessages.length)];
            endMessage.textContent = msg;
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

      window.addEventListener('resize', () => {
        if(window.innerWidth > 768){
          popupAd.style.display = 'none';
        }
      });

    })();
  </script>
</body>
</html>
