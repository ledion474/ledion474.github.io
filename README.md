<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>TrynaWinnin</title>
<style>
    /* Reset e body */
    * { box-sizing: border-box; margin:0; padding:0; }
    body {
        font-family: 'Arial', sans-serif;
        background: linear-gradient(135deg, #1b2735, #090a0f);
        color: #fff;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        flex-direction: column;
        overflow-x: hidden;
    }

    h1 {
        font-size: 2rem;
        margin-bottom: 10px;
        text-shadow: 0 0 10px #00ffcc;
        text-align: center;
    }

    #rules {
        font-size: 1rem;
        margin-bottom: 25px;
        color: #ccc;
        text-align: center;
        max-width: 90%;
    }

    #coin {
        font-size: 80px;
        margin: 20px;
        transition: transform 0.8s ease;
    }

    #counter {
        font-weight: bold;
        font-size: 1.2rem;
        margin-bottom: 15px;
    }

    button#launchBtn {
        padding: 15px 25px;
        font-size: 1.2rem;
        border-radius: 8px;
        border: none;
        background: #00ffcc;
        color: #111;
        cursor: pointer;
        box-shadow: 0 5px 15px rgba(0,255,204,0.3);
        transition: all 0.2s;
    }

    button#launchBtn:hover {
        background: #00e6b8;
        transform: scale(1.05);
    }

    #message {
        margin-top: 15px;
        font-size: 1rem;
        color: #ff5555;
        min-height: 1.2em;
        text-align: center;
    }

    /* End screen */
    #endScreen {
        position: fixed;
        top:0; left:0;
        width:100%; height:100%;
        background: rgba(0,0,0,0.9);
        color: #fff;
        display: none;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        padding: 20px;
        z-index: 999;
    }

    #retryBtn {
        margin-top: 20px;
        padding: 12px 25px;
        font-size: 1.1rem;
        border:none;
        border-radius: 8px;
        cursor:pointer;
        background:#00ffcc;
        color:#111;
    }

    #retryBtn:hover { background:#00e6b8; }

    /* Popup ad */
    #popupAd {
        display:none;
        position: fixed;
        top:0; left:0;
        width:100%; height:100%;
        background: rgba(0,0,0,0.95);
        z-index: 1000;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        color:#fff;
        text-align: center;
        font-size: 1.2rem;
    }

    #popupAd button {
        position: absolute;
        top:15px; right:20px;
        background:red;
        color:white;
        font-size:1.2rem;
        border:none;
        border-radius:50%;
        width:35px; height:35px;
        cursor:pointer;
    }

    /* Desktop ads - laterali */
    #ad-left, #ad-right {
        display:block;
        position:fixed;
        top:50px;
        width:90px; height:728px;
        background:#333;
        color:#aaa;
        display:flex;
        justify-content:center;
        align-items:center;
        font-size:16px;
        border:2px dashed #555;
    }

    #ad-left { left:0; }
    #ad-right { right:0; }

    /* Responsive */
    @media(max-width:768px){
        #ad-left, #ad-right { display:none; }
    }
</style>
</head>
<body>

<h1>TrynaWinnin</h1>
<p id="rules">Flip the coin 10 times consecutively without failing. One mistake ends your chance. Can you beat your friends?</p>

<div id="coin">🪙</div>
<p id="counter">Streak: 0 / 10</p>
<button id="launchBtn">Flip</button>
<div id="message"></div>

<!-- End Screen -->
<div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
</div>

<!-- Popup Ad -->
<div id="popupAd">
    <button id="closePopup">X</button>
    <div>🔥 Special Offer! Click here! 🔥</div>
</div>

<!-- Desktop Ads -->
<div id="ad-left">AD</div>
<div id="ad-right">AD</div>

<script>
(function(){
    const coin = document.getElementById("coin");
    const button = document.getElementById("launchBtn");
    const counterSpan = document.getElementById("counter");
    const messageDiv = document.getElementById("message");
    const endScreen = document.getElementById("endScreen");
    const endMessage = document.getElementById("endMessage");
    const retryBtn = document.getElementById("retryBtn");

    // Popup
    const popupAd = document.getElementById("popupAd");
    const closePopupBtn = document.getElementById("closePopup");

    let streak = 0;
    let last = null;

    const losePhrases = [
        "Ok, congrats… now cry like a baby 😏",
        "Oops, that was embarrassing 😈",
        "Better luck next time, loser!",
        "You call that skill? 😂",
        "Try again, maybe you’ll survive!",
        "Haha, almost but NO!",
        "Epic fail! Did you even try?",
        "That streak was your fantasy, not reality!",
        "Wrong move! Rage incoming?",
        "You lost… imagine being that bad!"
    ];

    const winPhrases = [
        "Wow, lucky this time! 😏",
        "Congrats… but you still suck 😂",
        "You won… try not to get cocky!",
        "Victory! Don’t let it get to your head 😈",
        "Nice one… now can you do it again?",
        "You got it! But don’t think you’re pro.",
        "Hah, beginner’s luck! 🤑",
        "You beat the odds… barely!",
        "Lucky streak! Or just skill? 🤔",
        "Congrats… shame there’s no prize!"
    ];

    // Flip function
    function flipCoin(){
        button.disabled = true;
        coin.style.transform = "rotateY(720deg)";
        setTimeout(()=>{
            coin.style.transform = "rotateY(0deg)";
            let r = Math.random() < 0.5 ? "H":"T";
            if(last === null || last === r){
                streak++;
                last = r;
                counterSpan.textContent = `Streak: ${streak} / 10`;
                if(streak === 10){
                    endMessage.textContent = winPhrases[Math.floor(Math.random()*winPhrases.length)];
                    endScreen.style.display = "flex";
                } else {
                    button.disabled = false;
                }
            } else {
                endMessage.textContent = losePhrases[Math.floor(Math.random()*losePhrases.length)];
                endScreen.style.display = "flex";
            }
        },800);
    }

    button.onclick = flipCoin;

    retryBtn.onclick = ()=>{
        endScreen.style.display = "none";
        streak=0; last=null;
        counterSpan.textContent = `Streak: ${streak} / 10`;
        button.disabled=false;
    };

    // Popup logic
    function showPopupAd(){
        if(window.innerWidth<=768){
            popupAd.style.display='flex';
            button.disabled=true;
        }
    }

    function hidePopupAd(){
        popupAd.style.display='none';
        button.disabled=false;
        setTimeout(showPopupAd,15000);
    }

    closePopupBtn.addEventListener('click', hidePopupAd);

    // Primo popup dopo 5 secondi
    setTimeout(showPopupAd,5000);

})();
</script>

</body>
</html>
