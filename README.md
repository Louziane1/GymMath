<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Viking Snatch Test Timer</title>
    <style>
        body {
            font-family: 'Arial Black', Arial, sans-serif;
            background: linear-gradient(135deg, #1a1a2e, #16213e);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            color: white;
        }

        .timer-container {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
            backdrop-filter: blur(10px);
            border: 2px solid rgba(255, 255, 255, 0.2);
            transition: all 0.3s ease;
        }

        .title {
            font-size: 2.5em;
            margin-bottom: 30px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            color: #ffffff;
        }

        .set-display {
            font-size: 3em;
            margin-bottom: 20px;
            font-weight: bold;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.7);
        }

        .main-timer {
            font-size: 6em;
            font-weight: bold;
            margin: 30px 0;
            text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.7);
            transition: all 0.3s ease;
        }

        .phase-indicator {
            font-size: 2em;
            margin: 20px 0;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
        }

        .controls {
            margin-top: 30px;
        }

        .btn {
            font-size: 1.5em;
            padding: 15px 30px;
            margin: 10px;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
            transition: all 0.3s ease;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
        }

        .start-btn {
            background: linear-gradient(45deg, #4CAF50, #45a049);
            color: white;
        }

        .start-btn:hover {
            background: linear-gradient(45deg, #45a049, #4CAF50);
            transform: translateY(-2px);
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
        }

        .reset-btn {
            background: linear-gradient(45deg, #f44336, #d32f2f);
            color: white;
        }

        .reset-btn:hover {
            background: linear-gradient(45deg, #d32f2f, #f44336);
            transform: translateY(-2px);
            box-shadow: 0 8px 16px rgba(0, 0, 0, 0.4);
        }

        .sound-toggle {
            background: linear-gradient(45deg, #ff9800, #f57c00);
            color: white;
            font-size: 1.2em;
            padding: 10px 20px;
        }

        .sound-toggle:hover {
            background: linear-gradient(45deg, #f57c00, #ff9800);
            transform: translateY(-2px);
        }

        /* Color phases */
        .green-phase {
            background: linear-gradient(135deg, #2e7d32, #388e3c);
            color: white;
        }

        .red-phase {
            background: linear-gradient(135deg, #c62828, #d32f2f);
            color: white;
        }

        .yellow-phase {
            background: linear-gradient(135deg, #f57f17, #f9a825);
            color: black;
        }

        .countdown-phase {
            background: linear-gradient(135deg, #1565c0, #1976d2);
            color: white;
        }

        .final-phase {
            background: linear-gradient(135deg, #4a148c, #6a1b9a);
            color: white;
        }

        .end-phase {
            background: linear-gradient(135deg, #ff6f00, #ff8f00);
            color: white;
        }

        .progress-bar {
            width: 100%;
            height: 10px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 5px;
            margin: 20px 0;
            overflow: hidden;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(45deg, #4CAF50, #8BC34A);
            transition: width 0.1s ease;
            border-radius: 5px;
        }

        .rep-counter {
            font-size: 2em;
            margin: 15px 0;
            opacity: 0.8;
        }
    </style>
</head>
<body>
    <div class="timer-container" id="timerContainer">
        <div class="title">VIKING SNATCH TEST</div>
        <div class="set-display" id="setDisplay">READY TO START</div>
        <div class="main-timer" id="mainTimer">5:00</div>
        <div class="phase-indicator" id="phaseIndicator">PRESS START</div>
        <div class="progress-bar">
            <div class="progress-fill" id="progressFill" style="width: 0%"></div>
        </div>
        <div class="rep-counter" id="repCounter"></div>
        <div class="controls">
            <button class="btn start-btn" id="startBtn" onclick="startTimer()">START</button>
            <button class="btn reset-btn" id="resetBtn" onclick="resetTimer()">RESET</button>
            <button class="btn sound-toggle" id="soundBtn" onclick="toggleSound()">SOUND: ON</button>
        </div>
    </div>

    <script>
        let isRunning = false;
        let currentSet = 1;
        let currentRep = 0;
        let totalTime = 300; // 5 minutes in seconds
        let timeRemaining = totalTime;
        let phaseTime = 0;
        let timerInterval;
        let soundEnabled = true;
        let lastPhase = '';
        
        // Rep intervals for each minute
        const repIntervals = [10.0, 4.2, 3.3, 2.7]; // seconds between reps for minutes 1-4
        
        const timerContainer = document.getElementById('timerContainer');
        const setDisplay = document.getElementById('setDisplay');
        const mainTimer = document.getElementById('mainTimer');
        const phaseIndicator = document.getElementById('phaseIndicator');
        const progressFill = document.getElementById('progressFill');
        const repCounter = document.getElementById('repCounter');
        const startBtn = document.getElementById('startBtn');
        const soundBtn = document.getElementById('soundBtn');

        // Create audio context for beep sound
        let audioContext;
        
        function initAudio() {
            if (!audioContext) {
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function playBeep() {
            if (!soundEnabled || !audioContext) return;
            
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            oscillator.frequency.value = 800; // High pitched beep
            oscillator.type = 'sine';
            
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.2);
            
            oscillator.start(audioContext.currentTime);
            oscillator.stop(audioContext.currentTime + 0.2);
        }

        function toggleSound() {
            soundEnabled = !soundEnabled;
            soundBtn.textContent = soundEnabled ? 'SOUND: ON' : 'SOUND: OFF';
            soundBtn.style.background = soundEnabled ? 
                'linear-gradient(45deg, #4CAF50, #45a049)' : 
                'linear-gradient(45deg, #757575, #616161)';
        }

        function updateDisplay() {
            const minutes = Math.floor(timeRemaining / 60);
            const seconds = timeRemaining % 60;
            mainTimer.textContent = `${minutes}:${seconds.toString().padStart(2, '0')}`;
            
            // Update progress bar
            const progress = ((totalTime - timeRemaining) / totalTime) * 100;
            progressFill.style.width = `${progress}%`;
        }

        function startTimer() {
            if (isRunning) return;
            
            // Initialize audio on first user interaction
            initAudio();
            
            isRunning = true;
            startBtn.textContent = 'RUNNING';
            startBtn.disabled = true;
            
            // Start with 3-second countdown
            startCountdown();
        }

        function startCountdown() {
            let countdown = 3;
            timerContainer.className = 'timer-container countdown-phase';
            setDisplay.textContent = 'GET READY';
            phaseIndicator.textContent = 'STARTING IN';
            lastPhase = 'countdown';
            
            const countdownInterval = setInterval(() => {
                mainTimer.textContent = countdown.toString();
                countdown--;
                
                if (countdown < 0) {
                    clearInterval(countdownInterval);
                    startMainTimer();
                }
            }, 1000);
        }

        function startMainTimer() {
            currentSet = 1;
            currentRep = 1;
            timeRemaining = 300;
            phaseTime = 0;
            
            timerInterval = setInterval(() => {
                timeRemaining--;
                phaseTime++;
                
                if (timeRemaining <= 0) {
                    endTest();
                    return;
                }
                
                if (currentSet <= 4) {
                    handleWorkingSets();
                } else {
                    handleFinalSet();
                }
                
                updateDisplay();
            }, 1000);
        }

        function handleWorkingSets() {
            const setStartTime = (currentSet - 1) * 60;
            const setElapsed = (300 - timeRemaining) - setStartTime;
            const currentInterval = repIntervals[currentSet - 1];
            const repCycle = setElapsed % currentInterval;
            const currentRepInSet = Math.floor(setElapsed / currentInterval) + 1;
            const totalRepsInMinute = Math.floor(60 / currentInterval);
            
            setDisplay.textContent = `SET ${currentSet} (${currentInterval}s intervals)`;
            repCounter.textContent = `Rep ${currentRepInSet} of ${totalRepsInMinute}`;
            
            // Calculate phase timing based on interval
            const redTime = Math.max(1, currentInterval - 3); // Most of interval is red (rest)
            const yellowTime = Math.min(3, currentInterval - 0.5); // Last 3 seconds (or less) yellow
            
            let currentPhase;
            
            if (repCycle < 0.5) {
                // GREEN - Time to do a rep (very brief)
                currentPhase = 'green';
                timerContainer.className = 'timer-container green-phase';
                phaseIndicator.textContent = 'SNATCH NOW!';
                
                // Play beep when entering green phase
                if (lastPhase !== 'green') {
                    playBeep();
                }
            } else if (repCycle < redTime) {
                // RED - Rest period
                currentPhase = 'red';
                timerContainer.className = 'timer-container red-phase';
                const timeToNext = Math.ceil(currentInterval - repCycle);
                phaseIndicator.textContent = `REST - ${timeToNext}`;
            } else {
                // YELLOW - Get ready
                currentPhase = 'yellow';
                timerContainer.className = 'timer-container yellow-phase';
                const timeToNext = Math.ceil(currentInterval - repCycle);
                phaseIndicator.textContent = `GET READY - ${timeToNext}`;
            }
            
            lastPhase = currentPhase;
            
            // Move to next set after 60 seconds
            if (setElapsed >= 60) {
                currentSet++;
                if (currentSet > 4) {
                    currentSet = 5;
                }
            }
        }

        function handleFinalSet() {
            // Set 5 - As many reps as possible
            timerContainer.className = 'timer-container final-phase';
            setDisplay.textContent = 'SET 5 - MAX REPS';
            phaseIndicator.textContent = 'GO FOR IT!';
            repCounter.textContent = `${timeRemaining} seconds remaining`;
            lastPhase = 'final';
        }

        function endTest() {
            clearInterval(timerInterval);
            timerContainer.className = 'timer-container end-phase';
            setDisplay.textContent = 'TEST COMPLETE!';
            mainTimer.textContent = '0:00';
            phaseIndicator.textContent = 'STOP - WELL DONE!';
            repCounter.textContent = 'Viking Test Finished!';
            
            startBtn.textContent = 'START';
            startBtn.disabled = false;
            isRunning = false;
            lastPhase = 'end';
        }

        function resetTimer() {
            clearInterval(timerInterval);
            isRunning = false;
            currentSet = 1;
            currentRep = 0;
            totalTime = 300;
            timeRemaining = totalTime;
            phaseTime = 0;
            lastPhase = '';
            
            timerContainer.className = 'timer-container';
            setDisplay.textContent = 'READY TO START';
            mainTimer.textContent = '5:00';
            phaseIndicator.textContent = 'PRESS START';
            progressFill.style.width = '0%';
            repCounter.textContent = '';
            
            startBtn.textContent = 'START';
            startBtn.disabled = false;
        }

        // Initialize display
        updateDisplay();
    </script>
</body>
</html>
