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
    margin: 0;
    padding-top: 50px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

#ad-placeholder1, #ad-placeholder2 {
    position: fixed;
    width: 90px;
    height: 728px;
    background: #333;
    display: flex;
    justify-content: center;
    align-items: center;
    border: 2px dashed #555;
    color: #aaa;
    font-size: 18px;
}

#ad-placeholder1 { right: 0; }
#ad-placeholder2 { left: 0; }

#coin {
    font-size: 100px;
    margin: 20px;
    transition: transform 2s ease;
}

#counter { font-weight: bold; font-size: 24px; margin:0; }
#message { font-size: 24px; margin-top: 15px; font-weight: bold; color: #0f0; }
#rules { margin: 0 0 25px 0; font-size: 18px; color: #ccc; font-style: italic; text-align:center; }

#launchBtn {
    padding: 15px 25px;
    font-size: 18px;
    cursor: pointer;
    margin-top: 15px;
    border: none;
    background: #333;
    color: white;
    border-radius: 8px;
}

#launchBtn:hover { background: #555; }

#endScreen {
    position: fixed;
    top:0; left:0;
    width:100%; height:100%;
    display:none;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    font-size: 32px;
    text-align:center;
    padding:20px;
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

/* Popup ads per schermi piccoli */
.popup-ad {
    display:none;
    position: fixed;
    top: 20px;
    left: 50%;
    transform: translateX(-50%);
    width: 90%;
    max-width: 300px;
    height: 200px;
    background:#333;
    color:#fff;
    border:2px solid #555;
    display:flex;
    justify-content:center;
    align-items:center;
    z-index:1000;
}

.close-popup {
    position:absolute;
    top:5px;
    right:10px;
    cursor:pointer;
    font-weight:bold;
    font-size:18px;
    color:#fff;
}
</style>
</head>
<body>

<h1>TrynaWinnin</h1>
<p id="rules">Flip the coin 10 times consecutively without failing. One mistake ends your chance. Can you win the secret prize??</p>

<div id="ad-placeholder1">AD</div>
<div id="ad-placeholder2">AD</div>

<div id="coin">🪙</div>
<p>Streak: <span id="counter">0</span> / 10</p>
<button id="launchBtn">Flip</button>
<div id="message"></div>

<div class="popup-ad" id="popupAd1">AD <span class="close-popup">X</span></div>
<div class="popup-ad" id="popupAd2">AD <span class="close-popup">X</span></div>

<div id="endScreen">
    <div id="endMessage"></div>
    <button id="retryBtn">Try Again</button>
</div>

<script>
(function(){
    const button = document.getElementById("launchBtn");
    const counterSpan = document.getElementById("counter");
    const messageDiv = document.getElementById("message");
    const coin = document.getElementById("coin");
    const endScreen = document.getElementById("endScreen");
    const retryBtn = document.getElementById("retryBtn");
    const endMessage = document.getElementById("endMessage");

    const popupAd1 = document.getElementById("popupAd1");
    const popupAd2 = document.getElementById("popupAd2");

    const phrases = [
        "Ok, good job… too bad there’s no prize. 😏",
        "Wow, you think that’s impressive? Keep dreaming.",
        "Nice streak… shame it ends here!",
        "Congrats… nothing for you, just a headache.",
        "You flipped it well… but not well enough. 😎",
        "Almost had it… better luck next time!",
        "Your skills? Meh, try again.",
        "So close… too bad the universe says NO.",
        "Epic fail avoided… just kidding, you failed.",
        "Not bad… for a beginner!"
    ];

    let streak = 0;
    let last = null;
    const hours24 = 24*60*60*1000;
    let lastFlip = localStorage.getItem('lastFlip');
    lastFlip = lastFlip ? parseInt(lastFlip) : 0;

    function updateBlock(){
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

    function hideAllAds(){ popupAd1.style.display='none'; popupAd2.style.display='none'; }

    function showPopupAd(ad){
        ad.style.display='flex';
    }

    function togglePopupAds(){
        if(window.innerWidth < 768){
            hideAllAds();
            let currentAd = 0;
            showPopupAd(currentAd===0?popupAd1:popupAd2);
            setInterval(()=>{
                currentAd = 1-currentAd;
                hideAllAds();
                showPopupAd(currentAd===0?popupAd1:popupAd2);
            }, 60000);
        } else {
            hideAllAds();
            document.getElementById("ad-placeholder1").style.display='flex';
            document.getElementById("ad-placeholder2").style.display='flex';
        }
    }
    togglePopupAds();
    window.addEventListener('resize', togglePopupAds);

    document.querySelectorAll('.close-popup').forEach(el=>{
        el.onclick=()=>{el.parentElement.style.display='none';};
    });

    button.onclick = ()=>{
        button.disabled=true;
        coin.style.transform="rotateY(720deg)";
        setTimeout(()=>{
            coin.style.transform="rotateY(0deg)";
            let r = Math.random()<0.5?"H":"T";

            if(last===null || last===r){
                streak++;
                last=r;
                counterSpan.textContent=streak;
                if(streak===10){
                    const phrase=phrases[Math.floor(Math.random()*phrases.length)];
                    endMessage.textContent=phrase;
                    endScreen.style.display='flex';
                    endScreen.style.color='white';
                    endScreen.style.background='black';
                    document.querySelector("h1").style.display='none';
                    document.querySelector("#rules").style.display='none';
                    document.getElementById("ad-placeholder1").style.display='none';
                    document.getElementById("ad-placeholder2").style.display='none';
                    coin.style.display='none';
                    counterSpan.parentElement.style.display='none';
                    button.style.display='none';
                    messageDiv.style.display='none';
                } else {
                    button.disabled=false;
                }
            } else {
                endMessage.textContent="Unlucky! Try Again!";
                endScreen.style.display='flex';
                endScreen.style.color='red';
                endScreen.style.background='black';
                document.querySelector("h1").style.display='none';
                document.querySelector("#rules").style.display='none';
                document.getElementById("ad-placeholder1").style.display='none';
                document.getElementById("ad-placeholder2").style.display='none';
                coin.style.display='none';
                counterSpan.parentElement.style.display='none';
                button.style.display='none';
                messageDiv.style.display='none';
                streak=0;
                last=null;
                counterSpan.textContent=streak;
                localStorage.setItem('lastFlip', Date.now());
                retryBtn.style.display='block';
            }
        },2000);
    };

    retryBtn.onclick = ()=>{
        endScreen.style.display='none';
        document.querySelector("h1").style.display='block';
        document.querySelector("#rules").style.display='block';
        coin.style.display='inline-block';
        counterSpan.parentElement.style.display='block';
        button.style.display='block';
        messageDiv.style.display='block';
        streak=0;
        last=null;
        counterSpan.textContent=streak;
        retryBtn.style.display='none';
        togglePopupAds();
        updateBlock(); // mantiene il blocco 24h
    };
})();
</script>

</body>
</html>
