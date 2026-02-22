# Gestion-Assiduite-LMT
Suivi de l'assiduité des élèves/ IE KONDRO 

<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Registre Appel - Lycée Moderne de Treichville</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>

<style>
body{
font-family:Arial;
background:#f4f6f9;
padding:20px;
}
h1{
text-align:center;
color:#003366;
}
.card{
background:white;
padding:15px;
border-radius:8px;
margin-bottom:15px;
box-shadow:0 3px 8px rgba(0,0,0,0.1);
}
input{
width:100%;
padding:8px;
margin-top:5px;
margin-bottom:10px;
border:1px solid #ccc;
border-radius:5px;
}
button{
background:#003366;
color:white;
border:none;
padding:10px;
width:100%;
border-radius:5px;
cursor:pointer;
margin-top:5px;
}
button:hover{
background:#0055aa;
}
.absence-box{
background:#eef3ff;
padding:10px;
border-radius:6px;
margin-top:10px;
}
.counter{
font-weight:bold;
color:#cc0000;
}
</style>
</head>

<body>

<h1>📘 Gestion Assiduité Lycée Moderne de Treichville</h1>

<div class="card">
<label>Semaine :</label>
<input type="text" id="semaine" placeholder="Ex: du 26/02 au 03/03/2026">

<label>Classe :</label>
<input type="text" id="classe" placeholder="Ex: 6ème 1">
</div>

<div class="card">
<h3>Ajouter élève absent</h3>

<input type="text" id="nom" placeholder="Nom de l’élève">
<input type="text" id="matricule" placeholder="Matricule de l’élève">

<div class="absence-box">
<h4>Absences par tranche horaire</h4>
<div id="joursContainer"></div>
</div>

<button onclick="ajouterEleve()">Ajouter à la liste</button>
</div>

<div class="card">
<h3>Élèves absents cette semaine (<span class="counter" id="compteur">0</span>)</h3>
<ul id="liste"></ul>

<button onclick="genererPDF()">📄 Générer Rapport PDF</button>
</div>

<script>

const jours = ["Lundi","Mardi","Mercredi","Jeudi","Vendredi"];
const horaires = ["07h00-08h00","08h00-09h00","09h00-10h00","10h00-11h00",
"11h00-12h00","12h00-13h00","13h00-14h00",
"14h00-15h00","15h00-16h00","16h00-16h20"];

let eleves = [];

function initialiserJours(){
const container = document.getElementById("joursContainer");
container.innerHTML = "";
jours.forEach(jour=>{
let bloc = `<strong>${jour}</strong><br>`;
horaires.forEach(h=>{
bloc += `<label>
<input type="checkbox" value="${jour}"> ${h}
</label><br>`;
});
container.innerHTML += bloc + "<br>";
});
}

initialiserJours();
mettreAJourCompteur();

function ajouterEleve(){
const nom = document.getElementById("nom").value;
const matricule = document.getElementById("matricule").value;

if(!nom || !matricule) return alert("Remplir nom et matricule");

let absences = [];
document.querySelectorAll("input[type=checkbox]:checked").forEach(c=>{
absences.push(c.value);
});

eleves.push({nom, matricule, absences});
afficherListe();
resetForm();
}

function afficherListe(){
const ul = document.getElementById("liste");
ul.innerHTML = "";
eleves.forEach((e,index)=>{
ul.innerHTML += `<li>
<strong>${e.nom}</strong> - ${e.absences.length} heure(s) d'absence
<button onclick="supprimer(${index})">❌</button>
</li>`;
});
mettreAJourCompteur();
}

function supprimer(index){
eleves.splice(index,1);
afficherListe();
}

function resetForm(){
document.getElementById("nom").value="";
document.getElementById("matricule").value="";
document.querySelectorAll("input[type=checkbox]").forEach(c=>c.checked=false);
}

function mettreAJourCompteur(){
document.getElementById("compteur").innerText = eleves.length;
}

function genererPDF(){

if(eleves.length===0){
alert("Aucun élève enregistré");
return;
}

const { jsPDF } = window.jspdf;
const doc = new jsPDF();

doc.setFontSize(16);
doc.text("REGISTRE D’APPEL", 70, 10);
doc.setFontSize(12);
doc.text("Lycée Moderne de Treichville", 55, 18);
doc.text("Semaine : " + document.getElementById("semaine").value, 10, 26);
doc.text("Classe : " + document.getElementById("classe").value, 10, 32);

// Construire le tableau
const headers = ["N°","Nom","Matricule","Lundi","Mardi","Mercredi","Jeudi","Vendredi","Total absences"];
const data = eleves.map((e,i)=>{
let absParJour = jours.map(j=>{
return e.absences.filter(a=>a===j).length; // nb d’heures d’absence ce jour
});
let total = e.absences.length;
return [i+1, e.nom, e.matricule, ...absParJour, total];
});

doc.autoTable({
head:[headers],
body:data,
startY:40,
theme:'grid',
styles:{fontSize:10}
});

doc.save("rapport_absences_LMTR.pdf");
alert("PDF généré. Partagez-le dans le groupe WhatsApp des parents.");
}

</script>

</body>
</html>
