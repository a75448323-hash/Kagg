<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Помой уточку</title>

<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>

:root{
--bg1:#ffb3d1;
--bg2:#ffd9ec;
--text:#3a2a33;
}

body{
margin:0;
font-family:sans-serif;
background:linear-gradient(180deg,var(--bg1),var(--bg2));
text-align:center;
color:var(--text);
overflow:hidden;
touch-action:none;
}

#page{
height:100vh;
overflow:auto;
}

#game{
position:relative;
width:100%;
height:320px;
background:radial-gradient(circle,#ffe6f2,#ffc1df);
}

#duck{
font-size:120px;
position:absolute;
left:50%;
top:50%;
transform:translate(-50%,-50%);
}

#sponge{
position:absolute;
font-size:50px;
transform:translate(-50%,-50%);
pointer-events:none;
}

.bubble{
position:absolute;
font-size:18px;
animation:foam 1s linear forwards;
}

@keyframes foam{
0%{opacity:1;transform:translateY(0)}
100%{opacity:0;transform:translateY(-40px)}
}

#progressBox{
width:80%;
height:22px;
background:#eee;
border-radius:20px;
margin:12px auto;
overflow:hidden;
}

#progress{
height:100%;
width:0%;
background:#ff5fa3;
}

button{
margin:4px;
padding:8px;
border-radius:10px;
border:none;
}

.slot{
font-size:40px;
}

.topBtn{
position:fixed;
top:10px;
width:44px;
height:44px;
border-radius:50%;
background:#ff8fb8;
color:white;
border:none;
font-size:18px;
}

#shopBtn{left:10px}
#settingsBtn{right:10px}

.panel{
display:none;
position:fixed;
top:60px;
left:50%;
transform:translateX(-50%);
background:white;
padding:15px;
border-radius:15px;
box-shadow:0 0 10px rgba(0,0,0,0.2);
}

</style>
</head>

<body>

<button id="shopBtn" class="topBtn">🛒</button>
<button id="settingsBtn" class="topBtn">⚙</button>

<div id="page">

<h2>🦆 Помой уточку</h2>

<div id="game">
<div id="duck">🦆</div>
<div id="sponge">🧽</div>
</div>

<div id="progressBox">
<div id="progress"></div>
</div>

<div>
Инструмент:
<button onclick="setTool('soap')">🧼 Мыло</button>
<button onclick="setTool('sponge')">🧽 Губка</button>
</div>

<div>
💰 <span id="coins">0</span> |
🎟 <span id="tickets">0</span> |
📦 <span id="cases">0</span> |
⭐ Lv <span id="level">0</span>
</div>

<br><br><br>

</div>

<div id="shop" class="panel">

<h3>🧼 Бафы</h3>

<button onclick="buySoap()">
Мыло Lv <span id="soapLv">1</span> |
<span id="soapCost"></span>
</button>

<button onclick="buySponge()">
Губка Lv <span id="spongeLv">1</span> |
<span id="spongeCost"></span>
</button>

<h3>🎰 Казино</h3>

<div id="slot" class="slot">❔ ❔ ❔</div>
<button onclick="spin()">Крутить (1 билет)</button>

<h3>📦 Кейсы</h3>

<button onclick="openCase()">Открыть кейс</button>

</div>

<div id="settings" class="panel">

<h3>⚙ Настройки</h3>

<button onclick="setText('#000')">Чёрный текст</button>
<button onclick="setText('#fff')">Белый текст</button>

<button onclick="setBg('pink')">Розовый фон</button>
<button onclick="setBg('blue')">Голубой фон</button>
<button onclick="setBg('green')">Зелёный фон</button>
<button onclick="setBg('dark')">Тёмный фон</button>

</div>

<script>

let clean=0
let coins=0
let tickets=3
let cases=0

let soapLevel=1
let spongeLevel=1

let level=0

let tool="sponge"
let soaped=false

const maxLevel=4
const baseCost=20
const ratio=2

let lastX=0
let lastY=0

const sponge=document.getElementById("sponge")
const progress=document.getElementById("progress")

function setTool(t){
tool=t
}

function foam(x,y){

let f=document.createElement("div")
f.className="bubble"
f.textContent="🫧"

f.style.left=x+"px"
f.style.top=y+"px"

document.getElementById("game").appendChild(f)

setTimeout(()=>f.remove(),1000)

}

function cost(level){
return baseCost*Math.pow(ratio,level-1)
}

function updatePrices(){

document.getElementById("soapCost").textContent=
soapLevel<maxLevel?cost(soapLevel):"MAX"

document.getElementById("spongeCost").textContent=
spongeLevel<maxLevel?cost(spongeLevel):"MAX"

}

function power(speed){

let base=0.05
base+=speed*0.002
base+=0.03*(soapLevel-1)
base+=0.03*(spongeLevel-1)

return base

}

function updateLevel(){

if(coins>=3000) level=6
else if(coins>=2000) level=5
else if(coins>=1000) level=4
else if(coins>=300) level=3
else if(coins>=200) level=2
else if(coins>=100) level=1
else level=0

}

function move(x,y){

let dx=x-lastX
let dy=y-lastY
let speed=Math.sqrt(dx*dx+dy*dy)

lastX=x
lastY=y

sponge.style.left=x+"px"
sponge.style.top=y+"px"

foam(x,y)

if(tool==="soap"){
soaped=true
document.getElementById("duck").textContent="🫧🦆🫧"
}

if(tool==="sponge" && soaped){

clean+=power(speed)

}

if(clean>100) clean=100

progress.style.width=clean+"%"

if(clean===100){

coins+=10

if(Math.random()<0.10) tickets++
if(Math.random()<0.30) coins+=200
if(Math.random()<0.07) cases++

soaped=false
document.getElementById("duck").textContent="🦆"

updateLevel()
updateUI()

clean=0

save()

}

}

function updateUI(){

document.getElementById("coins").textContent=coins
document.getElementById("tickets").textContent=tickets
document.getElementById("cases").textContent=cases
document.getElementById("level").textContent=level

}

document.addEventListener("touchmove",e=>{
const t=e.touches[0]
move(t.clientX,t.clientY)
e.preventDefault()
},{passive:false})

document.addEventListener("mousemove",e=>{
if(e.buttons===1) move(e.clientX,e.clientY)
})

function buySoap(){

if(soapLevel>=maxLevel) return

let price=cost(soapLevel)

if(coins>=price){
coins-=price
soapLevel++
}

document.getElementById("soapLv").textContent=soapLevel

updatePrices()
updateUI()
save()

}

function buySponge(){

if(spongeLevel>=maxLevel) return

let price=cost(spongeLevel)

if(coins>=price){
coins-=price
spongeLevel++
}

document.getElementById("spongeLv").textContent=spongeLevel

updatePrices()
updateUI()
save()

}

function spin(){

if(tickets<=0) return

tickets--

const items=["💰","🎟","❌"]

let a=items[Math.floor(Math.random()*items.length)]
let b=items[Math.floor(Math.random()*items.length)]
let c=items[Math.floor(Math.random()*items.length)]

document.getElementById("slot").textContent=a+" "+b+" "+c

if(a===b && b===c){

if(a==="💰") coins+=200
if(a==="🎟") tickets+=5

}

updateUI()
save()

}

function openCase(){

if(cases<=0) return

cases--

let r=Math.random()

if(r<0.5) coins+=100
else if(r<0.8) coins+=300
else tickets+=3

updateUI()
save()

}

function setText(c){
document.documentElement.style.setProperty('--text',c)
}

function setBg(type){

if(type==="pink"){
document.documentElement.style.setProperty('--bg1','#ffb3d1')
document.documentElement.style.setProperty('--bg2','#ffd9ec')
}

if(type==="blue"){
document.documentElement.style.setProperty('--bg1','#9edfff')
document.documentElement.style.setProperty('--bg2','#d9f3ff')
}

if(type==="green"){
document.documentElement.style.setProperty('--bg1','#a8ff93')
document.documentElement.style.setProperty('--bg2','#e3ffd9')
}

if(type==="dark"){
document.documentElement.style.setProperty('--bg1','#333')
document.documentElement.style.setProperty('--bg2','#111')
}

}

function save(){

localStorage.setItem("duckSave",JSON.stringify({
coins,
tickets,
cases,
soapLevel,
spongeLevel,
level
}))

}

function load(){

let s=localStorage.getItem("duckSave")

if(!s) return

let d=JSON.parse(s)

coins=d.coins
tickets=d.tickets
cases=d.cases
soapLevel=d.soapLevel
spongeLevel=d.spongeLevel
level=d.level

updateUI()

}

document.getElementById("shopBtn").onclick=()=>{
let p=document.getElementById("shop")
p.style.display=p.style.display==="block"?"none":"block"
}

document.getElementById("settingsBtn").onclick=()=>{
let p=document.getElementById("settings")
p.style.display=p.style.display==="block"?"none":"block"
}

updatePrices()
load()

</script>

</body>
</html>
