# UrbanCanvasAI
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Urban Canvas AI</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            padding: 0;
            overflow: hidden; /* Prevent scrollbars */
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: #f0f2f5;
        }
        #map {
            flex-grow: 1;
            width: 100%;
            height: 100%;
            border-radius: 0.75rem; /* rounded-xl */
            overflow: hidden;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05); /* shadow-xl */
        }
        .map-container {
            position: relative;
            flex-grow: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }
        .control-panel {
            position: absolute;
            top: 1rem;
            left: 1rem;
            z-index: 10;
            background-color: rgba(255, 255, 255, 0.9);
            padding: 1rem;
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            max-width: 300px;
        }
        .memory-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.6);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 100;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        .memory-modal.show {
            opacity: 1;
            visibility: visible;
        }
        .memory-content {
            background-color: #fff;
            padding: 2rem;
            border-radius: 0.75rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
            max-width: 500px;
            width: 90%;
            max-height: 90vh;
            overflow-y: auto;
            position: relative;
        }
        .loading-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 1.5rem;
            z-index: 200;
            opacity: 0;
            visibility: hidden;
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        .loading-overlay.show {
            opacity: 1;
            visibility: visible;
        }
        .spinner {
            border: 4px solid rgba(255, 255, 255, 0.3);
            border-top: 4px solid #fff;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin-right: 1rem;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Custom Map Styling Simulation (desaturated, vintage feel) */
        /* This is a visual effect applied to the overall map container,
           as direct styling of Google Maps base layers is done via Map IDs. */
        #map-container-wrapper {
            filter: grayscale(30%) saturate(80%) sepia(10%);
            transition: filter 0.5s ease;
        }
        #map-container-wrapper.active-memory {
            filter: grayscale(0%) saturate(100%) sepia(0%); /* Reset filter when memory is active */
        }

        /* Custom Marker Styling */
        .custom-marker {
            width: 24px;
            height: 24px;
            background-color: #ef4444; /* red-500 */
            border-radius: 50%;
            border: 3px solid #fca5a5; /* red-300 */
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            font-weight: bold;
            font-size: 0.8rem;
            cursor: pointer;
            transition: transform 0.2s ease, background-color 0.2s ease;
        }
        .custom-marker:hover {
            transform: scale(1.2);
            background-color: #dc2626; /* red-600 */
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">
    <div class="loading-overlay" id="loadingOverlay">
        <div class="spinner"></div>
        <span>Loading...</span>
    </div>

    <header class="bg-gradient-to-r from-blue-600 to-purple-700 text-white p-4 shadow-lg">
        <div class="container mx-auto flex justify-between items-center">
            <h1 class="text-3xl font-bold rounded-md px-3 py-1 bg-white bg-opacity-20">Urban Canvas AI</h1>
            <nav>
                <ul class="flex space-x-6">
                    <li><a href="#" class="hover:text-blue-200 transition duration-300">Home</a></li>
                    <li><a href="#" class="hover:text-blue-200 transition duration-300">About</a></li>
                    <li><a href="#" class="hover:text-blue-200 transition duration-300">Contribute</a></li>
                    <li><a href="#" class="hover:text-blue-200 transition duration-300">Contact</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <main class="flex-grow flex flex-col md:flex-row p-4 gap-4">
        <div id="map-container-wrapper" class="map-container flex-grow rounded-xl shadow-xl">
            <div id="map"></div>
            <div class="control-panel">
                <h2 class="text-xl font-semibold mb-3 text-gray-700">Explore Memories</h2>
                <p class="text-sm text-gray-600 mb-4">Click on the map to add a new memory, or click on an existing pin to view its story!</p>
                <div class="flex items-center space-x-2 mb-4">
                    <label for="yearFilter" class="text-sm font-medium">Filter by Year:</label>
                    <input type="range" id="yearFilter" min="2005" max="2025" value="2025" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer">
                    <span id="currentYear" class="text-sm font-semibold">2025</span>
                </div>
                <button id="addMemoryBtn" class="w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg transition duration-300 shadow-md mb-2">
                    Add New Memory
                </button>
                <!-- Conceptual Google Home Integration Button -->
                <button id="simulateGoogleHomeBtn" class="w-full bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-4 rounded-lg transition duration-300 shadow-md">
                    Simulate Google Home Command
                </button>
                <p class="text-xs text-gray-500 mt-2">Your User ID: <span id="userIdDisplay" class="font-mono text-blue-700 break-all">Loading...</span></p>
            </div>
        </div>
    </main>

    <!-- Add Memory Modal -->
    <div id="addMemoryModal" class="memory-modal">
        <div class="memory-content">
            <h3 class="text-2xl font-bold mb-4 text-gray-800">Add a New Memory</h3>
            <form id="addMemoryForm" class="space-y-4">
                <div>
                    <label for="memoryLat" class="block text-sm font-medium text-gray-700">Latitude</label>
                    <input type="text" id="memoryLat" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2 bg-gray-50" readonly>
                </div>
                <div>
                    <label for="memoryLng" class="block text-sm font-medium text-gray-700">Longitude</label>
                    <input type="text" id="memoryLng" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2 bg-gray-50" readonly>
                </div>
                <div>
                    <label for="photoUrl" class="block text-sm font-medium text-gray-700">Photo URL (Placeholder)</label>
                    <input type="url" id="photoUrl" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2" placeholder="e.g., https://placehold.co/400x300">
                </div>
                <div>
                    <label for="keywords" class="block text-sm font-medium text-gray-700">Keywords (e.g., "Rainy evening, Best phở, Nguyễn Trãi street")</label>
                    <textarea id="keywords" rows="3" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2" placeholder="Describe the moment..."></textarea>
                </div>
                <div>
                    <label for="memoryDate" class="block text-sm font-medium text-gray-700">Year of Memory</label>
                    <input type="number" id="memoryDate" min="2005" max="2025" value="2015" class="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 p-2">
                </div>
                <div class="flex justify-end space-x-3 mt-6">
                    <button type="button" id="cancelAddMemoryBtn" class="px-4 py-2 bg-gray-300 text-gray-800 rounded-lg hover:bg-gray-400 transition duration-300">Cancel</button>
                    <button type="submit" class="px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600 transition duration-300 shadow-md">Generate Narrative & Save</button>
                </div>
            </form>
        </div>
    </div>

    <!-- View Memory Modal -->
    <div id="viewMemoryModal" class="memory-modal">
        <div class="memory-content">
            <button id="closeViewMemoryBtn" class="absolute top-3 right-3 text-gray-500 hover:text-gray-800 text-2xl font-bold">&times;</button>
            <h3 id="viewMemoryTitle" class="text-2xl font-bold mb-4 text-gray-800">Memory Details</h3>
            <img id="viewMemoryPhoto" src="" alt="Memory Photo" class="w-full h-48 object-cover rounded-md mb-4 shadow-md">
            <p class="text-sm font-medium text-gray-700 mb-1">Keywords:</p>
            <p id="viewMemoryKeywords" class="text-gray-600 mb-4 italic"></p>
            <p class="text-sm font-medium text-gray-700 mb-1">Narrative:</p>
            <p id="viewMemoryNarrative" class="text-gray-800 leading-relaxed mb-4"></p>
            <p id="viewMemoryDate" class="text-sm text-gray-500 text-right"></p>
            <p id="viewMemoryUser" class="text-xs text-gray-400 text-right">Shared by: <span class="font-mono"></span></p>

            <div class="mt-6 flex justify-end space-x-3">
                <button id="thenNowBtn" class="px-4 py-2 bg-purple-500 text-white rounded-lg hover:bg-purple-600 transition duration-300 shadow-md">
                    Then & Now (Conceptual)
                </button>
                <button id="deleteMemoryBtn" class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition duration-300 shadow-md">
                    Delete Memory
                </button>
            </div>
        </div>
    </div>

    <!-- Confirmation Modal -->
    <div id="confirmationModal" class="memory-modal">
        <div class="memory-content">
            <h3 class="text-xl font-bold mb-4 text-gray-800">Confirm Deletion</h3>
            <p class="mb-6">Are you sure you want to delete this memory? This action cannot be undone.</p>
            <div class="flex justify-end space-x-3">
                <button id="cancelDeleteBtn" class="px-4 py-2 bg-gray-300 text-gray-800 rounded-lg hover:bg-gray-400 transition duration-300">Cancel</button>
                <button id="confirmDeleteBtn" class="px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition duration-300 shadow-md">Delete</button>
            </div>
        </div>
    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, setDoc, updateDoc, deleteDoc, onSnapshot, collection, query, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables provided by the Canvas environment
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-urban-canvas-app';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Initialize Firebase
        let app;
        let db;
        let auth;
        let currentUserId = null;
        let map;
        let markers = {}; // Store markers by memory ID
        let currentMemoryIdToDelete = null; // To store the ID of the memory being deleted

        // DOM Elements
        const loadingOverlay = document.getElementById('loadingOverlay');
        const addMemoryModal = document.getElementById('addMemoryModal');
        const viewMemoryModal = document.getElementById('viewMemoryModal');
        const confirmationModal = document.getElementById('confirmationModal');
        const addMemoryForm = document.getElementById('addMemoryForm');
        const addMemoryBtn = document.getElementById('addMemoryBtn');
        const cancelAddMemoryBtn = document.getElementById('cancelAddMemoryBtn');
        const closeViewMemoryBtn = document.getElementById('closeViewMemoryBtn');
        const thenNowBtn = document.getElementById('thenNowBtn');
        const deleteMemoryBtn = document.getElementById('deleteMemoryBtn');
        const cancelDeleteBtn = document.getElementById('cancelDeleteBtn');
        const confirmDeleteBtn = document.getElementById('confirmDeleteBtn');
        const yearFilter = document.getElementById('yearFilter');
        const currentYearSpan = document.getElementById('currentYear');
        const userIdDisplay = document.getElementById('userIdDisplay');
        const mapContainerWrapper = document.getElementById('map-container-wrapper');
        const simulateGoogleHomeBtn = document.getElementById('simulateGoogleHomeBtn'); // New button for Google Home simulation

        // Show loading overlay
        function showLoading(message = 'Loading...') {
            loadingOverlay.querySelector('span').textContent = message;
            loadingOverlay.classList.add('show');
        }

        // Hide loading overlay
        function hideLoading() {
            loadingOverlay.classList.remove('show');
        }

        // Function to initialize Firebase and authenticate
        async function initializeFirebase() {
            showLoading('Initializing Firebase...');
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Authenticate user
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                // Listen for auth state changes
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        currentUserId = user.uid;
                        userIdDisplay.textContent = currentUserId;
                        console.log("Firebase initialized. User ID:", currentUserId);
                        initMap(); // Initialize map after Firebase and auth are ready
                        setupMemoryListener(); // Start listening for memories
                        setupGoogleHomeCommandListener(); // Setup listener for conceptual Google Home commands
                    } else {
                        console.log("No user signed in.");
                        currentUserId = crypto.randomUUID(); // Use a random ID if not authenticated
                        userIdDisplay.textContent = currentUserId + " (Anonymous)";
                        initMap(); // Initialize map even if anonymous
                        setupMemoryListener(); // Start listening for memories
                        setupGoogleHomeCommandListener(); // Setup listener for conceptual Google Home commands
                    }
                    hideLoading();
                });

            } catch (error) {
                console.error("Error initializing Firebase:", error);
                hideLoading();
                // Display error message to user if possible
                alert("Failed to initialize the application. Please try again later.");
            }
        }

        // Initialize Google Map
        async function initMap() {
            if (!db || !auth) {
                console.warn("Firebase not ready yet. Retrying map initialization...");
                return;
            }

            // The Google Maps API key is automatically provided by the Canvas environment.
            // Do NOT set it explicitly here.
            const googleMapsApiKey = "";

            const script = document.createElement('script');
            // IMPORTANT: Replace 'YOUR_MAP_ID' with your actual Google Maps Map ID.
            // This Map ID must be configured in your Google Cloud Project for Photorealistic 3D Tiles.
            // If you get 'ApiProjectMapError', it's likely due to a missing or incorrect Map ID,
            // or the Maps JavaScript API / Photorealistic 3D Tiles API not being enabled in your project.
            script.src = `https://maps.googleapis.com/maps/api/js?key=${googleMapsApiKey}&map_ids=YOUR_MAP_ID&libraries=places&callback=initMapCallback`;
            script.async = true;
            script.defer = true;
            document.head.appendChild(script);
        }

        // Callback function for Google Maps API script load
        window.initMapCallback = function() {
            const HCMC_LOCATION = { lat: 10.7769, lng: 106.7009 }; // Ho Chi Minh City Opera House area

            map = new google.maps.Map(document.getElementById('map'), {
                center: HCMC_LOCATION,
                zoom: 17, // Zoom level for 3D view
                heading: 45, // Tilt for 3D view
                tilt: 67.5, // High tilt for immersive 3D
                // IMPORTANT: Ensure 'YOUR_MAP_ID' is replaced with a valid Map ID configured for Photorealistic 3D Tiles.
                mapId: 'YOUR_MAP_ID',
                disableDefaultUI: true, // Hide default UI controls
                gestureHandling: "greedy", // Allow easy panning/zooming
            });

            // Add click listener to map to add new memories
            map.addListener('click', (event) => {
                const lat = event.latLng.lat();
                const lng = event.latLng.lng();
                document.getElementById('memoryLat').value = lat.toFixed(6);
                document.getElementById('memoryLng').value = lng.toFixed(6);
                addMemoryModal.classList.add('show');
            });

            console.log("Google Map initialized.");
        };

        // Set up real-time listener for memories from Firestore
        function setupMemoryListener() {
            if (!db) {
                console.error("Firestore not initialized for memory listener.");
                return;
            }

            const memoriesCollectionRef = collection(db, `artifacts/${appId}/public/data/memories`);
            onSnapshot(memoriesCollectionRef, (snapshot) => {
                // Clear existing markers
                Object.values(markers).forEach(marker => {
                    if (marker.setMap) { // Check if it's a Google Maps Marker
                        marker.setMap(null);
                    } else if (marker.remove) { // Check if it's a custom DOM marker
                        marker.remove();
                    }
                });
                markers = {}; // Reset markers object

                const currentFilterYear = parseInt(yearFilter.value);

                snapshot.forEach((doc) => {
                    const memory = doc.data();
                    const memoryId = doc.id;
                    const memoryYear = new Date(memory.timestamp).getFullYear();

                    if (memoryYear <= currentFilterYear) {
                        addMemoryMarker(memoryId, memory);
                    }
                });
                console.log("Memories updated from Firestore.");
            }, (error) => {
                console.error("Error fetching memories:", error);
            });
        }

        // Add a memory marker to the map
        function addMemoryMarker(memoryId, memory) {
            if (!map) {
                console.error("Map not initialized when trying to add marker.");
                return;
            }

            const markerDiv = document.createElement('div');
            markerDiv.className = 'custom-marker';
            markerDiv.textContent = 'M'; // 'M' for Memory
            markerDiv.title = `Memory from ${new Date(memory.timestamp).getFullYear()}`;

            // Create a custom OverlayView for the marker
            class CustomMarkerOverlay extends google.maps.OverlayView {
                constructor(position, content, map, memoryId, memoryData) {
                    super();
                    this.position = position;
                    this.content = content;
                    this.map = map;
                    this.memoryId = memoryId;
                    this.memoryData = memoryData;
                    this.div = null; // This will hold the DOM element for the marker
                    this.setMap(map);
                }

                onAdd() {
                    this.div = document.createElement('div');
                    this.div.style.position = 'absolute';
                    this.div.appendChild(this.content);

                    // Add click listener to the marker
                    this.div.addEventListener('click', () => {
                        viewMemory(this.memoryId, this.memoryData);
                    });

                    const panes = this.getPanes();
                    panes.overlayMouseTarget.appendChild(this.div);
                }

                draw() {
                    const overlayProjection = this.getProjection();
                    if (!overlayProjection) return;

                    const point = overlayProjection.fromLatLngToDivPixel(this.position);
                    if (point) {
                        this.div.style.left = point.x - (this.div.clientWidth / 2) + 'px';
                        this.div.style.top = point.y - this.div.clientHeight + 'px'; // Position above the point
                    }
                }

                onRemove() {
                    if (this.div) {
                        this.div.parentNode.removeChild(this.div);
                        this.div = null;
                    }
                }

                getPosition() {
                    return this.position;
                }
            }

            const position = new google.maps.LatLng(memory.latitude, memory.longitude);
            const customOverlay = new CustomMarkerOverlay(position, markerDiv, map, memoryId, memory);
            markers[memoryId] = customOverlay; // Store the custom overlay

            console.log(`Marker added for memory ID: ${memoryId}`);
        }

        // Handle Add Memory Form Submission
        addMemoryForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            showLoading('Generating narrative...');

            const lat = parseFloat(document.getElementById('memoryLat').value);
            const lng = parseFloat(document.getElementById('memoryLng').value);
            const photoUrl = document.getElementById('photoUrl').value || 'https://placehold.co/400x300/e0e0e0/000000?text=No+Photo';
            const keywords = document.getElementById('keywords').value;
            const memoryYear = parseInt(document.getElementById('memoryDate').value);

            if (!keywords) {
                alert("Please provide some keywords for the memory.");
                hideLoading();
                return;
            }

            try {
                // Call Gemini API to generate narrative
                const narrative = await generateNarrative(keywords);

                const newMemory = {
                    userId: currentUserId,
                    latitude: lat,
                    longitude: lng,
                    photoUrl: photoUrl,
                    keywords: keywords,
                    narrative: narrative,
                    timestamp: new Date(memoryYear, 0, 1).toISOString(), // Set to Jan 1st of the selected year
                    createdAt: new Date().toISOString() // Actual creation timestamp
                };

                // Save memory to Firestore
                await addDoc(collection(db, `artifacts/${appId}/public/data/memories`), newMemory);

                addMemoryModal.classList.remove('show');
                addMemoryForm.reset();
                hideLoading();
                console.log("Memory added successfully!");

            } catch (error) {
                console.error("Error adding memory or generating narrative:", error);
                hideLoading();
                alert("Failed to add memory. Please try again. Error: " + error.message);
            }
        });

        // Function to call Gemini API for narrative generation
        async function generateNarrative(keywords) {
            // The Gemini API key is automatically provided by the Canvas environment.
            // Do NOT set it explicitly here.
            const geminiApiKey = "";

            const prompt = `Craft a short, poetic narrative (around 50-70 words) based on these keywords about a city memory: "${keywords}". Focus on evoking emotion and atmosphere. Example: "On a rain-slicked evening in 2010, the neon lights of Nguyễn Trãi street blurred into watercolor. Here, at a small, steamy stall, was a bowl of phở so perfect it became a memory, a warm anchor in the heart of a bustling Saigon."`;

            let chatHistory = [];
            chatHistory.push({ role: "user", parts: [{ text: prompt }] });

            const payload = { contents: chatHistory };
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${geminiApiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();

                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    return result.candidates[0].content.parts[0].text;
                } else {
                    console.error("Gemini API response structure unexpected:", result);
                    return "Could not generate narrative. Please try again.";
                }
            } catch (error) {
                console.error("Error calling Gemini API:", error);
                return "Failed to generate narrative due to API error.";
            }
        }

        // Display memory details in a modal
        function viewMemory(memoryId, memory) {
            // Move camera to memory location
            if (map && memory.latitude && memory.longitude) {
                map.moveCamera({
                    center: { lat: memory.latitude, lng: memory.longitude },
                    zoom: 19, // Zoom in closer
                    heading: 0, // Reset heading
                    tilt: 67.5, // Maintain high tilt
                    duration: 2000, // 2 seconds animation
                });
            }

            document.getElementById('viewMemoryPhoto').src = memory.photoUrl;
            document.getElementById('viewMemoryPhoto').onerror = () => {
                document.getElementById('viewMemoryPhoto').src = 'https://placehold.co/400x300/e0e0e0/000000?text=Image+Not+Found';
            };
            document.getElementById('viewMemoryKeywords').textContent = memory.keywords;
            document.getElementById('viewMemoryNarrative').textContent = memory.narrative;
            document.getElementById('viewMemoryDate').textContent = `Year: ${new Date(memory.timestamp).getFullYear()}`;
            document.getElementById('viewMemoryUser').querySelector('span').textContent = memory.userId;

            // Set data attributes for deletion
            viewMemoryModal.setAttribute('data-memory-id', memoryId);
            viewMemoryModal.setAttribute('data-memory-owner-id', memory.userId);

            // Show/hide delete button based on ownership
            if (currentUserId === memory.userId) {
                deleteMemoryBtn.style.display = 'inline-block';
            } else {
                deleteMemoryBtn.style.display = 'none';
            }

            viewMemoryModal.classList.add('show');
            mapContainerWrapper.classList.add('active-memory'); // Remove map filter
        }

        // Event Listeners for Modals and Controls
        addMemoryBtn.addEventListener('click', () => {
            // Reset form fields
            addMemoryForm.reset();
            // Clear any previous coordinates if the modal is opened manually
            document.getElementById('memoryLat').value = '';
            document.getElementById('memoryLng').value = '';
            addMemoryModal.classList.add('show');
        });

        cancelAddMemoryBtn.addEventListener('click', () => {
            addMemoryModal.classList.remove('show');
        });

        closeViewMemoryBtn.addEventListener('click', () => {
            viewMemoryModal.classList.remove('show');
            mapContainerWrapper.classList.remove('active-memory'); // Reapply map filter
        });

        // "Then & Now" button (Conceptual)
        thenNowBtn.addEventListener('click', () => {
            alert("This 'Then & Now' feature would ideally show historical Street View imagery. This is a conceptual feature for this demo.");
        });

        // Delete Memory Button Click
        deleteMemoryBtn.addEventListener('click', () => {
            currentMemoryIdToDelete = viewMemoryModal.getAttribute('data-memory-id');
            const memoryOwnerId = viewMemoryModal.getAttribute('data-memory-owner-id');

            if (currentMemoryIdToDelete && currentUserId === memoryOwnerId) {
                viewMemoryModal.classList.remove('show'); // Hide view modal
                confirmationModal.classList.add('show'); // Show confirmation modal
            } else {
                alert("You are not authorized to delete this memory.");
            }
        });

        // Confirm Delete Button Click
        confirmDeleteBtn.addEventListener('click', async () => {
            if (currentMemoryIdToDelete) {
                showLoading('Deleting memory...');
                try {
                    await deleteDoc(doc(db, `artifacts/${appId}/public/data/memories`, currentMemoryIdToDelete));
                    console.log("Memory deleted successfully!");
                    confirmationModal.classList.remove('show');
                    hideLoading();
                    currentMemoryIdToDelete = null; // Reset
                } catch (error) {
                    console.error("Error deleting memory:", error);
                    hideLoading();
                    alert("Failed to delete memory. Error: " + error.message);
                }
            }
        });

        // Cancel Delete Button Click
        cancelDeleteBtn.addEventListener('click', () => {
            confirmationModal.classList.remove('show');
            currentMemoryIdToDelete = null;
        });

        // Year Filter Slider
        yearFilter.addEventListener('input', (event) => {
            const selectedYear = event.target.value;
            currentYearSpan.textContent = selectedYear;
            // Re-render markers based on the new filter
            setupMemoryListener(); // This will re-fetch and filter markers
        });

        /*
         * Google Home Integration (Conceptual)
         *
         * A real Google Home integration would involve a backend fulfillment service
         * that listens for Google Assistant commands (Smart Home Action API).
         * This backend would then update a shared state (e.g., in Firestore)
         * that the client-side app listens to.
         *
         * For this client-side demo, we simulate this by having a button
         * that directly updates a conceptual 'google_home_commands' collection
         * in Firestore. The app will listen to this collection and react.
         */

        // Listen for conceptual Google Home commands from Firestore
        function setupGoogleHomeCommandListener() {
            if (!db) {
                console.error("Firestore not initialized for Google Home command listener.");
                return;
            }

            // We'll use a single document for simplicity to represent the latest command
            const commandDocRef = doc(db, `artifacts/${appId}/public/data/google_home_commands`, 'latest_command');

            onSnapshot(commandDocRef, (docSnapshot) => {
                if (docSnapshot.exists()) {
                    const command = docSnapshot.data();
                    console.log("Received conceptual Google Home command:", command);

                    // Example: If a command to change year filter is received
                    if (command.type === 'setYearFilter' && command.year) {
                        const targetYear = parseInt(command.year);
                        if (!isNaN(targetYear) && targetYear >= 2005 && targetYear <= 2025) {
                            yearFilter.value = targetYear;
                            currentYearSpan.textContent = targetYear;
                            setupMemoryListener(); // Re-filter memories
                            alert(`Google Home: Filter set to ${targetYear}!`);
                        }
                    }
                    // Add more command types as needed (e.g., move camera to location)
                }
            }, (error) => {
                console.error("Error listening for Google Home commands:", error);
            });
        }

        // Simulate Google Home Command Button
        simulateGoogleHomeBtn.addEventListener('click', async () => {
            // In a real scenario, this would be triggered by Google Assistant calling your backend.
            // Here, we simulate by asking the user for input.
            const yearInput = prompt("Simulate Google Home: Enter a year to filter memories (e.g., 2010):");
            const targetYear = parseInt(yearInput);

            if (!isNaN(targetYear) && targetYear >= 2005 && targetYear <= 2025) {
                showLoading(`Simulating Google Home command: Set filter to ${targetYear}...`);
                try {
                    // Update a Firestore document that the app is listening to
                    await setDoc(doc(db, `artifacts/${appId}/public/data/google_home_commands`, 'latest_command'), {
                        type: 'setYearFilter',
                        year: targetYear,
                        timestamp: new Date().toISOString()
                    }, { merge: true }); // Use merge to update existing fields without overwriting others
                    hideLoading();
                    console.log(`Simulated Google Home command sent for year: ${targetYear}`);
                } catch (error) {
                    console.error("Error simulating Google Home command:", error);
                    hideLoading();
                    alert("Failed to simulate Google Home command.");
                }
            } else if (yearInput !== null) { // If user didn't cancel
                alert("Invalid year. Please enter a year between 2005 and 2025.");
            }
        });

        // Initialize Firebase when the window loads
        window.onload = initializeFirebase;
    </script>
</body>
</html>

Urban Canvas AI turns Google's map into a time machine. It's an immersive app where you can fly through a city's 3D landscape and discover 20 years of crowd-sourced memories—first dates, lost landmarks, personal stories.  We use AI to help anyone instantly craft a beautiful narrative from just an old photo.
The Concept: A platform that transforms Google Maps into a 4D time capsule of human experience. We're not just mapping places; we're mapping the memories, stories, and emotions tied to them over the past 20 years. It’s a collaborative, living atlas of personal history, enhanced with AI and immersive 3D.
![IMG_1242](https://github.com/user-attachments/assets/18284b0b-5fd3-4c43-9eb9-c77f88a1999b)

Imagine looking at a 3D model of your current location in HCMC, like the area around the Opera House. You could filter by "2008" and see ghostly, glowing pins appear on the map. Tapping one reveals a photo of a couple on their first date at a now-closed cafe, with an AI-generated narrative that captures the feeling of that moment.
![IMG_1243](https://github.com/user-attachments/assets/1cc05694-c562-4ffa-9d36-f5f2c813f9bb)

Primary Categories:

Art of the Map: The map is our canvas for emotional data visualization.

Travel: It offers tourists and locals an authentic, profound way to discover a city's soul.

AI: We're using AI not as a gimmick, but to fundamentally enhance storytelling.

Immersive: We will use 3D to make exploring these memories feel like a cinematic journey.

Building the Winning Features (Based on Judging Criteria)

1. The Immersive Canvas (Immersive & Art of the Map)
![IMG_1249](https://github.com/user-attachments/assets/6be37755-d662-4280-a61d-e57bbc116790)

To create a stunning user experience and visualization, we'll build on a foundation of Google's most immersive products.

Photorealistic 3D Tiles: The entire experience will be built on the Maps JavaScript API's 3D Tiles. When a user explores, they're not panning a flat map; they're flying through a photorealistic 3D model of the city.

Cinematic Camera: Clicking a memory pin triggers a smooth, cinematic flight to that location using map.moveCamera(), swooping down to the exact building or spot. For parks or landmarks, we can trigger the Aerial View API for a breathtaking establishing shot.
![IMG_1244](https://github.com/user-attachments/assets/d657b787-339b-4fef-b2c9-bf0b7865fb2b)
![IMG_1245](https://github.com/user-attachments/assets/e7dc2c6d-4b7f-421b-bbff-6f360ffe9892)

Custom Styling: The base map will use custom styling to evoke a sense of nostalgia—slightly desaturated, with artistic fonts, inspired by vintage cartography like the Terrain Highlighting winner.

2. The AI Memory Weaver (AI & Content)

To make content creation seamless and beautiful, we integrate AI at the core. This addresses technical execution.

Gemini-Powered Narratives: A user can drop a pin, upload a photo from 2010, and provide a few simple keywords like "Rainy evening," "Best phở of my life," "Nguyễn Trãi street." The Gemini model via the Places API will analyze the photo, understand the location's context, and draft a short, poetic narrative.

![IMG_1246](https://github.com/user-attachments/assets/b09e36f0-1a5f-408f-8458-e26d2ed8cb22)

Example Output: "On a rain-slicked evening in 2010, the neon lights of Nguyễn Trãi street blurred into watercolor. Here, at a small, steamy stall, was a bowl of phở so perfect it became a memory, a warm anchor in the heart of a bustling Saigon."

Thematic Clustering: Our backend AI will analyze all memories and automatically tag them with themes ("First Dates," "Street Food Discoveries," "Lost Landmarks"). This allows users to explore the map thematically, providing deep, insightful content visualization.

3. The Time-Traveler View (Travel & Functionality)

This is the core feature that directly celebrates the 20-year anniversary and drives the purpose of the app.

Historical Street View Integration: When viewing a memory, if the date is old enough, a "Then & Now" button appears. This opens a split-screen view showing the current Street View next to the historical Street View imagery from the time of the memory. This is a powerful, direct use of Google Maps' historical data archive.
![IMG_1248](https://github.com/user-attachments/assets/20c9fa3c-6657-43ab-952a-a0e83868ee3a)
![IMG_1249](https://github.com/user-attachments/assets/ac9ce929-a97c-4402-9912-619fa9183112)

Data-Driven Heatmaps: Using the Deck.gl integration, we'll render a dynamic heatmap showing the emotional pulse of the city—the most "cherished" locations. A timeline slider will animate this heatmap, showing how the city's centers of memory have shifted over 20 years. This offers scalable, region-agnostic functionality.

{
  
  "canisters": {
    "urban_canvas_ai": {
      "type": "custom",
      "candid": "urban_canvas_ai.did",
      "wasm": "urban_canvas_ai.wasm",
      "build": "cargo build --target wasm32-unknown-unknown --release --package urban_canvas_ai",
      "root": "src/urban_canvas_ai",
      "dependencies": []
    }
  },
  "defaults": {
    "build": {
      "packtool": "mops"
    }
  },
  "output_env_file": ".env",
  "version": 1
}

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Urban Canvas AI</title>
  <link rel="stylesheet" href="/dist/output.css">
</head>
<body>
  <div id="root"></div>
  <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js" crossorigin></script>
  <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap&v=quarterly" async defer></script>
  <script type="text/babel">
    const { useState, useEffect, createContext, useContext } = React;

    const MemoryContext = createContext();

    const MemoryProvider = ({ children }) => {
      const [memories, setMemories] = useState([]);
      const [selectedMemory, setSelectedMemory] = useState(null);

      const addMemory = (memory) => {
        setMemories(prev => [...prev, { id: Date.now(), ...memory, lat: 10.7769, lng: 106.7009 }]); // HCMC Opera House
      };

      const selectMemory = (id) => {
        setSelectedMemory(memories.find(m => m.id === id));
      };

      return (
        <MemoryContext.Provider value={{ memories, addMemory, selectMemory, selectedMemory }}>
          {children}
        </MemoryContext.Provider>
      );
    };

    const MemoryPin = ({ memory }) => {
      const { selectMemory } = useContext(MemoryContext);
      return (
        <div
          onClick={() => selectMemory(memory.id)}
          className="absolute bg-blue-500 text-white p-2 rounded-full cursor-pointer hover:bg-blue-600"
          style={{ left: `${(memory.lng + 180) / 360 * 100}%`, top: `${(1 - (memory.lat + 90) / 180) * 100}%` }}
        >
          📍
        </div>
      );
    };

    const MemoryDetail = () => {
      const { selectedMemory } = useContext(MemoryContext);
      const generateNarrative = () => "AI narrative: A memory from " + (selectedMemory?.year || "unknown") + " at " + selectedMemory?.location;

      if (!selectedMemory) return null;
      return (
        <div className="bg-white p-4 rounded-lg shadow-md mt-4">
          <h3 className="text-lg font-semibold">{selectedMemory.location}</h3>
          <p className="text-sm text-gray-500">{selectedMemory.year}</p>
          <p className="mt-2">{generateNarrative()}</p>
          <button onClick={() => setSelectedMemory(null)} className="mt-2 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600">
            Close
          </button>
        </div>
      );
    };

    const App = () => {
      const { memories, addMemory } = useContext(MemoryContext);

      useEffect(() => {
        // Map initialization handled by initMap callback
      }, [addMemory]);

      return (
        <div className="container mx-auto p-4">
          <header className="mb-6">
            <h1 className="text-3xl font-bold text-indigo-600">Urban Canvas AI</h1>
            <button onClick={() => addMemory({ year: prompt("Year?"), location: prompt("Location?") })} className="mt-2 px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600">
              Add Memory
            </button>
          </header>
          <div id="map" style={{ height: "400px", width: "100%", position: "relative" }}>
            {memories.map(memory => <MemoryPin key={memory.id} memory={memory} />)}
          </div>
          <img id="streetView" alt="Historical Street View" className="mt-4" />
          <MemoryDetail />
        </div>
      );
    };

    // Global callback with error handling
    window.initMap = function() {
      try {
        const map = new google.maps.Map(document.getElementById("map"), {
          center: { lat: 10.7769, lng: 106.7009 }, // HCMC Opera House
          zoom: 15,
          mapTypeId: 'terrain',
          tilt: 45, // Enable 3D view
        });

        // Render React app after map initialization
        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<MemoryProvider><App /></MemoryProvider>);

        // Historical Street View
        const streetViewUrl = `https://maps.googleapis.com/maps/api/streetview?size=600x300&location=10.7769,106.7009&key=YOUR_API_KEY`;
        document.getElementById("streetView").src = streetViewUrl;
      } catch (error) {
        console.error("Error initializing map:", error);
        alert("Failed to load Urban Canvas AI. Check console for details.");
      }
    };
  </script>
  <script>
    if (process.env.NODE_ENV !== 'production') {
      const fs = require('fs');
      const postcss = require('postcss');
      const tailwindcss = require('tailwindcss');
      const autoprefixer = require('autoprefixer');
      postcss([tailwindcss, autoprefixer]).process('@tailwind base; @tailwind components; @tailwind utilities;', { from: undefined }).then(result => {
        fs.writeFileSync('dist/output.css', result.css);
      }).catch(err => console.error("Tailwind build error:", err));
    }
  </script>
</body>
</html>
