<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Run Kushal Run 3D - Ahmedabad City</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Bangers&family=Outfit:wght@400;700&display=swap');

        body {
            margin: 0;
            overflow: hidden;
            background-color: #0c1445; 
            font-family: 'Outfit', sans-serif;
            touch-action: none;
            user-select: none;
        }

        #game-container {
            position: absolute;
            width: 100%;
            height: 100%;
        }

        #ui-layer {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            z-index: 5;
        }

        .hud {
            padding: 20px;
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            color: #fff;
            text-shadow: 0px 4px 8px rgba(0,0,0,0.8);
            font-family: 'Bangers', cursive;
        }

        .hud-text {
            font-size: clamp(20px, 5vw, 32px);
            margin-bottom: 5px;
        }

        .powerup-container {
            display: flex;
            flex-direction: column;
            gap: 8px;
            margin-top: 10px;
        }

        .powerup-status {
            background: rgba(0, 0, 0, 0.7);
            padding: 5px 12px;
            border-radius: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 14px;
            border: 2px solid #fff;
            min-width: 130px;
        }

        .progress-bar {
            height: 6px;
            background: #fff;
            border-radius: 3px;
            flex-grow: 1;
            overflow: hidden;
        }

        .bar-fill {
            height: 100%;
            width: 100%;
            transition: width 0.1s linear;
        }

        .hidden { display: none !important; }

        .screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle, rgba(0, 0, 0, 0.5) 0%, rgba(0, 0, 0, 0.9) 100%);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            pointer-events: auto;
            z-index: 20;
            backdrop-filter: blur(6px);
        }

        h1 {
            font-family: 'Bangers', cursive;
            font-size: clamp(40px, 12vw, 90px);
            color: #ffcc00;
            text-shadow: 4px 4px 0px #d35400;
            margin: 0;
            text-align: center;
        }

        .btn {
            background: linear-gradient(to bottom, #ffcc00, #ff9933);
            border: none;
            padding: 15px 50px;
            color: #fff;
            font-family: 'Bangers', cursive;
            font-size: 32px;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 8px 0 #d35400;
            transition: transform 0.1s;
            margin-top: 20px;
        }

        .btn:active {
            transform: translateY(4px);
            box-shadow: 0 4px 0 #d35400;
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="game-container"></div>

    <div id="ui-layer">
        <div class="hud">
            <div class="hud-left">
                <div id="score-display" class="hud-text">0m</div>
                <div id="speed-display" class="hud-text" style="color: #ff4757; font-size: 18px;">SPEED: 1.0x</div>
                <div id="diamond-display" class="hud-text" style="color: #00f2ff;">💎 0</div>
                
                <div class="powerup-container">
                    <div id="hoverboard-ui" class="powerup-status hidden" style="border-color: #f1c40f;">
                        <span style="color: #f1c40f;">🛹 HOVER</span>
                        <div class="progress-bar">
                            <div id="hoverboard-bar-fill" class="bar-fill" style="background: #f1c40f;"></div>
                        </div>
                    </div>
                    <div id="magnet-ui" class="powerup-status hidden" style="border-color: #ff4757;">
                        <span style="color: #ff4757;">🧲 MAGNET</span>
                        <div class="progress-bar">
                            <div id="magnet-bar-fill" class="bar-fill" style="background: #ff4757;"></div>
                        </div>
                    </div>
                </div>
            </div>
            <div id="high-score-display" class="hud-text" style="color: #ffcc00;">BEST: 0m</div>
        </div>
    </div>

    <div id="start-screen" class="screen">
        <h1>AHMEDABAD NIGHT RUN</h1>
        <p style="color: white; font-weight: 700; margin-bottom: 10px;">🚨 POLICE PEECHE HAI! 🚨</p>
        <p style="color: #ffcc00; font-size: 14px; margin-bottom: 20px;">Dodge karein: Swipe ya Arrow Keys</p>
        <button class="btn" onclick="startGame()">START</button>
    </div>

    <div id="game-over-screen" class="screen hidden">
        <h1 id="game-over-title" style="color: #ff4757;">PAKDE GAYE!</h1>
        <p id="final-stats" style="color: white; font-size: 24px; font-weight: 700; margin: 20px 0;"></p>
        <button class="btn" onclick="resetGame()">TRY AGAIN</button>
    </div>

    <script>
        const CONFIG = {
            laneWidth: 4.8,
            baseSpeed: 1.1,
            maxSpeed: 8.0,
            acceleration: 0.0008, 
            jumpForce: 0.85,
            gravity: 0.05,
            powerupDuration: 600, 
            magnetRadius: 18,
            chaserDistance: 15, 
            colors: {
                skin: 0xffdbac,
                shirt: 0x3498db,
                pant: 0x2c3e50,
                sky: 0x060b21,
                road: 0x1a1a1a,
                hoverboard: 0xf1c40f,
                magnet: 0xff4757,
                police: 0x2980b9,
                dog: 0x8d6e63,
                buildingColors: [0x34495e, 0x2c3e50, 0x7f8c8d, 0x95a5a6, 0x1e272e],
                houseColors: [0xe67e22, 0xd35400, 0xecf0f1, 0xbdc3c7, 0xf39c12]
            }
        };

        let scene, camera, renderer;
        let playerGroup, player, hoverboardMesh, magnetMesh;
        let chaserGroup, policeCar, policeLights, streetDog;
        let environmentGroup;
        let groundSegments = [];
        let obstacles = [];
        let diamonds = [];
        let powerups = [];
        let enemyCars = [];
        
        let gameState = {
            isPlaying: false,
            speed: CONFIG.baseSpeed,
            score: 0,
            diamonds: 0,
            lane: 1, 
            isJumping: false,
            jumpVel: 0,
            isSliding: false,
            slideTimer: 0,
            zPos: 0,
            hoverboardTimer: 0,
            hasHoverboard: false,
            magnetTimer: 0,
            hasMagnet: false,
            chaserDist: CONFIG.chaserDistance
        };

        // --- MODELS ---

        function createPlayerModel() {
            const group = new THREE.Group();
            const head = new THREE.Mesh(new THREE.SphereGeometry(0.4, 16, 16), new THREE.MeshLambertMaterial({color: CONFIG.colors.skin}));
            head.position.y = 2.8;
            group.add(head);
            const shirt = new THREE.Mesh(new THREE.BoxGeometry(0.8, 1, 0.5), new THREE.MeshLambertMaterial({color: CONFIG.colors.shirt}));
            shirt.position.y = 2.0;
            group.add(shirt);
            const pants = new THREE.Mesh(new THREE.BoxGeometry(0.75, 1, 0.45), new THREE.MeshLambertMaterial({color: CONFIG.colors.pant}));
            pants.position.y = 1.0;
            group.add(pants);
            return group;
        }

        function createPoliceCar() {
            const group = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(2.8, 1.4, 5), new THREE.MeshLambertMaterial({color: 0xffffff}));
            body.position.y = 0.8;
            group.add(body);
            const bottom = new THREE.Mesh(new THREE.BoxGeometry(2.8, 0.4, 5), new THREE.MeshLambertMaterial({color: CONFIG.colors.police}));
            bottom.position.y = 0.3;
            group.add(bottom);
            const lightBar = new THREE.Group();
            const barGeom = new THREE.BoxGeometry(1.5, 0.3, 0.4);
            const blueLight = new THREE.Mesh(barGeom, new THREE.MeshBasicMaterial({color: 0x0000ff}));
            blueLight.position.x = -0.4;
            const redLight = new THREE.Mesh(barGeom, new THREE.MeshBasicMaterial({color: 0xff0000}));
            redLight.position.x = 0.4;
            lightBar.add(blueLight, redLight);
            lightBar.position.y = 1.6;
            group.add(lightBar);
            policeLights = lightBar; 
            return group;
        }

        function createDog() {
            const group = new THREE.Group();
            const body = new THREE.Mesh(new THREE.BoxGeometry(0.5, 0.6, 1.2), new THREE.MeshLambertMaterial({color: CONFIG.colors.dog}));
            body.position.y = 0.5;
            group.add(body);
            const head = new THREE.Mesh(new THREE.BoxGeometry(0.4, 0.4, 0.5), new THREE.MeshLambertMaterial({color: CONFIG.colors.dog}));
            head.position.set(0, 0.9, 0.5);
            group.add(head);
            return group;
        }

        function createStreetLight() {
            const group = new THREE.Group();
            const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.15, 0.2, 8), new THREE.MeshLambertMaterial({color: 0x555555}));
            pole.position.y = 4;
            group.add(pole);
            const bulb = new THREE.Mesh(new THREE.SphereGeometry(0.4, 16, 16), new THREE.MeshBasicMaterial({color: 0xffaa00}));
            bulb.position.set(1.8, 7.8, 0);
            group.add(bulb);
            const light = new THREE.PointLight(0xffaa00, 1.2, 25);
            light.position.set(1.8, 7.5, 0);
            group.add(light);
            return group;
        }

        // --- NEW: HOUSE AND BUILDING FACTORY ---

        function createResidentialObject(side) {
            const chance = Math.random();
            const group = new THREE.Group();
            const offset = side === 'left' ? -22 : 22;

            if (chance < 0.4) {
                // CHHOTA GHAR (POL Style)
                const color = CONFIG.colors.houseColors[Math.floor(Math.random()*CONFIG.colors.houseColors.length)];
                const body = new THREE.Mesh(new THREE.BoxGeometry(10, 6, 8), new THREE.MeshLambertMaterial({color: color}));
                body.position.y = 3;
                group.add(body);
                // Roof
                const roof = new THREE.Mesh(new THREE.BoxGeometry(11, 1, 9), new THREE.MeshLambertMaterial({color: 0x8e44ad}));
                roof.position.y = 6.5;
                group.add(roof);
                // Door
                const door = new THREE.Mesh(new THREE.PlaneGeometry(2, 3), new THREE.MeshBasicMaterial({color: 0x34495e}));
                door.position.set(side === 'left' ? 5.01 : -5.01, 1.5, 0);
                door.rotation.y = side === 'left' ? Math.PI/2 : -Math.PI/2;
                group.add(door);
            } else if (chance < 0.7) {
                // BADA BUNGALOW
                const body = new THREE.Mesh(new THREE.BoxGeometry(12, 12, 12), new THREE.MeshLambertMaterial({color: 0xfafafa}));
                body.position.y = 6;
                group.add(body);
                const terrace = new THREE.Mesh(new THREE.BoxGeometry(12, 1, 12), new THREE.MeshLambertMaterial({color: 0xbdc3c7}));
                terrace.position.y = 12.5;
                group.add(terrace);
            } else {
                // BUILDING (Skyscraper)
                const color = CONFIG.colors.buildingColors[Math.floor(Math.random()*CONFIG.colors.buildingColors.length)];
                const height = 30 + Math.random() * 50;
                const body = new THREE.Mesh(new THREE.BoxGeometry(15, height, 15), new THREE.MeshLambertMaterial({color: color}));
                body.position.y = height / 2;
                group.add(body);
            }

            group.position.x = offset;
            return group;
        }

        // --- CORE GAME ---

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(CONFIG.colors.sky);
            scene.fog = new THREE.Fog(CONFIG.colors.sky, 60, 220);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 8, 15);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(window.devicePixelRatio);
            document.getElementById('game-container').appendChild(renderer.domElement);

            scene.add(new THREE.AmbientLight(0xffffff, 0.4));
            const sun = new THREE.DirectionalLight(0xffffff, 0.6);
            sun.position.set(10, 50, 20);
            scene.add(sun);

            playerGroup = new THREE.Group();
            scene.add(playerGroup);
            player = createPlayerModel();
            playerGroup.add(player);
            
            hoverboardMesh = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.3, 3), new THREE.MeshLambertMaterial({color: CONFIG.colors.hoverboard}));
            hoverboardMesh.position.y = 0.4;
            hoverboardMesh.visible = false;
            player.add(hoverboardMesh);

            magnetMesh = new THREE.Mesh(new THREE.RingGeometry(1.4, 1.8, 32), new THREE.MeshBasicMaterial({color: CONFIG.colors.magnet, transparent: true, opacity: 0.6}));
            magnetMesh.rotation.x = Math.PI/2;
            magnetMesh.visible = false;
            player.add(magnetMesh);

            chaserGroup = new THREE.Group();
            scene.add(chaserGroup);
            policeCar = createPoliceCar();
            chaserGroup.add(policeCar);
            streetDog = createDog();
            streetDog.position.x = 2.5;
            chaserGroup.add(streetDog);

            environmentGroup = new THREE.Group();
            scene.add(environmentGroup);

            for (let i = 0; i < 15; i++) spawnSegment(-i * 20);

            window.addEventListener('resize', onResize);
            document.addEventListener('keydown', onKeyDown);
            setupTouch();
            animate();
        }

        function spawnSegment(z) {
            const seg = new THREE.Group();
            seg.position.z = z;
            
            const road = new THREE.Mesh(new THREE.PlaneGeometry(16, 20), new THREE.MeshLambertMaterial({color: CONFIG.colors.road}));
            road.rotation.x = -Math.PI/2;
            seg.add(road);

            if (Math.abs(z) % 40 === 0) {
                const lightL = createStreetLight();
                lightL.position.set(-8.5, 0, 0);
                seg.add(lightL);
                const lightR = createStreetLight();
                lightR.position.set(8.5, 0, 0);
                lightR.rotation.y = Math.PI;
                seg.add(lightR);
            }

            // Spawn random houses or buildings
            seg.add(createResidentialObject('left'));
            seg.add(createResidentialObject('right'));

            environmentGroup.add(seg);
            groundSegments.push(seg);

            if (gameState.isPlaying && z < -60) {
                const laneX = (Math.floor(Math.random()*3)-1) * CONFIG.laneWidth;
                
                if(Math.random() > 0.88) {
                    const box = new THREE.Mesh(new THREE.BoxGeometry(3, 2, 2), new THREE.MeshLambertMaterial({color: 0x555555}));
                    box.position.set(laneX, 1, z);
                    scene.add(box);
                    obstacles.push(box);
                }
                
                if(Math.random() > 0.95) {
                    const car = new THREE.Mesh(new THREE.BoxGeometry(3, 1.5, 5), new THREE.MeshLambertMaterial({color: 0x9b59b6}));
                    car.position.set(laneX, 0.75, z - 100);
                    scene.add(car);
                    enemyCars.push(car);
                }

                if(Math.random() > 0.6) {
                    const d = new THREE.Mesh(new THREE.OctahedronGeometry(0.5), new THREE.MeshPhongMaterial({color: 0x00f2ff, emissive: 0x00f2ff}));
                    d.position.set(laneX, 1.5, z - 5);
                    scene.add(d);
                    diamonds.push(d);
                }

                if(Math.random() > 0.98) {
                    const pMesh = new THREE.Mesh(new THREE.SphereGeometry(0.6), new THREE.MeshLambertMaterial({color: Math.random() > 0.5 ? CONFIG.colors.magnet : CONFIG.colors.hoverboard}));
                    pMesh.position.set(laneX, 1, z - 10);
                    pMesh.userData = { type: pMesh.material.color.getHex() === CONFIG.colors.magnet ? 'magnet' : 'hoverboard' };
                    scene.add(pMesh);
                    powerups.push(pMesh);
                }
            }
        }

        function setupTouch() {
            let sX, sY;
            document.addEventListener('touchstart', e => { sX = e.touches[0].clientX; sY = e.touches[0].clientY; }, {passive: false});
            document.addEventListener('touchend', e => {
                const dX = e.changedTouches[0].clientX - sX;
                const dY = e.changedTouches[0].clientY - sY;
                if(Math.abs(dX) > Math.abs(dY)) {
                    if(dX > 30) moveLane(1); else if(dX < -30) moveLane(-1);
                } else {
                    if(dY < -30) jump(); else if(dY > 30) slide();
                }
            }, {passive: false});
        }

        function moveLane(dir) { gameState.lane = Math.max(0, Math.min(2, gameState.lane + dir)); }
        function jump() { if(!gameState.isJumping) { gameState.isJumping = true; gameState.jumpVel = CONFIG.jumpForce; } }
        function slide() { if(!gameState.isSliding && !gameState.isJumping) { gameState.isSliding = true; gameState.slideTimer = 40; } }

        function onKeyDown(e) {
            if(!gameState.isPlaying) return;
            if(e.key === 'ArrowLeft') moveLane(-1);
            if(e.key === 'ArrowRight') moveLane(1);
            if(e.key === 'ArrowUp') jump();
            if(e.key === 'ArrowDown') slide();
        }

        function animate() {
            requestAnimationFrame(animate);
            if(!gameState.isPlaying) { renderer.render(scene, camera); return; }

            gameState.zPos -= gameState.speed;
            gameState.score += gameState.speed * 0.1;
            if(gameState.speed < CONFIG.maxSpeed) gameState.speed += CONFIG.acceleration;
            
            gameState.chaserDist = Math.max(3, gameState.chaserDist - 0.005); 
            chaserGroup.position.z = gameState.zPos + gameState.chaserDist;
            chaserGroup.position.x += (playerGroup.position.x - chaserGroup.position.x) * 0.06;
            
            if(Math.floor(Date.now()/100) % 2 === 0) {
                policeLights.children[0].visible = true; policeLights.children[1].visible = false;
            } else {
                policeLights.children[0].visible = false; policeLights.children[1].visible = true;
            }

            streetDog.position.y = 0.2 + Math.abs(Math.sin(Date.now()*0.01))*0.3;

            if(gameState.chaserDist < 3.8) gameOver("POLICE NE PAKAD LIYA!");

            if(gameState.hoverboardTimer > 0) {
                gameState.hoverboardTimer--;
                document.getElementById('hoverboard-ui').classList.remove('hidden');
                document.getElementById('hoverboard-bar-fill').style.width = (gameState.hoverboardTimer / CONFIG.powerupDuration * 100) + "%";
                hoverboardMesh.visible = true;
                if(gameState.hoverboardTimer <= 0) { gameState.hasHoverboard = false; hoverboardMesh.visible = false; document.getElementById('hoverboard-ui').classList.add('hidden'); }
            }
            if(gameState.magnetTimer > 0) {
                gameState.magnetTimer--;
                document.getElementById('magnet-ui').classList.remove('hidden');
                document.getElementById('magnet-bar-fill').style.width = (gameState.magnetTimer / CONFIG.powerupDuration * 100) + "%";
                magnetMesh.visible = true;
                if(gameState.magnetTimer <= 0) { gameState.hasMagnet = false; magnetMesh.visible = false; document.getElementById('magnet-ui').classList.add('hidden'); }
            }

            playerGroup.position.z = gameState.zPos;
            const targetX = (gameState.lane - 1) * CONFIG.laneWidth;
            playerGroup.position.x += (targetX - playerGroup.position.x) * 0.2;

            if(gameState.isJumping) {
                player.position.y += gameState.jumpVel;
                gameState.jumpVel -= CONFIG.gravity;
                if(player.position.y <= 0) { player.position.y = 0; gameState.isJumping = false; }
            } else if(gameState.isSliding) {
                gameState.slideTimer--;
                player.scale.y = 0.5;
                if(gameState.slideTimer <= 0) { player.scale.y = 1; gameState.isSliding = false; }
            }

            camera.position.z = playerGroup.position.z + 14;
            camera.lookAt(playerGroup.position.x * 0.5, 3, playerGroup.position.z - 20);

            const pBox = new THREE.Box3().setFromObject(player).expandByScalar(-0.1);
            
            enemyCars.forEach((car, i) => {
                car.position.z += gameState.speed * 0.7;
                if(pBox.intersectsBox(new THREE.Box3().setFromObject(car))) gameOver("ACCIDENT HO GAYA!");
            });

            diamonds.forEach((d, i) => {
                if(gameState.hasMagnet && d.position.distanceTo(playerGroup.position) < CONFIG.magnetRadius) d.position.lerp(playerGroup.position, 0.2);
                if(pBox.containsPoint(d.position)) {
                    gameState.diamonds++;
                    gameState.chaserDist = Math.min(20, gameState.chaserDist + 0.2);
                    scene.remove(d);
                    diamonds.splice(i, 1);
                }
                d.rotation.y += 0.05;
            });

            powerups.forEach((p, i) => {
                if(pBox.intersectsBox(new THREE.Box3().setFromObject(p))) {
                    if(p.userData.type === 'magnet') { gameState.hasMagnet = true; gameState.magnetTimer = CONFIG.powerupDuration; }
                    else { gameState.hasHoverboard = true; gameState.hoverboardTimer = CONFIG.powerupDuration; gameState.chaserDist += 5; }
                    scene.remove(p);
                    powerups.splice(i, 1);
                }
            });

            obstacles.forEach((o, i) => {
                if(pBox.intersectsBox(new THREE.Box3().setFromObject(o))) {
                    if(gameState.hasHoverboard) {
                        gameState.hasHoverboard = false;
                        gameState.hoverboardTimer = 0;
                        gameState.chaserDist -= 4;
                        scene.remove(o);
                        obstacles.splice(i, 1);
                    } else gameOver("TAKRA GAYE!");
                }
            });

            const lastZ = groundSegments[groundSegments.length-1].position.z;
            if(lastZ > playerGroup.position.z - 300) spawnSegment(lastZ - 20);
            if(groundSegments[0].position.z > playerGroup.position.z + 40) environmentGroup.remove(groundSegments.shift());

            document.getElementById('score-display').innerText = Math.floor(gameState.score) + "m";
            document.getElementById('diamond-display').innerText = "💎 " + gameState.diamonds;
            document.getElementById('speed-display').innerText = `SPEED: ${gameState.speed.toFixed(1)}x`;

            renderer.render(scene, camera);
        }

        function startGame() { 
            document.getElementById('start-screen').classList.add('hidden'); 
            gameState.isPlaying = true; 
        }

        function resetGame() { location.reload(); }

        function gameOver(reason) { 
            gameState.isPlaying = false; 
            document.getElementById('game-over-title').innerText = reason;
            document.getElementById('game-over-screen').classList.remove('hidden'); 
            document.getElementById('final-stats').innerText = `Score: ${Math.floor(gameState.score)}m | Diamonds: ${gameState.diamonds}`; 
        }

        function onResize() { 
            camera.aspect = window.innerWidth / window.innerHeight; 
            camera.updateProjectionMatrix(); 
            renderer.setSize(window.innerWidth, window.innerHeight); 
        }

        init();
    </script>
</body>
</html>
