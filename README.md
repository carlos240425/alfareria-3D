# alfareria-3D
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Diseñador de Alfarería 3D</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Inter font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Three.js CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <!-- OrbitControls for camera manipulation -->
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8; /* Light blue-gray background */
        }
        #app-container {
            max-width: 1200px;
            margin: 2rem auto;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 2rem;
            padding: 1rem;
        }
        #canvas-container {
            width: 100%;
            max-width: 800px; /* Max width for canvas */
            height: 500px;
            background-color: #ffffff;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            overflow: hidden; /* Ensure rounded corners clip canvas */
            display: flex;
            justify-content: center;
            align-items: center;
        }
        canvas {
            display: block;
            width: 100% !important;
            height: 100% !important;
            border-radius: 1rem; /* Apply border-radius to canvas if possible */
        }
        .controls-panel {
            background-color: #ffffff;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba{0, 0, 0, 0.05};
            width: 100%;
            max-width: 800px;
        }
        .control-group {
            margin-bottom: 1.5rem;
        }
        .control-group label {
            display: block;
            font-weight: 600;
            margin-bottom: 0.5rem;
            color: #374151; /* Dark gray text */
        }
        input[type="range"] {
            width: 100%;
            -webkit-appearance: none;
            height: 8px;
            border-radius: 5px;
            background: #e0e7ff; /* Light blue background */
            outline: none;
            transition: opacity .2s;
        }
        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #4f46e5; /* Indigo */
            cursor: pointer;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        }
        input[type="color"] {
            width: 100%;
            height: 40px;
            border-radius: 0.5rem;
            border: 1px solid #d1d5db; /* Light gray border */
            cursor: pointer;
        }
        .btn {
            background-color: #4f46e5; /* Indigo */
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem;
            font-weight: 600;
            transition: background-color 0.3s ease, transform 0.1s ease;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            cursor: pointer;
            border: none;
        }
        .btn:hover {
            background-color: #4338ca; /* Darker indigo on hover */
            transform: translateY(-1px);
        }
        .message-box {
            background-color: #dbeafe; /* Lighter blue for messages */
            color: #1e40af; /* Darker blue for message text */
            padding: 1rem;
            border-radius: 0.5rem;
            margin-top: 1rem;
            width: 100%;
            max-width: 800px;
            text-align: center;
            font-weight: 500;
            display: none; /* Hidden by default */
        }
    </style>
</head>
<body>
    <div id="app-container">
        <h1 class="text-4xl font-bold text-gray-800 mb-6 text-center">Diseña tu Objeto de Alfarería 3D</h1>

        <div id="canvas-container">
            <!-- Three.js will render here -->
        </div>

        <div class="controls-panel">
            <div class="control-group">
                <label for="object-color">Color del Objeto:</label>
                <input type="color" id="object-color" value="#a0522d" class="rounded-lg">
            </div>

            <div class="control-group">
                <label for="height-slider">Altura: <span id="height-value">1.0</span></label>
                <input type="range" id="height-slider" min="0.5" max="3.0" step="0.1" value="1.5">
            </div>

            <div class="control-group">
                <label for="top-radius-slider">Radio Superior: <span id="top-radius-value">0.3</span></label>
                <input type="range" id="top-radius-slider" min="0.1" max="1.0" step="0.05" value="0.3">
            </div>

            <div class="control-group">
                <label for="bottom-radius-slider">Radio Inferior: <span id="bottom-radius-value">0.5</span></label>
                <input type="range" id="bottom-radius-slider" min="0.1" max="1.0" step="0.05" value="0.5">
            </div>

            <button id="print-button" class="btn w-full">Preparar para Imprimir</button>
            <div id="message-box" class="message-box"></div>
        </div>
    </div>

    <script type="module">
        // Import Firebase modules (not used in this basic demo, but useful for future expansion)
        // import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        // import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        // import { getFirestore, doc, setDoc, collection, addDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global variables for Firebase configuration (if used in a real app)
        // const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        // const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        // const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Three.js Scene Setup
        let scene, camera, renderer, object, controls;
        let canvasContainer = document.getElementById('canvas-container');
        let messageBox = document.getElementById('message-box');

        /**
         * Initializes the Three.js scene, camera, renderer, and controls.
         */
        function init() {
            // Scene
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xf0f4f8); // Match background color

            // Camera
            camera = new THREE.PerspectiveCamera(75, canvasContainer.clientWidth / canvasContainer.clientHeight, 0.1, 1000);
            camera.position.set(0, 1.5, 3); // Position the camera to view the object

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(canvasContainer.clientWidth, canvasContainer.clientHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            canvasContainer.appendChild(renderer.domElement);

            // Controls
            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true; // Smooth rotation
            controls.dampingFactor = 0.05;
            controls.screenSpacePanning = false;
            controls.minDistance = 1;
            controls.maxDistance = 10;
            controls.maxPolarAngle = Math.PI / 2; // Prevent camera from going below the ground

            // Lighting
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.8); // Soft ambient light
            scene.add(ambientLight);

            const directionalLight1 = new THREE.DirectionalLight(0xffffff, 0.6);
            directionalLight1.position.set(5, 5, 5).normalize();
            scene.add(directionalLight1);

            const directionalLight2 = new THREE.DirectionalLight(0xffffff, 0.4);
            directionalLight2.position.set(-5, 5, -5).normalize();
            scene.add(directionalLight2);

            // Initial Object
            createPotteryObject();

            // Handle window resizing
            window.addEventListener('resize', onWindowResize, false);

            // Set up initial control values
            document.getElementById('object-color').value = '#A0522D';
            document.getElementById('height-slider').value = 1.5;
            document.getElementById('top-radius-slider').value = 0.3;
            document.getElementById('bottom-radius-slider').value = 0.5;

            document.getElementById('height-value').innerText = 1.5;
            document.getElementById('top-radius-value').innerText = 0.3;
            document.getElementById('bottom-radius-value').innerText = 0.5;
        }

        /**
         * Creates or updates the 3D pottery object based on current control values.
         */
        function createPotteryObject() {
            if (object) {
                scene.remove(object); // Remove previous object if exists
                object.geometry.dispose(); // Dispose previous geometry
                object.material.dispose(); // Dispose previous material
            }

            const color = new THREE.Color(document.getElementById('object-color').value);
            const height = parseFloat(document.getElementById('height-slider').value);
            const topRadius = parseFloat(document.getElementById('top-radius-slider').value);
            const bottomRadius = parseFloat(document.getElementById('bottom-radius-slider').value);

            // Create a custom LatheGeometry for pottery shape
            const points = [];
            points.push(new THREE.Vector2(bottomRadius, -height / 2)); // Bottom right point
            points.push(new THREE.Vector2(bottomRadius, -height / 2 + height * 0.1)); // Slight curve up from bottom
            points.push(new THREE.Vector2(bottomRadius * 0.9, 0)); // Wider mid-point
            points.push(new THREE.Vector2(topRadius * 1.1, height * 0.2)); // Narrower point near top
            points.push(new THREE.Vector2(topRadius, height / 2)); // Top right point

            const geometry = new THREE.LatheGeometry(points, 32); // 32 segments for smoothness
            const material = new THREE.MeshStandardMaterial({ color: color, roughness: 0.8, metalness: 0.1 });

            object = new THREE.Mesh(geometry, material);
            scene.add(object);
        }

        /**
         * Handles window resizing to adjust camera aspect ratio and renderer size.
         */
        function onWindowResize() {
            camera.aspect = canvasContainer.clientWidth / canvasContainer.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(canvasContainer.clientWidth, canvasContainer.clientHeight);
            renderer.setPixelRatio(window.devicePixelRatio); // Ensure pixel ratio is set
        }

        /**
         * Animation loop for rendering the scene.
         */
        function animate() {
            requestAnimationFrame(animate);
            controls.update(); // Only required if controls.enableDamping or controls.autoRotate are set to true
            renderer.render(scene, camera);
        }

        /**
         * Displays a temporary message in the message box.
         * @param {string} msg - The message to display.
         * @param {string} type - 'success' or 'error' (not fully implemented, but for future styling).
         */
        function showMessage(msg, type = 'info') {
            messageBox.innerText = msg;
            messageBox.style.display = 'block';
            // You could add classes for different types (e.g., bg-green-100 for success)
            setTimeout(() => {
                messageBox.style.display = 'none';
            }, 3000); // Hide after 3 seconds
        }

        // Event Listeners for Controls
        document.getElementById('object-color').addEventListener('input', (event) => {
            if (object) {
                object.material.color.set(event.target.value);
            }
        });

        document.getElementById('height-slider').addEventListener('input', (event) => {
            document.getElementById('height-value').innerText = parseFloat(event.target.value).toFixed(1);
            createPotteryObject(); // Recreate object with new dimensions
        });

        document.getElementById('top-radius-slider').addEventListener('input', (event) => {
            document.getElementById('top-radius-value').innerText = parseFloat(event.target.value).toFixed(2);
            createPotteryObject();
        });

        document.getElementById('bottom-radius-slider').addEventListener('input', (event) => {
            document.getElementById('bottom-radius-value').innerText = parseFloat(event.target.value).toFixed(2);
            createPotteryObject();
        });

        document.getElementById('print-button').addEventListener('click', () => {
            const currentColor = document.getElementById('object-color').value;
            const currentHeight = parseFloat(document.getElementById('height-slider').value).toFixed(1);
            const currentTopRadius = parseFloat(document.getElementById('top-radius-slider').value).toFixed(2);
            const currentBottomRadius = parseFloat(document.getElementById('bottom-radius-slider').value).toFixed(2);

            const designDetails = {
                color: currentColor,
                height: currentHeight,
                topRadius: currentTopRadius,
                bottomRadius: currentBottomRadius
            };
            console.log("Detalles del diseño para impresión:", designDetails);
            showMessage("¡Tu diseño está listo para ser 'impreso'!", 'success');
            // In a real application, you would send these details to a backend
            // for STL generation, cost calculation, and order processing.
        });

        // Initialize Three.js when the window loads
        window.onload = function() {
            init();
            animate();
        };

        // Firebase Initialization (Commented out for this demo, but structure shown)
        /*
        let app, db, auth;
        let userId = null;

        const initFirebase = async () => {
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

                auth.onAuthStateChanged((user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("Firebase Authenticated User ID:", userId);
                        // Now you can safely interact with Firestore using userId
                        // For example, if you wanted to save user designs:
                        // saveDesignToFirestore(designDetails);
                    } else {
                        console.log("No user signed in.");
                        userId = crypto.randomUUID(); // Fallback for anonymous users
                        console.log("Using anonymous user ID:", userId);
                    }
                });
            } catch (error) {
                console.error("Error initializing Firebase:", error);
                showMessage("Error al iniciar la aplicación. Por favor, inténtalo de nuevo.", 'error');
            }
        };

        // Call Firebase initialization
        // initFirebase();
        */
    </script>
</body>
</html>
