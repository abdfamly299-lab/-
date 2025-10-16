<!doctype html>

<html lang="ar">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1" />
<title>لعبة باتل رويال بسيطة</title>
<style>
  html,body{height:100%;margin:0;background:#0b0b0f;color:#ddd;font-family:Tahoma, Arial}
  #game{display:block;width:100%;height:100vh;background:#06060a}
  .hud{position:fixed;left:10px;top:10px;z-index:10}
  .btn{position:fixed;right:18px;bottom:18px;width:78px;height:78px;border-radius:50%;background:rgba(255,255,255,0.06);display:flex;align-items:center;justify-content:center;font-weight:bold}
  #joystick{position:fixed;left:10px;bottom:10px;width:140px;height:140px;border-radius:50%;background:rgba(255,255,255,0.03);touch-action:none}
  #controlsHint{position:fixed;left:50%;transform:translateX(-50%);bottom:14px;color:#999;font-size:12px}
  #info{position:fixed;right:10px;top:10px;text-align:right}
  #restart{position:fixed;left:50%;top:10px;transform:translateX(-50%);background:#ffd166;color:#111;padding:6px 10px;border-radius:6px;display:none}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div class="hud"><div id="score">قتلات: 0</div></div>
<div id="info"><div id="health">صحة: 100</div><div id="zone">المنطقة: كبيرة</div></div>
<div id="joystick"></div>
<div class="btn" id="shoot">اطلق</div>
<div id="controlsHint">حرّك باليد اليسرى — اطلق بالزر اليمين</div>
<button id="restart">اعادة اللعب</button>
<script>
// --- إعداد اللعبة ---
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
let DPR = window.devicePixelRatio || 1;
function resize(){
  canvas.width = innerWidth * DPR;
  canvas.height = innerHeight * DPR;
  canvas.style.width = innerWidth + 'px';
  canvas.style.height = innerHeight + 'px';
  ctx.setTransform(DPR,0,0,DPR,0,0);
}
addEventListener('resize', resize);
resize();// عناصر اللعب const player = {x:innerWidth/2, y:innerHeight/2, r:18, ang:0, speed:220, hp:100}; let bullets = []; let enemies = []; let score = 0; let running = true; let last = performance.now(); let spawnTimer = 0; let safeRadius = Math.max(innerWidth, innerHeight) * 0.7; let zoneCenter = {x: innerWidth/2, y: innerHeight/2}; let zoneShrinkTimer = 0;

// تحكمات اللمس: joystick const joystick = document.getElementById('joystick'); let joy = {active:false, startX:0, startY:0, dx:0, dy:0}; joystick.addEventListener('pointerdown', e=>{ e.preventDefault(); joystick.setPointerCapture(e.pointerId); joy.active=true; joy.startX=e.clientX; joy.startY=e.clientY;}); joystick.addEventListener('pointermove', e=>{ if(!joy.active) return; joy.dx = e.clientX - joy.startX; joy.dy = e.clientY - joy.startY; const max=60; const len=Math.hypot(joy.dx,joy.dy); if(len>max){ joy.dx = joy.dx/lenmax; joy.dy = joy.dy/lenmax; } }); joystick.addEventListener('pointerup', e=>{ joy.active=false; joy.dx=0; joy.dy=0;});

// زر اطلاق const shootBtn = document.getElementById('shoot'); let firing=false; shootBtn.addEventListener('pointerdown', e=>{ firing=true; }); shootBtn.addEventListener('pointerup', e=>{ firing=false; }); shootBtn.addEventListener('pointercancel', e=>{ firing=false; });

// موسيقى / صوت بسيط: نستخدم WebAudio API بسيط let audioCtx; function beep(freq, t=0.06){ if(!audioCtx) audioCtx = new (window.AudioContext||window.webkitAudioContext)(); const o = audioCtx.createOscillator(); const g = audioCtx.createGain(); o.type='sine'; o.frequency.value=freq; g.gain.value=0.06; o.connect(g); g.connect(audioCtx.destination); o.start(); o.stop(audioCtx.currentTime + t); }

// إطلاق الرصاص function fire(){ const ang = Math.atan2(joy.dy, joy.dx); // إذا لا يوجد إدخال من الجويستيك، اطلاق في اتجاه تصويب افتراضي (إلى الأمام) const dirX = joy.dx!==0 || joy.dy!==0 ? Math.cos(ang) : Math.cos(player.ang); const dirY = joy.dx!==0 || joy.dy!==0 ? Math.sin(ang) : Math.sin(player.ang); bullets.push({x:player.x + dirX*(player.r+6), y:player.y + dirY*(player.r+6), vx:dirX700, vy:dirY700, life:1.6}); beep(900,0.03); }

// توليد أعداء عشوائيين function spawnEnemy(){ const edge = Math.floor(Math.random()*4); let x,y; if(edge===0){ x = -40; y = Math.random()*innerHeight; } else if(edge===1){ x = innerWidth + 40; y = Math.random()*innerHeight; } else if(edge===2){ x = Math.random()*innerWidth; y = -40; } else { x = Math.random()*innerWidth; y = innerHeight + 40; } const sz = 14 + Math.random()*14; enemies.push({x,y,r:sz,hp:20+Math.floor(Math.random()*30), speed:40+Math.random()*40}); }

// تحديث كل فريم function update(dt){ if(!running) return; // حركة لاعب من الجويستيك if(joy.dx!==0 || joy.dy!==0){ const vx = (joy.dx/60) * player.speed; const vy = (joy.dy/60) * player.speed; player.x += vx * dt; player.y += vy * dt; player.ang = Math.atan2(vy, vx); } // حدود الشاشة player.x = Math.max(20, Math.min(innerWidth-20, player.x)); player.y = Math.max(20, Math.min(innerHeight-20, player.y));

// اطلاق الرصاص الالي if(firing){ // تبطئة إطلاق if(!fire._cooldown) fire._cooldown = 0; fire._cooldown -= dt; if(fire._cooldown <= 0){ fire(); fire._cooldown = 0.18; } } else fire._cooldown = 0;

// تحديث رصاصات bullets = bullets.filter(b=>b.life>0); for(const b of bullets){ b.x += b.vxdt; b.y += b.vydt; b.life -= dt; }

// أعداء يتحركون نحو اللاعب for(const e of enemies){ const dx = player.x - e.x; const dy = player.y - e.y; const d = Math.hypot(dx,dy) || 1; e.x += (dx/d) * e.speed * dt; e.y += (dy/d) * e.speed * dt; }

// ارتطام رصاص بأعداء for(const b of bullets){ for(const e of enemies){ const dx=b.x-e.x, dy=b.y-e.y; if(Math.hypot(dx,dy) < e.r + 4){ e.hp -= 30; b.life = 0; beep(1200,0.02); } } }

// اصتدام لاعب مع عدو for(const e of enemies){ const d = Math.hypot(player.x - e.x, player.y - e.y); if(d < player.r + e.r){ // يلحق الضرر ببطء player.hp -= 20 * dt; } }

// إزالة الأعداء الميتين for(let i=enemies.length-1;i>=0;i--){ if(enemies[i].hp<=0){ enemies.splice(i,1); score++; document.getElementById('score').textContent = 'قتلات: '+score; } }

// إحداثيات الرصاص خارج الشاشة for(let i=bullets.length-1;i>=0;i--){ const b=bullets[i]; if(b.x<-50||b.x>innerWidth+50||b.y<-50||b.y>innerHeight+50||b.life<=0) bullets.splice(i,1); }

// توليد أعداء تدريجيًا أسرع spawnTimer += dt; if(spawnTimer > Math.max(0.6, 1.6 - Math.min(1.2, score*0.06))){ spawnEnemy(); spawnTimer=0; }

// تصغير المنطقة الآمنة مع الوقت zoneShrinkTimer += dt; if(zoneShrinkTimer > 8 && safeRadius > Math.min(innerWidth, innerHeight)*0.18){ safeRadius -= 8; zoneShrinkTimer=0; }

// أذى خارج المنطقة الآمنة const distFromZone = Math.hypot(player.x - zoneCenter.x, player.y - zoneCenter.y); if(distFromZone > safeRadius){ player.hp -= 10 * dt; document.getElementById('zone').textContent = 'المنطقة: خارج!'; } else document.getElementById('zone').textContent = 'المنطقة: داخل';

// تحديث HUD صحة document.getElementById('health').textContent = 'صحة: ' + Math.max(0, Math.floor(player.hp));

if(player.hp <= 0) endGame(); }

function endGame(){ running=false; document.getElementById('restart').style.display='block'; beep(220,0.25); }

// رسم function draw(){ ctx.clearRect(0,0,innerWidth,innerHeight); // خلفية شبحي ctx.fillStyle = '#061022'; ctx.fillRect(0,0,innerWidth,innerHeight);

// منطقة آمنة ctx.beginPath(); ctx.arc(zoneCenter.x, zoneCenter.y, safeRadius, 0, Math.PI2); ctx.fillStyle = 'rgba(50,60,90,0.06)'; ctx.fill(); ctx.beginPath(); ctx.arc(zoneCenter.x, zoneCenter.y, safeRadius, 0, Math.PI2); ctx.strokeStyle='rgba(120,140,200,0.12)'; ctx.lineWidth=2; ctx.stroke();

// لاعب ctx.save(); ctx.translate(player.x, player.y); ctx.rotate(player.ang); ctx.beginPath(); ctx.arc(0,0,player.r,0,Math.PI2); ctx.fillStyle='#58d68d'; ctx.fill(); // علامة اتجاه ctx.beginPath(); ctx.moveTo(player.r0.6,0); ctx.lineTo(player.r1.4, -6); ctx.lineTo(player.r1.4,6); ctx.closePath(); ctx.fillStyle='#2b7a4b'; ctx.fill(); ctx.restore();

// رصاصات for(const b of bullets){ ctx.beginPath(); ctx.arc(b.x,b.y,4,0,Math.PI*2); ctx.fillStyle='#ffd166'; ctx.fill(); }

// أعداء for(const e of enemies){ ctx.beginPath(); ctx.arc(e.x,e.y,e.r,0,Math.PI*2); ctx.fillStyle='#ef476f'; ctx.fill(); ctx.fillStyle='rgba(255,255,255,0.7)'; ctx.font='11px Tahoma'; ctx.textAlign='center'; ctx.fillText(Math.max(0,Math.floor(e.hp)), e.x, e.y+4); }

// قياسات ctx.fillStyle='rgba(255,255,255,0.06)'; ctx.fillRect(8,innerHeight-28,120,18); ctx.fillStyle='#ff6b6b'; ctx.fillRect(10,innerHeight-26, Math.max(0, (player.hp/100)*116 ),12);

// تلميح الجويستيك ctx.fillStyle='rgba(255,255,255,0.02)'; ctx.fillRect(0,0,0,0); }

// حلقة اللعبة function loop(t){ const dt = Math.min(0.05, (t - last)/1000); last = t; update(dt); draw(); requestAnimationFrame(loop); } requestAnimationFrame(loop);

// زر إعادة التشغيل document.getElementById('restart').addEventListener('click', ()=>{ // إعادة تهيئة bullets = []; enemies = []; score = 0; player.hp=100; player.x=innerWidth/2; player.y=innerHeight/2; running=true; document.getElementById('score').textContent='قتلات: 0'; document.getElementById('restart').style.display='none'; safeRadius = Math.max(innerWidth, innerHeight) * 0.7; zoneCenter = {x: innerWidth/2, y: innerHeight/2}; spawnTimer=0; zoneShrinkTimer=0; last=performance.now(); requestAnimationFrame(loop); });

// اختصارات لوحية/لوحة مفاتيح (للتجريب على الكمبيوتر) addEventListener('keydown', e=>{ if(e.key==='w') player.y -= 10; if(e.key==='s') player.y += 10; if(e.key==='a') player.x -= 10; if(e.key==='d') player.x += 10; if(e.key===' ') firing=true; }); addEventListener('keyup', e=>{ if(e.key===' ') firing=false; });

// نصائح الاستخدام console.log('نصائح: احفظ الملف باسم game.html وافتحه في متصفح الموبايل. استخدم اليد اليسرى للتحرك والزر الأيمن للإطلاق.'); </script>

</body>
</html>
