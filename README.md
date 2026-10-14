<!doctype html>
<html lang="ar" dir="rtl">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#17251d">
<title>شطرنج — ضد الذكاء الاصطناعي</title>
<style>
:root{--bg:#101813;--panel:#1a2820;--text:#f1f5ef;--muted:#b6c3b8;--light:#e8d8b7;--dark:#6f8d68;--accent:#d9bd68}
*{box-sizing:border-box}body{margin:0;background:radial-gradient(circle at top,#26382b,var(--bg) 60%);color:var(--text);font-family:system-ui,-apple-system,"Segoe UI",sans-serif;min-height:100vh;padding:16px}
main{max-width:560px;margin:auto}.top{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:14px}h1{font-size:1.45rem;margin:0}.sub{color:var(--muted);font-size:.9rem;margin-top:4px}
.pill{background:#304535;border:1px solid #4a654f;border-radius:999px;padding:7px 11px;font-size:.82rem}
.card{background:#1a2820eF;border:1px solid #344638;border-radius:18px;padding:12px;box-shadow:0 12px 30px #0003}
.players{display:flex;justify-content:space-between;align-items:center;margin:0 2px 10px;font-size:.9rem}.players span{color:var(--muted);font-size:.8rem}
.board{display:grid;grid-template-columns:repeat(8,1fr);width:100%;aspect-ratio:1;border-radius:9px;overflow:hidden;border:2px solid #405744;direction:ltr;touch-action:manipulation}
.sq{border:0;padding:0;display:flex;align-items:center;justify-content:center;position:relative;font-size:clamp(28px,8.4vw,49px);line-height:1;cursor:pointer;user-select:none}
.sq.light{background:var(--light)}.sq.dark{background:var(--dark)}
.sq.selected{box-shadow:inset 0 0 0 4px #f4d34f}
.sq.last:after{content:"";position:absolute;inset:0;background:#e8d34b55;pointer-events:none}
.sq.legal:before{content:"";width:25%;aspect-ratio:1;border-radius:50%;background:#17251d55;position:absolute}
.sq.capture:before{content:"";position:absolute;inset:4px;border:4px solid #d35b4b99;border-radius:50%}
.piece{position:relative;z-index:1;text-shadow:0 1px 2px #0003}.white-piece{color:#fffdf3;text-shadow:0 2px 2px #111,0 0 2px #222}.black-piece{color:#202820;text-shadow:0 1px 0 #e9e0c8}
.coords{position:absolute;bottom:2px;left:3px;font-size:9px;opacity:.7;z-index:2;color:#202820}.dark .coords{color:#f5f0e3}
.status{margin:12px 2px 2px;font-weight:650}.hint{color:var(--muted);font-size:.85rem;margin:6px 2px 0}
.controls{display:grid;grid-template-columns:1fr 1fr;gap:9px;margin-top:12px}button.control,select{background:#2a3b2e;color:var(--text);border:1px solid #49614d;border-radius:12px;padding:11px 10px;font:inherit}button.control:active{transform:scale(.98)}button.primary{background:var(--accent);color:#1b241b;border-color:var(--accent);font-weight:700}
.bottom{display:grid;grid-template-columns:1fr;gap:10px;margin-top:12px}.log{max-height:130px;overflow:auto;color:var(--muted);font-size:.86rem;line-height:1.8}.log strong{color:var(--text)}.small{font-size:.8rem;color:var(--muted);margin-top:12px;text-align:center}
@media(min-width:480px){body{padding:24px}.sq{font-size:46px}}
</style>
</head>
<body>
<main>
  <div class="top"><div><h1>♟ شطرنج</h1><div class="sub">العب بالأبيض ضد الذكاء الاصطناعي</div></div><div class="pill" id="turnPill">دورك · الأبيض</div></div>
  <section class="card">
    <div class="players"><div>● أنت <span>الأبيض</span></div><div>الذكاء الاصطناعي ● <span>الأسود</span></div></div>
    <div id="board" class="board" aria-label="رقعة الشطرنج"></div>
    <div id="status" class="status" aria-live="polite">دورك الآن. اختر قطعة بيضاء.</div>
    <div class="hint" id="hint">اضغط على القطعة ثم على المربع الذي تريد نقلها إليه.</div>
    <div class="controls">
      <button class="control primary" id="newGame">لعبة جديدة</button>
      <button class="control" id="undo">تراجع عن الحركة</button>
    </div>
    <div class="controls">
      <button class="control" id="flip">قلب الرقعة</button>
      <select id="level" aria-label="مستوى الذكاء الاصطناعي">
        <option value="1">سهل</option><option value="2" selected>متوسط</option><option value="3">صعب</option>
      </select>
    </div>
  </section>
  <section class="card bottom">
    <strong>سجل الحركات</strong><div id="log" class="log">لم تبدأ المباراة بعد.</div>
  </section>
  <div class="small">يدعم الحركات القانونية، الكش، كش مات، التعادل، التبييت، والأخذ بالتجاوز والترقية.</div>
</main>
<script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
<script>
const GLYPHS={wp:"♙",wn:"♘",wb:"♗",wr:"♖",wq:"♕",wk:"♔",bp:"♟",bn:"♞",bb:"♝",br:"♜",bq:"♛",bk:"♚"};
const boardEl=document.getElementById('board'),statusEl=document.getElementById('status'),hintEl=document.getElementById('hint'),pill=document.getElementById('turnPill'),logEl=document.getElementById('log');
let game,selected=null,legal=[],lastMove=null,flipped=false,busy=false,history=[];
function newGame(){game=new Chess();selected=null;legal=[];lastMove=null;flipped=false;busy=false;history=[];render();say('دورك الآن. اختر قطعة بيضاء.');}
function say(s){statusEl.textContent=s;pill.textContent=game.game_over()?'انتهت المباراة':(game.turn()==='w'?'دورك · الأبيض':'دور الأسود');}
function squareName(r,c){return 'abcdefgh'[c]+(8-r);}
function render(){
 boardEl.innerHTML='';
 const arr=game.board();
 for(let vr=0;vr<8;vr++)for(let vc=0;vc<8;vc++){
   const r=flipped?7-vr:vr,c=flipped?7-vc:vc,p=arr[r][c],sq=squareName(r,c);
   const b=document.createElement('button');b.className='sq '+((r+c)%2===0?'light':'dark');b.type='button';b.setAttribute('aria-label',sq+(p?' '+p.type:''));
   if(selected===sq)b.classList.add('selected');
   if(lastMove&&(lastMove.from===sq||lastMove.to===sq))b.classList.add('last');
   const lm=legal.find(m=>m.to===sq);if(lm)b.classList.add(p?'capture':'legal');
   if(p){const span=document.createElement('span');span.className='piece '+(p.color==='w'?'white-piece':'black-piece');span.textContent=GLYPHS[p.color+p.type];b.appendChild(span);}
   if(vc===0){const co=document.createElement('span');co.className='coords';co.textContent=8-r;b.appendChild(co);}
   b.addEventListener('click',()=>tap(sq));boardEl.appendChild(b);
 }
 if(history.length){logEl.innerHTML=history.map((h,i)=>`<div><strong>${i+1}.</strong> ${escapeHtml(h.white||'…')} ${escapeHtml(h.black||'')}</div>`).join('');logEl.scrollTop=logEl.scrollHeight;}
 else logEl.textContent='لم تبدأ المباراة بعد.';
}
function escapeHtml(s){return String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));}
function tap(sq){
 if(busy||game.game_over()||game.turn()!=='w')return;
 const piece=game.get(sq);
 if(selected&&legal.some(m=>m.to===sq)){
   const moves=game.moves({square:selected,verbose:true}).filter(m=>m.to===sq);
   let promo='q';
   if(moves.some(m=>m.flags.includes('p'))){const choice=prompt('ترقية البيدق إلى: q وزير، r قلعة، b فيل، n حصان','q');promo=['q','r','b','n'].includes((choice||'q').toLowerCase())?(choice||'q').toLowerCase():'q';}
   const move=game.move({from:selected,to:sq,promotion:promo});
   if(move){lastMove={from:move.from,to:move.to};record(move,'w');selected=null;legal=[];render();if(finishIfOver())return;say('حركتك: '+move.san+'. يفكر الذكاء الاصطناعي…');busy=true;setTimeout(aiMove,120);return;}
 }
 if(piece&&piece.color==='w'){selected=sq;legal=game.moves({square:sq,verbose:true});hintEl.textContent=legal.length?'الحركات المتاحة محددة على الرقعة.':'هذه القطعة لا تملك حركات قانونية.';}
 else {selected=null;legal=[];}
 render();
}
function record(move,color){if(color==='w')history.push({white:move.san,black:''});else if(history.length)history[history.length-1].black=move.san;}
function finishIfOver(){
 if(!game.game_over())return false;
 busy=false;
 if(game.in_checkmate())say(game.turn()==='w'?'كش مات! فاز الأسود.':'كش مات! فاز الأبيض.');
 else if(game.in_stalemate())say('تعادل بسبب الجمود.');
 else if(game.in_draw())say('انتهت المباراة بالتعادل.');
 else say('انتهت المباراة.');
 hintEl.textContent='اضغط «لعبة جديدة» لبدء مباراة أخرى.';render();return true;
}
const val={p:100,n:320,b:335,r:500,q:900,k:20000};
function evaluate(){let s=0;const b=game.board();for(let r=0;r<8;r++)for(let c=0;c<8;c++){const p=b[r][c];if(!p)continue;let v=val[p.type];const center=3.5-(Math.abs(3.5-r)+Math.abs(3.5-c))/2;if(p.type==='p')v+=(p.color==='w'?6-r:r-1)*5;if(['n','b'].includes(p.type))v+=center*12;if(p.type==='q')v+=center*4;s+=(p.color==='b'?v:-v);}return s;}
function minimax(depth,alpha,beta,maximizing){
 if(depth===0||game.game_over())return evaluate()*(maximizing?1:1);
 const moves=game.moves({verbose:true});
 moves.sort((a,b)=>(b.captured?val[b.captured]:0)-(a.captured?val[a.captured]:0));
 if(maximizing){let best=-Infinity;for(const m of moves){game.move(m);const v=minimax(depth-1,alpha,beta,false);game.undo();best=Math.max(best,v);alpha=Math.max(alpha,best);if(beta<=alpha)break;}return best;}
 let best=Infinity;for(const m of moves){game.move(m);const v=minimax(depth-1,alpha,beta,true);game.undo();best=Math.min(best,v);beta=Math.min(beta,best);if(beta<=alpha)break;}return best;
}
function aiMove(){
 if(game.game_over()){busy=false;finishIfOver();return;}
 const depth=Number(document.getElementById('level').value);
 const moves=game.moves({verbose:true});let best=-Infinity,candidates=[];
 for(const m of moves){game.move(m);let score=minimax(depth-1,-Infinity,Infinity,false);game.undo();
   if(score>best){best=score;candidates=[m];}else if(score===best)candidates.push(m);
 }
 let chosen=candidates[Math.floor(Math.random()*candidates.length)]||moves[0];
 const move=game.move({from:chosen.from,to:chosen.to,promotion:chosen.promotion||'q'});
 lastMove={from:move.from,to:move.to};record(move,'b');busy=false;render();
 if(finishIfOver())return;
 say('رد الذكاء الاصطناعي: '+move.san+'. دورك الآن.');
 hintEl.textContent=game.in_check()?'ملكك في كش؛ أزل التهديد في حركتك التالية.':'اختر حركتك التالية.';
}
document.getElementById('newGame').addEventListener('click',newGame);
document.getElementById('undo').addEventListener('click',()=>{if(busy)return;let n=game.history().length?1:0;if(game.turn()==='w'&&n)n=2;while(n--&&game.history().length)game.undo();history=[];const hist=game.history({verbose:true});for(let i=0;i<hist.length;i+=2){history.push({white:hist[i]?.san||'',black:hist[i+1]?.san||''});}selected=null;legal=[];const h=hist[hist.length-1];lastMove=h?{from:h.from,to:h.to}:null;render();say(game.turn()==='w'?'دورك الآن.':'دور الأسود.');});
document.getElementById('flip').addEventListener('click',()=>{flipped=!flipped;render();});
newGame();
</script>
</body></html>