<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tapflo BINGO 2026</title>
    <style>
        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body { 
            font-family: system-ui, -apple-system, 'Segoe UI', sans-serif; 
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); 
            margin: 0; padding: 10px; min-height: 100vh; color: #333; 
            overscroll-behavior: none; /* blokuje zoom/pull-to-refresh */
        }
        .container { 
            max-width: 100%; width: 100%; margin: 0; background: white; 
            border-radius: 20px; box-shadow: 0 15px 35px rgba(0,0,0,0.15); 
            overflow: hidden; 
        }
        h1 { 
            text-align: center; background: linear-gradient(135deg, #ff6b6b, #ff8e8e); 
            color: white; margin: 0; padding: 18px 10px; 
            font-size: clamp(1.2em, 5vw, 1.6em); font-weight: 700; 
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
        }
        .status { 
            text-align: center; font-size: clamp(1em, 4vw, 1.2em); font-weight: 700; 
            margin: 12px 0; min-height: 45px; padding: 12px 16px; 
            border-radius: 15px; background: rgba(255,255,255,0.95); 
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        }
        .bingo-grid { 
            display: grid; grid-template-columns: repeat(5, 1fr); 
            gap: clamp(4px, 2vw, 8px); padding: clamp(10px, 4vw, 18px); 
            justify-items: center;
        }
        .cell { 
            aspect-ratio: 1; 
            border: 2.5px solid #ddd; border-radius: 12px; 
            display: flex; align-items: center; justify-content: center; 
            cursor: pointer; touch-action: manipulation; /* mobile-friendly */
            transition: all 0.25s ease; position: relative; overflow: hidden; 
            text-align: center; padding: clamp(2px, 1vw, 4px); 
            line-height: 1.02; font-weight: 600; 
            font-size: clamp(0.38em, 2.1vw, 0.7em) !important; 
            max-width: 100%; min-height: 0; /* flex-grow fix */
            word-break: break-word; hyphens: auto; white-space: normal; 
            background: #fff; box-shadow: inset 0 1px 3px rgba(0,0,0,0.05);
        }
        .cell:active { transform: scale(0.96) !important; transition: 0.1s; }
        .cell:hover:not(.free) { transform: scale(1.02); box-shadow: 0 6px 16px rgba(0,0,0,0.2); }
        .cell.marked { 
            background: linear-gradient(135deg, #4ecdc4, #44a08d); 
            color: white; border-color: #44a08d; 
            animation: pop 0.3s ease-out; box-shadow: 0 4px 12px rgba(68,160,141,0.4);
        }
        .cell.free { 
            background: linear-gradient(135deg, #ffd93d, #ff6b6b); 
            color: #333; border-color: #ff6b6b; font-weight: 800; 
            font-size: clamp(0.48em, 2.4vw, 0.75em) !important; 
            cursor: default; box-shadow: 0 4px 12px rgba(255,107,107,0.3);
        }
        @keyframes pop { 
            0% { transform: scale(0.92); } 50% { transform: scale(1.08); } 100% { transform: scale(1); } 
        }
        .controls { 
            padding: 20px 16px; text-align: center; background: #f8f9fa; 
            display: flex; flex-wrap: wrap; gap: 12px; justify-content: center;
        }
        button { 
            background: linear-gradient(135deg, #ff6b6b, #ff5252); 
            color: white; border: none; padding: clamp(14px, 4vw, 18px) 28px; 
            border-radius: 28px; font-size: clamp(1em, 3.5vw, 1.1em); 
            cursor: pointer; transition: all 0.3s; font-weight: 700; 
            min-height: 52px; min-width: 140px; /* touch target */
            box-shadow: 0 4px 12px rgba(255,107,107,0.3);
            touch-action: manipulation;
        }
        button:hover, button:active { 
            background: linear-gradient(135deg, #ff5252, #ff4757); 
            transform: translateY(-2px); box-shadow: 0 8px 20px rgba(255,107,107,0.5); 
        }
        button:disabled { background: #ccc; cursor: not-allowed; transform: none; box-shadow: none; }
        .bingo-text { color: #4ecdc4; animation: glow 1s infinite alternate; }
        .full-bingo { 
            background: linear-gradient(135deg, #ffd700, #ffed4e); 
            color: #333; border: 4px solid #ffaa00; 
            animation: full-bingo-celebration 2s infinite; 
            font-size: clamp(1.1em, 4.5vw, 1.35em) !important; 
            box-shadow: 0 8px 25px rgba(255,215,0,0.6);
        }
        @keyframes glow { from { text-shadow: 0 0 8px #4ecdc4; } to { text-shadow: 0 0 20px #4ecdc4; } }
        @keyframes full-bingo-celebration { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.04); } }

        /* Mobile-first breakpoints */
        @media (max-width: 480px) { 
            body { padding: 8px; } 
            .bingo-grid { padding: 8px; gap: 3px; } 
            .cell { padding: 1px 2px; border-width: 2px; border-radius: 10px; }
            .controls { padding: 16px 12px; flex-direction: column; }
            button { width: 100%; max-width: 280px; }
        }
        @media (max-width: 360px) { 
            .cell { font-size: clamp(0.32em, 2.8vw, 0.6em) !important; padding: 0px 1px; }
            .bingo-grid { gap: 2px; padding: 6px; }
            h1 { padding: 15px 8px; }
        }
        @media (max-width: 320px) { 
            body { padding: 5px; } 
            .cell { font-size: clamp(0.3em, 3vw, 0.55em) !important; }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎰🎲 Tapflo BINGO 2026 🎯<br>🔥 Spotkanie Kwartalne 🎪</h1>
        <div class="status" id="status">Gotowy do gry! Zaznaczaj frazy...</div>
        <div class="bingo-grid" id="grid"></div>
        <div class="controls">
            <button onclick="newGame()">🎲 Nowa Gra</button>
            <button onclick="checkBingo()">✅ Sprawdź BINGO</button>
        </div>
    </div>

    <script>
        // === WPISZ TUTAJ SWOJE HASŁA (dowolna ilość, gra wybierze 24 losowe) ===
        const phrases = [
            "Musimy działać bardziej proaktywnie w tym obszarze.",
            "Wykorzystajmy nasze core kompetencje i synergię między działami.",
            "To jest temat strategiczny z perspektywy zarządu.",
            "Potrzebujemy lepszej egzekucji i dowiezienia rezultatów.",
            "Zastanówmy się, jak możemy to zeskalić na inne rynki.",
            "Musimy podejść do tego bardziej holistycznie, nie w silosach.",
            "Zróbmy deep dive i wrócimy z konkretną rekomendacją.",
            "To jest temat, który realnie może przesunąć nam wskazówkę na wynikach.",
            "Na ten moment zostawmy to, do tematu wrócimy w kolejnym kwartale.",
            "Dziękuję Ci za to pytanie",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test"
            // Dodaj więcej!
        ];
        // ====================================

        let markedCells = new Set();
        const TOTAL_CELLS = 24;
        const STORAGE_KEY = 'tapflo_bingo_2026';
        const ORDER_KEY = 'tapflo_bingo_2026_order';

        if (phrases.length < 24) {
            console.error('⚠️ Potrzebujesz minimum 24 haseł!');
        }

        function saveProgress() {
            const progress = Array.from(markedCells);
            localStorage.setItem(STORAGE_KEY, JSON.stringify(progress));
        }

        function loadProgress() {
            const saved = localStorage.getItem(STORAGE_KEY);
            if (saved) {
                const progress = new Set(JSON.parse(saved));
                markedCells = progress;
                document.querySelectorAll('.cell').forEach(cell => {
                    const index = parseInt(cell.dataset.index);
                    if (markedCells.has(index)) {
                        cell.classList.add('marked');
                    }
                });
                updateStatus();
            }
        }

        function clearProgress() {
            localStorage.removeItem(STORAGE_KEY);
            markedCells.clear();
        }

        function fitTextToCell(cell) {
            const text = cell.textContent;
            let baseFontSize = parseFloat(getComputedStyle(cell).fontSize);
            if (text.length > 55) baseFontSize *= 0.7;
            else if (text.length > 45) baseFontSize *= 0.8;
            else if (text.length > 35) baseFontSize *= 0.9;
            const finalSize = Math.max(baseFontSize * 0.92, 6);
            cell.style.fontSize = finalSize + 'px';
            if (text.length > 40) {
                cell.style.fontWeight = '500';
                cell.style.lineHeight = '0.98';
                cell.style.letterSpacing = '-0.015em';
            }
        }

        // Reszta funkcji bez zmian (createCell, getOrCreateOrder, etc.) – identyczna jak poprzednio
        function createCell(i, orderedPhrases) {
            const cell = document.createElement('div');
            cell.className = 'cell';
            cell.dataset.index = i;
            
            if (i === 12) {
                cell.textContent = '🎁 WOLNY LOS';
                cell.classList.add('free');
                if (!markedCells.has(12)) markedCells.add(12);
            } else {
                const phraseIndex = i > 12 ? i - 1 : i;
                cell.textContent = orderedPhrases[phraseIndex];
                cell.onclick = () => toggleCell(i);
            }
            
            fitTextToCell(cell);
            return cell;
        }

        function getOrCreateOrder() {
            if (phrases.length < 24) return null;
            const savedOrder = localStorage.getItem(ORDER_KEY);
            if (savedOrder) return JSON.parse(savedOrder);
            const indices = [];
            const available = [...Array(phrases.length).keys()];
            while (indices.length < 24) {
                const randIdx = Math.floor(Math.random() * available.length);
                indices.push(available.splice(randIdx, 1)[0]);
            }
            localStorage.setItem(ORDER_KEY, JSON.stringify(indices));
            return indices;
        }

        function initGrid() {
            if (phrases.length < 24) {
                document.getElementById('status').textContent = '⚠️ Dodaj min. 24 hasła do `phrases`!';
                return;
            }
            const order = getOrCreateOrder();
            if (!order) return;
            const orderedPhrases = order.map(i => phrases[i]);
            const container = document.getElementById('grid');
            container.innerHTML = '';
            for (let i = 0; i < 25; i++) {
                container.appendChild(createCell(i, orderedPhrases));
            }
            loadProgress();
        }

        function toggleCell(index) {
            const cell = document.querySelector(`[data-index="${index}"]`);
            if (markedCells.has(index)) {
                markedCells.delete(index);
                cell.classList.remove('marked');
            } else {
                markedCells.add(index);
                cell.classList.add('marked');
            }
            saveProgress();
            updateStatus();
        }

        function checkBingo() {
            const bingos = checkLines();
            const allMarked = markedCells.size === TOTAL_CELLS + 1;
            const statusEl = document.getElementById('status');
            if (allMarked) {
                statusEl.innerHTML = '🏆 <strong>DZIĘKUJĘ CI ZA TĘ GRĘ!</strong><br>Wygrywasz CTX-a i roczny wjazd na Podlasie! 🏆';
                statusEl.className = 'status full-bingo';
            } else if (bingos > 0) {
                statusEl.innerHTML = `🎉 BINGO! ${bingos} linii! 🎉`;
                statusEl.className = 'status bingo-text';
            } else {
                statusEl.textContent = 'Jeszcze nie ma BINGO...';
                statusEl.className = 'status';
            }
        }

        function checkLines() {
            let count = 0;
            for (let row = 0; row < 5; row++) {
                if ([0,1,2,3,4].map(c => row*5+c).every(i => markedCells.has(i))) count++;
            }
            for (let col = 0; col < 5; col++) {
                if ([0,1,2,3,4].map(r => r*5+col).every(i => markedCells.has(i))) count++;
            }
            if ([0,6,12,18,24].every(i => markedCells.has(i))) count++;
            if ([4,8,12,16,20].every(i => markedCells.has(i))) count++;
            return count;
        }

        function newGame() {
            clearProgress();
            markedCells.clear();
            localStorage.removeItem(ORDER_KEY);
            initGrid();
            document.getElementById('status').textContent = '🎲 Nowa losowa gra!';
        }

        function updateStatus() {
            const markedCount = markedCells.size;
            const statusEl = document.getElementById('status');
            if (markedCount === TOTAL_CELLS + 1) {
                statusEl.innerHTML = '🏆 DZIĘKUJĘ CI ZA TĘ GRĘ! Wygrywasz CTX-a i roczny wjazd na Podlasie! 🏆';
                statusEl.className = 'status full-bingo';
            } else {
                statusEl.textContent = markedCount >= 12 ? `Zaznaczono ${markedCount-1} pól (FREE)` : 'Zaznaczaj frazy!';
            }
        }

        initGrid();
    </script>
</body>
</html>
