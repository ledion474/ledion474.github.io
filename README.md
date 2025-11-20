<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>TrynaWinnin</title>
<style>
    /* Base page style */
    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: linear-gradient(135deg, #1e1e1e, #111);
        color: #f5f5f5;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
    }

    /* Central column */
    .game-column {
        text-align: center;
        width: 90%;
        max-width: 400px;
    }

    h1 {
        font-size: 2.5em;
        margin-bottom: 10px;
        color: #FFD700;
        text-shadow: 1px 1px 5px #000;
    }

    #rules {
        font-style: italic;
        color: #ccc;
        margin-bottom: 25px;
    }

    #coin {
        font-size: 100px;
        margin: 20px 0;
        display: inline-block;
        transition: transform 0.7s ease;
    }

    #counter {
        font-weight: bold;
        font-size: 24px;
    }

    button {
        padding: 15px 25px;
        font-size: 18px;
        cursor: pointer;
        margin-top: 15px;
        border: none;
        background: #ff6600;
        color: white;
        border-radius: 12px;
        transition: background 0.3s;
    }

    button:hover {
        background: #ff8533;
    }

    #message {
        font-size: 20px;
        margin-top: 20px;
        min-height: 28px;
        font-weight: bold;
        color: #ff3333;
    }

    #endScreen {
        position: fixed;
        top:0;
        left:0;
        width:100%;
        height:100%;
        background: rgba(0,0,0,0.9);
        display:flex;
        flex-direction: column;
        justify-content:center;
        align-items:center;
        font-size: 28px;
        color:white;
        display:none;
        padding: 20px;
        text-align: center;
    }

    #retryBtn {
        padding: 12px 22px;
        font-size: 18px;
        margin-top: 20px;
        background: #ff6600;
        color: white;
        border-radius: 10px;
        border:none;
        cursor: pointer;
    }

    /* Ads styling */
    .ad-box {
        position: fixed;
        width: 90px;
        height: 300px;
        background: #333;
        color: #aaa;
        font-size: 16px;
        display: flex;
        justify-content: center;
        align-items: center;
        border: 2px dashed #555;
        z-index: 1000;
        transition: all 0.5s;
    }

    .ad-left { left: 10px; top: 50%; transform: translateY(-50%); }
    .ad-right { right: 10px; top: 50%; transform: translateY(-50%); }

    /* Popup for mobile */
    .popup-ad {
        position: fixed;
        width: 80%;
        max-width: 300px;
        height: 250px;
        background: #222;
        color: #fff;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        display: none;
        z-index: 2000;
        border-radius: 15px;
        box-shadow: 0 0 15px #000;
        padding: 10px;
    }

    .popup-ad button.close-popup {
        position: absolute;
        top:5px;
        right:8px;
        background: transparent;
        color:white;
        border:none;
        font-size:18px;
        cursor:pointer;
    }

    @media(max-width:768px){
        .ad-box { display:none; }
    }
</style>
</head>
<body>

<div class="game-column">
    <h1>TrynaWinnin</h1>
    <p id="rules">Flip the coin 10 times consecutively without failing. Can you beat the streak?</p>
    <div id="coin">🪙</div>
    <p>Streak: <span id="counter">0</span> / 10</p>
    <button id="launchBtn">Flip</button>
    <div id="message"></div>
</div>

<div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
</div>

<!-- Ads desktop -->
<div class="ad-box ad-left">AD SPACE</div>
<div class="ad-box ad-right">AD SPACE</div>

<!-- Popup ad mobile -->
<div class="popup-ad" id="popupAd">
    <button class="close-popup" onclick="closePopup()">✖</button>
    MOBILE AD POPUP
</div>

<script>
(function(){
    const coin = document.getElementById('coin');
    const btn = document.getElementById('launchBtn');
    const counterSpan = document.getElementById('counter');
    const messageDiv = document.getElementById('message');
    const endScreen = document.getElementById('endScreen');
    const endMessage = document.getElementById('endMessage');
    const retryBtn = document.getElementById('retryBtn');
    const popupAd = document.getElementById('popupAd');

    let streak = 0;
    let last = null;

    const loseMessages = [
        "Ok, nice try… oh wait, you still lost!",
        "Better luck next time… not that it helps!",
        "Are you even trying or just guessing?",
        "Wow… that streak didn’t last long!",
        "Hah! Losing looks good on you!",
        "Ouch, that's gonna sting… again.",
        "Flip, fail, repeat. Classic.",
        "Epic fail! You can do better… maybe.",
        "The coin laughs at your attempts!",
        "Not even close… keep trying, loser!"
    ];

    function flipCoin(){
        coin.style.transform = "rotateY(720deg)";
        setTimeout(()=>{
            coin.style.transform = "rotateY(0deg)";
            let r = Math.random() < 0.5 ? "H":"T";

            if(last===null || last===r){
                streak++;
                last=r;
                counterSpan.textContent=streak;
                if(streak===10){
                    endMessage.textContent="Unbelievable! You got 10 in a row!";
                    endScreen.style.display="flex";
                    btn.disabled=true;
                }
            } else {
                const msg = loseMessages[Math.floor(Math.random()*loseMessages.length)];
                endMessage.innerHTML="<span style='color:#ff4d4d;'>"+msg+"</span>";
                endScreen.style.display="flex";
                btn.disabled=true;
            }
        },700);
    }

    btn.addEventListener('click',flipCoin);

    retryBtn.addEventListener('click',()=>{
        streak=0;
        last=null;
        counterSpan.textContent=streak;
        endScreen.style.display='none';
        btn.disabled=false;
    });

    // Popup ads logic
    let showLeft = true;
    function showPopupAd(){
        if(window.innerWidth<=768){
            popupAd.style.display='flex';
        }
    }

    function closePopup(){
        popupAd.style.display='none';
        setTimeout(showPopupAd,60000); // 60s per popup successivo
    }

    // First popup after 10s
    setTimeout(showPopupAd,10000);

    // Dynamic resize handling
    window.addEventListener('resize',()=>{
        if(window.innerWidth>768){
            document.querySelectorAll('.ad-box').forEach(ad=>ad.style.display='flex');
            popupAd.style.display='none';
        } else {
            document.querySelectorAll('.ad-box').forEach(ad=>ad.style.display='none');
        }
    });

})();
</script>

</body>
</html>
