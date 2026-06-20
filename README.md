<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no"/>
<title>Wrapped · Alan & Dani</title>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:ital,opsz,wght@0,9..144,400;0,9..144,600;0,9..144,900;1,9..144,400&family=Space+Grotesk:wght@400;500;600;700&display=swap" rel="stylesheet"/>
<style>
*{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent;}
html,body{height:100%;overflow:hidden;}
body{font-family:"Space Grotesk",sans-serif;background:#0a0612;color:#fff;}

.stage{position:relative;height:100dvh;width:100vw;overflow:hidden;}

/* progress bars */
.progress{position:fixed;top:14px;left:50%;transform:translateX(-50%);display:flex;gap:4px;z-index:60;width:calc(100% - 40px);max-width:420px;}
.pbar{flex:1;height:3px;background:rgba(255,255,255,.25);border-radius:99px;overflow:hidden;}
.pbar-fill{height:100%;width:0;background:#fff;border-radius:99px;}
.pbar.done .pbar-fill{width:100%;}
.pbar.active .pbar-fill{animation:fill var(--dur,7s) linear forwards;}
@keyframes fill{from{width:0}to{width:100%}}

/* slides */
.slide{position:absolute;inset:0;display:none;flex-direction:column;align-items:center;justify-content:center;padding:80px 32px 100px;text-align:center;}
.slide.on{display:flex;}
.slide.on>*{opacity:0;animation:rise .7s cubic-bezier(.2,.7,.2,1) forwards;}
.slide.on>*:nth-child(1){animation-delay:.1s}
.slide.on>*:nth-child(2){animation-delay:.25s}
.slide.on>*:nth-child(3){animation-delay:.4s}
.slide.on>*:nth-child(4){animation-delay:.55s}
.slide.on>*:nth-child(5){animation-delay:.7s}
@keyframes rise{from{opacity:0;transform:translateY(28px)}to{opacity:1;transform:none}}

.eyebrow{font-size:13px;letter-spacing:.28em;text-transform:uppercase;font-weight:600;opacity:.65;margin-bottom:26px;}
.mega{font-family:"Fraunces",serif;font-weight:900;line-height:.92;letter-spacing:-.02em;}
.serif{font-family:"Fraunces",serif;}
.label{font-size:clamp(1.05rem,4.5vw,1.45rem);font-weight:500;margin-top:14px;max-width:340px;line-height:1.35;}
.detail{font-size:14px;opacity:.7;margin-top:18px;max-width:300px;line-height:1.6;font-weight:400;}
.glyph{font-size:clamp(56px,16vw,84px);margin-bottom:8px;line-height:1;}

/* tap zones */
.taps{position:fixed;inset:0;z-index:40;display:flex;}
.tap{flex:1;}
.tap.back{flex:0 0 28%;}

/* nav hint */
.hint{position:fixed;bottom:30px;left:50%;transform:translateX(-50%);font-size:12px;opacity:.45;z-index:45;letter-spacing:.05em;}
.replay{position:fixed;bottom:26px;left:50%;transform:translateX(-50%);z-index:55;background:rgba(255,255,255,.12);border:1px solid rgba(255,255,255,.25);color:#fff;border-radius:99px;padding:12px 30px;font-family:inherit;font-size:14px;font-weight:600;cursor:pointer;display:none;backdrop-filter:blur(8px);}

/* decorative floating particles */
.particles{position:absolute;inset:0;overflow:hidden;pointer-events:none;z-index:1;}
.particle{position:absolute;opacity:0;animation:floaty linear infinite;}
@keyframes floaty{0%{transform:translateY(20px) rotate(0);opacity:0}10%{opacity:.5}90%{opacity:.5}100%{transform:translateY(-120vh) rotate(60deg);opacity:0}}

/* count-up number wrapper */
.counter{font-variant-numeric:tabular-nums;}

/* intro pulse */
.tap-pulse{position:fixed;bottom:80px;left:50%;transform:translateX(-50%);z-index:45;font-size:13px;opacity:.6;animation:pulseHint 1.8s ease-in-out infinite;}
@keyframes pulseHint{0%,100%{opacity:.3}50%{opacity:.8}}

.gradient-text{background:linear-gradient(120deg,#fff,#ffd6ec 40%,#c9b8ff);-webkit-background-clip:text;-webkit-text-fill-color:transparent;background-clip:text;}
.chip{display:inline-block;background:rgba(255,255,255,.14);border:1px solid rgba(255,255,255,.2);border-radius:99px;padding:7px 16px;font-size:13px;font-weight:600;margin-top:20px;backdrop-filter:blur(6px);}

.quote-bubble{background:rgba(255,255,255,.12);border:1px solid rgba(255,255,255,.18);border-radius:20px 20px 20px 6px;padding:18px 22px;font-size:18px;line-height:1.5;max-width:320px;margin-top:10px;backdrop-filter:blur(6px);font-weight:500;}
</style>
</head>
<body>
<div class="stage" id="stage">
  <div class="particles" id="particles"></div>
  <div class="progress" id="progress"></div>
  <div id="slides"></div>
  <div class="taps"><div class="tap back" onclick="prev()"></div><div class="tap" onclick="next()"></div></div>
  <div class="tap-pulse" id="pulse">toca para avanzar →</div>
  <button class="replay" id="replay" onclick="restart()">Ver otra vez ↺</button>
</div>

<script>
// fresh facts — deliberately different from the dashboard
const SLIDES = [
  {
    bg:'radial-gradient(circle at 50% 30%,#3b1d5e,#0a0612)',
    eyebrow:'Su historia · 2024–2026',
    glyph:'✦',
    title:'<div class="mega gradient-text" style="font-size:clamp(2.6rem,11vw,4.5rem)">Alan<br>&amp; Dani</div>',
    label:'573 días. 33,627 mensajes.<br>Esto es lo que no habías visto.',
    detail:'Toca la pantalla para empezar →'
  },
  {
    bg:'radial-gradient(circle at 30% 70%,#1d3a5e,#0a0612)',
    eyebrow:'Si lo imprimieran…',
    glyph:'📖',
    title:'<div class="mega counter" id="c-words" style="font-size:clamp(3.5rem,16vw,6.5rem)">0</div>',
    label:'palabras escritas entre los dos',
    detail:'Son <b>445 páginas</b> — un libro entero. Tú escribiste 77,751 y Dani 55,663.'
  },
  {
    bg:'radial-gradient(circle at 70% 30%,#5e1d3a,#0a0612)',
    eyebrow:'Tiempo real juntos',
    glyph:'⏱️',
    title:'<div class="mega counter" id="c-hours" style="font-size:clamp(3.5rem,16vw,6.5rem)">0</div>',
    label:'horas chateando activamente',
    detail:'Equivale a <b>9 días enteros</b> sin parar, escribiéndose de verdad. Ni dormir.'
  },
  {
    bg:'radial-gradient(circle at 50% 60%,#1d5e4a,#0a0612)',
    eyebrow:'Las risas',
    glyph:'😂',
    title:'<div class="mega counter" id="c-jaja" style="font-size:clamp(3.5rem,16vw,6.5rem)">0</div>',
    label:'veces que se rieron juntos',
    detail:'Contando cada "jaja" y "jeje". <b>11 mil carcajadas</b>. Eso es una relación que se divierte.'
  },
  {
    bg:'radial-gradient(circle at 30% 40%,#4a1d5e,#0a0612)',
    eyebrow:'Cómo empezó todo',
    glyph:'🌱',
    title:'<div class="quote-bubble serif" style="font-style:italic">"Holi"</div>',
    label:'El primer mensaje. Lo mandó Dani.',
    detail:'20 de noviembre de 2024, 11:35 de la noche. Cuatro letras que empezaron 573 días.'
  },
  {
    bg:'radial-gradient(circle at 60% 50%,#5e3a1d,#0a0612)',
    eyebrow:'El mes más intenso',
    glyph:'🔥',
    title:'<div class="mega serif" style="font-size:clamp(2.4rem,10vw,3.8rem);text-transform:capitalize">Diciembre<br>2024</div>',
    label:'6,340 mensajes en un solo mes',
    detail:'El mes que más se hablaron. Las fiestas los tuvieron pegados al teléfono.'
  },
  {
    bg:'radial-gradient(circle at 40% 70%,#1d2d5e,#0a0612)',
    eyebrow:'Criaturas de la noche',
    glyph:'🌙',
    title:'<div class="mega counter" id="c-madru" style="font-size:clamp(3.5rem,16vw,6.5rem)">0</div><div style="font-size:2rem;font-weight:700;margin-top:-8px">%</div>',
    label:'de los mensajes fueron de madrugada',
    detail:'Entre medianoche y las 6 AM. Casi 4 de cada 10 mensajes. Su hora pico: las <b>2 AM</b>.'
  },
  {
    bg:'radial-gradient(circle at 50% 30%,#5e1d2d,#0a0612)',
    eyebrow:'El mensaje más largo',
    glyph:'💌',
    title:'<div class="quote-bubble serif" style="font-style:italic;font-size:15px;line-height:1.55">"…en estos tres días de tormenta, tu apoyo incondicional fue la única luz que hemos tenido."</div>',
    label:'Lo escribiste tú, Alan',
    detail:'350 caracteres en un momento difícil. A veces los mensajes más largos son los que más importan.'
  },
  {
    bg:'radial-gradient(circle at 30% 50%,#3a1d5e,#0a0612)',
    eyebrow:'Después de 318 días…',
    glyph:'🕊️',
    title:'<div class="mega serif" style="font-size:clamp(2.8rem,12vw,4.5rem)">Volvieron</div>',
    label:'Dani rompió el silencio',
    detail:'El 23/01/2026, tras casi un año, ella escribió primero — con un mensaje de apoyo. Algunas conexiones no se apagan.'
  },
  {
    bg:'radial-gradient(circle at 50% 40%,#5e1d4a,#2d0a1e)',
    eyebrow:'Y al final…',
    glyph:'💕',
    title:'<div class="mega gradient-text" style="font-size:clamp(3rem,13vw,5rem)">97%</div>',
    label:'de los días juntos fueron buenos',
    detail:'De cada día que hablaron, casi todos fueron equilibrados, cálidos, recíprocos. Eso es lo más raro de encontrar.',
    chip:'Feliz 2026, ustedes dos ✦'
  },
];

const COUNTERS = {
  'c-words': 133414,
  'c-hours': 224,
  'c-jaja': 11308,
  'c-madru': 39,
};

let cur = 0, timer = null;
const slidesEl = document.getElementById('slides');
const progEl = document.getElementById('progress');

// build slides
SLIDES.forEach((s,i)=>{
  const div = document.createElement('div');
  div.className = 'slide';
  div.style.background = s.bg;
  div.innerHTML = `
    <div class="eyebrow">${s.eyebrow}</div>
    ${s.glyph?`<div class="glyph">${s.glyph}</div>`:''}
    ${s.title}
    <div class="label">${s.label}</div>
    <div class="detail">${s.detail}</div>
    ${s.chip?`<div class="chip">${s.chip}</div>`:''}
  `;
  slidesEl.appendChild(div);
  const bar = document.createElement('div');
  bar.className='pbar';
  bar.innerHTML='<div class="pbar-fill"></div>';
  progEl.appendChild(bar);
});

function animateCounter(id, target){
  const el = document.getElementById(id);
  if(!el) return;
  const dur = 1400, start = performance.now();
  function step(now){
    const p = Math.min((now-start)/dur, 1);
    const eased = 1-Math.pow(1-p,3);
    el.textContent = Math.round(target*eased).toLocaleString('es');
    if(p<1) requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}

function show(i){
  if(i<0) i=0;
  if(i>=SLIDES.length){ finish(); return; }
  cur=i;
  document.querySelectorAll('.slide').forEach((s,idx)=>s.classList.toggle('on',idx===i));
  document.querySelectorAll('.pbar').forEach((b,idx)=>{
    b.classList.remove('active','done');
    const fill = b.querySelector('.pbar-fill');
    fill.style.animation='none'; void fill.offsetWidth;
    if(idx<i) b.classList.add('done');
    else if(idx===i){ b.style.setProperty('--dur','7s'); b.classList.add('active'); }
  });
  // trigger counters on this slide
  document.querySelector('.slide.on')?.querySelectorAll('.counter').forEach(c=>{
    if(COUNTERS[c.id]!==undefined) animateCounter(c.id, COUNTERS[c.id]);
  });
  document.getElementById('pulse').style.display = i===0 ? 'block':'none';
  clearTimeout(timer);
  timer = setTimeout(next, 7000);
}
function next(){ show(cur+1); }
function prev(){ clearTimeout(timer); show(cur-1); }
function finish(){
  clearTimeout(timer);
  document.querySelectorAll('.pbar').forEach(b=>b.classList.add('done'));
  document.getElementById('replay').style.display='block';
  document.getElementById('pulse').style.display='none';
  burst();
}
function restart(){
  document.getElementById('replay').style.display='none';
  show(0);
}

// floating particles
const glyphs=['✦','♥','✧','·','✦'];
const pc=document.getElementById('particles');
function spawnParticle(){
  const p=document.createElement('div');
  p.className='particle';
  p.textContent=glyphs[Math.floor(Math.random()*glyphs.length)];
  p.style.left=Math.random()*100+'%';
  p.style.bottom='-30px';
  p.style.fontSize=(10+Math.random()*18)+'px';
  p.style.color=['#ffd6ec','#c9b8ff','#ffffff'][Math.floor(Math.random()*3)];
  const dur=8+Math.random()*7;
  p.style.animationDuration=dur+'s';
  pc.appendChild(p);
  setTimeout(()=>p.remove(),dur*1000);
}
setInterval(spawnParticle,700);
for(let i=0;i<5;i++) setTimeout(spawnParticle,i*400);

function burst(){
  const colors=['#ffd6ec','#c9b8ff','#f472b6','#a78bfa','#fff'];
  for(let i=0;i<80;i++){
    const c=document.createElement('div');
    c.style.cssText=`position:fixed;width:9px;height:9px;z-index:100;pointer-events:none;left:${Math.random()*100}vw;top:-10px;background:${colors[Math.floor(Math.random()*colors.length)]};border-radius:${Math.random()>.5?'50%':'2px'}`;
    document.body.appendChild(c);
    const dur=2.5+Math.random()*2;
    c.animate([{transform:'translateY(0) rotate(0)',opacity:1},{transform:`translateY(105vh) rotate(${Math.random()*720}deg)`,opacity:0}],{duration:dur*1000,easing:'cubic-bezier(.4,0,.6,1)'});
    setTimeout(()=>c.remove(),dur*1000);
  }
}

// keyboard
document.addEventListener('keydown',e=>{
  if(e.key==='ArrowRight'||e.key===' ') next();
  if(e.key==='ArrowLeft') prev();
});

show(0);
</script>
</body>
</html>
