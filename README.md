<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, <!DOCTYPE html>
<html>
<head>
  <title>IA Apprentissage</title>
  <style>
    body { font-family: Arial; text-align: center; }
    #chat { max-width: 500px; margin: auto; }
    .box { margin: 10px; }
  </style>
</head>
<body>

<h2>IA 🧠 Apprentissage</h2>

<div class="box">
  <input id="key" placeholder="Mot clé (ex: chat)">
</div>

<div class="box">
  <input id="value" placeholder="Phrase à apprendre">
</div>

<button onclick="learn()">Apprendre</button>
<button onclick="ask()">Demander</button>

<p id="answer"></p>

<script>

let memory = JSON.parse(localStorage.getItem("ia_learn")) || {};

// 🧠 apprendre
function learn(){

  let key = document.getElementById("key").value.toLowerCase().trim();
  let value = document.getElementById("value").value.trim();

  if(!key || !value) return;

  if(!memory[key]){
    memory[key] = [];
  }

  memory[key].push(value);

  localStorage.setItem("ia_learn", JSON.stringify(memory));

  alert("Appris 🧠 !");
}

// 🔍 répondre
function ask(){

  let key = document.getElementById("key").value.toLowerCase().trim();

  if(memory[key] && memory[key].length > 0){

    let list = memory[key];

    let randomIndex = Math.floor(Math.random() * list.length);

    document.getElementById("answer").innerText = list[randomIndex];

  } else {
    document.getElementById("answer").innerText = "Je ne connais pas ce mot 😅";
  }
}

</script>

</body>
</html>
