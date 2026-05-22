<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Zombie Survival 2D - Ultimate Balance Edition</title>
    <style>
        * { box-sizing: border-box; user-select: none; -webkit-user-select: none; }
        body { margin: 0; padding: 0; background: #111; color: #fff; font-family: monospace; overflow: hidden; display: flex; justify-content: center; align-items: center; height: 100vh; }
        #game-container { position: relative; width: 100vw; height: 100vh; display: flex; flex-direction: column; justify-content: center; align-items: center; }
        canvas { background: #1a1a1a; border: 2px solid #00ff66; max-width: 100%; max-height: 65vh; box-shadow: 0 0 20px rgba(0,255,102,0.3); }
        
        #top-ui { width: 100%; max-width: 800px; padding: 8px 12px; display: flex; justify-content: space-between; font-size: 13px; background: #222; border-bottom: 2px solid #333; align-items: center; gap: 5px; flex-wrap: wrap; }
        .hud-item { padding: 4px 6px; background: #000; border-radius: 4px; border: 1px solid #444; font-size: 11px; }
        .highlight { color: #00ff66; font-weight: bold; }
        .danger { color: #ff3333; font-weight: bold; }
        
        #controls-wrapper { width: 100%; max-width: 800px; height: 28vh; display: flex; justify-content: space-between; align-items: center; padding: 10px; background: #151515; touch-action: none; }
        #drag-zone { width: 140px; height: 140px; background: #222; border: 3px solid #555; border-radius: 8px; display: flex; flex-direction: column; justify-content: center; align-items: center; text-align: center; font-size: 10px; color: #aaa; position: relative; touch-action: none; }
        #drag-joystick-visual { width: 40px; height: 40px; background: #00ff66; border-radius: 6px; position: absolute; display: none; pointer-events: none; opacity: 0.8; box-shadow: 0 0 10px #00ff66; }
        
        .action-group { display: flex; gap: 12px; align-items: center; touch-action: none; }
        .action-btn { width: 70px; height: 70px; border-radius: 50%; border: 2px solid #00ffff; background: #003344; color: white; font-weight: bold; font-size: 10px; display: flex; flex-direction: column; justify-content: center; align-items: center; box-shadow: 0 4px 6px rgba(0,0,0,0.5); cursor: pointer; text-align: center; }
        .action-btn:active { transform: scale(0.9); }
        .action-btn.shoot { border-color: #00ff66; background: #004411; width: 80px; height: 80px; font-size: 15px; }
        .cooldown { font-size: 9px; color: #ffcc00; margin-top: 2px; }

        #boss-announcement { display: none; position: absolute; top: 12%; left: 50%; transform: translateX(-50%); background: rgba(150, 0, 0, 0.9); color: white; padding: 8px 20px; font-size: 15px; font-weight: bold; border: 2px dashed gold; border-radius: 5px; text-align: center; z-index: 99; animation: blink 0.8s infinite; }
        #countdown-overlay { display: none; position: absolute; top: 12%; left: 50%; transform: translateX(-50%); background: rgba(0, 0, 0, 0.85); color: #ffff00; padding: 6px 15px; border-radius: 5px; border: 1px solid #ffff00; z-index: 98; text-align: center; font-weight: bold; }
        .small-title { font-size: 11px; color: #bbb; display: block; margin-bottom: 2px; font-weight: normal; }
        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }

        #shop-modal { display: none; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); width: 92%; max-width: 480px; max-height: 80vh; overflow-y: auto; background: rgba(15,15,15,0.98); border: 3px solid #00ff66; border-radius: 10px; padding: 15px; z-index: 100; box-shadow: 0 0 35px #000; }
        .shop-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #00ff66; padding-bottom: 8px; margin-bottom: 12px; }
        .shop-header h2 { margin: 0; color: #00ff66; font-size: 18px; text-transform: uppercase; }
        .close-x-btn { background: #ff3333; color: white; border: none; font-size: 14px; font-weight: bold; width: 26px; height: 26px; border-radius: 5px; cursor: pointer; display: flex; justify-content: center; align-items: center; }
        .shop-tabs { display: flex; gap: 6px; margin-bottom: 12px; }
        .tab-btn { flex: 1; background: #333; color: white; border: 1px solid #555; padding: 6px 2px; font-weight: bold; cursor: pointer; border-radius: 4px; font-size: 10px; }
        .tab-btn.active { background: #00ff66; color: black; border-color: #00ff66; }
        .shop-item { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; background: #222; padding: 8px; border-radius: 5px; border: 1px solid #444; }
        .shop-btn { background: #00ff66; color: black; border: none; padding: 6px 10px; font-weight: bold; border-radius: 4px; cursor: pointer; min-width: 95px; font-size: 11px; }
        .shop-btn:disabled { background: #555; color: #aaa; cursor: not-allowed; }

        #game-over-screen { display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.9); flex-direction: column; justify-content: center; align-items: center; z-index: 101; }
        #game-over-screen h1 { color: #ff3333; font-size: 32px; margin-bottom: 10px; }
        #game-over-screen p { font-size: 16px; margin-bottom: 20px; color: #ccc; text-align: center; line-height: 1.6; }
        .restart-btn { background: #00ff66; color: #000; border: none; font-size: 16px; font-weight: bold; padding: 10px 30px; border-radius: 25px; cursor: pointer; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="boss-announcement">⚠️ BOSS XUẤT HIỆN! ⚠️</div>
    <div id="countdown-overlay"><span class="small-title">Boss tiếp theo xuất hiện sau:</span><span id="countdown-time">30</span> giây</div>

    <div id="game-over-screen">
        <h1>BẠN ĐÃ TỬ TRẬN!</h1>
        <p>
            Thời gian sống sót: <span id="final-survive-time" class="highlight">0s</span><br>
            Cấp độ người chơi: <span id="final-level" class="highlight">1</span>
        </p>
        <button class="restart-btn" id="btn-restart-click">BẮT ĐẦU LẠI</button>
    </div>

    <div id="top-ui">
        <div class="hud-item" style="min-width: 75px;">HP: <span id="ui-hp" class="danger">100</span><span id="ui-revive-indicator" style="color:#ff33aa; display:none;"> 🛡️</span></div>
        <div class="hud-item">LV: <span id="ui-level" class="highlight">1</span></div>
        <div class="hud-item">VÀNG: <span id="ui-gold" style="color: #ffff00;">0</span></div>
        <div class="hud-item" style="color: #00ffff;">TIME: <span id="ui-survive">0s</span></div>
        <div class="hud-item" style="color: #ff00ff; font-weight: bold;">BOSS: <span id="ui-boss-state">LV.1 (Vòng 1)</span></div>
        <button onclick="toggleShop(true)" style="background: #ffff00; color: #000; border: none; font-weight: bold; border-radius: 4px; padding: 4px 8px; cursor: pointer; font-size: 11px;">SHOP</button>
    </div>

    <canvas id="gameCanvas" width="800" height="480"></canvas>

    <div id="controls-wrapper">
        <div id="drag-zone">
            <span>DI CHUYỂN</span>
            <div id="drag-joystick-visual"></div>
        </div>
        <div class="action-group">
            <div class="action-btn" id="btn-skill">
                <span id="txt-skill-name">SKILL LV.1</span>
                <span class="cooldown" id="sk-cd">SẴN SÀNG</span>
            </div>
            <div class="action-btn shoot" id="btn-shoot">BẮN</div>
        </div>
    </div>

    <div id="shop-modal">
        <div class="shop-header">
            <h2>Cửa Hàng Chiến Binh</h2>
            <button class="close-x-btn" onclick="toggleShop(false)">X</button>
        </div>
        <p style="text-align: center; margin-top: 0; font-size: 12px;">Vàng: <span id="shop-gold" style="color: #ffff00; font-weight: bold;">0</span> | Buff hiện tại: <span id="shop-current-mult" style="color: #ffcc00; font-weight: bold;">x1 Vàng</span></p>
        
        <div class="shop-tabs">
            <button class="tab-btn active" id="tab-stat" onclick="switchTab('stat')">Chỉ Số</button>
            <button class="tab-btn" id="tab-skill" onclick="switchTab('skill')">Tuyệt Kỹ & Hồi Sinh</button>
            <button class="tab-btn" id="tab-money" onclick="switchTab('money')">💰 Thần Tài Vàng</button>
        </div>

        <div id="panel-stat">
            <div class="shop-item">
                <div><strong>Sát Thương (+5 ATK)</strong><br><small>Hiện tại: <span id="stat-atk">15</span></small></div>
                <button class="shop-btn" onclick="buyUpgrade('atk')"><span id="cost-atk">30</span> Vàng</button>
            </div>
            <div class="shop-item">
                <div><strong>Máu Tối Đa (+20 HP)</strong><br><small>Hiện tại: <span id="stat-maxhp">100</span></small></div>
                <button class="shop-btn" onclick="buyUpgrade('maxhp')"><span id="cost-maxhp">40</span> Vàng</button>
            </div>
            <div class="shop-item">
                <div><strong>Tốc Độ Chạy (+0.4)</strong><br><small>Hiện tại: <span id="stat-spd">4</span></small></div>
                <button class="shop-btn" onclick="buyUpgrade('speed')"><span id="cost-speed">50</span> Vàng</button>
            </div>
            <div class="shop-item">
                <div><strong>Bình Máu (Hồi đầy HP)</strong></div>
                <button class="shop-btn" onclick="buyUpgrade('heal')">20 Vàng</button>
            </div>
        </div>

        <div id="panel-skill" style="display: none;">
            <div class="shop-item" style="border: 2px solid #ff33aa;">
                <div><strong style="color: #ff33aa;">🛡️ Bị Động: HỒI SINH (1 lần)</strong><br><small>Tự động hồi sinh khi chết. Dùng xong sẽ mất.</small></div>
                <button class="shop-btn" id="shop-btn-revive" style="background: #ff33aa; color: white;" onclick="buyUpgrade('revive_passive')">150.000 Vàng</button>
            </div>
            <div class="shop-item" style="border: 1px solid #00ffff;">
                <div style="max-width: 65%;">
                    <strong style="color: #00ffff;">Đột Phá Kỹ Năng (Max Lv.10)</strong><br>
                    <small id="skill-desc" style="font-size: 10px; color:#ddd;"></small>
                </div>
                <button class="shop-btn" id="shop-btn-upgrade-skill" onclick="buyUpgrade('upgrade_skill')">50 Vàng</button>
            </div>
            <div class="shop-item">
                <div><strong style="color: #ffff00;">[BỊ ĐỘNG] Tự Động Ngắm</strong><br><small>Tự xoay nòng tìm mục tiêu gần nhất</small></div>
                <button class="shop-btn" id="shop-btn-aim" style="background: #ffff00;" onclick="buyUpgrade('aim_passive')">100 Vàng</button>
            </div>
        </div>

        <div id="panel-money" style="display: none;">
            <div class="shop-item">
                <div><strong style="color: #ffaa00;">Gói Tài Lộc X2 Vàng</strong></div>
                <button class="shop-btn" id="btn-gold-x2" onclick="buyUpgrade('gold_x2')">100 Vàng</button>
            </div>
            <div class="shop-item">
                <div><strong style="color: #ffaa00;">Gói Phú Quý X4 Vàng</strong></div>
                <button class="shop-btn" id="btn-gold-x4" onclick="buyUpgrade('gold_x4')">400 Vàng</button>
            </div>
            <div class="shop-item" style="border: 1px solid #ffaa00;">
                <div><strong style="color: #ffaa00;">🔥 Gói Vạn Lộc X100 Vàng</strong><br><small>Quái x100 vàng rơi ra</small></div>
                <button class="shop-btn" id="btn-gold-x100" onclick="buyUpgrade('gold_x100')">2.000 Vàng</button>
            </div>
            <div class="shop-item" style="border: 2px solid #ffff00;">
                <div><strong style="color: #ffff00;">👑 Gói Hoàng Kim X1000 Vàng</strong><br><small>Quái x1000 vàng rơi ra</small></div>
                <button class="shop-btn" id="btn-gold-x1000" style="background:#ffff00;" onclick="buyUpgrade('gold_x1000')">20.000 Vàng</button>
            </div>
        </div>
    </div>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const SKILL_LEVELS = {
    1: { name: "Vòng Tia Sơ Cấp", count: 8, size: 5, color: "#00ffff", dmgMult: 1.2, speed: 6, desc: "Bắn vòng tròn 8 tia đạn cơ bản." },
    2: { name: "Song Trận Thủy Triều", count: 12, size: 6, color: "#00ffcc", dmgMult: 1.4, speed: 6.5, desc: "Tăng lên 12 tia đạn góc rộng." },
    3: { name: "Lục Địa Chấn Động", count: 16, size: 6.5, color: "#00ff88", dmgMult: 1.6, speed: 7, desc: "Xả liên thanh 16 tia năng lượng xanh." },
    4: { name: "Hỏa Liên Vũ Trụ", count: 20, size: 7, color: "#ffff00", dmgMult: 1.8, speed: 7.5, desc: "Bùng nổ trận pháp bọc lửa 20 tia vàng." },
    5: { name: "Huyết Nguyệt Bạo Kích", desc: "Sóng kích động đẩy lùi quái và gây lượng sát thương lớn xung quanh." },
    6: { name: "Tử Thần Tinh Nebula", desc: "Để lại 1 phân thân bóng tối bắn tỏa ra 20 viên đạn ma thuật tím." },
    7: { name: "Thiên Tai Cuồng Phong", desc: "Tạo Aura làm chậm và giật điện liên tục (Sát thương tăng gấp 3)." },
    8: { name: "Băng Hà Ma Vương", desc: "Hào quang đóng băng phạm vi cực đại, rút máu quái siêu nhanh." },
    9: { name: "Hỗn Loạn Thần Điện", desc: "Triệu hồi Hố Đen hút mọi kẻ địch vào trung tâm bản đồ." },
    10: { name: "👑VÔ ĐỊCH THẦN MA👑", desc: "Hố đen x2 size duy trì 5 giây! 10 phân thân vây quanh quét Lazer Tím cực khủng!" }
};

let player = {
    x: canvas.width / 2, y: canvas.height / 2, radius: 14, speed: 4, hp: 100, maxHp: 100,
    level: 1, exp: 0, nextExp: 100, gold: 0, damage: 15, direction: "right", isDead: false,
    skill_lv: 1, skill_cd: 0, aimPassive: false, goldMultiplier: 1,
    hasRevive: false 
};

let bossSystem = {
    level: 1,
    loop: 1,
    hpBase: 400,          
    alive: false,
    currentBoss: null,
    countdownActive: false,
    countdownFrames: 1800, 
    survivalFrames: 0      
};

const GOLD_REWARD_NORMAL = 10; 
const GOLD_REWARD_BOSS = 200;  

let shopCosts = { atk: 30, maxhp: 40, speed: 50, skill_up: 60 };
let bullets = []; let enemies = []; let particles = [];
let normalEnemyTimer = 0;
let isShopOpen = false; let currentTab = 'stat'; let animationFrameId = null;

let activeAura = null; let activeBlackHole = null; let activeClones = [];
let joystickTouchId = null;
const dragInput = { active: false, startX: 0, startY: 0, moveX: 0, moveY: 0 };
const inputState = { up: false, down: false, left: false, right: false };

function setupInputControls() {
    const dragZone = document.getElementById("drag-zone");
    const joystickVisual = document.getElementById("drag-joystick-visual");

    dragZone.addEventListener("touchstart", (e) => {
        if (player.isDead || isShopOpen) return;
        const touch = e.changedTouches[0]; joystickTouchId = touch.identifier; dragInput.active = true;
        const rect = dragZone.getBoundingClientRect();
        dragInput.startX = touch.clientX - rect.left; dragInput.startY = touch.clientY - rect.top;
        joystickVisual.style.display = "block";
        joystickVisual.style.left = `${dragInput.startX - 20}px`; joystickVisual.style.top = `${dragInput.startY - 20}px`;
    }, { passive: true });

    window.addEventListener("touchmove", (e) => {
        if (!dragInput.active || player.isDead) return;
        for (let i = 0; i < e.touches.length; i++) {
            if (e.touches[i].identifier === joystickTouchId) {
                const touch = e.touches[i]; const rect = dragZone.getBoundingClientRect();
                let currentX = touch.clientX - rect.left; let currentY = touch.clientY - rect.top;
                let dx = currentX - dragInput.startX; let dy = currentY - dragInput.startY;
                let maxRadius = 55; dx = Math.max(-maxRadius, Math.min(maxRadius, dx)); dy = Math.max(-maxRadius, Math.min(maxRadius, dy));
                joystickVisual.style.left = `${dragInput.startX + dx - 20}px`; joystickVisual.style.top = `${dragInput.startY + dy - 20}px`;
                dragInput.moveX = dx / maxRadius; dragInput.moveY = dy / maxRadius;
                if (Math.abs(dx) > Math.abs(dy)) { player.direction = dx > 0 ? "right" : "left"; } else { player.direction = dy > 0 ? "down" : "up"; }
                break;
            }
        }
    }, { passive: true });

    const endJoystick = (e) => {
        if (!dragInput.active) return;
        for (let i = 0; i < e.changedTouches.length; i++) {
            if (e.changedTouches[i].identifier === joystickTouchId) {
                dragInput.active = false; dragInput.moveX = 0; dragInput.moveY = 0;
                joystickVisual.style.display = "none"; joystickTouchId = null; break;
            }
        }
    };
    window.addEventListener("touchend", endJoystick); window.addEventListener("touchcancel", endJoystick);
    document.getElementById("btn-shoot").addEventListener("pointerdown", () => { shootBullet(); });
    document.getElementById("btn-skill").addEventListener("pointerdown", () => { triggerUltimateSkill(); });
    document.getElementById("btn-restart-click").addEventListener("pointerdown", () => { restartGame(); });
}

window.addEventListener("keydown", (e) => {
    if(isShopOpen) return;
    if (e.key === "w" || e.key === "ArrowUp") inputState.up = true;
    if (e.key === "s" || e.key === "ArrowDown") inputState.down = true;
    if (e.key === "a" || e.key === "ArrowLeft") inputState.left = true;
    if (e.key === "d" || e.key === "ArrowRight") inputState.right = true;
    if (e.key === " ") shootBullet();
    if (e.key === "1" || e.key === "f") triggerUltimateSkill();
});
window.addEventListener("keyup", (e) => {
    if (e.key === "w" || e.key === "ArrowUp") inputState.up = false;
    if (e.key === "s" || e.key === "ArrowDown") inputState.down = false;
    if (e.key === "a" || e.key === "ArrowLeft") inputState.left = false;
    if (e.key === "d" || e.key === "ArrowRight") inputState.right = false;
});

function shootBullet() {
    if (player.isDead || isShopOpen) return;
    let angle = 0; let aimed = false;
    if (player.aimPassive && enemies.length > 0) {
        let nearestEnemy = null; let minDist = Infinity;
        enemies.forEach(e => { let d = Math.hypot(e.x - player.x, e.y - player.y); if (d < minDist) { minDist = d; nearestEnemy = e; } });
        if (nearestEnemy) { angle = Math.atan2(nearestEnemy.y - player.y, nearestEnemy.x - player.x); aimed = true; }
    }
    if (!aimed) {
        if (player.direction === "up") angle = -Math.PI / 2; if (player.direction === "down") angle = Math.PI / 2;
        if (player.direction === "left") angle = Math.PI; if (player.direction === "right") angle = 0;
    }
    bullets.push({ x: player.x, y: player.y, vx: Math.cos(angle) * 8, vy: Math.sin(angle) * 8, radius: 4, damage: player.damage, color: player.aimPassive ? "#ffff00" : "#ffffff", isEnemy: false });
}

function triggerUltimateSkill() {
    if (player.isDead || isShopOpen || player.skill_cd > 0) return;
    const lv = player.skill_lv;
    
    if (lv <= 4) {
        const cfg = SKILL_LEVELS[lv];
        for (let i = 0; i < cfg.count; i++) {
            let angle = (i / cfg.count) * Math.PI * 2;
            bullets.push({ x: player.x, y: player.y, vx: Math.cos(angle) * cfg.speed, vy: Math.sin(angle) * cfg.speed, radius: cfg.size, damage: player.damage * cfg.dmgMult, color: cfg.color, isEnemy: false });
        }
        player.skill_cd = 240;
    } else if (lv === 5) {
        createExplosion(player.x, player.y, "#ff3300", 35);
        enemies.forEach(e => {
            let d = Math.hypot(e.x - player.x, e.y - player.y);
            if (d < 180) { e.hp -= player.damage * 4; let ang = Math.atan2(e.y - player.y, e.x - player.x); e.x += Math.cos(ang) * 40; e.y += Math.sin(ang) * 40; }
        });
        player.skill_cd = 600; 
    } else if (lv === 6) {
        activeClones.push({ x: player.x, y: player.y, life: 90, isLazerClone: false });
        for (let i = 0; i < 20; i++) {
            let angle = (i / 20) * Math.PI * 2;
            bullets.push({ x: player.x, y: player.y, vx: Math.cos(angle) * 7, vy: Math.sin(angle) * 7, radius: 7, damage: player.damage * 2.5, color: "#bf00ff", isEnemy: false });
        }
        player.skill_cd = 600; 
    } else if (lv === 7 || lv === 8) {
        let radius = lv === 7 ? 160 : 240; 
        let dps = lv === 7 ? 7.5 : 16.5; 
        activeAura = { x: player.x, y: player.y, radius: radius, dps: dps, life: 180, lv: lv, isEnemy: false };
        player.skill_cd = 600; 
    } else if (lv === 9) {
        activeBlackHole = { x: canvas.width / 2, y: canvas.height / 2, radius: 10, maxRadius: 220, life: 240, lv: 9, isEnemy: false };
        player.skill_cd = 600; 
    } else if (lv === 10) {
        let bhX = canvas.width / 2; let bhY = canvas.height / 2;
        activeBlackHole = { x: bhX, y: bhY, radius: 10, maxRadius: 640, life: 300, lv: 10, isEnemy: false };
        activeClones = [];
        for(let i=0; i<10; i++) {
            let angle = (i / 10) * Math.PI * 2;
            activeClones.push({ x: bhX + Math.cos(angle) * 180, y: bhY + Math.sin(angle) * 180, life: 300, isLazerClone: true, isEnemy: false });
        }
        player.skill_cd = 600; 
    }
    updateHUD();
}

function getPlayerSkillDamageEst(lv) {
    let baseDmg = player.damage;
    if (lv <= 4) return baseDmg * SKILL_LEVELS[lv].dmgMult;
    if (lv === 5) return baseDmg * 4;
    if (lv === 6) return baseDmg * 2.5;
    if (lv === 7) return baseDmg * 3;
    if (lv === 8) return baseDmg * 5;
    if (lv === 9) return baseDmg * 4;
    return baseDmg * 8; 
}

function spawnLeveledBoss(level = null, isSummoned = false, spawnX = null, spawnY = null) {
    let targetLevel = level !== null ? level : bossSystem.level;
    let loopMultiplier = Math.pow(3, bossSystem.loop - 1);
    
    let calculatedHP = (bossSystem.hpBase * targetLevel) * loopMultiplier;
    if (isSummoned) calculatedHP = Math.floor(calculatedHP * 0.4); 
    
    let rawSkillDmg = getPlayerSkillDamageEst(targetLevel);
    let bossFinalDmg = (rawSkillDmg / 10) * loopMultiplier;
    let bossSpd = Math.min(1.8, 0.6 + (targetLevel * 0.08));

    let bX = spawnX !== null ? spawnX : canvas.width + 40;
    let bY = spawnY !== null ? spawnY : canvas.height / 2;

    let bossObj = {
        x: bX, y: bY,
        hp: calculatedHP, maxHp: calculatedHP, speed: bossSpd,
        color: isSummoned ? "#cc00aa" : "#ff0055", radius: isSummoned ? 18 : 28,
        gold: isSummoned ? 0 : GOLD_REWARD_BOSS, 
        isBoss: true, 
        isSummonedMinion: isSummoned, 
        frozen: false, level: targetLevel, damage: bossFinalDmg,
        skillCooldown: 120, 
        summonCooldown: 600, 
        boss3ShotAngle: 0,
        activeBossAura: null, activeBossBlackHole: null, activeBossClones: []
    };

    enemies.push(bossObj);

    if (!isSummoned) {
        const announceDiv = document.getElementById("boss-announcement");
        announceDiv.innerText = `⚠️ BOSS CẤP ${bossSystem.level} (VÒNG ${bossSystem.loop}) XUẤT HIỆN! ⚠️`;
        announceDiv.style.display = "block";
        setTimeout(() => { announceDiv.style.display = "none"; }, 3000);

        bossSystem.currentBoss = bossObj;
        bossSystem.alive = true;
        bossSystem.countdownActive = false;
        document.getElementById("countdown-overlay").style.display = "none";
        updateHUD();
    }
}

function processBossSkills(boss, idx) {
    if (player.isDead || isShopOpen) return;
    
    if (boss.level === 3) {
        if (boss.skillCooldown > 0) boss.skillCooldown--;
        else {
            boss.boss3ShotAngle += 0.2;
            for (let i = 0; i < 8; i++) {
                bullets.push({ 
                    x: boss.x, y: boss.y, 
                    vx: Math.cos(boss.boss3ShotAngle + i) * 3.5, 
                    vy: Math.sin(boss.boss3ShotAngle + i) * 3.5, 
                    radius: 5, damage: boss.damage, color: "#ff00ff", isEnemy: true 
                });
            }
            boss.skillCooldown = 80;
        }
    }

    if (boss.level === 8) {
        if (boss.summonCooldown > 0) boss.summonCooldown--;
        else {
            spawnLeveledBoss(3, true, boss.x + 40, boss.y + 40);
            boss.summonCooldown = 600; 
        }
    }

    if (boss.activeBossAura) {
        boss.activeBossAura.life--;
        boss.activeBossAura.x = boss.x; boss.activeBossAura.y = boss.y;
        let d = Math.hypot(player.x - boss.activeBossAura.x, player.y - boss.activeBossAura.y);
        if (d < boss.activeBossAura.radius) {
            checkPlayerDamage(boss.activeBossAura.dps / 60);
        }
        if (boss.activeBossAura.life <= 0) boss.activeBossAura = null;
    }

    if (boss.activeBossBlackHole) {
        boss.activeBossBlackHole.life--;
        if (boss.activeBossBlackHole.radius < boss.activeBossBlackHole.maxRadius) {
            boss.activeBossBlackHole.radius += boss.activeBossBlackHole.lv === 10 ? 5 : 2.5;
        }
        let dx = boss.activeBossBlackHole.x - player.x; let dy = boss.activeBossBlackHole.y - player.y;
        let dist = Math.hypot(dx, dy);
        if (dist > 5) {
            let pull = boss.activeBossBlackHole.lv === 10 ? 2.5 : 1.5;
            player.x += (dx / dist) * pull; player.y += (dy / dist) * pull;
        }
        checkPlayerDamage(boss.activeBossBlackHole.lv === 9 ? 0.05 : 0.2);

        if (boss.activeBossBlackHole.life <= 0) {
            boss.activeBossBlackHole = null; boss.activeBossClones = [];
        }
    }

    for (let i = boss.activeBossClones.length - 1; i >= 0; i--) {
        let c = boss.activeBossClones[i]; c.life--;
        if (c.isLazerClone) {
            let dToPl = Math.hypot(player.x - c.x, player.y - c.y);
            if (dToPl < 500) {
                checkPlayerDamage(0.15);
            }
        }
        if (c.life <= 0) boss.activeBossClones.splice(i, 1);
    }

    if (boss.level === 3) return; 
    if (boss.skillCooldown > 0) { boss.skillCooldown--; return; }

    const lv = boss.level; boss.skillCooldown = 240; 

    if (lv <= 4) {
        let count = SKILL_LEVELS[lv].count || 8; let speed = SKILL_LEVELS[lv].speed || 5;
        let size = SKILL_LEVELS[lv].size || 5; let color = SKILL_LEVELS[lv].color || "#ff00ff";
        for (let i = 0; i < count; i++) {
            let angle = (i / count) * Math.PI * 2;
            bullets.push({ x: boss.x, y: boss.y, vx: Math.cos(angle) * (speed * 0.6), vy: Math.sin(angle) * (speed * 0.6), radius: size, damage: boss.damage, color: color, isEnemy: true });
        }
    } else if (lv === 5) {
        createExplosion(boss.x, boss.y, "#ff3300", 25);
        let d = Math.hypot(player.x - boss.x, player.y - boss.y);
        if (d < 200) {
            let ang = Math.atan2(player.y - boss.y, player.x - boss.x);
            player.x += Math.cos(ang) * 35; player.y += Math.sin(ang) * 35;
            checkPlayerDamage(boss.damage * 2);
        }
    } else if (lv === 6) {
        boss.activeBossClones.push({ x: boss.x, y: boss.y, life: 100, isLazerClone: false });
        for (let i = 0; i < 16; i++) {
            let angle = (i / 16) * Math.PI * 2;
            bullets.push({ x: boss.x, y: boss.y, vx: Math.cos(angle) * 4.5, vy: Math.sin(angle) * 4.5, radius: 6, damage: boss.damage * 1.5, color: "#e600ff", isEnemy: true });
        }
    } else if (lv === 7 || lv === 8) {
        let radius = lv === 7 ? 150 : 220; let dps = lv === 7 ? boss.damage * 0.5 : boss.damage * 1.2;
        boss.activeBossAura = { x: boss.x, y: boss.y, radius: radius, dps: dps, life: 180, lv: lv };
    } else if (lv === 9) {
        boss.activeBossBlackHole = { x: canvas.width / 2, y: canvas.height / 2, radius: 10, maxRadius: 200, life: 200, lv: 9 };
    } else if (lv === 10) {
        let bhX = canvas.width / 2; let bhY = canvas.height / 2;
        boss.activeBossBlackHole = { x: bhX, y: bhY, radius: 10, maxRadius: 550, life: 300, lv: 10 };
        boss.activeBossClones = [];
        for (let i = 0; i < 10; i++) {
            let angle = (i / 10) * Math.PI * 2;
            boss.activeBossClones.push({ x: bhX + Math.cos(angle) * 160, y: bhY + Math.sin(angle) * 160, life: 300, isLazerClone: true });
        }
    }
}

function checkPlayerDamage(amount) {
    if (player.isDead) return;
    player.hp -= amount;
    if (player.hp <= 0) {
        if (player.hasRevive) {
            player.hasRevive = false; 
            player.hp = player.maxHp;  
            createExplosion(player.x, player.y, "#ff33aa", 40); 
            document.getElementById("ui-revive-indicator").style.display = "none";
            const announceDiv = document.getElementById("boss-announcement");
            announceDiv.innerText = `🛡️ ĐÃ KÍCH HOẠT HỒI SINH THẦN THÁNH! 🛡️`;
            announceDiv.style.display = "block";
            setTimeout(() => { announceDiv.style.display = "none"; }, 2500);
        } else {
            endGame();
        }
    }
    updateHUD();
}

function spawnNormalEnemy() {
    if (isShopOpen || player.isDead) return;
    let x, y;
    if (Math.random() < 0.5) { x = Math.random() < 0.5 ? -20 : canvas.width + 20; y = Math.random() * canvas.height; } 
    else { x = Math.random() * canvas.width; y = Math.random() < 0.5 ? -20 : canvas.height + 20; }
    enemies.push({ x, y, hp: 20 + player.level * 4, speed: 1.2, color: "#88cc00", radius: 11, gold: GOLD_REWARD_NORMAL, isBoss: false, frozen: false });
}

function createExplosion(x, y, color, count = 8) {
    for (let i = 0; i < count; i++) { particles.push({ x, y, vx: (Math.random() - 0.5) * 6, vy: (Math.random() - 0.5) * 6, radius: Math.random() * 3 + 1, life: 20, color }); }
}

function toggleShop(open) {
    if (player.isDead) return;
    isShopOpen = open;
    document.getElementById("shop-modal").style.display = open ? "block" : "none";
    if(open) {
        document.getElementById("shop-gold").innerText = player.gold;
        document.getElementById("shop-current-mult").innerText = `x${player.goldMultiplier} Vàng`;
        document.getElementById("stat-atk").innerText = player.damage;
        document.getElementById("stat-maxhp").innerText = player.maxHp;
        document.getElementById("stat-spd").innerText = player.speed.toFixed(1);
        document.getElementById("cost-atk").innerText = shopCosts.atk;
        document.getElementById("cost-maxhp").innerText = shopCosts.maxhp;
        document.getElementById("cost-speed").innerText = shopCosts.speed;
        
        if (player.skill_lv < 10) {
            document.getElementById("shop-btn-upgrade-skill").innerText = `${shopCosts.skill_up} Vàng`;
            document.getElementById("shop-btn-upgrade-skill").disabled = false;
            document.getElementById("skill-desc").innerHTML = `<span style="color:#00ffff; font-weight:bold;">Cấp ${player.skill_lv}:</span> ${SKILL_LEVELS[player.skill_lv].name || "Tuyệt Kỹ"}<br><span style="color:#00ff66;font-weight:bold;">Nâng Cấp Cấp ${player.skill_lv+1}:</span> ${SKILL_LEVELS[player.skill_lv+1].desc}`;
        } else {
            document.getElementById("shop-btn-upgrade-skill").innerText = "MAX LEVEL"; document.getElementById("shop-btn-upgrade-skill").disabled = true;
            document.getElementById("skill-desc").innerHTML = `<span style="color:#ff00ff; font-weight:bold;">ĐÃ ĐẠT CẢNH GIỚI VÔ ĐỊCH THẦN MA!</span>`;
        }
        
        document.getElementById("shop-btn-aim").innerText = player.aimPassive ? "ĐÃ CÓ" : "100 Vàng";
        document.getElementById("shop-btn-aim").disabled = player.aimPassive;
        
        document.getElementById("shop-btn-revive").innerText = player.hasRevive ? "ĐÃ CÓ SẴN" : "150.000 Vàng";
        document.getElementById("shop-btn-revive").disabled = player.hasRevive;

        document.getElementById("btn-gold-x2").disabled = (player.goldMultiplier === 2);
        document.getElementById("btn-gold-x4").disabled = (player.goldMultiplier === 4);
        document.getElementById("btn-gold-x100").disabled = (player.goldMultiplier === 100);
        document.getElementById("btn-gold-x1000").disabled = (player.goldMultiplier === 1000);
    }
}

function switchTab(tab) {
    currentTab = tab;
    document.getElementById("tab-stat").classList.toggle("active", tab === 'stat');
    document.getElementById("tab-skill").classList.toggle("active", tab === 'skill');
    document.getElementById("tab-money").classList.toggle("active", tab === 'money');
    document.getElementById("panel-stat").style.display = tab === 'stat' ? 'block' : 'none';
    document.getElementById("panel-skill").style.display = tab === 'skill' ? 'block' : 'none';
    document.getElementById("panel-money").style.display = tab === 'money' ? 'block' : 'none';
}

function buyUpgrade(type) {
    if (type === 'heal' && player.gold >= 20) { player.gold -= 20; player.hp = player.maxHp; }
    else if (type === 'atk' && player.gold >= shopCosts.atk) { player.gold -= shopCosts.atk; player.damage += 5; shopCosts.atk = Math.floor(shopCosts.atk * 1.4); }
    else if (type === 'maxhp' && player.gold >= shopCosts.maxhp) { player.gold -= shopCosts.maxhp; player.maxHp += 20; player.hp += 20; shopCosts.maxhp = Math.floor(shopCosts.maxhp * 1.4); }
    else if (type === 'speed' && player.gold >= shopCosts.speed) { player.gold -= shopCosts.speed; player.speed += 0.4; shopCosts.speed = Math.floor(shopCosts.speed * 1.4); }
    else if (type === 'upgrade_skill' && player.skill_lv < 10 && player.gold >= shopCosts.skill_up) {
        player.gold -= shopCosts.skill_up; player.skill_lv++; shopCosts.skill_up = Math.floor(shopCosts.skill_up * 1.7);
        document.getElementById("txt-skill-name").innerText = `SKILL LV.${player.skill_lv}`;
    }
    else if (type === 'aim_passive' && !player.aimPassive && player.gold >= 100) { player.gold -= 100; player.aimPassive = true; }
    else if (type === 'revive_passive' && !player.hasRevive && player.gold >= 150000) { player.gold -= 150000; player.hasRevive = true; document.getElementById("ui-revive-indicator").style.display = "inline"; }
    else if (type === 'gold_x2' && player.gold >= 100) { player.gold -= 100; player.goldMultiplier = 2; }
    else if (type === 'gold_x4' && player.gold >= 400) { player.gold -= 400; player.goldMultiplier = 4; }
    else if (type === 'gold_x100' && player.gold >= 2000) { player.gold -= 2000; player.goldMultiplier = 100; }
    else if (type === 'gold_x1000' && player.gold >= 20000) { player.gold -= 20000; player.goldMultiplier = 1000; }
    else { return; }
    toggleShop(true); updateHUD();
}

function updateHUD() {
    document.getElementById("ui-hp").innerText = Math.max(0, Math.floor(player.hp));
    document.getElementById("ui-level").innerText = player.level;
    document.getElementById("ui-gold").innerText = player.gold;
    
    let sec = Math.floor(bossSystem.survivalFrames / 60);
    document.getElementById("ui-survive").innerText = `${sec}s`;
    document.getElementById("ui-boss-state").innerText = `LV.${bossSystem.level} (Vòng ${bossSystem.loop})`;
    document.getElementById("sk-cd").innerText = player.skill_cd > 0 ? `${Math.ceil(player.skill_cd / 60)}s` : "SẴN SÀNG";
}

function endGame() {
    player.isDead = true; 
    document.getElementById("final-level").innerText = player.level;
    document.getElementById("final-survive-time").innerText = `${Math.floor(bossSystem.survivalFrames / 60)} giây`;
    document.getElementById("game-over-screen").style.display = "flex";
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
}

function restartGame() {
    player.x = canvas.width / 2; player.y = canvas.height / 2;
    player.hp = 100; player.maxHp = 100; player.level = 1; player.exp = 0; player.nextExp = 100; player.gold = 0; player.damage = 15; player.speed = 4; player.direction = "right";
    player.skill_lv = 1; player.skill_cd = 0; player.aimPassive = false; player.isDead = false; player.goldMultiplier = 1; player.hasRevive = false;

    bossSystem.level = 1; bossSystem.loop = 1; bossSystem.alive = false; bossSystem.currentBoss = null; bossSystem.countdownActive = false; bossSystem.survivalFrames = 0;

    shopCosts = { atk: 30, maxhp: 40, speed: 50, skill_up: 60 };
    bullets = []; enemies = []; particles = []; activeClones = []; activeAura = null; activeBlackHole = null;
    normalEnemyTimer = 0; joystickTouchId = null; dragInput.active = false; dragInput.moveX = 0; dragInput.moveY = 0;

    document.getElementById("txt-skill-name").innerText = "SKILL LV.1";
    document.getElementById("game-over-screen").style.display = "none";
    document.getElementById("countdown-overlay").style.display = "none";
    document.getElementById("ui-revive-indicator").style.display = "none";
    
    toggleShop(false); updateHUD();
    if (animationFrameId) cancelAnimationFrame(animationFrameId);
    spawnLeveledBoss(); 
    animationFrameId = requestAnimationFrame(updateGame);
}

function updateGame() {
    if (player.isDead) return;

    if (!isShopOpen) {
        bossSystem.survivalFrames++; 

        let moved = false;
        if (dragInput.active) { player.x += dragInput.moveX * player.speed; player.y += dragInput.moveY * player.speed; moved = true; } 
        if (!moved) {
            if (inputState.up) { player.y -= player.speed; player.direction = "up"; }
            if (inputState.down) { player.y += player.speed; player.direction = "down"; }
            if (inputState.left) { player.x -= player.speed; player.direction = "left"; }
            if (inputState.right) { player.x += player.speed; player.direction = "right"; }
        }

        player.x = Math.max(player.radius, Math.min(canvas.width - player.radius, player.x));
        player.y = Math.max(player.radius, Math.min(canvas.height - player.radius, player.y));
        if (player.skill_cd > 0) player.skill_cd--;

        if (bossSystem.countdownActive) {
            bossSystem.countdownFrames--;
            let remSec = Math.ceil(bossSystem.countdownFrames / 60);
            document.getElementById("countdown-time").innerText = remSec;
            if (bossSystem.countdownFrames <= 0) {
                spawnLeveledBoss();
            }
        }

        normalEnemyTimer++;
        if (normalEnemyTimer >= 140) { spawnNormalEnemy(); normalEnemyTimer = 0; }

        if (activeAura) { activeAura.life--; activeAura.x = player.x; activeAura.y = player.y; if (activeAura.life <= 0) activeAura = null; }
        if (activeBlackHole) {
            activeBlackHole.life--;
            if (activeBlackHole.radius < activeBlackHole.maxRadius) activeBlackHole.radius += activeBlackHole.lv === 10 ? 6 : 3;
            if (activeBlackHole.life <= 0) {
                if (activeBlackHole.lv === 10) {
                    enemies.forEach(e => { let d = Math.hypot(e.x - activeBlackHole.x, e.y - activeBlackHole.y); if (d < 160) e.hp = 0; });
                    activeClones = [];
                }
                activeBlackHole = null;
            }
        }
        for(let i = activeClones.length-1; i >= 0; i--) { activeClones[i].life--; if(activeClones[i].life <= 0) activeClones.splice(i, 1); }

        for (let i = bullets.length - 1; i >= 0; i--) {
            let b = bullets[i]; b.x += b.vx; b.y += b.vy;
            if (b.isEnemy) {
                let d = Math.hypot(player.x - b.x, player.y - b.y);
                if (d < player.radius + b.radius) {
                    bullets.splice(i, 1);
                    checkPlayerDamage(b.damage);
                    continue;
                }
            }
            if (b.x < 0 || b.x > canvas.width || b.y < 0 || b.y > canvas.height) bullets.splice(i, 1);
        }

        for (let i = enemies.length - 1; i >= 0; i--) {
            let e = enemies[i]; e.frozen = false;

            if (e.isBoss) { processBossSkills(e, i); }

            if (activeBlackHole) {
                let dx = activeBlackHole.x - e.x; let dy = activeBlackHole.y - e.y; let dist = Math.hypot(dx, dy);
                if (dist > 5) { let pull = activeBlackHole.lv === 10 ? 4.5 : 3.0; e.x += (dx / dist) * pull; e.y += (dy / dist) * pull; }
                e.hp -= activeBlackHole.lv === 9 ? 0.3 : 1.2;
            }

            if (activeBlackHole && activeBlackHole.lv === 10) {
                activeClones.forEach(c => { if (Math.hypot(e.x - c.x, e.y - c.y) < 500) e.hp -= 0.6; });
            }

            if (activeAura) { if (Math.hypot(e.x - activeAura.x, e.y - activeAura.y) < activeAura.radius) { e.frozen = true; e.hp -= activeAura.dps; } }

            if (!e.frozen) {
                let dx = player.x - e.x; let dy = player.y - e.y; let dist = Math.hypot(dx, dy);
                if (dist > 1) { e.x += (dx / dist) * e.speed; e.y += (dy / dist) * e.speed; }
                if (dist < player.radius + e.radius) {
                    checkPlayerDamage(e.isBoss ? 0.6 : 0.3);
                }
            }

            for (let j = bullets.length - 1; j >= 0; j--) {
                let b = bullets[j]; if (b.isEnemy) continue;
                if (Math.hypot(b.x - e.x, b.y - e.y) < b.radius + e.radius) {
                    e.hp -= b.damage; bullets.splice(j, 1);
                    
                    if (e.hp <= 0) {
                        createExplosion(e.x, e.y, e.color, e.isBoss ? 50 : 6);
                        
                        if (!e.isSummonedMinion) {
                            player.gold += (e.gold * player.goldMultiplier); 
                        }
                        
                        player.exp += e.isBoss ? 100 : 12;
                        
                        // FIX TẠI ĐÂY: Khi Boss chết, hủy ngay lập tức các kỹ năng Hố Đen / Phân Thân của nó
                        if (e.isBoss && !e.isSummonedMinion) {
                            e.activeBossBlackHole = null;
                            e.activeBossClones = [];
                            e.activeBossAura = null;
                            
                            bossSystem.alive = false;
                            bossSystem.currentBoss = null; // Xóa tham chiếu boss hiện tại
                            if (bossSystem.level === 10) { bossSystem.level = 1; bossSystem.loop++; } else { bossSystem.level++; }
                            bossSystem.countdownActive = true; bossSystem.countdownFrames = 1800; 
                            document.getElementById("countdown-overlay").style.display = "block";
                        }
                        enemies.splice(i, 1); break;
                    }
                }
            }
        }

        for (let i = enemies.length - 1; i >= 0; i--) {
            if (enemies[i].hp <= 0) {
                createExplosion(enemies[i].x, enemies[i].y, enemies[i].color, enemies[i].isBoss ? 50 : 6);
                
                if (!enemies[i].isSummonedMinion) {
                    player.gold += (enemies[i].gold * player.goldMultiplier);
                }
                
                player.exp += enemies[i].isBoss ? 100 : 12;
                
                // ĐỒNG BỘ FIX CHO VÒNG LẶP KIỂM TRA HP PHỤ
                if (enemies[i].isBoss && !enemies[i].isSummonedMinion) {
                    enemies[i].activeBossBlackHole = null;
                    enemies[i].activeBossClones = [];
                    enemies[i].activeBossAura = null;

                    bossSystem.alive = false;
                    bossSystem.currentBoss = null;
                    if (bossSystem.level === 10) { bossSystem.level = 1; bossSystem.loop++; } else { bossSystem.level++; }
                    bossSystem.countdownActive = true; bossSystem.countdownFrames = 1800; 
                    document.getElementById("countdown-overlay").style.display = "block";
                }
                enemies.splice(i, 1);
            }
        }

        for (let i = particles.length - 1; i >= 0; i--) { let p = particles[i]; p.x += p.vx; p.y += p.vy; p.life--; if (p.life <= 0) particles.splice(i, 1); }
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (activeAura) {
        let grad = ctx.createRadialGradient(activeAura.x, activeAura.y, 10, activeAura.x, activeAura.y, activeAura.radius);
        grad.addColorStop(0, activeAura.lv === 7 ? 'rgba(0, 255, 255, 0.35)' : 'rgba(180, 0, 255, 0.35)'); grad.addColorStop(1, 'rgba(0,0,0,0)');
        ctx.fillStyle = grad; ctx.beginPath(); ctx.arc(activeAura.x, activeAura.y, activeAura.radius, 0, Math.PI * 2); ctx.fill();
    }

    if (bossSystem.currentBoss && bossSystem.currentBoss.activeBossAura) {
        let ba = bossSystem.currentBoss.activeBossAura;
        let bGrad = ctx.createRadialGradient(ba.x, ba.y, 10, ba.x, ba.y, ba.radius);
        bGrad.addColorStop(0, 'rgba(255, 0, 50, 0.3)'); bGrad.addColorStop(1, 'rgba(0,0,0,0)');
        ctx.fillStyle = bGrad; ctx.beginPath(); ctx.arc(ba.x, ba.y, ba.radius, 0, Math.PI * 2); ctx.fill();
    }

    if (activeBlackHole) {
        let bhGrad = ctx.createRadialGradient(activeBlackHole.x, activeBlackHole.y, 0, activeBlackHole.x, activeBlackHole.y, activeBlackHole.radius);
        if (activeBlackHole.lv === 9) { bhGrad.addColorStop(0, '#000000'); bhGrad.addColorStop(0.4, '#2d004d'); bhGrad.addColorStop(1, 'rgba(0,0,0,0)'); }
        else { bhGrad.addColorStop(0, '#ffffff'); bhGrad.addColorStop(0.1, '#000000'); bhGrad.addColorStop(0.6, '#4b0082'); bhGrad.addColorStop(1, 'rgba(0,0,0,0)'); }
        ctx.fillStyle = bhGrad; ctx.beginPath(); ctx.arc(activeBlackHole.x, activeBlackHole.y, activeBlackHole.radius, 0, Math.PI * 2); ctx.fill();
    }

    if (bossSystem.currentBoss && bossSystem.currentBoss.activeBossBlackHole) {
        let bbh = bossSystem.currentBoss.activeBossBlackHole;
        let bbhGrad = ctx.createRadialGradient(bbh.x, bbh.y, 0, bbh.x, bbh.y, bbh.radius);
        bbhGrad.addColorStop(0, '#ff0000'); bbhGrad.addColorStop(0.2, '#000000'); bbhGrad.addColorStop(1, 'rgba(0,0,0,0)');
        ctx.fillStyle = bbhGrad; ctx.beginPath(); ctx.arc(bbh.x, bbh.y, bbh.radius, 0, Math.PI * 2); ctx.fill();
    }

    particles.forEach(p => { ctx.fillStyle = p.color; ctx.globalAlpha = p.life / 20; ctx.beginPath(); ctx.arc(p.x, p.y, p.radius, 0, Math.PI * 2); ctx.fill(); ctx.globalAlpha = 1.0; });
    
    activeClones.forEach(c => { 
        ctx.fillStyle = "rgba(160, 0, 255, 0.6)"; ctx.beginPath(); ctx.arc(c.x, c.y, player.radius, 0, Math.PI * 2); ctx.fill(); 
        if (c.isLazerClone) {
            enemies.forEach(e => {
                if (Math.hypot(e.x - c.x, e.y - c.y) < 500) {
                    ctx.save(); ctx.strokeStyle = "rgba(220, 0, 255, 0.8)"; ctx.lineWidth = Math.random() * 3 + 1.5;
                    ctx.beginPath(); ctx.moveTo(c.x, c.y); ctx.lineTo(e.x, e.y); ctx.stroke(); ctx.restore();
                }
            });
        }
    });

    if (bossSystem.currentBoss) {
        bossSystem.currentBoss.activeBossClones.forEach(bc => {
            ctx.fillStyle = "rgba(255, 0, 50, 0.6)"; ctx.beginPath(); ctx.arc(bc.x, bc.y, 14, 0, Math.PI * 2); ctx.fill();
            if (bc.isLazerClone) {
                if (Math.hypot(player.x - bc.x, player.y - bc.y) < 500) {
                    ctx.save(); ctx.strokeStyle = "rgba(255, 50, 0, 0.8)"; ctx.lineWidth = Math.random() * 2 + 1;
                    ctx.beginPath(); ctx.moveTo(bc.x, bc.y); ctx.lineTo(player.x, player.y); ctx.stroke(); ctx.restore();
                }
            }
        });
    }

    bullets.forEach(b => { ctx.fillStyle = b.color; ctx.beginPath(); ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2); ctx.fill(); });
    
    enemies.forEach(e => {
        ctx.fillStyle = e.frozen ? "#00bfff" : e.color; ctx.beginPath(); ctx.arc(e.x, e.y, e.radius, 0, Math.PI * 2); ctx.fill();
        if (e.isBoss) {
            let barW = 80; let barH = 6; ctx.fillStyle = "#444"; ctx.fillRect(e.x - barW/2, e.y - e.radius - 12, barW, barH);
            ctx.fillStyle = "#ff0000"; ctx.fillRect(e.x - barW/2, e.y - e.radius - 12, barW * (e.hp / e.maxHp), barH);
            ctx.fillStyle = "#fff"; ctx.font = "10px monospace"; 
            
            let bossText = e.isSummonedMinion ? `ĐỆ BOSS LV.${e.level}` : `BOSS LV.${e.level}`;
            ctx.fillText(bossText, e.x - 30, e.y - e.radius - 16);
        }
    });

    ctx.fillStyle = "#00ff66"; ctx.beginPath(); ctx.arc(player.x, player.y, player.radius, 0, Math.PI * 2); ctx.fill();
    
    ctx.strokeStyle = "#ffffff"; ctx.lineWidth = 3; ctx.beginPath(); ctx.moveTo(player.x, player.y);
    let drawnAim = false;
    if (player.aimPassive && enemies.length > 0) {
        let target = null; let minD = Infinity;
        enemies.forEach(e => { let d = Math.hypot(e.x - player.x, e.y - player.y); if(d < minD){ minD = d; target = e; } });
        if (target) { let ang = Math.atan2(target.y - player.y, target.x - player.x); ctx.lineTo(player.x + Math.cos(ang)*22, player.y + Math.sin(ang)*22); drawnAim = true; }
    }
    if (!drawnAim) {
        if (player.direction === "up") ctx.lineTo(player.x, player.y - 22); if (player.direction === "down") ctx.lineTo(player.x, player.y + 22);
        if (player.direction === "left") ctx.lineTo(player.x - 22, player.y); if (player.direction === "right") ctx.lineTo(player.x + 22, player.y);
    }
    ctx.stroke();

    updateHUD();
    animationFrameId = requestAnimationFrame(updateGame);
}

setupInputControls();
spawnLeveledBoss(); 
animationFrameId = requestAnimationFrame(updateGame);
</script>
</body>
</html>
