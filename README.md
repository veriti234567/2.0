<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#1a1410">
<title>Ведьмак: Пиксельный Путь</title>
<style>
  * { margin:0; padding:0; box-sizing:border-box; -webkit-tap-highlight-color:transparent; user-select:none; -webkit-user-select:none; }
  html, body {
    width:100%; height:100%; overflow:hidden;
    background:#0a0806;
    font-family:'Courier New', monospace;
    touch-action:none;
    position:fixed;
    color:#e6d5b8;
  }
  #game {
    display:block;
    background:#1a2418;
    image-rendering:pixelated;
    image-rendering:crisp-edges;
    position:absolute;
  }
  #hud {
    position:fixed; top:8px; left:8px; right:8px;
    display:flex; justify-content:space-between;
    align-items:flex-start;
    pointer-events:none;
    font-weight:bold;
    font-size:13px;
    text-shadow:1px 1px 0 #000, 2px 2px 0 #000;
    z-index:5;
  }
  .stats { display:flex; gap:6px; flex-wrap:wrap; }
  .stat {
    background:rgba(30,26,23,0.85);
    padding:5px 10px;
    border-radius:16px;
    border:1px solid #7e6b53;
    display:flex;
    align-items:center;
    gap:4px;
  }
  .stat .bar {
    width:50px; height:8px;
    background:#2a1a1a;
    border:1px solid #000;
    border-radius:4px;
    overflow:hidden;
  }
  .stat .bar > div { height:100%; transition:width 0.2s; }
  #hpBar > div { background:linear-gradient(#e55,#a22); }
  #stBar > div { background:linear-gradient(#fc6,#d90); }

  #armorSlots {
    display:flex;
    gap:4px;
    background:rgba(30,26,23,0.85);
    padding:5px 8px;
    border-radius:16px;
    border:1px solid #7e6b53;
    pointer-events:none;
  }
  .armorSlot {
    width:22px; height:22px;
    background:#1a1410;
    border:1px solid #5a4530;
    border-radius:4px;
    display:flex; align-items:center; justify-content:center;
    font-size:14px;
    opacity:0.35;
    transition:all 0.3s;
  }
  .armorSlot.on {
    opacity:1;
    background:linear-gradient(#8b6b4b,#5a4530);
    border-color:#d4b58a;
    box-shadow:0 0 6px rgba(255,200,100,0.5);
  }

  #topRight { display:flex; gap:8px; pointer-events:auto; }
  #topRight button {
    background:rgba(30,26,23,0.85);
    border:1px solid #7e6b53;
    color:#e6d5b8;
    font-size:18px;
    width:44px; height:44px;
    border-radius:12px;
    display:flex; align-items:center; justify-content:center;
    cursor:pointer; font-family:inherit;
    text-shadow:1px 1px 0 #000;
  }
  #topRight button:active { background:rgba(90,60,40,0.9); transform:scale(0.95); }
  #installBtn {
    display:none;
    background:linear-gradient(#8b6b4b,#5a4530) !important;
    border-color:#d4b58a !important;
    font-size:14px !important;
    width:auto !important;
    padding:0 14px !important;
    font-weight:bold;
  }

  #joy {
    position:fixed; left:20px; bottom:20px;
    width:140px; height:140px;
    border-radius:50%;
    background:radial-gradient(circle at 40% 40%, rgba(90,75,58,0.5), rgba(40,32,26,0.5));
    border:2px solid rgba(180,155,120,0.7);
    touch-action:none; z-index:10;
    box-shadow:0 0 20px rgba(0,0,0,0.5), inset 0 0 15px rgba(0,0,0,0.4);
  }
  #joyKnob {
    position:absolute; left:50%; top:50%;
    width:60px; height:60px;
    margin-left:-30px; margin-top:-30px;
    border-radius:50%;
    background:radial-gradient(circle at 35% 35%, #e6d5b8, #8b6b4b);
    border:2px solid #f0e2c0;
    pointer-events:none;
    box-shadow:0 3px 8px rgba(0,0,0,0.6);
    transition:transform 0.05s linear;
  }

  #btns {
    position:fixed; right:20px; bottom:30px;
    display:flex; flex-direction:column; gap:14px;
    z-index:10;
    align-items:flex-end;
  }
  .btn {
    width:86px; height:86px;
    border-radius:50%;
    touch-action:none;
    position:relative;
    overflow:hidden;
    border:3px solid #d4b58a;
    box-shadow:0 5px 0 rgba(0,0,0,0.5), 0 0 15px rgba(0,0,0,0.5), inset 0 2px 6px rgba(255,255,255,0.2);
    background:#6b4a2a;
  }
  .btn canvas {
    position:absolute; inset:0;
    width:100%; height:100%;
    image-rendering:pixelated;
    image-rendering:crisp-edges;
    pointer-events:none;
  }
  .btn:active {
    transform:translateY(4px);
    box-shadow:0 1px 0 rgba(0,0,0,0.5), 0 0 10px rgba(0,0,0,0.5);
  }
  #btnAttack { background:radial-gradient(circle at 35% 30%, #b88a5a, #6b4a2a); }
  #btnSign {
    background:radial-gradient(circle at 35% 30%, #6a4a8a, #2a1a4a);
    border-color:#b88aff;
    width:76px; height:76px;
  }
  .btn .cd {
    position:absolute; inset:0; border-radius:50%;
    background:conic-gradient(rgba(0,0,0,0.75) var(--cd, 0deg), transparent 0deg);
    pointer-events:none;
  }
  /* Кнопка выбора знака */
  #signSwitch {
    position:fixed;
    right:120px;
    bottom:30px;
    width:56px; height:56px;
    border-radius:50%;
    background:radial-gradient(circle at 35% 30%, #8a6aaa, #3a1a5a);
    border:2px solid #d4b58a;
    color:#fff8e0;
    font-size:22px;
    display:flex; align-items:center; justify-content:center;
    cursor:pointer;
    z-index:10;
    box-shadow:0 3px 0 rgba(0,0,0,0.5), 0 0 12px rgba(180,120,255,0.4);
    touch-action:none;
    font-family:inherit;
  }
  #signSwitch:active { transform:translateY(3px); box-shadow:0 1px 0 rgba(0,0,0,0.5); }

  /* Экран выбора персонажа */
  #charSelect {
    position:fixed; inset:0;
    background:linear-gradient(160deg, #1a1410 0%, #2a1e14 100%);
    z-index:200;
    display:flex;
    flex-direction:column;
    align-items:center;
    justify-content:center;
    padding:20px;
    gap:16px;
    overflow-y:auto;
  }
  #charSelect.hidden { display:none; }
  #charSelect h1 {
    color:#e6d5b8;
    font-size:22px;
    letter-spacing:2px;
    text-shadow:2px 2px 0 #000;
    margin-bottom:4px;
    text-align:center;
  }
  #charSelect p {
    color:#a89878;
    font-size:13px;
    text-align:center;
    margin-bottom:10px;
  }
  .charGrid {
    display:grid;
    grid-template-columns:repeat(2, 1fr);
    gap:14px;
    max-width:460px;
    width:100%;
  }
  .charCard {
    background:linear-gradient(160deg, #3a2e22, #1a1410);
    border:2px solid #5a4530;
    border-radius:14px;
    padding:12px;
    cursor:pointer;
    display:flex;
    flex-direction:column;
    align-items:center;
    gap:6px;
    transition:all 0.2s;
    position:relative;
    overflow:hidden;
  }
  .charCard:active, .charCard.selected {
    border-color:#d4b58a;
    background:linear-gradient(160deg, #5a4530, #2a1e14);
    box-shadow:0 0 20px rgba(255,200,100,0.3);
    transform:scale(0.97);
  }
  .charCard canvas {
    image-rendering:pixelated;
    image-rendering:crisp-edges;
    width:80px; height:80px;
  }
  .charName {
    color:#e6d5b8;
    font-size:16px;
    font-weight:bold;
    letter-spacing:1px;
  }
  .charDesc {
    color:#a89878;
    font-size:11px;
    text-align:center;
    line-height:1.4;
  }
  .charSign {
    color:#ffcc88;
    font-size:12px;
    font-weight:bold;
  }
  #startBtn {
    margin-top:14px;
    background:linear-gradient(#8b6b4b,#5a4530);
    border:2px solid #d4b58a;
    color:#fff8e0;
    font-family:inherit;
    font-size:18px;
    font-weight:bold;
    padding:12px 50px;
    border-radius:12px;
    cursor:pointer;
    text-shadow:1px 1px 0 #000;
    box-shadow:0 4px 0 #2a1e14;
    letter-spacing:2px;
  }
  #startBtn:active { transform:translateY(3px); box-shadow:0 1px 0 #2a1e14; }
  #startBtn:disabled {
    opacity:0.4;
    cursor:not-allowed;
    transform:none;
  }

  #deathScreen {
    position:fixed; inset:0;
    background:rgba(20,5,5,0.85);
    display:none; align-items:center; justify-content:center;
    flex-direction:column; gap:20px;
    z-index:100; color:#e6d5b8; text-align:center; padding:20px;
  }
  #deathScreen.show { display:flex; }
  #deathScreen h1 {
    font-size:48px; color:#b33e3e;
    text-shadow:3px 3px 0 #000; letter-spacing:4px;
  }
  #deathScreen p { font-size:18px; }
  #deathScreen button {
    background:linear-gradient(#8b6b4b,#5a4530);
    border:2px solid #d4b58a; color:#fff8e0;
    font-family:inherit; font-size:20px; font-weight:bold;
    padding:12px 40px; border-radius:12px; cursor:pointer;
    text-shadow:1px 1px 0 #000;
    box-shadow:0 4px 0 #2a1e14;
  }
  #deathScreen button:active { transform:translateY(3px); box-shadow:0 1px 0 #2a1e14; }
  #deathScreen .small {
    font-size:14px;
    color:#a89878;
  }
</style>
</head>
<body>

<!-- ЭКРАН ВЫБОРА ПЕРСОНАЖА -->
<div id="charSelect">
  <h1>⚔ ВЫБЕРИ ВЕДЬМАКА ⚔</h1>
  <p>Каждый со своим знаком и стилем боя</p>
  <div class="charGrid" id="charGrid"></div>
  <button id="startBtn" disabled>НАЧАТЬ ПУТЬ</button>
</div>

<canvas id="game" style="display:none;"></canvas>

<div id="hud" style="display:none;">
  <div class="stats">
    <div class="stat">❤️ <div class="bar" id="hpBar"><div></div></div></div>
    <div class="stat">🔥 <div class="bar" id="stBar"><div></div></div></div>
    <div class="stat">⚔️ <span id="kills">0</span></div>
    <div class="stat">🌙 <span id="time">День</span></div>
    <div id="armorSlots">
      <div class="armorSlot" id="slotHelm">⛑</div>
      <div class="armorSlot" id="slotChest">🥼</div>
      <div class="armorSlot" id="slotLegs">🦵</div>
    </div>
    <div class="stat" id="signLabel">🔥 Игни</div>
  </div>
  <div id="topRight">
    <button id="installBtn">УСТАНОВИТЬ</button>
    <button id="fsBtn" title="Полный экран">⛶</button>
  </div>
</div>

<div id="joy" style="display:none;"><div id="joyKnob"></div></div>

<button id="signSwitch" style="display:none;" title="Сменить знак">✦</button>

<div id="btns" style="display:none;">
  <div class="btn" id="btnSign"><canvas width="80" height="80"></canvas><div class="cd" id="cdSign"></div></div>
  <div class="btn" id="btnAttack"><canvas width="80" height="80"></canvas><div class="cd" id="cdAttack"></div></div>
</div>

<div id="deathScreen">
  <h1>ВЫ ПОГИБЛИ</h1>
  <p>Убито монстров: <span id="deathKills">0</span></p>
  <p class="small">Ведьмак: <span id="deathChar">—</span></p>
  <button id="respawnBtn">ВОССТАТЬ</button>
</div>

<script>
(function(){
  'use strict';
  console.log('🐺 Ведьмак: старт');

  // ==================== ОПИСАНИЕ ПЕРСОНАЖЕЙ ====================
  var CHARACTERS = [
    {
      id: 'geralt',
      name: 'ГЕРАЛЬТ',
      desc: 'Белый Волк. Баланс силы и магии.',
      sign: 'igni',
      signName: 'Игни 🔥',
      color: '#d4b28c', hair: '#e8e8e8',
      speed: 2.6,
      maxHp: 5,
      dmg: 1,
      signDmg: 2,
      special: null
    },
    {
      id: 'ciri',
      name: 'ЦИРИ',
      desc: 'Львёнок из Цинтры. Быстрая, как ветер.',
      sign: 'blink',
      signName: 'Рывок 💨',
      color: '#e8d8b8', hair: '#c8c8c8',
      speed: 3.4,
      maxHp: 4,
      dmg: 1,
      signDmg: 0,
      special: 'blink'
    },
    {
      id: 'eskel',
      name: 'ЭСКЕЛЬ',
      desc: 'Ведьмак из Каэр Морхена. Танк.',
      sign: 'quen',
      signName: 'Квен 🛡',
      color: '#c8a882', hair: '#8a7a5a',
      speed: 2.2,
      maxHp: 8,
      dmg: 1,
      signDmg: 0,
      special: 'shield'
    },
    {
      id: 'lambert',
      name: 'ЛАМБЕРТ',
      desc: 'Маг-ведьмак. Знаки сильнее.',
      sign: 'yrden',
      signName: 'Ирден ⚡',
      color: '#d0a878', hair: '#a06848',
      speed: 2.4,
      maxHp: 5,
      dmg: 1,
      signDmg: 3,
      special: 'trap'
    }
  ];

  var selectedChar = null;

  // Рисование иконок кнопок
  function drawSwordIcon(cvs){
    var c = cvs.getContext('2d');
    c.imageSmoothingEnabled = false;
    c.clearRect(0,0,80,80);
    var P = 5;
    function px(gx, gy, color){ c.fillStyle = color; c.fillRect(gx*P, gy*P, P, P); }
    var blade='#e8e8e8', bladeD='#a0a0a0', bladeH='#ffffff';
    px(11,2, bladeH); px(11,3, blade);
    px(10,3, blade); px(10,4, blade);
    px(9,4, blade); px(9,5, blade);
    px(8,5, blade); px(8,6, blade);
    px(7,6, blade); px(7,7, blade);
    px(6,7, blade); px(6,8, blade);
    px(5,8, blade); px(5,9, blade);
    px(12,3, bladeD); px(11,4, bladeD);
    px(10,5, bladeD); px(9,6, bladeD);
    px(8,7, bladeD); px(7,8, bladeD);
    px(6,9, bladeD);
    var guard='#d4b58a', guardD='#8b6b4b';
    px(3,10, guard); px(4,10, guard); px(5,10, guard); px(6,10, guard);
    px(7,10, guard); px(8,10, guard); px(9,10, guard);
    px(3,11, guardD); px(5,11, guardD); px(7,11, guardD); px(9,11, guardD);
    var grip='#5a3a1a', gripH='#7a5028';
    px(4,12, grip); px(5,12, gripH); px(6,12, grip); px(7,12, gripH);
    px(3,13, grip); px(4,13, gripH); px(5,13, grip); px(6,13, gripH);
    var pommel='#e6c422', pommelD='#a8880a';
    px(2,14, pommel); px(3,14, pommelD); px(2,15, pommelD);
    px(9,3, bladeH); px(8,4, bladeH); px(7,5, bladeH);
  }

  function drawSignIcon(cvs, type){
    var c = cvs.getContext('2d');
    c.imageSmoothingEnabled = false;
    c.clearRect(0,0,80,80);
    var P = 5;
    function px(gx, gy, color){ c.fillStyle = color; c.fillRect(gx*P, gy*P, P, P); }

    if (type === 'igni' || type === 'fire'){
      var f1='#cc4411', f2='#ff8822', f3='#ffcc44', f4='#fff0a0';
      px(7,2,f1); px(8,2,f1);
      px(6,3,f1); px(9,3,f1);
      px(5,4,f1); px(10,4,f1);
      px(4,5,f1); px(6,5,f1); px(9,5,f1); px(11,5,f1);
      px(4,6,f1); px(11,6,f1);
      px(3,7,f1); px(12,7,f1);
      px(3,8,f1); px(12,8,f1);
      px(2,9,f1); px(13,9,f1);
      px(2,10,f1); px(13,10,f1);
      px(2,11,f1); px(13,11,f1);
      px(3,12,f1); px(12,12,f1);
      px(4,13,f1); px(11,13,f1);
      px(5,14,f1); px(6,14,f1); px(7,14,f1);
      px(8,14,f1); px(9,14,f1); px(10,14,f1);
      px(7,4,f2); px(8,4,f2);
      px(6,5,f2); px(9,5,f2);
      px(5,6,f2); px(10,6,f2);
      px(5,7,f2); px(10,7,f2);
      px(4,8,f2); px(11,8,f2);
      px(4,9,f2); px(11,9,f2);
      px(3,10,f2); px(12,10,f2);
      px(4,11,f2); px(11,11,f2);
      px(5,12,f2); px(10,12,f2);
      px(6,13,f2); px(7,13,f2); px(8,13,f2); px(9,13,f2);
      px(7,7,f3); px(8,7,f3);
      px(6,8,f3); px(9,8,f3);
      px(6,9,f3); px(9,9,f3);
      px(5,10,f3); px(10,10,f3);
      px(6,11,f3); px(9,11,f3);
      px(7,12,f3); px(8,12,f3);
      px(7,9,f4); px(8,9,f4);
      px(7,10,f4); px(8,10,f4);
      px(7,11,f4); px(8,11,f4);
    } else if (type === 'quen' || type === 'shield'){
      // Щит
      var sh='#88aaff', shD='#4466cc', shH='#cceeff';
      for (var y=2;y<14;y++){
        for (var x=0;x<16;x++){
          var dx = x-7.5;
          if (y<3 && Math.abs(dx)<5) px(x,y,shH);
          else if (y<10 && Math.abs(dx)<7-y*0.2) px(x,y,(Math.abs(dx)>4?shD:sh));
          else if (y<14 && Math.abs(dx)<5-y*0.3) px(x,y,shD);
        }
      }
      // Символ внутри
      px(7,5,shH); px(8,5,shH); px(7,6,shH); px(8,6,shH);
      px(6,7,shH); px(9,7,shH);
      px(7,8,shH); px(8,8,shH);
      px(7,9,shH); px(8,9,shH);
    } else if (type === 'aard' || type === 'wind'){
      // Ветер - полосы
      var w1='#aaddff', w2='#ffffff', w3='#6688cc';
      for (var i=0;i<5;i++){
        var yy = 3 + i*2.5;
        for (var j=0;j<10;j++){
          if (Math.random()<0.7) px(2+j, Math.floor(yy), (j%2===0?w1:w2));
        }
      }
    } else if (type === 'yrden' || type === 'trap'){
      // Ловушка - руна
      var r1='#aa66ff', r2='#ff88ff', r3='#6622aa';
      // Круг
      for (var a=0;a<20;a++){
        var ang = a/20*Math.PI*2;
        var cx = Math.floor(8 + Math.cos(ang)*6);
        var cy = Math.floor(8 + Math.sin(ang)*6);
        px(cx,cy,r1);
      }
      // Внутренний символ
      px(7,5,r2); px(8,5,r2);
      px(7,6,r3); px(8,6,r3);
      px(6,7,r3); px(9,7,r3);
      px(7,8,r2); px(8,8,r2);
      px(6,9,r3); px(9,9,r3);
      px(7,10,r2); px(8,10,r2);
      px(7,11,r2); px(8,11,r2);
    } else if (type === 'axii' || type === 'mind'){
      // Аксий - спираль
      var a1='#ffdd66', a2='#ffaa22', a3='#ffffff';
      var spiral = [[8,2],[10,3],[11,5],[11,7],[10,9],[8,11],[6,11],[4,10],[3,8],[3,6],[4,5],[6,4]];
      for (var s=0;s<spiral.length;s++){
        px(spiral[s][0], spiral[s][1], (s%2===0?a1:a2));
      }
      px(8,7,a3); px(7,8,a3); px(8,8,a3); px(9,8,a3);
      px(8,9,a3);
    } else if (type === 'blink'){
      // Рывок - молния / стрелки
      var b1='#88ddff', b2='#ffffff', b3='#4488cc';
      // Молния
      px(9,2,b2); px(8,3,b2); px(7,4,b1);
      px(7,5,b1); px(6,6,b1); px(5,7,b1);
      px(5,8,b1); px(6,9,b1); px(7,10,b1);
      px(8,11,b2); px(9,12,b2); px(10,13,b2);
      // Дополнительные стрелки
      px(3,4,b3); px(2,5,b3); px(2,6,b3);
      px(12,9,b3); px(13,10,b3); px(13,11,b3);
    }
  }

  // Нарисовать иконку знака
  function updateSignButton(sign){
    var cvs = document.querySelector('#btnSign canvas');
    drawSignIcon(cvs, sign);
  }

  // ==================== РИСОВАНИЕ КАРТОЧЕК ПЕРСОНАЖЕЙ ====================
  function drawCharPreview(cvs, char){
    var c = cvs.getContext('2d');
    c.imageSmoothingEnabled = false;
    c.clearRect(0,0,80,80);
    // Простой портрет: голова + волосы + глаза
    var P = 5;
    function px(gx, gy, color){ c.fillStyle = color; c.fillRect(gx*P, gy*P, P, P); }
    // Голова
    for (var y=2;y<14;y++)
      for (var x=2;x<14;x++)
        px(x,y,'#d4b28c');
    // Волосы
    var hc = char.hair || '#e8e8e8';
    var hd = '#888';
    for (var y=0;y<6;y++)
      for (var x=2;x<14;x++)
        px(x,y, (y<3?hc:hd));
    px(1,2,hc); px(1,3,hc); px(1,4,hc); px(1,5,hc);
    px(14,2,hc); px(14,3,hc); px(14,4,hc); px(14,5,hc);
    // Глаза (жёлтые)
    px(5,7,'#e6c422'); px(6,7,'#e6c422'); px(5,8,'#e6c422');
    px(9,7,'#e6c422'); px(10,7,'#e6c422'); px(9,8,'#e6c422');
    px(5,7,'#000'); px(9,7,'#000');
    // Шрам для Геральта
    if (char.id==='geralt'){
      px(7,6,'#a88a68'); px(7,7,'#a88a68'); px(7,8,'#a88a68'); px(7,9,'#a88a68');
    }
    // Борода для Эскеля
    if (char.id==='eskel'){
      for (var x=3;x<13;x++) px(x,13,'#8a7a5a');
      px(4,12,'#8a7a5a'); px(11,12,'#8a7a5a');
    }
    // Метка Цири (шрам на щеке)
    if (char.id==='ciri'){
      px(3,9,'#aa4444'); px(3,10,'#aa4444');
    }
    // Тёмные волосы Ламберта
    if (char.id==='lambert'){
      for (var y=0;y<7;y++)
        for (var x=2;x<14;x++)
          px(x,y, char.hair);
    }
  }

  // Построение карточек выбора
  function buildCharSelect(){
    var grid = document.getElementById('charGrid');
    grid.innerHTML = '';
    CHARACTERS.forEach(function(ch){
      var card = document.createElement('div');
      card.className = 'charCard';
      card.dataset.id = ch.id;

      var cvs = document.createElement('canvas');
      cvs.width = 80; cvs.height = 80;
      card.appendChild(cvs);

      var name = document.createElement('div');
      name.className = 'charName';
      name.textContent = ch.name;
      card.appendChild(name);

      var desc = document.createElement('div');
      desc.className = 'charDesc';
      desc.textContent = ch.desc;
      card.appendChild(desc);

      var sign = document.createElement('div');
      sign.className = 'charSign';
      sign.textContent = '✦ ' + ch.signName;
      card.appendChild(sign);

      card.addEventListener('click', function(){
        document.querySelectorAll('.charCard').forEach(function(c){ c.classList.remove('selected'); });
        card.classList.add('selected');
        selectedChar = ch;
        document.getElementById('startBtn').disabled = false;
      });

      grid.appendChild(card);
      drawCharPreview(cvs, ch);
    });
  }

  buildCharSelect();

  // ==================== ИГРА ====================
  var canvas = document.getElementById('game');
  var ctx = canvas.getContext('2d');
  ctx.imageSmoothingEnabled = false;

  var TILE = 32;
  var MAP_W = 30, MAP_H = 30;
  var VIEW_W = 16 * TILE;
  var VIEW_H = 16 * TILE;

  function resize(){
    var w = window.innerWidth, h = window.innerHeight;
    var scale = Math.min(w / VIEW_W, h / VIEW_H);
    var cw = Math.floor(VIEW_W * scale);
    var ch = Math.floor(VIEW_H * scale);
    canvas.width = VIEW_W;
    canvas.height = VIEW_H;
    canvas.style.width = cw + 'px';
    canvas.style.height = ch + 'px';
    canvas.style.left = ((w - cw) / 2) + 'px';
    canvas.style.top = ((h - ch) / 2) + 'px';
  }
  window.addEventListener('resize', resize);
  window.addEventListener('orientationchange', function(){ setTimeout(resize, 250); });

  var C = {
    g1:'#31402c', g2:'#28351f', g3:'#425a30', g4:'#4e6b3a', g5:'#5a7a48',
    tree:'#2d4a2a', treeD:'#16281a', treeH:'#5c7a4a', treeHD:'#7ea862', trunk:'#3a2a1a', trunkL:'#5a3a1a',
    rock:'#6a6a6a', rockD:'#3a3a3a', rockH:'#9a9a9a', rockS:'#2a2a2a',
    skin:'#d4b28c', skinS:'#a88a68', hair:'#e8e8e8', hairD:'#b8b8b8',
    armor:'#8b6b4b', armorD:'#5a4530', armorH:'#a88a68', cloak:'#4a3a2a', cloakD:'#2a1e14',
    helm:'#a8a8b8', helmD:'#6a6a7a', helmH:'#d8d8e8',
    legArmor:'#6a5a4a', legArmorD:'#3a2e22', legArmorH:'#8a7a6a',
    sword:'#e8e8e8', swordB:'#8a8a8a', swordH:'#fff', eye:'#e6c422', eyeGlow:'#ffe455',
    beard:'#c8c8c8',
    en1:'#7a3a2a', en1d:'#4a1e1a', en1h:'#a85040',
    en2:'#3a5a3a', en2d:'#1e331b', en2h:'#5a8a5a',
    en3:'#5a3a6a', en3d:'#2a1a3a', en3h:'#8a5aaa',
    fire:'#ff884d', fireG:'#ffcc66', fireW:'#fff0a0',
    blood:'#a82222', bloodD:'#6a1010',
    shadow:'rgba(0,0,0,0.4)',
    water:'#2a4a5a', waterL:'#4a7a8a', waterH:'#7aaaba'
  };

  var armor = { helm:false, chest:false, legs:false };

  var player = {
    x:15*TILE, y:15*TILE,
    hp:5, maxHp:5, st:100, maxSt:100,
    cd:0, atkT:0, face:'down', inv:0, walkT:0, moving:false,
    level:1, xp:0, xpNext:5,
    swordTrail: [],
    // Знак
    currentSign: 'igni',
    signCd: 0,
    shieldHp: 0,
    shieldTimer: 0,
    // Данные персонажа
    charData: null,
    speed: 2.6,
    dmg: 1,
    signDmg: 2
  };

  var enemies = [];
  var particles = [];
  var floatTexts = [];
  var kills = 0;
  var map = [];
  var decor = [];
  var igniFx = null;
  var yrdenFx = []; // ловушки Ирден
  var quenFx = null; // аура Квен
  var screenShake = 0;
  var gameTime = 0;
  var isDead = false;
  var spawnTimer = 2;
  var bloodDecals = [];

  // Список всех доступных знаков
  var ALL_SIGNS = ['igni', 'quen', 'aard', 'yrden', 'axii'];
  var SIGN_NAMES = {
    igni: '🔥 Игни',
    quen: '🛡 Квен',
    aard: '💨 Аард',
    yrden: '⚡ Ирден',
    axii: '✨ Аксий',
    blink: '💫 Рывок'
  };

  // ==================== ЗВУК ====================
  var audioCtx = null;
  function initAudio(){
    if (audioCtx) return;
    try { audioCtx = new (window.AudioContext || window.webkitAudioContext)(); } catch(e){}
  }
  function playSound(freq, dur, type, vol){
    if (!audioCtx) return;
    try {
      var o = audioCtx.createOscillator();
      var g = audioCtx.createGain();
      o.type = type || 'square';
      o.frequency.value = freq;
      g.gain.value = vol || 0.05;
      g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + dur);
      o.connect(g); g.connect(audioCtx.destination);
      o.start(); o.stop(audioCtx.currentTime + dur);
    } catch(e){}
  }
  function sndSwing(){ playSound(320, 0.08, 'square', 0.04); }
  function sndHit(){ playSound(180, 0.1, 'sawtooth', 0.06); }
  function sndKill(){ playSound(120, 0.2, 'triangle', 0.07); }
  function sndHurt(){ playSound(90, 0.15, 'sawtooth', 0.08); }
  function sndSign(){ playSound(500, 0.15, 'sine', 0.06); setTimeout(function(){ playSound(300, 0.2, 'sine', 0.05); }, 60); }
  function sndLevel(){ playSound(660, 0.1, 'sine', 0.06); setTimeout(function(){ playSound(880, 0.15, 'sine', 0.06); }, 100); }
  function sndArmor(){ playSound(520, 0.1, 'sine', 0.07); setTimeout(function(){ playSound(780, 0.15, 'sine', 0.07); }, 80); }
  function sndShield(){ playSound(600, 0.2, 'sine', 0.08); setTimeout(function(){ playSound(900, 0.15, 'sine', 0.06); }, 100); }

  // ==================== КАРТА ====================
  function genMap(){
    map = []; decor = []; bloodDecals = []; yrdenFx = [];
    for (var y=0; y<MAP_H; y++){
      map[y] = [];
      for (var x=0; x<MAP_W; x++){
        if (x===0||y===0||x===MAP_W-1||y===MAP_H-1) map[y][x]=1;
        else {
          var r = Math.random();
          if (r<0.11) map[y][x]=1;
          else if (r<0.17) map[y][x]=2;
          else if (r<0.20) map[y][x]=3;
          else map[y][x]=0;
        }
      }
    }
    for (var yy=13; yy<18; yy++)
      for (var xx=13; xx<18; xx++) map[yy][xx]=0;
    for (var y2=0; y2<MAP_H; y2++)
      for (var x2=0; x2<MAP_W; x2++)
        if (map[y2][x2]===0 && Math.random()<0.28)
          decor.push({ x:x2*TILE, y:y2*TILE,
            t:Math.random()<0.6?'grass':(Math.random()<0.7?'flower':'mushroom'),
            s:Math.random(), phase:Math.random()*Math.PI*2 });
  }

  function isSolid(tx,ty){
    if (tx<0||tx>=MAP_W||ty<0||ty>=MAP_H) return true;
    var t = map[ty][tx]; return t===1||t===2;
  }
  function canMove(px,py){
    var l=Math.floor(px/TILE), r=Math.floor((px+TILE-1)/TILE);
    var t=Math.floor(py/TILE), b=Math.floor((py+TILE-1)/TILE);
    for (var ty=t; ty<=b; ty++)
      for (var tx=l; tx<=r; tx++)
        if (isSolid(tx,ty)) return false;
    return true;
  }
  function tileAt(px,py){
    var tx = Math.floor(px/TILE), ty = Math.floor(py/TILE);
    if (tx<0||tx>=MAP_W||ty<0||ty>=MAP_H) return 0;
    return map[ty][tx];
  }

  // ==================== ЧАСТИЦЫ ====================
  function spawnParticles(x, y, color, count, opts){
    opts = opts || {};
    for (var i=0;i<count;i++){
      var a = Math.random()*Math.PI*2;
      var s = (opts.speed || 1) * (0.5 + Math.random()*2);
      particles.push({
        x:x, y:y, vx:Math.cos(a)*s, vy:Math.sin(a)*s - (opts.up||1),
        life:(opts.life||22) + Math.random()*(opts.lifeVar||15),
        maxLife:40, color:color,
        size:(opts.size||2) + Math.random()*(opts.sizeVar||2),
        gravity: opts.gravity !== undefined ? opts.gravity : 0.15,
        glow: opts.glow || false
      });
    }
  }
  function spawnFloatText(x, y, text, color){
    floatTexts.push({ x:x, y:y, text:text, color:color, life:60, maxLife:60 });
  }

  // ==================== СПАВН ====================
  function spawnEnemy(){
    for (var i=0; i<100; i++){
      var x = Math.floor(Math.random()*(MAP_W-2))+1;
      var y = Math.floor(Math.random()*(MAP_H-2))+1;
      if (map[y][x]===0 && !(Math.abs(x-15)<4 && Math.abs(y-15)<4)){
        var r = Math.random();
        var type = r<0.35?'nekker':(r<0.7?'drowner':'wraith');
        var hp = type==='wraith'?3:2;
        enemies.push({
          x:x*TILE, y:y*TILE, hp:hp, maxHp:hp, atkT:0,
          speed: type==='wraith'?0.75:(0.5+Math.random()*0.4),
          type: type, hitT:0, walkT: Math.random()*10, phase: Math.random()*Math.PI*2,
          mindControlled: 0,
          slowed: 0
        });
        return;
      }
    }
  }

  // ==================== ОБНОВЛЕНИЕ ====================
  function update(dt){
    if (isDead) return;
    gameTime += dt * 0.008;
    if (gameTime > 1) gameTime -= 1;

    // Скорость
    var speed = player.speed;
    if (tileAt(player.x+TILE/2, player.y+TILE/2)===3) speed *= 0.6;
    var dx = joy.dx, dy = joy.dy;
    var mag = Math.sqrt(dx*dx+dy*dy);
    player.moving = mag > 0.15;
    if (player.moving){
      if (mag>1){ dx/=mag; dy/=mag; }
      var nx = dx*speed, ny = dy*speed;
      if (Math.abs(dx)>Math.abs(dy)) player.face = dx>0?'right':'left';
      else player.face = dy>0?'down':'up';
      if (canMove(player.x+nx, player.y)) player.x+=nx;
      if (canMove(player.x, player.y+ny)) player.y+=ny;
      player.walkT += dt*11;
    } else player.walkT = 0;

    if (player.st<player.maxSt) player.st = Math.min(player.maxSt, player.st+0.7);
    if (player.cd>0) player.cd -= dt*60;
    if (player.atkT>0) player.atkT -= dt*60;
    if (player.inv>0) player.inv -= dt*60;
    if (player.signCd>0) player.signCd -= dt*60;

    // Щит Квен
    if (player.shieldTimer > 0){
      player.shieldTimer -= dt*60;
      if (player.shieldTimer <= 0){
        player.shieldHp = 0;
      }
    }

    // Шлейф меча
    if (player.atkT > 0){
      player.swordTrail.push({ face: player.face, x: player.x, y: player.y, life: 8 });
      if (player.swordTrail.length > 6) player.swordTrail.shift();
    }
    for (var st=player.swordTrail.length-1; st>=0; st--){
      player.swordTrail[st].life--;
      if (player.swordTrail[st].life<=0) player.swordTrail.splice(st,1);
    }

    // Ловушки Ирден (тик)
    for (var yi=yrdenFx.length-1; yi>=0; yi--){
      var yf = yrdenFx[yi];
      yf.timer--;
      // Урон врагам внутри
      for (var ei=enemies.length-1; ei>=0; ei--){
        var en = enemies[ei];
        var ddx = en.x+TILE/2 - yf.x;
        var ddy = en.y+TILE/2 - yf.y;
        if (Math.sqrt(ddx*ddx+ddy*ddy) < 60){
          if (yf.tick <= 0){
            en.hp -= 1;
            en.hitT = 8;
            en.slowed = 60;
            spawnFloatText(en.x+TILE/2, en.y, '-1', '#cc88ff');
            if (en.hp<=0){
              spawnParticles(en.x+TILE/2, en.y+TILE/2, C.blood, 15, {speed:3});
              enemies.splice(ei,1); kills++; gainXp(1); sndKill();
            }
          }
        }
      }
      if (yf.tick > 0) yf.tick--;
      else yf.tick = 30;
      if (yf.timer <= 0) yrdenFx.splice(yi,1);
    }

    // Враги
    for (var i=enemies.length-1; i>=0; i--){
      var e = enemies[i];
      if (e.atkT>0) e.atkT -= dt*60;
      if (e.hitT>0) e.hitT -= dt*60;
      if (e.slowed > 0) e.slowed -= dt*60;
      if (e.mindControlled > 0){
        e.mindControlled -= dt*60;
        // Атакует других врагов
        var target = null;
        var bestD = 999;
        for (var j=0;j<enemies.length;j++){
          if (j===i) continue;
          var other = enemies[j];
          var od = Math.hypot(other.x-e.x, other.y-e.y);
          if (od < bestD){ bestD = od; target = other; }
        }
        if (target && bestD < 200){
          var mdx = target.x - e.x, mdy = target.y - e.y;
          var md = Math.sqrt(mdx*mdx+mdy*mdy);
          var mmx = (mdx/md)*e.speed*0.9, mmy = (mdy/md)*e.speed*0.9;
          if (canMove(e.x+mmx, e.y)) e.x+=mmx;
          if (canMove(e.x, e.y+mmy)) e.y+=mmy;
          if (md < TILE*0.9 && e.atkT<=0){
            target.hp -= 1;
            target.hitT = 10;
            spawnParticles(target.x+TILE/2, target.y+TILE/2, C.blood, 8);
            if (target.hp<=0){
              spawnParticles(target.x+TILE/2, target.y+TILE/2, C.blood, 15, {speed:3});
              enemies.splice(j,1);
              if (j<i) i--;
              kills++;
              gainXp(1);
            }
            e.atkT = 40;
          }
        }
        e.walkT += dt*8;
        continue;
      }

      var edx = player.x - e.x, edy = player.y - e.y;
      var d = Math.sqrt(edx*edx+edy*edy);
      if (d>4 && d<300){
        var ms = e.speed;
        if (tileAt(e.x+TILE/2, e.y+TILE/2)===3) ms *= 0.6;
        if (e.slowed > 0) ms *= 0.5;
        var mx = (edx/d)*ms, my = (edy/d)*ms;
        if (canMove(e.x+mx, e.y)) e.x+=mx;
        if (canMove(e.x, e.y+my)) e.y+=my;
        e.walkT += dt*8;
      }
      if (d<TILE*0.95 && e.atkT<=0){
        if (player.inv<=0){
          // Проверка щита Квен
          if (player.shieldHp > 0){
            player.shieldHp--;
            spawnParticles(player.x+TILE/2, player.y+TILE/2, '#88aaff', 12, {speed:2, glow:true});
            spawnFloatText(player.x+TILE/2, player.y-5, 'БЛОК', '#88ccff');
            if (player.shieldHp<=0){
              player.shieldTimer = 0;
              spawnFloatText(player.x+TILE/2, player.y-25, 'ЩИТ РАЗБИТ', '#ff6666');
            }
          } else {
            player.hp--;
            player.inv = 50;
            screenShake = 10;
            sndHurt();
            spawnParticles(player.x+TILE/2, player.y+TILE/2, C.blood, 10);
            spawnFloatText(player.x+TILE/2, player.y-5, '-1', '#ff4444');
            bloodDecals.push({ x:player.x+8, y:player.y+20, r:6+Math.random()*4, a:0.6, t:0 });
            if (player.hp<=0) die();
          }
        }
        e.atkT = 45;
      }
    }

    // Частицы
    for (var p=particles.length-1; p>=0; p--){
      var pt = particles[p];
      pt.x += pt.vx; pt.y += pt.vy; pt.vy += pt.gravity; pt.vx *= 0.98; pt.life--;
      if (pt.life<=0) particles.splice(p,1);
    }
    for (var ft=floatTexts.length-1; ft>=0; ft--){
      var f = floatTexts[ft]; f.y -= 0.8; f.life--;
      if (f.life<=0) floatTexts.splice(ft,1);
    }
    for (var bd=bloodDecals.length-1; bd>=0; bd--){
      bloodDecals[bd].t++;
      if (bloodDecals[bd].t > 400) bloodDecals.splice(bd,1);
    }

    spawnTimer -= dt;
    if (spawnTimer<=0 && enemies.length<9){ spawnEnemy(); spawnTimer = 1.8 + Math.random()*1.8; }
    if (screenShake>0) screenShake -= dt*60;

    // HUD
    hpEl.style.width = (player.hp / player.maxHp) * 100 + '%';
    stEl.style.width = (player.st / player.maxSt) * 100 + '%';
    killsEl.textContent = kills;
    var hour = Math.floor(gameTime * 24);
    timeEl.textContent = hour>=6 && hour<18 ? '☀️' : '🌙';

    cdAttackEl.style.setProperty('--cd', Math.max(0, player.cd) / 18 * 360 + 'deg');
    cdSignEl.style.setProperty('--cd', Math.max(0, player.signCd) / 45 * 360 + 'deg');
  }

  // ==================== АТАКА ====================
  function attack(){
    if (isDead || player.cd>0 || player.st<15) return;
    initAudio();
    player.st -= 15;
    player.cd = player.charData.id === 'ciri' ? 12 : 18;
    player.atkT = 9;
    sndSwing();
    var ax = player.x, ay = player.y;
    if (player.face==='right') ax+=TILE; else if (player.face==='left') ax-=TILE;
    else if (player.face==='down') ay+=TILE; else ay-=TILE;
    var hitAny = false;
    for (var i=enemies.length-1; i>=0; i--){
      var e = enemies[i];
      var ddx = (e.x+TILE/2)-(ax+TILE/2);
      var ddy = (e.y+TILE/2)-(ay+TILE/2);
      if (Math.sqrt(ddx*ddx+ddy*ddy) < TILE*1.25){
        e.hp -= player.dmg;
        e.hitT = 10;
        hitAny = true;
        var a = Math.atan2(e.y-player.y, e.x-player.x);
        e.x += Math.cos(a)*16; e.y += Math.sin(a)*16;
        spawnParticles(e.x+TILE/2, e.y+TILE/2, '#ffcc88', 8, {speed:2, glow:true});
        spawnParticles(e.x+TILE/2, e.y+TILE/2, C.blood, 5);
        spawnFloatText(e.x+TILE/2, e.y, '-'+player.dmg, '#ffaa44');
        if (e.hp<=0){
          spawnParticles(e.x+TILE/2, e.y+TILE/2, C.blood, 20, {speed:3});
          spawnParticles(e.x+TILE/2, e.y+TILE/2, C.bloodD, 10, {speed:2});
          bloodDecals.push({ x:e.x+8, y:e.y+20, r:10+Math.random()*6, a:0.7, t:0 });
          enemies.splice(i,1); kills++; gainXp(1); sndKill();
        }
      }
    }
    if (hitAny){ screenShake = 5; sndHit(); }
  }

  // ==================== ЗНАКИ ====================
  function switchSign(){
    if (!player.charData) return;
    var idx = ALL_SIGNS.indexOf(player.currentSign);
    idx = (idx + 1) % ALL_SIGNS.length;
    player.currentSign = ALL_SIGNS[idx];
    updateSignButton(player.currentSign);
    document.getElementById('signLabel').textContent = SIGN_NAMES[player.currentSign];
    spawnFloatText(player.x+TILE/2, player.y-25, SIGN_NAMES[player.currentSign], '#ffcc88');
    playSound(700, 0.08, 'sine', 0.05);
  }

  function castSign(){
    if (isDead || player.signCd>0 || player.st<30) return;
    initAudio();
    player.st -= 30;
    player.signCd = 45;
    sndSign();
    var sign = player.currentSign;
    if (sign === 'igni') castIgni();
    else if (sign === 'quen') castQuen();
    else if (sign === 'aard') castAard();
    else if (sign === 'yrden') castYrden();
    else if (sign === 'axii') castAxii();
  }

  function castIgni(){
    var dx=0, dy=0;
    if (player.face==='right') dx=1; else if (player.face==='left') dx=-1;
    else if (player.face==='down') dy=1; else dy=-1;
    var sx = player.x+TILE/2+dx*TILE*0.6;
    var sy = player.y+TILE/2+dy*TILE*0.6;
    var range = TILE*3.2;
    screenShake = 8;
    for (var i=enemies.length-1; i>=0; i--){
      var e = enemies[i];
      var ex = e.x+TILE/2, ey = e.y+TILE/2;
      var ddx = ex-sx, ddy = ey-sy;
      var d = Math.sqrt(ddx*ddx+ddy*ddy);
      if (d<range){
        var dot = (ddx*dx+ddy*dy)/d;
        if (dot>0.45){
          e.hp -= player.signDmg;
          e.hitT = 14;
          e.x += dx*24; e.y += dy*24;
          spawnParticles(ex, ey, '#ff8844', 14, {speed:2.5, glow:true});
          spawnFloatText(ex, ey-5, '-'+player.signDmg, '#ffcc44');
          if (e.hp<=0){
            spawnParticles(ex, ey, C.blood, 20, {speed:3});
            spawnParticles(ex, ey, '#ff5522', 16, {speed:3, glow:true});
            enemies.splice(i,1); kills++; gainXp(1); sndKill();
          }
        }
      }
    }
    igniFx = { x:sx, y:sy, dx:dx, dy:dy, t:16 };
    for (var k=0;k<30;k++){
      var a2 = Math.atan2(dy,dx) + (Math.random()-0.5)*0.9;
      var sp = 2+Math.random()*4;
      particles.push({
        x:sx+(Math.random()-0.5)*8, y:sy+(Math.random()-0.5)*8,
        vx:Math.cos(a2)*sp, vy:Math.sin(a2)*sp,
        life:18+Math.random()*20, maxLife:38,
        color: Math.random()<0.4?C.fireW:(Math.random()<0.5?C.fireG:C.fire),
        size:3+Math.random()*4, gravity: -0.05, glow:true
      });
    }
  }

  function castQuen(){
    player.shieldHp = 3;
    player.shieldTimer = 8 * 60; // 8 секунд
    sndShield();
    spawnFloatText(player.x+TILE/2, player.y-20, 'КВЕН', '#88ccff');
    for (var i=0;i<30;i++){
      var a = Math.random()*Math.PI*2;
      var s = 1 + Math.random()*2;
      particles.push({
        x:player.x+TILE/2, y:player.y+TILE/2,
        vx:Math.cos(a)*s, vy:Math.sin(a)*s,
        life:30, maxLife:30, color:'#88aaff', size:3, gravity:0, glow:true
      });
    }
  }

  function castAard(){
    var dx=0, dy=0;
    if (player.face==='right') dx=1; else if (player.face==='left') dx=-1;
    else if (player.face==='down') dy=1; else dy=-1;
    var sx = player.x+TILE/2;
    var sy = player.y+TILE/2;
    var range = TILE*4.5;
    screenShake = 10;
    for (var i=enemies.length-1; i>=0; i--){
      var e = enemies[i];
      var ex = e.x+TILE/2, ey = e.y+TILE/2;
      var ddx = ex-sx, ddy = ey-sy;
      var d = Math.sqrt(ddx*ddx+ddy*ddy);
      if (d<range){
        var dot = (ddx*dx+ddy*dy)/d;
        if (dot>0.35){
          e.hp -= 1;
          e.hitT = 14;
          // Отбрасывание сильное
          e.x += dx*60; e.y += dy*60;
          spawnParticles(ex, ey, '#aaddff', 16, {speed:3, glow:true});
          spawnFloatText(ex, ey-5, 'ОТБРОС', '#aaddff');
          if (e.hp<=0){
            spawnParticles(ex, ey, C.blood, 20, {speed:3});
            enemies.splice(i,1); kills++; gainXp(1); sndKill();
          }
        }
      }
    }
    // Визуал — волна
    for (var k=0;k<40;k++){
      var off = k/40 * range;
      var px = sx + dx*off;
      var py = sy + dy*off;
      particles.push({
        x:px+(Math.random()-0.5)*30, y:py+(Math.random()-0.5)*30,
        vx:dx*3, vy:dy*3,
        life:20, maxLife:20, color:'#cceeff', size:4, gravity:0, glow:true
      });
    }
  }

  function castYrden(){
    // Ловушка на земле перед игроком
    var tx = player.x + TILE/2;
    var ty = player.y + TILE/2;
    yrdenFx.push({ x:tx, y:ty, timer: 8*60, tick: 0 });
    spawnFloatText(tx, ty-30, 'ИРДЕН', '#cc88ff');
    for (var i=0;i<25;i++){
      var a = Math.random()*Math.PI*2;
      var r = 20 + Math.random()*40;
      particles.push({
        x:tx + Math.cos(a)*r, y:ty + Math.sin(a)*r,
        vx:0, vy:-0.5 - Math.random(),
        life:40, maxLife:40, color:'#aa66ff', size:3, gravity:0, glow:true
      });
    }
  }

  function castAxii(){
    // Находим ближайшего врага
    var target = null;
    var bestD = 200;
    for (var i=0;i<enemies.length;i++){
      var e = enemies[i];
      var d = Math.hypot(e.x-player.x, e.y-player.y);
      if (d < bestD){ bestD = d; target = e; }
    }
    if (!target){
      spawnFloatText(player.x+TILE/2, player.y-20, 'НЕТ ЦЕЛИ', '#888');
      return;
    }
    target.mindControlled = 6*60; // 6 секунд
    target.hitT = 20;
    spawnFloatText(target.x+TILE/2, target.y-20, 'АКСИЙ', '#ffdd66');
    for (var k=0;k<20;k++){
      var a = Math.random()*Math.PI*2;
      var s = 1 + Math.random()*2;
      particles.push({
        x:target.x+TILE/2, y:target.y+TILE/2,
        vx:Math.cos(a)*s, vy:Math.sin(a)*s-1,
        life:30, maxLife:30, color:'#ffdd66', size:3, gravity:0, glow:true
      });
    }
  }

  // ==================== XP + БРОНЯ ====================
  function updateArmorUI(){
    document.getElementById('slotHelm').classList.toggle('on', armor.helm);
    document.getElementById('slotChest').classList.toggle('on', armor.chest);
    document.getElementById('slotLegs').classList.toggle('on', armor.legs);
  }

  function gainXp(n){
    player.xp += n;
    if (player.xp >= player.xpNext){
      player.xp -= player.xpNext;
      player.level++;
      player.xpNext = Math.floor(player.xpNext * 1.5);
      if (player.level === 2 && !armor.helm){
        armor.helm = true; player.maxHp++; player.hp = player.maxHp;
        spawnFloatText(player.x+TILE/2, player.y-30, 'ШЛЕМ НАЙДЕН!', '#aaddff');
        sndArmor(); updateArmorUI();
      } else if (player.level === 3 && !armor.chest){
        armor.chest = true; player.maxHp++; player.hp = player.maxHp;
        spawnFloatText(player.x+TILE/2, player.y-30, 'КУРТКА НАЙДЕНА!', '#ffcc88');
        sndArmor(); updateArmorUI();
      } else if (player.level === 4 && !armor.legs){
        armor.legs = true; player.maxHp++; player.hp = player.maxHp;
        spawnFloatText(player.x+TILE/2, player.y-30, 'ПОНОЖИ НАЙДЕНЫ!', '#ddaa88');
        sndArmor(); updateArmorUI();
      } else {
        player.maxHp++; player.hp = player.maxHp;
      }
      sndLevel();
      spawnFloatText(player.x+TILE/2, player.y-10, 'УРОВЕНЬ ' + player.level, '#ffdd44');
      for (var i=0;i<20;i++){
        var a = Math.random()*Math.PI*2;
        var s = 2+Math.random()*2;
        particles.push({
          x:player.x+TILE/2, y:player.y+TILE/2,
          vx:Math.cos(a)*s, vy:Math.sin(a)*s-1,
          life:40, maxLife:40, color:'#ffdd44', size:3, gravity:0.05, glow:true
        });
      }
    }
  }

  // ==================== СМЕРТЬ ====================
  function die(){
    isDead = true;
    document.getElementById('deathKills').textContent = kills;
    document.getElementById('deathChar').textContent = player.charData ? player.charData.name : '—';
    document.getElementById('deathScreen').classList.add('show');
    spawnParticles(player.x+TILE/2, player.y+TILE/2, C.blood, 30, {speed:3});
    playSound(80, 0.5, 'sawtooth', 0.1);
  }
  function respawn(){
    isDead = false;
    player.hp = player.maxHp;
    player.st = player.maxSt;
    player.x = 15*TILE; player.y = 15*TILE;
    player.inv = 60;
    enemies.forEach(function(en){
      var a = Math.atan2(en.y-player.y, en.x-player.x);
      en.x += Math.cos(a)*100; en.y += Math.sin(a)*100;
    });
    document.getElementById('deathScreen').classList.remove('show');
  }

  // ==================== РИСОВАНИЕ ИГРОКА ====================
  function drawPlayer(px, py, pBob){
    ctx.fillStyle = C.shadow;
    ctx.beginPath();
    ctx.ellipse(px+TILE/2, py+TILE-3, 13, 4, 0, 0, Math.PI*2);
    ctx.fill();

    if (player.inv>0 && Math.floor(player.inv/4)%2===0) ctx.globalAlpha = 0.45;

    // Данные персонажа
    var ch = player.charData || CHARACTERS[0];
    var skinColor = ch.color || C.skin;
    var hairColor = ch.hair || C.hair;
    var hairDark = ch.id === 'lambert' ? '#5a3a28' : (ch.id === 'ciri' ? '#a0a0a0' : C.hairD);

    var face = player.face;
    var cx = px + TILE/2;
    var cy = py + TILE/2;

    if (face === 'down'){
      ctx.fillStyle = C.cloakD;
      ctx.fillRect(px+3, py+10+pBob, 26, 18);
      ctx.fillStyle = C.cloak;
      ctx.fillRect(px+4, py+8+pBob, 24, 18);
      if (armor.legs){
        ctx.fillStyle = C.legArmorD;
        ctx.fillRect(px+7, py+26+pBob, 6, 6);
        ctx.fillRect(px+19, py+26+pBob, 6, 6);
        ctx.fillStyle = C.legArmor;
        ctx.fillRect(px+7, py+26+pBob, 6, 4);
        ctx.fillRect(px+19, py+26+pBob, 6, 4);
        ctx.fillStyle = C.legArmorH;
        ctx.fillRect(px+8, py+27+pBob, 4, 2);
        ctx.fillRect(px+20, py+27+pBob, 4, 2);
      } else {
        ctx.fillStyle = C.cloakD;
        ctx.fillRect(px+7, py+26+pBob, 6, 6);
        ctx.fillRect(px+19, py+26+pBob, 6, 6);
      }
      if (armor.chest){
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+5, py+11+pBob, 22, 14);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+6, py+10+pBob, 20, 13);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+8, py+12+pBob, 5, 4);
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+2, py+10+pBob, 4, 8);
        ctx.fillRect(px+26, py+10+pBob, 4, 8);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+2, py+10+pBob, 4, 3);
        ctx.fillRect(px+26, py+10+pBob, 4, 3);
        ctx.fillStyle = '#d4b58a';
        ctx.fillRect(px+15, py+14+pBob, 2, 2);
        ctx.fillRect(px+15, py+18+pBob, 2, 2);
      } else {
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+6, py+11+pBob, 20, 13);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+7, py+10+pBob, 18, 12);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+9, py+12+pBob, 5, 4);
      }
      // Голова
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+10, py+3+pBob, 12, 12);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+10, py+2+pBob, 12, 11);
      // Волосы
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+8, py+0+pBob, 16, 6);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+8, py+0+pBob, 16, 4);
      ctx.fillRect(px+6, py+2+pBob, 4, 8);
      ctx.fillRect(px+22, py+2+pBob, 4, 8);
      // Борода (не у Цири)
      if (ch.id !== 'ciri'){
        ctx.fillStyle = ch.id==='eskel' ? '#6a5a3a' : C.beard;
        ctx.fillRect(px+11, py+11+pBob, 10, 3);
        ctx.fillStyle = hairDark;
        ctx.fillRect(px+11, py+13+pBob, 10, 1);
      }
      // Шлем
      if (armor.helm){
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+8, py-2+pBob, 16, 8);
        ctx.fillStyle = C.helm;
        ctx.fillRect(px+9, py-1+pBob, 14, 6);
        ctx.fillStyle = C.helmH;
        ctx.fillRect(px+10, py+pBob, 5, 2);
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+10, py+4+pBob, 12, 1);
      }
      // Глаза
      ctx.globalAlpha *= 0.5;
      ctx.fillStyle = C.eyeGlow;
      ctx.fillRect(px+11, py+6+pBob, 5, 5);
      ctx.fillRect(px+17, py+6+pBob, 5, 5);
      ctx.globalAlpha = player.inv>0 && Math.floor(player.inv/4)%2===0 ? 0.45 : 1;
      ctx.fillStyle = C.eye;
      ctx.fillRect(px+12, py+7+pBob, 3, 3);
      ctx.fillRect(px+18, py+7+pBob, 3, 3);
      ctx.fillStyle = '#000';
      ctx.fillRect(px+13, py+8+pBob, 1, 1);
      ctx.fillRect(px+19, py+8+pBob, 1, 1);
      if (ch.id === 'geralt'){
        ctx.fillStyle = C.skinS;
        ctx.fillRect(px+16, py+6+pBob, 2, 5);
      }
      // Меч
      ctx.fillStyle = C.swordB;
      ctx.fillRect(px+28, py+13+pBob, 16, 5);
      ctx.fillStyle = C.sword;
      ctx.fillRect(px+28, py+12+pBob, 16, 3);
      ctx.fillStyle = C.swordH;
      ctx.fillRect(px+28, py+12+pBob, 16, 1);
      ctx.fillStyle = '#5a4530';
      ctx.fillRect(px+40, py+10+pBob, 4, 9);

    } else if (face === 'up'){
      ctx.fillStyle = C.cloakD;
      ctx.fillRect(px+3, py+10+pBob, 26, 18);
      ctx.fillStyle = C.cloak;
      ctx.fillRect(px+4, py+8+pBob, 24, 20);
      if (armor.legs){
        ctx.fillStyle = C.legArmorD;
        ctx.fillRect(px+7, py+26+pBob, 6, 6);
        ctx.fillRect(px+19, py+26+pBob, 6, 6);
        ctx.fillStyle = C.legArmor;
        ctx.fillRect(px+7, py+26+pBob, 6, 4);
        ctx.fillRect(px+19, py+26+pBob, 6, 4);
      } else {
        ctx.fillStyle = C.cloakD;
        ctx.fillRect(px+7, py+26+pBob, 6, 6);
        ctx.fillRect(px+19, py+26+pBob, 6, 6);
      }
      if (armor.chest){
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+5, py+11+pBob, 22, 14);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+6, py+10+pBob, 20, 13);
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+2, py+10+pBob, 4, 8);
        ctx.fillRect(px+26, py+10+pBob, 4, 8);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+2, py+10+pBob, 4, 3);
        ctx.fillRect(px+26, py+10+pBob, 4, 3);
        ctx.fillStyle = '#5a3a1a';
        ctx.fillRect(px+12, py+12+pBob, 8, 2);
        ctx.fillRect(px+14, py+10+pBob, 2, 8);
      } else {
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+6, py+11+pBob, 20, 13);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+7, py+10+pBob, 18, 12);
      }
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+10, py+3+pBob, 12, 12);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+10, py+2+pBob, 12, 11);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+8, py+0+pBob, 16, 12);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+8, py+0+pBob, 16, 10);
      ctx.fillRect(px+6, py+2+pBob, 4, 10);
      ctx.fillRect(px+22, py+2+pBob, 4, 10);
      ctx.fillStyle = '#5a3a1a';
      ctx.fillRect(px+14, py+11+pBob, 4, 2);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+15, py+13+pBob, 2, 6);
      if (armor.helm){
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+8, py-2+pBob, 16, 8);
        ctx.fillStyle = C.helm;
        ctx.fillRect(px+9, py-1+pBob, 14, 6);
        ctx.fillStyle = C.helmH;
        ctx.fillRect(px+10, py+pBob, 5, 2);
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+10, py+6+pBob, 12, 2);
      }
      ctx.fillStyle = C.swordB;
      ctx.fillRect(px+30, py+4+pBob, 6, 14);
      ctx.fillStyle = C.sword;
      ctx.fillRect(px+31, py+4+pBob, 4, 14);
      ctx.fillStyle = C.swordH;
      ctx.fillRect(px+31, py+4+pBob, 2, 14);
      ctx.fillStyle = '#5a4530';
      ctx.fillRect(px+30, py+18+pBob, 6, 3);

    } else if (face === 'right'){
      ctx.fillStyle = C.cloakD;
      ctx.fillRect(px+2, py+9+pBob, 8, 18);
      ctx.fillStyle = C.cloak;
      ctx.fillRect(px+3, py+8+pBob, 6, 18);
      if (armor.legs){
        ctx.fillStyle = C.legArmorD;
        ctx.fillRect(px+10, py+26+pBob, 8, 6);
        ctx.fillStyle = C.legArmor;
        ctx.fillRect(px+10, py+26+pBob, 8, 4);
        ctx.fillStyle = C.legArmorH;
        ctx.fillRect(px+11, py+27+pBob, 5, 2);
      } else {
        ctx.fillStyle = C.cloakD;
        ctx.fillRect(px+10, py+26+pBob, 8, 6);
      }
      if (armor.chest){
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+8, py+11+pBob, 16, 14);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+9, py+10+pBob, 14, 13);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+10, py+12+pBob, 4, 4);
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+7, py+9+pBob, 5, 8);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+7, py+9+pBob, 5, 3);
      } else {
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+9, py+11+pBob, 14, 13);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+10, py+10+pBob, 12, 12);
      }
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+11, py+3+pBob, 12, 12);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+11, py+2+pBob, 12, 11);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+23, py+6+pBob, 2, 3);
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+23, py+8+pBob, 1, 1);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+9, py+0+pBob, 14, 6);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+9, py+0+pBob, 14, 4);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+6, py+2+pBob, 4, 8);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+6, py+2+pBob, 4, 6);
      if (ch.id !== 'ciri'){
        ctx.fillStyle = ch.id==='eskel' ? '#6a5a3a' : C.beard;
        ctx.fillRect(px+13, py+11+pBob, 8, 3);
      }
      if (armor.helm){
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+9, py-2+pBob, 16, 8);
        ctx.fillStyle = C.helm;
        ctx.fillRect(px+10, py-1+pBob, 14, 6);
        ctx.fillStyle = C.helmH;
        ctx.fillRect(px+11, py+pBob, 5, 2);
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+11, py+4+pBob, 14, 1);
      }
      ctx.globalAlpha *= 0.5;
      ctx.fillStyle = C.eyeGlow;
      ctx.fillRect(px+17, py+6+pBob, 5, 5);
      ctx.globalAlpha = player.inv>0 && Math.floor(player.inv/4)%2===0 ? 0.45 : 1;
      ctx.fillStyle = C.eye;
      ctx.fillRect(px+18, py+7+pBob, 3, 3);
      ctx.fillStyle = '#000';
      ctx.fillRect(px+19, py+8+pBob, 1, 1);
      ctx.fillStyle = C.swordB;
      ctx.fillRect(px+24, py+13+pBob, 20, 5);
      ctx.fillStyle = C.sword;
      ctx.fillRect(px+24, py+12+pBob, 20, 3);
      ctx.fillStyle = C.swordH;
      ctx.fillRect(px+24, py+12+pBob, 20, 1);
      ctx.fillStyle = '#5a4530';
      ctx.fillRect(px+40, py+10+pBob, 4, 9);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+22, py+12+pBob, 4, 5);

    } else if (face === 'left'){
      ctx.fillStyle = C.cloakD;
      ctx.fillRect(px+22, py+9+pBob, 8, 18);
      ctx.fillStyle = C.cloak;
      ctx.fillRect(px+23, py+8+pBob, 6, 18);
      if (armor.legs){
        ctx.fillStyle = C.legArmorD;
        ctx.fillRect(px+14, py+26+pBob, 8, 6);
        ctx.fillStyle = C.legArmor;
        ctx.fillRect(px+14, py+26+pBob, 8, 4);
        ctx.fillStyle = C.legArmorH;
        ctx.fillRect(px+16, py+27+pBob, 5, 2);
      } else {
        ctx.fillStyle = C.cloakD;
        ctx.fillRect(px+14, py+26+pBob, 8, 6);
      }
      if (armor.chest){
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+8, py+11+pBob, 16, 14);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+9, py+10+pBob, 14, 13);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+18, py+12+pBob, 4, 4);
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+20, py+9+pBob, 5, 8);
        ctx.fillStyle = C.armorH;
        ctx.fillRect(px+20, py+9+pBob, 5, 3);
      } else {
        ctx.fillStyle = C.armorD;
        ctx.fillRect(px+9, py+11+pBob, 14, 13);
        ctx.fillStyle = C.armor;
        ctx.fillRect(px+10, py+10+pBob, 12, 12);
      }
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+9, py+3+pBob, 12, 12);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+9, py+2+pBob, 12, 11);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+7, py+6+pBob, 2, 3);
      ctx.fillStyle = C.skinS;
      ctx.fillRect(px+7, py+8+pBob, 1, 1);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+9, py+0+pBob, 14, 6);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+9, py+0+pBob, 14, 4);
      ctx.fillStyle = hairDark;
      ctx.fillRect(px+22, py+2+pBob, 4, 8);
      ctx.fillStyle = hairColor;
      ctx.fillRect(px+22, py+2+pBob, 4, 6);
      if (ch.id !== 'ciri'){
        ctx.fillStyle = ch.id==='eskel' ? '#6a5a3a' : C.beard;
        ctx.fillRect(px+11, py+11+pBob, 8, 3);
      }
      if (armor.helm){
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+7, py-2+pBob, 16, 8);
        ctx.fillStyle = C.helm;
        ctx.fillRect(px+8, py-1+pBob, 14, 6);
        ctx.fillStyle = C.helmH;
        ctx.fillRect(px+10, py+pBob, 5, 2);
        ctx.fillStyle = C.helmD;
        ctx.fillRect(px+7, py+4+pBob, 14, 1);
      }
      ctx.globalAlpha *= 0.5;
      ctx.fillStyle = C.eyeGlow;
      ctx.fillRect(px+10, py+6+pBob, 5, 5);
      ctx.globalAlpha = player.inv>0 && Math.floor(player.inv/4)%2===0 ? 0.45 : 1;
      ctx.fillStyle = C.eye;
      ctx.fillRect(px+11, py+7+pBob, 3, 3);
      ctx.fillStyle = '#000';
      ctx.fillRect(px+12, py+8+pBob, 1, 1);
      ctx.fillStyle = C.swordB;
      ctx.fillRect(px-12, py+13+pBob, 20, 5);
      ctx.fillStyle = C.sword;
      ctx.fillRect(px-12, py+12+pBob, 20, 3);
      ctx.fillStyle = C.swordH;
      ctx.fillRect(px-12, py+12+pBob, 20, 1);
      ctx.fillStyle = '#5a4530';
      ctx.fillRect(px-16, py+10+pBob, 4, 9);
      ctx.fillStyle = skinColor;
      ctx.fillRect(px+6, py+12+pBob, 4, 5);
    }

    // Анимация замаха
    if (player.atkT>0){
      var aAlpha = player.atkT/9;
      ctx.globalAlpha = aAlpha * 0.7;
      ctx.strokeStyle = '#fff8d0';
      ctx.lineWidth = 3;
      ctx.beginPath();
      var start = 0, end = 0;
      if (face==='right'){ start = -0.8; end = 0.8; }
      else if (face==='left'){ start = Math.PI-0.8; end = Math.PI+0.8; }
      else if (face==='up'){ start = -Math.PI/2-0.8; end = -Math.PI/2+0.8; }
      else { start = Math.PI/2-0.8; end = Math.PI/2+0.8; }
      ctx.arc(cx, cy, TILE*1.1, start, end);
      ctx.stroke();
    }

    // Аура щита Квен
    if (player.shieldHp > 0){
      var shieldAlpha = 0.3 + Math.sin(performance.now()*0.008)*0.15;
      ctx.globalAlpha = shieldAlpha;
      ctx.strokeStyle = '#88ccff';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(cx, cy, TILE*0.9, 0, Math.PI*2);
      ctx.stroke();
      ctx.globalAlpha = 1;
    }

    ctx.globalAlpha = 1;
  }

  // ==================== РИСОВАНИЕ ====================
  function draw(){
    ctx.clearRect(0,0,VIEW_W,VIEW_H);

    var nightFactor = 0;
    var hour = gameTime * 24;
    if (hour < 6) nightFactor = 0.5;
    else if (hour < 8) nightFactor = 0.5 - (hour-6)/2 * 0.5;
    else if (hour < 17) nightFactor = 0;
    else if (hour < 20) nightFactor = (hour-17)/3 * 0.5;
    else nightFactor = 0.5;

    var camX = player.x - VIEW_W/2 + TILE/2;
    var camY = player.y - VIEW_H/2 + TILE/2;
    var maxX = MAP_W*TILE - VIEW_W;
    var maxY = MAP_H*TILE - VIEW_H;
    if (camX<0) camX=0; if (camX>maxX) camX=maxX;
    if (camY<0) camY=0; if (camY>maxY) camY=maxY;

    var shakeX = 0, shakeY = 0;
    if (screenShake>0){
      shakeX = (Math.random()-0.5)*screenShake;
      shakeY = (Math.random()-0.5)*screenShake;
    }

    ctx.save();
    ctx.translate(shakeX, shakeY);

    var sc = Math.floor(camX/TILE);
    var ec = Math.min(MAP_W-1, sc+Math.ceil(VIEW_W/TILE)+1);
    var sr = Math.floor(camY/TILE);
    var er = Math.min(MAP_H-1, sr+Math.ceil(VIEW_H/TILE)+1);

    for (var r=sr; r<=er; r++){
      for (var c=sc; c<=ec; c++){
        var tile = map[r][c];
        var x = c*TILE-camX, y = r*TILE-camY;

        if (tile===0){
          ctx.fillStyle = ((r+c)%2===0)?C.g1:C.g2;
          ctx.fillRect(x,y,TILE,TILE);
          ctx.fillStyle = C.g3;
          for (var k=0;k<5;k++){
            var px = x+((r*13+c*7+k*11)%(TILE-4))+2;
            var py = y+((r*17+c*11+k*5)%(TILE-4))+2;
            ctx.fillRect(px,py,2,2);
          }
          ctx.fillStyle = C.g4;
          for (var k2=0;k2<3;k2++){
            var px2 = x+((r*23+c*31+k2*19)%(TILE-6))+2;
            var py2 = y+((r*31+c*23+k2*17)%(TILE-6))+2;
            ctx.fillRect(px2,py2,1,3);
          }
        } else if (tile===1){
          ctx.fillStyle = C.shadow;
          ctx.fillRect(x+3,y+5,TILE-4,TILE-5);
          ctx.fillStyle = C.treeD;
          ctx.fillRect(x,y,TILE,TILE);
          ctx.fillStyle = C.tree;
          ctx.fillRect(x+2,y+2,TILE-4,TILE-4);
          ctx.fillStyle = C.treeH;
          ctx.fillRect(x+6,y+5,8,8);
          ctx.fillRect(x+16,y+12,7,7);
          ctx.fillRect(x+10,y+20,6,6);
          ctx.fillStyle = C.treeHD;
          ctx.fillRect(x+8,y+7,3,3);
          ctx.fillRect(x+18,y+14,3,3);
          ctx.fillStyle = C.trunk;
          ctx.fillRect(x+12,y+24,8,8);
          ctx.fillStyle = C.trunkL;
          ctx.fillRect(x+13,y+25,2,6);
        } else if (tile===2){
          ctx.fillStyle = C.shadow;
          ctx.fillRect(x+3,y+5,TILE-4,TILE-5);
          ctx.fillStyle = C.rockD;
          ctx.fillRect(x,y,TILE,TILE);
          ctx.fillStyle = C.rock;
          ctx.fillRect(x+3,y+4,26,24);
          ctx.fillStyle = C.rockH;
          ctx.fillRect(x+6,y+8,8,6);
          ctx.fillRect(x+18,y+16,6,5);
          ctx.fillStyle = C.rockS;
          ctx.fillRect(x+20,y+8,5,5);
          ctx.fillRect(x+6,y+20,6,6);
        } else if (tile===3){
          var waveT = (r*7+c*13) + performance.now()*0.002;
          var wave = Math.sin(waveT)*2;
          ctx.fillStyle = C.water;
          ctx.fillRect(x,y,TILE,TILE);
          ctx.fillStyle = C.waterL;
          ctx.fillRect(x, y+4+wave, TILE, 3);
          ctx.fillRect(x, y+14-wave, TILE, 2);
          ctx.fillRect(x, y+24+wave, TILE, 2);
          ctx.fillStyle = C.waterH;
          for (var w=0;w<3;w++){
            var wx = x + ((r*11+c*7+w*13)%(TILE-6))+2;
            var wy = y + ((r*13+c*11+w*7)%(TILE-6))+2;
            ctx.fillRect(wx, wy+wave, 3, 1);
          }
        }
      }
    }

    // Кровавые пятна
    for (var bd=0; bd<bloodDecals.length; bd++){
      var bl = bloodDecals[bd];
      var alpha = Math.max(0, bl.a * (1 - bl.t/400));
      ctx.globalAlpha = alpha;
      ctx.fillStyle = C.bloodD;
      ctx.beginPath();
      ctx.ellipse(bl.x-camX, bl.y-camY, bl.r, bl.r*0.6, 0, 0, Math.PI*2);
      ctx.fill();
    }
    ctx.globalAlpha = 1;

    // Декор
    for (var d=0; d<decor.length; d++){
      var de = decor[d];
      if (de.x < camX-TILE || de.x > camX+VIEW_W+TILE) continue;
      if (de.y < camY-TILE || de.y > camY+VIEW_H+TILE) continue;
      var dx2 = de.x-camX, dy2 = de.y-camY;
      var sway = Math.sin(performance.now()*0.002 + de.phase) * 1.5;
      if (de.t==='grass'){
        ctx.fillStyle = C.g4;
        ctx.fillRect(dx2+10+de.s*8+sway, dy2+18, 2, 8);
        ctx.fillRect(dx2+14+de.s*6+sway, dy2+14, 2, 10);
        ctx.fillRect(dx2+18+de.s*4+sway, dy2+20, 2, 6);
      } else if (de.t==='flower'){
        ctx.fillStyle = C.g4;
        ctx.fillRect(dx2+14+sway, dy2+20, 2, 8);
        ctx.fillStyle = '#d4d45a';
        ctx.fillRect(dx2+13+sway, dy2+16, 4, 4);
        ctx.fillStyle = de.s<0.5?'#ff8888':'#cc88ff';
        ctx.fillRect(dx2+12+sway, dy2+14, 2, 2);
        ctx.fillRect(dx2+16+sway, dy2+18, 2, 2);
      } else {
        ctx.fillStyle = '#e8e8d4';
        ctx.fillRect(dx2+14, dy2+20, 3, 6);
        ctx.fillStyle = '#a83a2a';
        ctx.fillRect(dx2+11, dy2+16, 9, 5);
        ctx.fillStyle = '#fff';
        ctx.fillRect(dx2+13, dy2+17, 2, 2);
        ctx.fillRect(dx2+17, dy2+18, 2, 2);
      }
    }

    // Ловушки Ирден
    for (var yi=0; yi<yrdenFx.length; yi++){
      var yf = yrdenFx[yi];
      var yx = yf.x-camX, yy = yf.y-camY;
      var pulse = 0.4 + Math.sin(performance.now()*0.01)*0.2;
      ctx.globalAlpha = pulse;
      ctx.strokeStyle = '#aa66ff';
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.arc(yx, yy, 60, 0, Math.PI*2);
      ctx.stroke();
      // Внутренние руны
      ctx.globalAlpha = pulse * 0.6;
      ctx.beginPath();
      ctx.arc(yx, yy, 40, 0, Math.PI*2);
      ctx.stroke();
      // Магические точки
      for (var r2=0;r2<6;r2++){
        var ang = r2/6*Math.PI*2 + performance.now()*0.002;
        var rx = yx + Math.cos(ang)*40;
        var ry = yy + Math.sin(ang)*40;
        ctx.fillStyle = '#ff88ff';
        ctx.fillRect(rx-2, ry-2, 4, 4);
      }
      ctx.globalAlpha = 1;
    }

    // Враги
    for (var i=0;i<enemies.length;i++){
      var e = enemies[i];
      var ex = e.x-camX, ey = e.y-camY;
      var bob = Math.sin(e.walkT)*1.8;

      ctx.fillStyle = C.shadow;
      ctx.beginPath();
      ctx.ellipse(ex+TILE/2, ey+TILE-3, 12, 4, 0, 0, Math.PI*2);
      ctx.fill();

      var c1, c1d, c1h;
      if (e.type==='drowner'){ c1=C.en2; c1d=C.en2d; c1h=C.en2h; }
      else if (e.type==='wraith'){ c1=C.en3; c1d=C.en3d; c1h=C.en3h; }
      else { c1=C.en1; c1d=C.en1d; c1h=C.en1h; }

      if (e.type==='wraith'){
        ctx.globalAlpha = 0.25 + Math.sin(performance.now()*0.003 + e.phase)*0.1;
        ctx.fillStyle = '#aa88ff';
        ctx.beginPath();
        ctx.ellipse(ex+TILE/2, ey+TILE/2+bob, 18, 22, 0, 0, Math.PI*2);
        ctx.fill();
        ctx.globalAlpha = 1;
      }
      if (e.mindControlled > 0){
        ctx.globalAlpha = 0.4 + Math.sin(performance.now()*0.01)*0.15;
        ctx.strokeStyle = '#ffdd66';
        ctx.lineWidth = 3;
        ctx.beginPath();
        ctx.arc(ex+TILE/2, ey+TILE/2, 22, 0, Math.PI*2);
        ctx.stroke();
        ctx.globalAlpha = 1;
      }
      if (e.slowed > 0){
        ctx.globalAlpha = 0.4;
        ctx.fillStyle = '#aa66ff';
        ctx.fillRect(ex+2, ey+2, 28, 28);
        ctx.globalAlpha = 1;
      }

      ctx.fillStyle = c1d;
      ctx.fillRect(ex+4, ey+8+bob, 24, 20);
      ctx.fillStyle = c1;
      ctx.fillRect(ex+5, ey+7+bob, 22, 18);
      ctx.fillStyle = c1h;
      ctx.fillRect(ex+7, ey+9+bob, 6, 6);
      ctx.fillStyle = c1d;
      ctx.fillRect(ex+7, ey+24+bob, 5, 6);
      ctx.fillRect(ex+20, ey+24+bob, 5, 6);

      ctx.globalAlpha = 0.5;
      ctx.fillStyle = e.type==='wraith'?'#cc88ff':'#ff2222';
      ctx.fillRect(ex+8, ey+11+bob, 8, 6);
      ctx.fillRect(ex+16, ey+11+bob, 8, 6);
      ctx.globalAlpha = 1;
      ctx.fillStyle = e.type==='wraith'?'#ffccff':'#ff2222';
      ctx.fillRect(ex+10, ey+12+bob, 4, 4);
      ctx.fillRect(ex+18, ey+12+bob, 4, 4);
      ctx.fillStyle = '#fff';
      ctx.fillRect(ex+11, ey+13+bob, 2, 2);
      ctx.fillRect(ex+19, ey+13+bob, 2, 2);
      ctx.fillStyle = '#fff';
      ctx.fillRect(ex+12, ey+19+bob, 2, 3);
      ctx.fillRect(ex+16, ey+19+bob, 2, 3);
      ctx.fillRect(ex+20, ey+19+bob, 2, 3);

      if (e.hp<e.maxHp){
        ctx.fillStyle = '#2a1a1a';
        ctx.fillRect(ex+2, ey-8, 28, 5);
        var hpw = 28*(e.hp/e.maxHp);
        var grad2 = ctx.createLinearGradient(ex+2, ey-8, ex+2+hpw, ey-3);
        grad2.addColorStop(0, '#ff6666');
        grad2.addColorStop(1, '#a82222');
        ctx.fillStyle = grad2;
        ctx.fillRect(ex+2, ey-8, hpw, 5);
        ctx.strokeStyle = '#000';
        ctx.lineWidth = 1;
        ctx.strokeRect(ex+2, ey-8, 28, 5);
      }
      if (e.hitT>0){
        ctx.fillStyle = 'rgba(255,255,255,0.65)';
        ctx.fillRect(ex+2, ey+6+bob, 28, 22);
      }
    }

    // Игрок
    var px = player.x-camX, py = player.y-camY;
    var pBob = player.moving ? Math.sin(player.walkT)*1.8 : 0;
    drawPlayer(px, py, pBob);

    // Шлейф меча
    for (var st2=0; st2<player.swordTrail.length; st2++){
      var trail = player.swordTrail[st2];
      var tAlpha = trail.life / 8 * 0.3;
      ctx.globalAlpha = tAlpha;
      ctx.fillStyle = '#fff';
      var tx = trail.x-camX, ty = trail.y-camY;
      ctx.fillRect(tx+8, ty+8, 24, 24);
    }
    ctx.globalAlpha = 1;

    // Частицы
    for (var p2=0; p2<particles.length; p2++){
      var pt = particles[p2];
      var alpha = pt.life/pt.maxLife;
      if (pt.glow){
        ctx.globalAlpha = alpha * 0.5;
        ctx.fillStyle = pt.color;
        ctx.fillRect(pt.x-camX-pt.size, pt.y-camY-pt.size, pt.size*2, pt.size*2);
      }
      ctx.globalAlpha = alpha;
      ctx.fillStyle = pt.color;
      ctx.fillRect(pt.x-camX-pt.size/2, pt.y-camY-pt.size/2, pt.size, pt.size);
    }
    ctx.globalAlpha = 1;

    // Игни
    if (igniFx){
      var ix = igniFx.x-camX, iy = igniFx.y-camY;
      ctx.globalAlpha = 0.35;
      var igrad = ctx.createRadialGradient(ix, iy, 0, ix, iy, 80);
      igrad.addColorStop(0, '#ffdd88');
      igrad.addColorStop(0.5, 'rgba(255,140,60,0.5)');
      igrad.addColorStop(1, 'rgba(255,80,20,0)');
      ctx.fillStyle = igrad;
      ctx.fillRect(ix-80, iy-80, 160, 160);
      ctx.globalAlpha = 1;
      for (var j=0;j<12;j++){
        var off = j*5-24;
        var fx = ix + igniFx.dx*off + (Math.random()-0.5)*12;
        var fy = iy + igniFx.dy*off + (Math.random()-0.5)*12;
        var sz = 16-j;
        ctx.globalAlpha = 0.3;
        ctx.fillStyle = C.fireW;
        ctx.fillRect(fx-sz/2-3, fy-sz/2-3, sz+6, sz+6);
        ctx.globalAlpha = 1;
        ctx.fillStyle = j<3?C.fireW:(j<6?C.fireG:C.fire);
        ctx.fillRect(fx-sz/2, fy-sz/2, sz, sz);
      }
      igniFx.t--;
      if (igniFx.t<=0) igniFx=null;
    }

    ctx.restore();

    // Плавающий текст
    for (var ft2=0; ft2<floatTexts.length; ft2++){
      var f2 = floatTexts[ft2];
      var fa = f2.life/f2.maxLife;
      ctx.globalAlpha = fa;
      ctx.font = 'bold 16px "Courier New"';
      ctx.textAlign = 'center';
      ctx.strokeStyle = '#000';
      ctx.lineWidth = 3;
      ctx.strokeText(f2.text, f2.x-camX, f2.y-camY);
      ctx.fillStyle = f2.color;
      ctx.fillText(f2.text, f2.x-camX, f2.y-camY);
    }
    ctx.globalAlpha = 1;
    ctx.textAlign = 'left';

    if (nightFactor > 0){
      ctx.fillStyle = 'rgba(20,20,60,'+nightFactor*0.6+')';
      ctx.fillRect(0,0,VIEW_W,VIEW_H);
    }

    var grad = ctx.createRadialGradient(VIEW_W/2, VIEW_H/2, VIEW_H*0.35, VIEW_W/2, VIEW_H/2, VIEW_H*0.8);
    grad.addColorStop(0, 'rgba(0,0,0,0)');
    grad.addColorStop(1, 'rgba(0,0,0,0.6)');
    ctx.fillStyle = grad;
    ctx.fillRect(0,0,VIEW_W,VIEW_H);

    if (player.hp<=2 && !isDead){
      var pulse = 0.15 + Math.sin(Date.now()/200)*0.1;
      ctx.fillStyle = 'rgba(180,0,0,'+pulse+')';
      ctx.fillRect(0,0,VIEW_W,VIEW_H);
    }
  }

  // ==================== ЦИКЛ ====================
  var hpEl = document.getElementById('hpBar').firstElementChild;
  var stEl = document.getElementById('stBar').firstElementChild;
  var killsEl = document.getElementById('kills');
  var timeEl = document.getElementById('time');
  var cdAttackEl = document.getElementById('cdAttack');
  var cdSignEl = document.getElementById('cdSign');

  var last = 0;
  function loop(ts){
    var dt = Math.min(0.05, (ts-last)/1000);
    last = ts;
    update(dt);
    draw();
    requestAnimationFrame(loop);
  }

  // ==================== ДЖОЙСТИК ====================
  var joy = { active:false, id:null, cx:0, cy:0, dx:0, dy:0 };
  var joyEl = document.getElementById('joy');
  var knobEl = document.getElementById('joyKnob');
  var joyRect = null;

  function joyStart(e){
    e.preventDefault();
    initAudio();
    var t = e.changedTouches ? e.changedTouches[0] : e;
    joy.active = true;
    joy.id = t.identifier !== undefined ? t.identifier : 'mouse';
    joyRect = joyEl.getBoundingClientRect();
    joy.cx = joyRect.left + joyRect.width/2;
    joy.cy = joyRect.top + joyRect.height/2;
    joyMove(e);
  }
  function joyMove(e){
    if (!joy.active) return;
    e.preventDefault();
    var t = null;
    if (e.changedTouches){
      for (var i=0;i<e.changedTouches.length;i++)
        if (e.changedTouches[i].identifier === joy.id) t = e.changedTouches[i];
      if (!t) return;
    } else t = e;
    var dx = t.clientX - joy.cx;
    var dy = t.clientY - joy.cy;
    var max = joyRect.width/2;
    var d = Math.sqrt(dx*dx+dy*dy);
    if (d>max){ dx = dx/d*max; dy = dy/d*max; }
    joy.dx = dx/max;
    joy.dy = dy/max;
    knobEl.style.transform = 'translate(' + dx + 'px,' + dy + 'px)';
  }
  function joyEnd(e){
    if (!joy.active) return;
    e.preventDefault();
    joy.active = false;
    joy.dx = 0; joy.dy = 0;
    knobEl.style.transform = 'translate(0,0)';
  }
  joyEl.addEventListener('touchstart', joyStart, {passive:false});
  joyEl.addEventListener('touchmove', joyMove, {passive:false});
  joyEl.addEventListener('touchend', joyEnd, {passive:false});
  joyEl.addEventListener('touchcancel', joyEnd, {passive:false});
  joyEl.addEventListener('mousedown', joyStart);
  window.addEventListener('mousemove', joyMove);
  window.addEventListener('mouseup', joyEnd);

  // ==================== КНОПКИ ====================
  function bindBtn(el, fn){
    el.addEventListener('touchstart', function(e){ e.preventDefault(); initAudio(); fn(); }, {passive:false});
    el.addEventListener('mousedown', function(e){ e.preventDefault(); initAudio(); fn(); });
  }
  bindBtn(document.getElementById('btnAttack'), attack);
  bindBtn(document.getElementById('btnSign'), castSign);
  document.getElementById('signSwitch').addEventListener('click', function(e){
    e.preventDefault(); initAudio(); switchSign();
  });
  document.getElementById('signSwitch').addEventListener('touchstart', function(e){
    e.preventDefault(); initAudio(); switchSign();
  }, {passive:false});

  // ==================== ПОЛНЫЙ ЭКРАН ====================
  document.getElementById('fsBtn').addEventListener('click', function(){
    var el = document.documentElement;
    if (!document.fullscreenElement && !document.webkitFullscreenElement){
      if (el.requestFullscreen) el.requestFullscreen();
      else if (el.webkitRequestFullscreen) el.webkitRequestFullscreen();
      else if (el.msRequestFullscreen) el.msRequestFullscreen();
    } else {
      if (document.exitFullscreen) document.exitFullscreen();
      else if (document.webkitExitFullscreen) document.webkitExitFullscreen();
      else if (document.msExitFullscreen) document.msExitFullscreen();
    }
  });

  // ==================== КЛАВИАТУРА ====================
  var keys = {};
  window.addEventListener('keydown', function(e){
    keys[e.code] = true;
    if (e.code==='Space'){ e.preventDefault(); initAudio(); attack(); }
    if (e.code==='KeyE'){ e.preventDefault(); initAudio(); castSign(); }
    if (e.code==='KeyQ'){ e.preventDefault(); initAudio(); switchSign(); }
    if (e.code==='KeyF'){ e.preventDefault();
      var el = document.documentElement;
      if (!document.fullscreenElement) el.requestFullscreen && el.requestFullscreen();
      else document.exitFullscreen && document.exitFullscreen();
    }
  });
  window.addEventListener('keyup', function(e){ keys[e.code] = false; });

  setInterval(function(){
    if (joy.active) return;
    var dx=0, dy=0;
    if (keys['KeyW']||keys['ArrowUp']) dy-=1;
    if (keys['KeyS']||keys['ArrowDown']) dy+=1;
    if (keys['KeyA']||keys['ArrowLeft']) dx-=1;
    if (keys['KeyD']||keys['ArrowRight']) dx+=1;
    joy.dx = dx; joy.dy = dy;
  }, 30);

  // ==================== PWA ====================
  (function setupPWA(){
    try {
      var svgIcon = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">' +
        '<rect width="512" height="512" fill="#1a1410"/>' +
        '<rect x="60" y="60" width="392" height="392" fill="#2d4a2a" rx="30"/>' +
        '<circle cx="256" cy="200" r="80" fill="#d4b28c"/>' +
        '<rect x="200" y="140" width="112" height="40" fill="#e8e8e8"/>' +
        '<circle cx="230" cy="200" r="12" fill="#e6c422"/>' +
        '<circle cx="282" cy="200" r="12" fill="#e6c422"/>' +
        '<rect x="160" y="280" width="192" height="120" fill="#8b6b4b"/>' +
        '<rect x="320" y="200" width="20" height="200" fill="#d4d4d4"/>' +
        '<text x="256" y="470" font-size="52" text-anchor="middle" fill="#e6d5b8" font-family="monospace" font-weight="bold">WITCHER</text>' +
        '</svg>';
      var iconData = 'data:image/svg+xml;base64,' + btoa(unescape(encodeURIComponent(svgIcon)));
      var manifest = {
        "name": "Ведьмак: Пиксельный Путь",
        "short_name": "Ведьмак",
        "description": "Пиксельная экшн-игра в стиле Ведьмака",
        "start_url": ".",
        "display": "fullscreen",
        "orientation": "any",
        "background_color": "#0a0806",
        "theme_color": "#1a1410",
        "icons": [{ "src": iconData, "sizes": "512x512", "type": "image/svg+xml", "purpose": "any" }]
      };
      var manifestBlob = new Blob([JSON.stringify(manifest)], {type: 'application/manifest+json'});
      var manifestURL = URL.createObjectURL(manifestBlob);
      var link = document.createElement('link');
      link.rel = 'manifest';
      link.href = manifestURL;
      document.head.appendChild(link);

      var appleIcon = document.createElement('link');
      appleIcon.rel = 'apple-touch-icon';
      appleIcon.href = iconData;
      document.head.appendChild(appleIcon);

      if ('serviceWorker' in navigator) {
        var swCode = "self.addEventListener('install', function(e){ self.skipWaiting(); });\n" +
          "self.addEventListener('activate', function(e){ e.waitUntil(self.clients.claim()); });\n" +
          "self.addEventListener('fetch', function(e){ e.respondWith(fetch(e.request).catch(function(){ return new Response('', {status: 200}); })); });";
        var swBlob = new Blob([swCode], {type: 'application/javascript'});
        var swURL = URL.createObjectURL(swBlob);
        navigator.serviceWorker.register(swURL, {scope: './'}).then(function(reg){
          console.log('✅ SW зарегистрирован:', reg.scope);
        }).catch(function(err){
          console.log('⚠️ SW не зарегистрирован:', err.message);
        });
      }

      var deferredPrompt = null;
      var installBtn = document.getElementById('installBtn');
      window.addEventListener('beforeinstallprompt', function(e){
        e.preventDefault();
        deferredPrompt = e;
        installBtn.style.display = 'block';
      });
      installBtn.addEventListener('click', function(){
        if (!deferredPrompt) return;
        deferredPrompt.prompt();
        deferredPrompt.userChoice.then(function(choice){
          if (choice.outcome === 'accepted') installBtn.style.display = 'none';
          deferredPrompt = null;
        });
      });
      window.addEventListener('appinstalled', function(){
        installBtn.style.display = 'none';
        console.log('✅ Приложение установлено');
      });
    } catch(err){
      console.log('⚠️ PWA ошибка:', err.message);
    }
  })();

  // ==================== СТАРТ ====================
  document.getElementById('respawnBtn').addEventListener('click', respawn);

  document.getElementById('startBtn').addEventListener('click', function(){
    if (!selectedChar) return;
    initAudio();
    // Настройка игрока под персонажа
    player.charData = selectedChar;
    player.speed = selectedChar.speed;
    player.dmg = selectedChar.dmg;
    player.signDmg = selectedChar.signDmg;
    player.maxHp = selectedChar.maxHp;
    player.hp = selectedChar.maxHp;
    // Стартовый знак
    if (selectedChar.sign === 'blink'){
      // Цири — рывок вместо знака (основной кнопкой)
      player.currentSign = 'igni'; // для отображения
      // Но кнопка знака Цири делает рывок
      document.getElementById('signLabel').textContent = '💫 Рывок';
    } else {
      player.currentSign = selectedChar.sign;
      document.getElementById('signLabel').textContent = SIGN_NAMES[selectedChar.sign];
    }
    updateSignButton(player.currentSign);
    // Скрыть экран выбора
    document.getElementById('charSelect').classList.add('hidden');
    canvas.style.display = 'block';
    document.getElementById('hud').style.display = 'flex';
    document.getElementById('joy').style.display = 'block';
    document.getElementById('signSwitch').style.display = 'flex';
    document.getElementById('btns').style.display = 'flex';
    resize();
    genMap();
    for (var i=0;i<4;i++) spawnEnemy();
    updateArmorUI();
    console.log('🐺 Старт: ' + selectedChar.name);
    requestAnimationFrame(loop);
  });

  // Для Цири знак = рывок: переопределим castSign
  var originalCastSign = castSign;
  castSign = function(){
    if (!player.charData) return;
    if (player.charData.id === 'ciri'){
      // Рывок
      if (isDead || player.signCd>0 || player.st<30) return;
      initAudio();
      player.st -= 30;
      player.signCd = 45;
      sndSign();
      var dx = joy.dx, dy = joy.dy;
      var mag = Math.sqrt(dx*dx+dy*dy);
      if (mag < 0.2){
        if (player.face==='right'){ dx=1; dy=0; }
        else if (player.face==='left'){ dx=-1; dy=0; }
        else if (player.face==='up'){ dx=0; dy=-1; }
        else { dx=0; dy=1; }
      } else { dx/=mag; dy/=mag; }
      // Телепорт на 100 пикселей вперёд
      var tx = player.x + dx*TILE*3;
      var ty = player.y + dy*TILE*3;
      // Проверка: если нельзя — двигаем насколько можем
      var steps = 0;
      var sx = player.x, sy = player.y;
      for (var s=1; s<=10; s++){
        var nx = player.x + dx*TILE*0.3*s;
        var ny = player.y + dy*TILE*0.3*s;
        if (canMove(nx, ny)){ sx = nx; sy = ny; steps = s; }
        else break;
      }
      // Следы
      for (var k=0;k<20;k++){
        var t = k/20;
        var fx = player.x + (sx-player.x)*t;
        var fy = player.y + (sy-player.y)*t;
        particles.push({
          x:fx+TILE/2, y:fy+TILE/2,
          vx:(Math.random()-0.5)*2, vy:(Math.random()-0.5)*2,
          life:20, maxLife:20, color:'#aaddff', size:4, gravity:0, glow:true
        });
      }
      player.x = sx; player.y = sy;
      player.inv = 20;
      spawnFloatText(player.x+TILE/2, player.y-20, 'РЫВОК', '#aaddff');
      return;
    }
    originalCastSign();
  };

})();
</script>
</body>
</html>
