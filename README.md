<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">

<title>Flight Simulator</title>

<style>
* {
    box-sizing: border-box;
    touch-action: none;
}

body {
    margin: 0;
    overflow: hidden;
    background: #6bb7e8;
    font-family: Arial, sans-serif;
    color: white;
}

#game {
    position: fixed;
    inset: 0;
    overflow: hidden;
    background:
        linear-gradient(#63b4e6 0%, #a9dcf5 55%, #79b85b 56%, #477c35 100%);
}

/* Clouds */
.cloud {
    position: absolute;
    width: 180px;
    height: 60px;
    background: white;
    border-radius: 50px;
    opacity: .75;
}

.cloud:before,
.cloud:after {
    content: "";
    position: absolute;
    background: white;
    border-radius: 50%;
}

.cloud:before {
    width: 75px;
    height: 75px;
    left: 30px;
    top: -35px;
}

.cloud:after {
    width: 90px;
    height: 90px;
    right: 25px;
    top: -45px;
}

/* HUD */
#hud {
    position: absolute;
    top: 15px;
    left: 15px;
    z-index: 10;
    background: rgba(0,0,0,.55);
    padding: 12px 16px;
    border-radius: 14px;
    min-width: 170px;
    line-height: 1.7;
    font-size: 15px;
}

#hud b {
    color: #ffd84d;
}

/* Plane */
#plane {
    position: absolute;
    left: 50%;
    top: 48%;
    width: 100px;
    height: 35px;
    transform: translate(-50%, -50%);
    transition: transform .05s linear;
    z-index: 5;
}

.fuselage {
    position: absolute;
    width: 100px;
    height: 18px;
    top: 9px;
    left: 0;
    background: #eee;
    border-radius: 50% 65% 65% 50%;
}

.nose {
    position: absolute;
    right: -5px;
    top: 8px;
    width: 25px;
    height: 20px;
    background: #ddd;
    border-radius: 0 100% 100% 0;
}

.wing {
    position: absolute;
    width: 80px;
    height: 12px;
    left: 10px;
    top: 12px;
    background: #ccc;
    transform: rotate(-8deg);
    border-radius: 5px;
}

.tail {
    position: absolute;
    width: 28px;
    height: 25px;
    left: 8px;
    top: -3px;
    background: #bbb;
    clip-path: polygon(0 100%, 100% 100%, 60% 0);
}

/* Joystick */
#joystick {
    position: absolute;
    bottom: 35px;
    left: 35px;
    width: 150px;
    height: 150px;
    border-radius: 50%;
    background: rgba(0,0,0,.25);
    border: 2px solid rgba(255,255,255,.5);
    z-index: 20;
}

#stick {
    position: absolute;
    width: 65px;
    height: 65px;
    border-radius: 50%;
    background: rgba(255,255,255,.8);
    left: 42px;
    top: 42px;
    box-shadow: 0 5px 15px rgba(0,0,0,.3);
}

/* Throttle */
#throttleBox {
    position: absolute;
    right: 25px;
    bottom: 35px;
    width: 70px;
    height: 180px;
    background: rgba(0,0,0,.3);
    border-radius: 15px;
    border: 2px solid rgba(255,255,255,.5);
    z-index: 20;
}

#throttle {
    position: absolute;
    width: 50px;
    height: 35px;
    left: 8px;
    bottom: 20px;
    background: #ffd84d;
    border-radius: 10px;
}

#throttleLabel {
    position: absolute;
    top: 8px;
    width: 100%;
    text-align: center;
    font-size: 12px;
}

/* Buttons */
.buttons {
    position: absolute;
    right: 110px;
    bottom: 30px;
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
    width: 230px;
    z-index: 20;
}

button {
    border: 0;
    background: rgba(0,0,0,.6);
    color: white;
    padding: 12px;
    border-radius: 12px;
    font-size: 13px;
    min-width: 70px;
}

button:active {
    background: #d99f00;
}

/* Mobile */
@media(max-width:700px) {
    #hud {
        font-size: 12px;
        min-width: 145px;
    }

    #joystick {
        width: 125px;
        height: 125px;
    }

    #stick {
        width: 55px;
        height: 55px;
        left: 33px;
        top: 33px;
    }

    .buttons {
        right: 90px;
        width: 190px;
    }
}
</style>
</head>

<body>

<div id="game">

    <div class="cloud" style="left:10%;top:20%;"></div>
    <div class="cloud" style="left:65%;top:15%;transform:scale(.7);"></div>
    <div class="cloud" style="left:45%;top:30%;transform:scale(.5);"></div>

    <div id="hud">
        SPEED: <b id="speed">120</b> km/h<br>
        ALT: <b id="alt">1200</b> ft<br>
        HEADING: <b id="heading">090</b>°<br>
        THROTTLE: <b id="power">50</b>%<br>
        FLAPS: <b id="flaps">0</b>°
    </div>

    <div id="plane">
        <div class="fuselage"></div>
        <div class="nose"></div>
        <div class="wing"></div>
        <div class="tail"></div>
    </div>

    <!-- Joystick -->
    <div id="joystick">
        <div id="stick"></div>
    </div>

    <!-- Throttle -->
    <div id="throttleBox">
        <div id="throttleLabel">THROTTLE</div>
        <div id="throttle"></div>
    </div>

    <div class="buttons">
        <button id="flapsBtn">FLAPS</button>
        <button id="gearBtn">GEAR</button>
        <button id="brakeBtn">BRAKE</button>
        <button id="trimUp">TRIM +</button>
        <button id="trimDown">TRIM -</button>
    </div>

</div>

<script>

const plane = document.getElementById("plane");
const joystick = document.getElementById("joystick");
const stick = document.getElementById("stick");

let pitch = 0;
let roll = 0;
let heading = 90;

let speed = 120;
let altitude = 1200;
let throttle = 50;

let flaps = 0;
let gear = true;
let brake = false;
let trim = 0;

let joyActive = false;

function updateHUD() {

    document.getElementById("speed").textContent =
        Math.round(speed);

    document.getElementById("alt").textContent =
        Math.round(altitude);

    document.getElementById("heading").textContent =
        String(Math.round(heading + 360) % 360).padStart(3,"0");

    document.getElementById("power").textContent =
        Math.round(throttle);

    document.getElementById("flaps").textContent =
        flaps;
}

function moveJoystick(x,y) {

    const rect = joystick.getBoundingClientRect();

    let cx = rect.left + rect.width / 2;
    let cy = rect.top + rect.height / 2;

    let dx = x - cx;
    let dy = y - cy;

    const max = rect.width / 2 - 32;

    const distance = Math.sqrt(dx*dx + dy*dy);

    if(distance > max) {
        dx = dx / distance * max;
        dy = dy / distance * max;
    }

    stick.style.left =
        (rect.width/2 - 32 + dx) + "px";

    stick.style.top =
        (rect.height/2 - 32 + dy) + "px";

    roll = dx / max;
    pitch = -dy / max;
}

function resetJoystick() {

    stick.style.left = "42px";
    stick.style.top = "42px";

    if(window.innerWidth < 700) {
        stick.style.left = "33px";
        stick.style.top = "33px";
    }

    pitch = 0;
    roll = 0;
}

joystick.addEventListener("pointerdown", e => {

    joyActive = true;

    joystick.setPointerCapture(e.pointerId);

    moveJoystick(e.clientX,e.clientY);
});

joystick.addEventListener("pointermove", e => {

    if(joyActive)
        moveJoystick(e.clientX,e.clientY);
});

joystick.addEventListener("pointerup", e => {

    joyActive = false;

    resetJoystick();
});

/* Throttle */

const throttleBox =
    document.getElementById("throttleBox");

const throttleHandle =
    document.getElementById("throttle");

let throttleActive = false;

throttleBox.addEventListener("pointerdown", e => {

    throttleActive = true;

    throttleBox.setPointerCapture(e.pointerId);

    updateThrottle(e.clientY);
});

throttleBox.addEventListener("pointermove", e => {

    if(throttleActive)
        updateThrottle(e.clientY);
});

throttleBox.addEventListener("pointerup", () => {

    throttleActive = false;
});

function updateThrottle(y) {

    const rect =
        throttleBox.getBoundingClientRect();

    let percent =
        1 - ((y - rect.top) / rect.height);

    percent =
        Math.max(0,Math.min(1,percent));

    throttle = percent * 100;

    throttleHandle.style.bottom =
        (percent * (rect.height-60) + 10) + "px";
}

/* Buttons */

document.getElementById("flapsBtn")
.addEventListener("click", () => {

    flaps += 10;

    if(flaps > 30)
        flaps = 0;
});

document.getElementById("gearBtn")
.addEventListener("click", () => {

    gear = !gear;

    document.getElementById("gearBtn").textContent =
        gear ? "GEAR DOWN" : "GEAR UP";
});

document.getElementById("brakeBtn")
.addEventListener("pointerdown", () => {

    brake = true;
});

document.getElementById("brakeBtn")
.addEventListener("pointerup", () => {

    brake = false;
});

document.getElementById("trimUp")
.addEventListener("click", () => {

    trim += 1;
});

document.getElementById("trimDown")
.addEventListener("click", () => {

    trim -= 1;
});

/* Flight Physics */

function flightLoop() {

    /*
       Engine power
    */

    let acceleration =
        (throttle - 50) * 0.015;

    if(brake)
        acceleration -= 1.5;

    speed += acceleration;

    speed = Math.max(0,Math.min(900,speed));

    /*
       Pitch
    */

    altitude +=
        pitch * speed * 0.006;

    altitude +=
        trim * 0.01;

    altitude =
        Math.max(0,altitude);

    /*
       Roll / Heading
    */

    heading +=
        roll * speed * 0.025;

    /*
       Plane rotation
    */

    let visualRoll =
        roll * 35;

    let visualPitch =
        pitch * 15;

    plane.style.transform =
        `translate(-50%,-50%)
         rotateZ(${visualRoll}deg)
         rotateX(${visualPitch}deg)`;

    /*
       Camera / horizon movement
    */

    let horizon =
        (altitude - 1200) * 0.02;

    document.getElementById("game").style
        .backgroundPosition =
        `0px ${horizon}px`;

    updateHUD();

    requestAnimationFrame(flightLoop);
}

updateHUD();
flightLoop();

</script>

</body>
</html>
