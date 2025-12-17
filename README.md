<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<title>來財五軸老虎機 - 商用版</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{margin:0;background:radial-gradient(circle,#222,#000);color:#fff;font-family:"Microsoft JhengHei",sans-serif;text-align:center;}
h1{color:gold;margin:20px 0;text-shadow:2px 2px 5px #000;}
.slot-container{display:flex;justify-content:center;margin:20px 0;}
.column{display:flex;flex-direction:column;margin:0 5px;}
.slot{font-size:45px;width:60px;height:60px;line-height:60px;margin:5px 0;background:#111;border-radius:10px;box-shadow:inset 0 0 15px #000;transition:all 0.3s;}
button{margin:8px;padding:10px 18px;font-size:16px;border:none;border-radius:10px;background:gold;color:#000;font-weight:bold;}
input{width:90px;padding:5px;font-size:16px;text-align:center;}
.info{margin-top:8px;font-size:18px;}
.free-mode{background:radial-gradient(circle,#ff0,#f80);}
.flash{animation:flash 0.5s;}
@keyframes flash{0%{background:#ff0;color:#000;}50%{background:#f00;color:#fff;}100%{background:#111;color:#fff;}}
.flywin{position:absolute;font-size:24px;color:gold;font-weight:bold;animation:fly 1s ease;}
@keyframes fly{0%{top:50%;opacity:1;}100%{top:0;opacity:0;}}
.jackpot-effect{position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(255,215,0,0.9);color:#000;font-size:60px;display:flex;justify-content:center;align-items:center;animation:jackpotFlash 2s ease;}
@keyframes jackpotFlash{0%{opacity:0;}50%{opacity:1;}100%{opacity:0;}}
@media(max-width:600px){.slot{font-size:32px;width:45px;height:45px;line-height:45px;}}
</style>
</head>
<body>
<h1>🎰 來財五軸老虎機 - 商用版 🎰</h1>
<div class="slot-container" id="slots">
    <div class="column"><div class="slot">❓</div><div class="slot">❓</div><div class="slot">❓</div></div>
    <div class="column"><div class="slot">❓</div><div class="slot">❓</div><div class="slot">❓</div></div>
    <div class="column"><div class="slot">❓</div><div class="slot">❓</div><div class="slot">❓</div></div>
    <div class="column"><div class="slot">❓</div><div class="slot">❓</div><div class="slot">❓</div></div>
    <div class="column"><div class="slot">❓</div><div class="slot">❓</div><div class="slot">❓</div></div>
</div>

<div>下注金額：<input type="number" id="bet" value="50" min="1"></div>
<button onclick="spin()">拉動老虎機</button>
<button onclick="buyFree()">🎁 購買免費遊戲 (100×)</button>

<div class="info" id="message"></div>
<div class="info" id="balance"></div>
<div class="info" id="freeSpins"></div>
<div class="info" id="jackpot"></div>
<div class="info" id="history"></div>
<div class="info" id="rtpPanel">🎲 RTP: <input type="number" id="rtpInput" value="0.95" step="0.01" min="0.5" max="1"></div>

<!-- 音效 -->
<audio id="soundSpin" src="https://www.soundjay.com/misc/sounds/button-16.mp3"></audio>
<audio id="soundWin" src="https://www.soundjay.com/misc/sounds/bell-ringing-01.mp3"></audio>
<audio id="soundJackpot" src="https://www.soundjay.com/misc/sounds/bell-ringing-02.mp3"></audio>

<script>
// 初始化
let balance=1000, freeSpins=0, jackpot=0;
let historyArr=[];
let rtp=0.95;
const baseSymbols=["🍒","🍋","🍉","⭐","💎"], freeSymbol="🎁", multiSymbol="🔴";
updateUI();

// RTP 控制
document.getElementById("rtpInput").addEventListener("change",e=>{rtp=parseFloat(e.target.value);});

// 拉桿
function spin(){
    let bet=+document.getElementById("bet").value;
    if(freeSpins===0 && bet>balance){alert("餘額不足"); return;}
    document.getElementById("soundSpin").play();

    let grid=roll(freeSpins>0);
    draw(grid);
    animate(grid);

    // 免費遊戲判定
    if(grid.flat().filter(s=>s===freeSymbol).length>=3){startFree(); msg("🎁 觸發免費遊戲 20 局"); return;}

    let win=calcWin(grid,bet);
    if(freeSpins>0){freeSpins--; balance+=win;} else balance+=win;
    historyArr.push({grid,bet,win,free:freeSpins>0});

    // Jackpot 判定
    if(checkJackpot(grid)){triggerJackpot(); balance+=jackpot; jackpot=0;}

    if(win>0){document.getElementById("soundWin").play(); flyWin(win);}
    msg(win>0?`🎉 中獎 +${win}`:"😢 沒中獎");
    if(freeSpins===0) document.body.classList.remove("free-mode");
    updateUI();
}

// 購買免費遊戲
function buyFree(){
    let bet=+document.getElementById("bet").value;
    let cost=bet*100;
    if(cost>balance){alert("餘額不足購買免費遊戲"); return;}
    balance-=cost; startFree(); msg(`💰 花費 ${cost} 購買免費遊戲`);
    updateUI();
}

function startFree(){freeSpins=20; document.body.classList.add("free-mode");}

// 生成五軸滾輪
function roll(freeMode){
    let pool=[...baseSymbols,freeSymbol];
    if(freeMode){
        pool.push(multiSymbol,multiSymbol,multiSymbol); // 免費遊戲增加倍數球
    } else { pool.push(multiSymbol);}
    let grid=[];
    for(let c=0;c<5;c++){let col=[];for(let r=0;r<3;r++){
        // RTP控制
        col.push(Math.random()<rtp ? pool[Math.floor(Math.random()*pool.length)] : baseSymbols[Math.floor(Math.random()*baseSymbols.length)]);
    }grid.push(col);}
    return grid;
}

// 計算 1024 支付線
function calcWin(g,bet){
    let multiCount=g.flat().filter(s=>s===multiSymbol).length;
    let m=multiCount===1?2:multiCount===2?3:multiCount>=3?5:1;
    let win=0;
    // 1024條支付線簡化示範：每種可能連線
    let lines=[
        [[0,0],[1,0],[2,0],[3,0],[4,0]], // 第一行
        [[0,1],[1,1],[2,1],[3,1],[4,1]], // 第二行
        [[0,2],[1,2],[2,2],[3,2],[4,2]], // 第三行
        [[0,0],[1,1],[2,1],[3,1],[4,2]], // V字斜線
        [[0,2],[1,1],[2,1],[3,1],[4,0]]  // 反V字
    ];
    lines.forEach(line=>{
        let symbols=line.map(p=>g[p[0]][p[1]]);
        if(symbols.every(v=>v===symbols[0])) win+=bet*5*m;
        else if(new Set(symbols).size<5) win+=bet*2*m;
    });
    jackpot+=bet*0.01;
    return freeSpins>0?win:win>0?win:-bet;
}

// Jackpot
function checkJackpot(g){return g.flat().every(s=>s===multiSymbol) && jackpot>0;}
function triggerJackpot(){
    document.getElementById("soundJackpot").play();
    let div=document.createElement("div"); div.className="jackpot-effect"; div.innerText="💥 JACKPOT 💥"; document.body.appendChild(div);
    setTimeout(()=>div.remove(),2000);
}

// 更新畫面
function draw(g){const cols=document.querySelectorAll(".column"); for(let c=0;c<5;c++) for(let r=0;r<3;r++) cols[c].children[r].innerText=g[c][r];}
function animate(g){const slots=document.querySelectorAll(".slot"); slots.forEach(s=>{s.classList.add("flash"); setTimeout(()=>s.classList.remove("flash"),400);});}
function flyWin(win){let f=document.createElement("div"); f.className="flywin"; f.innerText=`+${win}`; document.body.appendChild(f); setTimeout(()=>f.remove(),1000);}
function msg(t){document.getElementById("message").innerText=t;}
function updateUI(){
    document.getElementById("balance").innerText=`💰 餘額：${balance}`;
    document.getElementById("freeSpins").innerText=freeSpins>0?`🎁 免費遊戲剩餘：${freeSpins}`:"";
    document.getElementById("jackpot").innerText=`🏆 Jackpot：${Math.floor(jackpot)}`;
    // 歷史紀錄顯示最近10筆
    let hist=document.getElementById("history"); hist.innerHTML=""; for(let i=Math.max(0,historyArr.length-10);i<historyArr.length;i++){
        let h=historyArr[i]; let div=document.createElement("div"); div.innerText=`下注 ${h.bet} ${h.free?"(免費)":" "}- 贏 ${h.win}`;
        hist.appendChild(div);
    }
}
</script>
</body>
</html>