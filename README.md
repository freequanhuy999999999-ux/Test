<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Zombie Survival 2D - Single File Edition</title>
    <style>
        * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
        body { margin: 0; padding: 0; background: #111; color: #fff; font-family: monospace; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #game-container { position: relative; width: 100vw; height: 100vh; display: flex; flex-direction: column; justify-content: center; align-items: center; }
        canvas { background: #1a1a1a; border: 2px solid #00ff66; max-width: 100%; max-height: 70vh; box-shadow: 0 0 20px rgba(0,255,102,0.3); }
        #top-ui { width: 100%; max-width: 800px; padding: 10px; display: flex; justify-content: space-between; font-size: 14px; background: #222; border-bottom: 2px solid #333; }
        .hud-item { padding: 5px 10px; background: #000; border-radius: 4px; border: 1px solid #444; }
        .highlight { color: #00ff66; font-weight: bold; }
        .danger { color: #ff3333; font-weight: bold; }
        #controls-wrapper { width: 100%; max-width: 800px; height: 25vh; display: flex; justify-content: space-between; align-items: center; padding: 10px; background: #151515; }
        .dpad { display: grid; grid-template-columns: repeat(3, 1fr); gap: 5px; width: 130px; height: 130px; }
        .ctrl-btn { background: #333; border: 2px solid #555; color: white; border-radius: 10px; display: flex; justify-content: center; align-items: center; font-weight: bold; font-size: 18px; }
        .ctrl-btn:active { background: #00ff66; color: #000; }
        .action-group { display: flex; gap: 15px; }
        .action-btn { width: 70px; height: 70px; border-radius: 50%; border: 3px solid #ff3333; background: #222; color: white; font-weight: bold; font-size: 14px; display: flex; flex-direction: column; justify-content: center; align-items: center; box-shadow: 0 4px 6px rgba(0,0,0,0.5); }
        .action-btn:active { transform: scale(0.9); }
        .action-btn.shoot { border-color: #00ff66; background: #004411; }
        .action-btn.skill { border-color: #00ffff; background: #004455; }
        .cooldown { font-size: 10px; color: #00ffff; }
        #shop-modal { display: none; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 90%; max-width: 450px; background: rgba(20,20,20,0.95); border: 3px solid #00ff66; border-radius: 10px; padding: 20px; z-index: 100; box-shadow: 0 0 30px #000; }
        #shop-modal h2 { text-align: center; color: #00ff66; margin-top: 0; text-transform: uppercase; }
        .shop-item { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; background: #222; padding: 10px; border-radius: 5px; border: 1px solid #444; }
        .shop-btn { background: #00ff66; color: black; border: none; padding: 8px 12px; font-weight: bold; border-radius: 4px; cursor: pointer; }
        .shop-btn:disabled { background: #555; color: #aaa; }
        .close-shop { width: 100%; background: #ff3333; color: white; border: none; padding: 10px; font-weight: bold; margin-top: 10px; border-radius: 4px; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="top-ui">
        <div class="hud-item">HP: <span id="ui-hp" class="danger">100</span></div>
        <div class="hud-item">LVL: <span id="ui-level" class="highlight">1</span> (<span id="ui-exp">0</span>/100 XP)</div>
        <div class="hud-item">VÀNG: <span id="ui-gold" style="color: #ffff00;">0</span></div>
        <button onclick="toggleShop(true)" style="background: #ffff00; color: #000; border: none; font-weight: bold; border-radius: 4px; padding: 5px 10px; cursor: pointer;">SHOP</button>
    </div>

    <canvas id="gameCanvas" width="800" height="500"></canvas>

    <div id="controls-wrapper">
        <div class="dpad">
            <div></div><div class="ctrl-btn" id="btn-up">▲</div><div></div>
            <div class="ctrl-btn" id="btn-left">◀</div><div style="background: #222; border-radius: 5px;"></div><div class="ctrl-btn" id="btn-right">▶</div>
            <div></div><div class="ctrl-btn" id="btn-down">▼</div><div></div>
        </div>
        <div class="action-group">
            <div class="action-btn skill" id="btn-skill"><span>SKILL</span><span class="cooldown" id="skill-cd">SẴN SÀNG</span></div>
            <div class="action-btn shoot" id="btn-shoot">BẮN</div>
        </div>
    </div>

    <div id="shop-modal">
        <h2>Cửa hàng nâng cấp VIP</h2>
        <p style="text-align: center;">Số vàng hiện có: <span id="shop-gold" style="color: #ffff00; font-weight: bold;">0</span></p>
        <div class="shop-item">
            <div><strong>Tăng Sát Thương (+5 ATK)</strong><br><small>Hiện tại: <span id="stat-atk">15</span></small></div>
            <button class="shop-btn" onclick="buyUpgrade('atk')"><span id="cost-atk">30</span> Vàng</button>
        </div>
        <div class="shop-item">
            <div><strong>Tăng Máu Tối Đa (+20 HP)</strong><br><small>Hiện tại: <span id="stat-maxhp">100</span></small></div>
            <button class="shop-btn" onclick="buyUpgrade('maxhp')"><span id="cost-maxhp">40</span> Vàng</button>
        </div>
        <div class="shop-item">
            <div><strong>Tăng Tốc Độ Chạy (+0.5)</strong><br><small>Hiện tại: <span id="stat-spd">4</span></small></div>
            <button class="shop-btn" onclick="buyUpgrade('speed')"><span id="cost-speed">50</span> Vàng</button>
        </div>
        <div class="shop-item">
            <div><strong>Mua Bình Máu (Hồi đầy HP)</strong></div>
            <button class="shop-btn" onclick="buyUpgrade('heal')">20 Vàng</button>
        </div>
        <button class="close-shop" onclick="toggleShop(false)">ĐÓNG CỬA HÀNG</button>
    </div>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const player = {
    x: canvas.width / 2,
    y: canvas.height / 2,
    radius: 15,
    speed: 4,
    hp: 100,
    maxHp: 100,
    level: 1,
    exp: 0,
    nextExp: 100,
    gold: 0,
    damage: 15,
    direction: "right",
    skillCooldown: 0,
    isDead: false
};

const shopCosts = { atk: 30, maxhp: 40, speed: 50 };
let bullets = [];
let enemies = [];
let particles = [];
let enemySpawnTimer = 0;
let enemySpawnInterval = 120;
let isShopOpen = false;

const inputState = { up: false, down: false, left: false, right: false };

function setupMobileControls() {
    const bindLink = (id, key) => {
        const el = document.getElementById(id);
        if(!el) return;
        el.addEventListener("touchstart", (e) => { e.preventDefault(); inputState[key] = true; });
        el.addEventListener("touchend", (e) => { e.preventDefault(); inputState[key] = false; });
    };
    bindLink("btn-up", "up");
    bindLink("btn-down", "down");
    bindLink("btn-left", "left");
    bindLink("btn-right", "right");

    document.getElementById("btn-shoot").addEventListener("touchstart", (e) => { e.preventDefault(); shootBullet(); });
    document.getElementById("btn-skill").addEventListener("touchstart", (e) => { e.preventDefault(); triggerSkill(); });
}

window.addEventListener("keydown", (e) => {
    if(isShopOpen) return;
    if (e.key === "w" || e.key === "ArrowUp") inputState.up = true;
    if (e.key === "s" || e.key === "ArrowDown") inputState.down = true;
    if (e.key === "a" || e.key === "ArrowLeft") inputState.left = true;
    if (e.key === "d" || e.key === "ArrowRight") inputState.right = true;
    if (e.key === " ") shootBullet();
    if (e.key === "e" || e.key === "E") triggerSkill();
});

window.addEventListener("keyup", (e) => {
    if (e.key === "w" || e.key === "ArrowUp") inputState.up = false;
    if (e.key === "s" || e.key === "ArrowDown") inputState.down = false;
    if (e.key === "a" || e.key === "ArrowLeft") inputState.left = false;
    if (e.key === "d" || e.key === "ArrowRight") inputState.right = false;
});

function shootBullet() {
    if (player.isDead || isShopOpen) return;
    let angle = 0;
    if (player.direction === "up") angle = -Math.PI / 2;
    if (player.direction === "down") angle = Math.PI / 2;
    if (player.direction === "left") angle = Math.PI;
    if (player.direction === "right") angle = 0;

    bullets.push({
        x: player.x,
        y: player.y,
        vx: Math.cos(angle) * 8,
        vy: Math.sin(angle) * 8,
        radius: 4,
        damage: player.damage,
        color: "#ffff00"
    });
}

function triggerSkill() {
    if (player.isDead || isShopOpen || player.skillCooldown > 0) return;
    const totalBullets = 16;
    for (let i = 0; i < totalBullets; i++) {
        let angle = (i / totalBullets) * Math.PI * 2;
        bullets.push({
            x: player.x,
            y: player.y,
            vx: Math.cos(angle) * 6,
            vy: Math.sin(angle) * 6,
            radius: 6,
            damage: player.damage * 2,
            color: "#00ffff"
        });
    }
    player.skillCooldown = 300;
}

function spawnEnemy() {
    if (isShopOpen || player.isDead) return;
    let x, y;
    if (Math.random() < 0.5) {
        x = Math.random() < 0.5 ? -20 : canvas.width + 20;
        y = Math.random() * canvas.height;
    } else {
        x = Math.random() * canvas.width;
        y = Math.random() < 0.5 ? -20 : canvas.height + 20;
    }

    const difficultyMultiplier = 1 + (player.level * 0.15);
    let typeRand = Math.random();
    let hp = 30 * difficultyMultiplier;
    let speed = (1 + Math.random() * 1.2);
    let color = "#88cc00";
    let rewardGold = 5 + player.level;

    if (typeRand > 0.85) {
        hp = 100 * difficultyMultiplier;
        speed = 0.6;
        color = "#ff3300";
        rewardGold = 15 + player.level * 2;
    } else if (typeRand > 0.6) {
        hp = 20 * difficultyMultiplier;
        speed = 2.2;
        color = "#ffcc00";
    }

    enemies.push({ x, y, hp, speed, color, radius: 12, gold: Math.floor(rewardGold) });
}

function createExplosion(x, y, color) {
    for (let i = 0; i < 8; i++) {
        particles.push({
            x: x, y: y,
            vx: (Math.random() - 0.5) * 4,
            vy: (Math.random() - 0.5) * 4,
            radius: Math.random() * 3 + 1,
            life: 30,
            color: color
        });
    }
}

function toggleShop(open) {
    isShopOpen = open;
    document.getElementById("shop-modal").style.display = open ? "block" : "none";
    if(open) {
        document.getElementById("shop-gold").innerText = player.gold;
        document.getElementById("stat-atk").innerText = player.damage;
        document.getElementById("stat-maxhp").innerText = player.maxHp;
        document.getElementById("stat-spd").innerText = player.speed.toFixed(1);
        document.getElementById("cost-atk").innerText = shopCosts.atk;
        document.getElementById("cost-maxhp").innerText = shopCosts.maxhp;
        document.getElementById("cost-speed").innerText = shopCosts.speed;
    }
}

function buyUpgrade(type) {
    if (type === 'heal' && player.gold >= 20) {
        player.gold -= 20;
        player.hp = player.maxHp;
    } else if (type === 'atk' && player.gold >= shopCosts.atk) {
        player.gold -= shopCosts.atk;
        player.damage += 5;
        shopCosts.atk = Math.floor(shopCosts.atk * 1.4);
    } else if (type === 'maxhp' && player.gold >= shopCosts.maxhp) {
        player.gold -= shopCosts.maxhp;
        player.maxHp += 20;
        player.hp += 20;
        shopCosts.maxhp = Math.floor(shopCosts.maxhp * 1.4);
    } else if (type === 'speed' && player.gold >= shopCosts.speed) {
        player.gold -= shopCosts.speed;
        player.speed += 0.5;
        shopCosts.speed = Math.floor(shopCosts.speed * 1.4);
    } else {
        return;
    }
    toggleShop(true);
    updateHUD();
}

function updateHUD() {
    document.getElementById("ui-hp").innerText = Math.max(0, Math.floor(player.hp));
    document.getElementById("ui-level").innerText = player.level;
    document.getElementById("ui-exp").innerText = `${player.exp}/${player.nextExp}`;
    document.getElementById("ui-gold").innerText = player.gold;
    
    const cdLabel = document.getElementById("skill-cd");
    if (player.skillCooldown > 0) {
        cdLabel.innerText = `${Math.ceil(player.skillCooldown / 60)}s`;
    } else {
        cdLabel.innerText = "SẴN SÀNG";
    }
}

function updateGame() {
    if (player.isDead) {
        ctx.fillStyle = "rgba(0,0,0,0.8)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = "#ff3333";
        ctx.font = "bold 40px monospace";
        ctx.textAlign = "center";
        ctx.fillText("GAME OVER!", canvas.width / 2, canvas.height / 2);
        return;
    }

    if (!isShopOpen) {
        if (inputState.up) { player.y -= player.speed; player.direction = "up"; }
        if (inputState.down) { player.y += player.speed; player.direction = "down"; }
        if (inputState.left) { player.x -= player.speed; player.direction = "left"; }
        if (inputState.right) { player.x += player.speed; player.direction = "right"; }

        player.x = Math.max(player.radius, Math.min(canvas.width - player.radius, player.x));
        player.y = Math.max(player.radius, Math.min(canvas.height - player.radius, player.y));

        if (player.skillCooldown > 0) player.skillCooldown--;

        enemySpawnTimer++;
        let dynamicInterval = Math.max(25, enemySpawnInterval - (player.level * 8));
        if (enemySpawnTimer >= dynamicInterval) {
            spawnEnemy();
            enemySpawnTimer = 0;
        }

        for (let i = bullets.length - 1; i >= 0; i--) {
            let b = bullets[i];
            b.x += b.vx;
            b.y += b.vy;
            if (b.x < 0 || b.x > canvas.width || b.y < 0 || b.y > canvas.height) {
                bullets.splice(i, 1);
            }
        }

        for (let i = enemies.length - 1; i >= 0; i--) {
            let e = enemies[i];
            let dx = player.x - e.x;
            let dy = player.y - e.y;
            let dist = Math.hypot(dx, dy);

            if (dist > 1) {
                e.x += (dx / dist) * e.speed;
                e.y += (dy / dist) * e.speed;
            }

            if (dist < player.radius + e.radius) {
                player.hp -= 0.4;
                if (player.hp <= 0) player.isDead = true;
            }

            for (let j = bullets.length - 1; j >= 0; j--) {
                let b = bullets[j];
                let bDist = Math.hypot(b.x - e.x, b.y - e.y);
                if (bDist < b.radius + e.radius) {
                    e.hp -= b.damage;
                    bullets.splice(j, 1);

                    if (e.hp <= 0) {
                        createExplosion(e.x, e.y, e.color);
                        player.gold += e.gold;
                        player.exp += 15;
                        
                        if (player.exp >= player.nextExp) {
                            player.level++;
                            player.exp -= player.nextExp;
                            player.nextExp = Math.floor(player.nextExp * 1.5);
                            player.maxHp += 10;
                            player.hp = player.maxHp;
                            player.damage += 3;
                        }
                        enemies.splice(i, 1);
                        break;
                    }
                }
            }
        }

        for (let i = particles.length - 1; i >= 0; i--) {
            let p = particles[i];
            p.x += p.vx;
            p.y += p.vy;
            p.life--;
            if (p.life <= 0) particles.splice(i, 1);
        }
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    particles.forEach(p => {
        ctx.fillStyle = p.color;
        ctx.globalAlpha = p.life / 30;
        ctx.beginPath();
        ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.globalAlpha = 1.0;
    });

    bullets.forEach(b => {
        ctx.fillStyle = b.color;
        ctx.beginPath();
        ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
        ctx.fill();
    });

    enemies.forEach(e => {
        ctx.fillStyle = e.color;
        ctx.beginPath();
        ctx.arc(e.x, e.y, e.radius, 0, Math.PI * 2);
        ctx.fill();
        ctx.fillStyle = "#000";
        ctx.beginPath();
        ctx.arc(e.x - 3, e.y - 3, 2, 0, Math.PI * 2);
        ctx.arc(e.x + 3, e.y - 3, 2, 0, Math.PI * 2);
        ctx.fill();
    });

    if (!player.isDead) {
        ctx.fillStyle = "#00ff66";
        ctx.beginPath();
        ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2);
        ctx.fill();

        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 4;
        ctx.beginPath();
        ctx.moveTo(player.x, player.y);
        if (player.direction === "up") ctx.lineTo(player.x, player.y - 25);
        if (player.direction === "down") ctx.lineTo(player.x, player.y + 25);
        if (player.direction === "left") ctx.lineTo(player.x - 25, player.y);
        if (player.direction === "right") ctx.lineTo(player.x + 25, player.y);
        ctx.stroke();
    }

    updateHUD();
    requestAnimationFrame(updateGame);
}

setupMobileControls();
requestAnimationFrame(updateGame);
</script>
</body>
</html>
