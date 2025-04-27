<!DOCTYPE html>
<html lang="ca">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cursa de Coets Espacials amb Planeta i Forat Negre Continu - Gràfic Energia Cinètica</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background-color: #000; /* Espai fosc */
            color: #fff;
            font-family: 'Press Start 2P', cursive; /* Font arcade */
            padding: 20px; /* Add some padding */
            box-sizing: border-box;
        }

        h1 {
             text-align: center;
             margin-bottom: 20px;
             font-size: 1.5em; /* Adjust for smaller screens */
        }

        canvas#gameCanvas {
            background-color: #0a0a2a; /* Fons de l'espai */
            display: block;
            border: 2px solid #fff;
            margin-bottom: 20px;
            max-width: 95%; /* Responsive width */
            height: auto; /* Maintain aspect ratio */
        }

         canvas#graphCanvas {
             background-color: #1a1a3a; /* Fons del gràfic */
             display: block;
             border: 2px solid #fff;
             margin-top: 20px;
             max-width: 95%; /* Responsive width */
             height: auto; /* Maintain aspect ratio */
         }


        .controls {
            display: flex;
            flex-direction: column; /* Stack controls vertically */
            align-items: center;
            gap: 15px; /* Space between control groups */
            margin-top: 20px;
            flex-wrap: wrap; /* Allow controls to wrap on small screens */
            justify-content: center;
        }

         .number-of-rockets-control {
             display: flex;
             align-items: center;
             gap: 10px;
             margin-bottom: 10px;
         }

         .number-of-rockets-control label {
             font-size: 0.9em;
         }

         .number-of-rockets-control input[type="number"] {
             font-family: 'Press Start 2P', cursive;
             padding: 5px;
             font-size: 0.9em;
             background-color: #333;
             color: #fff;
             border: 1px solid #fff;
             border-radius: 3px;
             width: 50px; /* Adjust width */
             text-align: center;
         }


        .rocket-names {
            display: flex;
            flex-direction: column; /* Stack name inputs vertically */
            gap: 10px; /* Space between name inputs */
            margin-bottom: 15px;
        }

        .rocket-name-input {
            display: flex;
            align-items: center;
            gap: 5px;
        }

        .rocket-name-input label {
            font-size: 0.8em; /* Smaller label font */
        }

        .rocket-name-input input[type="text"] {
            font-family: 'Press Start 2P', cursive;
            padding: 5px;
            font-size: 0.8em; /* Smaller input font */
            background-color: #333;
            color: #fff;
            border: 1px solid #fff;
            border-radius: 3px;
        }


        button {
            font-family: 'Press Start 2P', cursive;
            padding: 10px 20px;
            font-size: 1em;
            cursor: pointer;
            background-color: #4CAF50; /* Verd */
            color: white;
            border: none;
            border-radius: 5px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
            transition: background-color 0.3s ease, transform 0.1s ease;
        }

        button:hover {
            background-color: #45a049;
            transform: translateY(-2px);
        }

        button:active {
            background-color: #367c39;
            transform: translateY(0);
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
        }

        #messageBox {
            margin-top: 20px;
            font-size: 1.2em;
            text-align: center;
            min-height: 1.5em; /* Reserve space */
        }

        /* Estrella simple amb SVG */
        .star {
            fill: white;
        }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body>

    <h1>Cursa de Coets Espacials amb Planeta i Forat Negre Continu</h1>

    <canvas id="gameCanvas"></canvas>
     <canvas id="graphCanvas"></canvas> <div class="controls">
         <div class="number-of-rockets-control">
             <label for="numRockets">Nombre de Coets:</label>
             <input type="number" id="numRockets" value="3" min="1" max="10"> </div>
        <div class="rocket-names" id="rocketNameInputs">
            </div>
        <button id="startButton">Començar Cursa</button>
    </div>

    <div id="messageBox"></div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const graphCanvas = document.getElementById('graphCanvas'); // Get graph canvas
        const graphCtx = graphCanvas.getContext('2d'); // Get graph context

        const startButton = document.getElementById('startButton');
        const messageBox = document.getElementById('messageBox');
        const rocketNameInputsDiv = document.getElementById('rocketNameInputs');
        const numRocketsInput = document.getElementById('numRockets'); // Get the number input

        let gameRunning = false;
        let rockets = [];
        let numberOfRockets = parseInt(numRocketsInput.value); // Initialize with default value
        // Swapped width and height for horizontal orientation
        const rocketWidth = 40; // Was 20 (height)
        const rocketHeight = 20; // Was 40 (width)
        const finishLineX = 750; // X coordinate of the finish line
        const startLineX = 50; // X coordinate of the start line (vertical line on the left)
        const baseRocketSpeed = 1; // Base speed
        const acceleration = 0.05; // How much speed increases during normal race

        // Planet properties
        const planet = {
            // Initial position (will be updated by orbit)
            x: 0,
            y: 0,
            radius: 30, // Radius of the planet
            gravityRadius: 150, // Radius within which gravity is felt
            gravityStrength: 0.1, // How strong the pull is
            slingshotBoost: 2, // Speed multiplier after slingshot

            // Orbit properties (Circular)
            orbitCenterX: 0,
            orbitCenterY: 0,
            orbitRadius: 100, // Radius of the planet's orbit
            orbitSpeed: 0.01, // How fast the planet orbits
            orbitAngle: 0 // Current angle in its orbit
        };

        // Black Hole properties (modified for continuous movement)
        const blackHole = {
            x: 0,
            y: 0,
            radius: 10, // Radius of the black hole
            gravityRadius: 180, // Radius within which gravity is felt
            gravityStrength: 0.3, // How strong the pull is
            speedX: 0, // Horizontal speed of the black hole
            speedY: 0, // Vertical speed of the black hole
            movementSpeed: 0.8 // Base speed for continuous movement
        };

        let raceStartTime = 0; // To store the timestamp when the race starts
        const helixDuration = 3000; // Duration of rocket helix movement in milliseconds (3 seconds)

        let kineticEnergyHistory = []; // Array to store kinetic energy data for the graph


        // Adjust canvas size and celestial body orbit centers based on window size
        function resizeCanvas() {
            canvas.width = window.innerWidth * 0.9; // 90% of window width
            canvas.height = window.innerHeight * 0.6; // 60% of window height
            if (canvas.width < 600) { // Ensure minimum width on small screens
                canvas.width = 600;
            }
            if (canvas.height < 400) { // Ensure minimum height
                canvas.height = 400;
            }

            // Set graph canvas size relative to game canvas
            graphCanvas.width = canvas.width;
            graphCanvas.height = canvas.height * 0.5; // Graph is half the height of the game canvas

            // Set planet orbit center to the left half of the game canvas
            planet.orbitCenterX = canvas.width * 0.3;
            planet.orbitCenterY = canvas.height * 0.5;
             // Ensure planet orbit radius is not too large for its area
            const maxPlanetOrbitRadius = Math.min(planet.orbitCenterX, canvas.height / 2) - planet.radius - 20;
             if (planet.orbitRadius > maxPlanetOrbitRadius) {
                planet.orbitRadius = maxPlanetOrbitRadius;
            }

            // Black hole position and speed are set on race start, no fixed values here

            drawBackground(); // Redraw game background after resizing
             drawGraphBackground(); // Redraw graph background
            if (gameRunning) {
                updatePlanetPosition(); // Update planet position based on new canvas size
                updateBlackHolePosition(); // Update black hole position
                drawPlanet(); // Redraw planet
                drawBlackHole(); // Redraw black hole
                drawRockets(); // Redraw rockets if game is running
                drawFinishLine(); // Redraw finish line
                drawKineticEnergyGraph(); // Redraw graph
            } else {
                updatePlanetPosition(); // Update planet position even if game is not running
                // Black hole position is not set until race start
                drawStartLine(); // Draw start line if game is not running
                drawPlanet(); // Draw planet even if game is not running
            }
        }

        // Update the planet's position based on its orbit (Circular)
        function updatePlanetPosition() {
             planet.orbitAngle += planet.orbitSpeed;
             planet.x = planet.orbitCenterX + Math.cos(planet.orbitAngle) * planet.orbitRadius;
             planet.y = planet.orbitCenterY + Math.sin(planet.orbitAngle) * planet.orbitRadius;
        }

         // Update the black hole's position with continuous movement and boundary bouncing
        function updateBlackHolePosition() {
            // Update position based on speed
            blackHole.x += blackHole.speedX;
            blackHole.y += blackHole.speedY;

            // Bounce off horizontal boundaries
            if (blackHole.x - blackHole.radius < 0 || blackHole.x + blackHole.radius > canvas.width) {
                blackHole.speedX *= -1; // Reverse horizontal speed
                 // Adjust position to prevent sticking to the edge
                if (blackHole.x - blackHole.radius < 0) blackHole.x = blackHole.radius;
                if (blackHole.x + blackHole.radius > canvas.width) blackHole.x = canvas.width - blackHole.radius;
            }

            // Bounce off vertical boundaries
            if (blackHole.y - blackHole.radius < 0 || blackHole.y + blackHole.radius > canvas.height) {
                blackHole.speedY *= -1; // Reverse vertical speed
                 // Adjust position to prevent sticking to the edge
                if (blackHole.y - blackHole.radius < 0) blackHole.y = blackHole.radius;
                if (blackHole.y + blackHole.radius > canvas.height) blackHole.y = canvas.height - blackHole.radius;
            }
        }


        // Rocket class
        class Rocket {
            constructor(id, color, yPosition, name, speedBoostMultiplier = 1) { // Added name parameter
                this.id = id;
                this.color = color;
                this.x = startLineX; // Rocket starts on the fixed vertical line
                this.y = yPosition; // Rocket starts at the given random Y position
                this.name = name; // Store the rocket's name
                this.speedX = baseRocketSpeed * speedBoostMultiplier; // Apply multiplier to initial speed
                this.speedY = 0; // Speed in Y direction
                this.finished = false;
                this.state = 'racing'; // 'racing', 'approaching_planet', 'orbiting', 'leaving_planet'
                this.orbitAngle = 0; // Current angle relative to the planet's center for helix visualization
                this.orbitRadius = 0; // Radius of the orbit around the planet for helix visualization
                this.helixStartTime = 0; // Timestamp when helix movement started
                this.finishTime = null; // To store the timestamp when the rocket finishes
                this.kineticEnergyHistory = []; // Array to store kinetic energy data
            }

            // Draw the rocket (now horizontal)
            draw() {
                // Only draw if not finished (although update handles this too, good to be explicit)
                if (!this.finished) {
                    ctx.fillStyle = this.color;
                    // Body (now horizontal rectangle)
                    ctx.fillRect(this.x, this.y, rocketWidth, rocketHeight); // x, y, width, height

                    // Nose cone (pointing right)
                    ctx.beginPath();
                    ctx.moveTo(this.x + rocketWidth, this.y); // Right end of body, top corner
                    ctx.lineTo(this.x + rocketWidth + 15, this.y + rocketHeight / 2); // Point to the right
                    ctx.lineTo(this.x + rocketWidth, this.y + rocketHeight); // Right end of body, bottom corner
                    ctx.closePath();
                    ctx.fill();

                    // Fins (simple triangles, now on the bottom and top)
                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y + rocketHeight); // Left end of body, bottom corner
                    ctx.lineTo(this.x - 15, this.y + rocketHeight + 10); // Point down-left
                    ctx.lineTo(this.x - 10, this.y + rocketHeight); // Slightly in from bottom corner
                    ctx.closePath();
                    ctx.fill();

                    ctx.beginPath();
                    ctx.moveTo(this.x, this.y); // Left end of body, top corner
                    ctx.lineTo(this.x - 15, this.y - 10); // Point up-left
                    ctx.lineTo(this.x - 10, this.y); // Slightly in from top corner
                    ctx.closePath();
                    ctx.fill();
                }
            }


            // Update rocket position and speed
            update() {
                if (this.finished) return;

                const currentTime = performance.now(); // Get current time

                // Calculate distance and angle relative to the CURRENT planet position
                const distanceToPlanet = Math.sqrt(Math.pow(this.x - planet.x, 2) + Math.pow(this.y - planet.y, 2));
                const angleToPlanet = Math.atan2(planet.y - this.y, planet.x - this.x);

                // Calculate distance and angle relative to the CURRENT black hole position
                const distanceToBlackHole = Math.sqrt(Math.pow(this.x - blackHole.x, 2) + Math.pow(this.y - blackHole.y, 2));
                const angleToBlackHole = Math.atan2(blackHole.y - this.y, blackHole.x - this.x);

                // --- Black Hole Collision Check ---
                if (distanceToBlackHole < blackHole.radius) {
                    // Rocket is swallowed by the black hole
                    this.reset(); // Reset rocket to start
                    messageBox.innerHTML = `${this.name} absorbit pel forat negre! Tornant a la sortida...`; // Use custom name
                    return; // Stop updating this rocket for this frame
                }
                // --- End Black Hole Collision Check ---


                switch (this.state) {
                    case 'racing':
                        // Normal racing movement with random acceleration
                        this.speedX += (Math.random() * acceleration - (acceleration / 2));
                        if (this.speedX < baseRocketSpeed / 2) this.speedX = baseRocketSpeed / 2;
                        if (this.speedX > baseRocketSpeed * 3) this.speedX = baseRocketSpeed * 3;

                        // Apply gravitational pull from the planet if within its gravity radius and approaching
                         if (distanceToPlanet < planet.gravityRadius && this.x < planet.x + planet.radius) {
                             this.speedX += Math.cos(angleToPlanet) * planet.gravityStrength;
                             this.speedY += Math.sin(angleToPlanet) * planet.gravityStrength;
                         }


                        // Apply gravitational pull from the black hole if within its gravity radius
                         if (distanceToBlackHole < blackHole.gravityRadius) {
                             this.speedX += Math.cos(angleToBlackHole) * blackHole.gravityStrength;
                             this.speedY += Math.sin(angleToBlackHole) * blackHole.gravityStrength;
                         }


                        this.x += this.speedX;
                        this.y += this.speedY;

                        // Check if close enough to start approaching the planet
                        if (distanceToPlanet < planet.gravityRadius && this.x < planet.x + planet.radius && this.x > planet.x - planet.gravityRadius) {
                             this.state = 'approaching_planet'; // Transition to approaching state
                        }
                        // Note: Rockets don't orbit the black hole in this implementation, they are just pulled.
                        break;

                    case 'approaching_planet':
                        // Apply gravitational pull towards the planet
                        this.speedX += Math.cos(angleToPlanet) * planet.gravityStrength;
                        this.speedY += Math.sin(angleToPlanet) * planet.gravityStrength;

                         // Apply gravitational pull from the black hole if within its gravity radius
                         if (distanceToBlackHole < blackHole.gravityRadius) {
                             this.speedX += Math.cos(angleToBlackHole) * blackHole.gravityStrength;
                             this.speedY += Math.sin(angleToBlackHole) * blackHole.gravityStrength;
                         }


                        // Limit speed to avoid excessive acceleration before orbiting
                        const currentSpeedApproaching = Math.sqrt(Math.pow(this.speedX, 2) + Math.pow(this.speedY, 2));
                        if (currentSpeedApproaching > baseRocketSpeed * 4) {
                             const ratio = (baseRocketSpeed * 4) / currentSpeedApproaching;
                             this.speedX *= ratio;
                             this.speedY *= ratio;
                        }

                        this.x += this.speedX;
                        this.y += this.speedY;

                        // Check if close enough to start orbiting the planet
                        if (distanceToPlanet < planet.radius + 20) { // Orbit slightly outside planet radius
                            this.state = 'orbiting';
                            this.helixStartTime = currentTime; // Record start time of helix movement
                            // Store relative position to planet when entering orbit
                            this.orbitRelativeX = this.x - planet.x;
                            this.orbitRelativeY = this.y - planet.y;
                            this.orbitRadius = distanceToPlanet; // Set orbit radius based on current distance
                            this.orbitAngle = Math.atan2(this.orbitRelativeY, this.orbitRelativeX); // Set initial angle relative to planet
                        }
                        break;

                    case 'orbiting':
                        // Orbiting: move in a circle around the CURRENT planet position for a fixed duration
                        const orbitSpeed = 0.08; // How fast it orbits around the planet during helix

                        // Calculate time elapsed in helix state
                        const elapsedTime = currentTime - this.helixStartTime;

                        // Check if helix duration is over
                        if (elapsedTime >= helixDuration) {
                             this.state = 'leaving_planet';
                             // Set initial speed for leaving (away from the CURRENT planet center)
                             const angleAwayFromCurrentPlanet = Math.atan2(this.y - planet.y, this.x - planet.x);
                             // Accelerated speed on leaving
                             const leaveSpeed = baseRocketSpeed * planet.slingshotBoost; // Boosted speed
                             this.speedX = Math.cos(angleAwayFromCurrentPlanet) * leaveSpeed;
                             this.speedY = Math.sin(angleAwayFromCurrentPlanet) * leaveSpeed;

                             // Add some forward bias to speedX to ensure it heads towards the finish line
                             this.speedX += baseRocketSpeed * 0.5;
                             // End Accelerated speed on leaving

                        } else {
                            // Continue helix movement around the moving planet
                            // Increment angle around the planet's current position
                            this.orbitAngle += orbitSpeed;


                            // Calculate rocket's position relative to the planet's center
                            const relativeX = Math.cos(this.orbitAngle) * this.orbitRadius;
                            const relativeY = Math.sin(this.orbitAngle) * this.orbitRadius;

                            // Update rocket's absolute position based on planet's current position
                            this.x = planet.x + relativeX;
                            this.y = planet.y + relativeY;

                             // Apply gravitational pull from the black hole while orbiting the planet
                             if (distanceToBlackHole < blackHole.gravityRadius) {
                                 // This gravity pull will slightly distort the perfect circle around the planet,
                                 // contributing to the helix shape and interaction with the black hole.
                                 this.speedX += Math.cos(angleToBlackHole) * blackHole.gravityStrength;
                                 this.speedY += Math.sin(angleToBlackHole) * blackHole.gravityStrength;
                                  // Re-calculate position based on adjusted speed for smoother movement
                                  this.x += this.speedX;
                                  this.y += this.speedY;
                             } else {
                                // If not under black hole influence, just update based on calculated position
                                // The speedX and speedY are not directly used for movement here,
                                // but they will be set when leaving the planet.
                             }
                        }
                        break;

                    case 'leaving_planet':
                        // Continue moving away from the planet with boosted speed
                        this.x += this.speedX;
                        this.y += this.speedY;

                        // Gradually reduce Y speed to return to horizontal racing
                        this.speedY *= 0.98; // Dampen Y speed

                        // Apply gravitational pull from the black hole while leaving the planet
                         if (distanceToBlackHole < blackHole.gravityRadius) {
                             this.speedX += Math.cos(angleToBlackHole) * blackHole.gravityStrength;
                             this.speedY += Math.sin(angleToBlackHole) * blackHole.gravityStrength;
                         }


                        // Check if far enough from the planet to resume normal racing
                         if (distanceToPlanet > planet.gravityRadius * 1.5) { // Increased distance to leave gravity field
                            this.state = 'racing';
                            this.speedY = 0; // Reset Y speed
                        }
                        break;
                }

                // Calculate kinetic energy (proportional to v^2) and store history
                const velocitySquared = Math.pow(this.speedX, 2) + Math.pow(this.speedY, 2);
                const kineticEnergy = velocitySquared; // Using v^2 as proxy for KE (mass is constant)
                this.kineticEnergyHistory.push({ time: currentTime - raceStartTime, energy: kineticEnergy });


                // Check if finished
                if (this.x >= finishLineX && !this.finished) { // Ensure we only record finish time once
                    this.x = finishLineX; // Ensure it stops at the line
                    this.finished = true;
                    this.finishTime = performance.now(); // Record finish time
                    checkRaceEnd();
                }

                 // Keep rockets within vertical bounds
                if (this.y < 0) this.y = 0;
                if (this.y + rocketHeight > canvas.height) this.y = canvas.height - rocketHeight;
            }

            // Reset the rocket to the start line
            reset() {
                this.x = startLineX;
                // Recalculate random Y position on reset
                this.y = Math.random() * (canvas.height - rocketHeight);

                this.speedX = baseRocketSpeed; // Reset speed to base speed on reset
                this.speedY = 0;
                this.state = 'racing';
                this.orbitsCompleted = 0; // Still keep this for potential future use or clarity
                this.finished = false; // Ensure it's not marked as finished
                this.finishTime = null; // Reset finish time on reset
                this.helixStartTime = 0; // Reset helix start time
                this.kineticEnergyHistory = []; // Clear kinetic energy history on reset
            }
        }

        // Draw the starry background for the game canvas
        function drawBackground() {
            ctx.fillStyle = '#0a0a2a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw stars (simple white dots or SVGs)
            ctx.fillStyle = '#fff';
            for (let i = 0; i < 100; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height;
                // Simple dot
                ctx.beginPath();
                ctx.arc(x, y, Math.random() * 2, 0, Math.PI * 2);
                ctx.fill();
            }
        }

         // Draw the background for the graph canvas
         function drawGraphBackground() {
             graphCtx.fillStyle = '#1a1a3a';
             graphCtx.fillRect(0, 0, graphCanvas.width, graphCanvas.height);
         }


        // Draw the start line
        function drawStartLine() {
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(startLineX, 0);
            ctx.lineTo(startLineX, canvas.height);
            ctx.stroke();

            ctx.font = '12px "Press Start 2P"';
            ctx.fillStyle = '#fff';
            ctx.fillText('Sortida', startLineX - 40, canvas.height - 10);
        }

        // Draw the finish line
        function drawFinishLine() {
            ctx.strokeStyle = '#ffff00'; // Yellow finish line
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.moveTo(finishLineX, 0);
            ctx.lineTo(finishLineX, canvas.height);
            ctx.stroke();

            ctx.font = '12px "Press Start 2P"';
            ctx.fillStyle = '#ffff00';
            ctx.fillText('Meta', finishLineX - 30, canvas.height - 10);
        }

        // Draw the planet
        function drawPlanet() {
            ctx.fillStyle = '#336699'; // Blueish planet color
            ctx.beginPath();
            ctx.arc(planet.x, planet.y, planet.radius, 0, Math.PI * 2);
            ctx.fill();

            // Optional: Draw a ring or texture
            ctx.strokeStyle = '#6699cc';
            ctx.lineWidth = 5;
            ctx.beginPath();
            // Adjust ring radius based on new planet radius
            ctx.arc(planet.x, planet.y, planet.radius + 10, 0, Math.PI * 2);
            ctx.stroke();
        }

        // Draw the black hole (a black circle)
        function drawBlackHole() {
            ctx.fillStyle = '#000'; // Black color
            ctx.beginPath();
            ctx.arc(blackHole.x, blackHole.y, blackHole.radius, 0, Math.PI * 2);
            ctx.fill();

            // Optional: Draw an accretion disk (simple ring)
            ctx.strokeStyle = '#ff00ff'; // Pinkish color
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.arc(blackHole.x, blackHole.y, blackHole.radius + 5, 0, Math.PI * 2);
            ctx.stroke();
        }

        // Function to create rocket name input fields based on numberOfRockets
        function createNameInputs() {
            rocketNameInputsDiv.innerHTML = ''; // Clear previous inputs
            const colors = [
                { color: '#ff0000', name: 'Coet Vermell', speedBoost: 1 },
                { color: '#ffff00', name: 'Coet Groc', speedBoost: 1.2 },
                { color: '#0000ff', name: 'Coet Blau', speedBoost: 1 },
                // Add more colors/names if you plan for more rockets
                { color: '#00ffff', name: 'Coet Cyan', speedBoost: 1 },
                { color: '#ff00ff', name: 'Coet Magenta', speedBoost: 1 },
                { color: '#ffffff', name: 'Coet Blanc', speedBoost: 1 },
                { color: '#ffa500', name: 'Coet Taronja', speedBoost: 1 },
                { color: '#800080', name: 'Coet Lila', speedBoost: 1 },
                { color: '#008000', name: 'Coet Verd Fosc', speedBoost: 1 },
                { color: '#808080', name: 'Coet Gris', speedBoost: 1 }
            ];
             for (let i = 0; i < numberOfRockets; i++) {
                const inputContainer = document.createElement('div');
                inputContainer.classList.add('rocket-name-input');

                const label = document.createElement('label');
                // Use default name from the colors array
                const defaultName = colors[i % colors.length].name;
                label.textContent = `Nom Coet ${i + 1} (${defaultName}):`;
                label.style.color = colors[i % colors.length].color; // Set label color

                const input = document.createElement('input');
                input.type = 'text';
                input.id = `rocketName${i + 1}`;
                input.value = defaultName; // Default name
                input.maxLength = 15; // Limit name length

                inputContainer.appendChild(label);
                inputContainer.appendChild(input);
                rocketNameInputsDiv.appendChild(inputContainer);
            }
        }


        // Initialize rockets with random Y positions and custom names
        function initRockets() {
            rockets = [];
             kineticEnergyHistory = []; // Clear global history array

            // Recalculate vertical spacing based on the current numberOfRockets
            const verticalSpacing = (canvas.height - rocketHeight) / numberOfRockets;
            const colors = [
                { color: '#ff0000', speedBoost: 1 }, // Red rocket (normal speed)
                { color: '#ffff00', speedBoost: 1.2 }, // Yellow rocket (1.2x speed)
                { color: '#0000ff', speedBoost: 1 },  // Blue rocket (normal speed)
                 // Add more colors/speedBoosts here if needed for more rockets
                { color: '#00ffff', speedBoost: 1 },
                { color: '#ff00ff', speedBoost: 1 },
                { color: '#ffffff', speedBoost: 1 },
                { color: '#ffa500', speedBoost: 1 },
                { color: '#800080', speedBoost: 1 },
                { color: '#008000', speedBoost: 1 },
                { color: '#808080', speedBoost: 1 }
            ];


            for (let i = 0; i < numberOfRockets; i++) {
                // Random Y position within bounds, considering rocket height
                const yPos = Math.random() * (canvas.height - rocketHeight);
                const rocketData = colors[i % colors.length]; // Cycle through available colors
                // Get custom name from input field, default to Coet X if empty
                const customNameInput = document.getElementById(`rocketName${i + 1}`);
                const rocketName = customNameInput && customNameInput.value.trim() !== '' ? customNameInput.value.trim() : `Coet ${i + 1}`;

                // Pass yPos, rocketName, and speedBoost to the Rocket constructor
                const rocket = new Rocket(i + 1, rocketData.color, yPos, rocketName, rocketData.speedBoost);
                rockets.push(rocket);
                 // Initialize an empty array for this rocket's energy history in the global array
                 kineticEnergyHistory.push(rocket.kineticEnergyHistory);
            }
        }

        // Draw all rockets
        function drawRockets() {
            rockets.forEach(rocket => rocket.draw());
        }

        // Update all rockets
        function updateRockets() {
            rockets.forEach(rocket => {
                // Only update if the rocket is not finished
                if (!rocket.finished) {
                     rocket.update();
                }
            });
        }

        // Check if the race has ended
        function checkRaceEnd() {
            const finishedRockets = rockets.filter(rocket => rocket.finished);
            // Check if the number of finished rockets equals the current numberOfRockets
            if (finishedRockets.length === numberOfRockets) {
                gameRunning = false;
                startButton.disabled = false;
                enableNameInputs(); // Enable name inputs after race ends
                 numRocketsInput.disabled = false; // Enable number input
                displayResults();
                 drawKineticEnergyGraph(); // Draw final graph
            }
        }

        // Display race results with finish times and custom names
        function displayResults() {
            // Sort rockets by finish time
            const sortedRockets = [...rockets].sort((a, b) => {
                // Finished rockets come first, sorted by finish time
                if (a.finished && b.finished) {
                    return a.finishTime - b.finishTime;
                }
                // Finished rockets come before unfinished ones
                if (a.finished && b.finished === undefined) { // Handle case where b is not finished
                     return -1;
                }
                 if (b.finished && a.finished === undefined) { // Handle case where a is not finished
                     return 1;
                 }
                // Unfinished rockets remain in their current order (or could sort by distance)
                return 0; // Keep original order for unfinished
            });

            let resultsText = 'Resultats de la cursa:<br>';
            sortedRockets.forEach((rocket, index) => {
                resultsText += `${index + 1}. ${rocket.name} (${rocket.color}) - `; // Use custom name
                if (rocket.finished) {
                    const duration = (rocket.finishTime - raceStartTime) / 1000; // Duration in seconds
                    resultsText += `Arribat (${duration.toFixed(2)} s)`; // Display time in seconds
                } else {
                    resultsText += 'En cursa';
                }
                resultsText += '<br>';
            });
            messageBox.innerHTML = resultsText;
        }

        // Disable name input fields and number input during the race
        function disableNameInputs() {
            const inputs = rocketNameInputsDiv.querySelectorAll('input[type="text"]');
            inputs.forEach(input => input.disabled = true);
             numRocketsInput.disabled = true; // Disable number input
        }

        // Enable name input fields and number input after the race
        function enableNameInputs() {
             const inputs = rocketNameInputsDiv.querySelectorAll('input[type="text"]');
             inputs.forEach(input => input.disabled = false);
             numRocketsInput.disabled = false; // Enable number input
        }

         // Draw the kinetic energy graph
         function drawKineticEnergyGraph() {
             drawGraphBackground(); // Clear graph canvas

             const graphWidth = graphCanvas.width;
             const graphHeight = graphCanvas.height;
             const padding = 30; // Padding for axes

             // Find max time and max energy for scaling
             let maxTime = 0;
             let maxEnergy = 0;
             rockets.forEach(rocket => {
                 rocket.kineticEnergyHistory.forEach(dataPoint => {
                     if (dataPoint.time > maxTime) maxTime = dataPoint.time;
                     if (dataPoint.energy > maxEnergy) maxEnergy = dataPoint.energy;
                 });
             });

             // Add some buffer to max energy
             maxEnergy *= 1.1;


             // Draw axes
             graphCtx.strokeStyle = '#fff';
             graphCtx.lineWidth = 1;
             graphCtx.beginPath();
             // Y-axis
             graphCtx.moveTo(padding, padding);
             graphCtx.lineTo(padding, graphHeight - padding);
             // X-axis
             graphCtx.moveTo(padding, graphHeight - padding);
             graphCtx.lineTo(graphWidth - padding, graphHeight - padding);
             graphCtx.stroke();

             // Draw axis labels
             graphCtx.font = '10px "Press Start 2P"';
             graphCtx.fillStyle = '#fff';
             graphCtx.textAlign = 'center';
             graphCtx.fillText('Temps (ms)', graphWidth / 2, graphHeight - padding / 2);
             graphCtx.save(); // Save context state before rotating
             graphCtx.translate(padding / 2, graphHeight / 2);
             graphCtx.rotate(-Math.PI / 2);
             graphCtx.textAlign = 'center';
             graphCtx.fillText('Energia Cinètica (v^2)', 0, 0);
             graphCtx.restore(); // Restore context state

             // Draw energy lines for each rocket
             rockets.forEach(rocket => {
                 graphCtx.strokeStyle = rocket.color;
                 graphCtx.lineWidth = 2;
                 graphCtx.beginPath();

                 rocket.kineticEnergyHistory.forEach((dataPoint, index) => {
                     // Scale data points to canvas coordinates
                     const x = padding + (dataPoint.time / maxTime) * (graphWidth - 2 * padding);
                     const y = graphHeight - padding - (dataPoint.energy / maxEnergy) * (graphHeight - 2 * padding);

                     if (index === 0) {
                         graphCtx.moveTo(x, y);
                     } else {
                         graphCtx.lineTo(x, y);
                     }
                 });
                 graphCtx.stroke();

                 // Draw legend (simple colored square and name)
                 const legendX = graphWidth - padding - 100;
                 const legendY = padding + (rocket.id - 1) * 20;
                 graphCtx.fillStyle = rocket.color;
                 graphCtx.fillRect(legendX, legendY - 8, 10, 10); // Colored square
                 graphCtx.fillStyle = '#fff';
                 graphCtx.textAlign = 'left';
                 graphCtx.fillText(rocket.name, legendX + 15, legendY); // Rocket name
             });
         }


        // Game loop
        function gameLoop() {
            if (!gameRunning) return;

            updatePlanetPosition(); // Update planet's position each frame
            updateBlackHolePosition(); // Update black hole's position each frame

            drawBackground();
            drawPlanet(); // Draw the planet at its new position
            drawBlackHole(); // Draw the black hole at its new position
            drawFinishLine(); // Draw finish line after background and celestial bodies
            updateRockets();
            drawRockets();

             drawKineticEnergyGraph(); // Draw the graph in each frame


            requestAnimationFrame(gameLoop);
        }

        // Start the race
        function startRace() {
            if (!gameRunning) {
                // Get the number of rockets from the input field
                numberOfRockets = parseInt(numRocketsInput.value);
                 // Ensure numberOfRockets is within a reasonable range (e.g., 1 to 10)
                 if (isNaN(numberOfRockets) || numberOfRockets < 1) {
                     numberOfRockets = 1;
                     numRocketsInput.value = 1;
                 } else if (numberOfRockets > 10) {
                      numberOfRockets = 10;
                      numRocketsInput.value = 10;
                 }


                gameRunning = true;
                startButton.disabled = true;
                disableNameInputs(); // Disable name inputs and number input
                messageBox.textContent = 'Cursa en marxa...';
                // Re-create name inputs and initialize rockets based on the new numberOfRockets
                createNameInputs(); // Recreate inputs based on the new number
                initRockets(); // Initialize rockets with random Y positions and apply speed boost

                raceStartTime = performance.now(); // Record the start time
                // Ensure celestial bodies are at the start of their orbits when race begins
                planet.orbitAngle = 0;
                // Set initial position and random velocity for the black hole
                blackHole.x = canvas.width * 0.7; // Starting position for black hole
                blackHole.y = canvas.height * 0.5;
                // Give black hole a random initial direction and the defined movement speed
                const angle = Math.random() * Math.PI * 2;
                blackHole.speedX = Math.cos(angle) * blackHole.movementSpeed;
                blackHole.speedY = Math.sin(angle) * blackHole.movementSpeed;

                updatePlanetPosition(); // Set initial planet position
                 drawGraphBackground(); // Clear graph before starting new race
                gameLoop();
            }
        }

        // Event listeners
        startButton.addEventListener('click', startRace);
         // Add event listener to update name inputs when number of rockets changes
         numRocketsInput.addEventListener('change', createNameInputs);


        window.addEventListener('resize', resizeCanvas);

        // Initial setup
        window.onload = function() {
            resizeCanvas(); // Set initial canvas size and draw background/start line/celestial bodies
            createNameInputs(); // Create initial name input fields (default 3)
            initRockets(); // Initialize rockets on load (with default names initially)
            drawStartLine(); // Draw start line on load
            drawPlanet(); // Draw planet on load (at initial fixed position)
             drawGraphBackground(); // Draw initial graph background
            // Black hole is not drawn until race start
        };

    </script>
</body>
</html>
