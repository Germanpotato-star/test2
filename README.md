<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Football Game</title>

<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
}

body{
    overflow:hidden;
    background:#111;
    font-family:Arial,sans-serif;
}

canvas{
    display:block;
}

#hud{
    position:absolute;
    top:10px;
    left:50%;
    transform:translateX(-50%);
    color:white;
    text-align:center;
    z-index:100;
    text-shadow:2px 2px 4px #000;
}

#score{
    font-size:32px;
    font-weight:bold;
}

#timer{
    font-size:20px;
    margin-top:5px;
}

#staminaWrap{
    width:250px;
    height:20px;
    background:#333;
    border:2px solid white;
    margin-top:8px;
}

#stamina{
    width:100%;
    height:100%;
    background:limegreen;
}

#controls{
    position:absolute;
    left:10px;
    bottom:10px;
    color:white;
    background:rgba(0,0,0,.6);
    padding:10px;
    border-radius:8px;
    z-index:100;
}
</style>
</head>
<body>

<div id="hud">
    <div id="score">0 - 0</div>
    <div id="timer">90</div>

    <div id="staminaWrap">
        <div id="stamina"></div>
    </div>
</div>

<div id="controls">
WASD = Move<br>
SHIFT = Sprint<br>
SPACE = Shoot
</div>

<canvas id="game"></canvas>

<script>

const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

function resize(){
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;
}
resize();
window.addEventListener("resize", resize);

/* SMALLER FIELD */
const FIELD_W = 1800;
const FIELD_H = 1100;

/* BACKGROUND IMAGE IN ROOT */
const backgroundImage = new Image();
backgroundImage.src = "sprite_1280x720.png";

let scoreA = 0;
let scoreB = 0;
let matchTime = 90;

const keys = {};

document.addEventListener("keydown",e=>{
    keys[e.key.toLowerCase()] = true;
});

document.addEventListener("keyup",e=>{
    keys[e.key.toLowerCase()] = false;
});

setInterval(()=>{
    if(matchTime > 0){
        matchTime--;
        document.getElementById("timer").textContent = matchTime;
    }
},1000);

const camera={
    x:0,
    y:0
};

class Player{

    constructor(x,y,color,human=false){

        this.x=x;
        this.y=y;
        this.radius=18;
        this.color=color;
        this.human=human;
        this.stamina=100;
    }

    update(){

        if(this.human){

            /* 15% FASTER */
            let speed=4.6;

            if(keys["shift"] && this.stamina>0){

                speed=8.05;
                this.stamina-=0.4;

            }else{

                this.stamina+=0.2;
            }

            this.stamina=Math.max(
                0,
                Math.min(100,this.stamina)
            );

            document.getElementById("stamina")
                .style.width=this.stamina+"%";

            if(keys["w"]) this.y-=speed;
            if(keys["s"]) this.y+=speed;
            if(keys["a"]) this.x-=speed;
            if(keys["d"]) this.x+=speed;

            this.x=Math.max(20,Math.min(FIELD_W-20,this.x));
            this.y=Math.max(20,Math.min(FIELD_H-20,this.y));
        }
    }

    draw(){

        ctx.fillStyle=this.color;

        ctx.beginPath();

        ctx.arc(
            this.x-camera.x,
            this.y-camera.y,
            this.radius,
            0,
            Math.PI*2
        );

        ctx.fill();
    }
}

const player =
new Player(
    250,
    FIELD_H/2,
    "#00bfff",
    true
);

const teammates=[];
const enemies=[];

for(let i=0;i<5;i++){

    teammates.push(
        new Player(
            400+Math.random()*250,
            150+i*170,
            "#66d9ff"
        )
    );

    enemies.push(
        new Player(
            1100+Math.random()*250,
            150+i*170,
            "#ff5555"
        )
    );
}

const goalie={
    x:FIELD_W-70,
    y:FIELD_H/2
};

const ball={
    x:FIELD_W/2,
    y:FIELD_H/2,
    vx:0,
    vy:0,
    radius:12
};

function shoot(){

    const d=Math.hypot(
        ball.x-player.x,
        ball.y-player.y
    );

    if(d<60){

        ball.vx=16;
        ball.vy=(Math.random()-0.5)*4;
    }
}

document.addEventListener("keydown",e=>{

    if(e.code==="Space"){
        shoot();
    }
});

function resetKickoff(){

    ball.x=FIELD_W/2;
    ball.y=FIELD_H/2;
    ball.vx=0;
    ball.vy=0;
}

function checkGoal(){

    if(
        ball.x > FIELD_W-10 &&
        ball.y > FIELD_H/2-120 &&
        ball.y < FIELD_H/2+120
    ){
        scoreA++;
        resetKickoff();
    }

    if(
        ball.x < 10 &&
        ball.y > FIELD_H/2-120 &&
        ball.y < FIELD_H/2+120
    ){
        scoreB++;
        resetKickoff();
    }

    document.getElementById("score")
        .textContent =
        scoreA + " - " + scoreB;
}

function updateBall(){

    const dx=ball.x-player.x;
    const dy=ball.y-player.y;

    const dist=Math.hypot(dx,dy);

    if(dist<35){

        ball.vx+=dx*0.02;
        ball.vy+=dy*0.02;
    }

    enemies.forEach(enemy=>{

        const angle=Math.atan2(
            ball.y-enemy.y,
            ball.x-enemy.x
        );

        enemy.x+=Math.cos(angle)*1.4;
        enemy.y+=Math.sin(angle)*1.4;

        const d=Math.hypot(
            ball.x-enemy.x,
            ball.y-enemy.y
        );

        if(d<30){

            ball.vx+=(ball.x-enemy.x)*0.1;
            ball.vy+=(ball.y-enemy.y)*0.1;
        }
    });

    goalie.y += (ball.y-goalie.y)*0.05;

    if(
        ball.x>FIELD_W-90 &&
        Math.abs(ball.y-goalie.y)<60
    ){
        ball.vx*=-1;
    }

    ball.x+=ball.vx;
    ball.y+=ball.vy;

    ball.vx*=0.985;
    ball.vy*=0.985;

    if(ball.y<20 || ball.y>FIELD_H-20){
        ball.vy*=-1;
    }

    checkGoal();
}

function drawField(){

    if(backgroundImage.complete){

        ctx.drawImage(
            backgroundImage,
            -camera.x,
            -camera.y,
            FIELD_W,
            FIELD_H
        );

    }else{

        ctx.fillStyle="#1e8f40";
        ctx.fillRect(
            -camera.x,
            -camera.y,
            FIELD_W,
            FIELD_H
        );
    }

    ctx.strokeStyle="rgba(255,255,255,0.7)";
    ctx.lineWidth=4;

    ctx.strokeRect(
        -camera.x,
        -camera.y,
        FIELD_W,
        FIELD_H
    );

    ctx.beginPath();
    ctx.moveTo(
        FIELD_W/2-camera.x,
        -camera.y
    );
    ctx.lineTo(
        FIELD_W/2-camera.x,
        FIELD_H-camera.y
    );
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(
        FIELD_W/2-camera.x,
        FIELD_H/2-camera.y,
        90,
        0,
        Math.PI*2
    );
    ctx.stroke();
}

function drawBall(){

    ctx.fillStyle="white";

    ctx.beginPath();

    ctx.arc(
        ball.x-camera.x,
        ball.y-camera.y,
        ball.radius,
        0,
        Math.PI*2
    );

    ctx.fill();
}

function drawMinimap(){

    const w=200;
    const h=120;

    const x=canvas.width-w-15;
    const y=15;

    ctx.fillStyle="rgba(0,0,0,.5)";
    ctx.fillRect(x,y,w,h);

    ctx.strokeStyle="white";
    ctx.strokeRect(x,y,w,h);

    ctx.fillStyle="white";

    ctx.beginPath();
    ctx.arc(
        x+(ball.x/FIELD_W)*w,
        y+(ball.y/FIELD_H)*h,
        4,
        0,
        Math.PI*2
    );
    ctx.fill();

    ctx.fillStyle="cyan";

    ctx.beginPath();
    ctx.arc(
        x+(player.x/FIELD_W)*w,
        y+(player.y/FIELD_H)*h,
        5,
        0,
        Math.PI*2
    );
    ctx.fill();
}

function update(){

    player.update();

    teammates.forEach(t=>{

        t.x+=(ball.x-200-t.x)*0.002;
        t.y+=(ball.y-t.y)*0.002;
    });

    updateBall();

    camera.x=
        player.x-canvas.width/2;

    camera.y=
        player.y-canvas.height/2;

    camera.x=Math.max(
        0,
        Math.min(
            FIELD_W-canvas.width,
            camera.x
        )
    );

    camera.y=Math.max(
        0,
        Math.min(
            FIELD_H-canvas.height,
            camera.y
        )
    );
}

function render(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    drawField();

    teammates.forEach(t=>t.draw());
    enemies.forEach(e=>e.draw());

    player.draw();

    ctx.fillStyle="orange";

    ctx.fillRect(
        goalie.x-10-camera.x,
        goalie.y-50-camera.y,
        20,
        100
    );

    drawBall();

    drawMinimap();
}

function loop(){

    update();
    render();

    requestAnimationFrame(loop);
}

loop();

</script>
</body>
</html>
