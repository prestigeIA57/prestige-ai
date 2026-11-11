<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PRESTIGE AI</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#080808;color:white;
 font-family:Arial,sans-serif;text-align:center;
}
header{padding:30px;color:#d6b45b;letter-spacing:5px}
h1{font-size:42px;margin:25px}
.gold{color:#d6b45b}
.box{
 max-width:600px;margin:30px auto;padding:25px;
 border:1px solid #b58a32;border-radius:18px;
 background:#141414;
}
textarea{
 width:100%;height:130px;padding:15px;
 background:#080808;color:white;
 border:1px solid #b58a32;border-radius:10px;
 font-size:16px;
}
select{
 width:100%;padding:13px;margin-top:15px;
 background:#080808;color:white;
 border:1px solid #b58a32;border-radius:8px;
}
button{
 margin-top:20px;padding:16px 25px;
 background:#d6b45b;color:black;
 border:0;border-radius:10px;
 font-weight:bold;font-size:16px;
}
button:disabled{opacity:.5}
#message{color:#d6b45b;margin-top:20px}
#resultat{
 display:none;width:100%;margin-top:20px;
 border:1px solid #b58a32;border-radius:10px;
}
</style>
</head>
<body>

<header>PRESTIGE STUDIO</header>

<h1>PRESTIGE <span class="gold">AI</span></h1>

<p>Ton studio de création vidéo par intelligence artificielle.</p>

<div class="box">

<h2>🎬 Créer une vidéo</h2>

<textarea id="prompt" placeholder="Décris la vidéo que tu souhaites créer..."></textarea>

<select id="style">
<option>Réaliste</option>
<option>Cinématique</option>
<option>Animation</option>
<option>Futuriste</option>
</select>

<button id="bouton" onclick="generer()">GÉNÉRER MA VIDÉO</button>

<div id="message"></div>

<video id="resultat" controls autoplay loop playsinline></video>

</div>

<script>
// 👉 Remplace par l'adresse de ton Worker Cloudflare
const WORKER = "https://REMPLACE-MOI.workers.dev";

async function generer(){
 const texte = document.getElementById("prompt").value;
 const style = document.getElementById("style").value;
 const message = document.getElementById("message");
 const video = document.getElementById("resultat");
 const bouton = document.getElementById("bouton");

 if(texte.trim().length < 5){
  message.innerText = "Décris ta vidéo avant de continuer.";
  return;
 }

 if(WORKER.includes("REMPLACE-MOI")){
  message.innerText = "Le moteur vidéo IA doit encore être connecté.";
  return;
 }

 bouton.disabled = true;
 video.style.display = "none";
 message.innerText = "Génération en cours… cela peut prendre quelques minutes.";

 try{
  const res = await fetch(WORKER, {
   method: "POST",
   headers: {"Content-Type": "application/json"},
   body: JSON.stringify({prompt: texte, style: style})
  });
  let job = await res.json();

  if(!job.id){
   throw new Error("Réponse inattendue du serveur");
  }

  while(job.status !== "succeeded" && job.status !== "failed" && job.status !== "canceled"){
   await new Promise(r => setTimeout(r, 3000));
   job = await (await fetch(WORKER + "?id=" + job.id)).json();
  }

  if(job.status === "succeeded"){
   const url = Array.isArray(job.output) ? job.output[0] : job.output;
   video.src = url;
   video.style.display = "block";
   message.innerText = "Ta vidéo est prête !";
  }else{
   message.innerText = "La génération a échoué. Essaie une autre description.";
  }
 }catch(e){
  message.innerText = "Erreur : impossible de contacter le moteur vidéo.";
 }

 bouton.disabled = false;
}
</script>

</body>
</html>
