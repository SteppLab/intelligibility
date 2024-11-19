layout: calculate
title: "TOOL FOR INTELLIGIBILITY ASSESSMENT"
permalink: /calculate

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <title>Stepp Lab Calculator for Intelligibility</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      font-size: 12px;
      margin: 20px;
      background-color: #fff;
      color: #333;
    }

    h1 {
      text-align: center;
      color: #222;
      font-size: 1.5em;
    }

    .metadata {
      text-align: center;
      font-size: 12px;
      color: #555;
      margin-bottom: 20px;
    }

   .citation {
      font-size: 12px;
      color: #555;
    }

    form {
      background: #fff;
      border: 1px solid #ddd;
      padding: 20px;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      max-width: 100%;
      margin: 0 auto;
    }

    .form-group {
      margin-bottom: 15px;
    }

    .form-group label {
      display: block;
      font-weight: bold;
      margin-bottom: 5px;
    }

    .form-group .slider-container {
      position: relative;
      display: block;
      margin-top: 5px;
    }

    .form-group .slider-container input[type="range"] {
      width: 100%;
    }

    .form-group .slider-container .slider-label {
      position: absolute;
      top: -30px;
      left: 50%;
      transform: translateX(-50%);
      background: #007bff;
      color: #fff;
      padding: 5px 10px;
      border-radius: 3px;
      font-weight: bold;
      font-size: 12px;
    }

    select, input[type="range"] {
      width: 100%;
      padding: 8px;
      margin-bottom: 10px;
    }

    button {
      display: block;
      width: 100%;
      padding: 10px;
      background-color: #007bff;
      color: #fff;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-size: 12px;
    }

    button:hover {
      background-color: #0056b3;
    }

    #results {
      background: #fff;
      border: 1px solid #ddd;
      padding: 20px;
      border-radius: 5px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      margin: 20px auto;
      max-width: 100%;
    }

.scrollable-results {
      max-height: 200px;
      overflow-y: auto;
      padding-bottom: 20px;
padding-left: 20px;
padding-right: 20px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
font-size: 12px;
    }

    table, th, td {
      border: 1px solid #ddd;
    }

    th, td {
      padding: 8px;
      text-align: center;
    }

    th {
      background-color: #007bff;
      color: white;
    }

    .result-title {
      font-weight: bold;
      text-align: center;
      font-size: 14px;
      margin-bottom: 10px;
    }

    @media (max-width: 600px) {
      body {
        margin: 10px;
      }

      h1 {
        font-size: 1.4em;
      }

      .metadata {
        font-size: 10px;
      }

      .citation {
        font-size: 10px;
      }


      .form-group label {
        font-size: 14px;
      }

      .form-group .slider-container .slider-label {
        font-size: 10px;
        padding: 4px 8px;
      }

      select, input[type="range"] {
        padding: 6px;
      }

      button {
        font-size: 12px;
        padding: 8px;
      }

      .result-title {
        font-size: 14px;
      }
    }
  </style>
</head>
<body>
  <h1>Tool for Intelligibility Assessment</h1>  
<div class="metadata">
    <p>For the best experience, we recommend using this tool on a tablet or a larger screen device.</p>
  </div>
  <form id="criteriaForm">
    <div class="form-group">
      <label for="assessmentMethod">Assessment Method:</label>
      <select id="assessmentMethod" name="assessmentMethod">
        <option value="orthographic transcription">Orthographic Transcription</option>
        <option value="VAS">Visual Analog Scale</option>
      </select>
    </div>

    <div class="form-group">
      <label for="listenerType">Listener Type:</label>
      <select id="listenerType" name="listenerType">
        <option value="SLPs">SLPs</option>
        <option value="inexperienced">Inexperienced</option>
      </select>
    </div>

    <div class="form-group">
      <label for="accuracyLevel">Accuracy Level:</label>
      <div class="slider-container">
        <input type="range" id="accuracyLevel" name="accuracyLevel" min="0" max="100" step="1" value="50" oninput="updateAccuracyLabel()">
        <div class="slider-label" id="accuracyLabel">50%</div>
      </div>
    </div>

    <button type="button" onclick="findResults()">Find Results</button>
  </form>

  <div id="results">
    <div class="result-title" id="resultTitle"></div>
    <div class="scrollable-results"><table id="resultTable">
      <!-- Table content will be inserted here -->
    </table></div>
  </div>

<div class="metadata">
    <p>Last Updated: October 15, 2024 | Version: 1.1</p>
  </div>

  <div class="citation">
    <p><b>Citation: </b> Dahl, K.L., Balz, M.A., Díaz Cádiz, M., & Stepp, C.E. (<i>In Press</i>). How to efficiently measure the intelligibility of people with Parkinson’s disease. <i>American Journal of Speech-Language Pathology</i>.</p>
  </div>


  <script>
    const baseURL = 'https://raw.githubusercontent.com/SteppLab/listener_calculator/main/';

    const fileIdMap = {
      orthographic_transcription_inexperienced: 'orthographic_transcription_inexperienced.JSON',
      orthographic_transcription_SLPs: 'orthographic_transcription_SLPs.JSON',
      VAS_inexperienced: 'VAS_inexperienced.JSON',
      VAS_SLPs: 'VAS_SLPs.JSON'
    };

    function getFileURL(method, listenerType) {
      const fileId = fileIdMap[`${method.replace(/ /g, '_')}_${listenerType.replace(/ /g, '_')}`];
      return `${baseURL}${fileId}`;
    }

    async function fetchLookupData(url) {
      try {
        const response = await fetch(url);
        const data = await response.json();
        return data;
      } catch (error) {
        console.error('Error fetching lookup data:', error);
        return {};
      }
    }

    function findClosest(value, array) {
      return array.reduce((prev, curr) => 
        Math.abs(curr - value) < Math.abs(prev - value) ? curr : prev
      );
    }

    async function findResults() {
      const form = document.getElementById('criteriaForm');
      const method = form.assessmentMethod.value.replace(/ /g, '_');
      const listenerType = form.listenerType.value.replace(/ /g, '_');
      const accuracy = parseInt(form.accuracyLevel.value, 10); 
      
      const fileURL = getFileURL(method, listenerType);
      const data = await fetchLookupData(fileURL);

      if (!data.sentences || !data.listeners || !data.accuracy) {
        console.error('Invalid data format');
        return;
      }

      const sentenceValues = data.sentences;
      const listenerValues = data.listeners;
      const accuracyData = data.accuracy;

      let results = [];

      for (const sentence of sentenceValues) {
        for (const listener of listenerValues) {
          const currentAccuracy = accuracyData[sentence]?.[listener];
          if (currentAccuracy !== undefined && currentAccuracy >= accuracy) {
            results.push({ sentences: sentence, listeners: listener, accuracy: currentAccuracy });
          }
        }
      }

      if (results.length > 0) {
        displayResults(results);
      } else {
        document.getElementById('resultTable').innerHTML = '<tr><td colspan="3">No matching results found.</td></tr>';
      }
    }

    function displayResults(results) {
      const resultTitle = document.getElementById('resultTitle');
      const resultTable = document.getElementById('resultTable');
      const method = document.getElementById('assessmentMethod').value;
      const listenerType = document.getElementById('listenerType').value;

      const methodLabel = method === 'orthographic transcription' ? 'Orthographic Transcription' : 'a Visual Analog Scale (VAS)';
      const listenerLabel = listenerType === 'SLPs' ? 'SLPs' : 'Inexperienced Listeners';
      resultTitle.textContent = `Results for ${listenerLabel} using ${methodLabel}`;

      console.log(resultTable);
      if (results.length > 0) {
        const headers = `
        <tr>
          <th onclick="sortTable(0)">SIT Sentences ↑</th>
          <th onclick="sortTable(1)">${listenerLabel}</th>
          <th onclick="sortTable(2)">Accuracy</th>
        </tr>
      `;
      const rows = results
        .map(result => `
          <tr>
            <td>${result.sentences}</td>
            <td>${result.listeners}</td>
            <td>${result.accuracy}%</td>
          </tr>
        `)
        .join('');

      resultTable.innerHTML = headers + rows;
      } else {
        resultTable.innerHTML = '<tr><td colspan="3">No matching results found.</td></tr>';
      }
    }

    function updateAccuracyLabel() {
      const slider = document.getElementById('accuracyLevel');
      const label = document.getElementById('accuracyLabel');
      const value = slider.value;
      label.textContent = `${value}%`;
      const sliderWidth = slider.offsetWidth;
      const labelWidth = label.offsetWidth;
      const offset = (sliderWidth / 100) * value - (labelWidth / 2);
      label.style.left = `${offset}px`;
    }

    function sortTable(n) {
  const table = document.getElementById("resultTable");
  const headers = table.getElementsByTagName("TH");
  let rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
  switching = true;
  dir = "asc"; // Set the initial sorting direction to ascending

  // Remove existing arrows from all headers
  for (let header of headers) {
    header.innerHTML = header.innerHTML.replace(/ ↓| ↑/g, ""); // Remove any existing arrows
  }

  while (switching) {
    switching = false;
    rows = table.rows;
    for (i = 1; i < (rows.length - 1); i++) {
      shouldSwitch = false;
      x = rows[i].getElementsByTagName("TD")[n];
      y = rows[i + 1].getElementsByTagName("TD")[n];

      // Check if it's the "Accuracy" column (which contains percentages)
      let xContent, yContent;
      if (n === 2) { // Assuming column 2 is the "Accuracy" column
        xContent = parseFloat(x.innerHTML.replace('%', '')); // Remove % and convert to number
        yContent = parseFloat(y.innerHTML.replace('%', ''));
      } else {
        // For other columns (numbers or text)
        xContent = isNaN(x.innerHTML) ? x.innerHTML.toLowerCase() : parseFloat(x.innerHTML);
        yContent = isNaN(y.innerHTML) ? y.innerHTML.toLowerCase() : parseFloat(y.innerHTML);
      }

      if (dir === "asc") {
        if (xContent > yContent) {
          shouldSwitch = true;
          break;
        }
      } else if (dir === "desc") {
        if (xContent < yContent) {
          shouldSwitch = true;
          break;
        }
      }
    }
    if (shouldSwitch) {
      rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
      switching = true;
      switchcount++;
    } else {
      if (switchcount === 0 && dir === "asc") {
        dir = "desc";
        switching = true;
      }
    }
  }

  // Add the appropriate arrow to the sorted column header
  if (dir === "asc") {
    headers[n].innerHTML += " ↑";
  } else {
    headers[n].innerHTML += " ↓";
  }
}

    document.addEventListener('DOMContentLoaded', () => {
      updateAccuracyLabel();
    });
  </script>
</body>
</html>