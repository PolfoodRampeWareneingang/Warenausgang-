<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Warenausgang Polfood GmbH</title>

<script src="https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js"></script>

<style>
body { font-family: Arial; background:#f4f4f4; padding:15px; margin:0; }
.box { background:white; padding:15px; max-width:900px; margin:auto; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1);}
h2 { text-align:center; margin-top:0; }
label { font-weight:bold; display:block; margin-top:10px; }

input {
  width:100%;
  padding:10px;
  margin:6px 0;
  font-size:16px;
  box-sizing:border-box;
}

button {
  width:100%;
  padding:12px;
  margin-top:10px;
  background:black;
  color:white;
  border:none;
  border-radius:6px;
}

table {
  width:100%;
  margin-top:20px;
  border-collapse:collapse;
  font-size:12px;
}

th, td {
  border:1px solid #ddd;
  padding:6px;
  text-align:center;
}

th { background:#eee; }

.dropdown { position:relative; }

.dropdown-list {
  position:absolute;
  width:100%;
  border:1px solid #ccc;
  max-height:220px;
  overflow:auto;
  background:white;
  display:none;
  z-index:1000;
}

.dropdown-item {
  padding:12px;
  cursor:pointer;
}

.dropdown-item:hover { background:#eee; }

.action-btn {
  background:#c62828;
  color:white;
  border:none;
  padding:6px 10px;
  border-radius:4px;
}

.photo-preview img {
  max-width:100%;
  max-height:160px;
  border-radius:8px;
}

.summary-box {
  margin-top:20px;
  background:#fafafa;
  border:1px solid #ddd;
  border-radius:8px;
  padding:10px;
}

.summary-grid {
  display:grid;
  grid-template-columns:repeat(2,1fr);
  gap:10px;
}
</style>
</head>

<body>

<div class="box">
<h2>🚛 Warenausgang Polfood GmbH</h2>

<label>Datum</label>
<input type="date" id="datum">

<label>Kunde</label>
<div class="dropdown">
  <input id="kunde" placeholder="Kunde wählen oder eingeben">
  <div id="kundenList" class="dropdown-list"></div>
</div>

<label>E2 OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" min="0" step="1" id="e2_out">

<label>H1 OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" min="0" step="1" id="h1_out">

<label>Einweg OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" min="0" step="1" id="einweg_out">

<label>EPAL OUT</label>
<input type="number" inputmode="numeric" pattern="[0-9]*" min="0" step="1" id="epal_out">

<label>Foto</label>
<input type="file" id="foto" accept="image/*" capture="environment">

<div class="photo-preview" id="previewBox" style="display:none;">
<img id="preview">
</div>

<button onclick="addEntry()">➕ Speichern</button>
<button onclick="exportExcel()">📦 Excel</button>
<button onclick="clearData()">🗑️ Löschen</button>

<div class="summary-box">
<div class="summary-grid">
<div>E2: <strong id="sum_e2_out">0</strong></div>
<div>H1: <strong id="sum_h1_out">0</strong></div>
<div>Einweg: <strong id="sum_einweg_out">0</strong></div>
<div>EPAL: <strong id="sum_epal_out">0</strong></div>
</div>
</div>

<table>
<thead>
<tr>
<th>Datum</th>
<th>Kunde</th>
<th>E2</th>
<th>H1</th>
<th>Einweg</th>
<th>EPAL</th>
<th>Foto</th>
<th>Aktion</th>
</tr>
</thead>
<tbody id="table"></tbody>
</table>
</div>

<script>
const standardKunden = [
"1 / Wach","2 / Fed","3 / Willi Hof","4 / Bremen EB","5 / Bremerhaven",
"6 / Bad Oldesloe EB","8 / Havelland","9 / Schmidt","10 / GT",
"11 / Dres","12 / Atl","13 / Freiburg","14 / Freiburg 2",
"18 / Föl","29 / Rostock","30 / Peter","32 / BAR","33 / Frisch",
"45 / Wolf","48 / Tor","51 / Käfer","52 / Hamb",
"53 / Ham Riem","54 / Ham Berlin","55 / Ham Frankfurt","56 / Fisch",
"57 / Wolf + Kunt","58 / FMS","59 / DUSP","66 / Mehl","70 / FEK",
"74 / Landpute","76 / Wunder","77 / Elst","80 / Mär",
"81 / Mig","82 / Wal","83 / Dim","84 / Landau","85 / Sandmann",
"87 / Richt","88 / See","89 / Zimmer","90 / MEGEM",
"91 / Bingen","92 / Weisen","93 / Enders","94 / Rot",
"95 / TLC","96 / NK","97 / Atl 2","98 / BLF",
"99 / Chickeria","Unna","Yu An","Futterhappen","Tosbiks","100 / Konrad"
];

let data = JSON.parse(localStorage.getItem("warenausgang_data") || "[]");
let kunden = JSON.parse(localStorage.getItem("kunden_liste") || "null") || [...standardKunden];
let currentPhoto = null;

const kundeInput = document.getElementById("kunde");
const list = document.getElementById("kundenList");
const datumInput = document.getElementById("datum");

function setHeute(){
  const d = new Date();
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth()+1).padStart(2,"0");
  const dd = String(d.getDate()).padStart(2,"0");
  datumInput.value = `${yyyy}-${mm}-${dd}`;
}
setHeute();

function save(){
  localStorage.setItem("warenausgang_data", JSON.stringify(data));
}

function saveKunden(){
  localStorage.setItem("kunden_liste", JSON.stringify(kunden));
}

function renderList(filter=""){
  list.innerHTML="";
  kunden
    .filter(k=>k.toLowerCase().includes(filter.toLowerCase()))
    .forEach(k=>{
      const div=document.createElement("div");
      div.textContent=k;
      div.className="dropdown-item";
      div.onclick=()=>{
        kundeInput.value=k;
        list.style.display="none";
      };
      list.appendChild(div);
    });
  list.style.display="block";
}

function addKunde(k){
  const val = k.trim();
  if(val && !kunden.some(x=>x.toLowerCase()===val.toLowerCase())){
    kunden.push(val);
    saveKunden();
  }
}

kundeInput.addEventListener("input",()=>renderList(kundeInput.value));
kundeInput.addEventListener("focus",()=>renderList(kundeInput.value));

document.addEventListener("click",(e)=>{
  if(!e.target.closest(".dropdown")) list.style.display="none";
});

document.getElementById("foto").addEventListener("change",(e)=>{
  const file=e.target.files[0];
  if(!file) return;

  currentPhoto=file;

  const reader=new FileReader();
  reader.onload=(ev)=>{
    document.getElementById("preview").src=ev.target.result;
    document.getElementById("previewBox").style.display="block";
  };
  reader.readAsDataURL(file);
});

const felder=["datum","kunde","e2_out","h1_out","einweg_out","epal_out"];

felder.forEach((id,i)=>{
  document.getElementById(id).addEventListener("keydown",(e)=>{
    if(e.key==="Enter"){
      e.preventDefault();

      if(id==="kunde"){
        addKunde(kundeInput.value);
      }

      if(felder[i+1]){
        document.getElementById(felder[i+1]).focus();
      } else {
        document.getElementById("foto").focus();
      }
    }
  });
});

function sanitizeFileName(name){
  return name
    .replace(/[\\/:*?"<>|]/g, "_")
    .replace(/\s+/g, " ")
    .trim();
}

function getFileExtension(file){
  if(!file) return "jpg";

  const parts = file.name.split(".");
  if(parts.length > 1){
    return parts.pop().toLowerCase();
  }

  if(file.type === "image/png") return "png";
  if(file.type === "image/webp") return "webp";
  if(file.type === "image/heic") return "heic";
  return "jpg";
}

function buildPhotoName(datum,kunde,file){
  const cleanDate = sanitizeFileName(datum);
  const cleanKunde = sanitizeFileName(kunde.replace(/\n/g," "));
  const ext = getFileExtension(file);
  return `${cleanDate} - ${cleanKunde}.${ext}`;
}

function downloadPhoto(file,name){
  const url = URL.createObjectURL(file);
  const a = document.createElement("a");
  a.href = url;
  a.download = name;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);

  setTimeout(()=>{
    URL.revokeObjectURL(url);
  },1000);
}

function addEntry(){
  const datum = datumInput.value;
  const kunde = kundeInput.value.trim();

  if(!datum){
    alert("Datum fehlt.");
    return;
  }

  if(!kunde){
    alert("Kunde fehlt.");
    return;
  }

  addKunde(kunde);

  let fotoName = "";

  if(currentPhoto){
    fotoName = buildPhotoName(datum,kunde,currentPhoto);
    downloadPhoto(currentPhoto,fotoName);
  }

  data.push({
    datum,
    kunde,
    e2_out:+document.getElementById("e2_out").value||0,
    h1_out:+document.getElementById("h1_out").value||0,
    einweg_out:+document.getElementById("einweg_out").value||0,
    epal_out:+document.getElementById("epal_out").value||0,
    foto:fotoName
  });

  save();
  render();
  sum();

  document.getElementById("e2_out").value="";
  document.getElementById("h1_out").value="";
  document.getElementById("einweg_out").value="";
  document.getElementById("epal_out").value="";
  document.getElementById("foto").value="";

  currentPhoto=null;
  document.getElementById("previewBox").style.display="none";
  document.getElementById("preview").src="";

  setHeute();
  kundeInput.focus();
}

function render(){
  const t=document.getElementById("table");
  t.innerHTML="";

  data.forEach((r,i)=>{
    t.innerHTML+=`
<tr>
<td>${r.datum}</td>
<td>${r.kunde}</td>
<td>${r.e2_out}</td>
<td>${r.h1_out}</td>
<td>${r.einweg_out}</td>
<td>${r.epal_out}</td>
<td>${r.foto||""}</td>
<td><button class="action-btn" onclick="del(${i})">X</button></td>
</tr>`;
  });
}

function del(i){
  if(confirm("Eintrag löschen?")){
    data.splice(i,1);
    save();
    render();
    sum();
  }
}

function sum(){
  document.getElementById("sum_e2_out").textContent=data.reduce((a,b)=>a+Number(b.e2_out||0),0);
  document.getElementById("sum_h1_out").textContent=data.reduce((a,b)=>a+Number(b.h1_out||0),0);
  document.getElementById("sum_einweg_out").textContent=data.reduce((a,b)=>a+Number(b.einweg_out||0),0);
  document.getElementById("sum_epal_out").textContent=data.reduce((a,b)=>a+Number(b.epal_out||0),0);
}

function exportExcel(){
  if(!data.length){
    alert("Keine Daten.");
    return;
  }

  const ws=XLSX.utils.json_to_sheet(data);
  const wb=XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb,ws,"Warenausgang");

  const today = datumInput.value || "report";
  XLSX.writeFile(wb,`Warenausgang_${today}.xlsx`);
}

function clearData(){
  if(confirm("Alles löschen?")){
    data=[];
    save();
    render();
    sum();
  }
}

render();
sum();
</script>

</body>
</html>
