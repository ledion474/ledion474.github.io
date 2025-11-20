<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>TrynaWinnin</title>
<style>
    /* RESET & BODY */
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
        font-family: 'Arial', sans-serif;
        background: linear-gradient(to bottom, #0f0c29, #302b63, #24243e);
        color: #fff;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: flex-start;
        min-height: 100vh;
        overflow-x: hidden;
    }

    h1 {
        margin-top: 40px;
        font-size: 3rem;
        text-shadow: 2px 2px 10px #000;
        text-align: center;
    }

    #rules {
        margin: 10px 20px 30px 20px;
        font-size: 1.2rem;
        text-align: center;
        font-style: italic;
        color: #ccc;
    }

    /* COIN */
    #coin {
        font-size: 120px;
        margin: 20px;
        transition: transform 1s ease;
    }

    /* COUNTER */
    #counter-container {
        text-align: center;
        font-size: 1.5rem;
        margin-bottom: 20px;
    }

    /* BUTTON */
    #launchBtn {
        padding: 15px 40px;
        font-size: 1.5rem;
        cursor: pointer;
        border: none;
        border-radius: 10px;
        background: linear-gradient(45deg,#f39c12,#e74c3c);
        color: #fff;
        box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        transition: transform 0.2s;
    }
    #launchBtn:hover { transform: scale(1.1); }

    /* MESSAGE */
    #message {
        margin-top: 20px;
        font-size: 1.2rem;
        font-weight: bold;
    }

    /* END SCREEN */
    #endScreen {
        position: fixed;
        top:0; left:0;
        width: 100%; height: 100%;
        background: rgba(0,0,0,0.95);
        color: #fff;
        display: none;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        padding: 20px;
        font-size: 2rem;
        z-index: 9999;
    }
    #retryBtn {
        margin-top: 30px;
        padding: 15px 35px;
        font-size: 1.5rem;
        border: none;
        border-radius: 10px;
        background: #27ae60;
        cursor: pointer;
    }

    /* ADS */
    .ad {
        position: fixed;
        width: 120px;
        height: 600px;
        background: #444;
        border: 2px dashed #aaa;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 14px;
        color: #fff;
        z-index: 500;
        transition: all 0.5s;
    }
    #ad-left { left: 10px; top: 50%; transform: translateY(-50%); }
    #ad-right { right: 10px; top: 50%; transform: translateY(-50%); }

    /* POPUP ADS MOBILE */
    .popup-ad {
        position: fixed;
        top: 50px;
        left: 50%;
        transform: translateX(-50%);
        width: 90%;
        max-width: 350px;
        background: #222;
        color: #fff;
        border: 2px solid #fff;
        padding: 20px;
        display: none;
        z-index: 10000;
        border-radius: 10px;
        text-align: center;
    }
    .popup-ad .close {
        position: absolute;
        top: 5px;
        right: 10px;
        cursor: pointer;
        font-weight: bold;
    }

    /* RESPONSIVE */
    @media (max-width: 768px) {
        .ad { display: none; }
        #launchBtn { font-size: 1.3rem; padding: 12px 30px; }
        #coin { font-size: 100px; }
        #counter-container { font-size: 1.3rem; }
    }
</style>
</head>
<body>

<h1>TrynaWinnin</h1>
<p id="rules">Flip the coin 10 times in a row. One mistake ends your streak. Can you beat the odds?</p>

<!-- Ads -->
<div id="ad-left" class="ad">AD LEFT</div>
<div id="ad-right" class="ad">AD RIGHT</div>

<!-- Popup Ads Mobile -->
<div id="popup1" class="popup-ad">
    <span class="close" onclick="closePopup('popup1')">X</span>
    Mobile Ad #1
</div>
<div id="popup2" class="popup-ad">
    <span class="close" onclick="closePopup('popup2')">X</span>
    Mobile Ad #2
</div>

<!-- Coin -->
<div id="coin">🪙</div>

<div id="counter-container">Streak: <span id="counter">0</span> / 10</div>

<button id="launchBtn">Flip</button>

<div id="message"></div>

<div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
</div>

<script>
(function(){
    const button = document.getElementById("launchBtn");
    const counterSpan = document.getElementById("counter");
    const messageDiv = document.getElementById("message");
    const endScreen = document.getElementById("endScreen");
    const coin = document.getElementById("coin");
    const retryBtn = document.getElementById("retryBtn");
    const endMessage = document.getElementById("endMessage");

    // Popup ads
    const popup1 = document.getElementById("popup1");
    const popup2 = document.getElementById("popup2");
    let showFirstPopup = true;

    const phrases = [
        "Wow, lucky you… no prize anyway!",
        "Keep dreaming! You lost again. 😏",
        "Oops! Maybe coins aren’t your thing.",
        "Better luck next time… or not!",
        "Nice try, but the universe laughs at you!",
        "Streak broken! Don’t quit your day job.",
        "Hah! Is that all you got?",
        "Flipping coins, losing glory… classic!",
        "Pathetic! You’ll need more luck.",
        "Try harder next time, maybe in another life."
    ];

    let streak = 0;
    let last = null;

    const block24h = 24 * 60 * 60 * 1000;
    let lastFlip = localStorage.getItem('lastFlip');
    lastFlip = lastFlip ? parseInt(lastFlip) : 0;

    function updateBlock() {
        const now = Date.now();
        const remaining = block24h - (now - lastFlip);
        if (remaining > 0) {
            button.disabled = true;
            const hrs = Math.floor(remaining / (1000*60*60));
            const mins = Math.floor((remaining % (1000*60*60)) / (1000*60));
            const secs = Math.floor((remaining % (1000*60)) / 1000);
            messageDiv.textContent = `Next flip in ${hrs}h ${mins}m ${secs}s`;
        } else {
            button.disabled = false;
            messageDiv.textContent = '';
        }
    }
    setInterval(updateBlock, 1000);
    updateBlock();

    function showPopup(){
        if(window.innerWidth <= 768){
            if(showFirstPopup){
                popup1.style.display = "block";
                popup2.style.display = "none";
            } else {
                popup2.style.display = "block";
                popup1.style.display = "none";
            }
            showFirstPopup = !showFirstPopup;
        }
    }
    setInterval(showPopup, 60000); // every 60 sec

    window.closePopup = function(id){
        document.getElementById(id).style.display = "none";
    }

    button.onclick = () => {
        button.disabled = true;
        coin.style.transform = "rotateY(720deg)";
        setTimeout(() => {
            coin.style.transform = "rotateY(0deg)";
            let r = Math.random() < 0.5 ? "H" : "T";

            if(last===null || last===r){
                streak++;
                last = r;
                counterSpan.textContent = streak;

                if(streak === 10){
                    endMessage.textContent = "You got the perfect streak! Lucky you, still no prize!";
                    endScreen.style.display = "flex";
                    hideAll();
                } else {
                    button.disabled = false;
                }

            } else {
                endMessage.textContent = phrases[Math.floor(Math.random()*phrases.length)];
                endScreen.style.display = "flex";
                streak = 0;
                last = null;
                counterSpan.textContent = streak;
                localStorage.setItem('lastFlip', Date.now());
                retryBtn.style.display = "block";
                hideAll();
            }
        }, 1500);
    }

    function hideAll(){
        document.querySelector("h1").style.display = "none";
        document.querySelector("#rules").style.display = "none";
        document.querySelector("#coin").style.display = "none";
        document.querySelector("#counter-container").style.display = "none";
        document.querySelector("#launchBtn").style.display = "none";
        messageDiv.style.display = "none";
        document.getElementById("ad-left").style.display = "none";
        document.getElementById("ad-right").style.display = "none";
    }

    retryBtn.onclick = () => {
        endScreen.style.display = "none";
        document.querySelector("h1").style.display = "block";
        document.querySelector("#rules").style.display = "block";
        document.querySelector("#coin").style.display = "block";
        document.querySelector("#counter-container").style.display = "block";
        document.querySelector("#launchBtn").style.display = "block";
        messageDiv.style.display = "block";
        retryBtn.style.display = "none";
        updateBlock();
        if(window.innerWidth > 768){
            document.getElementById("ad-left").style.display = "block";
            document.getElementById("ad-right").style.display = "block";
        }
    }

})();
</script>
</body>
</html>
