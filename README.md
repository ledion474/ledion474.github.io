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
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        #ad-placeholder1 {
            position: fixed;
            right: 0px;
            width: 90%;
            max-width: 90px;
            height: 728px;
            background: #333;
            margin: 0 auto 25px auto;
            display: flex;
            justify-content: center;
            align-items: center;
            border: 2px dashed #555;
            color: #aaa;
            font-size: 18px;
        }

        #ad-placeholder2 {
            position: fixed;
            left: 0px;
            width: 90%;
            max-width: 90px;
            height: 728px;
            background: #333;
            margin: 0 auto 25px auto;
            display: flex;
            justify-content: center;
            align-items: center;
            border: 2px dashed #555;
            color: #aaa;
            font-size: 18px;
        }

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
            margin: 8px;
            border: none;
            background: #333;
            color: white;
            border-radius: 8px;
        }

        button:hover {
            background: #555;
        }

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
    </style>

</head>

<body>

    <h1>TrynaWinnin</h1>
    <p id="rules">Flip the coin 10 times consecutively without failing. One mistake ends your chance.
Can you win the secret prize??</p>

    <div id="ad-placeholder1">AD SPACE (Google AdSense placeholder)</div>
    <div id="ad-placeholder2">AD SPACE (Google AdSense placeholder)</div>

    <!-- Moneta -->
    <div id="coin">🪙</div>

    <p>Streak: <span id="counter">0</span> / 10</p>

    <button id="launchBtn">Flip</button>

    <div id="message"></div>
    <div id="endScreen">
        <div id="endMessage"></div>
        <button id="retryBtn">Try Again</button>
    </div>

    <script>

        (function () {

            const button = document.getElementById("launchBtn");
            const counterSpan = document.getElementById("counter");
            const messageDiv = document.getElementById("message");
            const endScreen = document.getElementById("endScreen");
            const coin = document.getElementById("coin");
            const retryBtn = document.getElementById("retryBtn");
            const endMessage = document.getElementById("endMessage");

            const phrases = [
                "Ok, nice job… too bad there’s no prize 😏",
                "You won?! Well… don’t expect anything 😎",
                "Wow, 10 in a row… the most you get is glory 😏",
                "Congrats, now you can brag… to yourself 😂",
                "Great job! Too bad it’s useless 😜",
                "You did 10? Fantastic… the only reward is my applause 👏",
                "Legend of flipping… too bad it’s just for your ego 😏",
                "Incredible… but nothing tangible for you 😬",
                "You beat the odds? Eh, so what? No prize 😎",
                "Victory! The only reward is the coin’s smile 😏"
            ];

            let streak = 0;
            let last = null;

            const hours24 = 24 * 60 * 60 * 1000; // blocco per errore
            let lastFlip = localStorage.getItem('lastFlip');
            lastFlip = lastFlip ? parseInt(lastFlip) : 0;

            function updateBlock() {
                const now = Date.now();
                const remaining = hours24 - (now - lastFlip);
                if (remaining > 0) {
                    button.disabled = true;
                    const hrs = Math.floor(remaining / (1000 * 60 * 60));
                    const mins = Math.floor((remaining % (1000 * 60 * 60)) / (1000 * 60));
                    const secs = Math.floor((remaining % (1000 * 60)) / 1000);
                    messageDiv.textContent = `Next flip available in ${hrs}h ${mins}m ${secs}s`;
                } else {
                    button.disabled = false;
                    messageDiv.textContent = '';
                }
            }

            setInterval(updateBlock, 1000);
            updateBlock();

            button.onclick = () => {
                button.disabled = true; // blocca subito per evitare doppio click
                streak = streak || 0;

                // Effetto rimbalzo / rotazione della moneta 2 secondi
                coin.style.transform = "rotateY(720deg)";

                setTimeout(() => {

                    coin.style.transform = "rotateY(0deg)";

                    let r = Math.random() < 0.5 ? "H" : "T";

                    if (last === null || last === r) {
                        streak++;
                        last = r;
                        counterSpan.textContent = streak;

                        if (streak === 10) {
                            const phrase = phrases[Math.floor(Math.random() * phrases.length)];
                            endMessage.textContent = phrase;
                            endScreen.style.display = "flex";
                            endScreen.style.color = "white";
                            endScreen.style.background = "black";

                            // Nasconde tutto il resto
                            document.querySelector("h1").style.display = "none";
                            document.querySelector("#rules").style.display = "none";
                            document.querySelector("#ad-placeholder1").style.display = "none";
                            document.querySelector("#ad-placeholder2").style.display = "none";
                            coin.style.display = "none";
                            counterSpan.parentElement.style.display = "none";
                            button.style.display = "none";
                            messageDiv.style.display = "none";

                        } else {
                            button.disabled = false; // riabilita il pulsante per il prossimo lancio
                        }

                    } else {
                        // Cambiamo il messaggio "GAME OVER" in "Unlucky! Try Again!"
                        endMessage.textContent = "Unlucky! Try Again!";
                        endScreen.style.display = "flex";
                        endScreen.style.color = "red";
                        endScreen.style.background = "black";

                        // Nasconde tutto il resto
                        document.querySelector("h1").style.display = "none";
                        document.querySelector("#rules").style.display = "none";
                        document.querySelector("#ad-placeholder1").style.display = "none";
                        document.querySelector("#ad-placeholder2").style.display = "none";
                        coin.style.display = "none";
                        counterSpan.parentElement.style.display = "none";
                        button.style.display = "none";
                        messageDiv.style.display = "none";

                        streak = 0;
                        last = null;
                        counterSpan.textContent = streak;

                        // Salva il blocco 24h solo in caso di errore
                        localStorage.setItem('lastFlip', Date.now());

                        // Mostra il pulsante "Try Again"
                        retryBtn.style.display = "block";
                    }

                }, 2000); // durata animazione moneta 2 secondi
            };

            // Funzione per il pulsante "Try Again"
            retryBtn.onclick = () => {
                // Riporta tutto alla schermata iniziale
                endScreen.style.display = "none";
                document.querySelector("h1").style.display = "block";
                document.querySelector("#rules").style.display = "block";
                document.querySelector("#ad-placeholder1").style.display = "block";
                document.querySelector("#ad-placeholder2").style.display = "block";
                coin.style.display = "inline-block";
                counterSpan.parentElement.style.display = "block";
                button.style.display = "block";
                messageDiv.style.display = "block";
                streak = 0;
                last = null;
                counterSpan.textContent = streak;
                retryBtn.style.display = "none"; // Nasconde il pulsante "Try Again"
            };

        })();

    </script>

</body>

</html>
