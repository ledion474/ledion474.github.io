<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TrynaWinnin</title>
    <style>
        body {
            margin: 0;
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #1a1a1a, #111);
            color: #f5f5f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            overflow-x: hidden;
        }

        h1 {
            margin-top: 40px;
            font-size: 3em;
            text-shadow: 2px 2px #000;
        }

        #rules {
            font-size: 1.2em;
            color: #ccc;
            margin: 10px 0 40px;
            text-align: center;
        }

        #coin {
            font-size: 120px;
            margin: 20px;
            transition: transform 1s ease;
        }

        #counter {
            font-weight: bold;
            font-size: 2em;
            margin-bottom: 20px;
        }

        #launchBtn {
            padding: 15px 40px;
            font-size: 1.5em;
            cursor: pointer;
            background: linear-gradient(45deg, #ff5722, #e91e63);
            border: none;
            border-radius: 12px;
            color: #fff;
            text-shadow: 1px 1px #000;
            box-shadow: 0 5px 15px rgba(0,0,0,0.4);
            transition: transform 0.2s ease;
            position: relative;
            z-index: 10;
        }

        #launchBtn:hover {
            transform: scale(1.1);
        }

        #message {
            font-size: 1.2em;
            margin-top: 15px;
            color: #0f0;
            min-height: 1.5em;
        }

        #endScreen {
            position: fixed;
            top:0;
            left:0;
            width:100%;
            height:100%;
            background: rgba(0,0,0,0.95);
            display:flex;
            flex-direction:column;
            justify-content:center;
            align-items:center;
            text-align:center;
            font-size:2em;
            color:#fff;
            display:none;
            padding:20px;
            z-index:1000;
        }

        #retryBtn {
            margin-top: 30px;
            padding: 15px 30px;
            font-size: 1.2em;
            cursor: pointer;
            background: linear-gradient(45deg, #2196f3, #00bcd4);
            border:none;
            border-radius:10px;
            color:#fff;
            text-shadow:1px 1px #000;
        }

        /* POPUP ADS */
        .popup-ad {
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #333;
            color: #fff;
            padding: 15px;
            border-radius: 10px;
            display: none;
            z-index: 999;
            width: 90%;
            max-width: 300px;
            text-align: center;
        }

        .popup-ad .close-btn {
            position: absolute;
            top:5px;
            right:10px;
            cursor:pointer;
            font-weight:bold;
        }

        /* Desktop ads */
        #ad-placeholder1, #ad-placeholder2 {
            position: fixed;
            top: 50px;
            width: 90px;
            height: 728px;
            background: #333;
            display:flex;
            justify-content:center;
            align-items:center;
            color:#aaa;
            border:2px dashed #555;
            font-size:18px;
        }

        #ad-placeholder1 { right:0; }
        #ad-placeholder2 { left:0; }

        @media screen and (max-width: 768px) {
            #ad-placeholder1, #ad-placeholder2 {
                display:none;
            }
            .popup-ad {
                display:block;
            }

            #coin {
                font-size: 100px;
                margin: 20px 0;
            }

            #launchBtn {
                position: relative;
                margin-top: 10px;
            }
        }
    </style>
</head>
<body>

    <h1>TrynaWinnin</h1>
    <p id="rules">Flip the coin 10 times consecutively without failing. Can you beat the coin?</p>

    <div id="ad-placeholder1">AD SPACE</div>
    <div id="ad-placeholder2">AD SPACE</div>

    <!-- Coin -->
    <div id="coin">🪙</div>

    <div id="counter">0 / 10</div>

    <button id="launchBtn">Flip</button>
    <div id="message"></div>

    <div class="popup-ad" id="popupAd">Ad Popup <span class="close-btn">X</span></div>

    <div id="endScreen">
        <div id="endMessage"></div>
        <button id="retryBtn">Try Again</button>
    </div>

    <script>
    (function() {

        const button = document.getElementById("launchBtn");
        const counterDiv = document.getElementById("counter");
        const coin = document.getElementById("coin");
        const messageDiv = document.getElementById("message");
        const endScreen = document.getElementById("endScreen");
        const endMessage = document.getElementById("endMessage");
        const retryBtn = document.getElementById("retryBtn");
        const popupAd = document.getElementById("popupAd");
        const popupClose = popupAd.querySelector(".close-btn");

        const phrases = [
            "Oops… better luck next life, loser!",
            "Not even close… try crying about it!",
            "Wow… you really thought you could win?",
            "Pathetic! Is that the best you’ve got?",
            "Try again, maybe one day you’ll be lucky.",
            "Keep flipping… maybe you’ll hit something one day!",
            "Haha, did you really think you could beat the coin?",
            "Loser detected! Refresh your dignity next time.",
            "Epic fail! Your streak ends here.",
            "Better go practice on a virtual coin, champ!",
            "What a disaster… even my grandma could do better!",
            "Your coin skills are tragic.",
            "Try harder, or just quit already.",
            "The coin laughs at your incompetence.",
            "Seriously? That’s your best effort?",
            "You call that flipping?",
            "I’ve seen toddlers with better luck!",
            "Maybe next century you’ll get lucky.",
            "Patience isn’t your strong suit, is it?",
            "Sigh… the coin wins again!"
        ];

        let streak = 0;
        let last = null;

        function flipCoin() {
            coin.style.transform = "rotateY(720deg)";

            setTimeout(() => {
                coin.style.transform = "rotateY(0deg)";
                let r = Math.random() < 0.5 ? "H" : "T";

                if (last === null || last === r) {
                    streak++;
                    last = r;
                    counterDiv.textContent = streak + " / 10";

                    if (streak === 10) {
                        endMessage.textContent = "Incredible! You beat the coin!";
                        endScreen.style.display = "flex";
                    } else {
                        button.disabled = false;
                    }
                } else {
                    const phrase = phrases[Math.floor(Math.random()*phrases.length)];
                    endMessage.textContent = phrase;
                    endScreen.style.display = "flex";
                    streak = 0;
                    last = null;
                    counterDiv.textContent = "0 / 10";
                }
            }, 1000);
        }

        button.onclick = () => {
            button.disabled = true;
            flipCoin();
        };

        retryBtn.onclick = () => {
            endScreen.style.display = "none";
            streak = 0;
            last = null;
            counterDiv.textContent = "0 / 10";
            button.disabled = false;
        };

        // POPUP ADS
        function showPopup() {
            popupAd.style.display = "block";
        }

        popupClose.onclick = () => {
            popupAd.style.display = "none";
            setTimeout(showPopup, 60000); // next popup in 60 sec
        }

        if(window.innerWidth <= 768) {
            setTimeout(showPopup, 1000); // first popup after 1 sec
        }

    })();
    </script>
</body>
</html>
