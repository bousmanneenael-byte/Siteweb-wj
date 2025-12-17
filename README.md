<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Lord.com</title>
<style>
body{margin:0;font-family:Arial;background:#121212;color:white}
button{background:white;color:black;border:none;padding:6px 10px;margin:3px;cursor:pointer}
button:disabled{opacity:.4}
.panel{padding:15px}
.country{border:1px solid #555;padding:10px;margin:10px;border-radius:6px}
.moneyBar{background:black;padding:10px;position:sticky;top:0}
.hidden{display:none}
input,select{padding:5px}
</style>
</head>
<body>

<!-- MENU PRINCIPAL -->
<div id="menu" class="panel">
<h1>Lord.com</h1>
<input id="pseudo" placeholder="Pseudo">
<br><br>
<button onclick="openCountryMenu()">Jouer</button>
</div>

<!-- MENU CHOIX PAYS -->
<div id="countryMenu" class="panel hidden">
<h2>Choisis ton pays de départ</h2>
<select id="countrySelect"></select>
<br><br>
<button onclick="startGame()">Confirmer</button>
</div>

<!-- BARRE ARGENT -->
<div id="moneyBar" class="moneyBar hidden">
💰 Argent : <span id="money">0</span> |
💸 /sec : <span id="moneySec">0</span>
<br>
⏳ Changement des prix : <span id="timer">30</span>s
</div>

<!-- JEU -->
<div id="game" class="panel hidden"></div>

<script>
/* ===== VARIABLES ===== */
let money = 100000000000;
let moneyPerSec = 0;
let timer = 30;
let baseCountryPrice = 1000;

/* ===== PAYS ===== */
const countries = {
 "Égypte":"pharaon",
 "France":"president",
 "Belgique":"roi",
 "USA":"president",
 "Russie":"president"
};

const state = {};
for(let c in countries){
 state[c]={
  unlocked:false,
  people:0,
  leader:0,
  pharaon:0,
  chat:0,
  dieu:0,
  price:{
   people:r(10,75),
   leader:r(1000,10000),
   pharaon:r(1000,10000),
   chat:r(10000,100000),
   dieu:r(100000,1000000)
  }
 };
}

/* ===== UTILS ===== */
function r(a,b){return Math.floor(Math.random()*(b-a+1))+a}
function fmt(n){
 if(n>=1e9)return (n/1e9).toFixed(1)+"B";
 if(n>=1e6)return (n/1e6).toFixed(1)+"M";
 if(n>=1e3)return (n/1e3).toFixed(1)+"k";
 return n;
}

/* ===== MENUS ===== */
function openCountryMenu(){
 menu.classList.add("hidden");
 countryMenu.classList.remove("hidden");
 countrySelect.innerHTML="";
 for(let c in countries){
  countrySelect.innerHTML+=`<option>${c}</option>`;
 }
}

function startGame(){
 state[countrySelect.value].unlocked=true;
 countryMenu.classList.add("hidden");
 game.classList.remove("hidden");
 moneyBar.classList.remove("hidden");
 update();
 save();
}

/* ===== AFFICHAGE ===== */
function update(){
 document.getElementById("money").textContent=fmt(money);
 document.getElementById("moneySec").textContent=moneyPerSec;
 game.innerHTML="";
 moneyPerSec=0;

 for(let c in state){
  const s=state[c];
  let html=`<div class="country"><h2>${c} ${s.unlocked?"":"🔒"}</h2>`;

  if(!s.unlocked){
   html+=`<button onclick="buyCountry('${c}')">
   Acheter (${fmt(baseCountryPrice)})
   </button></div>`;
   game.innerHTML+=html;
   continue;
  }

  html+=`
  👤 Population : ${s.people} (${fmt(s.price.people)})
  <button onclick="buy('${c}','people')">+</button>
  <button onclick="sell('${c}','people')">-</button><br>
  `;

  if(countries[c]==="pharaon"){
   html+=`
   🐪 Pharaon : ${s.pharaon} (${fmt(s.price.pharaon)})
   <button onclick="buy('${c}','pharaon')">+</button>
   <button onclick="sell('${c}','pharaon')">-</button><br>

   🐱 Chats : ${s.chat}/2 (${fmt(s.price.chat)})
   <button onclick="buy('${c}','chat')">+</button>
   <button onclick="sell('${c}','chat')">-</button><br>

   ⚡ Dieux : ${s.dieu} (${fmt(s.price.dieu)})
   <button onclick="buy('${c}','dieu')">+</button>
   <button onclick="sell('${c}','dieu')">-</button><br>
   `;
  }else{
   html+=`
   👑 Dirigeant : ${s.leader} (${fmt(s.price.leader)})
   <button onclick="buy('${c}','leader')">+</button>
   <button onclick="sell('${c}','leader')">-</button><br>
   `;
  }

  html+="</div>";
  game.innerHTML+=html;

  moneyPerSec+=s.leader+s.pharaon+s.dieu*5;
 }
}

/* ===== ACHATS ===== */
function buyCountry(c){
 if(money<baseCountryPrice)return alert("Pas assez d'argent");
 money-=baseCountryPrice;
 state[c].unlocked=true;
 baseCountryPrice=Math.floor(baseCountryPrice*1.15);
 save();update();
}

function buy(c,t){
 const s=state[c],p=s.price[t];
 if(money<p)return;
 if(t==="chat" && s.chat>=2)return;
 if(t==="dieu" && s.chat<=s.dieu)return;

 money-=p;
 s[t]++;
 save();update();
}

function sell(c,t){
 const s=state[c],p=s.price[t];
 if(s[t]<=0)return;
 money+=p;
 s[t]--;
 if(t==="chat" && s.dieu>s.chat)s.dieu=s.chat;
 save();update();
}

/* ===== CHRONO ===== */
setInterval(()=>{
 timer--;
 if(timer<=0){
  for(let c in state){
   const p=state[c].price;
   p.people=r(10,75);
   p.leader=r(1000,10000);
   p.pharaon=r(1000,10000);
   p.chat=r(10000,100000);
   p.dieu=r(100000,1000000);
  }
  timer=30;
 }
 document.getElementById("timer").textContent=timer;
},1000);

/* ===== ARGENT PASSIF ===== */
setInterval(()=>{money+=moneyPerSec;update()},1000);

/* ===== SAVE ===== */
function save(){
 localStorage.setItem("lord_save",JSON.stringify({
  money,baseCountryPrice,state,timer
 }));
}
</script>
</body>
</html>
