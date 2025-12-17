<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Lord.com</title>
<style>
body {
  background:#111;
  color:#fff;
  font-family:Arial;
  margin:0;
}
button {
  background:#fff;
  color:#000;
  border:none;
  padding:6px 10px;
  margin:2px;
  cursor:pointer;
}
button:disabled {
  opacity:0.4;
  cursor:not-allowed;
}
.country {
  border:1px solid #444;
  padding:10px;
  margin:10px;
}
.hidden { display:none; }
.money {
  font-size:20px;
  padding:10px;
  background:#000;
}
</style>
</head>
<body>

<!-- MENU D'ENTRÉE -->
<div id="menu">
  <h1>Lord.com</h1>
  <input id="pseudo" placeholder="Ton pseudo">
  <br><br>
  <button onclick="goCountrySelect()">Confirmer</button>
</div>

<!-- CHOIX PAYS -->
<div id="selectCountry" class="hidden">
  <h2>Choisis ton pays</h2>
  <select id="startCountry"></select><br><br>
  <button onclick="startGame()">Jouer</button>
</div>

<div class="money hidden" id="moneyBox">
💰 Argent : <span id="money"></span> |
💸 /sec : <span id="moneySec">0</span>
</div>

<div id="countries" class="hidden"></div>

<script>
let money = 100000000000;
let moneyPerSec = 0;
let countryPrice = 1000;
let timer = 30;

const countries = {
 "Égypte": { unlocked:false, type:"pharaon" },
 "France": { unlocked:false, type:"president" },
 "USA": { unlocked:false, type:"president" },
 "Russie": { unlocked:false, type:"president" },
 "Belgique": { unlocked:false, type:"roi" }
};

const data = {};
for (let c in countries) {
 data[c] = {
  people:0,
  leader:0,
  pharaon:0,
  chats:0,
  dieux:0,
  prices: {
    people: rand(10,75),
    leader: rand(1000,10000),
    pharaon: rand(1000,10000),
    chat: rand(10000,100000),
    dieu: rand(100000,1000000)
  }
 };
}

function rand(a,b){return Math.floor(Math.random()*(b-a+1))+a;}

function goCountrySelect(){
 document.getElementById("menu").classList.add("hidden");
 document.getElementById("selectCountry").classList.remove("hidden");
 const sel=document.getElementById("startCountry");
 for(let c in countries){
  sel.innerHTML+=`<option>${c}</option>`;
 }
}

function startGame(){
 const c=document.getElementById("startCountry").value;
 countries[c].unlocked=true;
 document.getElementById("selectCountry").classList.add("hidden");
 document.getElementById("countries").classList.remove("hidden");
 document.getElementById("moneyBox").classList.remove("hidden");
 update();
}

function update(){
 document.getElementById("money").textContent = format(money);
 document.getElementById("moneySec").textContent = moneyPerSec;
 const cont=document.getElementById("countries");
 cont.innerHTML="";
 moneyPerSec=0;

 for(let c in countries){
  const d=data[c];
  const unlocked=countries[c].unlocked;
  let html=`<div class="country"><h2>${c} ${unlocked?"":"🔒"}</h2>`;

  if(!unlocked){
   html+=`<button onclick="buyCountry('${c}')">Acheter pays (${countryPrice})</button>`;
   html+=`</div>`;
   cont.innerHTML+=html;
   continue;
  }

  html+=`👤 Personnes : ${d.people}
  <button onclick="buy('${c}','people')">+</button>
  <button onclick="sell('${c}','people')">-</button><br>`;

  if(countries[c].type==="pharaon"){
   html+=`🐫 Pharaon : ${d.pharaon}
   <button onclick="buy('${c}','pharaon')">+</button>
   <button onclick="sell('${c}','pharaon')">-</button><br>
   🐱 Chats : ${d.chats}/2
   <button onclick="buy('${c}','chat')">+</button>
   <button onclick="sell('${c}','chat')">-</button><br>
   ⚡ Dieux : ${d.dieux}
   <button onclick="buy('${c}','dieu')">+</button>
   <button onclick="sell('${c}','dieu')">-</button><br>`;
  } else {
   html+=`👑 Dirigeant : ${d.leader}
   <button onclick="buy('${c}','leader')">+</button>
   <button onclick="sell('${c}','leader')">-</button><br>`;
  }

  html+=`</div>`;
  cont.innerHTML+=html;

  moneyPerSec += d.people*0 + d.leader*1 + d.pharaon*1 + d.dieux*5;
 }
}

function buyCountry(c){
 if(money<countryPrice) return alert("Pas assez d'argent");
 money-=countryPrice;
 countries[c].unlocked=true;
 countryPrice=Math.floor(countryPrice*1.15);
 update();
}

function buy(c,t){
 const p=data[c].prices[t];
 if(money<p) return alert("Pas assez d'argent");
 if(t==="chat" && data[c].chats>=2) return;
 if(t==="dieu" && data[c].chats<data[c].dieux+1)
   return alert("Veuillez d'abord acheter un chat");

 money-=p;
 data[c][t==="leader"?"leader":t+"s"]++;
 update();
}

function sell(c,t){
 const p=data[c].prices[t];
 const key=t==="leader"?"leader":t+"s";
 if(data[c][key]<=0) return;
 money+=p;
 data[c][key]--;
 if(t==="chat" && data[c].dieux>data[c].chats) data[c].dieux--;
 update();
}

function format(n){
 if(n>=1e9) return (n/1e9).toFixed(1)+"B";
 if(n>=1e6) return (n/1e6).toFixed(1)+"M";
 if(n>=1e3) return (n/1e3).toFixed(1)+"k";
 return n;
}

setInterval(()=>{
 for(let c in data){
  for(let p in data[c].prices){
   data[c].prices[p]=rand(
    p==="people"?10:p==="chat"?10000:1000,
    p==="people"?75:p==="chat"?100000:1000000
   );
  }
 }
 update();
},30000);

setInterval(()=>{
 money+=moneyPerSec;
 update();
},1000);

update();
</script>
</body>
</html>
