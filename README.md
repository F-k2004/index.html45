<!DOCTYPE html>
<hml lang="fa">
<head>
<meta charset="UTF-8">
<title>🧠 PID Autopilot</title>
<style>
html,body{margin:0;overflow:hidden;background:#00030a;font-family:system-ui}
canvas{display:block}
.hud{
  position:absolute;left:16px;top:16px;width:320px;
  padding:14px;border-radius:14px;
  background:rgba(255,255,255,0.06);
  backdrop-filter:blur(10px);
  color:#d9f3ff;
}
h3{margin:0 0 10px;color:#9fdcff;font-size:15px}
.row{display:flex;justify-content:space-between;font-size:13px;margin:6px 0}
.bar{height:6px;background:rgba(255,255,255,0.1);border-radius:4px;overflow:hidden}
.fill{height:100%;background:linear-gradient(90deg,#7fdcff,#fff)}
.status{text-align:center;margin-top:10px;font-weight:600}
</style>
</head>
<body>

<canvas id="c"></canvas>

<div class="hud">
  <h3>🧠 AUTOPILOT (PID)</h3>
  <div class="row"><span>Target Orbit</span><span>180 km</span></div>
  <div class="row"><span>Altitude Error</span><span id="errTxt">0</span></div>
  <div class="row"><span>PID Output</span><span id="pidTxt">0</span></div>
  <div class="row"><span>Battery</span><span id="batTxt">100%</span></div>
  <div class="bar"><div class="fill" id="batBar"></div></div>
  <div class="status" id="status">AUTOPILOT ACTIVE</div>
</div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;function r(){w=c.width=innerWidth;h=c.height=innerHeight}
r();addEventListener("resize",r);

// Earth
const earth={x:w/2,y:h/2,r:60,mu:9000};

// Spacecraft
const ship={
  x:earth.x,
  y:earth.y-180,
  vx:2.1,vy:0,
  battery:100,
  thrust:0
};

// PID Controller
const targetAlt = 180;
let integral=0, prevError=0;
const Kp=0.015, Ki=0.0001, Kd=0.04;

// HUD
const errTxt=document.getElementById("errTxt");
const pidTxt=document.getElementById("pidTxt");
const batTxt=document.getElementById("batTxt");
const batBar=document.getElementById("batBar");
const statusEl=document.getElementById("status");

function gravity(){
  let dx=earth.x-ship.x;
  let dy=earth.y-ship.y;
  let d=Math.hypot(dx,dy);
  let f=earth.mu/(d*d);
  ship.vx+=f*dx/d;
  ship.vy+=f*dy/d;
}

function autopilot(){
  const dx=ship.x-earth.x;
  const dy=ship.y-earth.y;
  const dist=Math.hypot(dx,dy);
  const altitude=dist-earth.r;

  const error = targetAlt - altitude;
  integral += error;
  const derivative = error - prevError;
  prevError = error;

  let output = Kp*error + Ki*integral + Kd*derivative;
  output = Math.max(-0.05,Math.min(0.05,output));

  // thrust tangential
  const tangX = -dy/dist;
  const tangY = dx/dist;
  ship.vx += tangX * output;
  ship.vy += tangY * output;

  ship.thrust = Math.abs(output)*20;

  errTxt.textContent = error.toFixed(1)+" km";
  pidTxt.textContent = output.toFixed(4);
}

function updatePower(){
  ship.battery -= ship.thrust*0.02;
  ship.battery=Math.max(0,ship.battery);

  batTxt.textContent=ship.battery.toFixed(0)+"%";
  batBar.style.width=ship.battery+"%";

  if(ship.battery<10){
    statusEl.textContent="⚠️ LOW POWER";
    statusEl.style.color="#ff9a9a";
  }
}

function update(){
  gravity();
  autopilot();

  ship.x+=ship.vx;
  ship.y+=ship.vy;

  updatePower();
}

function draw(){
  ctx.fillStyle="rgba(0,3,10,0.35)";
  ctx.fillRect(0,0,w,h);

  update();

  ctx.beginPath();
  ctx.arc(earth.x,earth.y,earth.r,0,Math.PI*2);
  ctx.fillStyle="#0b3d91";
  ctx.fill();

  ctx.beginPath();
  ctx.arc(ship.x,ship.y,4,0,Math.PI*2);
  ctx.fillStyle="#e6f7ff";
  ctx.fill();

  requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>

