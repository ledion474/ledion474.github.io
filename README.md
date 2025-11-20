<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>TrynaWinnin</title>
<style>
    body {
        text-align: center;
        font-family: Arial;
        background: #111;
        color: #f5f5f5;
        padding-top: 50px;
        margin: 0;
    }

    #ad-placeholder1, #ad-placeholder2 {
        width: 90px;
        height: 728px;
        background: #333;
        display: flex;
        justify-content: center;
        align-items: center;
        border: 2px dashed #555;
        color: #aaa;
        font-size: 18px;
        position: fixed;
        top: 50%;
        transform: translateY(-50%);
    }

    #ad-placeholder1 { right: 0; }
    #ad-placeholder2 { left: 0; }

    #coin {
        font-size: 100px;
        margin: 20px;
        transition: transform 2s ease;
        display: inline-block;
    }

    button {
        padding: 15px 25px;
        font-size: 18px;
        cursor: pointer;
        margin: 15px;
        border: none;
        background: #333;
        color: white;
        border-radius: 8px;
    }

    button:hover { background: #555; }

    #counter {
        font-weight: bold;
        font-size: 24px;
    }

    #message {
        font-size: 24px;
        margin-top: 25px;
        font-weight: bold;
        color: #0f0;
    }

    #rules {
        margin-top: -10px;
        margin-bottom: 25px;
        font-size: 18px;
        color: #ccc;
        font-style: italic;
    }

    #endScreen {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 32px;
        text-align: center;
        padding: 20px;
        display: none;
        flex-direction: column;
        background: black;
    }

    #retryBtn {
        padding: 15px 25px;
        font-size: 18px;
        cursor: pointer;
        margin-top: 20px;
        border: none;
        background: #333;
        color: white;
        border-radius: 8px;
        display: none;
    }

    /* POPUP ADS per schermi piccoli */
    .popup-ad {
        display: none;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: rgba(20,20,20,0.95);
        color: white;
        z-index: 2000;
        padding: 20px;
        box-sizing: border-box;
        text-align: center;
        font-size: 32px;
        justify-content: center;
        align-items: center;
        display: flex;
    }

    .popup-ad .close-btn {
        position: absolute;
        top: 10px;
        right: 20px;
        cursor: pointer;
        font-weight: bold;
        font-size: 24px;
    }

    /* Nascondi adsense per schermi piccoli */
    @media screen and (max-width: 600px) {
        #ad-placeholder1, #ad-placeholder2 { display: none; }
    }
</style>
</head>
<body>

<h1>TrynaWinnin</h1>
<p id="rules">Flip the coin 10 times consecutively without failing. One mistake ends your chance.</p>

<div id="ad-placeholder1">AD SPACE</div>
<div id="ad-placeholder2">AD SPACE</div>

<div id="coin">🪙</div>

<p>Streak: <span id="counter">0</span> / 10</p>
<button id="launchBtn">Flip</button>

<div id="message"></div>

<div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
</div>

<!-- POPUP ADS per schermi piccoli -->
<div class="popup-ad" id="popup1">
    <span class="close-btn" onclick="this.parentElement.style.display='none'">X</span>
    Small screen ad 1
</div>
<div class="popup-ad" id="popup2">
    <span class="close-btn" onclick="this.parentElement.style.display='none'">X</span>
    Small screen ad 2
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

    const phrases = [
        "Ok nice, but too bad there’s no prize. 😏",
        "You got lucky, don’t expect a reward! 🤨",
        "10 flips? Meh… nothing for you. 😎",
        "Congrats? Just kidding, no prize. 🎉",
        "Think you won? Nope, try again!",
        "Perfect streak? Your reward is nothing!",
        "Master flipper? More like beginner luck!",
        "The coin laughs at your attempts!",
        "Incredible? Only in your dreams!",
        "Victory? Only frustration awaits!"
    ];

    let streak = 0;
    let last = null;

    const hours24 = 24*60*60*1000;
    let lastFlip = localStorage.getItem('lastFlip');
    lastFlip = lastFlip ? parseInt(lastFlip) : 0;

    function updateBlock() {
        const now = Date.now();
        const remaining = hours24 - (now - lastFlip);
        if(remaining > 0){
            button.disabled = true;
            const hrs = Math.floor(remaining / (1000*60*60));
            const mins = Math.floor((remaining % (1000*60*60)) / (1000*60));
            const secs = Math.floor((remaining % (1000*60)) / 1000);
            messageDiv.textContent = `Next flip available in ${hrs}h ${mins}m ${secs}s`;
        } else {
            button.disabled = false;
            messageDiv.textContent = '';
        }
    }
    setInterval(updateBlock,1000);
    updateBlock();

    button.onclick = () => {
        button.disabled = true;
        streak = streak || 0;

        coin.style.transform = "rotateY(720deg)";

        setTimeout(()=>{
            coin.style.transform = "rotateY(0deg)";
            let r = Math.random() < 0.5 ? "H" : "T";

            if(last === null || last === r){
                streak++;
                last = r;
                counterSpan.textContent = streak;

                if(streak===10){
                    endMessage.textContent = phrases[Math.floor(Math.random()*phrases.length)];
                    endScreen.style.display = "flex";
                    document.querySelector("h1").style.display="none";
                    document.querySelector("#rules").style.display="none";
                    document.querySelector("#ad-placeholder1").style.display="none";
                    document.querySelector("#ad-placeholder2").style.display="none";
                    coin.style.display="none";
                    counterSpan.parentElement.style.display="none";
                    button.style.display="none";
                    messageDiv.style.display="none";
                } else button.disabled=false;
            } else {
                endMessage.textContent="Unlucky! Try Again!";
                endScreen.style.display="flex";
                endScreen.style.color="red";
                document.querySelector("h1").style.display="none";
                document.querySelector("#rules").style.display="none";
                document.querySelector("#ad-placeholder1").style.display="none";
                document.querySelector("#ad-placeholder2").style.display="none";
                coin.style.display="none";
                counterSpan.parentElement.style.display="none";
                button.style.display="none";
                messageDiv.style.display="none";

                streak=0;
                last=null;
                counterSpan.textContent=streak;
                localStorage.setItem('lastFlip',Date.now());
                retryBtn.style.display="block";
            }
        },2000);
    };

    retryBtn.onclick=()=>{
        endScreen.style.display="none";
        document.querySelector("h1").style.display="block";
        document.querySelector("#rules").style.display="block";
        document.querySelector("#ad-placeholder1").style.display="block";
        document.querySelector("#ad-placeholder2").style.display="block";
        coin.style.display="inline-block";
        counterSpan.parentElement.style.display="block";
        button.style.display="block";
        messageDiv.style.display="block";
        streak=0;
        last=null;
        counterSpan.textContent=streak;
        retryBtn.style.display="none";
    };

    // POPUP ADS per schermi piccoli
    const popupAds = [document.getElementById("popup1"), document.getElementById("popup2")];
    let popupIndex = 0;
    function showPopupAd() {
        if(window.innerWidth <= 600){
            popupAds.forEach(p=>p.style.display='none');
            popupAds[popupIndex].style.display='flex';
            popupIndex = (popupIndex+1) % popupAds.length;
        }
    }
    showPopupAd();
    setInterval(showPopupAd,60000);

})();
</script>
</body>
</html>
