# Asteroids plus iphone  
  
<!DOCTYPE html>  
<html>  
<head>  
<meta charset="utf-8">  
<title>Asteroids</title>  
  
<style>  
  
body{  
margin:0;  
background:black;  
color:white;  
font-family:monospace;  
overflow:hidden;  
}  
  
canvas{  
display:block;  
}  
  
#hud{  
position:absolute;  
top:10px;  
left:10px;  
font-size:20px;  
}  
  
#touch{  
position:absolute;  
bottom:20px;  
left:50%;  
transform:translateX(-50%);  
display:flex;  
gap:10px;  
}  
  
button{  
font-size:20px;  
padding:15px;  
background:#111;  
color:white;  
border:1px solid white;  
}  
  
</style>  
</head>  
  
<body>  
  
<div id="hud">  
Score: <span id="score">0</span><br>  
Highscore: <span id="high">0</span>  
</div>  
  
<canvas id="game"></canvas>  
  
<div id="touch">  
<button id="left">◀</button>  
<button id="thrust">▲</button>  
<button id="right">▶</button>  
<button id="fire">●</button>  
<button id="hyper">H</button>  
</div>  
  
<script>  
  
const canvas = document.getElementById("game")  
const ctx = canvas.getContext("2d")  
  
canvas.width = window.innerWidth  
canvas.height = window.innerHeight  
  
let score = 0  
let high = localStorage.asteroidsHigh || 0  
document.getElementById("high").innerText = high  
  
let audio = new AudioContext()  
  
function beep(freq,time){  
  
let o = audio.createOscillator()  
let g = audio.createGain()  
  
o.connect(g)  
g.connect(audio.destination)  
  
o.frequency.value = freq  
o.type = "square"  
  
o.start()  
  
g.gain.setValueAtTime(0.1,audio.currentTime)  
g.gain.exponentialRampToValueAtTime(0.001,audio.currentTime+time)  
  
o.stop(audio.currentTime+time)  
  
}  
  
const ship={  
x:canvas.width/2,  
y:canvas.height/2,  
vx:0,  
vy:0,  
angle:0,  
radius:15  
}  
  
let bullets=[]  
let asteroids=[]  
let ufos=[]  
  
function wrap(o){  
  
if(o.x<0)o.x=canvas.width  
if(o.x>canvas.width)o.x=0  
if(o.y<0)o.y=canvas.height  
if(o.y>canvas.height)o.y=0  
  
}  
  
function createAsteroid(x,y,size){  
  
let shape=[]  
  
for(let i=0;i<12;i++)  
shape.push(Math.random()*size)  
  
asteroids.push({  
x,y,size,  
vx:(Math.random()*2-1)*1.5,  
vy:(Math.random()*2-1)*1.5,  
shape  
})  
  
}  
  
for(let i=0;i<5;i++)  
createAsteroid(Math.random()*canvas.width,Math.random()*canvas.height,60)  
  
function spawnUFO(){  
  
ufos.push({  
x:Math.random()<0.5?0:canvas.width,  
y:Math.random()*canvas.height,  
vx:Math.random()<0.5?2:-2  
})  
  
}  
  
setInterval(spawnUFO,15000)  
  
const keys={}  
  
document.addEventListener("keydown",e=>{  
  
keys[e.code]=true  
  
if(e.code==="Space")shoot()  
if(e.code==="KeyH")hyperspace()  
  
})  
  
document.addEventListener("keyup",e=>keys[e.code]=false)  
  
function shoot(){  
  
beep(800,0.1)  
  
bullets.push({  
x:ship.x,  
y:ship.y,  
vx:Math.cos(ship.angle)*8,  
vy:Math.sin(ship.angle)*8,  
life:60  
})  
  
}  
  
function hyperspace(){  
  
ship.x=Math.random()*canvas.width  
ship.y=Math.random()*canvas.height  
  
beep(200,0.3)  
  
}  
  
function update(){  
  
if(keys["ArrowLeft"])ship.angle-=0.08  
if(keys["ArrowRight"])ship.angle+=0.08  
  
if(keys["ArrowUp"]){  
  
ship.vx+=Math.cos(ship.angle)*0.1  
ship.vy+=Math.sin(ship.angle)*0.1  
  
beep(100,0.02)  
  
}  
  
ship.x+=ship.vx  
ship.y+=ship.vy  
  
ship.vx*=0.99  
ship.vy*=0.99  
  
wrap(ship)  
  
bullets.forEach(b=>{  
  
b.x+=b.vx  
b.y+=b.vy  
b.life--  
  
wrap(b)  
  
})  
  
bullets=bullets.filter(b=>b.life>0)  
  
asteroids.forEach(a=>{  
  
a.x+=a.vx  
a.y+=a.vy  
wrap(a)  
  
})  
  
ufos.forEach(u=>{  
  
u.x+=u.vx  
wrap(u)  
  
})  
  
checkCollisions()  
  
}  
  
function addScore(p){  
  
score+=p  
  
if(score>high){  
  
high=score  
localStorage.asteroidsHigh=high  
  
}  
  
document.getElementById("score").innerText=score  
document.getElementById("high").innerText=high  
  
}  
  
function checkCollisions(){  
  
bullets.forEach((b,bi)=>{  
  
asteroids.forEach((a,ai)=>{  
  
let dx=b.x-a.x  
let dy=b.y-a.y  
  
if(Math.hypot(dx,dy)<a.size){  
  
bullets.splice(bi,1)  
  
if(a.size>25){  
  
createAsteroid(a.x,a.y,a.size/2)  
createAsteroid(a.x,a.y,a.size/2)  
  
}  
  
asteroids.splice(ai,1)  
  
let pts=20  
if(a.size<50)pts=50  
if(a.size<30)pts=100  
  
addScore(pts)  
  
beep(400,0.1)  
  
}  
  
})  
  
})  
  
bullets.forEach((b,bi)=>{  
  
ufos.forEach((u,ui)=>{  
  
let dx=b.x-u.x  
let dy=b.y-u.y  
  
if(Math.hypot(dx,dy)<20){  
  
bullets.splice(bi,1)  
ufos.splice(ui,1)  
  
addScore(200)  
  
beep(600,0.2)  
  
}  
  
})  
  
})  
  
}  
  
function drawShip(){  
  
ctx.beginPath()  
  
ctx.moveTo(  
ship.x+Math.cos(ship.angle)*20,  
ship.y+Math.sin(ship.angle)*20  
)  
  
ctx.lineTo(  
ship.x+Math.cos(ship.angle+2.5)*20,  
ship.y+Math.sin(ship.angle+2.5)*20  
)  
  
ctx.lineTo(  
ship.x+Math.cos(ship.angle-2.5)*20,  
ship.y+Math.sin(ship.angle-2.5)*20  
)  
  
ctx.closePath()  
  
ctx.stroke()  
  
}  
  
function draw(){  
  
ctx.clearRect(0,0,canvas.width,canvas.height)  
  
ctx.strokeStyle="white"  
  
drawShip()  
  
bullets.forEach(b=>{  
  
ctx.fillRect(b.x,b.y,2,2)  
  
})  
  
asteroids.forEach(a=>{  
  
ctx.beginPath()  
  
for(let i=0;i<a.shape.length;i++){  
  
let ang=(Math.PI*2/a.shape.length)*i  
let r=a.size+a.shape[i]  
  
let x=a.x+Math.cos(ang)*r  
let y=a.y+Math.sin(ang)*r  
  
if(i===0)ctx.moveTo(x,y)  
else ctx.lineTo(x,y)  
  
}  
  
ctx.closePath()  
ctx.stroke()  
  
})  
  
ufos.forEach(u=>{  
  
ctx.strokeRect(u.x-10,u.y-5,20,10)  
  
})  
  
}  
  
function loop(){  
  
update()  
draw()  
  
requestAnimationFrame(loop)  
  
}  
  
loop()  
  
document.getElementById("left").ontouchstart=()=>keys["ArrowLeft"]=true  
document.getElementById("left").ontouchend=()=>keys["ArrowLeft"]=false  
  
document.getElementById("right").ontouchstart=()=>keys["ArrowRight"]=true  
document.getElementById("right").ontouchend=()=>keys["ArrowRight"]=false  
  
document.getElementById("thrust").ontouchstart=()=>keys["ArrowUp"]=true  
document.getElementById("thrust").ontouchend=()=>keys["ArrowUp"]=false  
  
document.getElementById("fire").ontouchstart=shoot  
document.getElementById("hyper").ontouchstart=hyperspace  
  
</script>  
  
</body>  
</html>  
