<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0,user-scalable=no">
<title>Neon Flappy</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;touch-action:none}
body{background:#101830;font-family:Arial,sans-serif;overflow:hidden}
canvas{display:block;width:100vw;height:100vh}
.overlay{
position:absolute;inset:0;display:flex;flex-direction:column;
justify-content:center;align-items:center;color:#fff;text-align:center;
background:linear-gradient(#2c6cff,#0f1630)
}
h1{font-size:3rem;margin-bottom:20px;text-shadow:0 0 15px #fff}
input{
padding:14px;border:none;border-radius:12px;font-size:1.1rem;
width:220px;text-align:center;margin-bottom:16px
}
button{
padding:16px 40px;border:none;border-radius:14px;
font-size:1.3rem;font-weight:bold;background:#ffe66d;
cursor:pointer
}
.hidden{display:none}
.info{font-size:1.2rem;margin:10px}
</style>
</head>
<body>

<canvas id="game"></canvas>

<div id="menu" class="overlay">
<h1>NEON FLAPPY</h1>
<input id="nameBox" maxlength="16" placeholder="Enter first name">
<button id="playBtn">PLAY</button>
</div>

<div id="gameOver" class="overlay hidden">
<h1>GAME OVER</h1>
<div id="finalScore" class="info"></div>
<div id="bestScore" class="info"></div>
<button id="againBtn">PLAY AGAIN</button>
</div>

<script>
const c=document.getElementById("game"),x=c.getContext("2d");
let w,h,bird,pipes,score,best=+localStorage.bestFlappy||0,game=false;
let speed,gap,name="Player",audio=false;

function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
addEventListener("resize",resize);resize();

function beep(f,t){
if(!audio)return;
let a=new(AudioContext||webkitAudioContext)(),o=a.createOscillator(),g=a.createGain();
o.frequency.value=f;o.connect(g);g.connect(a.destination);
g.gain.value=.05;o.start();o.stop(a.currentTime+t);
}

function start(){
name=document.getElementById("nameBox").value.trim()||"Player";
bird={x:w*.28,y:h*.45,v:0,r:16};
pipes=[];score=0;speed=3;gap=210;
game=true;
document.getElementById("menu").classList.add("hidden");
document.getElementById("gameOver").classList.add("hidden");
}

function flap(){
if(!audio){audio=true;beep(500,.05)}
if(game){bird.v=-8;beep(700,.04)}
}

function addPipe(){
let top=Math.random()*(h-gap-200)+80;
pipes.push({x:w+80,top});
}

function over(){
game=false;
if(score>best){best=score;localStorage.bestFlappy=best}
document.getElementById("finalScore").textContent="Score: "+score;
document.getElementById("bestScore").textContent="Best: "+best;
document.getElementById("gameOver").classList.remove("hidden");
beep(180,.2);
}

function update(){
if(!game)return;

bird.v+=0.42;
bird.y+=bird.v;

if(Math.random()<0.018)addPipe();

speed=3+score*0.05;
gap=Math.max(120,210-score*2);

for(let i=pipes.length-1;i>=0;i--){
let p=pipes[i];
p.x-=speed;

if(
bird.x+bird.r>p.x &&
bird.x-bird.r<p.x+70 &&
(bird.y-bird.r<p.top || bird.y+bird.r>p.top+gap)
) over();

if(!p.scored && p.x+70<bird.x){
p.scored=1;score++;beep(900,.03);
}

if(p.x<-80)pipes.splice(i,1);
}

if(bird.y<0||bird.y>h)over();
}

function draw(){
x.clearRect(0,0,w,h);

let g=x.createLinearGradient(0,0,0,h);
g.addColorStop(0,"#59f");g.addColorStop(1,"#0a1026");
x.fillStyle=g;x.fillRect(0,0,w,h);

for(let p of pipes){
x.fillStyle="#00ffb7";
x.fillRect(p.x,0,70,p.top);
x.fillRect(p.x,p.top+gap,70,h);
}

x.save();
x.translate(bird.x,bird.y);
x.rotate(bird.v*.05);
x.fillStyle="#ffe66d";
x.beginPath();
x.arc(0,0,bird.r,0,7);
x.fill();
x.fillStyle="#111";
x.beginPath();
x.arc(5,-4,2,0,7);
x.fill();
x.restore();

x.fillStyle="#fff";
x.font="bold 26px Arial";
x.fillText("Score: "+score,15,40);
