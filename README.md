<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AgroVision Pro | Alles-in-Één Landbouw Dashboard</title>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #2e7d32;
            --secondary: #388e3c;
            --accent: #8bc34a;
            --warning: #d32f2f;
            --background: #f5f7fa;
        }
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            margin: 0;
            padding: 0;
            background: var(--background);
            color: #333;
        }
        .dashboard {
            display: grid;
            grid-template-columns: 280px 1fr;
            min-height: 100vh;
        }
        .sidebar {
            background: white;
            padding: 20px;
            box-shadow: 2px 0 10px rgba(0,0,0,0.1);
            overflow-y: auto;
        }
        .main-content {
            padding: 20px;
            overflow-y: auto;
        }
        .card {
            background: white;
            border-radius: 10px;
            box-shadow: 0 2px 15px rgba(0,0,0,0.05);
            padding: 20px;
            margin-bottom: 20px;
        }
        h1, h2, h3 {
            color: var(--primary);
            margin-top: 0;
        }
        .nav-menu {
            list-style: none;
            padding: 0;
        }
        .nav-item {
            padding: 12px 15px;
            border-radius: 5px;
            margin-bottom: 5px;
            cursor: pointer;
            transition: all 0.3s;
        }
        .nav-item:hover, .nav-item.active {
            background: var(--accent);
            color: white;
        }
        .tab-content {
            display: none;
        }
        .tab-content.active {
            display: block;
        }
        .map-container {
            height: 500px;
            border-radius: 8px;
            margin-top: 15px;
        }
        .grid-2 {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
        }
        .grid-3 {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 20px;
        }
        button {
            background: var(--primary);
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 5px;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            background: var(--secondary);
        }
        .warning {
            color: var(--warning);
            font-weight: bold;
        }
        .sensor-value {
            font-size: 1.2em;
            font-weight: bold;
        }
        .ndvi-legend {
            background: linear-gradient(to right, #d73027, #f46d43, #fdae61, #fee08b, #ffffbf, #d9ef8b, #a6d96a, #66bd63, #1a9850);
            height: 20px;
            border-radius: 5px;
            margin: 10px 0;
        }
        #task-list {
            max-height: 300px;
            overflow-y: auto;
        }
        .task-item {
            padding: 10px;
            border-bottom: 1px solid #eee;
        }
        #disease-preview {
            max-width: 100%;
            border: 2px solid var(--primary);
            margin-top: 10px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="dashboard">
        <!-- Sidebar Navigatie -->
        <div class="sidebar">
            <h1>AgroVision Pro</h1>
            
            <ul class="nav-menu">
                <li class="nav-item active" onclick="showTab('dashboard')">Dashboard</li>
                <li class="nav-item" onclick="showTab('ndvi')">NDVI Analyse</li>
                <li class="nav-item" onclick="showTab('tasks')">Taakplanner</li>
                <li class="nav-item" onclick="showTab('disease')">Ziekteherkenning</li>
                <li class="nav-item" onclick="showTab('economy')">Economie</li>
                <li class="nav-item" onclick="showTab('weather')">Weer & Klimaat</li>
            </ul>
            
            <!-- Snelle Statusoverzicht -->
            <div class="card" style="margin-top: 20px;">
                <h3>Statusoverzicht</h3>
                <p><strong>Laatste update:</strong> <span id="last-update">-</span></p>
                <p><strong>Actieve waarschuwingen:</strong> <span id="alert-count">0</span></p>
                <p><strong>Volgende taak:</strong> <span id="next-task">-</span></p>
            </div>
        </div>

        <!-- Hoofdcontent -->
        <div class="main-content">
            <!-- Dashboard Tab -->
            <div id="dashboard" class="tab-content active">
                <h2>Dashboard Overzicht</h2>
                
                <div class="grid-3">
                    <!-- Weer Widget -->
                    <div class="card">
                        <h3>Weerstation IMARIA261</h3>
                        <div class="grid-2">
                            <div>
                                <p>Temperatuur:</p>
                                <p class="sensor-value" id="current-temp">-</p>
                            </div>
                            <div>
                                <p>Vochtigheid:</p>
                                <p class="sensor-value" id="current-humidity">-</p>
                            </div>
                        </div>
                        <div class="grid-2" style="margin-top: 10px;">
                            <div>
                                <p>Neerslag (24u):</p>
                                <p class="sensor-value" id="current-rain">-</p>
                            </div>
                            <div>
                                <p>Grondvocht:</p>
                                <p class="sensor-value" id="soil-moisture">-</p>
                            </div>
                        </div>
                        <button onclick="fetchWeatherData()" style="margin-top: 10px;">Vernieuw Data</button>
                    </div>
                    
                    <!-- NDVI Samenvatting -->
                    <div class="card">
                        <h3>Vegetatie-index</h3>
                        <div class="ndvi-legend"></div>
                        <p>Gemiddelde NDVI: <span class="sensor-value" id="avg-ndvi">0.72</span></p>
                        <p>Probleemgebieden: <span class="sensor-value" id="low-ndvi">3.5 ha</span></p>
                        <button onclick="showTab('ndvi')" style="margin-top: 10px;">Bekijk Kaart</button>
                    </div>
                    
                    <!-- Economie Samenvatting -->
                    <div class="card">
                        <h3>Economisch Overzicht</h3>
                        <p>Verwachte opbrengst: <span class="sensor-value">€42,500</span></p>
                        <p>Kosten deze maand: <span class="sensor-value">€8,200</span></p>
                        <p>Winstmarge: <span class="sensor-value">68%</span></p>
                        <button onclick="showTab('economy')" style="margin-top: 10px;">Details</button>
                    </div>
                </div>
                
                <!-- Phytophthora Risico -->
                <div class="card" style="margin-top: 20px;">
                    <h3>Gewasgezondheid</h3>
                    <div class="grid-2">
                        <div>
                            <p>Phytophthora risico:</p>
                            <p class="sensor-value" id="phytophthora-risk">-</p>
                            <p id="phytophthora-advice">-</p>
                        </div>
                        <div>
                            <p>Actuele waarschuwingen:</p>
                            <div id="health-alerts"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- NDVI Tab -->
            <div id="ndvi" class="tab-content">
                <h2>NDVI Vegetatie Analyse</h2>
                <div class="map-container" id="ndvi-map"></div>
                <div class="grid-2" style="margin-top: 20px;">
                    <div class="card">
                        <h3>Legenda</h3>
                        <div class="ndvi-legend"></div>
                        <p style="text-align: center;">0 (geen vegetatie) → 1 (gezonde vegetatie)</p>
                    </div>
                    <div class="card">
                        <h3>Acties</h3>
                        <button onclick="generateNDVIReport()">Genereer Rapport</button>
                        <button onclick="identifyProblemAreas()" style="margin-top: 10px;">Identificeer Probleemgebieden</button>
                    </div>
                </div>
            </div>

            <!-- Taakplanner Tab -->
            <div id="tasks" class="tab-content">
                <h2>Taakplanner</h2>
                <div class="grid-2">
                    <div class="card">
                        <h3>Nieuwe Taak</h3>
                        <label>Taaknaam:</label>
                        <input type="text" id="task-name" style="width: 100%; padding: 8px; margin-bottom: 10px;">
                        
                        <label>Prioriteit:</label>
                        <select id="task-priority" style="width: 100%; padding: 8px; margin-bottom: 10px;">
                            <option value="low">Laag</option>
                            <option value="medium">Gemiddeld</option>
                            <option value="high">Hoog</option>
                        </select>
                        
                        <label>Deadline:</label>
                        <input type="date" id="task-deadline" style="width: 100%; padding: 8px; margin-bottom: 10px;">
                        
                        <button onclick="addTask()">Taak Toevoegen</button>
                    </div>
                    <div class="card">
                        <h3>Openstaande Taken</h3>
                        <div id="task-list">
                            <!-- Taken worden hier dynamisch ingeladen -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- Ziekteherkenning Tab -->
            <div id="disease" class="tab-content">
                <h2>Ziekteherkenning</h2>
                <div class="grid-2">
                    <div class="card">
                        <h3>Diagnose Tool</h3>
                        <input type="file" id="disease-upload" accept="image/*" style="margin-bottom: 10px;">
                        <canvas id="disease-preview" width="400" height="300"></canvas>
                        <button onclick="analyzeDisease()" style="margin-top: 10px;">Analyseer Afbeelding</button>
                    </div>
                    <div class="card">
                        <h3>Resultaten</h3>
                        <div id="disease-results">
                            <p>Upload een foto van gewasbladeren voor analyse.</p>
                        </div>
                        <div id="treatment-advice" style="margin-top: 20px;"></div>
                    </div>
                </div>
            </div>

            <!-- Weer & Klimaat Tab -->
            <div id="weather" class="tab-content">
                <h2>Weer & Klimaat Data</h2>
                <div class="grid-2">
                    <div class="card">
                        <h3>Live Weerdata</h3>
                        <div id="weather-chart-container" style="height: 300px;">
                            <canvas id="weather-chart"></canvas>
                        </div>
                    </div>
                    <div class="card">
                        <h3>Weersvoorspelling</h3>
                        <div id="forecast-data">
                            <!-- Voorspellingsdata wordt hier ingeladen -->
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    <script>
        // Weerstation API configuratie
        const WEATHER_API = {
            baseUrl: "https://api.weather.com/v2/",
            key: "d9005fe733594906805fe733597906fd",
            stationId: "IMARIA261"
        };

        // Globale variabelen
        let ndviMap;
        let weatherChart;
        let tasks = [];
        let lastWeatherData = null;

        // Initialisatie
        function init() {
            // Laad initiële data
            fetchWeatherData();
            loadTasks();
            initNDVIMap();
            initWeatherChart();
            
            // Stel interval in voor updates
            setInterval(fetchWeatherData, 600000); // Elke 10 minuten
            updateLastUpdateTime();
        }

        // Toon tabblad
        function showTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.remove('active');
            });
            document.querySelectorAll('.nav-item').forEach(item => {
                item.classList.remove('active');
            });
            
            document.getElementById(tabId).classList.add('active');
            document.querySelector(`.nav-item[onclick="showTab('${tabId}')"]`).classList.add('active');
            
            // Update kaartgrootte bij tabwissel
            if (tabId === 'ndvi' && ndviMap) {
                setTimeout(() => {
                    ndviMap.invalidateSize();
                }, 100);
            }
        }

        // Haal weerdata op
        async function fetchWeatherData() {
            try {
                // Simulatie - vervang met echte API call
                const mockData = {
                    temperature: (15 + Math.random() * 10).toFixed(1),
                    humidity: (60 + Math.random() * 30).toFixed(0),
                    precipitation: (Math.random() * 5).toFixed(1),
                    soil_moisture: (30 + Math.random() * 50).toFixed(0),
                    timestamp: new Date().toLocaleString()
                };
                
                // Update UI
                document.getElementById('current-temp').textContent = `${mockData.temperature}°C`;
                document.getElementById('current-humidity').textContent = `${mockData.humidity}%`;
                document.getElementById('current-rain').textContent = `${mockData.precipitation} mm`;
                document.getElementById('soil-moisture').textContent = `${mockData.soil_moisture}%`;
                document.getElementById('last-update').textContent = mockData.timestamp;
                
                // Bewaar data voor analyse
                lastWeatherData = mockData;
                
                // Voer analyses uit
                analyzePhytophthoraRisk();
                updateWeatherChart();
                
                return mockData;
                
            } catch (error) {
                console.error("Fout bij ophalen weerdata:", error);
                document.getElementById('health-alerts').innerHTML = 
                    `<p class="warning">Kon geen verbinding maken met weerstation</p>`;
            }
        }

        // Phytophthora risicoanalyse
        function analyzePhytophthoraRisk() {
            if (!lastWeatherData) return;
            
            const humidity = parseFloat(lastWeatherData.humidity);
            const rain = parseFloat(lastWeatherData.precipitation);
            const temp = parseFloat(lastWeatherData.temperature);
            
            let riskLevel = "Laag";
            let advice = "Geen actie vereist";
            
            if (humidity > 80 && rain > 3 && temp > 10) {
                riskLevel = "Hoog";
                advice = "Preventieve behandeling aanbevolen";
                document.getElementById('alert-count').textContent = 
                    parseInt(document.getElementById('alert-count').textContent) + 1;
                
                document.getElementById('health-alerts').innerHTML = 
                    `<p class="warning">🚨 HOOG Phytophthora risico! (Vochtigheid: ${humidity}%, Regen: ${rain}mm)</p>`;
            } else if (humidity > 70 && rain > 1) {
                riskLevel = "Gemiddeld";
                advice = "Monitor gewas nauwlettend";
            }
            
            document.getElementById('phytophthora-risk').textContent = riskLevel;
            document.getElementById('phytophthora-advice').textContent = advice;
        }

        // Initialiseer NDVI kaart
        function initNDVIMap() {
            ndviMap = L.map('ndvi-map').setView([51.5, 5.5], 15);
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '&copy; OpenStreetMap contributors'
            }).addTo(ndviMap);
            
            // Simuleer NDVI data (in een echte implementatie zou dit van een API komen)
            L.geoJSON({
                type: "FeatureCollection",
                features: [/* NDVI data features zouden hier komen */]
            }).addTo(ndviMap);
        }

        // Initialiseer
