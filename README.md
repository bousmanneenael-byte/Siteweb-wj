<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Lord.com - Multi-pays</title>
<style>
body{font-family:Arial;background:#111;color:white;padding:20px;}
button{padding:6px 10px;margin:4px;cursor:pointer;}
.hidden{display:none;}
.panel{border:1px solid #555;padding:10px;margin-top:10px;}
.countryBox{border:2px solid #777;padding:10px;margin:10px 0;position:relative;}
.countryBox.locked{opacity:0.45;}
.buyCountry{background:white;color:black;font-weight:bold;border:2px solid black;}
.error{color:#ff5555;}
input{padding:5px;margin:5px;width:200px;}
</style>
</head>
<body>
<h1>🌍 Lord.com - Multi-pays</h1>
<div>
💰 Argent : <span id="money"></span> |
⏱️ Changement prix dans : <span id="timer"></span>s |
💸 Total / sec : <span id="totalPS"></span>
</div>
<hr>

<div id="startMenu">
<h2>Choisis ton pays de départ</h2>
<div id="startButtons"></div>
</div>

<input type="text" id="searchInput" placeholder="Rechercher un pays..." onkeyup="searchCountry()">
<div id="countriesList"></div>

<script>
let money=20;  // Argent de départ modifié
let timer=30;
let countryData={}, unlocked={}, prices={}, population={}, persons={}, special={}, leaders={};

// 50 pays
let countryNames = ["Égypte","Russie","Belgique","USA","Congo","France","Allemagne","Italie","Espagne","Portugal",
"Norvège","Suède","Finlande","Danemark","Islande","Japon","Chine","Inde","Brésil","Argentine",
"Mexique","Canada","Australie","Nouvelle-Zélande","Afrique du Sud","Kenya","Maroc","Tunisie","Algérie","Nigeria",
"France2","Allemagne2","Italie2","Espagne2","Portugal2","Norvège2","Suède2","Finlande2","Danemark2","Islande2",
"Japon2","Chine2","Inde2","Brésil2","Argentine2","Mexique2","Canada2","Australie2","Nouvelle-Zélande2","Afrique du Sud2"];

// Dirigeant réel
let leaderType = {
  c1:"president", c2:"king", c3:"president", c4:"president",
  c5:"president", c6:"president", c7:"president", c8:"president", c9:"president", c10:"president"
};

// Création des pays
countryNames.forEach((name,i)=>{
 let id="c"+i;
 countryData[id]={name:name};
 unlocked[id]=false;
 population[id]=0;
 persons[id]=0;
 special[id]={pharaoh:0,cat:0,god:0,ethanael:0,enael:0,nathan:0,nohlan:0};
 leaders[id]=0;
 prices[id]={person:0,pharaoh:0,cat:0,god:0,ethanael:0,enael:0,nathan:0,nohlan:0,president:0,king:0};
});

// Prix aléatoire
function r(a,b){return Math.floor(Math.random()*(b-a+1))+a;}
function newPrices(){
 Object.keys(countryData).forEach(id=>{
   prices[id].person=r(10,75);
   prices[id].pharaoh=r(1000,10000);
   prices[id].cat=r(10000,100000);
   prices[id].god=r(100000,1000000);
   prices[id].ethanael=r(1000000,10000000);
   prices[id].enael=r(1000000,10000000);
   prices[id].nathan=r(1000000,10000000);
   prices[id].nohlan=r(1000000,10000000);
   if(id!="c0" && leaderType[id]) prices[id][leaderType[id]] = r(1000,10000);
 });
}

// Menu départ
let startHTML="";
Object.keys(countryData).forEach(id=>{
 startHTML+=`<button onclick="start('${id}')">${countryData[id].name}</button>`;
});
document.getElementById("startButtons").innerHTML=startHTML;

// Ouvrir / fermer interface
function toggle(id){
  if(!unlocked[id]){
    alert("Vous devez d'abord acheter ce pays !");
    return;
  }
  document.getElementById(id).classList.toggle("hidden");
}

// Début avec pays choisi
function start(id){
 document.getElementById("startMenu").remove();
 searchCountry();
 unlock(id,true);
}

// Débloquer un pays
function unlock(id,free){
 unlocked[id]=true;
 let box=document.getElementById(id+"BoxSearch");
 if(box) box.classList.remove("locked");
 let btn=box.querySelector(".buyCountry");
 if(btn) btn.remove();
}

// Achat pays
function buyCountry(id){
 let box=document.getElementById(id+"BoxSearch");
 let btn=box.querySelector(".buyCountry");
 if(!btn) return;
 let price=parseInt(btn.textContent.match(/\d+/)[0]);
 if(money<price){alert("Pas assez d'argent !");return;}
 money-=price;
 unlock(id,false);
 Object.keys(countryData).forEach(cid=>{
   if(cid!=id){
     let b=document.getElementById(cid+"BoxSearch")?.querySelector(".buyCountry");
     if(b)b.textContent=`Acheter le pays (${Math.floor(parseInt(b.textContent.match(/\d+/)[0]*1.15))})`;
   }
 });
 update();
}

// Achat / Vente personnes
function buyPerson(c){
 if(money>=prices[c].person){money-=prices[c].person;persons[c]++;}else{alert("Pas assez d'argent !");}
 update();
}
function sellPerson(c){
 if(persons[c]>0){money+=prices[c].person;persons[c]--;}else{alert("Aucune personne !");}
 update();
}

// Achat / Vente spéciaux (Égypte)
function buySpecial(c,type){
 let max=1; if(type=="cat") max=2;
 if(type=="god" && special[c]["cat"]==0){alert("Veuillez d'abord acheter un chat !"); return;}
 if(special[c][type]>=max){alert("Max atteint pour "+type); return;}
 if(money>=prices[c][type]){money-=prices[c][type]; special[c][type]++;}else{alert("Pas assez d'argent !");}
 update();
}
function sellSpecial(c,type){
 if(special[c][type]>0){
   money+=prices[c][type];
   special[c][type]--;
   if(type=="cat" && special[c]["god"]>0){special[c]["god"]=0; alert("Le dieu lié au chat vendu !");}
 }
 update();
}

// Achat / Vente dirigeants
function buyLeader(c){
 let type = leaderType[c];
 if(!type) return;
 if(leaders[c]>=1){alert("Vous avez déjà un "+type+" !"); return;}
 if(money>=prices[c][type]){money-=prices[c][type]; leaders[c]=1;}else{alert("Pas assez d'argent !");}
 update();
}
function sellLeader(c){
 let type = leaderType[c];
 if(!type) return;
 if(leaders[c]>0){money+=prices[c][type]; leaders[c]=0;}
 update();
}

// Affichage / mise à jour
function update(){
 document.getElementById("money").textContent=money;
 document.getElementById("timer").textContent=timer;
 let total = 0;

 Object.keys(countryData).forEach(id=>{
   let pop = population[id] + persons[id];
   let cityType="", cityEmoji="", cityPS=0;
   if(pop>=1 && pop<=10){ cityType="Village"; cityEmoji="🏡"; cityPS=1; }
   else if(pop>=11 && pop<=25){ cityType="Petite Ville"; cityEmoji="🏘️"; cityPS=2; }
   else if(pop>=26 && pop<=49){ cityType="Ville"; cityEmoji="🏙️"; cityPS=3; }
   else if(pop>=50){ cityType="Métropole"; cityEmoji="🌆"; cityPS=4; }
   if(pop>=100) cityPS = 5;
   if(pop>=250) cityPS = 6 + Math.floor((pop-250)/250);

   let inc = cityPS + leaders[id] + special[id].pharaoh + special[id].ethanael*10 + special[id].enael*10 + special[id].nathan*10 + special[id].nohlan*10 + special[id].god*5;
   total += inc;

   let panel=document.getElementById(id);
   if(panel){
     document.getElementById("pop"+id).textContent = `${cityEmoji} ${pop} (${cityType})`;
     document.getElementById("ps"+id).textContent = inc;
     document.getElementById("pPrice"+id).textContent = prices[id].person;
     // Dirigeants
     let leaderHTML="";
     let type=leaderType[id];
     if(type) leaderHTML=`<button onclick="buyLeader('${id}')">Acheter ${type} (${prices[id][type]})</button>
     <button onclick="sellLeader('${id}')">Vendre ${type}</button>`;
     document.getElementById("leader"+id).innerHTML = leaderHTML;
     // Spéciaux
     let spHTML="";
     if(id=="c0"){
       spHTML+=`<br>Pharaon: ${special[id].pharaoh}/1 <button onclick="buySpecial('${id}','pharaoh')">Acheter (${prices[id].pharaoh})</button>`;
       spHTML+=`<br>Chats: ${special[id].cat}/2 <button onclick="buySpecial('${id}','cat')">Acheter (${prices[id].cat})</button>`;
       spHTML+=`<br>Dieux: ${special[id].god}/2 <button onclick="buySpecial('${id}','god')">Acheter (${prices[id].god})</button>`;
     }
     if(id=="c1") spHTML+=`<br>Ethanael: ${special[id].ethanael}/1 <button onclick="buySpecial('${id}','ethanael')">Acheter (${prices[id].ethanael})</button>`;
     if(id=="c2") spHTML+=`<br>Enaël: ${special[id].enael}/1 <button onclick="buySpecial('${id}','enael')">Acheter (${prices[id].enael})</button>`;
     if(id=="c3") spHTML+=`<br>Nathan: ${special[id].nathan}/1 <button onclick="buySpecial('${id}','nathan')">Acheter (${prices[id].nathan})</button>`;
     if(id=="c4") spHTML+=`<br>Nohlan: ${special[id].nohlan}/1 <button onclick="buySpecial('${id}','nohlan')">Acheter (${prices[id].nohlan})</button>`;
     document.getElementById("special"+id).innerHTML = spHTML;
   }
 });
 document.getElementById("totalPS").textContent = total;
}

// Affichage initial
newPrices();
update();

// Chrono prix
setInterval(()=>{
 timer--; 
 if(timer<=0){timer=30; newPrices();}
 update();
},1000);

// Gain par seconde
setInterval(()=>{
 Object.keys(countryData).forEach(id=>{
   let pop = population[id] + persons[id];
   let cityPS=0;
   if(pop>=1 && pop<=10) cityPS=1;
   else if(pop>=11 && pop<=25) cityPS=2;
   else if(pop>=26 && pop<=49) cityPS=3;
   else if(pop>=50) cityPS=4;
   if(pop>=100) cityPS = 5;
   if(pop>=250) cityPS = 6 + Math.floor((pop-250)/250);

   money += cityPS + leaders[id] + special[id].pharaoh + special[id].ethanael*10 + special[id].enael*10 + special[id].nathan*10 + special[id].nohlan*10 + special[id].god*5;
 });
 update();
},1000);

// Barre de recherche
function searchCountry() {
  let input = document.getElementById("searchInput").value.toLowerCase();
  let countriesDiv = document.getElementById("countriesList");
  countriesDiv.innerHTML = "";
  Object.keys(countryData).forEach(id => {
    let name = countryData[id].name.toLowerCase();
    if(name.includes(input)){
      let unlockedClass = unlocked[id] ? "" : "locked";
      countriesDiv.innerHTML += `
      <div class="countryBox ${unlockedClass}" id="${id}BoxSearch">
        <h2>${countryData[id].name}</h2>
        ${!unlocked[id] ? `<button class="buyCountry" onclick="buyCountry('${id}')">Acheter le pays (1000)</button>` : ""}
        <button onclick="toggle('${id}')">Ouvrir / Fermer</button>
        <div id="${id}" class="panel hidden">
          Population : <span id="pop${id}">0</span><br>
          💰 Argent/sec : <span id="ps${id}">0</span><br>
          <button onclick="buyPerson('${id}')">+ Personne (<span id="pPrice${id}">0</span>)</button>
          <button onclick="sellPerson('${id}')">- Personne</button>
          <div id="leader${id}"></div>
          <div id="special${id}"></div>
        </div>
      </div>`;
    }
  });
}
</script>
</body>
</html>
