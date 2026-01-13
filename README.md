<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tapflo BINGO 2026</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); margin: 0; padding: 20px; min-height: 100vh; color: #333; }
        .container { max-width: 600px; margin: 0 auto; background: white; border-radius: 20px; box-shadow: 0 20px 40px rgba(0,0,0,0.1); overflow: hidden; }
        h1 { text-align: center; background: #ff6b6b; color: white; margin: 0; padding: 20px; font-size: 1.5em; }
        .bingo-grid { display: grid; grid-template-columns: repeat(5, 1fr); gap: 8px; padding: 16px; }
        .cell { 
            aspect-ratio: 1; border: 2px solid #ddd; border-radius: 10px; display: flex; align-items: center; justify-content: center; 
            cursor: pointer; transition: all 0.3s; position: relative; overflow: hidden; text-align: center; padding: 2px 4px; 
            line-height: 1.05; font-weight: 500; font-size: clamp(0.45em, 2.2vw, 0.75em) !important; max-width: 100%; 
            word-wrap: break-word; overflow-wrap: break-word; hyphens: auto; white-space: normal; background: white;
        }
        .cell:hover { transform: scale(1.03); box-shadow: 0 4px 12px rgba(0,0,0,0.15); }
        .cell.marked { background: linear-gradient(135deg, #4ecdc4, #44a08d); color: white; border-color: #44a08d; animation: pop 0.3s; }
        .cell.free { background: linear-gradient(135deg, #ffd93d, #ff6b6b); color: #333; border-color: #ff6b6b; font-weight: bold; font-size: clamp(0.55em, 2.5vw, 0.8em) !important; cursor: default; }
        @keyframes pop { 0% { transform: scale(0.9); } 50% { transform: scale(1.1); } 100% { transform: scale(1); } }
        .controls { padding: 20px; text-align: center; background: #f8f9fa; }
        button { background: #ff6b6b; color: white; border: none; padding: 16px 32px; margin: 8px; border-radius: 25px; font-size: 1.1em; cursor: pointer; transition: all 0.3s; font-weight: 600; }
        button:hover { background: #ff5252; transform: translateY(-3px); box-shadow: 0 8px 20px rgba(255,107,107,0.4); }
        button:disabled { background: #ccc; cursor: not-allowed; transform: none; }
        .status { text-align: center; font-size: 1.3em; font-weight: bold; margin: 15px 0; min-height: 40px; padding: 12px; border-radius: 12px; background: rgba(255,255,255,0.8); }
        .bingo-text { color: #4ecdc4; animation: glow 1s infinite alternate; }
        .full-bingo { background: linear-gradient(135deg, #ffd700, #ffed4e); color: #333; border: 3px solid #ffaa00; animation: full-bingo-celebration 2s infinite; font-size: 1.4em !important; }
        @keyframes glow { from { text-shadow: 0 0 5px #4ecdc4; } to { text-shadow: 0 0 20px #4ecdc4; } }
        @keyframes full-bingo-celebration { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
        @media (max-width: 500px) { .bingo-grid { padding: 12px; gap: 6px; } .cell { padding: 1px 2px; font-size: clamp(0.4em, 2.8vw, 0.65em) !important; } h1 { font-size: 1.3em; } }
        @media (max-width: 400px) { .cell { font-size: clamp(0.35em, 3.2vw, 0.6em) !important; } .bingo-grid { gap: 4px; padding: 10px; } }
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
            "%Musimy działać bardziej proaktywnie w tym obszarze.",
            "%To jest temat strategiczny z perspektywy zarządu.",
            "%Zastanówmy się, jak możemy to zeskalić na inne rynki.",
            "%Musimy podejść do tego bardziej holistycznie.",
            "%Zróbmy deep dive i wrócimy z konkretną rekomendacją.",
            "%Na ten moment zostawmy to, do tematu wrócimy w kolejnym kwartale.",
            "Dziękuję Ci za to pytanie.",
            "A który to Andrzej?",
            "Mam pytanie, tak spontanicznie.",
            "I co tam? Jesz?",
            "Krawiec zadaje pytanie po polsku do CEO",
            "Daniel 'sprzedałem ostatnio CTX-a'",
            "Koliński narzeka za zbyt duży budżet",
            "Białystok ma za duży budżet",
            "KWASIAK",
            "mapowanie procesów",
            "tego nie da się sprzedawać",
            "ewolucja a nie rewolucja",
            "konkurencja ma taniej",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
            "test",
			
            // Dodaj tutaj więcej haseł - gra losowo wybierze 24!
        ];
        // ====================================

        let markedCells = new Set();
        const TOTAL_CELLS = 24;
        const STORAGE_KEY = 'tapflo_bingo_2026';
        const ORDER_KEY = 'tapflo_bingo_2026_order';

        if (phrases.length < 24) {
            console.error('⚠️ Potrzebujesz minimum 24 haseł w tablicy `phrases`!');
        }

        // 📱 ZAPIS POSTĘPÓW 📱
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
            
            if (text.length > 60) baseFontSize *= 0.65;
            else if (text.length > 50) baseFontSize *= 0.75;
            else if (text.length > 40) baseFontSize *= 0.85;
            else if (text.length > 30) baseFontSize *= 0.92;
            
            const finalSize = Math.max(baseFontSize * 0.88, 7);
            cell.style.fontSize = finalSize + 'px';
            
            if (text.length > 45) {
                cell.style.fontWeight = '400';
                cell.style.lineHeight = '0.95';
                cell.style.letterSpacing = '-0.02em';
            }
        }

        function createCell(i, orderedPhrases) {
            const cell = document.createElement('div');
            cell.className = 'cell';
            cell.dataset.index = i;
            
            if (i === 12) { // FREE SPACE
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
            if (savedOrder) {
                return JSON.parse(savedOrder);
            } else {
                const indices = [];
                const available = [...Array(phrases.length).keys()];
                while (indices.length < 24) {
                    const randIdx = Math.floor(Math.random() * available.length);
                    indices.push(available.splice(randIdx, 1)[0]);
                }
                localStorage.setItem(ORDER_KEY, JSON.stringify(indices));
                return indices;
            }
        }

        function initGrid() {
            if (phrases.length < 24) {
                document.getElementById('status').textContent = '⚠️ Dodaj minimum 24 hasła do tablicy `phrases` w kodzie!';
                return;
            }

            const order = getOrCreateOrder();
            if (!order) return;

            const orderedPhrases = order.map(i => phrases[i]);
            const container = document.getElementById('grid');
            container.innerHTML = '';

            for (let i = 0; i < 25; i++) {
                const cell = createCell(i, orderedPhrases);
                container.appendChild(cell);
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
            document.getElementById('status').className = 'status';
        }

        function updateStatus() {
            const markedCount = markedCells.size;
            const statusEl = document.getElementById('status');
            if (markedCount === TOTAL_CELLS + 1) {
                statusEl.innerHTML = '🏆 DZIĘKUJĘ CI ZA TĘ GRĘ! Wygrywasz CTX-a i roczny wjazd na Podlasie! 🏆';
                statusEl.className = 'status full-bingo';
            } else {
                statusEl.textContent = markedCount >= 12 ? `Zaznaczono ${markedCount-1} pól (FREE)` : 'Zaznaczaj frazy!';
                statusEl.className = 'status';
            }
        }

        // 🎲 START 🎲
        initGrid();
    </script>
</body>
</html>
