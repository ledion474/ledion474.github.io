<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<title>Banana Chaos - Corretto Mobile/Desktop</title>
<style>
body{margin:0;overflow:hidden;background:#ffe98a;font-family:Arial}
#gameCanvas{display:block;margin:auto;border:4px solid #ffcc00; background:#fff7d1;}
#ui{position:absolute;top:10px;left:10px;font-size:24px;font-weight:bold}
#gameOver,#levelUp{
    position:absolute;top:50%;left:50%;
    transform:translate(-50%,-50%);
    background:rgba(255,255,255,0.9);
    padding:25px;border-radius:20px;
    display:none;font-size:28px;text-align:center;
}
button{font-size:20px;padding:10px 20px;background:#ffcc00;border:0;border-radius:10px;cursor:pointer}

/* Joystick mobile */
#joystickContainer{
    position:absolute;
    width:140px;
    height:140px;
    border-radius:50%;
    background:rgba(255,255,255,0.4);
    display:none;
}
#joystick{
    width:60px;
    height:60px;
    border-radius:50%;
    background:#ffcc00;
    position:absolute;
    left:40px;
    top:40px;
}

/* Mobile responsive */
@media (max-width:768px){
    #gameCanvas{
        width:100vw;
        height:calc(100vw * 500 / 800);
    }
    #joystickContainer{
        bottom:10px;
        left:50%;
        transform:translateX(-50%);
        display:block;
    }
}
</style>
</head>
<body>

<div id="ui">Tempo: <span id="time">0</span> | Livello: <span id="level">1</span></div>
<canvas id="gameCanvas" width="800" height="500"></canvas>
<div id="joystickContainer"><div id="joystick"></div></div>

<div id="gameOver"><div id="msg"></div><br><button onclick="restart()">Ricomincia</button></div>
<div id="levelUp"><div id="msg2"></div><button onclick="startNextLevel()">➡ Continua</button></div>

<script>
// -------------------
// MOBILE JOYSTICK
// -------------------
function isMobile(){return /Android|iPhone|iPad|iPod/i.test(navigator.userAgent);}
const joyContainer=document.getElementById("joystickContainer");
const joy=document.getElementById("joystick");
if(isMobile()) joyContainer.style.display="block";

let joyX=0, joyY=0, joyActive=false;
joyContainer.addEventListener("touchstart",()=>joyActive=true);
joyContainer.addEventListener("touchend",()=>{joyActive=false;joyX=0;joyY=0;joy.style.left="40px";joy.style.top="40px";});
joyContainer.addEventListener("touchmove",e=>{
    const r=joyContainer.getBoundingClientRect();
    let x=e.touches[0].clientX-r.left-70;
    let y=e.touches[0].clientY-r.top-70;
    const d=Math.hypot(x,y), max=50;
    if(d>max){x*=max/d;y*=max/d;}
    joy.style.left=(40+x)+"px"; joy.style.top=(40+y)+"px";
    joyX=x/max; joyY=y/max;
});

// -------------------
// GAME VARIABLES
// -------------------
const canvas=document.getElementById("gameCanvas");
const ctx=canvas.getContext("2d");

let player={x:400,y:250,size:20,speed:4,color:"#ffdd22"};
let level=1;
let requiredTimes=[0,20,30,40,50,60];
let timeLeft = requiredTimes[level];
let timerId=null;

let enemyCount=5;
let enemySpeed=2;
let enemies=[];
let boss=null;

let gameRunning=false;
let keys={};
if(!isMobile()){
    document.addEventListener("keydown",e=>keys[e.key]=true);
    document.addEventListener("keyup",e=>keys[e.key]=false);
}

// -------------------
// SCALE FACTORS (per mobile responsive)
function getScaleX(){ return canvas.width/800; }
function getScaleY(){ return canvas.height/500; }

// -------------------
// ENEMIES
// -------------------
function initEnemies(){
    enemies=[];
    for(let i=0;i<enemyCount;i++){
        enemies.push({
            x:Math.random()*760+20,
            y:Math.random()*460+20,
            size:18,
            speed:enemySpeed+Math.random(),
            angle:Math.random()*Math.PI*2,
            color:"#ff5555"
        });
    }
    if(level===5){
        boss={x:400,y:100,size:40,speed:2,color:"#8000ff"};
    } else boss=null;
}

function moveEnemies(){
    enemies.forEach(e=>{
        e.x+=Math.cos(e.angle)*e.speed;
        e.y+=Math.sin(e.angle)*e.speed;
        if(e.x<10||e.x>790) e.angle=Math.PI-e.angle;
        if(e.y<10||e.y>490) e.angle=-e.angle;
        if(Math.hypot(e.x-player.x,e.y-player.y)<e.size+player.size){
            endGame("💥 Sei stato preso!");
        }
    });
    if(boss){
        let dx=player.x-boss.x;
        let dy=player.y-boss.y;
        let dist=Math.hypot(dx,dy);
        if(dist>0){
            boss.x+=dx/dist*boss.speed;
            boss.y+=dy/dist*boss.speed;
        }
        if(Math.hypot(boss.x-player.x,boss.y-player.y)<boss.size+player.size){
            endGame("💀 Sei stato preso dal Boss!");
        }
    }
}

// -------------------
// PLAYER
// -------------------
function movePlayer(){
    if(!isMobile()){
        if(keys["w"]||keys["ArrowUp"]) player.y-=player.speed;
        if(keys["s"]||keys["ArrowDown"]) player.y+=player.speed;
        if(keys["a"]||keys["ArrowLeft"]) player.x-=player.speed;
        if(keys["d"]||keys["ArrowRight"]) player.x+=player.speed;
    }
    if(joyActive){
        player.x+=joyX*player.speed*2; // velocità uniforme
        player.y+=joyY*player.speed*2;
    }
    player.x=Math.max(10,Math.min(790,player.x));
    player.y=Math.max(10,Math.min(490,player.y));
}

// -------------------
// DRAW
// -------------------
let particles=[];
function draw(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    const scaleX=getScaleX();
    const scaleY=getScaleY();

    // Player
    ctx.fillStyle=player.color;
    ctx.beginPath();
    ctx.arc(player.x*scaleX, player.y*scaleY, player.size*scaleX,0,Math.PI*2);
    ctx.fill();

    // Nemici piccoli
    enemies.forEach(e=>{
        ctx.fillStyle=e.color;
        ctx.beginPath();
        ctx.arc(e.x*scaleX,e.y*scaleY,e.size*scaleX,0,Math.PI*2);
        ctx.fill();
    });

    // Boss livello 5
    if(boss){
        ctx.fillStyle=boss.color;
        ctx.beginPath();
        ctx.arc(boss.x*scaleX,boss.y*scaleY,boss.size*scaleX,0,Math.PI*2);
        ctx.fill();
    }

    // Particelle level up
    particles.forEach(p=>{
        ctx.fillStyle=p.color;
        ctx.beginPath();
        ctx.arc(p.x*scaleX,p.y*scaleY,p.size*scaleX,0,Math.PI*2);
        ctx.fill();
        p.y-=p.speed;
        p.life--;
    });
    particles=particles.filter(p=>p.life>0);
}

// -------------------
// TIMER
// -------------------
function startTimer(){
    if(timerId) clearInterval(timerId);
    timeLeft = requiredTimes[level];
    document.getElementById("time").textContent = timeLeft;
    timerId=setInterval(()=>{
        if(!gameRunning){clearInterval(timerId);return;}
        timeLeft--;
        document.getElementById("time").textContent=timeLeft;
        if(timeLeft<=0){
            clearInterval(timerId);
            winLevel();
        }
    },1000);
}

// -------------------
// LEVEL UP ANIMATION
// -------------------
function showLevelUp(){
    gameRunning=false;
    const msg=document.getElementById("msg2");
    msg.innerHTML="✔ Livello "+level+" completato!";
    document.getElementById("levelUp").style.display="block";
    particles=[];
    for(let i=0;i<50;i++){
        particles.push({
            x:Math.random()*800,
            y:250,
            size:5+Math.random()*5,
            speed:1+Math.random()*3,
            color:`hsl(${Math.random()*60},100%,50%)`,
            life:50+Math.random()*50
        });
    }
}

// -------------------
// LEVEL
// -------------------
function winLevel(){
    showLevelUp();
}

// Bottone continua Level Up
function startNextLevel(){
    document.getElementById("levelUp").style.display="none";
    level++;
    enemyCount+=2;
    enemySpeed+=0.5;
    startLevel();
}

// -------------------
// GAME OVER
// -------------------
function endGame(msg){
    gameRunning=false;
    clearInterval(timerId);
    document.getElementById("msg").innerHTML=msg;
    document.getElementById("gameOver").style.display="block";
}

function restart(){
    level=1;
    enemyCount=5;
    enemySpeed=2;
    startLevel();
}

// -------------------
// START LEVEL
// -------------------
function startLevel(){
    gameRunning=true;
    player.x=400; player.y=250;
    document.getElementById("level").textContent=level;
    document.getElementById("gameOver").style.display="none";
    document.getElementById("levelUp").style.display="none";
    initEnemies();
    startTimer();
    gameLoop();
}

// -------------------
// GAME LOOP
// -------------------
function gameLoop(){
    if(!gameRunning) return;
    movePlayer();
    moveEnemies();
    draw();
    requestAnimationFrame(gameLoop);
}

// -------------------
// START GAME
// -------------------
startLevel();
</script>
</body>
</html>
