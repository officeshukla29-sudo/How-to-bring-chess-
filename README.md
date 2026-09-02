<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Online Chess — Firebase Multiplayer</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#0d1321;color:#eef2f7;font-family:Inter,system-ui,Arial,sans-serif}
.app{max-width:1180px;width:96%;margin:20px auto}.top{display:flex;justify-content:space-between;gap:15px;align-items:center;margin-bottom:16px}
h1{margin:0;font-size:32px}.muted{color:#94a3b8;font-size:13px}.grid{display:grid;grid-template-columns:minmax(320px,760px) 320px;gap:18px}
.card{background:#182235;border:1px solid #2c3a50;border-radius:18px;box-shadow:0 14px 45px #0005}.boardCard{padding:12px}.board{display:grid;grid-template-columns:repeat(8,1fr);aspect-ratio:1;border-radius:10px;overflow:hidden;user-select:none}
.sq{position:relative;display:flex;justify-content:center;align-items:center;cursor:pointer}.light{background:#f0d9b5}.dark{background:#b58863}
.sq.sel{box-shadow:inset 0 0 0 5px #facc15}.sq.last{background-image:linear-gradient(#facc1555,#facc1555)}.sq.check{background-image:linear-gradient(#ef4444aa,#ef4444aa)}
.sq.move:after{content:"";position:absolute;width:22%;height:22%;border-radius:50%;background:#22c55e88}.sq.capture:after{content:"";position:absolute;inset:7%;border:5px solid #22c55e88;border-radius:50%}
.piece{font-family:"Segoe UI Symbol","Noto Sans Symbols 2",serif;font-size:clamp(35px,7.4vw,68px);line-height:1;z-index:2}.wp{color:#fff;text-shadow:0 2px 2px #222}.bp{color:#111;text-shadow:0 1px 1px #fff}
.coord{position:absolute;font-size:10px;font-weight:700;opacity:.65}.rank{top:3px;left:4px}.file{bottom:3px;right:4px}.light .coord{color:#765536}.dark .coord{color:#f7dfb8}
.panel{padding:17px}.status{background:#0e1726;border:1px solid #2e3c52;border-radius:12px;padding:12px;margin-bottom:12px}.status b{font-size:18px}.status div{color:#94a3b8;font-size:12px;margin-top:3px}
input,button,select{font:inherit;border:0;border-radius:10px;padding:11px 12px}input,select{background:#0e1726;color:white;border:1px solid #35445b;width:100%}button{background:#334155;color:white;cursor:pointer;font-weight:650}button:hover{background:#475569}.primary{background:#22c55e;color:#05220e}.primary:hover{background:#16a34a}
.room{display:grid;grid-template-columns:1fr auto;gap:8px;margin-bottom:8px}.room input{text-transform:uppercase;font-weight:800;letter-spacing:2px}.buttons{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:12px}
.players{display:grid;gap:7px;margin:12px 0}.player{padding:9px 11px;background:#0e1726;border-radius:10px;display:flex;justify-content:space-between}.you{border:1px solid #22c55e55}
.title{font-size:12px;text-transform:uppercase;color:#94a3b8;letter-spacing:.08em;margin:14px 0 7px}.moves{height:240px;overflow:auto;background:#0e1726;border-radius:10px;padding:7px}.row{display:grid;grid-template-columns:34px 1fr 1fr;padding:5px 7px;border-bottom:1px solid #223047;font-size:13px}
.note{font-size:11px;color:#94a3b8;line-height:1.5;margin-top:10px}.online{color:#22c55e}.offline{color:#ef4444}.hidden{display:none!important}
@media(max-width:900px){.grid{grid-template-columns:1fr}.panel{display:grid;grid-template-columns:1fr 1fr;gap:10px}.status,.players,.room,.buttons,.title,.moves,.note{grid-column:span 2}.moves{height:190px}}
@media(max-width:500px){.app{width:98%;margin:8px auto}.boardCard{padding:6px}.panel{padding:11px}.piece{font-size:clamp(29px,11vw,46px)}}
</style>
</head>
<body>
<div class="app">
<div class="top"><div><h1>♟ Online Chess</h1><div class="muted">Real-time multiplayer • Firebase Firestore</div></div><div id="net" class="online">● Online</div></div>
<div class="grid">
<div class="card boardCard"><div id="board" class="board"></div></div>
<div class="card panel">
<div id="status" class="status"><b>Not connected</b><div>Create or join a room.</div></div>
<div class="room"><input id="roomCode" maxlength="6" placeholder="ROOM CODE"><button id="copy">Copy</button></div>
<div class="buttons"><button id="create" class="primary">Create Room</button><button id="join">Join Room</button></div>
<div class="players">
<div class="player" id="white"><span>♔ White</span><b>Waiting</b></div>
<div class="player" id="black"><span>♚ Black</span><b>Waiting</b></div>
</div>
<div class="buttons"><button id="resign">Resign</button><button id="flip">Flip Board</button></div>
<div class="title">Move History</div><div id="moves" class="moves"></div>
<div class="note">Create a room, copy the 6-character code and send it to your opponent. Both players must open this page and use the same Firebase project.</div>
</div>
</div>
</div>

<script type="module">
import {initializeApp} from "https://www.gstatic.com/firebasejs/12.18.0/firebase-app.js";
import {getFirestore,doc,getDoc,setDoc,updateDoc,onSnapshot,runTransaction} from "https://www.gstatic.com/firebasejs/12.18.0/firebase-firestore.js";

/* ==========================================================
   1) PASTE YOUR FIREBASE CONFIG HERE
   Firebase Console > Project settings > Your apps > Web app
   ========================================================== */
const firebaseConfig={
  apiKey:"AIzaSyCtRpxAiAdl0RIpLbx3JuuD5aO_gGYJnNY",
  authDomain:"shuklareports.firebaseapp.com",
  projectId:"shuklareports",
  storageBucket:"shuklareports.firebasestorage.app",
  messagingSenderId:"133308067679",
  appId:"1:133308067679:web:a921e81c720178a0bdd0ea",
  measurementId:"G-0VF9CRV1R4"
};

let db=null,roomRef=null,unsubscribe=null,room=null,myColor=null,playerId=localStorage.getItem("chessPlayerId");
if(!playerId){playerId=crypto.randomUUID();localStorage.setItem("chessPlayerId",playerId)}

const PIECES={K:"♔",Q:"♕",R:"♖",B:"♗",N:"♘",P:"♙",k:"♚",q:"♛",r:"♜",b:"♝",n:"♞",p:"♟"};
const V={P:100,N:320,B:330,R:500,Q:900,K:20000},files="abcdefgh";
let board,selected=null,legal=[],flipped=false,busy=false;

function initial(){return [["r","n","b","q","k","b","n","r"],["p","p","p","p","p","p","p","p"],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],["P","P","P","P","P","P","P","P"],["R","N","B","Q","K","B","N","R"]]}
function setupLocal(){board=initial()}
function color(p){return p&&p===p.toUpperCase()?"w":"b"} function opp(c){return c==="w"?"b":"w"} function ib(r,c){return r>=0&&r<8&&c>=0&&c<8}
function king(b,c){let k=c==="w"?"K":"k";for(let r=0;r<8;r++)for(let x=0;x<8;x++)if(b[r][x]===k)return[r,x];return null}
function attacked(b,r,c,by){
 let p=by==="w"?"P":"p",pr=r+(by==="w"?1:-1);for(let dc of[-1,1])if(ib(pr,c+dc)&&b[pr][c+dc]===p)return true;
 p=by==="w"?"N":"n";for(let [dr,dc] of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])if(ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 p=by==="w"?"K":"k";for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if((dr||dc)&&ib(r+dr,c+dc)&&b[r+dr][c+dc]===p)return true;
 for(let [dr,dc,type] of[[1,0,"R"],[-1,0,"R"],[0,1,"R"],[0,-1,"R"],[1,1,"B"],[1,-1,"B"],[-1,1,"B"],[-1,-1,"B"]]){
  let rr=r+dr,cc=c+dc;while(ib(rr,cc)){let q=b[rr][cc];if(q){if(color(q)===by&&(q.toUpperCase()===type||q.toUpperCase()==="Q"))return true;break}rr+=dr;cc+=dc}
 }return false
}
function check(b,c){let k=king(b,c);return !k||attacked(b,k[0],k[1],opp(c))}
function pseudo(b,c,cast,ep){
 const out=[],add=(fr,fc,tr,tc,e={})=>{if(!ib(tr,tc))return;let t=b[tr][tc];if(t&&color(t)===c)return;if(t&&t.toUpperCase()==="K")return;out.push({fr,fc,tr,tc,...e})};
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){let p=b[r][x];if(!p||color(p)!==c)continue;let u=p.toUpperCase();
  if(u==="P"){let d=c==="w"?-1:1,start=c==="w"?6:1,pr=c==="w"?0:7;
   if(ib(r+d,x)&&!b[r+d][x]){if(r+d===pr)for(let q of["Q","R","B","N"])out.push({fr:r,fc:x,tr:r+d,tc:x,promotion:c==="w"?q:q.toLowerCase()});else add(r,x,r+d,x);if(r===start&&!b[r+2*d][x])add(r,x,r+2*d,x)}
   for(let dc of[-1,1]){let tr=r+d,tc=x+dc;if(!ib(tr,tc))continue;if(b[tr][tc]&&color(b[tr][tc])!==c&&b[tr][tc].toUpperCase()!=="K"){if(tr===pr)for(let q of["Q","R","B","N"])out.push({fr:r,fc:x,tr,tc,promotion:c==="w"?q:q.toLowerCase()});else add(r,x,tr,tc)}if(ep&&ep[0]===tr&&ep[1]===tc)out.push({fr:r,fc:x,tr,tc,ep:true})}
  }else if(u==="N"){for(let[dr,dc]of[[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]])add(r,x,r+dr,x+dc)}
  else if(u==="K"){for(let dr=-1;dr<=1;dr++)for(let dc=-1;dc<=1;dc++)if(dr||dc)add(r,x,r+dr,x+dc);
   let home=c==="w"?7:0;if(r===home&&x===4&&!check(b,c)){let k=c==="w"?cast.wK:cast.bK,q=c==="w"?cast.wQ:cast.bQ;
    if(k&&!b[home][5]&&!b[home][6]&&!attacked(b,home,5,opp(c))&&!attacked(b,home,6,opp(c)))out.push({fr:r,fc:x,tr:home,tc:6,castle:"K"});
    if(q&&!b[home][1]&&!b[home][2]&&!b[home][3]&&!attacked(b,home,3,opp(c))&&!attacked(b,home,2,opp(c)))out.push({fr:r,fc:x,tr:home,tc:2,castle:"Q"});
   }
  }else{let ds=u==="R"?[[1,0],[-1,0],[0,1],[0,-1]]:u==="B"?[[1,1],[1,-1],[-1,1],[-1,-1]]:[[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]];
   for(let[dr,dc]of ds){let rr=r+dr,cc=x+dc;while(ib(rr,cc)){let t=b[rr][cc];if(!t)add(r,x,rr,cc);else{if(color(t)!==c&&t.toUpperCase()!=="K")add(r,x,rr,cc);break}rr+=dr;cc+=dc}}
  }
 }return out
}
function apply(b,m,cast,ep){
 let p=b[m.fr][m.fc],t=b[m.tr][m.tc];b[m.tr][m.tc]=m.promotion||p;b[m.fr][m.fc]=null;
 if(m.ep)b[m.tr+(color(p)==="w"?1:-1)][m.tc]=null;
 if(m.castle){let r=m.fr;if(m.tc===6){b[r][5]=b[r][7];b[r][7]=null}else{b[r][3]=b[r][0];b[r][0]=null}}
 if(p==="K")cast.wK=cast.wQ=false;if(p==="k")cast.bK=cast.bQ=false;
 if(p==="R"){if(m.fr===7&&m.fc===0)cast.wQ=false;if(m.fr===7&&m.fc===7)cast.wK=false}
 if(p==="r"){if(m.fr===0&&m.fc===0)cast.bQ=false;if(m.fr===0&&m.fc===7)cast.bK=false}
 if(t==="R"){if(m.tr===7&&m.tc===0)cast.wQ=false;if(m.tr===7&&m.tc===7)cast.wK=false}
 if(t==="r"){if(m.tr===0&&m.tc===0)cast.bQ=false;if(m.tr===0&&m.tc===7)cast.bK=false}
 let nep=null;if(p.toUpperCase()==="P"&&Math.abs(m.tr-m.fr)===2)nep=[(m.tr+m.fr)/2,m.fc];
 return nep
}
function legalMoves(b,c,cast,ep){let out=[];for(let m of pseudo(b,c,cast,ep)){let x=b.map(a=>a.slice()),cc={...cast};let e=apply(x,m,cc,ep);if(!check(x,c))out.push(m)}return out}
function notation(b,m,cast,ep){
 let p=b[m.fr][m.fc],t=b[m.tr][m.tc];if(m.castle)return m.tc===6?"O-O":"O-O-O";let s=p.toUpperCase()==="P"?"":p.toUpperCase();
 if(t||m.ep)s+=(p.toUpperCase()==="P"?files[m.fc]:"")+"x";s+=files[m.tc]+(8-m.tr);if(m.promotion)s+="="+m.promotion.toUpperCase();return s
}
function code(){return Math.random().toString(36).slice(2,8).toUpperCase()}
function configReady(){return firebaseConfig.apiKey!=="YOUR_API_KEY"&&firebaseConfig.projectId!=="YOUR_PROJECT_ID"}
function say(a,b){document.getElementById("status").innerHTML="<b>"+a+"</b><div>"+b+"</div>"}
function updatePlayers(){
 const w=room?.whiteName||"Waiting",b=room?.blackName||"Waiting";
 document.querySelector("#white b").textContent=w+(myColor==="w"?" (You)":"");
 document.querySelector("#black b").textContent=b+(myColor==="b"?" (You)":"");
}
async function createRoom(){
 if(!configReady()){alert("First paste your Firebase web configuration into the file.");return}
 const id=code();roomRef=doc(db,"chessRooms",id);
 const init={board:initial(),turn:"w",castling:{wK:true,wQ:true,bK:true,bQ:true},ep:null,history:[],lastMove:null,whiteId:playerId,whiteName:"Player 1",blackId:null,blackName:null,status:"waiting",winner:null,updatedAt:Date.now()};
 await setDoc(roomRef,init);document.getElementById("roomCode").value=id;myColor="w";listen(id)
}
async function joinRoom(){
 if(!configReady()){alert("First paste your Firebase web configuration into the file.");return}
 const id=document.getElementById("roomCode").value.trim().toUpperCase();if(id.length!==6){alert("Enter a 6-character room code.");return}
 roomRef=doc(db,"chessRooms",id);let s=await getDoc(roomRef);if(!s.exists()){alert("Room not found.");return}
 let d=s.data();if(d.blackId&&d.blackId!==playerId){alert("Room already has two players.");return}
 if(d.whiteId===playerId)myColor="w";else{await updateDoc(roomRef,{blackId:playerId,blackName:"Player 2",status:"playing",updatedAt:Date.now()});myColor="b"}
 listen(id)
}
function listen(id){
 if(unsubscribe)unsubscribe();document.getElementById("roomCode").value=id;
 unsubscribe=onSnapshot(roomRef,s=>{if(!s.exists()){say("Room closed","");return}room=s.data();board=room.board;render();updatePlayers();updateStatus()},e=>{say("Firebase error",e.message)})
}
function updateStatus(){
 if(!room)return;
 if(room.winner){say(room.winner==="draw"?"Draw":room.winner==="w"?"White wins":"Black wins","Game over");return}
 if(room.status==="waiting"){say("Waiting for opponent","Send room code: "+document.getElementById("roomCode").value);return}
 say(room.turn===myColor?"Your turn":"Opponent's turn",room.turn==="w"?"White to move":"Black to move");
}
async function sendMove(m){
 if(busy||!room||room.status!=="playing"||room.turn!==myColor)return;
 busy=true;
 try{
  await runTransaction(db,async tx=>{
   const snap=await tx.get(roomRef);if(!snap.exists())throw Error("Room does not exist");
   const d=snap.data();if(d.turn!==myColor)throw Error("Not your turn");
   const b=d.board.map(a=>a.slice()),cast={...d.castling},ep=d.ep;
   const lm=legalMoves(b,myColor,cast,ep).find(x=>JSON.stringify(x)===JSON.stringify(m));if(!lm)throw Error("Illegal move");
   const note=notation(b,lm,cast,ep),nep=apply(b,lm,cast,ep);
   let hist=[...(d.history||[]),note],next=opp(myColor),ms=legalMoves(b,next,cast,nep),winner=null,status="playing";
   if(!ms.length){status="finished";winner=check(b,next)?myColor:"draw"}
   tx.update(roomRef,{board:b,castling:cast,ep:nep,turn:next,history:hist,lastMove:lm,status,winner,updatedAt:Date.now()});
  });
 }catch(e){console.error(e);alert(e.message)}finally{busy=false}
}
function render(){
 const el=document.getElementById("board");el.innerHTML="";if(!board)return;
 const rs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7],cs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7];
 legal=selected&&room?legalMoves(board,myColor,room.castling,room.ep).filter(m=>m.fr===selected[0]&&m.fc===selected[1]):[];
 for(let ri=0;ri<8;ri++)for(let ci=0;ci<8;ci++){let r=rs[ri],c=cs[ci],s=document.createElement("div");s.className="sq "+((r+c)%2?"dark":"light");
  if(room?.lastMove&&((room.lastMove.fr===r&&room.lastMove.fc===c)||(room.lastMove.tr===r&&room.lastMove.tc===c)))s.classList.add("last");
  if(selected?.[0]===r&&selected?.[1]===c)s.classList.add("sel");let lm=legal.find(m=>m.tr===r&&m.tc===c);if(lm)s.classList.add(board[r][c]?"capture":"move");
  let k=room&&king(board,room.turn);if(k?.[0]===r&&k?.[1]===c&&check(board,room.turn))s.classList.add("check");
  if(board[r][c]){let z=document.createElement("span");z.className="piece "+(color(board[r][c])==="w"?"wp":"bp");z.textContent=PIECES[board[r][c]];s.appendChild(z)}
  if(ci===0){let z=document.createElement("span");z.className="coord rank";z.textContent=8-r;s.appendChild(z)}if(ri===7){let z=document.createElement("span");z.className="coord file";z.textContent=files[c];s.appendChild(z)}
  s.onclick=()=>click(r,c);el.appendChild(s)
 }
 document.getElementById("moves").innerHTML=(room?.history||[]).map((x,i)=>i%2===0?`<div class="row"><span>${i/2+1}.</span><span>${x}</span><span>${room.history[i+1]||""}</span></div>`:"").join("");
}
function click(r,c){
 if(!room||room.status!=="playing"||room.turn!==myColor)return;
 let p=board[r][c],m=legal.find(x=>x.tr===r&&x.tc===c);if(selected&&m){sendMove(m);selected=null;return}
 if(p&&color(p)===myColor){selected=[r,c];render()}else{selected=null;render()}
}
document.getElementById("create").onclick=createRoom;document.getElementById("join").onclick=joinRoom;
document.getElementById("flip").onclick=()=>{flipped=!flipped;render()};
document.getElementById("copy").onclick=async()=>{let v=document.getElementById("roomCode").value;if(v)await navigator.clipboard.writeText(v)};
document.getElementById("resign").onclick=async()=>{if(room?.status==="playing"&&myColor){if(confirm("Resign this game?"))await updateDoc(roomRef,{status:"finished",winner:opp(myColor),updatedAt:Date.now()})}};
document.getElementById("roomCode").addEventListener("input",e=>e.target.value=e.target.value.toUpperCase());

try{
 if(configReady()){const app=initializeApp(firebaseConfig);db=getFirestore(app);say("Ready","Create a room or join an existing room.")}
 else say("Firebase setup required","Open this file and replace the YOUR_* configuration values.");
}catch(e){say("Firebase setup error",e.message)}
</script>
</body>
</html>
