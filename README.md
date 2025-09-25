# Nikita-s-studying-app

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Nikita Study Buddy</title>
  <style>
    /* ---------- Basic reset & layout ---------- */
    :root{
      --bg1: #fff1f2; --bg2:#ecfeff; --bg3:#f3fef8;
      --accent: #0f172a; --muted:#475569; --card:#ffffffcc;
      --primary:#0b69ff; --success:#10b981; --gold:#ffc857;
    }
    *{box-sizing:border-box;font-family:Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;}
    html,body{height:100%;margin:0;background:linear-gradient(120deg,var(--bg1),var(--bg2),var(--bg3));}
    .wrap{max-width:1080px;margin:28px auto;padding:20px;}
    header{display:flex;align-items:center;justify-content:space-between;gap:12px}
    h1{margin:0;font-size:26px;letter-spacing:0.2px}
    .sub{color:var(--muted);font-size:13px}
    nav{margin-top:14px;display:flex;flex-wrap:wrap;gap:8px}
    .btn{background:#fff;padding:8px 12px;border-radius:12px;border:0;box-shadow:0 6px 18px rgba(12,12,12,0.06);cursor:pointer;font-weight:600}
    .btn.primary{background:var(--accent);color:#fff}
    .btn.ghost{background:transparent;border:1px solid rgba(0,0,0,0.06)}
    main{margin-top:18px}
    .grid{display:grid;gap:14px}
    .cols-2{grid-template-columns:1fr 380px}
    .card{background:var(--card);backdrop-filter: blur(6px);padding:16px;border-radius:14px;box-shadow:0 8px 24px rgba(15,23,42,0.06)}
    .small{font-size:13px;color:var(--muted)}
    /* ---------- colorful panels ---------- */
    .panel-1{background:linear-gradient(135deg,#fff3e0,#ffe7f3);border:1px solid rgba(0,0,0,0.02)}
    .panel-2{background:linear-gradient(135deg,#eef9ff,#eefbe9)}
    .panel-3{background:linear-gradient(135deg,#f0f7ff,#f6f0ff)}
    .panel-4{background:linear-gradient(135deg,#fff8f0,#f0fff6)}
    /* ---------- flashcards ---------- */
    .flashcard{border-radius:12px;padding:18px;min-height:110px;display:flex;flex-direction:column;justify-content:center;text-align:center}
    /* ---------- list ---------- */
    ul{padding-left:18px;margin:8px 0}
    li{margin:6px 0}
    label{display:block;margin-top:8px;font-size:13px;color:var(--muted)}
    input[type="text"], input[type="number"], select, textarea{
      width:100%;padding:8px;border-radius:8px;border:1px solid rgba(0,0,0,0.06);margin-top:6px;
    }
    .row{display:flex;gap:8px}
    .row .col{flex:1}
    .small-btn{padding:6px 10px;border-radius:10px;border:0;cursor:pointer}
    .badge{display:inline-block;padding:6px 10px;border-radius:999px;background:linear-gradient(90deg,#ffd966,#ff8a65);color:#000;font-weight:700}
    .muted{color:var(--muted)}
    .center{text-align:center}
    /* ---------- nav active ---------- */
    .nav-active{background:var(--accent);color:#fff}
    /* ---------- responsive ---------- */
    @media (max-width:880px){ .cols-2{grid-template-columns:1fr} header{flex-direction:column;align-items:flex-start} }
    /* ---------- little animations ---------- */
    .pop { transition: transform .18s ease; }
    .pop:active{ transform: scale(.98); }
    .confetti { position:fixed; pointer-events:none; z-index:9999; left:0; top:0; width:100%; height:100%; overflow:hidden; }
    /* progress bar */
    .prog{height:10px;background:#e6eefb;border-radius:999px;overflow:hidden}
    .prog > i{display:block;height:100%;background:linear-gradient(90deg,#7dd3fc,#60a5fa)}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1 id="title">🦉 NikitaStudyBuddy</h1>
        <div class="sub">Fun study space for 3rd grade — colorful, interactive, and kid-friendly</div>
        <nav id="mainNav" style="margin-top:10px"></nav>
      </div>
      <div style="text-align:right">
        <div style="font-weight:700" id="profileName">Nikita — Grade 3</div>
        <div class="small muted">Points: <span id="points">0</span> <span class="badge" id="avatar">🦉</span></div>
      </div>
    </header>

    <main>
      <!-- Container where pages show -->
      <div id="appContent"></div>
    </main>
  </div>

  <div id="confettiRoot" class="confetti" aria-hidden="true"></div>

  <script>
  /*****************************************************************
   * Nikita Study Buddy - Single-file HTML/CSS/JS app
   * - Saves state to localStorage
   * - Colorful interface, quiz + explanations, study guide -> flashcards
   *****************************************************************/

  /* ---------- Utilities & State ---------- */
  const STORAGE_KEY = 'nikita_nsb_v1';
  const defaultState = {
    name: 'Nikita',
    grade: 3,
    avatar: '🦉',
    points: 0,
    subjects: { Science:78, ELA:81, 'Social Studies':100, Math:94 },
    schedule: [],
    flashcards: [],   // {subject, front, back}
    quizLog: [],      // {title, score, date}
  };

  let state = loadState();

  function loadState(){
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      if(raw) return JSON.parse(raw);
    } catch(e){}
    return JSON.parse(JSON.stringify(defaultState));
  }
  function saveState(){ localStorage.setItem(STORAGE_KEY, JSON.stringify(state)); updateTop(); }

  function addPoints(n){
    state.points = (state.points || 0) + n;
    saveState();
    showConfetti();
  }

  function updateTop(){
    document.getElementById('points').textContent = state.points || 0;
    document.getElementById('avatar').textContent = state.avatar || '🦉';
    document.getElementById('profileName').textContent = `${state.name} — Grade ${state.grade}`;
  }

  /* ---------- Page Navigation ---------- */
  const pages = [
    {id:'dashboard', label:'Dashboard'},
    {id:'schedule', label:'Schedule'},
    {id:'flashcards', label:'Flashcards'},
    {id:'drill', label:'Multiplication Drill'},
    {id:'reading', label:'Reading Practice'},
    {id:'science', label:'Science Guide'},
    {id:'quiz', label:'Quiz Center'},
    {id:'studyguide', label:'Study Guide'},
    {id:'progress', label:'Progress Tracker'}
  ];

  function mountNav(){
    const nav = document.getElementById('mainNav');
    nav.innerHTML = '';
    pages.forEach(p=>{
      const btn = document.createElement('button');
      btn.className = 'btn';
      btn.textContent = p.label;
      btn.onclick = ()=>renderPage(p.id);
      nav.appendChild(btn);
    });
  }

  /* ---------- Confetti simple effect ---------- */
  function showConfetti(){
    const root = document.getElementById('confettiRoot');
    for(let i=0;i<18;i++){
      const el = document.createElement('div');
      const size = 6 + Math.round(Math.random()*12);
      el.style.position='absolute';
      el.style.left = Math.random()*100 + '%';
      el.style.top = '-10%';
      el.style.width = size+'px';
      el.style.height = size+'px';
      el.style.background = ['#ffd166','#06d6a0','#118ab2','#ef476f'][Math.floor(Math.random()*4)];
      el.style.opacity = Math.random()*0.9+0.3;
      el.style.transform = `rotate(${Math.random()*360}deg)`;
      el.style.borderRadius = Math.random()>.5 ? '2px':'50%';
      root.appendChild(el);
      (function(a){
        requestAnimationFrame(()=> a.style.transition='transform 1500ms linear, top 1500ms linear, opacity 1500ms');
        a.style.top = (40 + Math.random()*40) + '%';
        a.style.transform = `translateY(80vh) rotate(${Math.random()*720}deg)`;
        a.style.opacity = '0';
        setTimeout(()=> a.remove(), 1600);
      })(el);
    }
  }

  /* ---------- Render helpers ---------- */
  function el(html){ const d=document.createElement('div'); d.innerHTML=html.trim(); return d.firstChild; }
  function qsel(sel){ return document.querySelector(sel); }
  function clearContent(){ document.getElementById('appContent').innerHTML=''; }
  function toast(msg, time=1600){ const t=el(`<div class="card" style="position:fixed;right:18px;bottom:18px;border-radius:10px">${msg}</div>`); document.body.appendChild(t); setTimeout(()=>t.remove(),time); }

  /* ---------- Pages implementation ---------- */

  function renderPage(id){
    // highlight nav
    Array.from(document.querySelectorAll('#mainNav button')).forEach(b=>{
      b.classList.toggle('nav-active', b.textContent===pages.find(p=>p.id===id).label);
    });

    clearContent();
    if(id==='dashboard') return renderDashboard();
    if(id==='schedule') return renderSchedule();
    if(id==='flashcards') return renderFlashcards();
    if(id==='drill') return renderDrill();
    if(id==='reading') return renderReading();
    if(id==='science') return renderScienceBuilder();
    if(id==='quiz') return renderQuizCenter();
    if(id==='studyguide') return renderStudyGuide();
    if(id==='progress') return renderProgress();
  }

  /* ---------- DASHBOARD ---------- */
  function renderDashboard(){
    const container = el(`<div class="grid cols-2"></div>`);
    const left = el(`<div class="card panel-1"></div>`);
    const right = el(`<div class="card panel-2"></div>`);

    left.innerHTML = `
      <h2>📊 Dashboard</h2>
      <div class="small muted">Grades snapshot</div>
      <div style="margin-top:10px">
        ${Object.entries(state.subjects).map(([s,v])=>`<div style="display:flex;justify-content:space-between;margin:8px 0"><div>${s}</div><div style="font-weight:700">${v}%</div></div>`).join('')}
      </div>
      <div style="margin-top:10px" class="small muted">Progress toward 100%</div>
      <div style="margin-top:8px" class="prog"><i style="width:${Math.round((state.subjects.ELA||0))}%"></i></div>
    `;
    right.innerHTML = `
      <h3>Upcoming Study</h3>
      <div class="small muted">Schedule (first 4)</div>
      ${state.schedule.length? `<ul>${state.schedule.slice(0,4).map(s=>`<li><strong>${s.subject}</strong> — ${s.duration} mins on ${s.day}</li>`).join('')}</ul>` : '<div class="small muted">No scheduled sessions yet</div>'}
      <div style="margin-top:8px"><button class="btn primary" id="startDaily">Start Daily Challenge</button></div>
      <div style="margin-top:12px">
        <h4 style="margin:0">Recent quizzes</h4>
        ${state.quizLog.length? `<ul>${state.quizLog.slice(0,5).map(q=>`<li>${q.title} — ${q.score}%</li>`).join('')}</ul>` : '<div class="small muted">No quizzes taken yet</div>'}
      </div>
    `;

    container.appendChild(left);
    container.appendChild(right);
    document.getElementById('appContent').appendChild(container);

    document.getElementById('startDaily').onclick = ()=> {
      // start a 3-question mixed quiz
      renderQuizCenter({daily:true});
    };
  }

  /* ---------- SCHEDULE ---------- */
  function renderSchedule(){
    const wrap = el(`<div class="card panel-3"></div>`);
    wrap.innerHTML = `
      <h2>🗓️ Schedule Builder</h2>
      <div class="small muted">Create simple weekly study items</div>
      <div style="margin-top:8px" class="row">
        <div class="col"><label>Day<select id="daySel"><option>Monday</option><option>Tuesday</option><option>Wednesday</option><option>Thursday</option><option>Friday</option><option>Saturday</option><option>Sunday</option></select></label></div>
        <div class="col"><label>Subject<select id="subSel"><option>ELA</option><option>Math</option><option>Science</option><option>Social Studies</option><option>Reading Practice</option></select></label></div>
        <div style="width:110px"><label>Duration (min)<input type="number" id="durSel" value="20" min="5" /></label></div>
      </div>
      <div style="margin-top:8px"><button class="btn primary" id="addSched">Add to Schedule</button></div>
      <div id="schedList" style="margin-top:12px"></div>
    `;

    document.getElementById('appContent').appendChild(wrap);
    function redraw(){
      const elList = document.getElementById('schedList');
      if(!state.schedule.length) elList.innerHTML = '<div class="small muted">No items yet</div>';
      else elList.innerHTML = `<ul>${state.schedule.map((s,i)=>`<li>${s.day} — <strong>${s.subject}</strong> (${s.duration} min) <button data-i="${i}" class="small-btn">Remove</button></li>`).join('')}</ul>`;
      Array.from(elList.querySelectorAll('button')).forEach(b=>{
        b.onclick = ()=>{ state.schedule.splice(Number(b.dataset.i),1); saveState(); renderSchedule(); };
      });
    }
    document.getElementById('addSched').onclick = ()=>{
      const day = document.getElementById('daySel').value;
      const subject = document.getElementById('subSel').value;
      const duration = Number(document.getElementById('durSel').value) || 20;
      state.schedule.unshift({day,subject,duration});
      saveState();
      toast('Added to schedule');
      renderSchedule();
    };
    redraw();
  }

  /* ---------- FLASHCARDS ---------- */
  function renderFlashcards(){
    const wrap = el(`<div class="card panel-4"></div>`);
    wrap.innerHTML = `
      <h2>🃏 Flashcards</h2>
      <div class="small muted">Create quick Q/A cards and study</div>
      <div style="margin-top:8px" class="row">
        <div class="col"><label>Subject<input id="fcSub" type="text" value="Science" /></label></div>
        <div class="col"><label>Front (question)<input id="fcFront" type="text" /></label></div>
      </div>
      <label>Back (answer)<input id="fcBack" type="text" /></label>
      <div style="margin-top:8px"><button class="btn primary" id="addCard">Add Card</button> <button class="btn" id="clearCards">Clear All</button></div>
      <div id="fcStudy" style="margin-top:12px"></div>
    `;
    document.getElementById('appContent').appendChild(wrap);

    document.getElementById('addCard').onclick = ()=>{
      const sub = document.getElementById('fcSub').value || 'General';
      const front = document.getElementById('fcFront').value.trim();
      const back = document.getElementById('fcBack').value.trim();
      if(!front) return toast('Add a question (front)');
      state.flashcards.unshift({subject:sub,front,back});
      saveState();
      document.getElementById('fcFront').value='';document.getElementById('fcBack').value='';
      renderFlashcards();
    };
    document.getElementById('clearCards').onclick = ()=>{
      if(!confirm('Clear all flashcards?')) return;
      state.flashcards=[];saveState();renderFlashcards();
    };

    // study mode
    const study = el(`<div class="card" style="margin-top:10px"><h4 style="margin:0">Study Mode</h4><div id="studyArea"></div></div>`);
    wrap.appendChild(study);
    const studyArea = study.querySelector('#studyArea');
    if(!state.flashcards.length){
      studyArea.innerHTML = '<div class="small muted" style="margin-top:8px">No flashcards yet. Add cards to begin.</div>';
    } else {
      let idx = 0, show=false;
      function draw(){
        const c = state.flashcards[idx%state.flashcards.length];
        studyArea.innerHTML = `
          <div class="flashcard" style="background:linear-gradient(90deg,#fff,#f8fafc)">
            <div style="font-size:13px;color:var(--muted)">${c.subject}</div>
            <div style="font-size:18px;margin-top:8px">${c.front}</div>
            <div id="ans" style="margin-top:10px;display:${show?'block':'none'}">${c.back}</div>
            <div style="margin-top:10px" class="row">
              <button class="small-btn" id="toggleAns">${show?'Hide':'Show'} Answer</button>
              <button class="small-btn" id="nextCard">Next</button>
              <button class="small-btn" id="correct">Got it</button>
            </div>
          </div>
        `;
        studyArea.querySelector('#toggleAns').onclick = ()=>{ show=!show; draw(); };
        studyArea.querySelector('#nextCard').onclick = ()=>{ idx++; show=false; draw(); };
        studyArea.querySelector('#correct').onclick = ()=>{ addPoints(5); toast('+5 points'); idx++; show=false; draw(); saveState(); };
      }
      draw();
    }
  }

  /* ---------- MULTIPLICATION DRILL ---------- */
  function renderDrill(){
    const wrap = el(`<div class="card panel-2"></div>`);
    wrap.innerHTML = `
      <h2>🏁 Multiplication Drill</h2>
      <div class="small muted">Practice facts 0–12. Earn points for correct answers.</div>
      <div style="margin-top:10px" class="row">
        <div class="col"><label>Mode<select id="modeSel"><option>Mixed</option><option>Times Table (pick one)</option></select></label></div>
        <div style="width:120px"><label>Duration (questions)<input id="qCount" type="number" value="10" min="3" /></label></div>
      </div>
      <div id="drillArea" style="margin-top:12px"></div>
    `;
    document.getElementById('appContent').appendChild(wrap);

    const area = document.getElementById('drillArea');
    function start(){
      const mode = document.getElementById('modeSel').value;
      const qTotal = Number(document.getElementById('qCount').value) || 10;
      let asked=0, correct=0;
      let fixedTable = null;
      if(mode!=='Mixed') fixedTable = Number(prompt('Which times table (0–12)?','7')) || 7;
      area.innerHTML = `<div class="small muted">Question 1 of ${qTotal}</div><div style="margin-top:8px;font-size:22px;font-weight:700" id="qbox"></div>
        <div style="margin-top:8px" class="row"><input id="ans" type="number" placeholder="Answer" /><button class="btn primary" id="subbtn">Submit</button></div>
        <div style="margin-top:10px" id="scoreBox">Score: 0 / 0</div>
      `;
      const qbox = area.querySelector('#qbox'), ansIn = area.querySelector('#ans'), subbtn = area.querySelector('#subbtn'), scoreBox = area.querySelector('#scoreBox');

      function nextQ(){
        asked++;
        const a = fixedTable !== null ? fixedTable : Math.floor(Math.random()*13);
        const b = Math.floor(Math.random()*13);
        qbox.textContent = `${a} × ${b} = ?`;
        qbox.dataset.correct = a*b;
        area.querySelector('.small').textContent = `Question ${asked} of ${qTotal}`;
        ansIn.value = ''; ansIn.focus();
      }
      subbtn.onclick = ()=>{
        const got = Number(ansIn.value);
        const correctAns = Number(qbox.dataset.correct);
        if(got===correctAns){ correct++; addPoints(3); toast('+3 pts'); }
        scoreBox.textContent = `Score: ${correct} / ${asked}`;
        if(asked < qTotal) nextQ(); else {
          area.innerHTML = `<div class="center"><h3>Done!</h3><div>Score: ${correct} / ${qTotal}</div><div style="margin-top:8px"><button class="btn" id="ret">Back</button></div></div>`;
          state.quizLog.unshift({title:`Multiplication drill ${qTotal}q`, score: Math.round((correct/qTotal)*100), date: new Date().toISOString()});
          saveState();
          document.getElementById('ret').onclick = ()=>renderPage('drill');
        }
      };
      nextQ();
    }

    area.innerHTML = `<div><button class="btn primary" id="startDrill">Start Drill</button></div>`;
    document.getElementById('startDrill').onclick = start;
  }

  /* ---------- READING PRACTICE ---------- */
  const readingPassages = [
    {
      title:'Short passage: Opinion — Dogs',
      text: "Many people think dogs are the best pets because they are loyal and like to play. The author believes dogs make great companions.",
      questions: [
        {q:'What is the author’s opinion?', a:'Dogs are the best pets'},
        {q:'Give one reason the author uses.', a:'They are loyal'}
      ]
    },
    {
      title:'Short passage: The Pond',
      text: "The pond had frogs and lilies. Many animals use the pond for food and shelter. The author says ponds are important for wildlife.",
      questions:[
        {q:'What is the main idea?', a:'Ponds are important for wildlife'},
        {q:'Name one animal mentioned.', a:'Frogs'}
      ]
    }
  ];

  function renderReading(){
    const idx = 0;
    const wrap = el(`<div class="card panel-3"></div>`);
    wrap.innerHTML = `<h2>📚 Reading Practice</h2><div class="small muted">Work on main idea and author’s opinion. Type answers and submit for quick feedback.</div>
      <div id="readArea" style="margin-top:10px"></div>
    `;
    document.getElementById('appContent').appendChild(wrap);

    let cur = 0;
    function draw(){
      const p = readingPassages[cur%readingPassages.length];
      const area = document.getElementById('readArea');
      area.innerHTML = `<div style="padding:12px;border-radius:10px;background:#fff"><div style="font-weight:700">${p.title}</div><p style="margin-top:8px">${p.text}</p></div>
        <div style="margin-top:8px">${p.questions.map((qq,i)=>`<div><div class="small">${qq.q}</div><input data-i="${i}" class="qans" /></div>`).join('')}</div>
        <div style="margin-top:10px"><button class="btn primary" id="submitRead">Submit</button> <button class="btn" id="nextRead">Next Passage</button></div>
      `;
      document.getElementById('submitRead').onclick = ()=>{
        let correct=0;
        const inputs = Array.from(document.querySelectorAll('.qans'));
        inputs.forEach(inp=>{
          const i = Number(inp.dataset.i);
          const val = (inp.value||'').toLowerCase();
          if(val.includes(readingPassages[cur].questions[i].a.toLowerCase())) correct++;
        });
        const percent = Math.round((correct/readingPassages[cur].questions.length)*100);
        state.quizLog.unshift({title:`Reading: ${p.title}`, score: percent, date: new Date().toISOString()});
        saveState();
        if(percent>=70){ addPoints(6); toast('+6 points'); } else { toast('Keep trying!'); }
        alert(`You scored ${percent}%\nExplanation:\n` + p.questions.map(q=>`- ${q.q}\n  Answer hint: ${q.a}`).join('\n\n'));
      };
      document.getElementById('nextRead').onclick = ()=>{ cur++; draw(); };
    }
    draw();
  }

  /* ---------- SCIENCE GUIDE BUILDER (Quick Georgia habitats) ---------- */
  const habitats = [
    {name:'Coastal', features:'Beaches, marshes, estuaries', animals:'Sea turtles, shorebirds'},
    {name:'Mountain', features:'Higher elevation, cooler', animals:'Black bears, salamanders'},
    {name:'Piedmont', features:'Rolling hills, mixed forest', animals:'Deer, squirrels'},
    {name:'Wetlands/Swamp', features:'Water-saturated soil', animals:'Alligators, wading birds'}
  ];

  function renderScienceBuilder(){
    const wrap = el(`<div class="card panel-4"></div>`);
    wrap.innerHTML = `<h2>🌿 Science Quick Guide</h2><div class="small muted">Georgia habitats — preview and export as flashcards</div>
      <div style="margin-top:10px" id="habGrid" class="row"></div>
      <div style="margin-top:10px"><button class="btn primary" id="exportHab">Export all as Flashcards</button></div>
    `;
    document.getElementById('appContent').appendChild(wrap);
    const grid = document.getElementById('habGrid');
    grid.innerHTML = habitats.map(h=>`<div class="card flashcard" style="flex:1;margin-right:8px;min-width:170px"><div style="font-weight:700">${h.name}</div><div class="small muted" style="margin-top:6px">${h.features}</div><div style="margin-top:6px" class="small muted">Animals: ${h.animals}</div></div>`).join('');
    document.getElementById('exportHab').onclick = ()=>{
      const cards = habitats.map(h=>({subject:'Science', front:h.name + ' — Features', back: `${h.features}; Animals: ${h.animals}`}));
      state.flashcards = [...cards, ...state.flashcards];
      saveState();
      toast('Exported habitats as flashcards');
    };
  }

  /* ---------- QUIZ CENTER (generates questions & explanations) ---------- */

  // Small in-browser question generator using templates appropriate for grade 3
  function makeQuizQuestion(subject){
    if(subject==='Science'){
      const h = habitats[Math.floor(Math.random()*habitats.length)];
      return {
        q: `Which habitat in Georgia is home to ${h.animals.split(',')[0]}?`,
        options: habitats.map(x=>x.name).sort(()=>Math.random()-0.5),
        answer: h.name,
        explanation: `${h.animals.split(',')[0]} live in ${h.name} habitats. ${h.features}.`
      };
    }
    if(subject==='Math'){
      const a = Math.floor(Math.random()*13);
      const b = Math.floor(Math.random()*13);
      return {
        q: `What is ${a} × ${b}?`,
        options: [(a*b), (a*b + Math.floor(Math.random()*5)+1), Math.max(0,a*b - (Math.floor(Math.random()*6)+1))].sort(()=>Math.random()-0.5),
        answer: a*b,
        explanation: `${a} groups of ${b} equals ${a*b}.`
      };
    }
    // ELA - author's opinion / main idea style
    if(subject==='ELA'){
      const texts = [
        {t:'The author says apples are the best snack because they are crunchy and sweet.', opinion:'Apples are the best snack', reason:'they are crunchy and sweet'},
        {t:'Cars are loud and fast; the author prefers bicycles because they are quiet and healthy.', opinion:'Bicycles are better than cars', reason:'they are quiet and healthy'}
      ];
      const pick = texts[Math.floor(Math.random()*texts.length)];
      return {
        q: `Read: "${pick.t}" What is the author's opinion?`,
        options: [pick.opinion, 'Apples are crunchy', 'Cars are loud'].sort(()=>Math.random()-0.5),
        answer: pick.opinion,
        explanation: `The opinion is what the author believes: "${pick.opinion}". The reason: ${pick.reason}.`
      };
    }
    // fallback
    return {q:'Which is correct?', options:['A','B','C'], answer:'A', explanation:'Because A is correct.'};
  }

  function renderQuizCenter(opts={}){
    const wrap = el(`<div class="card panel-1"></div>`);
    wrap.innerHTML = `<h2>📝 Quiz Center</h2><div class="small muted">Pick a subject and try a 1–5 question quiz. Explanations shown after each answer.</div>
      <div style="margin-top:8px" class="row">
        <div class="col"><label>Subject<select id="quizSub"><option>Science</option><option>Math</option><option>ELA</option></select></label></div>
        <div style="width:150px"><label>Questions<select id="quizCount"><option>1</option><option selected>3</option><option>5</option></select></label></div>
      </div>
      <div style="margin-top:10px"><button class="btn primary" id="startQuiz">Start Quiz</button></div>
      <div id="quizArea" style="margin-top:12px"></div>
    `;
    document.getElementById('appContent').appendChild(wrap);

    if(opts.daily){
      // if daily, preselect mixed subjects and 3 questions
      document.getElementById('quizSub').value = 'Science';
      document.getElementById('quizCount').value = '3';
      startQuiz();
    }

    document.getElementById('startQuiz').onclick = startQuiz;

    function startQuiz(){
      const sub = document.getElementById('quizSub').value;
      const qcount = Number(document.getElementById('quizCount').value);
      const questions = Array.from({length:qcount}).map(()=>makeQuizQuestion(sub));
      let idx = 0, correct = 0;
      const area = document.getElementById('quizArea');

      function showQ(){
        const cur = questions[idx];
        area.innerHTML = `<div style="padding:12px;border-radius:10px;background:#fff"><div style="font-weight:700">Q${idx+1}: ${cur.q}</div>
          <div style="margin-top:8px">${cur.options.map((o,i)=>`<button class="btn" data-i="${i}" style="display:block;margin:6px 0">${o}</button>`).join('')}</div>
          <div id="explain" style="margin-top:8px;display:none;background:#f8fafc;padding:8px;border-radius:8px"></div>
          <div style="margin-top:8px"><button class="btn" id="nextQ" disabled>Next</button></div>
          </div>`;
        // attach handlers
        area.querySelectorAll('button[data-i]').forEach(b=>{
          b.onclick = ()=>{
            const chosen = b.textContent;
            const correctAns = String(cur.answer);
            const gotIt = String(chosen) === correctAns;
            if(gotIt){ correct++; addPoints(8); toast('+8 pts'); }
            // show explanation
            const expl = area.querySelector('#explain');
            expl.style.display='block';
            expl.innerHTML = `<strong>${gotIt? '✅ Correct' : '❌ That was close'}</strong><div style="margin-top:6px">${cur.explanation}</div>`;
            area.querySelector('#nextQ').disabled = false;
            // log result
            state.quizLog.unshift({title:`Quiz: ${cur.q}`, score: gotIt?100:0, date:new Date().toISOString()});
            saveState();
          };
        });
        area.querySelector('#nextQ').onclick = ()=>{
          idx++;
          if(idx < questions.length) showQ(); else finish();
        };
      }

      function finish(){
        area.innerHTML = `<div class="center"><h3>All done!</h3><div>Score: ${correct} / ${questions.length}</div><div style="margin-top:8px"><button class="btn" id="doneBtn">Back</button></div></div>`;
        document.getElementById('doneBtn').onclick = ()=>renderPage('quiz');
      }
      showQ();
    }
  }

  /* ---------- STUDY GUIDE GENERATOR ---------- */
  function renderStudyGuide(){
    const wrap = el(`<div class="card panel-2"></div>`);
    wrap.innerHTML = `
      <h2>📘 Study Guide Generator</h2>
      <div class="small muted">Choose a subject and get a one-page study guide you can export as flashcards.</div>
      <div style="margin-top:8px" class="row">
        <div class="col"><label>Subject<select id="sgSub"><option>Science</option><option>ELA</option><option>Math</option></select></label></div>
        <div style="width:200px"><label>Topic (optional)<input id="sgTopic" placeholder="e.g., Georgia habitats" /></label></div>
      </div>
      <div style="margin-top:8px"><button class="btn primary" id="genGuide">Generate Guide</button></div>
      <div id="guideArea" style="margin-top:10px"></div>
    `;
    document.getElementById('appContent').appendChild(wrap);
    document.getElementById('genGuide').onclick = ()=>{
      const s = document.getElementById('sgSub').value;
      const t = document.getElementById('sgTopic').value.trim();
      generateGuide(s,t);
    };

    function generateGuide(subject, topic){
      let guide;
      if(subject==='Science'){
        guide = { title: topic||'Georgia Habitats Study Guide', bullets: habitats.map(h=>`${h.name}: ${h.features} — Animals: ${h.animals}`) };
      } else if(subject==='ELA'){
        guide = { title: topic||'Opinion Writing Guide', bullets: [
          'State your opinion clearly in the first sentence.',
          'Give at least two reasons with examples.',
          'Use words like because, for example to connect ideas.',
          'End with a short conclusion restating your opinion.'
        ]};
      } else {
        guide = { title: topic||`${subject} Essentials`, bullets:[ 'Key facts appear here for review.' ]};
      }
      const area = document.getElementById('guideArea');
      area.innerHTML = `<div class="card"><h3 style="margin:0">${guide.title}</h3>
        <ul style="margin-top:8px">${guide.bullets.map(b=>`<li>${b}</li>`).join('')}</ul>
        <div style="margin-top:8px"><button class="btn primary" id="exportGuide">Export as Flashcards</button></div></div>`;
      document.getElementById('exportGuide').onclick = ()=>{
        const cards = guide.bullets.map(b=>({subject, front: b.split(':')[0], back:b}));
        state.flashcards = [...cards, ...state.flashcards];
        saveState();
        toast('Guide exported as flashcards');
      }
    }
  }

  /* ---------- PROGRESS ---------- */
  function renderProgress(){
    const wrap = el(`<div class="card panel-3"></div>`);
    wrap.innerHTML = `<h2>📈 Progress Tracker</h2>
      <div class="small muted">Points & recent quiz history</div>
      <div style="margin-top:8px"><strong>Points:</strong> ${state.points}</div>
      <div style="margin-top:10px"><h4 style="margin:0">Recent Quizzes</h4>
        ${state.quizLog.length? `<ul>${state.quizLog.slice(0,10).map(q=>`<li>${q.title} — ${q.score}%</li>`).join('')}</ul>` : '<div class="small muted">No quizzes taken yet</div>'}
      </div>
    `;
    document.getElementById('appContent').appendChild(wrap);
  }

  /* ---------- INIT ---------- */
  mountNav();
  updateTop();
  renderPage('dashboard');

  /* ---------- Helpful tip: allow clearing local data for testing ---------- */
  window.nsb = {
    state,
    save: saveState,
    reset: ()=>{ if(confirm('Reset all NikitaStudyBuddy data?')){ localStorage.removeItem(STORAGE_KEY); state = loadState(); updateTop(); renderPage('dashboard'); } }
  };

  // ensure top updates when user returns
  window.addEventListener('storage', ()=>{ state = loadState(); updateTop(); });

  </script>
</body>
</html>
