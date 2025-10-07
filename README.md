<!doctype html>
<html lang="he">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1" />
<title>משחק עוגייה משודרג</title>
<style>
html, body {
  height:100%;
  margin:0;
  font-family:Arial,Helvetica,sans-serif;
  overflow:hidden;
  touch-action: manipulation;
  background: repeating-linear-gradient(
    0deg,
    #1e90ff,
    #1e90ff 10px,
    #187bcd 10px,
    #187bcd 20px
  );
}
body {
  display:flex;
  justify-content:center;
  align-items:center;
  flex-direction:column;
}
.game {
  text-align:center;
  display:flex;
  flex-direction:column;
  justify-content:center;
  align-items:center;
  width:100%;
  height:100%;
}
.main-cookie {
  width: clamp(100px,30vw,180px);
  height: clamp(100px,30vw,180px);
  background: linear-gradient(145deg,#d2996e,#b8733a);
  border-radius:50%;
  position:relative;
  margin:20px 0;
  box-shadow:0 8px 25px rgba(0,0,0,0.3);
  cursor:pointer;
  transition:transform 0.15s;
}
.chip {
  position:absolute;
  width:10%;
  height:8%;
  background:#3b2b1b;
  border-radius:50%;
}
#counter { font-size:clamp(22px,6vw,36px); color:#fff; margin-top:10px; }
#chance { font-size:clamp(12px,3.5vw,16px); color:#fff; opacity:0.9; }
.stats {
  position:fixed;
  top:1vh;
  left:2vw;
  display:flex;
  flex-direction:column;
  gap:4px;
  font-size:clamp(12px,3vw,18px);
  color:#fff;
  z-index:10;
}
.days-stats {
  position:fixed;
  top:1vh;
  right:2vw;
  font-size:clamp(12px,3vw,18px);
  color:#fff;
  z-index:10;
}
.bonus-cookie {
  width:clamp(35px,10vw,60px);
  height:clamp(35px,10vw,60px);
  background:linear-gradient(145deg,#d2996e,#b8733a);
  border-radius:50%;
  position:absolute;
  cursor:pointer;
  box-shadow:0 4px 15px rgba(0,0,0,0.25);
  transition:transform 0.1s ease,opacity 0.3s ease;
}

/* כפתורים */
#resetBtn { position: fixed; bottom: 2vh; right: calc(2vw + 110px); background: #e53935; font-weight: bold; border: none; border-radius: 6px; padding: 8px 12px; color: #fff; font-size: clamp(12px, 3.5vw, 16px); cursor: pointer; z-index: 60; }
#creditsBtn { position: fixed; bottom: 2vh; right: 2vw; background: #444; color: #fff; border: none; border-radius: 6px; padding: 8px 12px; font-size: clamp(12px, 3.5vw, 16px); cursor: pointer; z-index: 60; }
#achievementsBtn { position: fixed; bottom: 2vh; left: 2vw; background: #444; color: #fff; border: none; border-radius: 6px; padding: 8px 12px; font-size: clamp(12px, 3.5vw, 16px); cursor: pointer; z-index: 60; }

#creditsList, #achievementsList {
  position: fixed;
  bottom: calc(2vh + 50px);
  background: rgba(0,0,0,0.85);
  color: #fff;
  padding: 10px;
  border-radius: 8px;
  display: none;
  max-height: 40vh;
  overflow-y: auto;
  font-size: clamp(12px, 3vw, 14px);
  z-index: 60;
}
#creditsList { right: 2vw; }
#achievementsList { left: 2vw; width: 220px; }

.modal-overlay {
  position:fixed;
  top:0; left:0;
  width:100%; height:100%;
  background:rgba(0,0,0,0.7);
  display:flex; justify-content:center; align-items:center;
  z-index:999; visibility:hidden; opacity:0;
  transition:0.3s;
}
.modal-overlay.active { visibility:visible; opacity:1; }
.modal-box {
  background:#222;
  color:#fff;
  padding:20px;
  border-radius:12px;
  text-align:center;
  width:80%;
  max-width:300px;
}
.modal-buttons { display:flex; justify-content:space-around; margin-top:15px; }
.modal-buttons button {
  padding:8px 14px;
  border:none;
  border-radius:6px;
  font-size:clamp(12px,3.5vw,16px);
  cursor:pointer;
  font-weight:bold;
}
.btn-yes { background:#e53935; color:#fff; }
.btn-no { background:#555; color:#fff; }
</style>
</head>
<body>

<div class="stats">
  <div id="totalClicks">מספר לחיצות: 0</div>
  <div id="highScore">שיא: 0</div>
  <div id="bonusClicks">עוגיות קטנות: 0</div>
</div>
<div class="days-stats"><div id="daysPlayed">ימים: 0</div></div>

<div id="gameArea" class="game">
  <div id="mainCookie" class="main-cookie">
    <div class="chip" style="top:25%;left:25%;"></div>
    <div class="chip" style="top:15%;right:25%;"></div>
    <div class="chip" style="bottom:25%;left:20%;"></div>
    <div class="chip" style="bottom:20%;right:30%;"></div>
    <div class="chip" style="top:45%;left:45%;"></div>
  </div>
  <div id="counter">0</div>
  <div id="chance">סיכוי לאיפוס: 0%</div>
</div>

<button id="resetBtn">!!!איפוס המשחק</button>
<button id="creditsBtn">Credits</button>
<button id="achievementsBtn">🏆 הישגים</button>

<div id="creditsList"></div>
<div id="achievementsList"></div>

<div class="modal-overlay" id="resetModal">
  <div class="modal-box">
    <h3>לאפס הכול?</h3>
    <div class="modal-buttons">
      <button class="btn-yes" id="confirmReset">✅ כן</button>
      <button class="btn-no" id="cancelReset">❌ לא</button>
    </div>
  </div>
</div>

<script>
const mainCookie = document.getElementById('mainCookie');
const counterEl = document.getElementById('counter');
const chanceEl = document.getElementById('chance');
const totalClicksEl = document.getElementById('totalClicks');
const highScoreEl = document.getElementById('highScore');
const bonusClicksEl = document.getElementById('bonusClicks');
const daysPlayedEl = document.getElementById('daysPlayed');
const creditsBtn = document.getElementById('creditsBtn');
const creditsList = document.getElementById('creditsList');
const achievementsBtn = document.getElementById('achievementsBtn');
const achievementsList = document.getElementById('achievementsList');
const resetBtn = document.getElementById('resetBtn');
const resetModal = document.getElementById('resetModal');
const confirmReset = document.getElementById('confirmReset');
const cancelReset = document.getElementById('cancelReset');

let count = parseInt(localStorage.getItem('currentCount'))||0;
let totalClicks = parseInt(localStorage.getItem('totalClicks'))||0;
let highScore = parseInt(localStorage.getItem('highScore'))||0;
let bonusClicks = parseInt(localStorage.getItem('bonusClicks'))||0;
let resetAtThree = localStorage.getItem('resetAtThree')==='true';
let smallCookie = null;

if(!localStorage.getItem('startDate')) localStorage.setItem('startDate',Date.now());

function updateDisplay(){
  counterEl.textContent=count;
  chanceEl.textContent=`סיכוי לאיפוס: ${count}%`;
  totalClicksEl.textContent=`מספר לחיצות: ${totalClicks}`;
  highScoreEl.textContent=`שיא: ${highScore}`;
  bonusClicksEl.textContent=`עוגיות קטנות: ${bonusClicks}`;
  const days=Math.floor((Date.now()-parseInt(localStorage.getItem('startDate')))/(1000*60*60*24));
  daysPlayedEl.textContent=`ימים: ${days}`;
  checkAchievements();
}

// פונקציה ליצירת פיצפוצי שוקולד סטטיים
function staticChocolatePop(container){
  for(let i=0;i<5;i++){
    const choc=document.createElement('div');
    choc.style.position='absolute';
    choc.style.width='8px'; choc.style.height='8px';
    choc.style.background='#3b2b1b';
    choc.style.borderRadius='50%';
    const rect=container.getBoundingClientRect();
    choc.style.left=(Math.random()*rect.width)+'px';
    choc.style.top=(Math.random()*rect.height)+'px';
    container.appendChild(choc);
  }
}

function incrementClick(){
  totalClicks++; count++;
  if(Math.random()*100<count){
    if(count===3){resetAtThree=true;localStorage.setItem('resetAtThree','true');}
    count=0;
  }
  if(count>highScore) highScore=count;
  localStorage.setItem('currentCount',count);
  localStorage.setItem('totalClicks',totalClicks);
  localStorage.setItem('highScore',highScore);
  updateDisplay(); checkSmallCookie();
}
mainCookie.addEventListener('click',incrementClick);

// עוגיות קטנות
function spawnSmallCookie(){
  if(smallCookie) return;
  smallCookie=document.createElement('div');
  smallCookie.classList.add('bonus-cookie');
  const gameRect=document.getElementById('gameArea').getBoundingClientRect();
  smallCookie.style.left=Math.random()*(gameRect.width-60)+'px';
  smallCookie.style.top=Math.random()*(gameRect.height-60)+'px';
  document.getElementById('gameArea').appendChild(smallCookie);

  // פיצפוצי שוקולד סטטיים על העוגייה הקטנה
  staticChocolatePop(smallCookie);

  smallCookie.addEventListener('click',()=>{
    bonusClicks++; 
    localStorage.setItem('bonusClicks',bonusClicks);
    updateDisplay(); 
    removeSmallCookie();
  });
}
function removeSmallCookie(){if(!smallCookie)return; smallCookie.remove(); smallCookie=null;}
function checkSmallCookie(){if(totalClicks>0 && totalClicks%100===0) spawnSmallCookie();}

// הישגים
function checkAchievements(){
  const unlocked=[highScore>=30,totalClicks>=3000,bonusClicks>=40,resetAtThree];
  const achNames=[
    'השג 1: שיא 30',
    'השג 2: 3000 לחיצות',
    'השג 3: 40 עוגיות קטנות',
    'השג 4: איפוס אחרי 3 לחיצות',
    'השג 5: כל ההשגים פתוחים'
  ];
  achievementsList.innerHTML='';
  for(let i=0;i<achNames.length;i++){
    const div=document.createElement('div');
    div.textContent=achNames[i];
    div.style.padding='4px';
    div.style.borderBottom='1px solid rgba(255,255,255,0.2)';
    div.style.background=unlocked[i]|| (i===4 && unlocked.every(x=>x)) ? '#28a745' : '#444';
    achievementsList.appendChild(div);
  }
}

// כפתורים קרדיט והישגים
creditsBtn.addEventListener('click',()=>{
  creditsList.style.display=creditsList.style.display==='block'?'none':'block';
  if(creditsList.style.display==='block'){
    const names=['יהונתן שור','יונתן שורן','יונתן שורץ','יונה שור','יונתן שורק','יונתן שורה','יונתן שורט','יונתן שורמ','יונתן שורג','יונתן שורי'];
    creditsList.innerHTML=names.sort(()=>Math.random()-0.5).map(n=>`<div>${n}</div>`).join('');
  }
});
achievementsBtn.addEventListener('click',()=>{
  achievementsList.style.display=achievementsList.style.display==='block'?'none':'block';
});

// איפוס
resetBtn.addEventListener('click',()=>{resetModal.classList.add('active');});
cancelReset.addEventListener('click',()=>{resetModal.classList.remove('active');});
confirmReset.addEventListener('click',()=>{
  count=0; totalClicks=0; highScore=0; bonusClicks=0; resetAtThree=false;
  localStorage.clear();
  resetModal.classList.remove('active');
  updateDisplay();
});

updateDisplay();
</script>
</body>
</html>
