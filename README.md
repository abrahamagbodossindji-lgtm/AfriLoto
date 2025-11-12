# AfriLoto
AfriLoto est le jeu de tirage en ligne 100 % africain qui te donne la chance de changer ta vie ! Joue depuis ton téléphone, choisis tes numéros et tente de décrocher le jackpot. Tirages sécurisés, gains instantanés et plaisir garanti ! AfriLoto, la chance de tout un continent !
<!doctype html>
<html lang="fr">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>AfriLoto - Démo</title>
  <style>
    :root{--bg:#f7f9fc;--card:#fff;--accent:#0a74da;--muted:#666}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,Arial,Helvetica,sans-serif;background:linear-gradient(180deg,#e8f0ff, #f7f9fc);color:#111;display:flex;align-items:flex-start;justify-content:center;padding:28px}
    .wrap{width:100%;max-width:980px}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    h1{margin:0;font-size:20px}
    .card{background:var(--card);border-radius:12px;padding:16px;box-shadow:0 6px 18px rgba(10,20,50,0.06);margin-bottom:14px}
    .top{display:flex;gap:14px;align-items:center}
    .credits{font-weight:700;font-size:18px;color:var(--accent)}
    .small{font-size:13px;color:var(--muted)}
    .numbers{display:flex;flex-wrap:wrap;gap:8px;margin-top:10px}
    .num{width:44px;height:44px;border-radius:8px;border:1px solid #dbe7ff;display:flex;align-items:center;justify-content:center;cursor:pointer;background:#fff}
    .num.selected{background:#ffd54f;border-color:#ffb300}
    .controls{display:flex;gap:8px;margin-top:10px}
    button{padding:10px 12px;border-radius:8px;border:none;background:var(--accent);color:#fff;cursor:pointer}
    button.ghost{background:transparent;color:var(--accent);border:1px solid #cfe0ff}
    .row{display:flex;gap:12px;flex-wrap:wrap}
    .col{flex:1;min-width:240px}
    .drawArea .num{background:#f3f7ff;border-color:#cfe0ff}
    .result{margin-top:10px;font-weight:700}
    .history{max-height:220px;overflow:auto;margin-top:10px;border-top:1px dashed #eef3ff;padding-top:10px}
    table{width:100%;border-collapse:collapse}
    td,th{padding:6px 8px;border-bottom:1px solid #f1f6ff;font-size:13px}
    footer{margin-top:6px;font-size:12px;color:var(--muted);text-align:center}
    @media(max-width:600px){.num{width:38px;height:38px;font-size:14px}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>AfriLoto — Démo gratuite</h1>
      <div class="top">
        <div class="small">Crédits :</div>
        <div id="credits" class="credits">0</div>
      </div>
    </header>

    <div class="card">
      <div class="small">Sélectionne 6 numéros (1–49)</div>
      <div id="choices" class="numbers" aria-label="choix-numeros"></div>

      <div class="controls">
        <button id="autoBtn" class="ghost">Choix automatique</button>
        <button id="playBtn">Jouer (50 pts)</button>
        <button id="addCredits" class="ghost">+500 pts (démo)</button>
        <button id="resetBtn" class="ghost">Réinitialiser</button>
      </div>

      <div class="row" style="margin-top:14px">
        <div class="col card drawArea">
          <div class="small">Tirage :</div>
          <div id="draw" class="numbers" aria-live="polite"></div>
          <div id="result" class="result"></div>
        </div>

        <div class="col card">
          <div class="small">Règles & Gains</div>
          <table>
            <thead><tr><th>Matchs</th><th>Gain (points)</th></tr></thead>
            <tbody>
              <tr><td>0-2</td><td>0</td></tr>
              <tr><td>3</td><td>200</td></tr>
              <tr><td>4</td><td>1 000</td></tr>
              <tr><td>5</td><td>20 000</td></tr>
              <tr><td>6</td><td>1 000 000</td></tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <div class="card">
      <div class="small">Historique des parties</div>
      <div id="history" class="history"></div>
    </div>

    <footer class="small card">Prototype gratuit — AfriLoto. Pas d'argent réel. Héberge sur GitHub Pages.</footer>
  </div>

<script>
/*
  AfriLoto - Prototype Loto Demo
  - Coût ticket : 50 points
  - Credits sauvegardés en localStorage
  - Historique sauvegardé en localStorage
*/

const TICKET_COST = 50
const CHOICES_COUNT = 6
const MAX_NUM = 49
const STORAGE_KEY = 'afriloto_v1'

let state = {
  credits: 1000,
  history: []
}

// load state
function loadState(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY)
    if(raw) state = JSON.parse(raw)
  }catch(e){ console.warn('load error', e) }
}
function saveState(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(state)) }

loadState()

// DOM
const creditsEl = document.getElementById('credits')
const choicesEl = document.getElementById('choices')
const drawEl = document.getElementById('draw')
const resultEl = document.getElementById('result')
const historyEl = document.getElementById('history')
const autoBtn = document.getElementById('autoBtn')
const playBtn = document.getElementById('playBtn')
const addBtn = document.getElementById('addCredits')
const resetBtn = document.getElementById('resetBtn')

let selected = []

function renderCredits(){
  creditsEl.textContent = state.credits.toLocaleString()
}

function renderChoices(){
  choicesEl.innerHTML = ''
  for(let i=1;i<=MAX_NUM;i++){
    const d = document.createElement('div')
    d.className = 'num' + (selected.includes(i)?' selected':'')
    d.textContent = i
    d.onclick = ()=>{
      if(selected.includes(i)) selected = selected.filter(x=>x!==i)
      else if(selected.length < CHOICES_COUNT) selected.push(i)
      else {
        // remplace le dernier selectionné si déjà plein (UX optionnelle)
        // selected[selected.length-1] = i
        // or ignore
        alert('Tu as déjà choisi 6 numéros.')
      }
      renderChoices()
    }
    choicesEl.appendChild(d)
  }
}

function randUnique(n, max){
  const arr = []
  while(arr.length < n){
    const r = Math.floor(Math.random()*max) + 1
    if(!arr.includes(r)) arr.push(r)
  }
  return arr.sort((a,b)=>a-b)
}

function renderDraw(arr){
  drawEl.innerHTML = ''
  arr.forEach(n=>{
    const d = document.createElement('div')
    d.className = 'num'
    d.textContent = n
    drawEl.appendChild(d)
  })
}

function computeGain(matches){
  if(matches >= 6) return 1000000
  if(matches === 5) return 20000
  if(matches === 4) return 1000
  if(matches === 3) return 200
  return 0
}

function addHistory(entry){
  state.history.unshift(entry)
  if(state.history.length > 100) state.history.length = 100
  saveState()
  renderHistory()
}

function renderHistory(){
  if(state.history.length === 0){ historyEl.innerHTML = '<div class="small">Aucune partie pour l\'instant.</div>'; return }
  let html = '<table><thead><tr><th>Date</th><th>Choix</th><th>Tirage</th><th>Matches</th><th>Gain</th></tr></thead><tbody>'
  state.history.forEach(h=>{
    html += `<tr><td>${new Date(h.time).toLocaleString()}</td><td>${h.choice.join(', ')}</td><td>${h.draw.join(', ')}</td><td>${h.matches}</td><td>${h.gain.toLocaleString()}</td></tr>`
  })
  html += '</tbody></table>'
  historyEl.innerHTML = html
}

// Buttons
autoBtn.onclick = ()=> {
  selected = randUnique(CHOICES_COUNT, MAX_NUM)
  renderChoices()
}

playBtn.onclick = ()=>{
  if(selected.length !== CHOICES_COUNT){ alert('Choisis 6 numéros.') ; return }
  if(state.credits < TICKET_COST){ alert('Pas assez de crédits. Utilise +500 pts pour démo.') ; return }
  state.credits -= TICKET_COST
  const draw = randUnique(CHOICES_COUNT, MAX_NUM)
  renderDraw(draw)
  const matchesArr = selected.filter(x => draw.includes(x)).sort((a,b)=>a-b)
  const matchesCount = matchesArr.length
  const gain = computeGain(matchesCount)
  state.credits += gain
  saveState()
  renderCredits()
  resultEl.textContent = `Tu as ${matchesCount} bonnes réponses (${matchesArr.join(', ') || '—'}). Gain : ${gain.toLocaleString()} pts.`
  addHistory({
    time: Date.now(),
    choice: [...selected].sort((a,b)=>a-b),
    draw,
    matches: matchesCount,
    gain
  })
  // reset selection
  selected = []
  renderChoices()
}

addBtn.onclick = ()=>{
  state.credits += 500
  saveState()
  renderCredits()
  alert('+500 pts ajoutés (mode démo)')
}

resetBtn.onclick = ()=>{
  if(!confirm('Supprimer les données locales (crédits + historique) ?')) return
  state = { credits: 1000, history: [] }
  saveState()
  renderCredits()
  renderHistory()
  selected = []
  renderChoices()
  drawEl.innerHTML = ''
  resultEl.textContent = ''
}

// init
renderCredits()
renderChoices()
renderHistory()

// keyboard quick actions (optionnel)
document.addEventListener('keydown', (e)=>{
  if(e.key === 'a') autoBtn.click()
  if(e.key === 'p') playBtn.click()
})
</script>
</body>
</html>
