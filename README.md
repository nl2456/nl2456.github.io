<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GatorGuessr - UF Edition</title>
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
     integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
     crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
     integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
     crossorigin=""></script>

    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f0f0;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }
        
        /* --- Header --- */
        header {
            background-color: #0021A5; /* UF Blue */
            color: #FA4616; /* UF Orange */
            padding: 10px 20px;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            z-index: 1001;
        }
        header h1 {
            margin: 0;
            font-size: 1.8em;
        }

        /* --- Game Area --- */
        #game-container {
            display: flex;
            flex-grow: 1;
            height: calc(100vh - 60px); /* Full height minus header */
        }

        /* --- Left Panel (Image) --- */
        #image-panel {
            width: 50%;
            background-color: #222;
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            position: relative;
        }
        #guess-image {
            width: 100%;
            height: 100%;
            object-fit: contain;
        }

        /* --- Right Panel (Map) --- */
        #map-panel {
            width: 50%;
            position: relative;
            height: 100%;
        }
        #map {
            height: 100%;
            width: 100%;
        }

        /* --- UI Controls & Popups --- */
        #controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 1000;
        }
        #guess-button, #next-round-button {
            padding: 12px 25px;
            font-size: 1.2em;
            font-weight: bold;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            box-shadow: 0 4px 8px rgba(0,0,0,0.3);
            transition: all 0.2s ease;
        }
        #guess-button {
            background-color: #FA4616;
            color: white;
        }
        #guess-button:disabled {
            background-color: #999;
            cursor: not-allowed;
        }
        #next-round-button {
            background-color: #0021A5;
            color: white;
            display: none; /* Hidden by default */
        }

        /* --- Score & Round Info --- */
        #info-bar {
            position: absolute;
            top: 10px;
            left: 10px;
            background: rgba(255, 255, 255, 0.9);
            padding: 10px;
            border-radius: 5px;
            z-index: 1000;
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
        }
        #info-bar div {
            font-size: 1.1em;
            font-weight: bold;
            margin-bottom: 5px;
        }
        
        /* --- Result Modal --- */
        #result-modal {
            position: absolute;
            bottom: 80px; /* Above the buttons */
            left: 50%;
            transform: translateX(-50%);
            background: rgba(0, 33, 165, 0.9); /* UF Blue */
            color: white;
            padding: 20px;
            border-radius: 10px;
            z-index: 1000;
            text-align: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.4);
            display: none; /* Hidden by default */
        }
        #result-modal h2 {
            margin: 0 0 10px 0;
            color: #FA4616;
        }
        #result-modal p {
            margin: 5px 0;
            font-size: 1.1em;
        }

        /* --- Final Score Screen --- */
        #final-score-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            z-index: 2000;
            display: none; /* Hidden by default */
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            text-align: center;
        }
        #final-score-screen h1 {
            font-size: 3em;
            color: #FA4616;
        }
        #final-score-screen h2 {
            font-size: 2em;
        }
        #final-score-screen button {
            padding: 12px 25px;
            font-size: 1.2em;
            font-weight: bold;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            background-color: #FA4616;
            color: white;
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <header>
        <h1>GatorGuessr 🐊</h1>
    </header>

    <div id="game-container">
        <div id="image-panel">
            <img id="guess-image" src="" alt="Guess this location on the UF campus">
        </div>

        <div id="map-panel">
            <div id="map"></div>
            
            <div id="info-bar">
                <div id="round-display">Round: 1 / 5</div>
                <div id="score-display">Total Score: 0</div>
            </div>

            <div id="controls">
                <button id="guess-button" disabled>Place your pin on the map</button>
                <button id="next-round-button">Next Round</button>
            </div>

            <div id="result-modal">
                <h2 id="result-score"></h2>
                <p id="result-distance"></p>
                <p id="result-location-name"></p>
            </div>
        </div>
    </div>

    <div id="final-score-screen">
        <h1>Game Over!</h1>
        <h2 id="final-score-text">Your Final Score: 0</h2>
        <button id="play-again-button">Play Again</button>
    </div>

    <script>
// ######################################################################
// ##### 🐊 YOUR LOCATION DATABASE 🐊 #####
// ##### Add your new locations right here! #####
// ######################################################################
//
// Template to copy and paste:
// {
//   name: "A short description (e.g., 'Century Tower')",
//   img: "PASTE_YOUR_IMGUR_LINK_HERE.jpg",
//   lat: 29.6488,  // Your latitude
//   lon: -82.3482   // Your longitude
// },
//
const gameLocations = [
    {
        name: "Malachowsky",
        img: "https://i.imgur.com/Sf3brUu.jpeg",
        lat: 29.64483333,
        lon: -82.34763889
    },
    {
        name: "Chem lab area",
        img: "https://i.imgur.com/YKXO2NH.jpeg",
        lat: 29.6505874,
        lon: -82.343707
    },
    {
        name: "The reitz ravine woods trail",
        img: "https://i.imgur.com/TO0YKmo.jpeg",
        lat: 29.64517649,
        lon: -82.3500921
    },
    {
        name: "ben hill griffin stadium",
        img: "https://i.imgur.com/Sc0Niqd.jpeg",
        lat: 29.649396,
        lon: -82.348934
    },
    {
        name: "baughman next to lake alice",
        img: "https://i.imgur.com/TbIXy3r.jpeg",
        lat: 29.6424301,
        lon: -82.363995
    },

  {
    name: "2 Barrel Lake on Inner Rd",
    img: "https://i.imgur.com/Jpvr9ib.jpeg",
    lat: 29.647739,  // Paste your latitude
    lon: -82.340165  // Paste your longitude
  },
  {
    name: "Alley to Lib West",
    img: "https://i.imgur.com/ehyrlWC.jpeg",
    lat: 29.651062,  // Paste your latitude
    lon: -82.344635  // Paste your longitude
  },
  {
    name: "Murphree Hall Corner Rock",
    img: "https://i.imgur.com/AolhfxB.jpeg",
    lat: 29.651664,  // Paste your latitude
    lon: -82.346477  // Paste your longitude
  },
  {
    name: "Architecture Building",
    img: "https://i.imgur.com/mKzXgIN.jpeg",
    lat: 29.647721,  // Paste your latitude
    lon: -82.341963  // Paste your longitude
  },
  {
    name: "School of Music Overlook",
    img: "https://i.imgur.com/WwKUMk5.jpeg",
    lat: 29.648392,  // Paste your latitude
    lon: -82.343258  // Paste your longitude
  },
  {
    name: "Sidewalk Around Lake Alice",
    img: "https://i.imgur.com/fpItoTI.jpeg",
    lat: 29.644906,  // Paste your latitude
    lon: -82.360896  // Paste your longitude
  },
  {
    name: "Jennings Creek Bridge",
    img: "https://i.imgur.com/n8FZUG7.jpeg",
    lat: 29.643837,  // Paste your latitude
    lon: -82.341460  // Paste your longitude
  },
  {
    name: "Shands Courtyard",
    img: "https://i.imgur.com/jmDeiTY.jpeg",
    lat: 29.641204,  // Paste your latitude
    lon: -82.343940  // Paste your longitude
  },
  {
    name: "Shands Wendys",
    img: "https://i.imgur.com/ZfEfoxo.jpeg",
    lat: 29.640150,  // Paste your latitude
    lon: -82.343641  // Paste your longitude
  },
  {
    name: "Florida Museum",
    img: "https://i.imgur.com/4F6NQSq.jpeg",
    lat: 29.644312,  // Paste your latitude
    lon: -82.344089  // Paste your longitude
  },
  {
    name: "Stadium Lawn",
    img: "https://i.imgur.com/KQVkzBD.jpeg",
    lat: 29.651412,  // Paste your latitude
    lon: -82.349339  // Paste your longitude
  },
  {
    name: "Bat House/F&F Yard",
    img: "https://i.imgur.com/EhXAuLv.jpeg",
    lat: 29.644922,  // Paste your latitude
    lon: -82.362956  // Paste your longitude
  },
  {
    name: "Hume Field",
    img: "https://i.imgur.com/qrpyoFs.jpeg",
    lat: 29.643694,  // Paste your latitude
    lon: -82.353384  // Paste your longitude
  },
  {
    name: "Plaza of Americas",
    img: "https://i.imgur.com/wsZOtv2.jpeg",
    lat: 29.650427,  // Paste your latitude
    lon: -82.342770  // Paste your longitude
  },
  {
    name: "Lake Alice Field",
    img: "https://i.imgur.com/Q7z0FVY.jpeg",
    lat: 29.644767,  // Paste your latitude
    lon: -82.356223  // Paste your longitude
  },
  {
    name: "Maguire Field",
    img: "https://i.imgur.com/mfg3fqu.jpeg",
    lat: 29.641161,  // Paste your latitude
    lon: -82.369045  // Paste your longitude
  },
  {
    name: "Reitz Union Bookstore",
    img: "https://i.imgur.com/rwZmUuW.jpeg",
    lat: 29.645389,  // Paste your latitude
    lon: -82.347803  // Paste your longitude
  },
  {
    name: "Norman Hall Path",
    img: "https://i.imgur.com/Pt6CRa0.jpeg",
    lat: 29.646089,  // Paste your latitude
    lon: -82.338354  // Paste your longitude
  },
  {
    name: "Parking Garage 5&14",
    img: "https://i.imgur.com/vnhuNwV.jpeg",
    lat: 29.643552,  // Paste your latitude
    lon: -82.351117  // Paste your longitude
  },
  {
    name: "Normal Library",
    img: "https://i.imgur.com/BJUJtj3.jpeg",
    lat: 29.646399,  // Paste your latitude
    lon: -82.337794  // Paste your longitude
  },
  {
    name: "13th and Museum Intersection",
    img: "https://i.imgur.com/Y9i2uW.jpeg",
    lat: 29.644930,  // Paste your latitude
    lon: -82.339115  // Paste your longitude
  },
  {
    name: "Honors Village Crossing",
    img: "https://i.imgur.com/InOpJUp.jpeg",
    lat: 29.644808,  // Paste your latitude
    lon: -82.340898  // Paste your longitude
  },
  {
    name: "Broward Melt Lab",
    img: "https://i.imgur.com/YNnLlYW.jpeg",
    lat: 29.647110,  // Paste your latitude
    lon: -82.341461  // Paste your longitude
  }

    // <-- ADD YOUR NEW LOCATIONS HERE
    // Don't forget the comma after the closing }
];
// ######################################################################
// ##### End of Location Database #####
// ######################################################################


// --- Game State Variables ---
let map;
let guessMarker;
let answerMarker;
let answerLine;
let currentRound = 0;
let totalScore = 0;
let rounds = []; // This will hold the 5 locations for the game
const MAX_ROUNDS = 5;
const UF_CAMPUS_CENTER = [29.6483, -82.3494]; // Center of campus

// --- DOM Elements ---
const guessButton = document.getElementById('guess-button');
const nextRoundButton = document.getElementById('next-round-button');
const guessImage = document.getElementById('guess-image');
const roundDisplay = document.getElementById('round-display');
const scoreDisplay = document.getElementById('score-display');
const resultModal = document.getElementById('result-modal');
const resultScore = document.getElementById('result-score');
const resultDistance = document.getElementById('result-distance');
const resultLocationName = document.getElementById('result-location-name');
const finalScoreScreen = document.getElementById('final-score-screen');
const finalScoreText = document.getElementById('final-score-text');
const playAgainButton = document.getElementById('play-again-button');


// --- Game Logic ---

/**
 * Initializes the map using Leaflet
 */
function initMap() {
    map = L.map('map').setView(UF_CAMPUS_CENTER, 15); // Centered on UF, zoom level 15
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    // Event listener for map clicks
    map.on('click', onMapClick);
}

/**
 * Handles the user clicking on the map to place a guess
 */
function onMapClick(e) {
    if (guessMarker) {
        map.removeLayer(guessMarker); // Remove old marker
    }
    guessMarker = L.marker(e.latlng).addTo(map); // Add new marker
    guessButton.disabled = false;
    guessButton.textContent = "Make Your Guess!";
}

/**
 * Starts a new game
 */
function startGame() {
    // Reset state
    currentRound = 0;
    totalScore = 0;
    rounds = shuffleArray(gameLocations).slice(0, MAX_ROUNDS); // Get 5 random locations
    
    // Check if we have enough locations
    if (rounds.length < MAX_ROUNDS) {
        alert(`Warning: Not enough locations! Please add at least ${MAX_ROUNDS} locations to the database.`);
        return;
    }
    
    updateScoreboard();
    resetUI();
    loadNextRound();
}

/**
 * Loads the next round or ends the game
 */
function loadNextRound() {
    if (currentRound >= MAX_ROUNDS) {
        endGame();
        return;
    }

    currentRound++;
    resetRound();
    
    // Load new round data
    const currentLocation = rounds[currentRound - 1];
    guessImage.src = currentLocation.img;
    roundDisplay.textContent = `Round: ${currentRound} / ${MAX_ROUNDS}`;
}

/**
 * Resets the UI elements for a new round
 */
function resetUI() {
    finalScoreScreen.style.display = 'none';
    nextRoundButton.style.display = 'none';
    guessButton.style.display = 'block';
    guessButton.disabled = true;
    guessButton.textContent = 'Place your pin on the map';
    resultModal.style.display = 'none';
}

/**
 * Resets the map state for a new round
 */
function resetRound() {
    if (guessMarker) map.removeLayer(guessMarker);
    if (answerMarker) map.removeLayer(answerMarker);
    if (answerLine) map.removeLayer(answerLine);
    
    guessMarker = null;
    answerMarker = null;
    answerLine = null;
    
    map.setView(UF_CAMPUS_CENTER, 15);
    resetUI();
}

/**
 * Handles the user clicking the "Guess" button
 */
function handleGuess() {
    if (!guessMarker) return;

    // 1. Get answer and guess coordinates
    const answer = rounds[currentRound - 1];
    const answerLatLng = [answer.lat, answer.lon];
    const guessLatLng = guessMarker.getLatLng();

    // 2. Calculate distance
    const distance = getDistanceInKm(answerLatLng, [guessLatLng.lat, guessLatLng.lng]);
    
    // 3. Calculate score
    const score = calculateScore(distance);
    totalScore += score;
    updateScoreboard();

    // 4. Show results on map
    showAnswer(answerLatLng, guessLatLng, distance, score, answer.name);
}

/**
 * Updates the score display
 */
function updateScoreboard() {
    scoreDisplay.textContent = `Total Score: ${totalScore.toLocaleString()}`;
}

/**
 * Displays the answer on the map and in the modal
 */
function showAnswer(answerLatLng, guessLatLng, distance, score, name) {
    // Create a red marker for the correct answer
    answerMarker = L.marker(answerLatLng, {
        icon: L.icon({
            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-red.png',
            shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
            iconSize: [25, 41],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41]
        })
    }).addTo(map);

    // Draw a line between guess and answer
    answerLine = L.polyline([answerLatLng, guessLatLng], { color: '#FA4616' }).addTo(map);

    // Fit map to show both points
    map.fitBounds([answerLatLng, guessLatLng], { padding: [50, 50] });

    // Show result modal
    resultScore.textContent = `${score.toLocaleString()} Points!`;
    resultDistance.textContent = `You were ${distance.toFixed(2)} km away.`;
    resultLocationName.textContent = `The location was: ${name}`;
    resultModal.style.display = 'block';

    // Toggle buttons
    guessButton.style.display = 'none';
    nextRoundButton.style.display = 'block';
}

/**
 * Ends the game and shows the final score
 */
function endGame() {
    finalScoreText.textContent = `Your Final Score: ${totalScore.toLocaleString()}`;
    finalScoreScreen.style.display = 'flex';
}

// --- Utility Functions ---

/**
 * Calculates the score based on distance.
 * Max 5000 points. 0 points if 2km or further.
 */
function calculateScore(distanceKm) {
    const MAX_POINTS = 5000;
    const MAX_DISTANCE_KM = 2; // Max distance for any points on campus

    if (distanceKm < 0.015) { // ~15 meters, basically perfect
        return MAX_POINTS;
    }
    
    let score = MAX_POINTS * (1 - (distanceKm / MAX_DISTANCE_KM));
    return Math.max(0, Math.round(score));
}

/**
 * Haversine formula to calculate distance between two lat/lon points in km
 */
function getDistanceInKm(latlng1, latlng2) {
    const R = 6371; // Radius of the Earth in km
    const dLat = deg2rad(latlng2[0] - latlng1[0]);
    const dLon = deg2rad(latlng2[1] - latlng1[1]);
    const a =
        Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(deg2rad(latlng1[0])) * Math.cos(deg2rad(latlng2[0])) *
        Math.sin(dLon / 2) * Math.sin(dLon / 2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    const d = R * c; // Distance in km
    return d;
}

function deg2rad(deg) {
    return deg * (Math.PI / 180);
}

/**
 * Shuffles an array randomly (Fisher-Yates shuffle)
 */
function shuffleArray(array) {
    let currentIndex = array.length, randomIndex;
    while (currentIndex != 0) {
        randomIndex = Math.floor(Math.random() * currentIndex);
        currentIndex--;
        [array[currentIndex], array[randomIndex]] = [
            array[randomIndex], array[currentIndex]];
    }
    return array;
}


// --- Event Listeners ---
guessButton.addEventListener('click', handleGuess);
nextRoundButton.addEventListener('click', loadNextRound);
playAgainButton.addEventListener('click', startGame);

// --- Start the game on page load ---
document.addEventListener('DOMContentLoaded', () => {
    initMap();
    startGame();
});

    </script>
</body>
</html>
