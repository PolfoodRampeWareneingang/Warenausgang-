

    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      padding: 15px;
      margin: 0;
    }

    .box {
      background: white;
      padding: 15px;
      max-width: 900px;
      margin: auto;
      border-radius: 10px;
      box-shadow: 0 0 10px rgba(0,0,0,0.1);
    }

    h2 {
      text-align: center;
      margin-top: 0;
    }

    label {
      font-weight: bold;
      display: block;
      margin-top: 10px;
    }

    input {
      width: 100%;
      padding: 10px;
      margin: 6px 0;
      box-sizing: border-box;
      font-size: 16px;
    }

    button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      background: black;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 16px;
    }

    button:hover {
      opacity: 0.9;
    }

    table {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
      font-size: 12px;
      background: white;
    }

    th, td {
      border: 1px solid #ddd;
      padding: 6px;
      text-align: center;
      vertical-align: middle;
    }

    th {
      background: #eee;
    }

    .dropdown {
      position: relative;
      width: 100%;
    }

    .dropdown input {
      width: 100%;
    }

    .dropdown-list {
      position: absolute;
      width: 100%;
      border: 1px solid #ccc;
      border-top: none;
      max-height: 220px;
      overflow-y: auto;
      background: white;
      display: none;
      z-index: 1000;
      box-sizing: border-box;
      -webkit-overflow-scrolling: touch;
    }

    .dropdown-item {
      padding: 14px;
      font-size: 16px;
      cursor: pointer;
      text-align: left;
      white-space: pre-line;
    }

    .dropdown-item:hover {
      background: #f0f0f0;
    }

    .action-btn {
      background: #c62828;
      color: white;
      border: none;
      padding: 6px 10px;
      border-radius: 4px;
      cursor: pointer;
      width: auto;
      margin: 0;
      font-size: 12px;
    }

    .summary-box {
      margin-top: 20px;
      background: #fafafa;
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 12px;
    }

    .summary-title {
      font-weight: bold;
      margin-bottom: 10px;
      font-size: 16px;
    }

    .summary-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
    }

    .summary-item {
      background: white;
      border: 1px solid #ddd;
      border-radius: 8px;
      padding: 10px;
      text-align: center;
    }

    .summary-item strong {
      display: block;
      font-size: 18px;
      margin-top: 4px;
    }

    @media (max-width: 600px) {
      .summary-grid {
        grid-template-columns: 1fr 1fr;
      }

      table {
        font-size: 11px;
      }

      th, td {
        padding: 4px;
      }
    }
  </style>
</head>
<body>

<div class="box">
  <h2>🚛 Warenausgang Polfood GmbH</h2>

  <label for="datum">Datum</label>
  <input type="date" id="datum">

  <label for="kunde">Kunde</label>
  <div class="dropdown">
    <input type="text" id="kunde" placeholder="Kunde wählen oder eingeben" autocomplete="off">
    <div id="kundenList" class="dropdown-list"></div>
  </div>

  <label for="e2_out">E2 OUT</label>
  <input type="text" id="e2_out" inputmode="numeric" pattern="[0-9]*">

  <label for="h1_out">H1 OUT</label>
  <input type="text" id="h1_out" inputmode="numeric" pattern="[0-9]*">

  <label for="einweg_out">Einweg OUT</label>
  <input type="text" id="einweg_out" inputmode="numeric" pattern="[0-9]*">

  <label for="epal_out">EPAL OUT</label>
  <input type="text" id="epal_out" inputmode="numeric" pattern="[0-9]*">

  <button type="button" onclick="addEntry()">➕ Eintrag speichern</button>
  <button type="button" onclick="exportExcel()">📦 Excel Tagesbericht exportieren</button>
  <button type="button" onclick="clearData()">🗑️ Alle Daten löschen</button>
  <button type="button" onclick="resetKunden()">♻️ Kundenliste zurücksetzen</button>

  <div class="summary-box">
    <div class="summary-title">Automatische Summen für das gewählte Datum</div>
    <div class="summary-grid">
      <div class="summary-item">E2 OUT<strong id="sum_e2_out">0</strong></div>
      <div class="summary-item">H1 OUT<strong id="sum_h1_out">0</strong></div>
      <div class="summary-item">Einweg OUT<strong id="sum_einweg_out">0</strong></div>
      <div class="summary-item">EPAL OUT<strong id="sum_epal_out">0</strong></div>
    </div>
  </div>

  <table>
    <thead>
      <tr>
        <th>Datum</th>
        <th>Kunde</th>
        <th>E2 OUT</th>
        <th>H1 OUT</th>
        <th>Einweg OUT</th>
        <th>EPAL OUT</th>
        <th>Aktion</th>
      </tr>
    </thead>
    <tbody id="table"></tbody>
  </table>
</div>

<script>
  const standardKunden = [
    "1 / Wache",
    "2 / Feddersen",
    "3 / Willi Hofner",
    "4 / Bremen\nEB:",
    "5 / Bremerhaven",
    "6 / Bad Oldesloe\nEB:",
    "8 / Havelland\nNr. 618338",
    "9 / Fl. Schmidt und Sohn",
    "10 / GT Emporium",
    "11 / Metzgerei Dressel",
    "12 / Atlas CC Köln",
    "13 / Freiburger Frischwaren",
    "14 / Freiburger Frischwaren",
    "18 / Fölster",
    "29 / Rostock",
    "30 / Petersen",
    "32 / BARLU",
    "33 / Frisch Frost\nNr.",
    "45 / Wolf 2 Annaberg",
    "48 / Torney",
    "51 / Käferstein",
    "52 / Hamberger Friedenstraße\nNr. 23307",
    "53 / Hamb. Riem\nNr. 23315",
    "54 / Hamberger Berlin\nNr. 23059, 23517",
    "55 / Hamberger Frankfurt\nNr.",
    "56 / Fischer Gmbh",
    "57 / Wolf + Kunt.",
    "58 / FMS",
    "59 / DUSP",
    "66 / Karl Mehl",
    "70 / FEK H/G",
    "74 / Landpute",
    "76 / Wunderland",
    "77 / Stefan Elst",
    "80 / März",
    "81 / MigroMa\nNr. 202620",
    "82 / M. Walk",
    "83 / Dimter",
    "84 / C+C Landau",
    "85 / Meemken Sandmann",
    "87 / Richter\nNr.",
    "88 / Deu. See\nNr.",
    "89 / Zimmermann",
    "90 / MEGEM",
    "91 / C&C Bingen",
    "92 / Weisenhorn",
    "93 / Enders\nNr.",
    "94 / Rothe\nNr.",
    "95 / TLC",
    "96 / NK",
    "97 / Atlas",
    "98 / BLF",
    "99 / Chickeria",
    "Unna",
    "Yu An",
    "Futterhappen",
    "Tosbiks",
    "100 / Konrad Böhnlein"
  ];

  let data = JSON.parse(localStorage.getItem("warenausgang_data") || "[]");
  let kunden = JSON.parse(localStorage.getItem("kunden_liste") || "null") || [...standardKunden];

  const kundeInput = document.getElementById("kunde");
  const kundenList = document.getElementById("kundenList");
  const datumInput = document.getElementById("datum");

  if (!datumInput.value) {
    datumInput.value = new Date().toISOString().split("T")[0];
  }

  function save() {
    localStorage.setItem("warenausgang_data", JSON.stringify(data));
  }

  function saveKunden() {
    localStorage.setItem("kunden_liste", JSON.stringify(kunden));
  }

  function sortiereKunden() {
    kunden.sort((a, b) => a.localeCompare(b, "de", { sensitivity: "base" }));
  }

  function renderKunden(filter = "") {
    kundenList.innerHTML = "";

    const gefiltert = kunden.filter(k =>
      k.toLowerCase().includes(filter.toLowerCase())
    );

    gefiltert.forEach(k => {
      const div = document.createElement("div");
      div.className = "dropdown-item";
      div.textContent = k;
      div.onclick = () => {
        kundeInput.value = k;
        kundenList.style.display = "none";
      };
      kundenList.appendChild(div);
    });

    kundenList.style.display = gefiltert.length ? "block" : "none";
  }

  function addKunde(value) {
    const val = value.trim();
    if (!val) return;

    const exists = kunden.some(k => k.toLowerCase() === val.toLowerCase());

    if (!exists) {
      kunden.push(val);
      sortiereKunden();
      saveKunden();
    }
  }

  function resetKunden() {
    if (confirm("Kundenliste auf Standard zurücksetzen?")) {
      kunden = [...standardKunden];
      sortiereKunden();
      saveKunden();
      renderKunden(kundeInput.value);
      alert("Kundenliste wurde zurückgesetzt.");
    }
  }

  kundeInput.addEventListener("input", () => {
    renderKunden(kundeInput.value);
  });

  kundeInput.addEventListener("focus", () => {
    renderKunden(kundeInput.value);
  });

  kundeInput.addEventListener("keydown", (e) => {
    if (e.key === "Enter") {
      e.preventDefault();
      addKunde(kundeInput.value);
      renderKunden(kundeInput.value);
    }
  });

  datumInput.addEventListener("change", () => {
    render();
    renderSummen();
  });

  document.addEventListener("click", (e) => {
    if (!e.target.closest(".dropdown")) {
      kundenList.style.display = "none";
    }
  });

  function addEntry() {
    const datum = document.getElementById("datum").value;
    const kunde = document.getElementById("kunde").value.trim();

    if (!datum) {
      alert("Bitte Datum eingeben.");
      return;
    }

    if (!kunde) {
      alert("Bitte Kunde eingeben.");
      return;
    }

    addKunde(kunde);

    data.push({
      datum: datum,
      kunde: kunde,
      e2_out: Number(document.getElementById("e2_out").value || 0),
      h1_out: Number(document.getElementById("h1_out").value || 0),
      einweg_out: Number(document.getElementById("einweg_out").value || 0),
      epal_out: Number(document.getElementById("epal_out").value || 0)
    });

    save();
    render();
    renderSummen();

    document.getElementById("e2_out").value = "";
    document.getElementById("h1_out").value = "";
    document.getElementById("einweg_out").value = "";
    document.getElementById("epal_out").value = "";

    document.getElementById("e2_out").focus();
  }

  function deleteEntry(index) {
    if (confirm("Diesen Eintrag löschen?")) {
      data.splice(index, 1);
      save();
      render();
      renderSummen();
    }
  }

  function render() {
    const tbody = document.getElementById("table");
    tbody.innerHTML = "";

    const selectedDate = datumInput.value;

    const filteredData = selectedDate
      ? data.filter(r => r.datum === selectedDate)
      : data;

    filteredData.forEach((r) => {
      const originalIndex = data.indexOf(r);

      tbody.innerHTML += `
        <tr>
          <td>${r.datum}</td>
          <td style="white-space: pre-line;">${r.kunde}</td>
          <td>${r.e2_out}</td>
          <td>${r.h1_out}</td>
          <td>${r.einweg_out}</td>
          <td>${r.epal_out}</td>
          <td><button type="button" class="action-btn" onclick="deleteEntry(${originalIndex})">Löschen</button></td>
        </tr>
      `;
    });
  }

  function renderSummen() {
    const selectedDate = datumInput.value;

    const filteredData = selectedDate
      ? data.filter(r => r.datum === selectedDate)
      : data;

    const sumE2 = filteredData.reduce((sum, r) => sum + Number(r.e2_out || 0), 0);
    const sumH1 = filteredData.reduce((sum, r) => sum + Number(r.h1_out || 0), 0);
    const sumEinweg = filteredData.reduce((sum, r) => sum + Number(r.einweg_out || 0), 0);
    const sumEpal = filteredData.reduce((sum, r) => sum + Number(r.epal_out || 0), 0);

    document.getElementById("sum_e2_out").textContent = sumE2;
    document.getElementById("sum_h1_out").textContent = sumH1;
    document.getElementById("sum_einweg_out").textContent = sumEinweg;
    document.getElementById("sum_epal_out").textContent = sumEpal;
  }

  function exportExcel() {
    if (data.length === 0) {
      alert("Keine Daten!");
      return;
    }

    const selectedDate = datumInput.value;

    const filteredData = selectedDate
      ? data.filter(r => r.datum === selectedDate)
      : data;

    if (filteredData.length === 0) {
      alert("Keine Daten für dieses Datum!");
      return;
    }

    const exportData = filteredData.map(r => ({
      Datum: r.datum,
      Kunde: r.kunde.replace(/\n/g, " "),
      "E2 OUT": r.e2_out,
      "H1 OUT": r.h1_out,
      "Einweg OUT": r.einweg_out,
      "EPAL OUT": r.epal_out
    }));

    const summen = filteredData.reduce((acc, r) => {
      acc["E2 OUT"] += Number(r.e2_out || 0);
      acc["H1 OUT"] += Number(r.h1_out || 0);
      acc["Einweg OUT"] += Number(r.einweg_out || 0);
      acc["EPAL OUT"] += Number(r.epal_out || 0);
      return acc;
    }, {
      "E2 OUT": 0,
      "H1 OUT": 0,
      "Einweg OUT": 0,
      "EPAL OUT": 0
    });

    exportData.push({
      Datum: "",
      Kunde: "SUMME",
      "E2 OUT": summen["E2 OUT"],
      "H1 OUT": summen["H1 OUT"],
      "Einweg OUT": summen["Einweg OUT"],
      "EPAL OUT": summen["EPAL OUT"]
    });

    const ws = XLSX.utils.json_to_sheet(exportData);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "Tagesbericht");

    const today = selectedDate || "report";
    XLSX.writeFile(wb, `Warenausgang_${today}.xlsx`);
  }

  function clearData() {
    if (confirm("Wirklich alle Daten löschen?")) {
      data = [];
      save();
      render();
      renderSummen();
    }
  }

  sortiereKunden();
  saveKunden();
  render();
  renderSummen();
</script>

</body>
</html>
