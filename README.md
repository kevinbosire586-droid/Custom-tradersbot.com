<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bosire Trader Bot</title>
i
<style>
*{
margin:0;
padding:0;
box-sizing:border-box;
font-family:Arial,sans-serif;
}

body{
background:#0d1117;
color:white;
}

header{
display:flex;
justify-content:space-between;
align-items:center;
padding:20px 40px;
background:#161b22;
}

.logo{
font-size:28px;
font-weight:bold;
color:#00ff99;
}

nav a{
color:white;
text-decoration:none;
margin:0 15px;
}

nav a:hover{
color:#00ff99;
}

.login{
background:#00ff99;
color:black;
padding:12px 25px;
border:none;
border-radius:8px;
cursor:pointer;
font-weight:bold;
}

.hero{
height:60vh;
display:flex;
flex-direction:column;
justify-content:center;
align-items:center;
text-align:center;
background:linear-gradient(135deg,#0d1117,#1b263b);
}

.hero h1{
font-size:60px;
margin-bottom:15px;
}

.hero p{
color:#ccc;
margin-bottom:25px;
}

.start{
background:#00ff99;
color:black;
padding:15px 40px;
border:none;
border-radius:10px;
cursor:pointer;
font-weight:bold;
}

.dashboard{
display:grid;
grid-template-columns:repeat(auto-fit,minmax(250px,1fr));
gap:20px;
padding:30px;
}

.card{
background:#161b22;
padding:25px;
border-radius:15px;
text-align:center;
box-shadow:0 0 15px rgba(0,255,153,.2);
}

.trade,
.api,
.chart-section,
.history{
padding:30px;
text-align:center;
}

input,select{
width:300px;
padding:15px;
margin:10px;
background:#161b22;
color:white;
border:1px solid #333;
border-radius:10px;
}

button{
padding:15px 25px;
margin:10px;
border:none;
border-radius:10px;
cursor:pointer;
font-weight:bold;
}

.buy-btn{
background:#00ff99;
color:black;
}

.sell-btn{
background:#ff4444;
color:white;
}

table{
width:100%;
border-collapse:collapse;
margin-top:20px;
background:#161b22;
}

th,td{
padding:15px;
border-bottom:1px solid #333;
}

th{
background:#00ff99;
color:black;
}

#chart{
height:350px;
background:#161b22;
border-radius:15px;
display:flex;
justify-content:center;
align-items:center;
font-size:24px;
color:#00ff99;
}

@media(max-width:768px){

header{
flex-direction:column;
}

nav{
margin-top:15px;
}

.hero h1{
font-size:38px;
}

input,select{
width:95%;
}
}
</style>
</head>

<body>

<header>
<div class="logo">Bosire Trader</div>

<nav>
<a href="#">Home</a>
<a href="#">Dashboard</a>
<a href="#">Bot</a>
<a href="#">Markets</a>
</nav>

<button class="login">Connect Deriv</button>
</header>

<section class="hero">
<h1>Automated Trading Bot</h1>
<p>AI Powered Trading Dashboard</p>
<button class="start" onclick="startBot()">Start Bot</button>
</section>

<section class="dashboard">

<div class="card">
<h3>Balance</h3>
<h2 id="balance">$10,000</h2>
</div>

<div class="card">
<h3>Bot Status</h3>
<h2 id="status">Stopped</h2>
</div>

<div class="card">
<h3>Signal</h3>
<h2 id="signal">Waiting...</h2>
</div>

<div class="card">
<h3>Total Profit</h3>
<h2 id="profit">$0.00</h2>
</div>

</section>

<section class="api">
<h2>Deriv API Connection</h2>

<input type="text" id="token" placeholder="Enter API Token">

<button class="buy-btn" onclick="connectDeriv()">
Connect
</button>

<p id="connection">Not Connected</p>
</section>

<section class="trade">

<h2>Trading Controls</h2>

<input type="number" id="amount" placeholder="Trade Amount">

<select id="market">
<option>Volatility 75</option>
<option>Volatility 100</option>
<option>Boom 500</option>
<option>Crash 500</option>
</select>

<br>

<button class="buy-btn" onclick="startBot()">
Start Bot
</button>

<button class="sell-btn" onclick="stopBot()">
Stop Bot
</button>

</section>

<section class="chart-section">
<h2>Market Chart</h2>
<div id="chart">📈 Live Chart Area</div>
</section>

<section class="history">

<h2>Trade History</h2>

<table>

<thead>
<tr>
<th>Time</th>
<th>Market</th>
<th>Signal</th>
<th>Amount</th>
<th>Result</th>
</tr>
</thead>

<tbody id="historyTable"></tbody>

</table>

</section>

<script>

let running=false;
let profit=0;
let trades=0;
let timer;
let socket;

function startBot(){

if(running) return;

running=true;

document.getElementById("status").innerHTML=
"🟢 Running";

timer=setInterval(makeTrade,5000);

}

function stopBot(){

running=false;

clearInterval(timer);

document.getElementById("status").innerHTML=
"🔴 Stopped";

}

function makeTrade(){

const signal=Math.random()>0.5?
"BUY 📈":"SELL 📉";

const result=Math.random()>0.5?
"WIN":"LOSS";

const amount=
Number(document.getElementById("amount").value)||10;

if(result==="WIN"){
profit+=amount*0.95;
}else{
profit-=amount;
}

document.getElementById("signal").innerHTML=
signal;

document.getElementById("profit").innerHTML=
"$"+profit.toFixed(2);

const row=document.createElement("tr");

row.innerHTML=`
<td>${new Date().toLocaleTimeString()}</td>
<td>${document.getElementById("market").value}</td>
<td>${signal}</td>
<td>$${amount}</td>
<td>${result}</td>
`;

document.getElementById("historyTable")
.prepend(row);

}

function connectDeriv(){

const token=
document.getElementById("token").value;

socket=new WebSocket(
"wss://ws.derivws.com/websockets/v3?app_id=1089"
);

socket.onopen=()=>{

document.getElementById("connection").innerHTML=
"🟢 Connected";

socket.send(JSON.stringify({
authorize:token
}));

};

socket.onmessage=(event)=>{

const data=JSON.parse(event.data);

console.log(data);

};

socket.onerror=()=>{

document.getElementById("connection").innerHTML=
"❌ Error";

};

socket.onclose=()=>{

document.getElementById("connection").innerHTML=
"🔴 Disconnected";

};

}
</script>

</body>
</html>
