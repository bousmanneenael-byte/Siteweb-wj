<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Lord.com</title>
<style>
html,body{
  margin:0;
  height:100%;
  font-family:Arial;
  background:#111;
  color:white;
}
.hidden{display:none}
button{
  padding:6px 10px;
  margin:3px;
  cursor:pointer;
}
.panel{
  padding:20px;
  height:100%;
  box-sizing:border-box;
}
#menu{
  background:linear-gradient(blue,red);
  display:flex;
  flex-direction:column;
  align-items:center;
  justify-content:center;
}
#countrySelect{
  background:linear-gradient(red,blue);
}
.country{
  border:1px solid white;
  padding:10px;
  margin:8px;
}
.locked{opacity:0.4}
.countryPanel{
  border:1px solid #888;
  margin:10px;
  padding:10px;
}
</style>
</head>

<body>

<div id="menu" class="panel">
  <h1>Lord.com</h1>
  <input id="pseudo" placeholder="Pseudo">
  <button onclick="confirmPseudo()">Confirmer</button>
  <button onclick="start()">Jouer</button>
</div>

<div id="countrySelect" class="panel hidden">
  <h2>Choisis ton pays de départ</h2>
  <div id="countryList"></div>
</div>

<div id="game" class="panel hidden">
  <h2 id="playerName"></h2>
  <h3>💰 Argent : <span id="money"></span> (+<span id="moneyps"></span>/s)</h3>
  <p>⏱️ Changement de prix dans : <span id="timer">30</span>s</p>
  <div id="countries"></div>
</div>

<script>
/* ===== DONNÉES ===== */
let money = 100000000000;
let pseudo = "";
let timer = 30;

const countriesData = {
  Egypte:{type:"pharaon"},
  USA:{type:"president",special:"Nathan"},
  Russie:{type:"president",special:"Ethanael"},
  Belgique:{type:"roi",special:"Enaël"},
  Congo:{type:"president",special:"Nohlan"}
};

let state = {};

function rand(min,max){
  return Math.floor(Math.random()*(max-min+1))+min;
}

/* ===== INITIALISATION ===== */
for(let c in countriesData){
  state[c]={
    unlocked:false,
    people:0,
    population:0,
    leader:0,
    cats:0,
    gods:0,
    prices:{
      people:rand(10,75),
      leader:rand(1000,10000),
      cat:rand(10000,100000),
      god:rand(100000,1000000),
      special:rand(100000,1000000)
    }
  }
}

/* ===== MENUS ===== */
function confirmPseudo(){
  pseudo=document.getElementById("pseudo").value.trim();
  if(pseudo) alert("Pseudo confirmé");
}

function start(){
  if(!pseudo){alert("Entre un pseudo");return;}
  document.getElementById("menu").classList.add("hidden");
  document.getElementById("countrySelect").classList.remove("hidden");
  renderCountrySelect();
}

function renderCountrySelect(){
  const list=document.getElementById("countryList");
  list.innerHTML="";
  for(let c in countriesData){
    const b=document.createElement("button");
    b.textContent=c;
    b.onclick=()=>chooseCountry(c);
    list.appendChild(b);
  }
}

function chooseCountry(c){
  state[c].unlocked=true;
  document.getElementById("countrySelect").classList.add("hidden");
  document.getElementById("game").classList.remove("hidden");
  document.getElementById("playerName").textContent=pseudo;
  render();
}

/* ===== VILLES ===== */
function city(pop){
  if(pop==0) return {emoji:"",gain:0};
  if(pop<=10) return {emoji:"🛖",gain:1};
  if(pop<=25) return {emoji:"🏠",gain:2};
  if(pop<=49) return {emoji:"🏙️",gain:3};
  return {emoji:"🌆",gain:4};
}

/* ===== RENDER ===== */
function render(){
  document.getElementById("money").textContent=format(money);
  let mps=0;
  const cont=document.getElementById("countries");
  cont.innerHTML="";
  for(let c in state){
    const s=state[c];
    const cd=countriesData[c];
    const ct=city(s.population);
    mps+=ct.gain + s.leader*1 + s.gods*5 + (cd.special&&s.special?10:0);

    const d=document.createElement("div");
    d.className="country "+(s.unlocked?"":"locked");
    d.innerHTML=`
      <h3>${c}</h3>
      ${s.unlocked?`
      Population: ${s.population} ${ct.emoji}<br>
      <button onclick="buyPerson('${c}')">+ Personne (${s.prices.people})</button>
      <button onclick="sellPerson('${c}')">- Personne (${s.prices.people})</button><br>
      Dirigeant: ${s.leader}/1
      <button onclick="buyLeader('${c}')">Acheter (${s.prices.leader})</button>
      <button onclick="sellLeader('${c}')">Vendre</button><br>
      ${c=="Egypte"?`
      Chats: ${s.cats}/2
      <button onclick="buyCat()">Acheter (${s.prices.cat})</button>
      <button onclick="sellCat()">Vendre</button><br>
      Dieux: ${s.gods}
      <button onclick="buyGod()">Acheter (${s.prices.god})</button>
      <button onclick="sellGod()">Vendre</button>
      `:""}
      `:`<button onclick="buyCountry('${c}')">Acheter pays (1000)</button>`}
    `;
    cont.appendChild(d);
  }
  document.getElementById("moneyps").textContent=mps;
}

function format(n){
  if(n>=1e9) return (n/1e9).toFixed(1)+"B";
  if(n>=1e6) return (n/1e6).toFixed(1)+"M";
  if(n>=1e3) return (n/1e3).toFixed(1)+"K";
  return n;
}

/* ===== ACHATS ===== */
function buyPerson(c){
  const s=state[c];
  if(money<s.prices.people)return;
  money-=s.prices.people;
  s.people++; s.population++;
  render();
}
function sellPerson(c){
  const s=state[c];
  if(s.people<=0)return;
  s.people--; s.population--;
  money+=s.prices.people;
  render();
}
function buyLeader(c){
  const s=state[c];
  if(s.leader>=1||money<s.prices.leader)return;
  money-=s.prices.leader; s.leader=1; render();
}
function sellLeader(c){
  const s=state[c];
  if(s.leader<=0)return;
  s.leader=0; money+=s.prices.leader; render();
}
function buyCountry(c){
  if(money<1000)return;
  money-=1000;
  state[c].unlocked=true;
  render();
}

/* Égypte */
function buyCat(){
  const s=state["Egypte"];
  if(s.cats>=2||money<s.prices.cat)return;
  money-=s.prices.cat; s.cats++; render();
}
function sellCat(){
  const s=state["Egypte"];
  if(s.cats<=0)return;
  if(s.gods>0)s.gods--;
  s.cats--; money+=s.prices.cat; render();
}
function buyGod(){
  const s=state["Egypte"];
  if(s.cats<=s.gods||money<s.prices.god)return;
  money-=s.prices.god; s.gods++; render();
}
function sellGod(){
  const s=state["Egypte"];
  if(s.gods<=0)return;
  s.gods--; money+=s.prices.god; render();
}

/* ===== TIMER ===== */
setInterval(()=>{
  timer--;
  if(timer<=0){
    timer=30;
    for(let c in state){
      const p=state[c].prices;
      p.people=rand(10,75);
      p.leader=rand(1000,10000);
      p.cat=rand(10000,100000);
      p.god=rand(100000,1000000);
    }
  }
  document.getElementById("timer").textContent=timer;
},1000);

/* ===== ARGENT / SEC ===== */
setInterval(()=>{
  let gain=0;
  for(let c in state){
    const s=state[c];
    if(!s.unlocked)continue;
    const ct=city(s.population);
    gain+=ct.gain + s.leader + s.gods*5;
  }
  money+=gain;
  render();
},1000);

/* ===== SAUVEGARDE ===== */
setInterval(()=>{
  localStorage.setItem("lordSave",JSON.stringify({money,pseudo,state}));
},5000);
</script>
</body>
</html>
