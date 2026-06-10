# 天天酷跑 HTML 游戏 — 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单 HTML 文件的"天天酷跑"像素风横版跑酷游戏，包含完整的开始/游戏/结算流程。

**Architecture:** 单文件 `index.html`，所有 CSS 内联 `<style>`，JS 内联 `<script>`。Canvas 2D 渲染，requestAnimationFrame 驱动 60fps 游戏循环。JS 内部使用对象命名空间划分模块（Renderer、Player、ObstacleManager、CoinManager、Collision、ParticleSystem、UI）。

**Tech Stack:** 纯 HTML5 + CSS3 + Canvas 2D + localStorage（最高分），无外部依赖（除 Google Fonts `Press Start 2P`）。

---

### Task 1: HTML 骨架 + CSS 样式 + UI 覆盖层

**Files:**
- Create: `e:/claude code learning/project_1/index.html`

此任务建立 HTML 基础结构，包括 Canvas 容器和三个 UI 覆盖层（开始画面/结算画面/HUD）。所有后续任务都在此文件内添加 JS 代码。

- [ ] **Step 1: 创建 HTML 基本骨架**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>天天酷跑</title>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
<style>
/* CSS 写在这里 */
</style>
</head>
<body>
<!-- HTML 结构写在这里 -->
<script>
// JS 写在这里
</script>
</body>
</html>
```

- [ ] **Step 2: 添加 CSS 样式**

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
    background: #1a1a2e;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    overflow: hidden;
    font-family: 'Press Start 2P', monospace;
}
#gameContainer {
    position: relative;
    width: 800px;
    height: 400px;
    border: 4px solid #333;
    border-radius: 4px;
    box-shadow: 0 0 30px rgba(0,0,0,0.5);
}
canvas {
    display: block;
    width: 800px;
    height: 400px;
    image-rendering: pixelated;
}
.overlay {
    position: absolute;
    top: 0; left: 0;
    width: 100%; height: 100%;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background: rgba(0,0,0,0.7);
    color: #fff;
    pointer-events: none;
    transition: opacity 0.3s;
}
.overlay.hidden { opacity: 0; pointer-events: none; }
.overlay h1 {
    font-size: 32px;
    color: #ffd700;
    text-shadow: 3px 3px 0 #8b4513;
    margin-bottom: 20px;
}
.overlay .subtitle {
    font-size: 12px;
    color: #aaa;
    margin-bottom: 30px;
    line-height: 2;
}
.overlay .hint {
    font-size: 10px;
    color: #ffd700;
    animation: blink 1s steps(1) infinite;
}
@keyframes blink { 50% { opacity: 0; } }
#gameOverOverlay .result-row {
    font-size: 12px;
    margin: 8px 0;
    color: #fff;
}
#gameOverOverlay .result-row span {
    color: #ffd700;
}
#gameOverOverlay .best-row {
    font-size: 10px;
    color: #aaa;
    margin-top: 15px;
}
```

- [ ] **Step 3: 添加 HTML UI 结构**

```html
<div id="gameContainer">
    <canvas id="gameCanvas" width="800" height="400"></canvas>

    <!-- 开始画面 -->
    <div id="startOverlay" class="overlay">
        <h1>天天酷跑</h1>
        <div class="subtitle">
            🏃 空格 = 跳跃<br>
            ⬇  ↓  = 下滑
        </div>
        <div class="hint">按 空格 开始</div>
    </div>

    <!-- 结算画面 -->
    <div id="gameOverOverlay" class="overlay hidden">
        <h1>游戏结束</h1>
        <div class="result-row">距离: <span id="finalDistance">0</span> 米</div>
        <div class="result-row">金币: <span id="finalCoins">0</span> 个</div>
        <div class="result-row">总分: <span id="finalScore">0</span></div>
        <div class="best-row">🏆 最高记录: <span id="bestScore">0</span> 米</div>
        <div class="hint" style="margin-top:25px">按 空格 重新开始</div>
    </div>
</div>
```

- [ ] **Step 4: 添加 JS 骨架（空游戏对象占位）**

```javascript
// ========== 游戏配置 ==========
const CONFIG = {
    WIDTH: 800,
    HEIGHT: 400,
    GROUND_HEIGHT: 80,
    BASE_SPEED: 6,
    MAX_SPEED: 20,
    SPEED_INCREASE_PER_10S: 0.5,
    GRAVITY: 0.5,
    JUMP_VELOCITY: -9,
    PLAYER_SCALE: 3,
};

// ========== DOM 引用 ==========
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const startOverlay = document.getElementById('startOverlay');
const gameOverOverlay = document.getElementById('gameOverOverlay');
const finalDistance = document.getElementById('finalDistance');
const finalCoins = document.getElementById('finalCoins');
const finalScore = document.getElementById('finalScore');
const bestScore = document.getElementById('bestScore');

// ========== 模块占位 ==========
const Renderer = {};
const Player = { x: 100, y: 0, width: 48, height: 48, state: 'IDLE', vy: 0, frame: 0, frameTimer: 0 };
const ObstacleManager = { obstacles: [], timer: 0, spawnInterval: 1.5 };
const CoinManager = { coins: [], timer: 0 };
const ParticleSystem = { particles: [] };
const Collision = {};
const UI = {};

// ========== 游戏主控 ==========
const Game = {
    state: 'MENU', // MENU | PLAYING | DEAD | GAME_OVER
    speed: CONFIG.BASE_SPEED,
    distance: 0,
    coins: 0,
    time: 0,
    highScore: parseInt(localStorage.getItem('tianTianPaoHighScore') || '0'),
    keys: { space: false, down: false },

    init() {
        this.highScore = parseInt(localStorage.getItem('tianTianPaoHighScore') || '0');
        this.setupInput();
        this.loop();
    },

    setupInput() {
        document.addEventListener('keydown', (e) => {
            if (e.key === ' ') { e.preventDefault(); this.keys.space = true; }
            if (e.key === 'ArrowDown') { e.preventDefault(); this.keys.down = true; }
        });
        document.addEventListener('keyup', (e) => {
            if (e.key === ' ') { e.preventDefault(); this.keys.space = false; }
            if (e.key === 'ArrowDown') { e.preventDefault(); this.keys.down = false; }
        });
    },

    loop() {
        this.update();
        this.render();
        requestAnimationFrame(() => this.loop());
    },

    update() {
        if (this.state === 'MENU' && this.keys.space) { this.startGame(); }
        if (this.state === 'PLAYING') { this.updateGame(); }
        if (this.state === 'GAME_OVER' && this.keys.space) { this.backToMenu(); }
    },

    startGame() {
        this.state = 'PLAYING';
        this.speed = CONFIG.BASE_SPEED;
        this.distance = 0;
        this.coins = 0;
        this.time = 0;
        Player.state = 'RUNNING';
        Player.y = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT - Player.height;
        Player.vy = 0;
        ObstacleManager.obstacles = [];
        ObstacleManager.timer = 0;
        CoinManager.coins = [];
        CoinManager.timer = 0;
        ParticleSystem.particles = [];
        startOverlay.classList.add('hidden');
        gameOverOverlay.classList.add('hidden');
    },

    updateGame() {
        this.time += 1/60;
        // 加速
        this.speed = Math.min(CONFIG.MAX_SPEED,
            CONFIG.BASE_SPEED + Math.floor(this.time / 10) * CONFIG.SPEED_INCREASE_PER_10S);
        // 距离
        this.distance += this.speed / 100;
        // 玩家更新
        PlayerUpdate.update();
        // 障碍更新
        ObstacleManagerUpdate.update(this.speed);
        // 金币更新
        CoinManagerUpdate.update(this.speed);
        // 碰撞检测
        if (CollisionCheck.check()) { this.die(); }
        // 粒子更新
        ParticleSystemUpdate.update();
    },

    die() {
        this.state = 'DEAD';
        Player.state = 'DEAD';
        // 1秒后显示结算
        setTimeout(() => {
            this.state = 'GAME_OVER';
            this.showGameOver();
        }, 1000);
    },

    showGameOver() {
        const dist = Math.floor(this.distance);
        const score = dist + this.coins * 10;
        finalDistance.textContent = dist;
        finalCoins.textContent = this.coins;
        finalScore.textContent = score;
        if (dist > this.highScore) {
            this.highScore = dist;
            localStorage.setItem('tianTianPaoHighScore', String(dist));
        }
        bestScore.textContent = this.highScore;
        gameOverOverlay.classList.remove('hidden');
    },

    backToMenu() {
        this.state = 'MENU';
        gameOverOverlay.classList.add('hidden');
        startOverlay.classList.remove('hidden');
        Player.state = 'IDLE';
    },

    render() {
        Renderer.clear();
        Renderer.drawBackground(this.time);
        Renderer.drawGround(this.speed);
        Renderer.drawObstacles();
        Renderer.drawCoins();
        Renderer.drawPlayer();
        Renderer.drawParticles();
        if (this.state === 'PLAYING') {
            Renderer.drawHUD();
        }
    }
};

// 启动游戏
Game.init();
```

注意：以上 Step 4 包含完整结构逻辑，但 PlayerUpdate、ObstacleManagerUpdate 等函数需要在后续任务中实现。当前步骤完成后，游戏应能运行并显示开始画面，按空格后进入空白游戏世界（除了背景和地面外无内容）。

---

### Task 2: 渲染器 — 背景、地面、HUD

**Files:**
- Modify: `e:/claude code learning/project_1/index.html`（在 `<script>` 中填充 Renderer 对象）

- [ ] **Step 1: 实现 Renderer.clear 和 Renderer.drawBackground**

```javascript
// ========== Renderer ==========
Renderer.clear = function() {
    ctx.clearRect(0, 0, CONFIG.WIDTH, CONFIG.HEIGHT);
};

Renderer.drawBackground = function(time) {
    // 天空渐变
    const skyGrad = ctx.createLinearGradient(0, 0, 0, CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT);
    skyGrad.addColorStop(0, '#4dc9f6');
    skyGrad.addColorStop(0.5, '#87ceeb');
    skyGrad.addColorStop(1, '#e0f0ff');
    ctx.fillStyle = skyGrad;
    ctx.fillRect(0, 0, CONFIG.WIDTH, CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT);

    // 云朵（视差层1，速度×0.2）
    const cloudSpeed = Game.speed * 0.2;
    Renderer.cloudOffset = (Renderer.cloudOffset || 0) + cloudSpeed;
    ctx.fillStyle = 'rgba(255,255,255,0.8)';
    for (let i = 0; i < 5; i++) {
        const cx = (i * 200 + 50 - Renderer.cloudOffset % 1000) % 1000 - 100;
        const cy = 40 + i * 25;
        drawCloud(cx, cy, 60 + i * 10);
    }
    function drawCloud(x, y, size) {
        const s = size / 60;
        ctx.fillRect(x, y, 40*s, 12*s);
        ctx.fillRect(x - 8*s, y + 8*s, 56*s, 10*s);
        ctx.fillRect(x + 12*s, y - 6*s, 20*s, 10*s);
    }

    // 远山（视差层2，速度×0.5）
    const mountSpeed = Game.speed * 0.5;
    Renderer.mountOffset = (Renderer.mountOffset || 0) + mountSpeed;
    ctx.fillStyle = '#6b8e9e';
    for (let i = 0; i < 6; i++) {
        const mx = (i * 180 - Renderer.mountOffset % 1080) % 1080 - 100;
        const baseY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        ctx.beginPath();
        ctx.moveTo(mx, baseY);
        ctx.lineTo(mx + 40, baseY - 60 - (i % 3) * 20);
        ctx.lineTo(mx + 80, baseY);
        ctx.fill();
    }
    ctx.fillStyle = '#5a7d8a';
    for (let i = 0; i < 5; i++) {
        const mx = (i * 200 + 80 - Renderer.mountOffset % 1000) % 1000 - 100;
        const baseY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        ctx.beginPath();
        ctx.moveTo(mx, baseY);
        ctx.lineTo(mx + 30, baseY - 35 - (i % 2) * 15);
        ctx.lineTo(mx + 60, baseY);
        ctx.fill();
    }
};
```

- [ ] **Step 2: 实现 Renderer.drawGround**

```javascript
Renderer.drawGround = function() {
    const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
    // 地面基底
    ctx.fillStyle = '#5d4a36';
    ctx.fillRect(0, groundY, CONFIG.WIDTH, CONFIG.GROUND_HEIGHT);
    // 草地表面
    ctx.fillStyle = '#4a8c3f';
    ctx.fillRect(0, groundY, CONFIG.WIDTH, 8);
    // 像素纹理（滚动的泥土块）
    const scrollX = (Renderer.groundOffset || 0) + Game.speed;
    Renderer.groundOffset = scrollX % 32;
    for (let x = -Renderer.groundOffset; x < CONFIG.WIDTH; x += 32) {
        ctx.fillStyle = (Math.floor(x / 32) % 2 === 0) ? '#6b5a44' : '#5d4a36';
        ctx.fillRect(x, groundY + 8, 32, 8);
        ctx.fillStyle = (Math.floor(x / 32) % 2 === 0) ? '#5d4a36' : '#4d3d2a';
        ctx.fillRect(x, groundY + 16, 32, CONFIG.GROUND_HEIGHT - 16);
    }
};
```

- [ ] **Step 3: 实现 Renderer.drawHUD**

```javascript
Renderer.drawHUD = function() {
    ctx.font = '14px "Press Start 2P", monospace';
    ctx.textBaseline = 'top';
    // 距离（左上）
    const dist = Math.floor(Game.distance);
    ctx.fillStyle = '#000';
    ctx.fillText(`距: ${dist}m`, 13, 13);
    ctx.fillStyle = '#fff';
    ctx.fillText(`距: ${dist}m`, 11, 11);
    // 金币（右上）
    ctx.fillStyle = '#000';
    ctx.fillText(`×${Game.coins}`, CONFIG.WIDTH - 80, 13);
    ctx.fillStyle = '#ffd700';
    ctx.fillText(`×${Game.coins}`, CONFIG.WIDTH - 82, 11);
    // 金币图标
    ctx.fillStyle = '#ffd700';
    ctx.fillRect(CONFIG.WIDTH - 102, 13, 12, 12);
    ctx.fillStyle = '#ffec8b';
    ctx.fillRect(CONFIG.WIDTH - 100, 15, 8, 8);
};
```

- [ ] **Step 4: 实现 Renderer.drawParticles**

```javascript
Renderer.drawParticles = function() {
    for (const p of ParticleSystem.particles) {
        const alpha = p.life / p.maxLife;
        ctx.globalAlpha = alpha;
        ctx.fillStyle = p.color;
        ctx.fillRect(p.x - p.size/2, p.y - p.size/2, p.size, p.size);
    }
    ctx.globalAlpha = 1;
};
```

---

### Task 3: 玩家系统 — 像素数据 + 物理 + 动画

**Files:**
- Modify: `e:/claude code learning/project_1/index.html`（替换 Player 对象 + 添加 PlayerUpdate 函数）

- [ ] **Step 1: 定义角色像素数据**

```javascript
// ========== 角色像素数据（16×16）==========
const PLAYER_PIXELS = {
    // 跑步帧1：右腿在前
    run1: [
        "....rrrrrr......",
        "...rrrrrrr....r.",
        "...rryyrrr...rr.",
        "..rryyyrrr..rr..",
        "..rrrrrrrrrrr...",
        "...rrrrrrrr.....",
        "...bbbbbbbb.....",
        "...bbbbbbbb.....",
        "....bbbbbb......",
        "....ww..ww......",
        "...wwww..ww.....",
        "...wwww..ww.....",
        "..rrww..wwrr....",
        ".rrrr....rrrr...",
        "................",
        "................",
    ],
    // 跑步帧2：左腿在前
    run2: [
        "....rrrrrr......",
        "...rrrrrrr....r.",
        "...rryyrrr...rr.",
        "..rryyyrrr..rr..",
        "..rrrrrrrrrrr...",
        "...rrrrrrrr.....",
        "...bbbbbbbb.....",
        "...bbbbbbbb.....",
        "....bbbbbb......",
        "....ww..ww......",
        "...wwww..ww.....",
        "....wwww..ww....",
        "....ww..wwrr....",
        "...rrrr....rrrr.",
        "................",
        "................",
    ],
    // 跑步帧3（中间帧，和 run1 略不同）
    run3: [
        "....rrrrrr......",
        "...rrrrrrr....r.",
        "...rryyrrr...rr.",
        "..rryyyrrr..rr..",
        "..rrrrrrrrrrr...",
        "...rrrrrrrr.....",
        "...bbbbbbbb.....",
        "...bbbbbbbb.....",
        "....bbbbbb......",
        "....ww..ww......",
        "...wwww..ww.....",
        "..wwww..ww......",
        "..rrww..ww......",
        ".rrrr....rrrr...",
        "................",
        "................",
    ],
    // 跳跃帧
    jump: [
        "....rrrrrr......",
        "...rrrrrrr....r.",
        "...rryyrrr...rr.",
        "..rryyyrrr..rr..",
        "..rrrrrrrrrrr...",
        "...rrrrrrrr.....",
        "...bbbbbbbb.....",
        "...bbbbbbbb.....",
        "....bbbbbb......",
        "....ww..ww......",
        "...ww..ww..ww...",
        "..ww..wwww..ww..",
        "..ww..wwww..ww..",
        ".rr..ww..ww..rr.",
        "................",
        "................",
    ],
    // 下滑帧（扁平，高度减半—用下半部分）
    slide: [
        "................",
        "................",
        "................",
        "................",
        "................",
        "................",
        "................",
        "................",
        "....rrrrrr......",
        "...rrrrrrr..r...",
        "...rryyrrr.rr...",
        "..rryyyrrrrr....",
        "..rrrrrrrrrr....",
        "...bbbbbbbb.....",
        "...wwwwwwww.....",
        "..wwwwwwwwww....",
    ],
    // 待机（开始画面）
    idle: [
        "....rrrrrr......",
        "...rrrrrrr....r.",
        "...rryyrrr...rr.",
        "..rryyyrrr..rr..",
        "..rrrrrrrrrrr...",
        "...rrrrrrrr.....",
        "...bbbbbbbb.....",
        "...bbbbbbbb.....",
        "....bbbbbb......",
        "....ww..ww......",
        "...wwww..ww.....",
        "...wwww..ww.....",
        "..wwww..wwww....",
        "..wwww..wwww....",
        "...ww..ww..ww...",
        "....rr..rr......",
    ],
};

// 像素字符到颜色的映射
const PIXEL_COLORS = {
    'r': '#e74c3c',  // 红色（帽子/上衣）
    'y': '#f1c40f',  // 黄色（皮肤/脸）
    'b': '#3498db',  // 蓝色（衣服主体）
    'w': '#ecf0f1',  // 白色（鞋子/手）
    '.': null,       // 透明
};
```

- [ ] **Step 2: 实现绘制角色像素函数**

```javascript
Renderer.drawPlayer = function() {
    const p = Player;
    // 选择当前动画帧
    let pixelData;
    if (p.state === 'IDLE') pixelData = PLAYER_PIXELS.idle;
    else if (p.state === 'JUMPING' || p.state === 'FALLING') pixelData = PLAYER_PIXELS.jump;
    else if (p.state === 'SLIDING') pixelData = PLAYER_PIXELS.slide;
    else if (p.state === 'DEAD') pixelData = PLAYER_PIXELS.jump; // 死亡用跳跃帧
    else { // RUNNING
        const frames = [PLAYER_PIXELS.run1, PLAYER_PIXELS.run2, PLAYER_PIXELS.run3];
        pixelData = frames[p.frame % 3];
    }

    const scale = CONFIG.PLAYER_SCALE;
    const size = 16;
    let drawY = p.y;

    // 下滑时高度减半，位置下移
    let effectiveHeight = size;
    if (p.state === 'SLIDING') {
        effectiveHeight = 8;
        drawY = p.y + size * scale / 2;
    }

    for (let row = 0; row < size; row++) {
        if (p.state === 'SLIDING' && row < 8) continue; // 下滑只画下半部分
        const pixelRow = pixelData[row];
        if (!pixelRow) continue;
        for (let col = 0; col < size; col++) {
            const ch = pixelRow[col];
            const color = PIXEL_COLORS[ch];
            if (!color) continue;
            ctx.fillStyle = color;
            ctx.fillRect(p.x + col * scale, drawY + (row - (p.state === 'SLIDING' ? 8 : 0)) * scale, scale, scale);
        }
    }
};
```

- [ ] **Step 3: 实现 PlayerUpdate 物理和动画更新**

```javascript
// ========== PlayerUpdate ==========
const PlayerUpdate = {
    update() {
        const p = Player;
        const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;

        if (p.state === 'DEAD') return;

        // 状态转换
        if (p.state === 'RUNNING') {
            if (Game.keys.space && p.vy === 0) {
                p.state = 'JUMPING';
                p.vy = CONFIG.JUMP_VELOCITY;
            }
            if (Game.keys.down) {
                p.state = 'SLIDING';
            }
        }

        if (p.state === 'SLIDING') {
            if (!Game.keys.down) {
                p.state = 'RUNNING';
            }
            // 下滑时碰撞盒变矮（在碰撞检测中处理）
        }

        // 跳跃物理
        if (p.state === 'JUMPING' || p.state === 'FALLING') {
            p.vy += CONFIG.GRAVITY;
            p.y += p.vy;
            if (p.vy < 0) p.state = 'JUMPING';
            else p.state = 'FALLING';
            // 落地
            if (p.y >= groundY - p.height) {
                p.y = groundY - p.height;
                p.vy = 0;
                p.state = 'RUNNING';
            }
        }

        // 动画帧更新
        p.frameTimer++;
        if (p.frameTimer >= 8) { // 每8帧换一次
            p.frameTimer = 0;
            p.frame++;
        }

        // 坑洞检测（掉坑）
        this.checkPit();
    },

    checkPit() {
        const p = Player;
        const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        // 如果没有在跳跃中，且脚在地面高度，检测脚下是否有坑
        if (p.state === 'RUNNING' || p.state === 'SLIDING') {
            for (const obs of ObstacleManager.obstacles) {
                if (obs.type === 'pit') {
                    const px = p.x + p.width / 2;
                    if (px >= obs.x && px <= obs.x + obs.width) {
                        // 掉坑
                        Game.die();
                    }
                }
            }
        }
    }
};
```

---

### Task 4: 障碍物 + 金币系统

**Files:**
- Modify: `e:/claude code learning/project_1/index.html`（填充 ObstacleManagerUpdate 和 CoinManagerUpdate）

- [ ] **Step 1: 实现 ObstacleManagerUpdate**

```javascript
// ========== ObstacleManagerUpdate ==========
const ObstacleManagerUpdate = {
    update(speed) {
        const mgr = ObstacleManager;
        // 更新计时器
        const minInterval = Math.max(0.8, 1.5 - Game.time * 0.01);
        mgr.timer += 1/60;
        if (mgr.timer >= minInterval) {
            mgr.timer = 0;
            this.spawnObstacle();
        }
        // 移动障碍物
        for (let i = mgr.obstacles.length - 1; i >= 0; i--) {
            const obs = mgr.obstacles[i];
            obs.x -= speed;
            if (obs.type === 'mover') {
                obs.moverTime += 1/60;
                obs.y = obs.baseY + Math.sin(obs.moverTime * 3) * 30;
            }
            if (obs.x + obs.width < 0) {
                mgr.obstacles.splice(i, 1);
            }
        }
    },

    spawnObstacle() {
        const types = ['stump', 'bird'];
        if (Game.time > 5) types.push('pit');
        if (Game.time > 20) types.push('mover');
        const type = types[Math.floor(Math.random() * types.length)];

        const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        let obs = { x: CONFIG.WIDTH, type: type };

        switch (type) {
            case 'stump':
                obs.width = 24 * CONFIG.PLAYER_SCALE;
                obs.height = 40 * CONFIG.PLAYER_SCALE;
                obs.y = groundY - obs.height;
                break;
            case 'bird':
                obs.width = 20 * CONFIG.PLAYER_SCALE;
                obs.height = 16 * CONFIG.PLAYER_SCALE;
                obs.y = groundY - 60 - 16 * CONFIG.PLAYER_SCALE;
                break;
            case 'pit':
                obs.width = 48 * CONFIG.PLAYER_SCALE;
                obs.y = groundY;
                obs.height = CONFIG.GROUND_HEIGHT;
                break;
            case 'mover':
                obs.width = 24 * CONFIG.PLAYER_SCALE;
                obs.height = 24 * CONFIG.PLAYER_SCALE;
                obs.baseY = groundY - 70;
                obs.y = obs.baseY;
                obs.moverTime = 0;
                break;
        }
        ObstacleManager.obstacles.push(obs);
    }
};
```

- [ ] **Step 2: 实现 Renderer.drawObstacles**

```javascript
Renderer.drawObstacles = function() {
    for (const obs of ObstacleManager.obstacles) {
        switch (obs.type) {
            case 'stump':
                // 棕色渐变柱子
                ctx.fillStyle = '#8B4513';
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
                ctx.fillStyle = '#A0522D';
                ctx.fillRect(obs.x + 4, obs.y, obs.width - 8, obs.height - 4);
                // 顶部高光
                ctx.fillStyle = '#CD853F';
                ctx.fillRect(obs.x + 2, obs.y + 2, obs.width - 4, 6);
                break;
            case 'bird':
                // 红色像素鸟
                ctx.fillStyle = '#e74c3c';
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
                ctx.fillStyle = '#c0392b';
                ctx.fillRect(obs.x + 4, obs.y + 4, obs.width - 8, obs.height - 4);
                // 眼睛
                ctx.fillStyle = '#fff';
                ctx.fillRect(obs.x + obs.width - 8, obs.y + 3, 4, 4);
                ctx.fillStyle = '#000';
                ctx.fillRect(obs.x + obs.width - 6, obs.y + 4, 2, 2);
                // 翅膀（上下扇动用正弦）
                const wingOffset = Math.sin(Game.time * 8) * 3;
                ctx.fillStyle = '#e74c3c';
                ctx.fillRect(obs.x - 4, obs.y + 6 + wingOffset, 6, 6);
                ctx.fillRect(obs.x + obs.width - 2, obs.y + 6 - wingOffset, 6, 6);
                break;
            case 'pit':
                // 坑洞（黑色深渊）
                ctx.fillStyle = '#1a1a2e';
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
                // 边缘警示
                ctx.fillStyle = '#8B4513';
                ctx.fillRect(obs.x - 2, obs.y, 4, 6);
                ctx.fillRect(obs.x + obs.width - 2, obs.y, 4, 6);
                break;
            case 'mover':
                // 紫色悬浮方块
                const glow = Math.sin(Game.time * 4) * 0.2 + 0.8;
                ctx.fillStyle = `rgba(138, 43, 226, ${glow})`;
                ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
                ctx.fillStyle = `rgba(186, 85, 211, ${glow})`;
                ctx.fillRect(obs.x + 3, obs.y + 3, obs.width - 6, obs.height - 6);
                break;
        }
    }
};
```

- [ ] **Step 3: 实现 CoinManagerUpdate**

```javascript
// ========== CoinManagerUpdate ==========
const CoinManagerUpdate = {
    update(speed) {
        const mgr = CoinManager;
        mgr.timer += 1/60;
        if (mgr.timer >= 1.5 + Math.random() * 1.5) {
            mgr.timer = 0;
            this.spawnCoins();
        }
        // 移动金币
        const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        for (let i = mgr.coins.length - 1; i >= 0; i--) {
            const c = mgr.coins[i];
            c.x -= speed;
            c.animTimer += 1/60;
            if (c.x + c.size < 0) {
                mgr.coins.splice(i, 1);
            }
        }
    },

    spawnCoins() {
        const groundY = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT;
        const count = 1 + Math.floor(Math.random() * 3);
        for (let i = 0; i < count; i++) {
            CoinManager.coins.push({
                x: CONFIG.WIDTH + i * 30,
                y: groundY - 30 - Math.random() * 80,
                size: 16,
                animTimer: Math.random() * Math.PI * 2,
                collected: false,
            });
        }
    }
};
```

- [ ] **Step 4: 实现 Renderer.drawCoins**

```javascript
Renderer.drawCoins = function() {
    for (const c of CoinManager.coins) {
        if (c.collected) continue;
        // 脉冲动画
        const pulse = 0.8 + Math.sin(c.animTimer * 5) * 0.2;
        const size = c.size * pulse;
        const cx = c.x + c.size/2;
        const cy = c.y + c.size/2;
        // 光晕
        ctx.fillStyle = `rgba(255, 215, 0, ${0.3 * pulse})`;
        ctx.fillRect(cx - size/2 - 4, cy - size/2 - 4, size + 8, size + 8);
        // 金币主体
        ctx.fillStyle = '#ffd700';
        ctx.fillRect(cx - size/2, cy - size/2, size, size);
        // 高光
        ctx.fillStyle = '#ffec8b';
        ctx.fillRect(cx - size/4, cy - size/4, size/2, size/2);
        // 闪光
        if (Math.sin(c.animTimer * 10) > 0.7) {
            ctx.fillStyle = 'rgba(255,255,255,0.8)';
            ctx.fillRect(cx - size/3, cy - size/3, size/4, size/4);
        }
    }
};
```

---

### Task 5: 碰撞检测 + 粒子系统 + 最终集成

**Files:**
- Modify: `e:/claude code learning/project_1/index.html`（填充 CollisionCheck、优化 ParticleSystem、修正像素风格、首次可玩）

- [ ] **Step 1: 实现 CollisionCheck**

```javascript
// ========== CollisionCheck ==========
const CollisionCheck = {
    check() {
        const p = Player;
        // 玩家碰撞盒（宽容判定，宽70% 高80%）
        const pw = p.width * 0.7;
        const ph = (p.state === 'SLIDING') ? p.height * 0.4 : p.height * 0.8;
        const px = p.x + (p.width - pw) / 2;
        const py = (p.state === 'SLIDING') ? p.y + p.height * 0.6 : p.y + (p.height - ph) / 2;

        for (const obs of ObstacleManager.obstacles) {
            if (obs.type === 'pit') continue; // 坑洞在 PlayerUpdate 中检测

            // 障碍碰撞盒
            const ow = obs.width * 0.8;
            const oh = obs.height * 0.8;
            const ox = obs.x + (obs.width - ow) / 2;
            const oy = obs.y + (obs.height - oh) / 2;

            // AABB 碰撞
            if (px < ox + ow && px + pw > ox &&
                py < oy + oh && py + ph > oy) {
                return true;
            }
        }
        return false;
    }
};
```

- [ ] **Step 2: 实现 ParticleSystem**

```javascript
// ========== ParticleSystemUpdate ==========
const ParticleSystemUpdate = {
    emit(x, y, color, count) {
        for (let i = 0; i < count; i++) {
            ParticleSystem.particles.push({
                x: x,
                y: y,
                vx: (Math.random() - 0.5) * 8,
                vy: (Math.random() - 0.5) * 8 - 3,
                size: 4 + Math.random() * 4,
                color: color,
                life: 1,
                maxLife: 30 + Math.random() * 20,
            });
        }
    },

    update() {
        for (let i = ParticleSystem.particles.length - 1; i >= 0; i--) {
            const p = ParticleSystem.particles[i];
            p.x += p.vx;
            p.y += p.vy;
            p.vy += 0.2; // 重力
            p.life--;
            if (p.life <= 0) {
                ParticleSystem.particles.splice(i, 1);
            }
        }
    }
};

// 收集金币时发射粒子——在 Game.update 中调用
function onCoinCollected(coin) {
    ParticleSystemUpdate.emit(
        coin.x + coin.size/2,
        coin.y + coin.size/2,
        '#ffd700', 8
    );
    Game.coins++;
}

// 死亡时发射粒子——在 Game.die 中调用
function onDeath() {
    ParticleSystemUpdate.emit(
        Player.x + Player.width/2,
        Player.y + Player.height/2,
        '#e74c3c', 20
    );
    ParticleSystemUpdate.emit(
        Player.x + Player.width/2,
        Player.y + Player.height/2,
        '#ffd700', 10
    );
}
```

- [ ] **Step 3: 集成金币收集到游戏循环**

修改 Game.updateGame 函数，在每帧检测金币收集：

```javascript
// 在 Game.updateGame 函数中添加金币收集检测
// 放在 "碰撞检测" 之前：
// 金币收集
for (let i = CoinManager.coins.length - 1; i >= 0; i--) {
    const c = CoinManager.coins[i];
    if (c.collected) continue;
    const pw = Player.width * 0.7;
    const ph = (Player.state === 'SLIDING') ? Player.height * 0.4 : Player.height * 0.8;
    const px = Player.x + (Player.width - pw) / 2;
    const py = (Player.state === 'SLIDING') ? Player.y + Player.height * 0.6 : Player.y + (Player.height - ph) / 2;
    if (px < c.x + c.size && px + pw > c.x &&
        py < c.y + c.size && py + ph > c.y) {
        c.collected = true;
        onCoinCollected(c);
    }
}
```

- [ ] **Step 4: 死亡效果 — 屏幕闪红**

在 Game.die 函数中添加：

```javascript
// 屏幕闪红效果
Renderer.flashTimer = 5;
```

然后在 Renderer.drawParticles 末尾添加：

```javascript
if (Renderer.flashTimer > 0) {
    Renderer.flashTimer--;
    ctx.fillStyle = `rgba(255, 0, 0, ${Renderer.flashTimer / 5 * 0.3})`;
    ctx.fillRect(0, 0, CONFIG.WIDTH, CONFIG.HEIGHT);
}
```

- [ ] **Step 5: 修复开始画面中的角色绘制**

在 Game.render 中，当处于 MENU 状态时也在 Canvas 上绘制角色待机动画：

```javascript
// 在 Game.render 中添加：
if (Game.state === 'MENU') {
    // 在背景上绘制待机角色（放大，居中偏左）
    Player.x = 350;
    Player.y = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT - Player.height - 20;
    Player.state = 'IDLE';
    Renderer.drawPlayer();
}
```

---

### Task 6: 最终润色 — 像素细节 + 手感调优 + 测试

**Files:**
- Modify: `e:/claude code learning/project_1/index.html`（整体打磨）

- [ ] **Step 1: 锁定 Canvas 像素渲染（防模糊）**

```css
canvas {
    display: block;
    width: 800px;
    height: 400px;
    image-rendering: crisp-edges; /* 更广泛兼容 */
    image-rendering: pixelated;
}
```

- [ ] **Step 2: 微调物理手感**

```javascript
// 在 CONFIG 中调整跳跃参数（如果感觉太重或太轻）
JUMP_VELOCITY: -10,  // 从 -9 微调到 -10，跳更高一点
GRAVITY: 0.45,       // 从 0.5 微调到 0.45，滞空感更好
```

- [ ] **Step 3: 确保开始时玩家处于正确地面位置**

```javascript
// 在 Game.startGame 中修正
Player.width = 16 * CONFIG.PLAYER_SCALE;  // 48
Player.height = 16 * CONFIG.PLAYER_SCALE; // 48
Player.x = 100;
Player.y = CONFIG.HEIGHT - CONFIG.GROUND_HEIGHT - Player.height;
```

- [ ] **Step 4: 打开浏览器测试完整流程**

在 Git Bash 中打开文件：

```bash
start e:/claude\ code\ learning/project_1/index.html
```

测试清单：
- [ ] 开始画面显示 → 按空格进入游戏
- [ ] 角色自动奔跑，动画切换
- [ ] 空格跳跃，下箭头下滑
- [ ] 地面柱出现 → 跳跃可躲避
- [ ] 飞鸟出现 → 下滑可躲避
- [ ] 坑洞出现（5秒后）→ 跳跃可躲避
- [ ] 移动障碍出现（20秒后）→ 根据位置操作
- [ ] 金币出现 → 靠近自动收集，有粒子效果
- [ ] HUD 显示距离和金币
- [ ] 速度逐渐加快
- [ ] 碰撞障碍后死亡，闪红，结算
- [ ] 结算画面显示正确数据
- [ ] 最高分持久化存储
- [ ] 按空格回到开始画面

- [ ] **Step 5: 提交完成（如使用 Git）**

```bash
git add index.html docs/superpowers/specs/2026-06-10-tian-tian-ku-pao-game-design.md docs/superpowers/plans/2026-06-10-tian-tian-ku-pao-implementation.md
git commit -m "feat: 天天酷跑像素风跑酷游戏完整实现"
```
