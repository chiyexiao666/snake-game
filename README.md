[index.html](https://github.com/user-attachments/files/31537647/index.html)
# snake-game<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>贪吃蛇·进化</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

    :root {
      --font-pixel: 'Press Start 2P', monospace;
      --font-ui: "Microsoft YaHei", "PingFang SC", "Noto Sans SC", "Segoe UI", system-ui, -apple-system, sans-serif;
      --text-muted: #c8d0dc;
      --text-muted-soft: #a8b4c4;
    }

    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      min-height: 100vh;
      overflow: hidden;
      font-family: var(--font-ui);
      color: #e0e0e0;
      background: #050510;
      image-rendering: pixelated;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
    }

    #starfield {
      position: fixed;
      inset: 0;
      z-index: 0;
    }

    .screen {
      position: relative;
      z-index: 1;
      min-height: 100vh;
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      padding: 2rem 1rem;
    }

    .screen.active { display: flex; }

    .title-main {
      font-family: var(--font-pixel);
      font-size: clamp(1rem, 4vw, 1.6rem);
      color: #4ade80;
      text-shadow: 0 0 20px rgba(74, 222, 128, 0.8), 3px 3px 0 #166534;
      margin-bottom: 3rem;
      letter-spacing: 4px;
      text-align: center;
      animation: titlePulse 3s ease-in-out infinite;
    }

    @keyframes titlePulse {
      0%, 100% { text-shadow: 0 0 20px rgba(74, 222, 128, 0.8), 3px 3px 0 #166534; }
      50% { text-shadow: 0 0 40px rgba(74, 222, 128, 1), 0 0 60px rgba(74, 222, 128, 0.4), 3px 3px 0 #166534; }
    }

    .title-sub {
      font-family: var(--font-pixel);
      font-size: clamp(0.7rem, 2.5vw, 1rem);
      color: #60a5fa;
      text-shadow: 0 0 15px rgba(96, 165, 250, 0.6);
      margin-bottom: 2rem;
      text-align: center;
    }

    .menu-buttons {
      display: flex;
      flex-direction: column;
      gap: 1.2rem;
      align-items: center;
      width: 100%;
      max-width: 420px;
    }

    .glow-btn {
      font-family: var(--font-pixel);
      font-size: clamp(0.55rem, 2vw, 0.7rem);
      padding: 1rem 1.5rem;
      width: 100%;
      background: transparent;
      color: #e0e0e0;
      border: 2px solid #4ade80;
      cursor: pointer;
      position: relative;
      transition: all 0.2s;
      box-shadow: 0 0 15px rgba(74, 222, 128, 0.3), inset 0 0 15px rgba(74, 222, 128, 0.05);
      text-align: center;
      line-height: 1.6;
    }

    .glow-btn:hover:not(.locked) {
      background: rgba(74, 222, 128, 0.15);
      box-shadow: 0 0 30px rgba(74, 222, 128, 0.6), inset 0 0 20px rgba(74, 222, 128, 0.1);
      transform: scale(1.02);
    }

    .glow-btn:active:not(.locked) {
      transform: scale(0.98);
    }

    .glow-btn.blue {
      border-color: #60a5fa;
      box-shadow: 0 0 15px rgba(96, 165, 250, 0.3), inset 0 0 15px rgba(96, 165, 250, 0.05);
    }

    .glow-btn.blue:hover:not(.locked) {
      background: rgba(96, 165, 250, 0.15);
      box-shadow: 0 0 30px rgba(96, 165, 250, 0.6), inset 0 0 20px rgba(96, 165, 250, 0.1);
    }

    .glow-btn.locked {
      border-color: #4b5563;
      color: #9ca3af;
      cursor: not-allowed;
      box-shadow: none;
    }

    .glow-btn.back {
      font-family: var(--font-ui);
      border-color: #9ca3af;
      font-size: 0.875rem;
      font-weight: 500;
      margin-top: 0.5rem;
      box-shadow: 0 0 10px rgba(156, 163, 175, 0.2);
    }

    .glow-btn.back:hover {
      background: rgba(156, 163, 175, 0.1);
      box-shadow: 0 0 20px rgba(156, 163, 175, 0.3);
    }

    .lock-hint {
      font-family: var(--font-ui);
      font-size: 0.8125rem;
      font-weight: 400;
      color: var(--text-muted);
      margin-top: 0.5rem;
      display: block;
      line-height: 1.5;
      letter-spacing: 0.02em;
    }

    .glow-btn.locked .lock-hint {
      color: var(--text-muted-soft);
    }

    /* Game screen */
    .game-header {
      font-family: var(--font-pixel);
      display: flex;
      flex-wrap: wrap;
      gap: 1rem 2rem;
      justify-content: center;
      margin-bottom: 1rem;
      font-size: 0.6rem;
    }

    .game-header span { color: #fbbf24; }

    .game-wrapper {
      position: relative;
      border: 4px solid #4ade80;
      box-shadow: 0 0 0 4px #050510, 0 0 0 8px #4ade80, 0 0 40px rgba(74, 222, 128, 0.25);
      background: #0a0a18;
    }

    #gameCanvas {
      display: block;
      image-rendering: pixelated;
      image-rendering: crisp-edges;
    }

    .overlay {
      position: absolute;
      inset: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      background: rgba(5, 5, 16, 0.93);
      gap: 1rem;
      z-index: 10;
      padding: 1rem;
      text-align: center;
    }

    .overlay.hidden { display: none; }

    .overlay h2 {
      font-family: var(--font-pixel);
      font-size: 0.85rem;
      line-height: 1.8;
    }

    .overlay.win h2 {
      color: #4ade80;
      text-shadow: 0 0 20px rgba(74, 222, 128, 0.8);
    }

    .overlay.lose h2 {
      color: #f87171;
      text-shadow: 0 0 15px rgba(248, 113, 113, 0.6);
    }

    .overlay .info {
      font-family: var(--font-ui);
      font-size: 0.9375rem;
      font-weight: 500;
      color: #fbbf24;
      line-height: 1.8;
    }

    .overlay .info-sub {
      font-family: var(--font-ui);
      font-size: 0.8125rem;
      font-weight: 400;
      color: var(--text-muted);
      line-height: 1.7;
    }

    .overlay-btns {
      display: flex;
      flex-direction: column;
      gap: 0.8rem;
      margin-top: 0.5rem;
      width: 100%;
      max-width: 260px;
    }

    .overlay-btns .glow-btn {
      font-family: var(--font-ui);
      font-size: 0.875rem;
      font-weight: 500;
      padding: 0.7rem 1rem;
    }

    .hint {
      font-family: var(--font-ui);
      margin-top: 1rem;
      font-size: 0.8125rem;
      font-weight: 400;
      color: var(--text-muted);
      text-align: center;
      line-height: 1.6;
      letter-spacing: 0.02em;
    }

    .mode-desc {
      font-family: var(--font-ui);
      font-size: 0.875rem;
      font-weight: 400;
      color: var(--text-muted);
      text-align: center;
      max-width: 420px;
      line-height: 1.7;
      margin-bottom: 1.5rem;
      padding: 0.75rem 1rem;
      background: rgba(255, 255, 255, 0.04);
      border: 1px solid rgba(255, 255, 255, 0.08);
      border-radius: 6px;
      letter-spacing: 0.02em;
    }

    .overlay .prompt-title {
      font-family: var(--font-pixel);
      font-size: 0.85rem;
      color: #60a5fa;
      line-height: 1.8;
    }

    .unlock-msg {
      color: #4ade80;
      font-weight: 500;
    }
  </style>
</head>
<body>
  <canvas id="starfield"></canvas>

  <!-- 主菜单 -->
  <div id="mainMenu" class="screen active">
    <h1 class="title-main">贪吃蛇·进化</h1>
    <div class="menu-buttons">
      <button class="glow-btn" data-go="classicMenu">经典闯关</button>
      <button class="glow-btn blue" data-go="extremeMenu">极限挑战</button>
    </div>
  </div>

  <!-- 经典闯关子菜单 -->
  <div id="classicMenu" class="screen">
    <h2 class="title-sub">经典闯关</h2>
    <div class="menu-buttons">
      <button class="glow-btn" data-level="1">第一关：初入蛇穴</button>
      <button class="glow-btn locked" id="level2Btn" data-level="2">
        第二关：速度激情
        <span class="lock-hint">通关第一关解锁</span>
      </button>
      <button class="glow-btn locked" id="level3Btn" data-level="3">
        第三关：障碍迷阵
        <span class="lock-hint">通关第二关解锁</span>
      </button>
      <button class="glow-btn back" data-go="mainMenu">返回主菜单</button>
    </div>
  </div>

  <!-- 极限挑战子菜单 -->
  <div id="extremeMenu" class="screen">
    <h2 class="title-sub">极限挑战</h2>
    <p class="mode-desc">在无尽模式中分数超过 200 分解锁，或通关经典模式全部三关解锁</p>
    <div class="menu-buttons">
      <button class="glow-btn" data-mode="endless">无尽模式</button>
      <button class="glow-btn blue locked" id="hellBtn" data-mode="hell">
        地狱模式
        <span class="lock-hint" id="hellLockHint">尚未解锁</span>
      </button>
      <button class="glow-btn back" data-go="mainMenu">返回主菜单</button>
    </div>
  </div>

  <!-- 游戏界面 -->
  <div id="gameScreen" class="screen">
    <div class="game-header" id="gameHeader"></div>
    <div class="game-wrapper">
      <canvas id="gameCanvas" width="400" height="400"></canvas>
      <div id="pauseOverlay" class="overlay hidden">
        <h2 class="prompt-title">按方向键继续</h2>
        <p class="info-sub">使用 ↑ ↓ ← → 方向键开始移动</p>
      </div>
      <div id="resultOverlay" class="overlay hidden">
        <h2 id="resultTitle"></h2>
        <div id="resultInfo" class="info"></div>
        <div id="resultExtra" class="info-sub"></div>
        <div class="overlay-btns" id="resultBtns"></div>
      </div>
    </div>
    <p class="hint">使用 ↑ ↓ ← → 控制方向 · 按 ESC 返回菜单</p>
  </div>

  <script>
    // ========== 常量与配置 ==========
    const GRID = 20;
    const COLS = 20;
    const ROWS = 20;
    const STORAGE_KEY = 'snakeEvolution_v1';

    const LEVEL_CONFIG = {
      1: { name: '第一关：初入蛇穴', speed: 160, target: 50,  obstacles: 0 },
      2: { name: '第二关：速度激情', speed: 115, target: 100, obstacles: 0 },
      3: { name: '第三关：障碍迷阵', speed: 85,  target: 200, obstacles: 12 },
    };

    const ENDLESS_CONFIG = { baseSpeed: 160, accelEvery: 50, accelRate: 0.10 };
    const HELL_CONFIG     = { baseSpeed: 130, accelEvery: 30, accelRate: 0.15, obstacles: 14 };

    // ========== DOM ==========
    const starCanvas = document.getElementById('starfield');
    const starCtx = starCanvas.getContext('2d');
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const screens = {
      mainMenu: document.getElementById('mainMenu'),
      classicMenu: document.getElementById('classicMenu'),
      extremeMenu: document.getElementById('extremeMenu'),
      gameScreen: document.getElementById('gameScreen'),
    };
    const gameHeader = document.getElementById('gameHeader');
    const pauseOverlay = document.getElementById('pauseOverlay');
    const resultOverlay = document.getElementById('resultOverlay');
    const resultTitle = document.getElementById('resultTitle');
    const resultInfo = document.getElementById('resultInfo');
    const resultExtra = document.getElementById('resultExtra');
    const resultBtns = document.getElementById('resultBtns');

    // ========== 存档 ==========
    function loadSave() {
      try {
        return JSON.parse(localStorage.getItem(STORAGE_KEY)) || {};
      } catch { return {}; }
    }

    function getSave() {
      const d = loadSave();
      return {
        unlockedLevels: d.unlockedLevels || [1],
        hellUnlocked: !!d.hellUnlocked,
        endlessHighScore: d.endlessHighScore || 0,
        endlessMaxLength: d.endlessMaxLength || 0,
        hellHighScore: d.hellHighScore || 0,
        hellMaxLength: d.hellMaxLength || 0,
        classicComplete: !!d.classicComplete,
      };
    }

    function writeSave(data) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify({ ...loadSave(), ...data }));
    }

    // ========== 星空背景 ==========
    let stars = [];

    function initStars() {
      starCanvas.width = window.innerWidth;
      starCanvas.height = window.innerHeight;
      stars = Array.from({ length: 200 }, () => ({
        x: Math.random() * starCanvas.width,
        y: Math.random() * starCanvas.height,
        r: Math.random() * 1.8 + 0.3,
        speed: Math.random() * 0.4 + 0.1,
        alpha: Math.random() * 0.7 + 0.3,
      }));
    }

    function drawStars() {
      starCtx.fillStyle = '#050510';
      starCtx.fillRect(0, 0, starCanvas.width, starCanvas.height);
      stars.forEach(s => {
        starCtx.beginPath();
        starCtx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
        starCtx.fillStyle = `rgba(255,255,255,${s.alpha})`;
        starCtx.fill();
        s.y += s.speed;
        if (s.y > starCanvas.height) { s.y = 0; s.x = Math.random() * starCanvas.width; }
      });
      requestAnimationFrame(drawStars);
    }

    window.addEventListener('resize', initStars);
    initStars();
    drawStars();

    // ========== 屏幕导航 ==========
    function showScreen(name) {
      Object.values(screens).forEach(s => s.classList.remove('active'));
      screens[name].classList.add('active');
      if (name === 'classicMenu' || name === 'extremeMenu') refreshMenus();
    }

    function refreshMenus() {
      const save = getSave();
      [2, 3].forEach(lv => {
        const btn = document.getElementById(`level${lv}Btn`);
        const unlocked = save.unlockedLevels.includes(lv);
        btn.classList.toggle('locked', !unlocked);
        btn.querySelector('.lock-hint').textContent =
          unlocked ? '' : (lv === 2 ? '通关第一关解锁' : '通关第二关解锁');
      });
      const hellBtn = document.getElementById('hellBtn');
      hellBtn.classList.toggle('locked', !save.hellUnlocked);
      document.getElementById('hellLockHint').textContent =
        save.hellUnlocked ? '已解锁，点击开始挑战' : '无尽模式 200 分解锁，或通关经典三关';
    }

    document.querySelectorAll('[data-go]').forEach(btn => {
      btn.addEventListener('click', () => showScreen(btn.dataset.go));
    });

    document.querySelectorAll('[data-level]').forEach(btn => {
      btn.addEventListener('click', () => {
        if (btn.classList.contains('locked')) return;
        startGame('classic', parseInt(btn.dataset.level));
      });
    });

    document.querySelectorAll('[data-mode]').forEach(btn => {
      btn.addEventListener('click', () => {
        if (btn.classList.contains('locked')) return;
        startGame(btn.dataset.mode);
      });
    });

    // ========== 游戏引擎 ==========
    let snake, direction, nextDirection, food, obstacles, score;
    let gameLoop = null, gameState = 'idle', currentMode = null, currentLevel = null;
    let baseTick = 160, speedLevel = 1, tickMs = 160;
    let paused = true;

    function occupiedCells() {
      const cells = new Set();
      snake.forEach(s => cells.add(`${s.x},${s.y}`));
      if (food) cells.add(`${food.x},${food.y}`);
      obstacles.forEach(o => cells.add(`${o.x},${o.y}`));
      return cells;
    }

    function spawnFood() {
      const blocked = occupiedCells();
      let pos, tries = 0;
      do {
        pos = { x: Math.floor(Math.random() * COLS), y: Math.floor(Math.random() * ROWS) };
        tries++;
      } while (blocked.has(`${pos.x},${pos.y}`) && tries < 500);
      food = pos;
    }

    function spawnObstacles(count) {
      obstacles = [];
      const blocked = occupiedCells();
      let placed = 0, tries = 0;
      while (placed < count && tries < 1000) {
        tries++;
        const pos = { x: Math.floor(Math.random() * COLS), y: Math.floor(Math.random() * ROWS) };
        const key = `${pos.x},${pos.y}`;
        if (!blocked.has(key)) {
          obstacles.push(pos);
          blocked.add(key);
          placed++;
        }
      }
    }

    function resetSnake() {
      const sx = Math.floor(COLS / 2);
      const sy = Math.floor(ROWS / 2);
      snake = [{ x: sx, y: sy }, { x: sx - 1, y: sy }, { x: sx - 2, y: sy }];
      direction = { x: 1, y: 0 };
      nextDirection = { x: 1, y: 0 };
    }

    function initGameState(mode, level) {
      currentMode = mode;
      currentLevel = level;
      score = 0;
      speedLevel = 1;
      obstacles = [];
      resetSnake();

      if (mode === 'classic') {
        const cfg = LEVEL_CONFIG[level];
        baseTick = cfg.speed;
        tickMs = cfg.speed;
        if (cfg.obstacles > 0) spawnObstacles(cfg.obstacles);
      } else if (mode === 'endless') {
        baseTick = ENDLESS_CONFIG.baseSpeed;
        tickMs = ENDLESS_CONFIG.baseSpeed;
      } else if (mode === 'hell') {
        baseTick = HELL_CONFIG.baseSpeed;
        tickMs = HELL_CONFIG.baseSpeed;
        spawnObstacles(HELL_CONFIG.obstacles);
      }

      spawnFood();
      paused = true;
      gameState = 'playing';
      updateHeader();
      draw();
      pauseOverlay.classList.remove('hidden');
      resultOverlay.classList.add('hidden');
    }

    function updateHeader() {
      let html = `得分：<span>${score}</span>`;
      if (currentMode === 'classic') {
        html += ` &nbsp; 目标：<span>${LEVEL_CONFIG[currentLevel].target}</span>`;
      }
      if (currentMode === 'endless' || currentMode === 'hell') {
        html += ` &nbsp; 速度等级：<span>${speedLevel}</span>`;
        html += ` &nbsp; 蛇身长度：<span>${snake.length}</span>`;
      }
      gameHeader.innerHTML = html;
    }

    function applySpeedBoost() {
      let cfg, interval, rate;
      if (currentMode === 'endless') {
        cfg = ENDLESS_CONFIG;
      } else if (currentMode === 'hell') {
        cfg = HELL_CONFIG;
      } else return;

      interval = cfg.accelEvery;
      rate = cfg.accelRate;
      const newLevel = Math.floor(score / interval) + 1;
      if (newLevel > speedLevel) {
        speedLevel = newLevel;
        tickMs = Math.max(40, Math.round(baseTick * Math.pow(1 - rate, speedLevel - 1)));
        restartLoop();
      }
    }

    function restartLoop() {
      clearInterval(gameLoop);
      gameLoop = setInterval(tick, tickMs);
    }

    // ========== 渲染 ==========
    function drawGrid() {
      ctx.fillStyle = '#0a0a18';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.strokeStyle = '#151528';
      ctx.lineWidth = 1;
      for (let x = 0; x <= COLS; x++) {
        ctx.beginPath(); ctx.moveTo(x * GRID, 0); ctx.lineTo(x * GRID, canvas.height); ctx.stroke();
      }
      for (let y = 0; y <= ROWS; y++) {
        ctx.beginPath(); ctx.moveTo(0, y * GRID); ctx.lineTo(canvas.width, y * GRID); ctx.stroke();
      }
    }

    function drawObstacles() {
      obstacles.forEach(o => {
        ctx.fillStyle = '#6b7280';
        ctx.fillRect(o.x * GRID + 1, o.y * GRID + 1, GRID - 2, GRID - 2);
        ctx.fillStyle = '#4b5563';
        ctx.fillRect(o.x * GRID + 3, o.y * GRID + 3, GRID - 6, GRID - 6);
      });
    }

    function drawFood() {
      const p = 2;
      ctx.fillStyle = '#f87171';
      ctx.fillRect(food.x * GRID + p, food.y * GRID + p, GRID - p * 2, GRID - p * 2);
      ctx.fillStyle = '#fca5a5';
      ctx.fillRect(food.x * GRID + p + 2, food.y * GRID + p + 2, 4, 4);
    }

    function drawSnake() {
      snake.forEach((seg, i) => {
        const pad = i === 0 ? 1 : 2;
        ctx.fillStyle = i === 0 ? '#4ade80' : (i % 2 === 0 ? '#22c55e' : '#16a34a');
        ctx.fillRect(seg.x * GRID + pad, seg.y * GRID + pad, GRID - pad * 2, GRID - pad * 2);
        if (i === 0) {
          ctx.fillStyle = '#0a0a18';
          const e = 3;
          const bx = seg.x * GRID, by = seg.y * GRID;
          if (direction.x === 1) {
            ctx.fillRect(bx + 12, by + 5, e, e); ctx.fillRect(bx + 12, by + 12, e, e);
          } else if (direction.x === -1) {
            ctx.fillRect(bx + 5, by + 5, e, e); ctx.fillRect(bx + 5, by + 12, e, e);
          } else if (direction.y === -1) {
            ctx.fillRect(bx + 5, by + 5, e, e); ctx.fillRect(bx + 12, by + 5, e, e);
          } else {
            ctx.fillRect(bx + 5, by + 12, e, e); ctx.fillRect(bx + 12, by + 12, e, e);
          }
        }
      });
    }

    function draw() {
      drawGrid();
      drawObstacles();
      drawFood();
      drawSnake();
    }

    // ========== 游戏逻辑 ==========
    function tick() {
      if (paused || gameState !== 'playing') return;

      direction = { ...nextDirection };
      const head = { x: snake[0].x + direction.x, y: snake[0].y + direction.y };

      if (head.x < 0 || head.x >= COLS || head.y < 0 || head.y >= ROWS) {
        return onDeath();
      }
      if (snake.some(s => s.x === head.x && s.y === head.y)) {
        return onDeath();
      }
      if (obstacles.some(o => o.x === head.x && o.y === head.y)) {
        return onDeath();
      }

      snake.unshift(head);

      if (head.x === food.x && head.y === food.y) {
        score += 10;
        spawnFood();
        applySpeedBoost();
        if (currentMode === 'endless' && score >= 200 && !getSave().hellUnlocked) {
          writeSave({ hellUnlocked: true });
        }
        updateHeader();

        if (currentMode === 'classic' && score >= LEVEL_CONFIG[currentLevel].target) {
          return onVictory();
        }
      } else {
        snake.pop();
      }

      updateHeader();
      draw();
    }

    function onDeath() {
      gameState = 'over';
      clearInterval(gameLoop);
      pauseOverlay.classList.add('hidden');

      const save = getSave();
      let extra = '';

      if (currentMode === 'endless') {
        const justUnlockedHell = score >= 200 && !save.hellUnlocked;
        if (score > save.endlessHighScore) writeSave({ endlessHighScore: score });
        if (snake.length > save.endlessMaxLength) writeSave({ endlessMaxLength: snake.length });
        if (justUnlockedHell) writeSave({ hellUnlocked: true });
        const s = getSave();
        extra = `历史最高：${s.endlessHighScore} 分<br>最长蛇身：${s.endlessMaxLength} 节`;
        if (justUnlockedHell) extra += '<br><span class="unlock-msg">🔓 地狱模式已解锁！</span>';
      } else if (currentMode === 'hell') {
        if (score > save.hellHighScore) writeSave({ hellHighScore: score });
        if (snake.length > save.hellMaxLength) writeSave({ hellMaxLength: snake.length });
        const s = getSave();
        extra = `地狱最高：${s.hellHighScore} 分<br>最长蛇身：${s.hellMaxLength} 节`;
      }

      showResult(false, extra);
    }

    function onVictory() {
      gameState = 'win';
      clearInterval(gameLoop);
      pauseOverlay.classList.add('hidden');

      const save = getSave();
      let extra = '';
      let btns = [];

      if (currentLevel < 3) {
        const next = currentLevel + 1;
        if (!save.unlockedLevels.includes(next)) {
          writeSave({ unlockedLevels: [...save.unlockedLevels, next] });
        }
        extra = `下一关已解锁！`;
        btns = [
          { text: '下一关', action: () => startGame('classic', next) },
          { text: '返回关卡选择', action: () => { stopGame(); showScreen('classicMenu'); } },
        ];
      } else {
        writeSave({ classicComplete: true, hellUnlocked: true });
        extra = '全部经典关卡通关！<br><span class="unlock-msg">🔓 地狱模式已解锁！</span>';
        btns = [
          { text: '返回主菜单', action: () => { stopGame(); showScreen('mainMenu'); } },
        ];
      }

      showResult(true, extra, btns);
    }

    function showResult(win, extra, customBtns) {
      resultOverlay.classList.remove('hidden');
      resultOverlay.classList.toggle('win', win);
      resultOverlay.classList.toggle('lose', !win);

      if (win) {
        resultTitle.textContent = currentLevel === 3 ? '恭喜通关！' : '关卡胜利！';
        resultInfo.innerHTML = `得分：${score} / ${LEVEL_CONFIG[currentLevel].target}`;
      } else {
        resultTitle.textContent = '游戏结束';
        if (currentMode === 'classic') {
          resultInfo.innerHTML = `得分：${score} / ${LEVEL_CONFIG[currentLevel].target}`;
        } else {
          resultInfo.innerHTML = `最终得分：${score}<br>蛇身长度：${snake.length} 节`;
        }
      }

      resultExtra.innerHTML = extra || '';
      resultBtns.innerHTML = '';

      const btns = customBtns || getDefaultBtns(win);
      btns.forEach(b => {
        const btn = document.createElement('button');
        btn.className = 'glow-btn' + (b.primary ? '' : ' back');
        btn.textContent = b.text;
        btn.addEventListener('click', b.action);
        resultBtns.appendChild(btn);
      });
    }

    function getDefaultBtns(win) {
      if (currentMode === 'classic') {
        return win ? [] : [
          { text: '重新挑战', action: () => startGame('classic', currentLevel) },
          { text: '返回关卡选择', action: () => { stopGame(); showScreen('classicMenu'); } },
        ];
      }
      return [
        { text: '再来一局', action: () => startGame(currentMode) },
        { text: '返回模式选择', action: () => { stopGame(); showScreen('extremeMenu'); } },
      ];
    }

    function startGame(mode, level) {
      stopGame();
      showScreen('gameScreen');
      initGameState(mode, level);
      restartLoop();
    }

    function stopGame() {
      clearInterval(gameLoop);
      gameLoop = null;
      gameState = 'idle';
      paused = true;
      pauseOverlay.classList.add('hidden');
      resultOverlay.classList.add('hidden');
    }

    function unpause() {
      if (gameState !== 'playing' || !paused) return;
      paused = false;
      pauseOverlay.classList.add('hidden');
    }

    // ========== 输入 ==========
    document.addEventListener('keydown', e => {
      if (e.key === 'Escape') {
        if (screens.gameScreen.classList.contains('active') && gameState === 'playing') {
          stopGame();
          if (currentMode === 'classic') showScreen('classicMenu');
          else showScreen('extremeMenu');
        }
        return;
      }

      if (!['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) return;
      e.preventDefault();

      const map = {
        ArrowUp: { x: 0, y: -1 }, ArrowDown: { x: 0, y: 1 },
        ArrowLeft: { x: -1, y: 0 }, ArrowRight: { x: 1, y: 0 },
      };
      const nd = map[e.key];

      if (screens.gameScreen.classList.contains('active') && gameState === 'playing') {
        if (paused) {
          direction = { ...nd };
          nextDirection = { ...nd };
          unpause();
          return;
        }
        if (nd.x + direction.x === 0 && nd.y + direction.y === 0) return;
        nextDirection = nd;
      }
    });

    // 初始化菜单状态
    refreshMenus();
  </script>
</body>
</html>
