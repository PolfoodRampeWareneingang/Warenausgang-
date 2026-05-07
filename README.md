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
input { width:100%; padding:10px; margin:6px 0; font-size:16px; box-sizing:border-box; }
button { width:100%; padding:12px; margin-top:10px; background:black; color:white; border:none; border-radius:6px; }
table { width:100%; margin-top:20px; border-collapse:collapse; font-size:12px; }
th, td { border:1px solid #ddd; padding:6px; text-align:center; }
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
.dropdown-item { padding:12px; cursor:pointer; }
.dropdown-item:hover { background:#eee; }

.action-btn {
  background:#c62828;
  color:white;
  border:none;
  padding:6px 10px;
  border-radius:4px;
  width:auto;
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

.ok { color:green; font-weight:bold; }
.warn { color:red; font-weight:bold; }
.open { color:#b36b00; font-weight:bold; }
</style>
</head>

<body>

<div class="box">
<h2>🚛 Warenausgang Polfood GmbH</h2>

<label>AVIS Excel importieren</label>
<input type="file" id="avisFile" accept=".xlsx,.xls">

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
<div>E2 gesamt: <strong id="sum_e2_out">0</strong></div>
<div>H1 gesamt: <strong id="sum_h1_out">0</strong></div>
<div>Einweg gesamt: <strong id="sum_einweg_out">0</strong></div>
<div>EPAL gesamt: <strong id="sum_epal_out">0</strong></div>
</div>
</div>

<table>
<thead>
<tr>
<th>Kunde</th>
<th>AVIS E2</th>
<th>E2 Zusammensetzung</th>
<th>E2 Gesamt</th>
<th>Differenz</th>
<th>Status</th>
<th>H1 Gesamt</th>
<th>Einweg Gesamt</th>
<th>EPAL Gesamt</th>
<th>Fotos</th>
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

const kundenMapping = {
  "1 / Wach": "Wache",
  "2 / Fed": "Feddersen",
  "3 / Willi Hof": "Willi Hofner",
  "4 / Bremen EB": "Bremen",
  "5 / Bremerhaven": "Bremerhaven",
  "6 / Bad Oldesloe EB": "Bad Oldesloe",
  "8 / Havelland": "Havelland",
  "9 / Schmidt": "Schmidt und Sohn",
  "10 / GT": "GT Emporium",
  "11 / Dres": "Metzgerei Dressel",
  "12 / Atl": "Atlas CC Köln",
  "13 / Freiburg": "Freiburger Frischwaren",
  "14 / Freiburg 2": "Freiburger Frischwaren",
  "18 / Föl": "Fölster",
  "29 / Rostock": "Rostock",
  "30 / Peter": "Petersen",
  "32 / BAR": "BARLU",
  "33 / Frisch": "Frisch Frost",
  "45 / Wolf": "Wolf 2 Annaberg",
  "48 / Tor": "Torney",
  "51 / Käfer": "Käferstein",
  "52 / Hamb": "Hamberger Friedenstraße",
  "53 / Ham Riem": "Hamb. Riem",
  "54 / Ham Berlin": "Hamberger Berlin",
  "55 / Ham Frankfurt": "Hamberger Frankfurt",
  "56 / Fisch": "Fischer Gmbh",
  "57 / Wolf + Kunt": "Wolf + Kunt.",
  "58 / FMS": "FMS",
  "59 / DUSP": "DUSP",
  "66 / Mehl": "Karl Mehl",
  "70 / FEK": "FEK H/G",
  "74 / Landpute": "Landpute",
  "76 / Wunder": "Wunderland",
  "77 / Elst": "Stefan Elst",
  "80 / Mär": "März",
  "81 / Mig": "MigroMa",
  "82 / Wal": "M. Walk",
  "83 / Dim": "Dimter",
  "84 / Landau": "C+C Landau",
  "85 / Sandmann": "Meemken Sandmann",
  "87 / Richt": "Richter",
  "88 / See": "Deu. See",
  "89 / Zimmer": "Zimmermann",
  "90 / MEGEM": "MEGEM",
  "91 / Bingen": "C&C Bingen",
  "92 / Weisen": "Weisenhorn",
  "93 / Enders": "Enders",
  "94 / Rot": "Rothe",
  "95 / TLC": "TLC",
  "96 / NK": "NK",
  "97 / Atl 2": "Atlas",
  "98 / BLF": "BLF",
  "99 / Chickeria": "Chickeria",
  "Unna": "Unna",
  "Yu An": "Yu An",
  "Futterhappen": "Futterhappen",
  "Tosbiks": "Tosbiks",
  "100 / Konrad": "Konrad Böhnlein"
};

let data = JSON.parse(localStorage.getItem("warenausgang_data") || "[]");
let kunden = JSON.parse(localStorage.getItem("kunden_liste") || "null") || [...standardKunden];
let avisData = {};
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

function normalizeText(value){
  return String(value || "")
    .toLowerCase()
    .replace(/\n/g," ")
    .replace(/ä/g,"ae")
    .replace(/ö/g,"oe")
    .replace(/ü/g,"ue")
    .replace(/ß/g,"ss")
    .replace(/[^a-z0-9]/g,"")
    .trim();
}

function getAvisKeyForAppKunde(appKunde){
  const mapped = kundenMapping[appKunde] || appKunde;
  const normalizedMapped = normalizeText(mapped);

  if(avisData[normalizedMapped] !== undefined){
    return normalizedMapped;
  }

  const numberMatch = appKunde.match(/^(\d+)/);
  if(numberMatch){
    const num = numberMatch[1];
    for(const key in avisData){
      if(key.startsWith(num)){
        return key;
      }
    }
  }

  for(const key in avisData){
    if(key.includes(normalizedMapped) || normalizedMapped.includes(key)){
      return key;
    }

    const relevantParts = normalizedMapped
      .split(/[^a-z0-9]+/)
      .map(x => normalizeText(x))
      .filter(x => x.length >= 3);

    for(const part of relevantParts){
      if(key.includes(part)){
        return key;
      }
    }
  }

  return normalizedMapped;
}

document.getElementById("avisFile").addEventListener("change", function(e){
  const file = e.target.files[0];
  if(!file) return;

  const reader = new FileReader();

  reader.onload = function(event){
    const dataArray = new Uint8Array(event.target.result);
    const workbook = XLSX.read(dataArray, { type: "array" });
    const sheetName = workbook.SheetNames[0];
    const sheet = workbook.Sheets[sheetName];
    const rows = XLSX.utils.sheet_to_json(sheet, { header: 1 });

    avisData = {};

    rows.forEach(row => {
      const kunde = row[0];
      const avisE2 = row[1];

      if(kunde && avisE2 !== undefined && avisE2 !== null && avisE2 !== ""){
        avisData[normalizeText(kunde)] = Number(avisE2);
      }
    });

    alert("AVIS-Datei wurde geladen.");
    render();
  };

  reader.readAsArrayBuffer(file);
});

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

function getCustomerAvis(kunde){
  const avisKey = getAvisKeyForAppKunde(kunde);
  return avisData[avisKey] ?? null;
}

function getCustomerSummary(kunde, extraE2 = 0){
  const entries = data.filter(r => r.kunde === kunde);
  const e2Parts = entries.map(r => Number(r.e2_out || 0));

  if(extraE2 !== null && extraE2 !== undefined){
    e2Parts.push(Number(extraE2 || 0));
  }

  const e2Total = e2Parts.reduce((a,b)=>a+b,0);
  const avisE2 = getCustomerAvis(kunde);

  let status = "OK";
  let differenz = "";
  let abweichung = "";

  if(avisE2 === null){
    status = "KEIN AVIS";
    differenz = "";
    abweichung = "Kein AVIS-Eintrag gefunden";
  } else {
    differenz = e2Total - avisE2;

    if(e2Total < avisE2){
      status = "OFFEN";
      abweichung = `Kunde: ${kunde}\nAVIS E2: ${avisE2}\nGesamt E2 OUT: ${e2Total}\nNoch offen: ${avisE2 - e2Total}`;
    } else if(e2Total === avisE2){
      status = "OK";
      abweichung = `Kunde: ${kunde}\nAVIS E2 vollständig erreicht: ${avisE2}`;
    } else {
      status = "ZU VIEL";
      abweichung = `Kunde: ${kunde}\nAVIS E2: ${avisE2}\nGesamt E2 OUT: ${e2Total}\nZu viel: ${e2Total - avisE2}`;
    }
  }

  return {
    avisE2,
    e2Parts,
    e2Total,
    differenz,
    status,
    abweichung
  };
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

  const e2Out = Number(document.getElementById("e2_out").value || 0);
  const previewSummary = getCustomerSummary(kunde, e2Out);

  if(previewSummary.status === "ZU VIEL"){
    const weiter = confirm(
      "Achtung! Für diesen Kunden wurde mehr E2 OUT erfasst als avisiert:\n\n" +
      previewSummary.abweichung +
      "\n\nTrotzdem speichern?"
    );

    if(!weiter){
      return;
    }
  }

  let fotoName = "";

  if(currentPhoto){
    fotoName = buildPhotoName(datum,kunde,currentPhoto);
    downloadPhoto(currentPhoto,fotoName);
  }

  data.push({
    datum,
    kunde,
    avis_e2: previewSummary.avisE2,
    e2_out: e2Out,
    h1_out:+document.getElementById("h1_out").value||0,
    einweg_out:+document.getElementById("einweg_out").value||0,
    epal_out:+document.getElementById("epal_out").value||0,
    status: previewSummary.status,
    differenz: previewSummary.differenz,
    abweichung: previewSummary.abweichung,
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

function getGroupedData(){
  const groups = {};

  data.forEach((r, index)=>{
    const kunde = r.kunde || "Ohne Kunde";

    if(!groups[kunde]){
      groups[kunde] = {
        kunde,
        avisE2: getCustomerAvis(kunde),
        e2Parts: [],
        e2Total: 0,
        h1Total: 0,
        einwegTotal: 0,
        epalTotal: 0,
        fotos: [],
        indexes: []
      };
    }

    groups[kunde].e2Parts.push(Number(r.e2_out || 0));
    groups[kunde].e2Total += Number(r.e2_out || 0);
    groups[kunde].h1Total += Number(r.h1_out || 0);
    groups[kunde].einwegTotal += Number(r.einweg_out || 0);
    groups[kunde].epalTotal += Number(r.epal_out || 0);

    if(r.foto){
      groups[kunde].fotos.push(r.foto);
    }

    groups[kunde].indexes.push(index);
  });

  Object.values(groups).forEach(g=>{
    if(g.avisE2 === null){
      g.status = "KEIN AVIS";
      g.differenz = "";
    } else {
      g.differenz = g.e2Total - g.avisE2;

      if(g.e2Total < g.avisE2){
        g.status = "OFFEN";
      } else if(g.e2Total === g.avisE2){
        g.status = "OK";
      } else {
        g.status = "ZU VIEL";
      }
    }

    g.zusammensetzung = g.e2Parts.length
      ? `${g.e2Parts.join(" + ")} = ${g.e2Total}`
      : "";
  });

  return Object.values(groups);
}

function render(){
  const t=document.getElementById("table");
  t.innerHTML="";

  const groups = getGroupedData();

  groups.forEach(g=>{
    let statusClass = "ok";
    if(g.status === "OFFEN") statusClass = "open";
    if(g.status === "ZU VIEL" || g.status === "KEIN AVIS") statusClass = "warn";

    t.innerHTML+=`
<tr>
<td>${g.kunde}</td>
<td>${g.avisE2 ?? ""}</td>
<td>${g.zusammensetzung}</td>
<td>${g.e2Total}</td>
<td>${g.differenz}</td>
<td class="${statusClass}">${g.status}</td>
<td>${g.h1Total}</td>
<td>${g.einwegTotal}</td>
<td>${g.epalTotal}</td>
<td>${g.fotos.join("<br>")}</td>
<td><button class="action-btn" onclick="delCustomer('${encodeURIComponent(g.kunde)}')">X</button></td>
</tr>`;
  });
}

function delCustomer(encodedKunde){
  const kunde = decodeURIComponent(encodedKunde);

  if(confirm(`Alle Einträge für ${kunde} löschen?`)){
    data = data.filter(r => r.kunde !== kunde);
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

  const exportData = data.map(r => ({
    Datum: r.datum,
    Kunde: r.kunde,
    "AVIS E2": getCustomerAvis(r.kunde) ?? "",
    "E2 OUT Einzel": r.e2_out,
    "H1 OUT": r.h1_out,
    "Einweg OUT": r.einweg_out,
    "EPAL OUT": r.epal_out,
    Foto: r.foto || ""
  }));

  exportData.push({});
  exportData.push({
    Datum: "",
    Kunde: "SUMMEN JE KUNDE",
    "AVIS E2": "",
    "E2 OUT Einzel": "",
    "H1 OUT": "",
    "Einweg OUT": "",
    "EPAL OUT": "",
    Foto: ""
  });

  const groups = getGroupedData();

  groups.forEach(g=>{
    exportData.push({
      Datum: "",
      Kunde: g.kunde,
      "AVIS E2": g.avisE2 ?? "",
      "E2 OUT Einzel": g.zusammensetzung,
      "H1 OUT": g.h1Total,
      "Einweg OUT": g.einwegTotal,
      "EPAL OUT": g.epalTotal,
      Status: g.status,
      Differenz: g.differenz,
      Foto: g.fotos.join(", ")
    });
  });

  exportData.push({});
  exportData.push({
    Datum: "",
    Kunde: "GESAMTSUMME",
    "AVIS E2": groups.reduce((a,b)=>a+Number(b.avisE2||0),0),
    "E2 OUT Einzel": data.reduce((a,b)=>a+Number(b.e2_out||0),0),
    "H1 OUT": data.reduce((a,b)=>a+Number(b.h1_out||0),0),
    "Einweg OUT": data.reduce((a,b)=>a+Number(b.einweg_out||0),0),
    "EPAL OUT": data.reduce((a,b)=>a+Number(b.epal_out||0),0),
    Status: "",
    Differenz: "",
    Foto: ""
  });

  const ws = XLSX.utils.json_to_sheet(exportData);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Warenausgang");

  const today = datumInput.value || "report";
  XLSX.writeFile(wb, `Warenausgang_${today}.xlsx`);
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
