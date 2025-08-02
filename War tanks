<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
<title>Tank Game with Roaming AI Tanks</title>
<style>
  body {
    margin: 0;
    overflow: hidden;
    font-family: Arial, sans-serif;
    background: #c0c0c0;
    -webkit-user-select: none;
    user-select: none;
    touch-action: manipulation;
  }
  #gameContainer {
    position: relative;
    width: 100vw;
    height: 100vh;
    background: #c0c0c0;
  }
  canvas {
    background-color: #c0c0c0;
    display: block;
    margin: 0 auto;
    width: 100%;
    height: 60vh;
    max-height: 400px;
    border: 1px solid black;
    touch-action: none;
  }
  #healthBarContainer, #reloadBarContainer {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    width: 80vw;
    max-width: 300px;
    height: 12px;
    border-radius: 6px;
    overflow: hidden;
    box-shadow: 0 0 3px #0003 inset;
    background: #aaa;
    user-select: none;
    z-index: 10;
  }
  #healthBarContainer {
    bottom: 85px;
    background-color: #aaa;
  }
  #reloadBarContainer {
    bottom: 60px;
    background-color: #ccc;
  }
  #healthBar {
    height: 100%;
    width: 100%;
    background-color: red;
    transition: width 0.2s ease-out;
  }
  #reloadBar {
    height: 100%;
    width: 0%;
    background-color: green;
    transition: width 0.1s linear;
  }
  #fireButton {
    position: absolute;
    bottom: 15px;
    right: 15px;
    width: 80px;
    height: 80px;
    border-radius: 50%;
    font-size: 20px;
    font-weight: bold;
    color: white;
    background-color: darkred;
    border: none;
    z-index: 20;
    user-select: none;
    touch-action: manipulation;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
  }
  #controls {
    position: absolute;
    bottom: 15px;
    left: 15px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    z-index: 20;
    user-select: none;
  }
  .control-row {
    display: flex;
    gap: 10px;
    justify-content: center;
  }
  .control-btn {
    width: 70px;
    height: 70px;
    border-radius: 50%;
    font-size: 26px;
    font-weight: bold;
    background-color: #333;
    color: white;
    border: none;
    user-select: none;
    touch-action: manipulation;
    box-shadow: 0 4px 8px rgba(0,0,0,0.4);
  }
  .control-btn:active {
    background-color: #555;
  }
  #countryControls {
    text-align: center;
    margin: 15px auto;
    max-width: 320px;
  }
  #startBtn {
    margin-left: 10px;
    padding: 8px 16px;
    font-size: 16px;
    cursor: pointer;
    user-select: none;
    border-radius: 5px;
    border: 1px solid #333;
    background-color: white;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
  }
  #countrySelect {
    font-size: 16px;
    padding: 4px 8px;
    border-radius: 5px;
    border: 1px solid #333;
  }
</style>
</head>
<body>

<div id="countryControls">
  <label for="countrySelect">Choose your country:</label>
  <select id="countrySelect">
    <option value="Germany">Germany (gray)</option>
    <option value="France">France (blue)</option>
    <option value="UK">UK (red)</option>
  </select>
  <button id="startBtn" onclick="startGame()">Start Game</button>
</div>

<div id="gameContainer" style="display:none;">
  <canvas id="gameCanvas" width="600" height="400"></canvas>

  <div id="healthBarContainer">
    <div id="healthBar"></div>
  </div>

  <div id="reloadBarContainer">
    <div id="reloadBar"></div>
  </div>

  <div id="controls">
    <div class="control-row">
      <button class="control-btn" id="btnLeft">←</button>
      <button class="control-btn" id="btnForward">↑</button>
      <button class="control-btn" id="btnRight">→</button>
    </div>
    <div class="control-row" style="justify-content:center;">
      <button class="control-btn" id="btnBackward">↓</button>
    </div>
  </div>

  <button id="fireButton" onclick="fire()">Fire</button>
</div>

<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");
  const reloadBar = document.getElementById("reloadBar");
  const healthBar = document.getElementById("healthBar");

  const stats = {
    Germany: { color: 'gray', reload: 4000, hp: 50 },
    France: { color: 'blue', reload: 6000, hp: 50 },
    UK: { color: 'red', reload: 8000, hp: 100 }
  };

  // Tank class
  class Tank {
    constructor(x, y, color, reloadTime, hp, isPlayer = false, isEnemy = false) {
      this.x = x;
      this.y = y;
      this.angle = -Math.PI / 2;
      this.color = color;
      this.reloadTime = reloadTime;
      this.hp = hp;
      this.maxHp = hp;
      this.isPlayer = isPlayer;
      this.isEnemy = isEnemy;
      this.canFire = true;
      this.fireCooldown = 0;
      this.speed = 2;
      this.turnSpeed = 0.04;

      // For roaming AI
      this.roamTarget = this.getRandomPoint();
      this.roamWaitTime = 0;
    }

    draw() {
      ctx.save();
      ctx.translate(this.x, this.y);
      ctx.rotate(this.angle);

      // Draw tank body
      ctx.fillStyle = this.color;
      ctx.fillRect(-15, -10, 30, 20);

      // Draw barrel facing forward
      ctx.fillStyle = "black";
      ctx.fillRect(0, -3, 20, 6);

      ctx.restore();
    }

    moveForward() {
      this.x += Math.cos(this.angle) * this.speed;
      this.y += Math.sin(this.angle) * this.speed;
      this.keepInBounds();
    }

    moveBackward() {
      this.x -= Math.cos(this.angle) * this.speed;
      this.y -= Math.sin(this.angle) * this.speed;
      this.keepInBounds();
    }

    turnLeft() {
      this.angle -= this.turnSpeed;
    }

    turnRight() {
      this.angle += this.turnSpeed;
    }

    keepInBounds() {
      if(this.x < 15) this.x = 15;
      if(this.x > canvas.width - 15) this.x = canvas.width - 15;
      if(this.y < 15) this.y = 15;
      if(this.y > canvas.height - 15) this.y = canvas.height - 15;
    }

    canShoot() {
      return this.canFire;
    }

    fire() {
      if (!this.canFire) return null;
      this.canFire = false;
      this.fireCooldown = this.reloadTime;
      // Projectile spawns at turret tip
      let tipX = this.x + Math.cos(this.angle) * 20;
      let tipY = this.y + Math.sin(this.angle) * 20;
      return new Projectile(tipX, tipY, this.angle, this);
    }

    updateCooldown(delta) {
      if (this.fireCooldown > 0) {
        this.fireCooldown -= delta;
      } else {
        this.canFire = true;
        this.fireCooldown = 0;
      }
    }

    takeDamage(dmg) {
      this.hp -= dmg;
    }

    isDead() {
      return this.hp <= 0;
    }

    // AI roaming behavior
    getRandomPoint() {
      return {
        x: 20 + Math.random() * (canvas.width - 40),
        y: 20 + Math.random() * (canvas.height - 40)
      };
    }

    roam(delta) {
      if(this.roamWaitTime > 0){
        this.roamWaitTime -= delta;
        return; // waiting at point
      }
      let dx = this.roamTarget.x - this.x;
      let dy = this.roamTarget.y - this.y;
      let dist = Math.sqrt(dx*dx + dy*dy);
      if(dist < 5) {
        // Reached target - wait a bit then pick new target
        this.roamWaitTime = 1000 + Math.random() * 2000;
        this.roamTarget = this.getRandomPoint();
      } else {
        let targetAngle = Math.atan2(dy, dx);
        let angleDiff = this.normalizeAngle(targetAngle - this.angle);

        // Turn towards target smoothly
        if(Math.abs(angleDiff) > this.turnSpeed){
          if(angleDiff > 0) this.angle += this.turnSpeed;
          else this.angle -= this.turnSpeed;
        } else {
          this.angle = targetAngle;
        }
        this.moveForward();
      }
    }

    normalizeAngle(angle) {
      while (angle > Math.PI) angle -= 2 * Math.PI;
      while (angle < -Math.PI) angle += 2 * Math.PI;
      return angle;
    }
  }

  // Projectile class
  class Projectile {
    constructor(x, y, angle, owner) {
      this.x = x;
      this.y = y;
      this.angle = angle;
      this.speed = 10;
      this.size = 5;
      this.owner = owner; // Tank instance who fired it
      this.damage = 20;
    }

    update() {
      this.x += Math.cos(this.angle) * this.speed;
      this.y += Math.sin(this.angle) * this.speed;
    }

    draw() {
      ctx.fillStyle = 'orange';
      ctx.beginPath();
      ctx.arc(this.x, this.y, this.size, 0, 2 * Math.PI);
      ctx.fill();
    }

    outOfBounds() {
      return this.x < 0 || this.x > canvas.width || this.y < 0 || this.y > canvas.height;
    }
  }

  // Globals
  let playerTank;
  let teamTanks = [];
  let enemyTanks = [];
  let projectiles = [];
  let lastTime = 0;

  // Controls state
  let movingForward = false;
  let movingBackward = false;
  let turningLeft = false;
  let turningRight = false;

  // Setup game
  function startGame() {
    const selectedCountry = document.getElementById("countrySelect").value;
    const s = stats[selectedCountry];

    // Create player tank
    playerTank = new Tank(canvas.width/2, canvas.height/2, s.color, s.reload, s.hp, true, false);

    // Create 4 team AI tanks around player
    const teamOffsets = [
      {x: -40, y: 40},
      {x: 40, y: 40},
      {x: -80, y: 80},
      {x: 80, y: 80}
    ];
    teamTanks = [];
    teamOffsets.forEach(off => {
      let t = new Tank(playerTank.x + off.x, playerTank.y + off.y, s.color, s.reload, s.hp, false, false);
      teamTanks.push(t);
    });

    // Create 5 enemy tanks scattered
    enemyTanks = [];
    for(let i=0; i<5; i++){
      let et = new Tank(100 + i * 70, 350, 'black', 7000, 50, false, true);
      enemyTanks.push(et);
    }

    projectiles = [];
    lastTime = performance.now();

    document.getElementById("countryControls").style.display = "none";
    document.getElementById("gameContainer").style.display = "block";

    updateHealthBar();
    reloadBar.style.width = '0%';

    requestAnimationFrame(gameLoop);
  }

  // Draw all
  function drawAll() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw all tanks
    playerTank.draw();
    teamTanks.forEach(t => t.draw());
    enemyTanks.forEach(t => t.draw());

    // Draw projectiles
    projectiles.forEach(p => p.draw());
  }

  // Player control handlers
  function updatePlayer(delta) {
    if(movingForward) playerTank.moveForward();
    if(movingBackward) playerTank.moveBackward();
    if(turningLeft) playerTank.turnLeft();
    if(turningRight) playerTank.turnRight();
    playerTank.updateCooldown(delta);
  }

  // AI: roaming and shooting logic
  function updateAI(tanks, enemies, delta) {
    tanks.forEach(tank => {
      if(tank.isDead()) return;

      // Roam around
      tank.roam(delta);

      tank.updateCooldown(delta);

      // Try to find closest enemy to shoot
      let nearestEnemy = null;
      let nearestDist = Infinity;
      enemies.forEach(e => {
        if(e.isDead()) return;
        let dx = e.x - tank.x;
        let dy = e.y - tank.y;
        let dist = Math.sqrt(dx*dx + dy*dy);
        if(dist < 220 && dist < nearestDist) {
          nearestDist = dist;
          nearestEnemy = e;
        }
      });

      if(nearestEnemy) {
        // Aim turret toward enemy
        let angleToEnemy = Math.atan2(nearestEnemy.y - tank.y, nearestEnemy.x - tank.x);
        let angleDiff = tank.normalizeAngle(angleToEnemy - tank.angle);
        if(Math.abs(angleDiff) > tank.turnSpeed){
          if(angleDiff > 0) tank.angle += tank.turnSpeed;
          else tank.angle -= tank.turnSpeed;
        } else {
          tank.angle = angleToEnemy;
        }

        // Shoot if possible
        if(tank.canShoot()) {
          let proj = tank.fire();
          if(proj) projectiles.push(proj);
        }
      }
    });
  }

  // Update projectiles positions and check collisions
  function updateProjectiles() {
    for(let i = projectiles.length - 1; i >= 0; i--) {
      let p = projectiles[i];
      p.update();

      // Remove if out of bounds
      if(p.outOfBounds()){
        projectiles.splice(i,1);
        continue;
      }

      // Check collision with tanks except owner
      let targets = [];
      if(p.owner.isEnemy) {
        // Enemy fired => hit player and team
        targets.push(playerTank, ...teamTanks);
      } else {
        // Player or ally fired => hit enemies
        targets = enemyTanks;
      }

      for(let j = targets.length - 1; j >= 0; j--) {
        let t = targets[j];
        if(t.isDead()) continue;
        // Simple collision: circle vs rect approx
        let distX = Math.abs(p.x - t.x);
        let distY = Math.abs(p.y - t.y);
        if(distX < 20 && distY < 15){
          // Hit
          t.takeDamage(p.damage);
          projectiles.splice(i,1);
          // Remove tank if dead
          if(t.isDead()) {
            if(t.isPlayer) {
              alert("You died! Reload the page to play again.");
              // Optionally, stop the game loop or reset
            } else {
              if(t.isEnemy){
                enemyTanks.splice(j,1);
              } else {
                teamTanks.splice(j,1);
              }
            }
          }
          break;
        }
      }
    }
  }

  // Update health bar for player
  function updateHealthBar() {
    const percent = (playerTank.hp / playerTank.maxHp) * 100;
    healthBar.style.width = percent + "%";
  }

  // Controls event listeners (touch + mouse)
  const btnForward = document.getElementById('btnForward');
  btnForward.addEventListener('touchstart', e => { e.preventDefault(); movingForward = true; });
  btnForward.addEventListener('touchend', e => { e.preventDefault(); movingForward = false; });
  btnForward.addEventListener('mousedown', e => { e.preventDefault(); movingForward = true; });
  btnForward.addEventListener('mouseup', e => { e.preventDefault(); movingForward = false; });

  const btnBackward = document.getElementById('btnBackward');
  btnBackward.addEventListener('touchstart', e => { e.preventDefault(); movingBackward = true; });
  btnBackward.addEventListener('touchend', e => { e.preventDefault(); movingBackward = false; });
  btnBackward.addEventListener('mousedown', e => { e.preventDefault(); movingBackward = true; });
  btnBackward.addEventListener('mouseup', e => { e.preventDefault(); movingBackward = false; });

  const btnLeft = document.getElementById('btnLeft');
  btnLeft.addEventListener('touchstart', e => { e.preventDefault(); turningLeft = true; });
  btnLeft.addEventListener('touchend', e => { e.preventDefault(); turningLeft = false; });
  btnLeft.addEventListener('mousedown', e => { e.preventDefault(); turningLeft = true; });
  btnLeft.addEventListener('mouseup', e => { e.preventDefault(); turningLeft = false; });

  const btnRight = document.getElementById('btnRight');
  btnRight.addEventListener('touchstart', e => { e.preventDefault(); turningRight = true; });
  btnRight.addEventListener('touchend', e => { e.preventDefault(); turningRight = false; });
  btnRight.addEventListener('mousedown', e => { e.preventDefault(); turningRight = true; });
  btnRight.addEventListener('mouseup', e => { e.preventDefault(); turningRight = false; });

  // Keyboard controls (wasd)
  window.addEventListener('keydown', e => {
    if(e.repeat) return;
    switch(e.key.toLowerCase()){
      case 'w': movingForward = true; break;
      case 's': movingBackward = true; break;
      case 'a': turningLeft = true; break;
      case 'd': turningRight = true; break;
      case ' ': fire(); break;
    }
  });
  window.addEventListener('keyup', e => {
    switch(e.key.toLowerCase()){
      case 'w': movingForward = false; break;
      case 's': movingBackward = false; break;
      case 'a': turningLeft = false; break;
      case 'd': turningRight = false; break;
    }
  });

  // Fire button
  function fire() {
    if(playerTank.canShoot()) {
      let proj = playerTank.fire();
      if(proj) projectiles.push(proj);

      // Reload bar animation
      reloadBar.style.width = '0%';
      let startTime = performance.now();
      let anim = () => {
        let elapsed = performance.now() - startTime;
        let pct = Math.min(elapsed / playerTank.reloadTime, 1) * 100;
        reloadBar.style.width = pct + '%';
        if(pct < 100) {
          requestAnimationFrame(anim);
        }
      };
      anim();
    }
  }

  // Main game loop
  function gameLoop(time=0) {
    let delta = time - lastTime;
    lastTime = time;

    updatePlayer(delta);
    updateAI(teamTanks, enemyTanks.concat(playerTank), delta);
    updateAI(enemyTanks, teamTanks.concat(playerTank), delta);
    updateProjectiles();
    drawAll();
    updateHealthBar();

    requestAnimationFrame(gameLoop);
  }
</script>
</body>
</html>
