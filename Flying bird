<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>The Flying Face Game</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Set font to Inter for clean look */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f4f8;
            height: 100vh;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        /* Game Container */
        #game-container {
            width: 100%;
            max-width: 450px; /* Mobile-first width constraint */
            height: 80vh;
            max-height: 700px;
            background: linear-gradient(to bottom, #7dd3fc, #38bdf8);
            position: relative;
            overflow: hidden;
            border: 8px solid #334155;
            border-radius: 16px;
            box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
            cursor: pointer;
        }

        /* Player (The Flying Face) */
        #player {
            position: absolute;
            left: 50px;
            width: 60px;
            height: 60px;
            transition: transform 0.05s linear;
            z-index: 10;
            border-radius: 50%; /* Makes the player element circular */
            background-color: transparent;
            /* Using the face image for the player */
            background-image: url('uploaded:87552b3a-cc0e-4fe8-876e-47a6c4a1d324-1_all_64.jpg-fb26d422-359e-40a5-a837-57d77bc59d75');
            background-size: cover;
            background-position: center;
            border: 4px solid #f97316;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.7);
        }

        /* Pipes (Walls) */
        .pipe {
            position: absolute;
            width: 80px; 
            background-repeat: repeat;
            /* Using the second photo as the wall/pipe texture */
            background-image: url('uploaded:87552b3a-cc0e-4fe8-876e-47a6c4a1d324-1_all_1356.jpg-80c831f3-2874-479d-88a2-e1767e43d760');
            background-size: 100% 100px; /* Stretch to width, repeat vertically */
            border: 4px solid #14532d; /* Dark green border */
            box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.5);
        }

        .pipe-top {
            border-bottom: none;
            border-radius: 0 0 12px 12px;
        }

        .pipe-bottom {
            border-top: none;
            border-radius: 12px 12px 0 0;
        }

        /* Ground */
        #ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 10vh;
            background-color: #059669;
            border-top: 5px solid #065f46;
            z-index: 20;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 0.9rem;
            font-weight: bold;
            color: #ecfdf5;
        }

        /* Game Over Screen/Start Screen */
        .message-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 30;
            background-color: rgba(255, 255, 255, 0.95);
            padding: 24px;
            border-radius: 12px;
            text-align: center;
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
            max-width: 90%;
            min-width: 280px;
        }

        .message-box button {
            transition: all 0.2s;
        }
        .message-box button:hover {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }
    </style>
</head>
<body>
    <!-- Game Container -->
    <div id="game-container" class="touch-manipulation" onclick="handleTapOrClick()">
        <div id="score-display" class="absolute top-4 left-4 text-2xl font-black text-white drop-shadow-lg z-10">SCORE: 0</div>
        <div id="high-score-display" class="absolute top-4 right-4 text-xl font-bold text-white drop-shadow-lg z-10">HIGH: 0</div>
        <div id="player"></div>
        <div id="ground">The Flying Face Game</div>

        <!-- Start/Game Over Message -->
        <div id="start-message" class="message-box">
            <h2 class="text-3xl font-extrabold text-blue-600 mb-2">The Flying Face</h2>
            <p class="text-gray-700 mb-6">Click or **Tap** to Fly!</p>
            <p id="final-score" class="text-2xl font-bold text-red-500 mb-4 hidden"></p>
            <button id="start-button" class="px-6 py-3 bg-green-500 text-white font-bold rounded-full shadow-lg hover:bg-green-600 transition duration-150">START GAME</button>
        </div>
    </div>

    <script>
        // --- Global Variables and Constants ---
        const GAME_STATE = {
            WAITING: 'waiting',
            PLAYING: 'playing',
            GAME_OVER: 'gameOver'
        };

        let gameState = GAME_STATE.WAITING;
        let score = 0;
        let highscore = localStorage.getItem('flyingFaceHighscore') || 0;
        let lastTimestamp = 0;

        // Game Element IDs
        const gameContainer = document.getElementById('game-container');
        const player = document.getElementById('player');
        const scoreDisplay = document.getElementById('score-display');
        const highScoreDisplay = document.getElementById('high-score-display');
        const startMessage = document.getElementById('start-message');
        const finalScoreDisplay = document.getElementById('final-score');
        const startButton = document.getElementById('start-button');
        const ground = document.getElementById('ground');
        
        // Game Physics
        const gravity = 1000; // pixels/second/second
        const jumpVelocity = -350; // pixels/second
        const pipeSpeed = 150; // pixels/second
        const pipeGap = 200; // horizontal distance between pipes (pixels)
        const pipeVerticalGap = 160; // vertical gap for the player to fly through (pixels)

        let playerY; 
        let playerVelocity = 0;
        let pipes = [];

        // --- Helper Functions ---

        function updateScore(newScore) {
            score = newScore;
            scoreDisplay.textContent = `SCORE: ${score}`;
            if (score > highscore) {
                highscore = score;
                localStorage.setItem('flyingFaceHighscore', highscore);
                highScoreDisplay.textContent = `HIGH: ${highscore}`;
            }
        }

        function handleTapOrClick() {
            if (gameState === GAME_STATE.WAITING) {
                startGame();
            } else if (gameState === GAME_STATE.PLAYING) {
                playerVelocity = jumpVelocity;
            }
        }

        // --- Pipe Management ---

        function createPipePair() {
            // Using offsetHeight/Width for stable measurements
            const containerHeight = gameContainer.offsetHeight;
            const containerWidth = gameContainer.offsetWidth;
            const groundHeight = ground.offsetHeight;

            // Calculate available height for pipe generation
            const availableHeight = containerHeight - groundHeight - pipeVerticalGap;
            const minPipeHeight = 50;
            
            // Randomly determine the height of the top pipe 
            const topPipeHeight = Math.max(minPipeHeight, Math.random() * (availableHeight - 2 * minPipeHeight) + minPipeHeight);
            
            // Bottom pipe fills the remaining space
            const bottomPipeHeight = containerHeight - groundHeight - topPipeHeight - pipeVerticalGap;

            const pipeX = containerWidth;

            // Top Pipe
            const topPipe = document.createElement('div');
            topPipe.className = 'pipe pipe-top';
            topPipe.style.height = `${topPipeHeight}px`;
            topPipe.style.top = '0';
            topPipe.style.left = `${pipeX}px`;
            topPipe.dataset.x = pipeX;

            // Bottom Pipe
            const bottomPipe = document.createElement('div');
            bottomPipe.className = 'pipe pipe-bottom';
            bottomPipe.style.height = `${bottomPipeHeight}px`;
            bottomPipe.style.bottom = `${groundHeight}px`;
            bottomPipe.style.left = `${pipeX}px`;
            bottomPipe.dataset.x = pipeX;

            gameContainer.appendChild(topPipe);
            gameContainer.appendChild(bottomPipe);

            pipes.push({ top: topPipe, bottom: bottomPipe, scored: false });
        }

        function updatePipes(deltaTime) {
            const distance = pipeSpeed * deltaTime / 1000;
            const playerRect = player.getBoundingClientRect();

            pipes = pipes.filter(pair => {
                // Update position
                const newX = parseFloat(pair.top.dataset.x) - distance;
                pair.top.dataset.x = newX;
                pair.bottom.dataset.x = newX;

                pair.top.style.left = `${newX}px`;
                pair.bottom.style.left = `${newX}px`;

                const pipeRect = pair.top.getBoundingClientRect();

                // Check for score
                if (!pair.scored && pipeRect.right < playerRect.left) {
                    updateScore(score + 1);
                    pair.scored = true;
                }

                // Check for collision 
                if (checkCollision(player, pair.top) || checkCollision(player, pair.bottom)) {
                    endGame();
                }

                // Remove pipes that are off-screen
                if (newX + 80 < 0) {
                    pair.top.remove();
                    pair.bottom.remove();
                    return false;
                }
                return true;
            });
        }

        // --- Collision Detection ---
        function checkCollision(element1, element2) {
            const rect1 = element1.getBoundingClientRect();
            const rect2 = element2.getBoundingClientRect();

            const padding = 5; // A little buffer for forgiving collisions

            return (
                rect1.left + padding < rect2.right &&
                rect1.right - padding > rect2.left &&
                rect1.top + padding < rect2.bottom &&
                rect1.bottom - padding > rect2.top
            );
        }

        function checkGroundCollision() {
            const playerRect = player.getBoundingClientRect();
            const groundRect = ground.getBoundingClientRect();
            
            // Check if player has hit the ground or the ceiling
            if (playerRect.bottom > groundRect.top || playerY <= 0) {
                endGame();
                return true;
            }
            return false;
        }

        // --- Game Loop ---

        function gameLoop(timestamp) {
            if (gameState !== GAME_STATE.PLAYING) return;

            if (!lastTimestamp) {
                lastTimestamp = timestamp;
            }

            const deltaTime = timestamp - lastTimestamp;
            lastTimestamp = timestamp;

            const dt_sec = deltaTime / 1000;

            // 1. Update Player Physics
            playerVelocity += gravity * dt_sec;
            playerY += playerVelocity * dt_sec;
            
            // Boundary Check 
            if (playerY < 0) {
                playerY = 0;
                playerVelocity = 0;
            }

            // Apply position and rotation for tilt effect
            const rotation = Math.min(Math.max(-45, playerVelocity * 0.05), 90);
            player.style.top = `${playerY}px`;
            player.style.transform = `rotate(${rotation}deg)`;

            // 2. Check for Ground/Ceiling Collision
            if (checkGroundCollision()) return;

            // 3. Update Pipes and Check Collision
            updatePipes(deltaTime);

            // 4. Periodically Create New Pipes (Checks width using offsetWidth)
            if (pipes.length === 0 || gameContainer.offsetWidth - parseFloat(pipes[pipes.length - 1].top.dataset.x) >= pipeGap) {
                createPipePair();
            }
            
            // 5. Continue Loop
            requestAnimationFrame(gameLoop);
        }

        // --- Game State Functions ---

        function resetGame() {
            // Remove all existing pipes
            pipes.forEach(pair => {
                pair.top.remove();
                pair.bottom.remove();
            });
            pipes = [];

            // Reset player position and velocity
            // Use offsetHeight for stable centering within the playable area
            const playableHeight = gameContainer.offsetHeight - ground.offsetHeight;
            playerY = playableHeight / 2 - player.offsetHeight / 2;
            
            playerVelocity = 0;
            player.style.top = `${playerY}px`;
            player.style.transform = 'rotate(0deg)';

            // Reset score
            updateScore(0);
        }

        function startGame() {
            // Only start if the game is not already running
            if (gameState !== GAME_STATE.PLAYING) {
                resetGame();
                gameState = GAME_STATE.PLAYING;
    
                // Hide start message
                startMessage.classList.add('hidden');
    
                // Start the main loop
                lastTimestamp = 0;
                requestAnimationFrame(gameLoop);
            }
        }

        function endGame() {
            if (gameState === GAME_STATE.GAME_OVER) return;

            gameState = GAME_STATE.GAME_OVER;
            
            player.style.transform = 'rotate(90deg)'; // Drop the player
            
            // Show Game Over message
            finalScoreDisplay.textContent = `Score: ${score}`;
            finalScoreDisplay.classList.remove('hidden');
            startButton.textContent = 'PLAY AGAIN';
            startMessage.classList.remove('hidden');
            
            console.log("Game Over! Score:", score);
        }

        // --- Initialization ---
        // This function ensures all event listeners and initial setup happens after the page is fully loaded.
        function initializeGame() {
            // Display initial highscore
            highScoreDisplay.textContent = `HIGH: ${highscore}`;

            // Attach listeners
            startButton.addEventListener('click', handleTapOrClick);
            gameContainer.addEventListener('click', handleTapOrClick);
            document.addEventListener('keydown', (e) => {
                if (e.code === 'Space' || e.code === 'ArrowUp') {
                    e.preventDefault();
                    handleTapOrClick();
                }
            });

            // Set initial player position
            resetGame();
        }

        window.onload = initializeGame;

        window.addEventListener('resize', () => {
             if (gameState === GAME_STATE.PLAYING) {
                endGame();
             }
             resetGame();
        });
        
    </script>
</body>
</html>

