# How-to-bring-chess-
team 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>Chess Arena — Tournament</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#0b1220;color:#eef2f7;font-family:Inter,system-ui,Arial,sans-serif}
.app{max-width:1250px;width:96%;margin:18px auto}.top{display:flex;justify-content:space-between;align-items:center;gap:12px;margin-bottom:16px}
h1{margin:0;font-size:30px}.muted{color:#94a3b8;font-size:13px}.grid{display:grid;grid-template-columns:minmax(330px,760px) 360px;gap:16px}
.card{background:#172033;border:1px solid #2b3950;border-radius:18px;box-shadow:0 12px 40px #0005}.boardCard{padding:12px}.board{display:grid;grid-template-columns:repeat(8,1fr);aspect-ratio:1;border-radius:12px;overflow:hidden;user-select:none}
.sq{position:relative;display:flex;justify-content:center;align-items:center;cursor:pointer}.light{background:#f0d9b5}.dark{background:#b58863}
.sq.sel{box-shadow:inset 0 0 0 5px #facc15}.sq.last{background-image:linear-gradient(#facc1555,#facc1555)}.sq.check{background-image:linear-gradient(#ef4444aa,#ef4444aa)}
.sq.move:after{content:"";position:absolute;width:20%;height:20%;border-radius:50%;background:#16a34a99}.sq.capture:after{content:"";position:absolute;inset:7%;border:5px solid #16a34a99;border-radius:50%}
.piece{font-family:"Segoe UI Symbol","Noto Sans Symbols 2",serif;font-size:clamp(34px,7.3vw,68px);line-height:1;z-index:2}.wp{color:#fff;text-shadow:0 2px 2px #222}.bp{color:#111;text-shadow:0 1px 1px #fff}
.coord{position:absolute;font-size:10px;font-weight:700;opacity:.65}.rank{top:3px;left:4px}.file{bottom:3px;right:4px}.light .coord{color:#765536}.dark .coord{color:#f7dfb8}
.panel{padding:15px;display:flex;flex-direction:column;gap:9px}.status{background:#0e1726;border:1px solid #2e3c52;border-radius:12px;padding:11px}.status b{font-size:16px}.status div{color:#94a3b8;font-size:12px;margin-top:3px}
input,button,select{font:inherit;border:0;border-radius:9px;padding:10px 11px}input,select{background:#0e1726;color:#fff;border:1px solid #35445b;width:100%}button{background:#334155;color:#fff;cursor:pointer;font-weight:650}button:hover{background:#475569}.primary{background:#22c55e;color:#05220e}.danger{background:#b91c1c}.small{font-size:12px;padding:8px 10px}
.row2{display:grid;grid-template-columns:1fr 1fr;gap:7px}.row3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:7px}.room{display:grid;grid-template-columns:1fr auto;gap:7px}.players{display:grid;gap:6px}.player{padding:8px 10px;background:#0e1726;border-radius:9px;display:flex;justify-content:space-between;font-size:13px}.you{border:1px solid #22c55e66}
.title{font-size:11px;text-transform:uppercase;color:#94a3b8;letter-spacing:.09em;margin-top:5px}.moves{height:170px;overflow:auto;background:#0e1726;border-radius:9px;padding:6px}.moveRow{display:grid;grid-template-columns:35px 1fr 1fr;padding:4px 6px;border-bottom:1px solid #223047;font-size:12px}
.timer{font-size:17px}.activeTimer{border:1px solid #facc15}.online{color:#22c55e}.offline{color:#ef4444}.hidden{display:none!important}
.tableWrap{overflow:auto}.standings{width:100%;border-collapse:collapse;font-size:12px}.standings th,.standings td{padding:6px;border-bottom:1px solid #2b3950;text-align:left}.standings th{color:#94a3b8}
.badge{display:inline-block;padding:3px 7px;border-radius:999px;background:#26344a;font-size:10px}.ok{background:#14532d}.warn{background:#713f12}.err{background:#7f1d1d}
#tournamentPanel{max-height:390px;overflow:auto}.fixture{background:#0e1726;border-radius:9px;padding:8px;margin:5px 0;font-size:12px}.fixture button{float:right;padding:4px 7px;font-size:10px}.bracket{display:grid;gap:6px}.champ{font-size:18px;text-align:center;padding:10px;background:#713f12;border-radius:10px}
.note{font-size:11px;color:#94a3b8;line-height:1.45}@media(max-width:950px){.grid{grid-template-columns:1fr}.panel{display:grid;grid-template-columns:1fr 1fr}.panel>*{min-width:0}.wide{grid-column:span 2}}@media(max-width:600px){.panel{grid-template-columns:1fr}.wide{grid-column:auto}.app{width:98%;margin:7px auto}.boardCard{padding:6px}.piece{font-size:clamp(28px,11vw,46px)}}
</style>
</head>
<body>
<div class="app">
  <div class="top">
    <div><h1>♟ Chess Arena</h1><div class="muted">Firebase multiplayer • Teams • League • Playoffs</div></div>
    <div id="net" class="online">● Connecting</div>
  </div>
  <div class="grid">
    <div class="card boardCard"><div id="board" class="board"></div></div>

    <div class="card panel">
      <div id="status" class="status wide"><b>Ready</b><div>Create or join a chess room.</div></div>

      <div class="title wide">Quick Chess</div>
      <div class="room wide"><input id="roomCode" maxlength="6" placeholder="ROOM CODE"><button id="copy">Copy</button></div>
      <div class="row2 wide"><button id="create" class="primary">Create Room</button><button id="join">Join Room</button></div>
      <div class="row2 wide"><button id="computer" class="primary">Play Computer</button><button id="hint">💡 Hint</button></div>
      <div class="row2 wide"><button id="resign" class="danger">Resign</button><button id="flip">Flip Board</button></div>

      <div class="players wide">
        <div class="player" id="white"><span>♔ White</span><b>Waiting</b></div>
        <div class="player" id="black"><span>♚ Black</span><b>Waiting</b></div>
      </div>
      <div class="row2 wide">
        <div class="player" id="timerWhite"><span>♔</span><b class="timer">30s</b></div>
        <div class="player" id="timerBlack"><span>♚</span><b class="timer">30s</b></div>
      </div>
      <div id="hintBox" class="status wide hidden"><b>💡 Hint</b><div id="hintText"></div></div>

      <div class="title wide">Player Account</div>
      <input id="email" type="email" placeholder="Email" class="wide">
      <input id="password" type="password" placeholder="Password (6+ characters)" class="wide">
      <div class="row3 wide"><button id="signup">Sign Up</button><button id="signin">Login</button><button id="signout">Logout</button></div>
      <div id="accountStatus" class="status wide"><b>Not logged in</b><div>Use your own account for tournament play.</div></div>

      <div class="title wide">Team</div>
      <input id="teamName" maxlength="24" placeholder="New team name" class="wide">
      <div class="row2 wide"><button id="createTeam" class="primary">Create Team</button><input id="teamCode" maxlength="8" placeholder="TEAM CODE"></div>
      <button id="joinTeam" class="wide">Join Team</button>
      <div id="teamStatus" class="status wide"><b>No team</b><div>Create or join a team.</div></div>

      <div class="title wide">Tournament</div>
      <div class="row2 wide"><button id="registerTeam" class="primary">Register Team</button><button id="startTournament">Start Tournament</button></div>
      <button id="refreshTournament" class="wide">Refresh Tournament</button>
      <div id="tournamentPanel" class="status wide"><b>No tournament</b><div>Register teams when ready.</div></div>

      <div class="title wide">Move History</div>
      <div id="moves" class="moves wide"></div>
      <div class="note wide">Each player gets <b>30 seconds per turn</b>. Tournament scoring: Win 3, Draw 1, Loss 0. League is round-robin; top 4 enter semi-finals (1st vs 4th, 2nd vs 3rd), then final.</div>
    </div>
  </div>
</div>

<script type="module">
import {initializeApp} from "https://www.gstatic.com/firebasejs/12.18.0/firebase-app.js";
import {getFirestore,doc,getDoc,setDoc,updateDoc,onSnapshot,runTransaction,collection,getDocs,query,where} from "https://www.gstatic.com/firebasejs/12.18.0/firebase-firestore.js";
import {getAuth,createUserWithEmailAndPassword,signInWithEmailAndPassword,signOut,onAuthStateChanged} from "https://www.gstatic.com/firebasejs/12.18.0/firebase-auth.js";

const firebaseConfig={
 apiKey:"AIzaSyCtRpxAiAdl0RIpLbx3JuuD5aO_gGYJnNY",
 authDomain:"shuklareports.firebaseapp.com",
 projectId:"shuklareports",
 storageBucket:"shuklareports.firebasestorage.app",
 messagingSenderId:"133308067679",
 appId:"1:133308067679:web:a921e81c720178a0bdd0ea",
 measurementId:"G-0VF9CRV1R4"
};

const app=initializeApp(firebaseConfig),db=getFirestore(app),auth=getAuth(app);
const TURN=30000, files="abcdefgh";
const PIECES={K:"♔",Q:"♕",R:"♖",B:"♗",N:"♘",P:"♙",k:"♚",q:"♛",r:"♜",b:"♝",n:"♞",p:"♟"};
const V={P:100,N:320,B:330,R:500,Q:900,K:20000};

let user=null,profile=null,team=null,tournament=null;
let roomRef=null,room=null,myColor=null,unsubscribe=null,tournamentUnsub=null;
let board=null,selected=null,legal=[],flipped=false,busy=false,gameMode="multi",aiBusy=false,timer=null;

const $=id=>document.getElementById(id);
const initial=()=>[
 ["r","n","b","q","k","b","n","r"],["p","p","p","p","p","p","p","p"],
 [null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],
 [null,null,null,null,null,null,null,null], [null,null,null,null,null,null,null,null],
 ["P","P","P","P","P","P","P","P"],["R","N","B","Q","K","B","N","R"]
];
const color=p=>p&&p===p.toUpperCase()?"w":"b", opp=c=>c==="w"?"b":"w";
const ib=(r,c)=>r>=0&&r<8&&c>=0&&c<8;
const king=(b,c)=>{const k=c==="w"?"K":"k";for(let r=0;r<8;r++)for(let c2=0;c2<8;c2++)if(b[r][c2]===k)return[r,c2];return null};

function attacked(b,r,c,by){
 let p=by==="w"?"P":"p",pr=r+(by==="w"?1:-1);
 for(const dc of[-1,1])if(ib(pr,c+dc)&&b[pr][c+dc]===p)return true;
 p=by==="w"?"N":"n";
 for(const [dr,dc] of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])if(ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 p=by==="w"?"K":"k";
 for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if((dr||dc)&&ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 for(const [dr,dc,t] of[[1,0,"R"],[-1,0,"R"],[0,1,"R"],[0,-1,"R"],[1,1,"B"],[1,-1,"B"],[-1,1,"B"],[-1,-1,"B"]]){
   let rr=r+dr,cc=c+dc;
   while(ib(rr,cc)){const p2=b[rr][cc];if(p2){if(color(p2)===by&&(p2.toUpperCase()===t||p2.toUpperCase()==="Q"))return true;break}rr+=dr;cc+=dc}
 }
 return false;
}
const check=(b,c)=>{const k=king(b,c);return !k||attacked(b,k[0],k[1],opp(c))};

function pseudo(b,c,cast,ep){
 const out=[],add=(fr,fc,tr,tc,e={})=>{if(!ib(tr,tc))return;const t=b[tr][tc];if(t&&color(t)===c)return;if(t&&t.toUpperCase()==="K")return;out.push({fr,fc,tr,tc,...e})};
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){
  const p=b[r][x];if(!p||color(p)!==c)continue;const u=p.toUpperCase();
  if(u==="P"){
   const d=c==="w"?-1:1,start=c==="w"?6:1,last=c==="w"?0:7;
   if(ib(r+d,x)&&!b[r+d][x]){
    if(r+d===last)for(const q of["Q","R","B","N"])out.push({fr:r,fc:x,tr:r+d,tc:x,promotion:c==="w"?q:q.toLowerCase()});
    else add(r,x,r+d,x);
    if(r===start&&!b[r+2*d][x])add(r,x,r+2*d,x);
   }
   for(const dc of[-1,1]){
    const tr=r+d,tc=x+dc;if(!ib(tr,tc))continue;
    if(b[tr][tc]&&color(b[tr][tc])!==c&&b[tr][tc].toUpperCase()!=="K"){
      if(tr===last)for(const q of["Q","R","B","N"])out.push({fr:r,fc:x,tr,tc,promotion:c==="w"?q:q.toLowerCase()});
      else add(r,x,tr,tc);
    }
    if(ep&&ep[0]===tr&&ep[1]===tc)out.push({fr:r,fc:x,tr,tc,ep:true});
   }
  }else if(u==="N"){
   for(const[dr,dc]of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])add(r,x,r+dr,x+dc);
  }else if(u==="K"){
   for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if(dr||dc)add(r,x,r+dr,x+dc);
   const home=c==="w"?7:0;
   if(r===home&&x===4&&!check(b,c)){
    const ks=c==="w"?cast.wK:cast.bK,qs=c==="w"?cast.wQ:cast.bQ;
    if(ks&&!b[home][5]&&!b[home][6]&&!attacked(b,home,5,opp(c))&&!attacked(b,home,6,opp(c))&&b[home][7]===(c==="w"?"R":"r"))out.push({fr:r,fc:x,tr:home,tc:6,castle:"K"});
    if(qs&&!b[home][1]&&!b[home][2]&&!b[home][3]&&!attacked(b,home,3,opp(c))&&!attacked(b,home,2,opp(c))&&b[home][0]===(c==="w"?"R":"r"))out.push({fr:r,fc:x,tr:home,tc:2,castle:"Q"});
   }
  }else{
   const ds=u==="R"?[[1,0],[-1,0],[0,1],[0,-1]]:u==="B"?[[1,1],[1,-1],[-1,1],[-1,-1]]:[[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]];
   for(const[dr,dc]of ds){let rr=r+dr,cc=x+dc;while(ib(rr,cc)){const t=b[rr][cc];if(!t)add(r,x,rr,cc);else{if(color(t)!==c&&t.toUpperCase()!=="K")add(r,x,rr,cc);break}rr+=dr;cc+=dc}}
  }
 }
 return out;
}
function apply(b,m,cast,ep){
 const p=b[m.fr][m.fc],t=b[m.tr][m.tc];b[m.tr][m.tc]=m.promotion||p;b[m.fr][m.fc]=null;
 if(m.ep)b[m.tr+(color(p)==="w"?1:-1)][m.tc]=null;
 if(m.castle){const r=m.fr;if(m.tc===6){b[r][5]=b[r][7];b[r][7]=null}else{b[r][3]=b[r][0];b[r][0]=null}}
 if(p==="K")cast.wK=cast.wQ=false;if(p==="k")cast.bK=cast.bQ=false;
 if(p==="R"){if(m.fr===7&&m.fc===0)cast.wQ=false;if(m.fr===7&&m.fc===7)cast.wK=false}
 if(p==="r"){if(m.fr===0&&m.fc===0)cast.bQ=false;if(m.fr===0&&m.fc===7)cast.bK=false}
 if(t==="R"){if(m.tr===7&&m.tc===0)cast.wQ=false;if(m.tr===7&&m.tc===7)cast.wK=false}
 if(t==="r"){if(m.tr===0&&m.tc===0)cast.bQ=false;if(m.tr===0&&m.tc===7)cast.bK=false}
 return p.toUpperCase()==="P"&&Math.abs(m.tr-m.fr)===2?[(m.tr+m.fr)/2,m.fc]:null;
}
function legalMoves(b,c,cast,ep){
 const out=[];
 for(const m of pseudo(b,c,cast,ep)){const x=b.map(a=>a.slice()),cc={...cast};apply(x,m,cc,ep);if(!check(x,c))out.push(m)}
 return out;
}
function notation(b,m){
 const p=b[m.fr][m.fc],t=b[m.tr][m.tc];if(m.castle)return m.tc===6?"O-O":"O-O-O";
 let s=p.toUpperCase()==="P"?"":p.toUpperCase();
 if(t||m.ep)s+=(p.toUpperCase()==="P"?files[m.fc]:"")+"x";
 s+=files[m.tc]+(8-m.tr);if(m.promotion)s+="="+m.promotion.toUpperCase();return s;
}
const sameMove=(a,b)=>a.fr===b.fr&&a.fc===b.fc&&a.tr===b.tr&&a.tc===b.tc&&a.promotion===b.promotion&&!!a.ep===!!b.ep;

function roomBase(extra={}){
 return {board:initial(),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],lastMove:null,
  whiteId:null,whiteName:null,blackId:null,blackName:null,status:"waiting",winner:null,
  whiteTime:TURN,blackTime:TURN,turnStartedAt:Date.now(),updatedAt:Date.now(),...extra};
}
function code(){return Math.random().toString(36).slice(2,8).toUpperCase()}
function teamCode(){return "T"+Math.random().toString(36).slice(2,8).toUpperCase()}
function esc(s){return String(s??"").replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#39;"}[m]))}
function msg(title,detail){$("status").innerHTML=`<b>${esc(title)}</b><div>${esc(detail)}</div>`}
function tournamentMsg(title,detail){$("tournamentPanel").innerHTML=`<b>${esc(title)}</b><div>${esc(detail)}</div>`}

function render(){
 const el=$("board");el.innerHTML="";if(!board)return;
 const rs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7],cs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7];
 legal=selected&&room&&room.status==="playing"?legalMoves(board,myColor,room.castling,room.ep).filter(m=>m.fr===selected[0]&&m.fc===selected[1]):[];
 for(let ri=0;ri<8;ri++)for(let ci=0;ci<8;ci++){
  const r=rs[ri],c=cs[ci],s=document.createElement("div");s.className="sq "+((r+c)%2?"dark":"light");
  if(room?.lastMove&&((room.lastMove.fr===r&&room.lastMove.fc===c)||(room.lastMove.tr===r&&room.lastMove.tc===c)))s.classList.add("last");
  if(selected?.[0]===r&&selected?.[1]===c)s.classList.add("sel");
  const lm=legal.find(m=>m.tr===r&&m.tc===c);if(lm)s.classList.add(board[r][c]?"capture":"move");
  const k=room&&king(board,room.turn);if(k?.[0]===r&&k?.[1]===c&&check(board,room.turn))s.classList.add("check");
  if(board[r][c]){const z=document.createElement("span");z.className="piece "+(color(board[r][c])==="w"?"wp":"bp");z.textContent=PIECES[board[r][c]];s.appendChild(z)}
  if(ci===0){const z=document.createElement("span");z.className="coord rank";z.textContent=8-r;s.appendChild(z)}
  if(ri===7){const z=document.createElement("span");z.className="coord file";z.textContent=files[c];s.appendChild(z)}
  s.onclick=()=>clickSquare(r,c);el.appendChild(s);
 }
 $("moves").innerHTML=(room?.history||[]).map((x,i)=>i%2===0?`<div class="moveRow"><span>${Math.floor(i/2)+1}.</span><span>${esc(x)}</span><span>${esc(room.history[i+1]||"")}</span></div>`:"").join("");
}
function renderTimers(){
 if(!room){$("timerWhite").querySelector("b").textContent="30s";$("timerBlack").querySelector("b").textContent="30s";return}
 let wt=room.whiteTime??TURN,bt=room.blackTime??TURN;
 if(room.status==="playing"){const e=Math.max(0,Date.now()-(room.turnStartedAt||Date.now()));if(room.turn==="w")wt=Math.max(0,wt-e);else bt=Math.max(0,bt-e)}
 $("timerWhite").querySelector("b").textContent=Math.ceil(wt/1000)+"s";$("timerBlack").querySelector("b").textContent=Math.ceil(bt/1000)+"s";
 $("timerWhite").classList.toggle("activeTimer",room.status==="playing"&&room.turn==="w");
 $("timerBlack").classList.toggle("activeTimer",room.status==="playing"&&room.turn==="b");
}
function updatePlayers(){
 $("white").querySelector("b").textContent=(room?.whiteName||"Waiting")+(myColor==="w"?" (You)":"");
 $("black").querySelector("b").textContent=(room?.blackName||"Waiting")+(myColor==="b"?" (You)":"");
}
function updateStatus(){
 if(!room)return;
 if(room.winner){msg(room.winner==="draw"?"Draw":room.winner==="w"?"White wins":"Black wins","Game over");return}
 if(room.status==="waiting"){msg("Waiting for opponent","Share room code: "+$("roomCode").value);return}
 msg(room.turn===myColor?"Your turn":"Opponent's turn",room.turn==="w"?"White to move":"Black to move");
}
function startTimer(){if(timer)clearInterval(timer);timer=setInterval(clockTick,250);renderTimers()}
async function clockTick(){
 renderTimers();
 if(!room||room.status!=="playing"||!roomRef||busy)return;
 const e=Math.max(0,Date.now()-(room.turnStartedAt||Date.now()));
 const key=room.turn==="w"?"whiteTime":"blackTime";
 if((room[key]??TURN)-e<=0){
  try{await runTransaction(db,async tx=>{
   const s=await tx.get(roomRef);if(!s.exists())return;const d=s.data();if(d.status!=="playing")return;
   const rem=(d[d.turn==="w"?"whiteTime":"blackTime"]??TURN)-Math.max(0,Date.now()-(d.turnStartedAt||Date.now()));
   if(rem<=0)tx.update(roomRef,{status:"finished",winner:opp(d.turn),[d.turn==="w"?"whiteTime":"blackTime"]:0,updatedAt:Date.now()});
  })}catch(e){console.error(e)}
 }
}

async function createRoom(){
 if(!user){alert("Login first.");return}
 const id=code();roomRef=doc(db,"chessRooms",id);myColor="w";gameMode="multi";
 await setDoc(roomRef,roomBase({whiteId:user.uid,whiteName:profile?.name||user.email,status:"waiting"}));
 $("roomCode").value=id;listenRoom(id);
}
async function joinRoom(){
 if(!user){alert("Login first.");return}
 const id=$("roomCode").value.trim().toUpperCase();if(id.length!==6){alert("Enter a 6-character room code.");return}
 const ref=doc(db,"chessRooms",id),s=await getDoc(ref);if(!s.exists()){alert("Room not found.");return}
 const d=s.data();roomRef=ref;
 if(d.whiteId===user.uid)myColor="w";
 else if(d.blackId===user.uid)myColor="b";
 else if(!d.blackId){myColor="b";await updateDoc(ref,{blackId:user.uid,blackName:profile?.name||user.email,status:"playing",turnStartedAt:Date.now(),whiteTime:TURN,blackTime:TURN,updatedAt:Date.now()})}
 else{alert("Room already has two players.");return}
 gameMode="multi";listenRoom(id);
}
function listenRoom(id){
 if(unsubscribe)unsubscribe();
 let previousStatus=null, previousWinner=null;
 unsubscribe=onSnapshot(roomRef,s=>{
  if(!s.exists()){msg("Room closed","");return}
  const d=s.data();
  room=d;board=room.board;render();updatePlayers();updateStatus();renderTimers();
  if(gameMode==="computer")computerTurn();
  if(d.status==="finished" && d.winner && (previousStatus!=="finished" || previousWinner!==d.winner)){
    previousStatus=d.status;previousWinner=d.winner;
    checkTournamentResult(d).catch(console.error);
  }else{previousStatus=d.status;previousWinner=d.winner||null}
 },e=>msg("Firebase error",e.message));
}
async function sendMove(m){
 if(busy||!room||room.status!=="playing"||room.turn!==myColor)return;
 busy=true;
 try{await runTransaction(db,async tx=>{
  const s=await tx.get(roomRef);if(!s.exists())throw Error("Room does not exist");const d=s.data();
  if(d.status!=="playing"||d.turn!==myColor)throw Error("Not your turn");
  const elapsed=Math.max(0,Date.now()-(d.turnStartedAt||Date.now())),key=d.turn==="w"?"whiteTime":"blackTime",remaining=(d[key]??TURN)-elapsed;
  if(remaining<=0){tx.update(roomRef,{status:"finished",winner:opp(d.turn),[key]:0,updatedAt:Date.now()});return}
  const b=d.board.map(a=>a.slice()),cast={...d.castling},lm=legalMoves(b,myColor,cast,d.ep).find(x=>sameMove(x,m));
  if(!lm)throw Error("Illegal move");
  const note=notation(b,lm),nep=apply(b,lm,cast,d.ep),next=opp(myColor),ms=legalMoves(b,next,cast,nep);
  const hist=[...(d.history||[]),note];let status="playing",winner=null;
  if(!ms.length){status="finished";winner=check(b,next)?myColor:"draw"}
  tx.update(roomRef,{board:b,castling:cast,ep:nep,turn:next,history:hist,lastMove:lm,status,winner,
   [key]:remaining,turnStartedAt:Date.now(),updatedAt:Date.now()});
 })}catch(e){alert(e.message)}finally{busy=false}
}
function clickSquare(r,c){
 if(!room||room.status!=="playing"||room.turn!==myColor||gameMode==="computer"&&myColor==="b")return;
 const p=board[r][c],m=legal.find(x=>x.tr===r&&x.tc===c);
 if(selected&&m){sendMove(m);selected=null;return}
 if(p&&color(p)===myColor){selected=[r,c];render()}else{selected=null;render()}
}
function evaluate(b){let s=0;for(const row of b)for(const p of row)if(p)s+=(color(p)==="b"?1:-1)*(V[p.toUpperCase()]||0);return s}
function aiSearch(b,c,cast,ep,depth,alpha=-Infinity,beta=Infinity){
 const moves=legalMoves(b,c,cast,ep);
 if(!moves.length)return {score:check(b,c)?(c==="b"?-999999:999999):0,move:null};
 if(depth===0)return {score:evaluate(b),move:null};
 let best=null;
 for(const m of moves){const x=b.map(a=>a.slice()),cc={...cast};const ne=apply(x,m,cc,ep);const ch=aiSearch(x,opp(c),cc,ne,depth-1,alpha,beta);const score=ch.score;
  if(!best||(c==="b"?score>best.score:score<best.score))best={score,move:m};
  if(c==="b")alpha=Math.max(alpha,score);else beta=Math.min(beta,score);if(beta<=alpha)break;
 }
 return best;
}
function bestComputerMove(){return room?aiSearch(room.board,"b",{...room.castling},room.ep,2).move:null}
async function startComputerGame(){
 if(!user){alert("Login first.");return}
 if(unsubscribe){unsubscribe();unsubscribe=null}
 gameMode="computer";myColor="w";roomRef=null;selected=null;
 room=roomBase({whiteId:user.uid,whiteName:profile?.name||"You",blackId:"COMPUTER",blackName:"Computer",status:"playing"});
 board=room.board;render();updatePlayers();updateStatus();renderTimers();
}
async function computerTurn(){
 if(gameMode!=="computer"||!room||room.status!=="playing"||room.turn!=="b"||aiBusy)return;
 aiBusy=true;await new Promise(r=>setTimeout(r,250));
 try{
  const m=bestComputerMove();if(!m)return;
  const b=room.board.map(a=>a.slice()),cast={...room.castling},nep=apply(b,m,cast,room.ep),next="w",ms=legalMoves(b,next,cast,nep);
  room={...room,board:b,castling:cast,ep:nep,turn:next,history:[...(room.history||[]),notation(room.board,m)],lastMove:m,status:ms.length?"playing":"finished",winner:ms.length?null:(check(b,next)?"b":"draw"),whiteTime:TURN,blackTime:TURN,turnStartedAt:Date.now()};
  board=b;render();updateStatus();renderTimers();
 }finally{aiBusy=false}
}
function hint(){
 const box=$("hintBox"),text=$("hintText");
 if(!room||room.status!=="playing"||!myColor){box.classList.remove("hidden");text.textContent="Start a game first.";return}
 const m=aiSearch(board,myColor,{...room.castling},room.ep,2).move;
 if(!m){box.classList.remove("hidden");text.textContent="No legal move.";return}
 selected=[m.fr,m.fc];render();box.classList.remove("hidden");text.textContent=`Move ${files[m.fc]}${8-m.fr} → ${files[m.tc]}${8-m.tr}.`;
}

async function loadProfile(){
 if(!user)return;
 const ref=doc(db,"chessPlayers",user.uid),s=await getDoc(ref);
 if(s.exists())profile=s.data();else{profile={name:user.email.split("@")[0],email:user.email};await setDoc(ref,{...profile,createdAt:Date.now()})}
 $("accountStatus").innerHTML=`<b>Logged in</b><div>${esc(profile.name)} • ${esc(user.email)}</div>`;
 await loadMyTeam();await loadTournament();
}
async function signup(){
 try{await createUserWithEmailAndPassword(auth,$("email").value.trim(),$("password").value)}catch(e){alert(e.message)}
}
async function signin(){
 try{await signInWithEmailAndPassword(auth,$("email").value.trim(),$("password").value)}catch(e){alert(e.message)}
}
async function logout(){await signOut(auth);location.reload()}

async function loadMyTeam(){
 if(!user)return;
 const q=query(collection(db,"chessTournamentTeams"),where("memberIds","array-contains",user.uid)),s=await getDocs(q);
 team=s.empty?null:{id:s.docs[0].id,...s.docs[0].data()};
 if(team){$("teamCode").value=team.code;$("teamStatus").innerHTML=`<b>${esc(team.name)}</b><div>${team.memberIds.length}/6 players • Code: ${esc(team.code)}</div><div>${(team.members||[]).map(x=>esc(x.name)).join(" • ")}</div>`}
 else $("teamStatus").innerHTML="<b>No team</b><div>Create or join a team.</div>";
}
async function createTeam(){
 if(!user){alert("Login first.");return} if(team){alert("You are already in a team.");return}
 const name=$("teamName").value.trim();if(!name){alert("Enter team name.");return}
 const tc=teamCode(),ref=doc(db,"chessTournamentTeams",tc);
 const member={uid:user.uid,name:profile?.name||user.email};
 await setDoc(ref,{code:tc,name,ownerId:user.uid,memberIds:[user.uid],members:[member],registered:false,createdAt:Date.now()});
 await loadMyTeam();alert("Team created. Share the team code.");
}
async function joinTeam(){
 if(!user){alert("Login first.");return} if(team){alert("You are already in a team.");return}
 const tc=$("teamCode").value.trim().toUpperCase(),ref=doc(db,"chessTournamentTeams",tc),s=await getDoc(ref);
 if(!s.exists()){alert("Team not found.");return}
 await runTransaction(db,async tx=>{
  const x=await tx.get(ref);if(!x.exists())throw Error("Team not found");const d=x.data(),ids=[...(d.memberIds||[])];
  if(ids.includes(user.uid))return;if(ids.length>=6)throw Error("Team already has 6 players.");
  ids.push(user.uid);const members=[...(d.members||[]),{uid:user.uid,name:profile?.name||user.email}];tx.update(ref,{memberIds:ids,members});
 });await loadMyTeam();
}
async function registerTeam(){
 if(!team){alert("Create/join a team first.");return}
 if(team.ownerId!==user.uid){alert("Only the team owner can register the team.");return}
 if(team.memberIds.length<5||team.memberIds.length>6){alert("Team must have 5 or 6 players.");return}
 await updateDoc(doc(db,"chessTournamentTeams",team.id),{registered:true});await loadMyTeam();alert("Team registered.");
}
function pairings(teams){
 const a=[];for(let i=0;i<teams.length;i++)for(let j=i+1;j<teams.length;j++)a.push({id:`L${i+1}-${j+1}`,teamA:teams[i].id,teamB:teams[j].id,status:"pending",winner:null,result:null,roomId:null});
 return a;
}
function standings(t,teams){
 const map={};for(const x of teams)map[x.id]={id:x.id,name:x.name,played:0,wins:0,draws:0,losses:0,points:0};
 for(const m of(t.league?.matches||[])){if(m.status!=="finished"||!m.result)continue;const a=map[m.teamA],b=map[m.teamB];if(!a||!b)continue;a.played++;b.played++;
  if(m.result==="draw"){a.draws++;b.draws++;a.points++;b.points++}
  else{const w=m.result==="A"?a:b,l=m.result==="A"?b:a;w.wins++;l.losses++;w.points+=3}
 }
 return Object.values(map).sort((a,b)=>b.points-a.points||b.wins-a.wins||a.name.localeCompare(b.name));
}
function teamName(t,id){return t.teams?.find(x=>x.id===id)?.name||id}
function renderTournament(){
 const p=$("tournamentPanel");if(!tournament){p.innerHTML="<b>No tournament</b><div>Register teams when ready.</div>";return}
 const teams=tournament.teams||[],st=standings(tournament,teams);
 let h=`<b>${esc((tournament.name||"Chess Arena Tournament"))}</b><div>Status: <span class="badge">${esc(tournament.status)}</span> • Teams: ${teams.length}</div>`;
 h+=`<div class="title">Standings</div><div class="tableWrap"><table class="standings"><tr><th>#</th><th>Team</th><th>P</th><th>W</th><th>D</th><th>L</th><th>Pts</th></tr>`;
 st.forEach((x,i)=>h+=`<tr><td>${i+1}</td><td>${esc(x.name)}</td><td>${x.played}</td><td>${x.wins}</td><td>${x.draws}</td><td>${x.losses}</td><td><b>${x.points}</b></td></tr>`);
 h+=`</table></div>`;
 if(tournament.status==="league"){h+=`<div class="title">League Fixtures</div>`;(tournament.league?.matches||[]).forEach(m=>{h+=fixtureHtml(m,teamName(tournament,m.teamA),teamName(tournament,m.teamB),true)})}
 if(tournament.status==="playoffs"||tournament.status==="finished"){
  h+=`<div class="title">Semi-Finals</div>`;(tournament.semiFinals||[]).forEach(m=>h+=fixtureHtml(m,teamName(tournament,m.teamA),teamName(tournament,m.teamB),true));
  if(tournament.final)h+=`<div class="title">Final</div>`+fixtureHtml(tournament.final,teamName(tournament,tournament.final.teamA),teamName(tournament,tournament.final.teamB),true);
 }
 if(tournament.champion)h+=`<div class="champ">🏆 Champion: ${esc(teamName(tournament,tournament.champion))}</div>`;
 p.innerHTML=h;
}
function fixtureHtml(m,a,b,joinable){
 let result=m.result==="draw"?"Draw":m.result==="A"?a+" won":m.result==="B"?b+" won":m.status;
 let btn=joinable&&m.status==="pending"?`<button onclick="window.playFixture('${m.id}')">PLAY</button>`:"";
 return `<div class="fixture"><b>${esc(a)}</b> vs <b>${esc(b)}</b> — ${esc(result)} ${btn}<div class="muted">${m.roomId?"Room: "+esc(m.roomId):""}</div></div>`;
}
async function loadTournament(){
 const ref=doc(db,"chessTournaments","current");if(tournamentUnsub)tournamentUnsub();
 tournamentUnsub=onSnapshot(ref,s=>{tournament=s.exists()?s.data():null;renderTournament()});
}
async function startTournament(){
 if(!user){alert("Login first.");return}
 const s=await getDocs(query(collection(db,"chessTournamentTeams"),where("registered","==",true)));
 const teams=s.docs.map(d=>({id:d.id,...d.data()})).filter(x=>x.memberIds?.length>=5&&x.memberIds?.length<=6);
 if(teams.length<4){alert("At least 4 registered teams with 5–6 players are required.");return}
 const ref=doc(db,"chessTournaments","current"),old=await getDoc(ref);
 if(old.exists()&&old.data().status!=="finished"){alert("A tournament is already running.");return}
 const data={id:"current",name:"Chess Arena Tournament",status:"league",teams,league:{matches:pairings(teams)},semiFinals:[],final:null,champion:null,createdAt:Date.now()};
 await setDoc(ref,data);alert("Tournament started.");
}
async function advanceTournament(){
 const ref=doc(db,"chessTournaments","current"),s=await getDoc(ref);if(!s.exists())return;const t=s.data();
 if(t.status!=="league")return;
 const st=standings(t,t.teams);if((t.league?.matches||[]).some(m=>m.status!=="finished"))return;
 const top=st.slice(0,4);if(top.length<4)return;
 await updateDoc(ref,{status:"playoffs",semiFinals:[
  {id:"S1",teamA:top[0].id,teamB:top[3].id,status:"pending",winner:null,result:null,roomId:null},
  {id:"S2",teamA:top[1].id,teamB:top[2].id,status:"pending",winner:null,result:null,roomId:null}
 ]});
}
async function findFixture(t,fixtureId){
 if(t.league?.matches?.some(m=>m.id===fixtureId)) return {stage:"league",fixture:t.league.matches.find(m=>m.id===fixtureId)};
 if(t.semiFinals?.some(m=>m.id===fixtureId)) return {stage:"semiFinals",fixture:t.semiFinals.find(m=>m.id===fixtureId)};
 if(t.final?.id===fixtureId) return {stage:"final",fixture:t.final};
 return null;
}
async function playFixture(fixtureId){
 if(!user){alert("Login first.");return}
 const t=tournament;if(!t)return;
 const found=await findFixture(t,fixtureId);if(!found){alert("Fixture not found.");return}
 const fixture=found.fixture;
 if(fixture.status!=="pending"){alert("Fixture already completed.");return}
 const A=t.teams.find(x=>x.id===fixture.teamA),B=t.teams.find(x=>x.id===fixture.teamB);
 if(!A||!B||(!A.memberIds.includes(user.uid)&&!B.memberIds.includes(user.uid))){alert("You must be a player of one of the two teams.");return}

 const roomId=code(),ref=doc(db,"chessRooms",roomId);
 const myTeam=A.memberIds.includes(user.uid)?A:B;
 const rd=roomBase({
   whiteId:user.uid,whiteName:profile?.name||user.email,status:"waiting",
   tournamentId:"current",fixtureId,stage:found.stage,
   teamA:A.id,teamB:B.id,createdByTeam:myTeam.id
 });
 await setDoc(ref,rd);
 await updateFixtureRoom(fixtureId,roomId,found.stage);
 $("roomCode").value=roomId;roomRef=ref;myColor="w";gameMode="tournament";listenRoom(roomId);
}
async function updateFixtureRoom(id,roomId,stage){
 const ref=doc(db,"chessTournaments","current");
 await runTransaction(db,async tx=>{
  const s=await tx.get(ref);if(!s.exists())return;const d=s.data();
  if(stage==="league"){
    const matches=(d.league?.matches||[]).map(m=>m.id===id?{...m,roomId}:m);
    tx.update(ref,{league:{...d.league,matches}});
  }else if(stage==="semiFinals"){
    const matches=(d.semiFinals||[]).map(m=>m.id===id?{...m,roomId}:m);
    tx.update(ref,{semiFinals:matches});
  }else if(stage==="final"&&d.final?.id===id){
    tx.update(ref,{final:{...d.final,roomId}});
  }
 });
}
async function checkTournamentResult(completedRoom){
 if(gameMode!=="tournament"||!completedRoom?.winner||!completedRoom.fixtureId)return;
 const ref=doc(db,"chessTournaments","current");
 let changed=false;
 await runTransaction(db,async tx=>{
  const s=await tx.get(ref);if(!s.exists())return;const t=s.data();
  const result=completedRoom.winner==="draw"?"draw":completedRoom.winner==="w"?"A":"B";
  const finish=(m)=>m.id===completedRoom.fixtureId&&m.status==="pending"?{...m,status:"finished",result,winner:result==="A"?m.teamA:result==="B"?m.teamB:null}:m;
  if(t.league?.matches?.some(m=>m.id===completedRoom.fixtureId)){
    const matches=t.league.matches.map(finish);changed=matches.some((m,i)=>m!==t.league.matches[i]);
    if(changed)tx.update(ref,{league:{...t.league,matches},updatedAt:Date.now()});
  }else if(t.semiFinals?.some(m=>m.id===completedRoom.fixtureId)){
    const matches=t.semiFinals.map(finish);changed=matches.some((m,i)=>m!==t.semiFinals[i]);
    if(changed)tx.update(ref,{semiFinals:matches,updatedAt:Date.now()});
  }else if(t.final?.id===completedRoom.fixtureId&&t.final.status==="pending"){
    const f=finish(t.final);changed=true;
    tx.update(ref,{final:f,champion:f.winner, status:"finished",updatedAt:Date.now()});
  }
 });
 if(!changed)return;

 const fresh=await getDoc(ref);if(!fresh.exists())return;const t=fresh.data();

 // League -> top 4 -> semi-finals.
 if(t.status==="league" && (t.league?.matches||[]).length && t.league.matches.every(m=>m.status==="finished")){
   const st=standings(t,t.teams),top=st.slice(0,4);
   if(top.length===4){
    const sf=[
      {id:"S1",teamA:top[0].id,teamB:top[3].id,status:"pending",winner:null,result:null,roomId:null},
      {id:"S2",teamA:top[1].id,teamB:top[2].id,status:"pending",winner:null,result:null,roomId:null}
    ];
    await updateDoc(ref,{status:"playoffs",semiFinals:sf,final:null,updatedAt:Date.now()});
   }
   return;
 }
 // Semi-finals -> final.
 if(t.status==="playoffs" && (t.semiFinals||[]).length===2 && t.semiFinals.every(m=>m.status==="finished")){
   const winners=t.semiFinals.map(m=>m.winner).filter(Boolean);
   if(winners.length===2){
    await updateDoc(ref,{final:{id:"F1",teamA:winners[0],teamB:winners[1],status:"pending",winner:null,result:null,roomId:null},updatedAt:Date.now()});
   }
 }
}
window.playFixture=playFixture;

$("create").onclick=createRoom;$("join").onclick=joinRoom;$("computer").onclick=startComputerGame;$("hint").onclick=hint;
$("flip").onclick=()=>{flipped=!flipped;render()};$("resign").onclick=resign;
$("copy").onclick=async()=>{if($("roomCode").value)await navigator.clipboard.writeText($("roomCode").value)};
$("signup").onclick=signup;$("signin").onclick=signin;$("signout").onclick=logout;
$("createTeam").onclick=createTeam;$("joinTeam").onclick=joinTeam;$("registerTeam").onclick=registerTeam;$("startTournament").onclick=startTournament;
$("refreshTournament").onclick=async()=>{await loadMyTeam();await loadTournament()};

onAuthStateChanged(auth,async u=>{
 user=u;
 if(u){$("net").textContent="● Online";$("accountStatus").innerHTML="<b>Loading account...</b><div></div>";await loadProfile()}
 else{$("accountStatus").innerHTML="<b>Not logged in</b><div>Login to play tournament games.</div>"}
});
$("roomCode").addEventListener("input",e=>e.target.value=e.target.value.toUpperCase());
startTimer();render();
</script>
</body>
</html>
