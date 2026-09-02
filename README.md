<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Chess — Local, Online &amp; Tournament</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#0d1321;color:#eef2f7;font-family:Inter,system-ui,Arial,sans-serif}
.app{max-width:1180px;width:96%;margin:16px auto}.top{display:flex;justify-content:space-between;gap:15px;align-items:center;margin-bottom:14px;flex-wrap:wrap}
h1{margin:0;font-size:28px}.muted{color:#94a3b8;font-size:13px}
.nav{display:flex;gap:8px;margin-bottom:16px;flex-wrap:wrap}
.nav button{background:#182235;border:1px solid #2c3a50;color:#94a3b8;padding:10px 16px;border-radius:10px;cursor:pointer;font-weight:650}
.nav button.active{background:#22c55e;color:#05220e;border-color:#22c55e}
.screen{display:none}.screen.active{display:block}
.grid{display:grid;grid-template-columns:minmax(320px,760px) 320px;gap:18px}
.card{background:#182235;border:1px solid #2c3a50;border-radius:18px;box-shadow:0 14px 45px #0005}.boardCard{padding:12px}.board{display:grid;grid-template-columns:repeat(8,1fr);aspect-ratio:1;border-radius:10px;overflow:hidden;user-select:none}
.sq{position:relative;display:flex;justify-content:center;align-items:center;cursor:pointer}.light{background:#f0d9b5}.dark{background:#b58863}
.sq.sel{box-shadow:inset 0 0 0 5px #facc15}.sq.check{background-image:linear-gradient(#ef4444aa,#ef4444aa)}.sq.hint{box-shadow:inset 0 0 0 4px #38bdf8}
.sq.move:after{content:"";position:absolute;width:22%;height:22%;border-radius:50%;background:#22c55e88}.sq.capture:after{content:"";position:absolute;inset:7%;border:5px solid #22c55e88;border-radius:50%}
.piece{font-family:"Segoe UI Symbol","Noto Sans Symbols 2",serif;font-size:clamp(35px,7.4vw,68px);line-height:1;z-index:2}.wp{color:#fff;text-shadow:0 2px 2px #222}.bp{color:#111;text-shadow:0 1px 1px #fff}
.coord{position:absolute;font-size:10px;font-weight:700;opacity:.65}.rank{top:3px;left:4px}.file{bottom:3px;right:4px}.light .coord{color:#765536}.dark .coord{color:#f7dfb8}
.panel{padding:17px}.status{background:#0e1726;border:1px solid #2e3c52;border-radius:12px;padding:12px;margin-bottom:12px}.status b{font-size:18px}.status div{color:#94a3b8;font-size:12px;margin-top:3px}
select,button,input{font:inherit;border:0;border-radius:10px;padding:11px 12px}select,input{background:#0e1726;color:white;border:1px solid #35445b;width:100%}
button{background:#334155;color:white;cursor:pointer;font-weight:650}button:hover{background:#475569}.primary{background:#22c55e;color:#05220e}.primary:hover{background:#16a34a}
.hintbtn{background:#0284c7}.hintbtn:hover{background:#0369a1}
.modeRow{display:grid;grid-template-columns:1fr;gap:8px;margin-bottom:10px}.buttons{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:12px}
.room{display:grid;grid-template-columns:1fr auto;gap:8px;margin-bottom:8px}.room input{text-transform:uppercase;font-weight:800;letter-spacing:2px}
.title{font-size:12px;text-transform:uppercase;color:#94a3b8;letter-spacing:.08em;margin:14px 0 7px}.moves{height:200px;overflow:auto;background:#0e1726;border-radius:10px;padding:7px}.row{display:grid;grid-template-columns:34px 1fr 1fr;padding:5px 7px;border-bottom:1px solid #223047;font-size:13px}
.cap{display:flex;gap:2px;flex-wrap:wrap;min-height:22px;font-size:16px}
.players{display:grid;gap:7px;margin:12px 0}.player{padding:9px 11px;background:#0e1726;border-radius:10px;display:flex;justify-content:space-between}.you{border:1px solid #22c55e55}
.hidden{display:none!important}
.tform{display:grid;gap:8px;margin-bottom:14px}
.bracket{display:grid;gap:10px}
.bmatch{background:#0e1726;border:1px solid #2e3c52;border-radius:12px;padding:12px}
.bmatch .rd{font-size:11px;text-transform:uppercase;color:#94a3b8;letter-spacing:.08em;margin-bottom:6px}
.bp2{display:flex;justify-content:space-between;padding:6px 0;border-bottom:1px solid #223047}
.bp2:last-of-type{border-bottom:none}
.bp2.win{color:#22c55e;font-weight:700}
.champ{font-size:20px;font-weight:800;color:#facc15;text-align:center;padding:16px;background:#0e1726;border-radius:12px;margin-top:10px}
.online .card{margin-bottom:0}
@media(max-width:900px){.grid{grid-template-columns:1fr}.panel{display:grid;grid-template-columns:1fr 1fr;gap:10px}.status,.modeRow,.buttons,.title,.moves,.room,.players{grid-column:span 2}.moves{height:170px}}
@media(max-width:500px){.app{width:98%;margin:8px auto}.boardCard{padding:6px}.panel{padding:11px}.piece{font-size:clamp(29px,11vw,46px)}}
</style>
</head>
<body>
<div class="app">
<div class="top"><div><h1>♟ Chess</h1><div class="muted">Local · Online 1v1 · 6-Team Tournament</div></div><div id="net" class="muted">Firebase: connecting…</div></div>
<div class="nav">
<button id="navLocal" class="active">Quick Play</button>
<button id="navOnline">Online 1v1</button>
<button id="navTourn">Tournament</button>
</div>

<!-- LOCAL SCREEN -->
<div id="scrLocal" class="screen active">
<div class="grid">
<div class="card boardCard"><div id="board-local" class="board"></div></div>
<div class="card panel">
<div id="status-local" class="status"><b>White to move</b><div>New game</div></div>
<div class="modeRow">
<select id="mode"><option value="pvp">Pass &amp; Play (2 players)</option><option value="cpu-b">Vs Computer — I play White</option><option value="cpu-w">Vs Computer — I play Black</option></select>
<select id="diff"><option value="1">Computer: Easy</option><option value="2" selected>Computer: Medium</option><option value="3">Computer: Hard</option></select>
</div>
<div class="buttons"><button id="newgame" class="primary">New Game</button><button id="flip-local">Flip Board</button></div>
<div class="buttons"><button id="hint-local" class="hintbtn">💡 Hint</button><button id="undo">Undo</button></div>
<div class="title">Captured</div><div class="cap" id="capWhite-local"></div><div class="cap" id="capBlack-local"></div>
<div class="title">Move History</div><div id="moves-local" class="moves"></div>
</div>
</div>
</div>

<!-- ONLINE 1v1 SCREEN -->
<div id="scrOnline" class="screen">
<div class="grid">
<div class="card boardCard"><div id="board-online" class="board"></div></div>
<div class="card panel">
<div id="status-online" class="status"><b>Not connected</b><div>Create or join a room.</div></div>
<div id="timer-online" class="muted" style="margin:-6px 0 10px;font-weight:700"></div>
<div id="rejoinBanner-online" class="hidden" style="background:#0e1726;border:1px solid #22c55e55;border-radius:12px;padding:10px 12px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center;gap:8px"><span class="muted">You have a game in progress</span><button id="rejoinBtn-online" class="primary" style="padding:8px 12px">Rejoin</button></div>
<div id="inviteBanner-online" class="hidden" style="background:#0e1726;border:1px solid #facc1555;border-radius:12px;padding:10px 12px;margin-bottom:10px"></div>
<div class="title" style="margin-top:0">Your Player ID</div>
<div class="room"><input id="myCodeShow" readonly><button id="copyMyCode">Copy</button></div>
<input id="myNameInput" placeholder="Your display name" style="margin-bottom:8px">
<div class="room"><input id="friendCode" maxlength="6" placeholder="FRIEND'S PLAYER ID"><button id="sendInvite" class="primary">Invite</button></div>
<button id="showActiveBtn" style="width:100%;margin-bottom:10px">👥 See Active Players</button>
<div class="title">Or use a Room Code</div>
<div class="room"><input id="roomCode" maxlength="6" placeholder="ROOM CODE"><button id="copyRoom">Copy</button></div>
<div class="buttons"><button id="createRoom" class="primary">Create Room</button><button id="joinRoom">Join Room</button></div>
<div class="players"><div class="player" id="online-white"><span>♔ White</span><b>Waiting</b></div><div class="player" id="online-black"><span>♚ Black</span><b>Waiting</b></div></div>
<div class="buttons"><button id="hint-online" class="hintbtn">💡 Hint</button><button id="flip-online">Flip Board</button></div>
<div class="buttons"><button id="resignOnline">Resign</button></div>
<div class="title">Move History</div><div id="moves-online" class="moves"></div>
</div>
</div>
</div>

<!-- TOURNAMENT SCREEN -->
<div id="scrTourn" class="screen">
<div id="tournHome">
<div class="grid" style="grid-template-columns:1fr 1fr">
<div class="card panel">
<div class="title">Create Tournament (6 players)</div>
<div class="tform">
<input id="t1" placeholder="Player 1 (seed 1)"><input id="t2" placeholder="Player 2 (seed 2)">
<input id="t3" placeholder="Player 3 (seed 3)"><input id="t4" placeholder="Player 4 (seed 4)">
<input id="t5" placeholder="Player 5 (seed 5)"><input id="t6" placeholder="Player 6 (seed 6)">
</div>
<button id="createTourn" class="primary">Create Tournament</button>
<div class="muted" style="margin-top:8px">Seeds 1 &amp; 2 get a bye straight to semifinals. Seeds 3-6 play quarterfinals.</div>
</div>
<div class="card panel">
<div class="title">Join Tournament</div>
<div id="rejoinBanner-tourn" class="hidden" style="background:#0e1726;border:1px solid #22c55e55;border-radius:12px;padding:10px 12px;margin-bottom:10px;display:flex;justify-content:space-between;align-items:center;gap:8px"><span class="muted">You were in a tournament</span><button id="rejoinBtn-tourn" class="primary" style="padding:8px 12px">Rejoin</button></div>
<div class="room"><input id="tid" placeholder="TOURNAMENT ID"><button id="loadTourn" class="primary">Load</button></div>
<div class="muted">Ask the organizer for the Tournament ID and enter it here to view the bracket and play your matches.</div>
</div>
</div>
</div>
<div id="tournView" class="hidden">
<div class="card panel" style="margin-bottom:16px">
<div class="top" style="margin-bottom:0"><div><b id="tournName" style="font-size:20px"></b><div class="muted">Tournament ID: <b id="tournIdShow"></b></div></div><button id="backHome">← Back</button></div>
</div>
<div id="champBox"></div>
<div class="bracket" id="bracketBox"></div>
</div>
<div id="tournMatch" class="hidden">
<div class="grid">
<div class="card boardCard"><div id="board-tourn" class="board"></div></div>
<div class="card panel">
<div id="status-tourn" class="status"><b>Not connected</b><div></div></div>
<div id="timer-tourn" class="muted" style="margin:-6px 0 10px;font-weight:700"></div>
<div class="players"><div class="player" id="tourn-white"><span>♔ White</span><b>Waiting</b></div><div class="player" id="tourn-black"><span>♚ Black</span><b>Waiting</b></div></div>
<div class="buttons"><button id="hint-tourn" class="hintbtn">💡 Hint</button><button id="flip-tourn">Flip Board</button></div>
<button id="backBracket" style="margin-bottom:12px">← Back to Bracket</button>
<div class="title">Move History</div><div id="moves-tourn" class="moves"></div>
</div>
</div>
</div>
</div>

</div>

<div id="activeModal" class="hidden" style="position:fixed;inset:0;background:#000a;z-index:50;display:flex;align-items:center;justify-content:center;padding:16px">
 <div class="card panel" style="max-width:440px;width:100%;max-height:75vh;overflow:auto">
  <div class="top" style="margin-bottom:10px"><b style="font-size:18px">👥 Active Players</b><button id="closeActiveModal">✕</button></div>
  <div id="activeList" class="muted">Loading…</div>
 </div>
</div>

<script type="module">
const firebaseConfig={
  apiKey:"AIzaSyBPwEOdI8WwPKZVCJhvyANp5MPQKmAG79w",
  authDomain:"chess-dab2c.firebaseapp.com",
  projectId:"chess-dab2c",
  storageBucket:"chess-dab2c.firebasestorage.app",
  messagingSenderId:"639996000069",
  appId:"1:639996000069:web:716d98a89d0fbb3e10702c"
};
let db=null,doc,getDoc,setDoc,updateDoc,onSnapshot,runTransaction,collection,query,where,addDoc,deleteDoc,orderBy,limit;
let playerId=localStorage.getItem("chessPlayerId");
if(!playerId){playerId=crypto.randomUUID();localStorage.setItem("chessPlayerId",playerId)}
let myCode=localStorage.getItem("chessMyCode");
if(!myCode){myCode=code();localStorage.setItem("chessMyCode",myCode)}
let myName=localStorage.getItem("chessMyName")||("Player-"+myCode.slice(0,3));
let heartbeatInterval=null;

let firebaseLoadPromise=null;
function loadFirebase(){
 if(firebaseLoadPromise)return firebaseLoadPromise;
 firebaseLoadPromise=(async()=>{
  try{
   document.getElementById("net").textContent="Firebase: connecting…";
   const appMod=await import("https://www.gstatic.com/firebasejs/12.18.0/firebase-app.js");
   const fsMod=await import("https://www.gstatic.com/firebasejs/12.18.0/firebase-firestore.js");
   const app=appMod.initializeApp(firebaseConfig);
   db=fsMod.getFirestore(app);
   doc=fsMod.doc;getDoc=fsMod.getDoc;setDoc=fsMod.setDoc;updateDoc=fsMod.updateDoc;onSnapshot=fsMod.onSnapshot;runTransaction=fsMod.runTransaction;
   collection=fsMod.collection;query=fsMod.query;where=fsMod.where;addDoc=fsMod.addDoc;deleteDoc=fsMod.deleteDoc;orderBy=fsMod.orderBy;limit=fsMod.limit;
   document.getElementById("net").textContent="Firebase: connected";
   registerPlayer();
   listenIncomingInvites();
   if(!heartbeatInterval)heartbeatInterval=setInterval(registerPlayer,25000);
   return true;
  }catch(e){
   console.error(e);
   document.getElementById("net").textContent="Firebase: unavailable (Online/Tournament disabled)";
   return false;
  }
 })();
 return firebaseLoadPromise;
}
async function registerPlayer(){
 try{await setDoc(doc(db,"players",myCode),{ownerId:playerId,name:myName,updatedAt:Date.now()})}catch(e){console.error(e)}
}

/* ============ CORE CHESS ENGINE (shared by all modes) ============ */
const PIECES={K:"♔",Q:"♕",R:"♖",B:"♗",N:"♘",P:"♙",k:"♚",q:"♛",r:"♜",b:"♝",n:"♞",p:"♟"};
const VAL={P:100,N:320,B:330,R:500,Q:900,K:20000},FILES="abcdefgh";
const PST_P=[0,0,0,0,0,0,0,0,50,50,50,50,50,50,50,50,10,10,20,30,30,20,10,10,5,5,10,25,25,10,5,5,0,0,0,20,20,0,0,0,5,-5,-10,0,0,-10,-5,5,5,10,10,-20,-20,10,10,5,0,0,0,0,0,0,0,0];
const PST_N=[-50,-40,-30,-30,-30,-30,-40,-50,-40,-20,0,0,0,0,-20,-40,-30,0,10,15,15,10,0,-30,-30,5,15,20,20,15,5,-30,-30,0,15,20,20,15,0,-30,-30,5,10,15,15,10,5,-30,-40,-20,0,5,5,0,-20,-40,-50,-40,-30,-30,-30,-30,-40,-50];

function initialBoard(){return [["r","n","b","q","k","b","n","r"],["p","p","p","p","p","p","p","p"],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],["P","P","P","P","P","P","P","P"],["R","N","B","Q","K","B","N","R"]]}
/* Firestore does not support nested arrays, so the 8x8 board (array of arrays) must be
   flattened to a single 1D array of 64 cells before writing, and rebuilt into 2D on read. */
function flattenBoard(b){return b.flat()}
function unflattenBoard(flat){let b=[];for(let r=0;r<8;r++)b.push(flat.slice(r*8,r*8+8));return b}
function pcolor(p){return p&&p===p.toUpperCase()?"w":"b"}
function opp(c){return c==="w"?"b":"w"}
function ib(r,c){return r>=0&&r<8&&c>=0&&c<8}
function king(b,c){let k=c==="w"?"K":"k";for(let r=0;r<8;r++)for(let x=0;x<8;x++)if(b[r][x]===k)return[r,x];return null}
function attacked(b,r,c,by){
 let p=by==="w"?"P":"p",pr=r+(by==="w"?1:-1);
 for(let dc of[-1,1])if(ib(pr,c+dc)&&b[pr][c+dc]===p)return true;
 p=by==="w"?"N":"n";
 for(let[dr,dc]of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])if(ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 p=by==="w"?"K":"k";
 for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if((dr||dc)&&ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 for(let[dr,dc,type]of[[1,0,"R"],[-1,0,"R"],[0,1,"R"],[0,-1,"R"],[1,1,"B"],[1,-1,"B"],[-1,1,"B"],[-1,-1,"B"]]){
  let rr=r+dr,cc=c+dc;
  while(ib(rr,cc)){let q=b[rr][cc];if(q){if(pcolor(q)===by&&(q.toUpperCase()===type||q.toUpperCase()==="Q"))return true;break}rr+=dr;cc+=dc}
 }
 return false
}
function inCheck(b,c){let k=king(b,c);return !k||attacked(b,k[0],k[1],opp(c))}
function pseudo(b,c,cast,epSq){
 const out=[],add=(fr,fc,tr,tc,e={})=>{if(!ib(tr,tc))return;let t=b[tr][tc];if(t&&pcolor(t)===c)return;out.push({fr,fc,tr,tc,piece:b[fr][fc],captured:t,...e})};
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){
  let p=b[r][x];if(!p||pcolor(p)!==c)continue;let u=p.toUpperCase();
  if(u==="P"){
   let d=c==="w"?-1:1,start=c==="w"?6:1,pr=c==="w"?0:7;
   if(ib(r+d,x)&&!b[r+d][x]){
    if(r+d===pr)for(let q of["Q","R","B","N"])out.push({fr:r,fc:x,tr:r+d,tc:x,piece:p,captured:null,promotion:c==="w"?q:q.toLowerCase()});
    else add(r,x,r+d,x);
    if(r===start&&!b[r+2*d][x])add(r,x,r+2*d,x,{double:true})
   }
   for(let dc of[-1,1]){
    let tr=r+d,tc=x+dc;if(!ib(tr,tc))continue;
    if(b[tr][tc]&&pcolor(b[tr][tc])!==c){
     if(tr===pr)for(let q of["Q","R","B","N"])out.push({fr:r,fc:x,tr,tc,piece:p,captured:b[tr][tc],promotion:c==="w"?q:q.toLowerCase()});
     else add(r,x,tr,tc)
    }
    if(epSq&&epSq[0]===tr&&epSq[1]===tc)out.push({fr:r,fc:x,tr,tc,piece:p,captured:b[r][tc],ep:true})
   }
  }else if(u==="N"){
   for(let[dr,dc]of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])add(r,x,r+dr,x+dc)
  }else if(u==="K"){
   for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if(dr||dc)add(r,x,r+dr,x+dc);
   let home=c==="w"?7:0;
   if(r===home&&x===4&&!inCheck(b,c)){
    let k=c==="w"?cast.wK:cast.bK,q=c==="w"?cast.wQ:cast.bQ;
    if(k&&!b[home][5]&&!b[home][6]&&b[home][7]===(c==="w"?"R":"r")&&!attacked(b,home,5,opp(c))&&!attacked(b,home,6,opp(c)))out.push({fr:r,fc:x,tr:home,tc:6,piece:p,captured:null,castle:"K"});
    if(q&&!b[home][1]&&!b[home][2]&&!b[home][3]&&b[home][0]===(c==="w"?"R":"r")&&!attacked(b,home,3,opp(c))&&!attacked(b,home,2,opp(c)))out.push({fr:r,fc:x,tr:home,tc:2,piece:p,captured:null,castle:"Q"})
   }
  }else{
   let ds=u==="R"?[[1,0],[-1,0],[0,1],[0,-1]]:u==="B"?[[1,1],[1,-1],[-1,1],[-1,-1]]:[[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]];
   for(let[dr,dc]of ds){
    let rr=r+dr,cc=x+dc;
    while(ib(rr,cc)){let t=b[rr][cc];if(!t)add(r,x,rr,cc);else{if(pcolor(t)!==c)add(r,x,rr,cc);break}rr+=dr;cc+=dc}
   }
  }
 }
 return out
}
function applyMove(b,m,cast,epSq){
 let nb=b.map(a=>a.slice()),nc={...cast},nep=null;
 nb[m.tr][m.tc]=m.promotion||m.piece;nb[m.fr][m.fc]=null;
 if(m.ep)nb[m.fr][m.tc]=null;
 if(m.castle==="K"){let home=m.fr;nb[home][5]=nb[home][7];nb[home][7]=null}
 if(m.castle==="Q"){let home=m.fr;nb[home][3]=nb[home][0];nb[home][0]=null}
 if(m.piece.toUpperCase()==="K"){if(pcolor(m.piece)==="w"){nc.wK=false;nc.wQ=false}else{nc.bK=false;nc.bQ=false}}
 if(m.piece.toUpperCase()==="R"){
  if(m.fr===7&&m.fc===0)nc.wQ=false;if(m.fr===7&&m.fc===7)nc.wK=false;
  if(m.fr===0&&m.fc===0)nc.bQ=false;if(m.fr===0&&m.fc===7)nc.bK=false;
 }
 if(m.captured&&m.captured.toUpperCase()==="R"){
  if(m.tr===7&&m.tc===0)nc.wQ=false;if(m.tr===7&&m.tc===7)nc.wK=false;
  if(m.tr===0&&m.tc===0)nc.bQ=false;if(m.tr===0&&m.tc===7)nc.bK=false;
 }
 if(m.double)nep=[(m.fr+m.tr)/2,m.fc];
 return{nb,nc,nep}
}
function legalMoves(b,c,cast,epSq){
 return pseudo(b,c,cast,epSq).filter(m=>{let{nb}=applyMove(b,m,cast,epSq);return !inCheck(nb,c)})
}
function notation(b,m,cast,epSq){
 let u=m.piece.toUpperCase(),to=FILES[m.tc]+(8-m.tr);
 if(m.castle==="K")return "O-O";
 if(m.castle==="Q")return "O-O-O";
 let s=u==="P"?(m.captured?FILES[m.fc]+"x"+to:to):(u+(m.captured?"x":"")+to);
 if(m.promotion)s+="="+m.promotion.toUpperCase();
 let{nb}=applyMove(b,m,cast,epSq),nc2=opp(pcolor(m.piece));
 if(inCheck(nb,nc2)){let hasMoves=legalMoves(nb,nc2,cast,null).length>0;s+=hasMoves?"+":"#"}
 return s
}
function evalBoard(b){
 let s=0;
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){
  let p=b[r][x];if(!p)continue;
  let u=p.toUpperCase(),c=pcolor(p),sign=c==="w"?1:-1;
  s+=sign*VAL[u];
  let idx=c==="w"?r*8+x:(7-r)*8+x;
  if(u==="P")s+=sign*PST_P[idx];
  if(u==="N")s+=sign*PST_N[idx];
 }
 return s
}
function orderMoves(moves){return moves.length>1?moves.slice().sort((a,b2)=>(b2.captured?VAL[b2.captured.toUpperCase()]:0)-(a.captured?VAL[a.captured.toUpperCase()]:0)):moves}
function minimax(b,c,cast,epSq,depth,alpha,beta){
 let moves=orderMoves(legalMoves(b,c,cast,epSq));
 if(depth===0||!moves.length){
  if(!moves.length){if(inCheck(b,c))return c==="w"?-99999:99999;return 0}
  return evalBoard(b);
 }
 if(c==="w"){
  let v=-Infinity;
  for(let m of moves){let{nb,nc,nep}=applyMove(b,m,cast,epSq);v=Math.max(v,minimax(nb,"b",nc,nep,depth-1,alpha,beta));alpha=Math.max(alpha,v);if(beta<=alpha)break}
  return v
 }else{
  let v=Infinity;
  for(let m of moves){let{nb,nc,nep}=applyMove(b,m,cast,epSq);v=Math.min(v,minimax(nb,"w",nc,nep,depth-1,alpha,beta));beta=Math.min(beta,v);if(beta<=alpha)break}
  return v
 }
}
function bestMove(board,turn,castling,ep,depth){
 let moves=legalMoves(board,turn,castling,ep);
 if(!moves.length)return null;
 let best=null,bestScore=turn==="w"?-Infinity:Infinity;
 let order=orderMoves(moves);
 for(let m of order){
  let{nb,nc,nep}=applyMove(board,m,castling,ep);
  let score=minimax(nb,opp(turn),nc,nep,depth-1,-Infinity,Infinity);
  if(turn==="w"?score>bestScore:score<bestScore){bestScore=score;best=m}
 }
 return best||moves[0]
}

/* ============ REUSABLE BOARD RENDERER ============ */
function makeBoardUI(elId,statusId,movesId,capWId,capBId){
 return {
  el:document.getElementById(elId),statusEl:statusId?document.getElementById(statusId):null,
  movesEl:movesId?document.getElementById(movesId):null,capWEl:capWId?document.getElementById(capWId):null,capBEl:capBId?document.getElementById(capBId):null
 }
}
function say(statusEl,a,b){if(statusEl)statusEl.innerHTML="<b>"+a+"</b><div>"+(b||"")+"</div>"}
function renderBoard(ui,state){
 const el=ui.el;el.innerHTML="";if(!state.board)return;
 const rs=state.flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7],cs=state.flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7];
 for(let ri=0;ri<8;ri++)for(let ci=0;ci<8;ci++){
  let r=rs[ri],c=cs[ci],s=document.createElement("div");
  s.className="sq "+((r+c)%2?"dark":"light");
  if(state.selected&&state.selected[0]===r&&state.selected[1]===c)s.classList.add("sel");
  let lm=state.legal.find(m=>m.tr===r&&m.tc===c);if(lm)s.classList.add(state.board[r][c]?"capture":"move");
  let k=king(state.board,state.turn);if(k&&k[0]===r&&k[1]===c&&inCheck(state.board,state.turn))s.classList.add("check");
  if(state.hint&&((state.hint.fr===r&&state.hint.fc===c)||(state.hint.tr===r&&state.hint.tc===c)))s.classList.add("hint");
  if(state.board[r][c]){let z=document.createElement("span");z.className="piece "+(pcolor(state.board[r][c])==="w"?"wp":"bp");z.textContent=PIECES[state.board[r][c]];s.appendChild(z)}
  if(ci===0){let z=document.createElement("span");z.className="coord rank";z.textContent=8-r;s.appendChild(z)}
  if(ri===7){let z=document.createElement("span");z.className="coord file";z.textContent=FILES[c];s.appendChild(z)}
  s.onclick=()=>state.onClick(r,c);
  el.appendChild(s)
 }
 if(ui.movesEl){
  ui.movesEl.innerHTML=(state.history||[]).map((x,i)=>i%2===0?`<div class="row"><span>${i/2+1}.</span><span>${x}</span><span>${state.history[i+1]||""}</span></div>`:"").join("");
  ui.movesEl.scrollTop=ui.movesEl.scrollHeight;
 }
 if(ui.capWEl)ui.capWEl.innerHTML=(state.captured?.b||[]).map(p=>PIECES[p]).join(" ");
 if(ui.capBEl)ui.capBEl.innerHTML=(state.captured?.w||[]).map(p=>PIECES[p]).join(" ");
}

/* ============ QUICK PLAY (LOCAL) ============ */
const localUI=makeBoardUI("board-local","status-local","moves-local","capWhite-local","capBlack-local");
let L={};
function localHumanColor(){let m=document.getElementById("mode").value;if(m==="pvp")return null;return m==="cpu-b"?"w":"b"}
function newLocalGame(){
 L={board:initialBoard(),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],captured:{w:[],b:[]},selected:null,legal:[],flipped:false,hint:null,gameOver:false,onClick:localClick};
 say(localUI.statusEl,"White to move","New game");
 renderBoard(localUI,L);
 maybeComputerMove();
}
function localClick(r,c){
 if(L.gameOver)return;
 let hc=localHumanColor();
 if(hc!==null&&L.turn!==hc)return;
 L.hint=null;
 let p=L.board[r][c],m=L.legal.find(x=>x.tr===r&&x.tc===c);
 if(L.selected&&m){makeLocalMove(m);return}
 if(p&&pcolor(p)===L.turn){L.selected=[r,c];L.legal=legalMoves(L.board,L.turn,L.castling,L.ep).filter(x=>x.fr===r&&x.fc===c);renderBoard(localUI,L)}
 else{L.selected=null;L.legal=[];renderBoard(localUI,L)}
}
function makeLocalMove(m){
 let note=notation(L.board,m,L.castling,L.ep);
 let{nb,nc,nep}=applyMove(L.board,m,L.castling,L.ep);
 if(m.captured)L.captured[pcolor(m.captured)].push(m.captured);
 L.history.push(note);L.board=nb;L.castling=nc;L.ep=nep;L.turn=opp(L.turn);L.selected=null;L.legal=[];L.hint=null;
 let moves=legalMoves(L.board,L.turn,L.castling,L.ep);
 if(!moves.length){L.gameOver=true;if(inCheck(L.board,L.turn))say(localUI.statusEl,L.turn==="w"?"Black wins":"White wins","Checkmate");else say(localUI.statusEl,"Draw","Stalemate")}
 else if(inCheck(L.board,L.turn))say(localUI.statusEl,(L.turn==="w"?"White":"Black")+" to move","Check!");
 else say(localUI.statusEl,(L.turn==="w"?"White":"Black")+" to move","");
 renderBoard(localUI,L);
 maybeComputerMove();
}
function maybeComputerMove(){
 if(L.gameOver)return;
 let hc=localHumanColor();
 if(hc===null||L.turn===hc)return;
 setTimeout(()=>{
  let diff=parseInt(document.getElementById("diff").value);
  let depth=diff===1?1:diff===2?2:3;
  let moves=legalMoves(L.board,L.turn,L.castling,L.ep);
  if(!moves.length)return;
  let m=(diff===1&&Math.random()<0.35)?moves[Math.floor(Math.random()*moves.length)]:bestMove(L.board,L.turn,L.castling,L.ep,depth);
  if(m)makeLocalMove(m);
 },150);
}
document.getElementById("newgame").onclick=newLocalGame;
document.getElementById("flip-local").onclick=()=>{L.flipped=!L.flipped;renderBoard(localUI,L)};
document.getElementById("mode").onchange=newLocalGame;
document.getElementById("hint-local").onclick=()=>{
 if(L.gameOver)return;
 let hc=localHumanColor();if(hc!==null&&L.turn!==hc)return;
 let m=bestMove(L.board,L.turn,L.castling,L.ep,2);
 if(m){L.hint=m;renderBoard(localUI,L);say(localUI.statusEl,(L.turn==="w"?"White":"Black")+" to move","Hint: "+FILES[m.fc]+(8-m.fr)+" → "+FILES[m.tc]+(8-m.tr))}
};
document.getElementById("undo").onclick=()=>{
 alert("Undo isn't available mid-game in this version — start New Game to reset.");
};

/* ============ ONLINE 1v1 ============ */
const ONLINE_SECONDS=30;
const onlineUI=makeBoardUI("board-online","status-online","moves-online",null,null);
let O={board:null,turn:"w",castling:null,ep:null,history:[],selected:null,legal:[],flipped:false,hint:null,onClick:onlineClick};
let onlineRoomRef=null,onlineUnsub=null,onlineRoom=null,onlineMyColor=null,onlineBusy=false;
let onlineTurnKey=null,onlineAutoMoved=false,myInviteUnsub=null,incomingUnsub=null;
function onlineClick(r,c){
 if(!onlineRoom||onlineRoom.status!=="playing"||onlineRoom.turn!==onlineMyColor||onlineBusy)return;
 O.hint=null;
 let p=O.board[r][c],m=O.legal.find(x=>x.tr===r&&x.tc===c);
 if(O.selected&&m){optimisticOnlineMove(m);return}
 if(p&&pcolor(p)===onlineMyColor){O.selected=[r,c];O.legal=legalMoves(O.board,onlineMyColor,onlineRoom.castling,onlineRoom.ep).filter(x=>x.fr===r&&x.fc===c);renderBoard(onlineUI,O)}
 else{O.selected=null;O.legal=[];renderBoard(onlineUI,O)}
}
function optimisticOnlineMove(m){
 // Render the move instantly (don't wait for the Firestore round-trip) so play feels fast; the
 // background write in sendOnlineMove() is still the source of truth and will correct any conflict.
 const ap=applyMove(O.board,m,onlineRoom.castling,onlineRoom.ep);
 O.board=ap.nb;O.selected=null;O.legal=[];O.hint=null;
 renderBoard(onlineUI,O);
 say(onlineUI.statusEl,"Opponent's turn","Move sent…");
 sendOnlineMove(m);
}
async function sendOnlineMove(m){
 if(onlineBusy)return;onlineBusy=true;
 try{
  await runTransaction(db,async tx=>{
   const snap=await tx.get(onlineRoomRef);if(!snap.exists())throw Error("Room does not exist");
   const d=snap.data();if(d.turn!==onlineMyColor)throw Error("Not your turn");
   const b=unflattenBoard(d.board),cast={...d.castling},ep=d.ep;
   const lm=legalMoves(b,onlineMyColor,cast,ep).find(x=>JSON.stringify(x)===JSON.stringify(m));if(!lm)throw Error("Illegal move");
   const note=notation(b,lm,cast,ep),ap=applyMove(b,lm,cast,ep);
   let hist=[...(d.history||[]),note],next=opp(onlineMyColor),ms=legalMoves(ap.nb,next,ap.nc,ap.nep),winner=null,status="playing";
   if(!ms.length){status="finished";winner=inCheck(ap.nb,next)?onlineMyColor:"draw"}
   tx.update(onlineRoomRef,{board:flattenBoard(ap.nb),castling:ap.nc,ep:ap.nep,turn:next,history:hist,status,winner,turnStartedAt:Date.now(),updatedAt:Date.now()});
  });
 }catch(e){console.error(e);alert(e.message)}finally{onlineBusy=false}
}
function code(){let s="";for(let i=0;i<6;i++)s+="ABCDEFGHJKLMNPQRSTUVWXYZ23456789"[Math.floor(Math.random()*32)];return s}
function configReady(){return firebaseConfig.apiKey!=="YOUR_API_KEY"}
document.getElementById("createRoom").onclick=async()=>{
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 try{
  const id=code();onlineRoomRef=doc(db,"chessRooms",id);
  const init={board:flattenBoard(initialBoard()),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],whiteId:playerId,whiteName:myName,blackId:null,blackName:null,status:"waiting",winner:null,turnStartedAt:Date.now(),updatedAt:Date.now()};
  await setDoc(onlineRoomRef,init);document.getElementById("roomCode").value=id;onlineMyColor="w";listenOnline(id)
 }catch(e){console.error(e);say(onlineUI.statusEl,"Firebase error",e.code||e.message);alert("Create room failed: "+(e.code||e.message)+"\n\nCheck Firestore is enabled and rules allow read/write in the Firebase console.")}
};
document.getElementById("joinRoom").onclick=async()=>{
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 try{
  const id=document.getElementById("roomCode").value.trim().toUpperCase();if(id.length!==6){alert("Enter a 6-character room code.");return}
  onlineRoomRef=doc(db,"chessRooms",id);let s=await getDoc(onlineRoomRef);if(!s.exists()){alert("Room not found.");return}
  let d=s.data();if(d.blackId&&d.blackId!==playerId){alert("Room already has two players.");return}
  if(d.whiteId===playerId)onlineMyColor="w";else{await updateDoc(onlineRoomRef,{blackId:playerId,blackName:myName,status:"playing",turnStartedAt:Date.now(),updatedAt:Date.now()});onlineMyColor="b"}
  listenOnline(id)
 }catch(e){console.error(e);say(onlineUI.statusEl,"Firebase error",e.code||e.message);alert("Join room failed: "+(e.code||e.message))}
};
function listenOnline(id){
 if(onlineUnsub)onlineUnsub();document.getElementById("roomCode").value=id;
 localStorage.setItem("chessLastRoom",id);checkRejoinAvailable();
 onlineUnsub=onSnapshot(onlineRoomRef,s=>{
  if(!s.exists()){say(onlineUI.statusEl,"Room closed","");return}
  onlineRoom=s.data();O.board=unflattenBoard(onlineRoom.board);O.turn=onlineRoom.turn;O.history=onlineRoom.history||[];O.selected=null;O.legal=[];
  const turnKey=onlineRoom.turn+"-"+(onlineRoom.history?onlineRoom.history.length:0);
  if(turnKey!==onlineTurnKey){onlineTurnKey=turnKey;onlineAutoMoved=false}
  document.querySelector("#online-white b").textContent=(onlineRoom.whiteName||"Waiting")+(onlineMyColor==="w"?" (You)":"");
  document.querySelector("#online-black b").textContent=(onlineRoom.blackName||"Waiting")+(onlineMyColor==="b"?" (You)":"");
  if(onlineRoom.winner){say(onlineUI.statusEl,onlineRoom.winner==="draw"?"Draw":onlineRoom.winner==="w"?"White wins":"Black wins","Game over")}
  else if(onlineRoom.status==="waiting")say(onlineUI.statusEl,"Waiting for opponent","Send your Player ID or room code: "+id);
  else say(onlineUI.statusEl,onlineRoom.turn===onlineMyColor?"Your turn":"Opponent's turn",onlineRoom.turn==="w"?"White to move":"Black to move");
  renderBoard(onlineUI,O);
  updateOnlineTimerDisplay();
 },e=>{say(onlineUI.statusEl,"Firebase error",e.message)});
}
function updateOnlineTimerDisplay(){
 const el=document.getElementById("timer-online");if(!el)return;
 if(!onlineRoom||onlineRoom.winner||onlineRoom.status!=="playing"){el.textContent="";return}
 const elapsed=Math.floor((Date.now()-(onlineRoom.turnStartedAt||Date.now()))/1000);
 const left=Math.max(0,ONLINE_SECONDS-elapsed);
 const whoseTurn=onlineRoom.turn===onlineMyColor?"Your":(onlineRoom.turn==="w"?"White's":"Black's");
 el.textContent="⏱ "+whoseTurn+" time: "+left+"s";
 if(left<=0&&onlineRoom.turn===onlineMyColor&&!onlineAutoMoved&&!onlineBusy){onlineAutoMoved=true;autoPlayOnline()}
}
async function autoPlayOnline(){
 let m=bestMove(O.board,onlineMyColor,onlineRoom.castling,onlineRoom.ep,1);
 if(!m){let ms=legalMoves(O.board,onlineMyColor,onlineRoom.castling,onlineRoom.ep);if(ms.length)m=ms[Math.floor(Math.random()*ms.length)]}
 if(m)await sendOnlineMove(m);
}
setInterval(()=>{updateOnlineTimerDisplay();updateTournTimerDisplay()},1000);
document.getElementById("flip-online").onclick=()=>{O.flipped=!O.flipped;renderBoard(onlineUI,O)};
document.getElementById("copyRoom").onclick=async()=>{let v=document.getElementById("roomCode").value;if(v)await navigator.clipboard.writeText(v)};
document.getElementById("resignOnline").onclick=async()=>{if(!db)return;if(onlineRoom?.status==="playing"&&onlineMyColor){if(confirm("Resign this game?")){await updateDoc(onlineRoomRef,{status:"finished",winner:opp(onlineMyColor),updatedAt:Date.now()});localStorage.removeItem("chessLastRoom");checkRejoinAvailable()}}};
document.getElementById("hint-online").onclick=()=>{
 if(!onlineRoom||onlineRoom.status!=="playing"||onlineRoom.turn!==onlineMyColor)return;
 let m=bestMove(O.board,onlineMyColor,onlineRoom.castling,onlineRoom.ep,2);
 if(m){O.hint=m;renderBoard(onlineUI,O);say(onlineUI.statusEl,"Your turn","Hint: "+FILES[m.fc]+(8-m.fr)+" → "+FILES[m.tc]+(8-m.tr))}
};

/* ---- Rejoin (reconnect after crash/back/refresh) ---- */
function checkRejoinAvailable(){
 const r=localStorage.getItem("chessLastRoom");
 document.getElementById("rejoinBanner-online").classList.toggle("hidden",!r);
 const t=localStorage.getItem("chessLastTournId");
 document.getElementById("rejoinBanner-tourn").classList.toggle("hidden",!t);
}
document.getElementById("rejoinBtn-online").onclick=async()=>{
 const id=localStorage.getItem("chessLastRoom");if(!id)return;
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection.");return}
 try{
  onlineRoomRef=doc(db,"chessRooms",id);
  const s=await getDoc(onlineRoomRef);
  if(!s.exists()){alert("That room no longer exists.");localStorage.removeItem("chessLastRoom");checkRejoinAvailable();return}
  const d=s.data();
  if(d.whiteId===playerId)onlineMyColor="w";
  else if(d.blackId===playerId)onlineMyColor="b";
  else{alert("Couldn't rejoin — this room no longer belongs to you.");localStorage.removeItem("chessLastRoom");checkRejoinAvailable();return}
  listenOnline(id);document.getElementById("rejoinBanner-online").classList.add("hidden");
 }catch(e){alert("Rejoin failed: "+(e.code||e.message))}
};
document.getElementById("rejoinBtn-tourn").onclick=async()=>{
 const id=localStorage.getItem("chessLastTournId");if(!id)return;
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection.");return}
 document.getElementById("tid").value=id;openTournament(id);
 document.getElementById("rejoinBanner-tourn").classList.add("hidden");
};

/* ---- Player ID / friend invites (play a specific person instead of sharing a room code) ---- */
document.getElementById("myCodeShow").value=myCode;
document.getElementById("myNameInput").value=myName;
document.getElementById("copyMyCode").onclick=async()=>{await navigator.clipboard.writeText(myCode)};
document.getElementById("myNameInput").onchange=async(e)=>{
 myName=e.target.value.trim()||myName;localStorage.setItem("chessMyName",myName);
 if(db)await registerPlayer();
};
document.getElementById("sendInvite").onclick=async()=>{
 const target=document.getElementById("friendCode").value.trim().toUpperCase();
 if(target.length!==6){alert("Enter a 6-character Player ID.");return}
 await sendInviteToCode(target);
};
async function sendInviteToCode(target){
 if(target===myCode){alert("That's your own Player ID.");return}
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 try{
  const tSnap=await getDoc(doc(db,"players",target));
  if(!tSnap.exists()){alert("Player ID not found. Ask them to open the Online tab once so their ID registers.");return}
  const tData=tSnap.data();
  const invRef=await addDoc(collection(db,"invites"),{fromId:playerId,fromCode:myCode,fromName:myName,toId:tData.ownerId,toCode:target,toName:tData.name,status:"pending",roomCode:null,createdAt:Date.now()});
  listenMyInvite(invRef.id);
  say(onlineUI.statusEl,"Invite sent","Waiting for "+(tData.name||target)+" to accept…");
 }catch(e){console.error(e);alert("Invite failed: "+(e.code||e.message))}
}
function listenMyInvite(id){
 if(myInviteUnsub)myInviteUnsub();
 myInviteUnsub=onSnapshot(doc(db,"invites",id),s=>{
  if(!s.exists())return;
  const d=s.data();
  if(d.status==="accepted"&&d.roomCode){
   myInviteUnsub();onlineMyColor="w";document.getElementById("roomCode").value=d.roomCode;
   onlineRoomRef=doc(db,"chessRooms",d.roomCode);listenOnline(d.roomCode);
  }else if(d.status==="declined"){
   myInviteUnsub();say(onlineUI.statusEl,"Invite declined",(d.toName||d.toCode)+" declined your invite.");
  }
 });
}
function listenIncomingInvites(){
 if(incomingUnsub)incomingUnsub();
 const q=query(collection(db,"invites"),where("toId","==",playerId),where("status","==","pending"));
 incomingUnsub=onSnapshot(q,snap=>{
  const banner=document.getElementById("inviteBanner-online");
  if(snap.empty){banner.classList.add("hidden");banner.innerHTML="";return}
  banner.classList.remove("hidden");
  banner.innerHTML=snap.docs.map(d=>{
   const v=d.data();
   return `<div style="display:flex;justify-content:space-between;align-items:center;gap:8px;margin-bottom:6px"><span><b>${v.fromName||v.fromCode}</b> wants to play</span><span style="display:flex;gap:6px"><button data-acc="${d.id}" class="primary" style="padding:8px 10px">Accept</button><button data-dec="${d.id}" style="padding:8px 10px">Decline</button></span></div>`;
  }).join("");
  banner.querySelectorAll("[data-acc]").forEach(btn=>{
   const inv=snap.docs.find(d=>d.id===btn.getAttribute("data-acc")).data();
   btn.onclick=()=>acceptInvite(btn.getAttribute("data-acc"),inv);
  });
  banner.querySelectorAll("[data-dec]").forEach(btn=>{btn.onclick=()=>declineInvite(btn.getAttribute("data-dec"))});
 },e=>console.error(e));
}
async function acceptInvite(id,inv){
 try{
  const roomId=code();onlineRoomRef=doc(db,"chessRooms",roomId);
  const init={board:flattenBoard(initialBoard()),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],whiteId:inv.fromId,whiteName:inv.fromName||inv.fromCode,blackId:playerId,blackName:myName,status:"playing",winner:null,turnStartedAt:Date.now(),updatedAt:Date.now()};
  await setDoc(onlineRoomRef,init);
  await updateDoc(doc(db,"invites",id),{status:"accepted",roomCode:roomId});
  onlineMyColor="b";document.getElementById("roomCode").value=roomId;
  listenOnline(roomId);showScreen("Online");
 }catch(e){console.error(e);alert("Could not start game: "+(e.code||e.message))}
}
async function declineInvite(id){
 try{await updateDoc(doc(db,"invites",id),{status:"declined"})}catch(e){console.error(e)}
}

/* ---- Active Players popup ---- */
let activePlayersUnsub=null;
document.getElementById("showActiveBtn").onclick=async()=>{
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 document.getElementById("activeModal").classList.remove("hidden");
 document.getElementById("activeList").innerHTML="Loading…";
 if(activePlayersUnsub)activePlayersUnsub();
 const q=query(collection(db,"players"),orderBy("updatedAt","desc"),limit(50));
 activePlayersUnsub=onSnapshot(q,snap=>{
  const now=Date.now(),list=document.getElementById("activeList");
  const rows=snap.docs.filter(d=>{const v=d.data();return d.id!==myCode&&(now-(v.updatedAt||0))<90000});
  if(!rows.length){list.innerHTML='<div class="muted">No one else is active right now. Share your Player ID above so a friend can invite you!</div>';return}
  list.innerHTML=rows.map(d=>{
   const v=d.data();
   return `<div class="player" style="margin-bottom:8px"><span>🟢 <b>${v.name||d.id}</b><div class="muted" style="font-size:11px">${d.id}</div></span><button data-inv="${d.id}" class="primary" style="padding:8px 12px">Invite</button></div>`;
  }).join("");
  list.querySelectorAll("[data-inv]").forEach(btn=>{
   btn.onclick=async()=>{
    document.getElementById("activeModal").classList.add("hidden");
    if(activePlayersUnsub){activePlayersUnsub();activePlayersUnsub=null}
    await sendInviteToCode(btn.getAttribute("data-inv"));
   };
  });
 },e=>{console.error(e);document.getElementById("activeList").innerHTML='<div class="muted">Could not load active players: '+(e.code||e.message)+'</div>'});
};
document.getElementById("closeActiveModal").onclick=()=>{
 document.getElementById("activeModal").classList.add("hidden");
 if(activePlayersUnsub){activePlayersUnsub();activePlayersUnsub=null}
};
checkRejoinAvailable();

/* ============ TOURNAMENT ============ */
function initialBracket(){
 return {
  qf1:{aSeed:3,bSeed:6,winnerSeed:null,status:"pending",roomCode:null},
  qf2:{aSeed:4,bSeed:5,winnerSeed:null,status:"pending",roomCode:null},
  sf1:{aSeed:1,bSeed:null,winnerSeed:null,status:"pending",roomCode:null},
  sf2:{aSeed:2,bSeed:null,winnerSeed:null,status:"pending",roomCode:null},
  final:{aSeed:null,bSeed:null,winnerSeed:null,status:"pending",roomCode:null}
 };
}
function tid(){let s="";for(let i=0;i<5;i++)s+="ABCDEFGHJKLMNPQRSTUVWXYZ23456789"[Math.floor(Math.random()*32)];return s}
let currentTournament=null,currentTournamentId=null,tournUnsub=null;

document.getElementById("createTourn").onclick=async()=>{
 let names=[1,2,3,4,5,6].map(i=>document.getElementById("t"+i).value.trim());
 if(names.some(n=>!n)){alert("Enter all 6 player names.");return}
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 try{
  const id=tid();const ref=doc(db,"tournaments",id);
  const data={name:"6-Player Tournament",teams:names,bracket:initialBracket(),champion:null,createdAt:Date.now()};
  await setDoc(ref,data);
  openTournament(id);
 }catch(e){console.error(e);alert("Create tournament failed: "+(e.code||e.message))}
};
document.getElementById("loadTourn").onclick=async()=>{
 let id=document.getElementById("tid").value.trim().toUpperCase();
 if(!id){alert("Enter a Tournament ID.");return}
 if(!await loadFirebase()){alert("Firebase couldn't connect. Check your internet connection or Firebase setup.");return}
 openTournament(id);
};
function openTournament(id){
 currentTournamentId=id;
 localStorage.setItem("chessLastTournId",id);checkRejoinAvailable();
 document.getElementById("tournHome").classList.add("hidden");
 document.getElementById("tournView").classList.remove("hidden");
 document.getElementById("tournMatch").classList.add("hidden");
 document.getElementById("tournIdShow").textContent=id;
 if(tournUnsub)tournUnsub();
 const ref=doc(db,"tournaments",id);
 tournUnsub=onSnapshot(ref,s=>{
  if(!s.exists()){alert("Tournament not found.");backToHome();return}
  currentTournament=s.data();
  document.getElementById("tournName").textContent=currentTournament.name;
  renderBracket();
 },e=>alert("Firebase error: "+e.message));
}
function backToHome(){
 if(tournUnsub)tournUnsub();
 document.getElementById("tournHome").classList.remove("hidden");
 document.getElementById("tournView").classList.add("hidden");
 document.getElementById("tournMatch").classList.add("hidden");
}
document.getElementById("backHome").onclick=backToHome;

function teamName(seed){return seed?currentTournament.teams[seed-1]:"TBD"}
function renderBracket(){
 const t=currentTournament,b=t.bracket;
 document.getElementById("champBox").innerHTML=t.champion?`<div class="champ">🏆 Champion: ${teamName(t.champion)}</div>`:"";
 const order=[["qf1","Quarterfinal 1"],["qf2","Quarterfinal 2"],["sf1","Semifinal 1"],["sf2","Semifinal 2"],["final","Final"]];
 document.getElementById("bracketBox").innerHTML=order.map(([key,label])=>{
  const m=b[key];
  const canPlay=m.aSeed&&m.bSeed&&m.status!=="done";
  const aWin=m.winnerSeed===m.aSeed,bWin=m.winnerSeed===m.bSeed;
  return `<div class="bmatch"><div class="rd">${label}</div>
   <div class="bp2 ${aWin?"win":""}">${teamName(m.aSeed)}${aWin?" ✓":""}</div>
   <div class="bp2 ${bWin?"win":""}">${teamName(m.bSeed)}${bWin?" ✓":""}</div>
   ${canPlay?`<button data-match="${key}" class="playMatchBtn primary" style="margin-top:8px">Play This Match</button>`:(m.status==="done"?"":"<div class=\"muted\" style=\"margin-top:6px\">Waiting for players</div>")}
  </div>`;
 }).join("");
 document.querySelectorAll(".playMatchBtn").forEach(btn=>{
  btn.onclick=()=>openMatch(btn.getAttribute("data-match"));
 });
}

const tournUI=makeBoardUI("board-tourn","status-tourn","moves-tourn",null,null);
let T={board:null,turn:"w",selected:null,legal:[],flipped:false,hint:null,onClick:tournClick};
let tournRoomRef=null,tournRoomUnsub=null,tournRoom=null,tournMyColor=null,tournBusy=false,tournMatchKey=null;
let tournTurnKey=null,tournAutoMoved=false;

async function openMatch(matchKey){
 tournMatchKey=matchKey;
 const m=currentTournament.bracket[matchKey];
 document.getElementById("tournView").classList.add("hidden");
 document.getElementById("tournMatch").classList.remove("hidden");
 let roomId=m.roomCode;
 const tref=doc(db,"tournaments",currentTournamentId);
 if(!roomId){
  roomId=code();
  try{
   await runTransaction(db,async tx=>{
    const snap=await tx.get(tref);const data=snap.data();
    if(data.bracket[matchKey].roomCode){roomId=data.bracket[matchKey].roomCode;return}
    const nb={...data.bracket,[matchKey]:{...data.bracket[matchKey],roomCode:roomId,status:"live"}};
    tx.update(tref,{bracket:nb});
   });
  }catch(e){console.error(e);alert("Could not start match: "+e.message);return}
  const rref=doc(db,"chessRooms",roomId);
  const rsnap=await getDoc(rref);
  if(!rsnap.exists()){
   await setDoc(rref,{board:flattenBoard(initialBoard()),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],
    whiteId:null,whiteName:teamName(m.aSeed),blackId:null,blackName:teamName(m.bSeed),
    status:"waiting",winner:null,tournamentId:currentTournamentId,matchKey,whiteSeed:m.aSeed,blackSeed:m.bSeed,turnStartedAt:Date.now(),updatedAt:Date.now()});
  }
 }
 tournRoomRef=doc(db,"chessRooms",roomId);
 let s=await getDoc(tournRoomRef);let d=s.data();
 if(d.whiteId===playerId)tournMyColor="w";
 else if(d.blackId===playerId)tournMyColor="b";
 else if(!d.whiteId){await updateDoc(tournRoomRef,{whiteId:playerId,updatedAt:Date.now()});tournMyColor="w"}
 else if(!d.blackId){await updateDoc(tournRoomRef,{blackId:playerId,status:"playing",turnStartedAt:Date.now(),updatedAt:Date.now()});tournMyColor="b"}
 else{tournMyColor=null} // spectator
 listenTournMatch();
}
function listenTournMatch(){
 if(tournRoomUnsub)tournRoomUnsub();
 tournRoomUnsub=onSnapshot(tournRoomRef,async s=>{
  if(!s.exists())return;
  tournRoom=s.data();T.board=unflattenBoard(tournRoom.board);T.turn=tournRoom.turn;T.history=tournRoom.history||[];T.selected=null;T.legal=[];
  const turnKey=tournRoom.turn+"-"+(tournRoom.history?tournRoom.history.length:0);
  if(turnKey!==tournTurnKey){tournTurnKey=turnKey;tournAutoMoved=false}
  document.querySelector("#tourn-white b").textContent=(tournRoom.whiteName||"Waiting")+(tournMyColor==="w"?" (You)":"");
  document.querySelector("#tourn-black b").textContent=(tournRoom.blackName||"Waiting")+(tournMyColor==="b"?" (You)":"");
  if(tournRoom.winner){
   say(tournUI.statusEl,tournRoom.winner==="draw"?"Draw":(tournRoom.winner==="w"?tournRoom.whiteName:tournRoom.blackName)+" wins","Game over");
   await recordMatchResult(tournRoom.winner);
  }else if(tournRoom.status==="waiting")say(tournUI.statusEl,"Waiting for opponent","");
  else say(tournUI.statusEl,tournRoom.turn===tournMyColor?"Your turn":(tournMyColor?"Opponent's turn":"Spectating"),tournRoom.turn==="w"?"White to move":"Black to move");
  renderBoard(tournUI,T);
  updateTournTimerDisplay();
 });
}
function updateTournTimerDisplay(){
 const el=document.getElementById("timer-tourn");if(!el)return;
 if(!tournRoom||tournRoom.winner||tournRoom.status!=="playing"){el.textContent="";return}
 const elapsed=Math.floor((Date.now()-(tournRoom.turnStartedAt||Date.now()))/1000);
 const left=Math.max(0,ONLINE_SECONDS-elapsed);
 const whoseTurn=tournRoom.turn===tournMyColor?"Your":(tournRoom.turn==="w"?"White's":"Black's");
 el.textContent="⏱ "+whoseTurn+" time: "+left+"s";
 if(left<=0&&tournMyColor&&tournRoom.turn===tournMyColor&&!tournAutoMoved&&!tournBusy){tournAutoMoved=true;autoPlayTourn()}
}
async function autoPlayTourn(){
 let m=bestMove(T.board,tournMyColor,tournRoom.castling,tournRoom.ep,1);
 if(!m){let ms=legalMoves(T.board,tournMyColor,tournRoom.castling,tournRoom.ep);if(ms.length)m=ms[Math.floor(Math.random()*ms.length)]}
 if(m)await sendTournMove(m);
}
async function recordMatchResult(winnerColor){
 if(winnerColor==="draw")return; // draws not auto-advanced; organizer can replay by re-opening match
 const winnerSeed=winnerColor==="w"?tournRoom.whiteSeed:tournRoom.blackSeed;
 const tref=doc(db,"tournaments",currentTournamentId);
 try{
  await runTransaction(db,async tx=>{
   const snap=await tx.get(tref);const data=snap.data();
   const bkt=data.bracket;
   if(bkt[tournMatchKey].status==="done")return; // already recorded
   bkt[tournMatchKey]={...bkt[tournMatchKey],winnerSeed,status:"done"};
   let champion=data.champion;
   if(tournMatchKey==="qf1")bkt.sf2.bSeed=winnerSeed;
   if(tournMatchKey==="qf2")bkt.sf1.bSeed=winnerSeed;
   if(tournMatchKey==="sf1")bkt.final.aSeed=winnerSeed;
   if(tournMatchKey==="sf2")bkt.final.bSeed=winnerSeed;
   if(tournMatchKey==="final")champion=winnerSeed;
   tx.update(tref,{bracket:bkt,champion});
  });
 }catch(e){console.error(e)}
}
function tournClick(r,c){
 if(!tournRoom||tournRoom.status!=="playing"||tournRoom.turn!==tournMyColor||tournBusy)return;
 T.hint=null;
 let p=T.board[r][c],m=T.legal.find(x=>x.tr===r&&x.tc===c);
 if(T.selected&&m){optimisticTournMove(m);return}
 if(p&&pcolor(p)===tournMyColor){T.selected=[r,c];T.legal=legalMoves(T.board,tournMyColor,tournRoom.castling,tournRoom.ep).filter(x=>x.fr===r&&x.fc===c);renderBoard(tournUI,T)}
 else{T.selected=null;T.legal=[];renderBoard(tournUI,T)}
}
function optimisticTournMove(m){
 const ap=applyMove(T.board,m,tournRoom.castling,tournRoom.ep);
 T.board=ap.nb;T.selected=null;T.legal=[];T.hint=null;
 renderBoard(tournUI,T);
 say(tournUI.statusEl,"Opponent's turn","Move sent…");
 sendTournMove(m);
}
async function sendTournMove(m){
 if(tournBusy)return;tournBusy=true;
 try{
  await runTransaction(db,async tx=>{
   const snap=await tx.get(tournRoomRef);if(!snap.exists())throw Error("Room does not exist");
   const d=snap.data();if(d.turn!==tournMyColor)throw Error("Not your turn");
   const b=unflattenBoard(d.board),cast={...d.castling},ep=d.ep;
   const lm=legalMoves(b,tournMyColor,cast,ep).find(x=>JSON.stringify(x)===JSON.stringify(m));if(!lm)throw Error("Illegal move");
   const note=notation(b,lm,cast,ep),ap=applyMove(b,lm,cast,ep);
   let hist=[...(d.history||[]),note],next=opp(tournMyColor),ms=legalMoves(ap.nb,next,ap.nc,ap.nep),winner=null,status="playing";
   if(!ms.length){status="finished";winner=inCheck(ap.nb,next)?tournMyColor:"draw"}
   tx.update(tournRoomRef,{board:flattenBoard(ap.nb),castling:ap.nc,ep:ap.nep,turn:next,history:hist,status,winner,turnStartedAt:Date.now(),updatedAt:Date.now()});
  });
 }catch(e){console.error(e);alert(e.message)}finally{tournBusy=false}
}
document.getElementById("flip-tourn").onclick=()=>{T.flipped=!T.flipped;renderBoard(tournUI,T)};
document.getElementById("hint-tourn").onclick=()=>{
 if(!tournRoom||tournRoom.status!=="playing"||tournRoom.turn!==tournMyColor)return;
 let m=bestMove(T.board,tournMyColor,tournRoom.castling,tournRoom.ep,2);
 if(m){T.hint=m;renderBoard(tournUI,T);say(tournUI.statusEl,"Your turn","Hint: "+FILES[m.fc]+(8-m.fr)+" → "+FILES[m.tc]+(8-m.tr))}
};
document.getElementById("backBracket").onclick=()=>{
 if(tournRoomUnsub)tournRoomUnsub();
 document.getElementById("tournMatch").classList.add("hidden");
 document.getElementById("tournView").classList.remove("hidden");
};

/* ============ NAV ============ */
function showScreen(name){
 document.querySelectorAll(".screen").forEach(s=>s.classList.remove("active"));
 document.querySelectorAll(".nav button").forEach(b=>b.classList.remove("active"));
 document.getElementById("scr"+name).classList.add("active");
 document.getElementById("nav"+name).classList.add("active");
}
document.getElementById("navLocal").onclick=()=>showScreen("Local");
document.getElementById("navOnline").onclick=()=>showScreen("Online");
document.getElementById("navTourn").onclick=()=>showScreen("Tourn");

/* ============ INIT ============ */
newLocalGame();
document.getElementById("net").textContent="Firebase: not connected yet";
loadFirebase(); // background load; Quick Play never waits on this
</script>
</body>
</html>
