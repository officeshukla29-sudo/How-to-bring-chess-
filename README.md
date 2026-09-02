<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1.0">
<title>Chess — Play vs Friend or Computer</title>
<style>
*{box-sizing:border-box}body{margin:0;background:#0d1321;color:#eef2f7;font-family:Inter,system-ui,Arial,sans-serif}
.app{max-width:1180px;width:96%;margin:20px auto}.top{display:flex;justify-content:space-between;gap:15px;align-items:center;margin-bottom:16px;flex-wrap:wrap}
h1{margin:0;font-size:30px}.muted{color:#94a3b8;font-size:13px}.grid{display:grid;grid-template-columns:minmax(320px,760px) 320px;gap:18px}
.card{background:#182235;border:1px solid #2c3a50;border-radius:18px;box-shadow:0 14px 45px #0005}.boardCard{padding:12px}.board{display:grid;grid-template-columns:repeat(8,1fr);aspect-ratio:1;border-radius:10px;overflow:hidden;user-select:none}
.sq{position:relative;display:flex;justify-content:center;align-items:center;cursor:pointer}.light{background:#f0d9b5}.dark{background:#b58863}
.sq.sel{box-shadow:inset 0 0 0 5px #facc15}.sq.last{background-image:linear-gradient(#facc1555,#facc1555)}.sq.check{background-image:linear-gradient(#ef4444aa,#ef4444aa)}
.sq.move:after{content:"";position:absolute;width:22%;height:22%;border-radius:50%;background:#22c55e88}.sq.capture:after{content:"";position:absolute;inset:7%;border:5px solid #22c55e88;border-radius:50%}
.piece{font-family:"Segoe UI Symbol","Noto Sans Symbols 2",serif;font-size:clamp(35px,7.4vw,68px);line-height:1;z-index:2}.wp{color:#fff;text-shadow:0 2px 2px #222}.bp{color:#111;text-shadow:0 1px 1px #fff}
.coord{position:absolute;font-size:10px;font-weight:700;opacity:.65}.rank{top:3px;left:4px}.file{bottom:3px;right:4px}.light .coord{color:#765536}.dark .coord{color:#f7dfb8}
.panel{padding:17px}.status{background:#0e1726;border:1px solid #2e3c52;border-radius:12px;padding:12px;margin-bottom:12px}.status b{font-size:18px}.status div{color:#94a3b8;font-size:12px;margin-top:3px}
select,button{font:inherit;border:0;border-radius:10px;padding:11px 12px}select{background:#0e1726;color:white;border:1px solid #35445b;width:100%}
button{background:#334155;color:white;cursor:pointer;font-weight:650}button:hover{background:#475569}.primary{background:#22c55e;color:#05220e}.primary:hover{background:#16a34a}
.modeRow{display:grid;grid-template-columns:1fr;gap:8px;margin-bottom:10px}.buttons{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-bottom:12px}
.title{font-size:12px;text-transform:uppercase;color:#94a3b8;letter-spacing:.08em;margin:14px 0 7px}.moves{height:260px;overflow:auto;background:#0e1726;border-radius:10px;padding:7px}.row{display:grid;grid-template-columns:34px 1fr 1fr;padding:5px 7px;border-bottom:1px solid #223047;font-size:13px}
.cap{display:flex;gap:2px;flex-wrap:wrap;min-height:22px;font-size:16px}
.hidden{display:none!important}
@media(max-width:900px){.grid{grid-template-columns:1fr}.panel{display:grid;grid-template-columns:1fr 1fr;gap:10px}.status,.modeRow,.buttons,.title,.moves{grid-column:span 2}.moves{height:190px}}
@media(max-width:500px){.app{width:98%;margin:8px auto}.boardCard{padding:6px}.panel{padding:11px}.piece{font-size:clamp(29px,11vw,46px)}}
</style>
</head>
<body>
<div class="app">
<div class="top"><div><h1>♟ Chess</h1><div class="muted">Pass &amp; play, or play vs computer</div></div></div>
<div class="grid">
<div class="card boardCard"><div id="board" class="board"></div></div>
<div class="card panel">
<div id="status" class="status"><b>White to move</b><div>New game</div></div>
<div class="modeRow">
<select id="mode">
<option value="pvp">Pass &amp; Play (2 players)</option>
<option value="cpu-b">Vs Computer — I play White</option>
<option value="cpu-w">Vs Computer — I play Black</option>
</select>
<select id="diff">
<option value="1">Computer: Easy</option>
<option value="2" selected>Computer: Medium</option>
<option value="3">Computer: Hard</option>
</select>
</div>
<div class="buttons"><button id="newgame" class="primary">New Game</button><button id="flip">Flip Board</button></div>
<div class="buttons"><button id="undo">Undo Move</button><button id="resign">Resign</button></div>
<div class="title">Captured</div>
<div class="cap" id="capWhite"></div>
<div class="cap" id="capBlack"></div>
<div class="title">Move History</div><div id="moves" class="moves"></div>
</div>
</div>
</div>

<script>
const PIECES={K:"♔",Q:"♕",R:"♖",B:"♗",N:"♘",P:"♙",k:"♚",q:"♛",r:"♜",b:"♝",n:"♞",p:"♟"};
const VAL={P:100,N:320,B:330,R:500,Q:900,K:20000},FILES="abcdefgh";
const PST_P=[0,0,0,0,0,0,0,0,50,50,50,50,50,50,50,50,10,10,20,30,30,20,10,10,5,5,10,25,25,10,5,5,0,0,0,20,20,0,0,0,5,-5,-10,0,0,-10,-5,5,5,10,10,-20,-20,10,10,5,0,0,0,0,0,0,0,0];
const PST_N=[-50,-40,-30,-30,-30,-30,-40,-50,-40,-20,0,0,0,0,-20,-40,-30,0,10,15,15,10,0,-30,-30,5,15,20,20,15,5,-30,-30,0,15,20,20,15,0,-30,-30,5,10,15,15,10,5,-30,-40,-20,0,5,5,0,-20,-40,-50,-40,-30,-30,-30,-30,-40,-50];

let board,turn,castling,ep,history,captured,selected,legal,flipped=false,mode="pvp",busy=false,gameOver=false,moveLog=[];

function initial(){return [["r","n","b","q","k","b","n","r"],["p","p","p","p","p","p","p","p"],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],[null,null,null,null,null,null,null,null],["P","P","P","P","P","P","P","P"],["R","N","B","Q","K","B","N","R"]]}
function color(p){return p&&p===p.toUpperCase()?"w":"b"}
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
  while(ib(rr,cc)){let q=b[rr][cc];if(q){if(color(q)===by&&(q.toUpperCase()===type||q.toUpperCase()==="Q"))return true;break}rr+=dr;cc+=dc}
 }
 return false
}
function check(b,c){let k=king(b,c);return !k||attacked(b,k[0],k[1],opp(c))}

function pseudo(b,c,cast,epSq){
 const out=[],add=(fr,fc,tr,tc,e={})=>{if(!ib(tr,tc))return;let t=b[tr][tc];if(t&&color(t)===c)return;out.push({fr,fc,tr,tc,piece:b[fr][fc],captured:t,...e})};
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){
  let p=b[r][x];if(!p||color(p)!==c)continue;let u=p.toUpperCase();
  if(u==="P"){
   let d=c==="w"?-1:1,start=c==="w"?6:1,pr=c==="w"?0:7;
   if(ib(r+d,x)&&!b[r+d][x]){
    if(r+d===pr)for(let q of["Q","R","B","N"])out.push({fr:r,fc:x,tr:r+d,tc:x,piece:p,captured:null,promotion:c==="w"?q:q.toLowerCase()});
    else add(r,x,r+d,x);
    if(r===start&&!b[r+2*d][x])add(r,x,r+2*d,x,{double:true})
   }
   for(let dc of[-1,1]){
    let tr=r+d,tc=x+dc;if(!ib(tr,tc))continue;
    if(b[tr][tc]&&color(b[tr][tc])!==c){
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
   if(r===home&&x===4&&!check(b,c)){
    let k=c==="w"?cast.wK:cast.bK,q=c==="w"?cast.wQ:cast.bQ;
    if(k&&!b[home][5]&&!b[home][6]&&b[home][7]===(c==="w"?"R":"r")&&!attacked(b,home,5,opp(c))&&!attacked(b,home,6,opp(c)))out.push({fr:r,fc:x,tr:home,tc:6,piece:p,captured:null,castle:"K"});
    if(q&&!b[home][1]&&!b[home][2]&&!b[home][3]&&b[home][0]===(c==="w"?"R":"r")&&!attacked(b,home,3,opp(c))&&!attacked(b,home,2,opp(c)))out.push({fr:r,fc:x,tr:home,tc:2,piece:p,captured:null,castle:"Q"})
   }
  }else{
   let ds=u==="R"?[[1,0],[-1,0],[0,1],[0,-1]]:u==="B"?[[1,1],[1,-1],[-1,1],[-1,-1]]:[[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]];
   for(let[dr,dc]of ds){
    let rr=r+dr,cc=x+dc;
    while(ib(rr,cc)){let t=b[rr][cc];if(!t)add(r,x,rr,cc);else{if(color(t)!==c)add(r,x,rr,cc);break}rr+=dr;cc+=dc}
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
 if(m.piece.toUpperCase()==="K"){if(color(m.piece)==="w"){nc.wK=false;nc.wQ=false}else{nc.bK=false;nc.bQ=false}}
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
 return pseudo(b,c,cast,epSq).filter(m=>{
  let{nb}=applyMove(b,m,cast,epSq);
  return !check(nb,c)
 })
}

function notation(b,m,cast,epSq,allLegal){
 let u=m.piece.toUpperCase(),from=FILES[m.fc]+(8-m.fr),to=FILES[m.tc]+(8-m.tr);
 if(m.castle==="K")return "O-O";
 if(m.castle==="Q")return "O-O-O";
 let s="";
 if(u==="P"){
  s=m.captured?FILES[m.fc]+"x"+to:to;
 }else{
  s=u+(m.captured?"x":"")+to;
 }
 if(m.promotion)s+="="+m.promotion.toUpperCase();
 let{nb}=applyMove(b,m,cast,epSq),nc2=opp(color(m.piece));
 if(check(nb,nc2)){
  let hasMoves=legalMoves(nb,nc2,cast,null).length>0;
  s+=hasMoves?"+":"#"
 }
 return s
}

function newGame(){
 board=initial();turn="w";castling={wK:true,wQ:true,bK:true,bQ:true};ep=null;history=[];captured={w:[],b:[]};
 selected=null;legal=[];gameOver=false;moveLog=[];
 mode=document.getElementById("mode").value;
 say("White to move","New game");
 render();
 maybeComputerMove();
}

function humanColor(){
 if(mode==="pvp")return null;
 return mode==="cpu-b"?"w":"b";
}

function makeMove(m){
 if(gameOver)return;
 let note=notation(board,m,castling,ep);
 let{nb,nc,nep}=applyMove(board,m,castling,ep);
 if(m.captured)captured[color(m.captured)].push(m.captured);
 history.push(note);moveLog.push({board:board.map(a=>a.slice()),castling:{...castling},ep,turn,capturedSnap:{w:captured.w.slice(),b:captured.b.slice()}});
 board=nb;castling=nc;ep=nep;turn=opp(turn);
 selected=null;legal=[];
 let moves=legalMoves(board,turn,castling,ep);
 if(!moves.length){
  gameOver=true;
  if(check(board,turn))say(turn==="w"?"Black wins":"White wins","Checkmate");
  else say("Draw","Stalemate");
 }else if(check(board,turn)){
  say((turn==="w"?"White":"Black")+" to move","Check!")
 }else{
  say((turn==="w"?"White":"Black")+" to move","")
 }
 render();
 maybeComputerMove();
}

function maybeComputerMove(){
 if(gameOver)return;
 let hc=humanColor();
 if(hc===null)return;
 if(turn===hc)return;
 busy=true;
 setTimeout(()=>{
  let m=computerMove();
  busy=false;
  if(m)makeMove(m);
 },350);
}

function evalBoard(b){
 let s=0;
 for(let r=0;r<8;r++)for(let x=0;x<8;x++){
  let p=b[r][x];if(!p)continue;
  let u=p.toUpperCase(),c=color(p),sign=c==="w"?1:-1;
  s+=sign*VAL[u];
  let idx=c==="w"?r*8+x:(7-r)*8+x;
  if(u==="P")s+=sign*PST_P[idx];
  if(u==="N")s+=sign*PST_N[idx];
 }
 return s
}

function computerMove(){
 let diff=parseInt(document.getElementById("diff").value);
 let depth=diff===1?1:diff===2?2:3;
 let moves=legalMoves(board,turn,castling,ep);
 if(!moves.length)return null;
 if(diff===1&&Math.random()<0.35){
  return moves[Math.floor(Math.random()*moves.length)];
 }
 let best=null,bestScore=turn==="w"?-Infinity:Infinity;
 let order=moves.slice().sort((a,b2)=>(b2.captured?VAL[b2.captured.toUpperCase()]:0)-(a.captured?VAL[a.captured.toUpperCase()]:0));
 for(let m of order){
  let{nb,nc,nep}=applyMove(board,m,castling,ep);
  let score=minimax(nb,opp(turn),nc,nep,depth-1,-Infinity,Infinity,turn==="w");
  if(turn==="w"?score>bestScore:score<bestScore){bestScore=score;best=m}
 }
 return best||moves[0]
}

function minimax(b,c,cast,epSq,depth,alpha,beta,maximizingWasWhite){
 let moves=legalMoves(b,c,cast,epSq);
 if(depth===0||!moves.length){
  if(!moves.length){
   if(check(b,c))return c==="w"?-99999:99999;
   return 0;
  }
  return evalBoard(b);
 }
 if(c==="w"){
  let v=-Infinity;
  for(let m of moves){
   let{nb,nc,nep}=applyMove(b,m,cast,epSq);
   v=Math.max(v,minimax(nb,"b",nc,nep,depth-1,alpha,beta,maximizingWasWhite));
   alpha=Math.max(alpha,v);if(beta<=alpha)break;
  }
  return v
 }else{
  let v=Infinity;
  for(let m of moves){
   let{nb,nc,nep}=applyMove(b,m,cast,epSq);
   v=Math.min(v,minimax(nb,"w",nc,nep,depth-1,alpha,beta,maximizingWasWhite));
   beta=Math.min(beta,v);if(beta<=alpha)break;
  }
  return v
 }
}

function say(a,b){document.getElementById("status").innerHTML="<b>"+a+"</b><div>"+b+"</div>"}

function click(r,c){
 if(gameOver||busy)return;
 let hc=humanColor();
 if(hc!==null&&turn!==hc)return;
 let p=board[r][c],m=legal.find(x=>x.tr===r&&x.tc===c);
 if(selected&&m){makeMove(m);return}
 if(p&&color(p)===turn){selected=[r,c];render()}
 else{selected=null;render()}
}

function render(){
 const el=document.getElementById("board");el.innerHTML="";if(!board)return;
 const rs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7],cs=flipped?[7,6,5,4,3,2,1,0]:[0,1,2,3,4,5,6,7];
 legal=selected?legalMoves(board,turn,castling,ep).filter(m=>m.fr===selected[0]&&m.fc===selected[1]):[];
 let lastM=moveLog.length?null:null;
 for(let ri=0;ri<8;ri++)for(let ci=0;ci<8;ci++){
  let r=rs[ri],c=cs[ci],s=document.createElement("div");
  s.className="sq "+((r+c)%2?"dark":"light");
  if(selected&&selected[0]===r&&selected[1]===c)s.classList.add("sel");
  let lm=legal.find(m=>m.tr===r&&m.tc===c);if(lm)s.classList.add(board[r][c]?"capture":"move");
  let k=king(board,turn);if(k&&k[0]===r&&k[1]===c&&check(board,turn))s.classList.add("check");
  if(board[r][c]){let z=document.createElement("span");z.className="piece "+(color(board[r][c])==="w"?"wp":"bp");z.textContent=PIECES[board[r][c]];s.appendChild(z)}
  if(ci===0){let z=document.createElement("span");z.className="coord rank";z.textContent=8-r;s.appendChild(z)}
  if(ri===7){let z=document.createElement("span");z.className="coord file";z.textContent=FILES[c];s.appendChild(z)}
  s.onclick=()=>click(r,c);
  el.appendChild(s)
 }
 document.getElementById("moves").innerHTML=history.map((x,i)=>i%2===0?`<div class="row"><span>${i/2+1}.</span><span>${x}</span><span>${history[i+1]||""}</span></div>`:"").join("");
 document.getElementById("moves").scrollTop=document.getElementById("moves").scrollHeight;
 document.getElementById("capWhite").innerHTML=captured.b.map(p=>PIECES[p]).join(" ");
 document.getElementById("capBlack").innerHTML=captured.w.map(p=>PIECES[p]).join(" ");
}

document.getElementById("newgame").onclick=newGame;
document.getElementById("flip").onclick=()=>{flipped=!flipped;render()};
document.getElementById("undo").onclick=()=>{
 if(!moveLog.length||busy)return;
 let last=moveLog.pop();
 board=last.board;castling=last.castling;ep=last.ep;turn=last.turn;captured=last.capturedSnap;
 history.pop();gameOver=false;selected=null;
 if(mode!=="pvp"&&moveLog.length&&turn!==humanColor()){
  let last2=moveLog.pop();
  board=last2.board;castling=last2.castling;ep=last2.ep;turn=last2.turn;captured=last2.capturedSnap;
  history.pop();
 }
 say((turn==="w"?"White":"Black")+" to move","");
 render();
};
document.getElementById("resign").onclick=()=>{
 if(gameOver)return;
 gameOver=true;
 say((opp(turn)==="w"?"White":"Black")+" wins","Resignation");
};
document.getElementById("mode").onchange=newGame;

newGame();
</script>
</body>
</html>
