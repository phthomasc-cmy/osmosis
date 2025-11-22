<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>High-Precision Osmosis Simulator</title>
    <style>
        :root {
            --water-blue: #3498db;
            --solute-red: #e74c3c;
            --text-dark: #2c3e50;
            --bg-grey: #f4f7f6;
            --panel-white: #ffffff;
        }

        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-grey);
            color: var(--text-dark);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        h1 { margin-top: 0; color: var(--text-dark); }
        .subtitle { color: #7f8c8d; margin-bottom: 20px; font-size: 0.9rem; }

        .main-card {
            background: var(--panel-white);
            border-radius: 12px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.08);
            padding: 25px;
            width: 100%;
            max-width: 900px;
        }

        /* Legend */
        .legend {
            display: flex;
            justify-content: center;
            gap: 30px;
            margin-bottom: 15px;
            font-size: 0.85rem;
            font-weight: 600;
        }
        .legend-item { display: flex; align-items: center; gap: 6px; }
        .dot-water { width: 8px; height: 8px; background: var(--water-blue); border-radius: 50%; }
        .dot-solute { width: 8px; height: 8px; background: var(--solute-red); border-radius: 1px; }

        /* Simulation Area */
        .sim-container {
            position: relative;
            width: 100%;
            height: 400px;
            border: 1px solid #dcdcdc;
            background: #fff;
            border-radius: 4px;
            overflow: hidden;
        }

        canvas { display: block; width: 100%; height: 100%; }

        /* Overlays */
        .labels-top {
            position: absolute;
            top: 10px;
            width: 100%;
            display: flex;
            justify-content: space-between;
            padding: 0 20px;
            box-sizing: border-box;
            pointer-events: none;
            font-size: 0.8rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }
        .lbl-left { color: var(--water-blue); }
        .lbl-right { color: var(--solute-red); }

        /* Controls & Stats Grid */
        .dashboard {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-top: 25px;
        }

        @media (max-width: 700px) {
            .dashboard { grid-template-columns: 1fr; }
        }

        /* Slider Section */
        .control-panel {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 8px;
            border: 1px solid #eee;
        }
        
        label { font-weight: 600; font-size: 0.9rem; margin-bottom: 10px; display: block; }
        
        input[type=range] {
            width: 100%;
            -webkit-appearance: none;
            background: transparent;
        }
        input[type=range]::-webkit-slider-track {
            width: 100%;
            height: 6px;
            background: #ddd;
            border-radius: 3px;
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: 18px;
            width: 18px;
            border-radius: 50%;
            background: var(--solute-red);
            margin-top: -6px;
            cursor: pointer;
        }

        /* Stats Table */
        .stats-panel {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            border: 1px solid #eee;
            font-size: 0.85rem;
        }
        
        .stat-row {
            display: flex;
            justify-content: space-between;
            padding: 8px 0;
            border-bottom: 1px solid #e0e0e0;
        }
        .stat-row:last-child { border-bottom: none; }
        .stat-val { font-family: monospace; font-weight: 700; font-size: 1rem; }

        /* Explanation */
        .explanation {
            margin-top: 20px;
            font-size: 0.9rem;
            line-height: 1.5;
            color: #555;
            border-top: 1px solid #eee;
            padding-top: 15px;
        }
    </style>
</head>
<body>

    <h1>Osmosis Precision Simulator</h1>
    <div class="subtitle">High-density particle system demonstrating osmotic pressure</div>

    <div class="main-card">
        
        <div class="legend">
            <div class="legend-item">
                <div class="dot-water"></div> <span>Water (Solvent)</span>
            </div>
            <div class="legend-item">
                <div class="dot-solute"></div> <span>Solute (Salt/Sugar)</span>
            </div>
        </div>

        <div class="sim-container">
            <div class="labels-top">
                <span id="txt-left" class="lbl-left">Low Solute</span>
                <span id="txt-right" class="lbl-right">High Solute</span>
            </div>
            <canvas id="simCanvas"></canvas>
        </div>

        <div class="dashboard">
            <!-- Controls -->
            <div class="control-panel">
                <label>
                    Right Side Solute Concentration
                    <span style="float:right; color:var(--solute-red);" id="conc-display">0%</span>
                </label>
                <input type="range" id="soluteSlider" min="0" max="100" value="0">
                <p style="font-size: 0.8rem; color: #666; margin-top: 10px;">
                    Drag slider to add solute particles. Water will naturally migrate to the right side to dilute the solute.
                </p>
            </div>

            <!-- Data Stats -->
            <div class="stats-panel">
                <div style="font-weight:bold; margin-bottom:10px; color:#333;">Real-time Stats</div>
                <div class="stat-row">
                    <span>Water Count (Left):</span>
                    <span id="count-water-left" class="stat-val text-blue">0</span>
                </div>
                <div class="stat-row">
                    <span>Water Count (Right):</span>
                    <span id="count-water-right" class="stat-val text-blue">0</span>
                </div>
                <div class="stat-row">
                    <span>Net Water Movement:</span>
                    <span id="net-movement" class="stat-val">Balanced</span>
                </div>
            </div>
        </div>

        <div class="explanation">
            <strong>Note on Precision:</strong> This simulation uses <strong>~2,000 particles</strong> to approximate fluid dynamics. 
            Unlike simple animations, the water level rising is an <em>emergent property</em> of the random movement of individual molecules 
            reacting to the semi-permeable membrane logic.
        </div>
    </div>

    <script>
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');
        
        // DOM Elements
        const slider = document.getElementById('soluteSlider');
        const concDisplay = document.getElementById('conc-display');
        const elWaterLeft = document.getElementById('count-water-left');
        const elWaterRight = document.getElementById('count-water-right');
        const elNetMove = document.getElementById('net-movement');
        const txtLeft = document.getElementById('txt-left');
        const txtRight = document.getElementById('txt-right');

        // SIMULATION CONFIG (Precision Mode)
        // Reduced sizes to 1/10th feel, increased counts greatly
        const WATER_RADIUS = 1.5; 
        const SOLUTE_SIZE = 3;    
        const WATER_COUNT = 1500; 
        const MAX_SOLUTES = 400;  
        const SPEED = 1.2;        // Slower speed for small particles looks more like liquid
        
        // Physics State
        let particles = [];
        let width, height, membraneX;
        let leftWaterHeight = 0;
        let rightWaterHeight = 0;

        class Particle {
            constructor(type, forcedSide = null) {
                this.type = type;
                this.radius = (type === 'water') ? WATER_RADIUS : (SOLUTE_SIZE / 2);
                this.color = (type === 'water') ? '#3498db' : '#e74c3c';
                
                // Initialize Position
                const side = forcedSide || (Math.random() < 0.5 ? 'left' : 'right');
                const minX = side === 'left' ? 5 : membraneX + 5;
                const maxX = side === 'left' ? membraneX - 5 : width - 5;
                
                this.x = Math.random() * (maxX - minX) + minX;
                this.y = Math.random() * (height - 10) + 10;

                // Velocity (Brownian motion)
                this.vx = (Math.random() - 0.5) * SPEED;
                this.vy = (Math.random() - 0.5) * SPEED;
            }

            update(soluteFactor) {
                this.x += this.vx;
                this.y += this.vy;

                // Bounce off walls
                if (this.y < 2 || this.y > height - 2) this.vy *= -1;
                if (this.x < 2 || this.x > width - 2) this.vx *= -1;

                // Membrane Interaction
                // Membrane is at membraneX
                const margin = 4;
                
                if (this.type === 'solute') {
                    // Solute: HARD collision with membrane
                    if (this.x > membraneX - margin && this.x < membraneX + margin) {
                        // If on right side, bounce right
                        if (this.x > membraneX) {
                            this.x = membraneX + margin;
                            this.vx = Math.abs(this.vx);
                        } else {
                            this.x = membraneX - margin;
                            this.vx = -Math.abs(this.vx);
                        }
                    }
                } else {
                    // Water: Semi-permeable Logic
                    // Logic: It is harder to leave a high-concentration area (Osmotic binding)
                    
                    // Crossing Right -> Left
                    if (this.x < membraneX && (this.x - this.vx) >= membraneX) {
                        // Attempting to cross from Right to Left
                        // If solute concentration is high on right (soluteFactor),
                        // there is a probability it gets "stuck" and bounces back to right.
                        if (Math.random() < soluteFactor) {
                            this.x = membraneX + 1; // Stay on right
                            this.vx *= -1;
                        }
                    }
                    
                    // Crossing Left -> Right
                    // Usually free flow, or slightly pulled in.
                    // We leave this natural.
                }
            }

            draw() {
                ctx.fillStyle = this.color;
                if (this.type === 'water') {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                    ctx.fill();
                } else {
                    // Tiny square for solute
                    ctx.fillRect(this.x - this.radius, this.y - this.radius, SOLUTE_SIZE, SOLUTE_SIZE);
                }
            }
        }

        function resize() {
            const rect = canvas.parentElement.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;
            width = canvas.width;
            height = canvas.height;
            membraneX = width / 2;
        }

        function initParticles() {
            particles = [];
            // Create massive amount of water
            for(let i=0; i<WATER_COUNT; i++) {
                // Distribute evenly 50/50 initially
                const side = i < WATER_COUNT / 2 ? 'left' : 'right';
                particles.push(new Particle('water', side));
            }
            updateSimulationState();
        }

        function updateSimulationState() {
            const val = parseInt(slider.value); // 0 to 100
            concDisplay.innerText = val + "%";

            // Calculate target solute count
            const targetSolutes = Math.floor((val / 100) * MAX_SOLUTES);
            
            // Current solutes
            let currentSolutes = particles.filter(p => p.type === 'solute');
            const waters = particles.filter(p => p.type === 'water');

            if (currentSolutes.length < targetSolutes) {
                // Add solutes (always to right side)
                for(let i = currentSolutes.length; i < targetSolutes; i++) {
                    currentSolutes.push(new Particle('solute', 'right'));
                }
            } else if (currentSolutes.length > targetSolutes) {
                // Remove solutes
                currentSolutes = currentSolutes.slice(0, targetSolutes);
            }

            particles = [...waters, ...currentSolutes];

            // Update Text Labels
            if (val > 10) {
                txtLeft.innerText = "Hypotonic (High Water Potential)";
                txtRight.innerText = "Hypertonic (Low Water Potential)";
                txtLeft.style.color = "#3498db";
                txtRight.style.color = "#e74c3c";
            } else {
                txtLeft.innerText = "Isotonic";
                txtRight.innerText = "Isotonic";
                txtLeft.style.color = "#7f8c8d";
                txtRight.style.color = "#7f8c8d";
            }
        }

        function drawEnvironment() {
            // 1. Calculate Counts
            const waterLeft = particles.filter(p => p.type === 'water' && p.x < membraneX).length;
            const waterRight = particles.filter(p => p.type === 'water' && p.x > membraneX).length;
            
            // Update Table UI
            elWaterLeft.innerText = waterLeft;
            elWaterRight.innerText = waterRight;
            
            const diff = waterRight - waterLeft;
            if (Math.abs(diff) < 20) elNetMove.innerText = "Equilibrium";
            else if (diff > 0) elNetMove.innerText = "→ Right";
            else elNetMove.innerText = "← Left";

            // 2. Draw Water Background Levels (Volume Visualization)
            // Base volume calculation
            const halfTotal = WATER_COUNT / 2;
            // If side has more water, level rises.
            // Max height roughly 90% of canvas
            const maxLiquidH = height * 0.95;
            
            const targetHLeft = (waterLeft / halfTotal) * (maxLiquidH * 0.55); 
            const targetHRight = (waterRight / halfTotal) * (maxLiquidH * 0.55);
            
            // Lerp for smooth visual transition of the water line
            leftWaterHeight += (targetHLeft - leftWaterHeight) * 0.1;
            rightWaterHeight += (targetHRight - rightWaterHeight) * 0.1;
            
            // Clamp
            const renderLeft = Math.min(height, Math.max(20, leftWaterHeight));
            const renderRight = Math.min(height, Math.max(20, rightWaterHeight));

            // Draw Background Liquid (Left)
            ctx.fillStyle = "rgba(52, 152, 219, 0.1)";
            ctx.fillRect(0, height - renderLeft, membraneX, renderLeft);
            
            // Draw Background Liquid (Right)
            ctx.fillRect(membraneX, height - renderRight, width - membraneX, renderRight);

            // Draw Surface Lines
            ctx.strokeStyle = "#3498db";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(0, height - renderLeft);
            ctx.lineTo(membraneX, height - renderLeft);
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(membraneX, height - renderRight);
            ctx.lineTo(width, height - renderRight);
            ctx.stroke();

            // Draw Membrane
            ctx.beginPath();
            ctx.moveTo(membraneX, 0);
            ctx.lineTo(membraneX, height);
            ctx.strokeStyle = "#bdc3c7";
            ctx.lineWidth = 2;
            ctx.setLineDash([4, 4]); // Dashed line
            ctx.stroke();
            ctx.setLineDash([]);
        }

        function animate() {
            ctx.clearRect(0, 0, width, height);
            
            drawEnvironment();

            // Calculate bias based on slider
            // 0.0 to 0.9 (Don't go to 1.0 or water can never return)
            const soluteFactor = (parseInt(slider.value) / 100) * 0.85;

            // Draw & Move Particles
            for (let i = 0; i < particles.length; i++) {
                const p = particles[i];
                p.update(soluteFactor);
                p.draw();
            }

            requestAnimationFrame(animate);
        }

        // Initialization
        window.addEventListener('resize', () => { resize(); });
        slider.addEventListener('input', updateSimulationState);
        
        resize();
        initParticles();
        animate();

    </script>
</body>
</html>
