<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>꿀벌 지키기 3D - Beekeeper vs Pests</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Bangers&family=Noto+Sans+KR:wght@400;700;900&display=swap" rel="stylesheet">
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Three.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            overflow: hidden;
            background-color: #1a1a1a;
        }

        .comic-font {
            font-family: 'Bangers', 'Noto Sans KR', cursive;
            letter-spacing: 1px;
        }

        /* Comic Book UI Elements inspired by The Mosquito Gang */
        .comic-box {
            background: #fffb00;
            border: 4px solid #000;
            box-shadow: 6px 6px 0px #000;
            transform: rotate(-1deg);
        }

        .comic-box-red {
            background: #ff3366;
            border: 4px solid #000;
            box-shadow: 6px 6px 0px #000;
            color: white;
        }

        .comic-box-blue {
            background: #00d2ff;
            border: 4px solid #000;
            box-shadow: 6px 6px 0px #000;
            color: black;
        }

        .comic-button {
            background: #ff9900;
            border: 4px solid #000;
            box-shadow: 5px 5px 0px #000;
            transition: all 0.1s ease;
            font-family: 'Bangers', 'Noto Sans KR', cursive;
        }

        .comic-button:hover {
            transform: translate(-2px, -2px);
            box-shadow: 7px 7px 0px #000;
            background: #ffaa00;
        }

        .comic-button:active {
            transform: translate(3px, 3px);
            box-shadow: 2px 2px 0px #000;
        }

        /* Comic Popup text animation */
        @keyframes popIn {
            0% { transform: translate(-50%, -50%) scale(0) rotate(-15deg); opacity: 0; }
            50% { transform: translate(-50%, -50%) scale(1.3) rotate(5deg); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(1) rotate(0deg); opacity: 0; }
        }

        .hit-pop {
            position: absolute;
            top: 50%;
            left: 50%;
            pointer-events: none;
            animation: popIn 0.5s ease-out forwards;
            font-family: 'Bangers', cursive;
            text-shadow: 4px 4px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000;
            z-index: 50;
        }

        /* Crosshair */
        #crosshair {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 20px;
            height: 20px;
            transform: translate(-50%, -50%);
            pointer-events: none;
            z-index: 30;
        }

        #crosshair::before, #crosshair::after {
            content: '';
            position: absolute;
            background: #ff0044;
            border: 1px solid #000;
        }

        #crosshair::before {
            top: 9px;
            left: 0;
            width: 20px;
            height: 3px;
        }

        #crosshair::after {
            top: 0;
            left: 9px;
            width: 3px;
            height: 20px;
        }

        /* Halftone overlay style */
        .halftone-bg {
            background-image: radial-gradient(#000 15%, transparent 16%);
            background-size: 12px 12px;
        }
    </style>
</head>
<body class="relative w-screen h-screen overflow-hidden">

    <!-- Canvas container -->
    <div id="canvas-container" class="w-full h-full absolute inset-0"></div>

    <!-- Crosshair -->
    <div id="crosshair" class="hidden"></div>

    <!-- Comic Hit Effects Container -->
    <div id="pop-container" class="absolute inset-0 pointer-events-none z-40 overflow-hidden"></div>

    <!-- HUD Overlay -->
    <div id="hud" class="hidden absolute inset-0 pointer-events-none p-6 flex flex-col justify-between z-20">
        <!-- Top Bar -->
        <div class="flex justify-between items-start w-full">
            <!-- Bee Protection Bar -->
            <div class="comic-box p-4 rounded-xl flex flex-col gap-1 w-72">
                <div class="flex justify-between items-center">
                    <span class="font-black text-xl text-black flex items-center gap-2">
                        <i class="fa-solid font-bold fa-bee text-amber-600"></i> 남은 꿀벌 수
                    </span>
                    <span id="bee-count-text" class="comic-font text-3xl font-bold text-black">100 / 100</span>
                </div>
                <div class="w-full bg-black h-6 rounded-full p-1 border-2 border-black">
                    <div id="bee-health-bar" class="bg-amber-400 h-full rounded-full transition-all duration-300" style="width: 100%;"></div>
                </div>
            </div>

            <!-- Game Title / Mosquito Gang Style Tag -->
            <div class="comic-box-red p-3 rounded-lg transform rotate-2">
                <h1 class="comic-font text-3xl font-extrabold tracking-wider text-yellow-300 drop-shadow-md">
                    BEEKEEPER VS PESTS!
                </h1>
            </div>

            <!-- Pests Counter -->
            <div class="comic-box-blue p-4 rounded-xl flex flex-col gap-1 w-64 text-right">
                <div class="flex justify-between items-center">
                    <span id="pest-count-text" class="comic-font text-3xl font-bold text-white">12 LEFT</span>
                    <span class="font-black text-xl text-white flex items-center gap-2">
                        퇴치할 천적 <i class="fa-solid fa-bug text-red-400"></i>
                    </span>
                </div>
                <div class="w-full bg-black h-6 rounded-full p-1 border-2 border-black">
                    <div id="pest-bar" class="bg-red-500 h-full rounded-full transition-all duration-300" style="width: 100%;"></div>
                </div>
            </div>
        </div>

        <!-- Bottom Bar: Current Weapon & Controls Tip -->
        <div class="flex justify-between items-end w-full">
            <!-- Controls Quick Info -->
            <div class="bg-black/80 text-white p-3 rounded-lg border-2 border-yellow-400 text-xs flex flex-col gap-1">
                <div><span class="text-yellow-400 font-bold">W,A,S,D:</span> 이동 | <span class="text-yellow-400 font-bold">SPACE:</span> 점프</div>
                <div><span class="text-yellow-400 font-bold">마우스 좌클릭:</span> 공격 | <span class="text-yellow-400 font-bold">Shift:</span> 무기 교체</div>
            </div>

            <!-- Selected Weapon HUD -->
            <div class="comic-box p-4 rounded-xl flex items-center gap-4 transform -rotate-1">
                <div id="weapon-icon" class="text-4xl text-black w-12 text-center">
                    <i class="fa-solid fa-hand-sparkles"></i>
                </div>
                <div>
                    <div class="text-xs font-bold text-gray-800 uppercase tracking-wider">현재 선택된 무기 (Shift로 변경)</div>
                    <div id="weapon-name" class="comic-font text-3xl font-extrabold text-red-600">파리채 (Flyswatter)</div>
                    <div id="weapon-desc" class="text-xs text-black font-medium">빠르고 표준적인 공격력</div>
                </div>
            </div>
        </div>
    </div>

    <!-- Start / Pause Overlay -->
    <div id="start-screen" class="absolute inset-0 bg-amber-400/90 halftone-bg flex items-center justify-center z-50 p-4">
        <div class="bg-white border-8 border-black rounded-3xl p-8 max-w-2xl w-full text-center shadow-[12px_12px_0px_0px_rgba(0,0,0,1)] transform rotate-1">
            
            <div class="inline-block comic-box-red px-6 py-2 rounded-full mb-4 transform -rotate-2">
                <span class="comic-font text-2xl text-yellow-300 tracking-wider">THE MOSQUITO GANG STYLE</span>
            </div>

            <h1 class="comic-font text-6xl text-amber-500 tracking-wider drop-shadow-[4px_4px_0px_#000] -rotate-1 mb-2">
                꿀벌 지키기 3D
            </h1>
            <p class="text-lg font-bold text-gray-800 mb-6">
                장수말벌, 두꺼비, 잠자리, 거미가 꿀벌들을 노리고 있습니다!<br>
                양봉업자가 되어 다양한 도구로 천적들을 모두 물리치세요!
            </p>

            <!-- Weapons Wheel Showcase -->
            <div class="grid grid-cols-4 gap-2 mb-6 bg-yellow-100 p-4 rounded-xl border-4 border-black text-xs font-bold">
                <div class="p-2 bg-white rounded border-2 border-black">👋 파리채</div>
                <div class="p-2 bg-white rounded border-2 border-black">🥣 양푼</div>
                <div class="p-2 bg-white rounded border-2 border-black">🕸️ 잠자리채</div>
                <div class="p-2 bg-white rounded border-2 border-black">👔 옷걸이</div>
                <div class="p-2 bg-white rounded border-2 border-black">🥢 집게</div>
                <div class="p-2 bg-white rounded border-2 border-black">🧤 목장갑</div>
                <div class="p-2 bg-white rounded border-2 border-black">💨 연기기</div>
                <div class="p-2 bg-amber-300 rounded border-2 border-black text-red-600">Shift로 교체</div>
            </div>

            <!-- Controls Guide -->
            <div class="bg-gray-100 p-4 rounded-xl border-4 border-black mb-6 text-left text-sm font-semibold text-gray-700">
                <p class="font-bold text-black text-center mb-2">🎮 조작법 안내</p>
                <ul class="grid grid-cols-2 gap-2">
                    <li>• <span class="bg-black text-white px-1.5 py-0.5 rounded">마우스 이동</span>: 시점 회전</li>
                    <li>• <span class="bg-black text-white px-1.5 py-0.5 rounded">좌클릭</span>: 공격 / 휘두르기</li>
                    <li>• <span class="bg-black text-white px-1.5 py-0.5 rounded">W, A, S, D</span>: 양봉업자 이동</li>
                    <li>• <span class="bg-black text-white px-1.5 py-0.5 rounded">Space</span>: 점프</li>
                    <li>• <span class="bg-black text-white px-1.5 py-0.5 rounded">Shift</span>: 무기 교체 (7종)</li>
                </ul>
            </div>

            <button id="start-btn" class="comic-button text-4xl text-white px-10 py-4 rounded-2xl w-full cursor-pointer uppercase tracking-wider">
                게임 시작하기 (PLAY NOW)
            </button>
        </div>
    </div>

    <!-- Game End Screen (Victory / Defeat) -->
    <div id="end-screen" class="hidden absolute inset-0 bg-black/80 flex items-center justify-center z-50 p-4">
        <div class="bg-white border-8 border-black rounded-3xl p-8 max-w-md w-full text-center shadow-[12px_12px_0px_0px_rgba(0,0,0,1)]">
            <h2 id="end-title" class="comic-font text-6xl tracking-wider mb-4 drop-shadow-[3px_3px_0px_#000]">
                VICTORY!
            </h2>
            <p id="end-desc" class="text-xl font-bold text-gray-800 mb-6">
                모든 천적을 퇴치하고 꿀벌을 지켜냈습니다!
            </p>
            <div class="bg-yellow-100 p-4 rounded-xl border-4 border-black mb-6 font-bold text-gray-800">
                <p id="end-score">남은 꿀벌: 100마리</p>
            </div>
            <button id="restart-btn" class="comic-button text-3xl text-white px-8 py-3 rounded-2xl w-full cursor-pointer">
                다시 하기 (RETRY)
            </button>
        </div>
    </div>

    <script>
        /* --- GAME CONFIG & WEAPONS DATA --- */
        const WEAPONS = [
            { id: 'swatter', name: '파리채', desc: '빠르고 표준적인 공격범위', icon: 'fa-hand-sparkles', range: 4.5, damage: 35, cooldown: 300, color: 0xff3366 },
            { id: 'basin', name: '양푼', desc: '강력하지만 둔탁한 묵직한 한 방', icon: 'fa-bowl-food', range: 3.5, damage: 70, cooldown: 650, color: 0xcccccc },
            { id: 'net', name: '잠자리채', desc: '사거리가 긴 넓은 휩쓸기', icon: 'fa-circle-nodes', range: 7.0, damage: 25, cooldown: 400, color: 0x33ccff },
            { id: 'hanger', name: '옷걸이', desc: '날카롭고 신속한 연타', icon: 'fa-shirt', range: 4.0, damage: 30, cooldown: 250, color: 0xff9900 },
            { id: 'tongs', name: '집게', desc: '조준이 필요하지만 높은 단일 데미지', icon: 'fa-align-center', range: 3.8, damage: 85, cooldown: 500, color: 0x888888 },
            { id: 'gloves', name: '목장갑', desc: '근접 육탄전 스팽킹', icon: 'fa-mitten', range: 3.0, damage: 45, cooldown: 200, color: 0xdddddd },
            { id: 'smoker', name: '연기기', desc: '넓은 범위 지속 연막 피해', icon: 'fa-smog', range: 6.0, damage: 20, cooldown: 150, color: 0x999999 }
        ];

        let currentWeaponIdx = 0;
        let isAttacking = false;
        let lastAttackTime = 0;

        // Game State Variables
        let gameActive = false;
        let beeCount = 100;
        const maxBees = 100;
        let pests = [];
        let bees = [];
        const TOTAL_PESTS = 14;

        /* --- THREE.JS GLOBAL SETUP --- */
        let scene, camera, renderer;
        let playerContainer, cameraPitchGroup;
        let weaponMeshGroup;

        // Physics & Movement Controls
        const keys = { w: false, a: false, s: false, d: false, space: false };
        let moveSpeed = 0.22;
        let velocityY = 0;
        const gravity = -0.015;
        let isGrounded = true;

        // Pointer Lock / Mouse Look Variables
        let yaw = 0;
        let pitch = 0;

        // Initialize Audio Context for Sound Effects
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        let audioCtx;

        function playSound(type) {
            if (!audioCtx) audioCtx = new AudioContext();
            if (audioCtx.state === 'suspended') audioCtx.resume();

            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            const now = audioCtx.currentTime;

            if (type === 'swing') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(400, now);
                osc.frequency.exponentialRampToValueAtTime(100, now + 0.15);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.15);
                osc.start(now);
                osc.stop(now + 0.15);
            } else if (type === 'hit') {
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(150, now);
                osc.frequency.setValueAtTime(300, now + 0.05);
                gain.gain.setValueAtTime(0.5, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.2);
                osc.start(now);
                osc.stop(now + 0.2);
            } else if (type === 'bee_eat') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(600, now);
                osc.frequency.linearRampToValueAtTime(200, now + 0.2);
                gain.gain.setValueAtTime(0.2, now);
                gain.gain.linearRampToValueAtTime(0.01, now + 0.2);
                osc.start(now);
                osc.stop(now + 0.2);
            }
        }

        function init3D() {
            const container = document.getElementById('canvas-container');

            // Scene setup
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb); // Sky blue
            scene.fog = new THREE.FogExp2(0x87ceeb, 0.015);

            // Camera setup
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

            // Renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.appendChild(renderer.domElement);

            // Player Setup (First-Person View Hierarchy)
            playerContainer = new THREE.Group();
            playerContainer.position.set(0, 2, 15);
            scene.add(playerContainer);

            cameraPitchGroup = new THREE.Group();
            playerContainer.add(cameraPitchGroup);
            cameraPitchGroup.add(camera);

            // Lights
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
            scene.add(ambientLight);

            const sunLight = new THREE.DirectionalLight(0xfffaed, 0.9);
            sunLight.position.set(30, 50, 20);
            sunLight.castShadow = true;
            sunLight.shadow.mapSize.width = 2048;
            sunLight.shadow.mapSize.height = 2048;
            sunLight.shadow.camera.near = 0.5;
            sunLight.shadow.camera.far = 150;
            const d = 40;
            sunLight.shadow.camera.left = -d;
            sunLight.shadow.camera.right = d;
            sunLight.shadow.camera.top = d;
            sunLight.shadow.camera.bottom = -d;
            scene.add(sunLight);

            // Create Environment (Garden, Hive, Flowers)
            buildGarden();

            // Create First-Person Weapon
            createWeaponMesh();

            window.addEventListener('resize', onWindowResize);
        }

        function buildGarden() {
            // Ground Mesh
            const groundGeo = new THREE.PlaneGeometry(120, 120);
            const groundMat = new THREE.MeshStandardMaterial({ color: 0x55aa33, roughness: 0.8 });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // Fences surrounding area
            const fenceMat = new THREE.MeshStandardMaterial({ color: 0x8b5a2b });
            for (let i = -50; i <= 50; i += 10) {
                // Outer Boundary Posts
                const postGeo = new THREE.BoxGeometry(0.8, 3, 0.8);
                const p1 = new THREE.Mesh(postGeo, fenceMat);
                p1.position.set(i, 1.5, -50);
                scene.add(p1);

                const p2 = new THREE.Mesh(postGeo, fenceMat);
                p2.position.set(i, 1.5, 50);
                scene.add(p2);
            }

            // Bee Hives (Center of Attention)
            createBeeHive(0, 0, -2);
            createBeeHive(-6, 0, -5);
            createBeeHive(6, 0, -5);

            // Decorative Flowers and Trees
            for (let i = 0; i < 40; i++) {
                createFlower(
                    (Math.random() - 0.5) * 80,
                    (Math.random() - 0.5) * 80
                );
            }

            for (let i = 0; i < 15; i++) {
                const rx = (Math.random() - 0.5) * 90;
                const rz = (Math.random() - 0.5) * 90;
                if (Math.abs(rx) > 10 || Math.abs(rz) > 10) {
                    createTree(rx, rz);
                }
            }

            // Spawn Decorative Flying Honey Bees
            spawnBees();
        }

        function createBeeHive(x, y, z) {
            const hiveGroup = new THREE.Group();
            hiveGroup.position.set(x, y, z);

            const hiveMat = new THREE.MeshStandardMaterial({ color: 0xe6a100, roughness: 0.4 });
            const box1 = new THREE.Mesh(new THREE.BoxGeometry(2.5, 1.2, 2.5), hiveMat);
            box1.position.y = 0.6;
            box1.castShadow = true;
            hiveGroup.add(box1);

            const box2 = new THREE.Mesh(new THREE.BoxGeometry(2.2, 1.0, 2.2), hiveMat);
            box2.position.y = 1.7;
            box2.castShadow = true;
            hiveGroup.add(box2);

            const box3 = new THREE.Mesh(new THREE.BoxGeometry(1.8, 0.8, 1.8), hiveMat);
            box3.position.y = 2.5;
            box3.castShadow = true;
            hiveGroup.add(box3);

            // Roof
            const roofGeo = new THREE.ConeGeometry(2.2, 1.0, 4);
            const roofMat = new THREE.MeshStandardMaterial({ color: 0xa65214 });
            const roof = new THREE.Mesh(roofGeo, roofMat);
            roof.position.y = 3.4;
            roof.rotation.y = Math.PI / 4;
            hiveGroup.add(roof);

            scene.add(hiveGroup);
        }

        function createFlower(x, z) {
            const flower = new THREE.Group();
            flower.position.set(x, 0, z);

            const stemMat = new THREE.MeshBasicMaterial({ color: 0x228b22 });
            const stem = new THREE.Mesh(new THREE.CylinderGeometry(0.05, 0.05, 1), stemMat);
            stem.position.y = 0.5;
            flower.add(stem);

            const colors = [0xff0055, 0xffff00, 0xff9900, 0x9900ff, 0xffffff];
            const petalColor = colors[Math.floor(Math.random() * colors.length)];
            const petalMat = new THREE.MeshStandardMaterial({ color: petalColor });

            const center = new THREE.Mesh(new THREE.SphereGeometry(0.2, 8, 8), new THREE.MeshBasicMaterial({ color: 0xffcc00 }));
            center.position.y = 1.0;
            flower.add(center);

            for (let i = 0; i < 5; i++) {
                const angle = (i / 5) * Math.PI * 2;
                const petal = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), petalMat);
                petal.position.set(Math.cos(angle) * 0.25, 1.0, Math.sin(angle) * 0.25);
                flower.add(petal);
            }
            scene.add(flower);
        }

        function createTree(x, z) {
            const tree = new THREE.Group();
            tree.position.set(x, 0, z);

            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x5c4033 });
            const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.8, 5), trunkMat);
            trunk.position.y = 2.5;
            trunk.castShadow = true;
            tree.add(trunk);

            const leavesMat = new THREE.MeshStandardMaterial({ color: 0x2e8b57, roughness: 0.6 });
            const leaves = new THREE.Mesh(new THREE.DodecahedronGeometry(3), leavesMat);
            leaves.position.y = 5.5;
            leaves.castShadow = true;
            tree.add(leaves);

            scene.add(tree);
        }

        function spawnBees() {
            const beeGeo = new THREE.SphereGeometry(0.15, 8, 8);
            const beeMat = new THREE.MeshBasicMaterial({ color: 0xffcc00 });

            for (let i = 0; i < maxBees; i++) {
                const bee = new THREE.Mesh(beeGeo, beeMat);
                resetBeePosition(bee);
                scene.add(bee);
                bees.push(bee);
            }
        }

        function resetBeePosition(bee) {
            bee.position.set(
                (Math.random() - 0.5) * 20,
                1.5 + Math.random() * 3,
                (Math.random() - 0.5) * 20 - 2
            );
            bee.userData = {
                angle: Math.random() * Math.PI * 2,
                speed: 0.03 + Math.random() * 0.03,
                radius: 2 + Math.random() * 8
            };
        }

        function createWeaponMesh() {
            weaponMeshGroup = new THREE.Group();
            // Attach weapon directly to camera view
            cameraPitchGroup.add(weaponMeshGroup);
            weaponMeshGroup.position.set(0.4, -0.4, -0.6);

            updateWeaponVisual();
        }

        function updateWeaponVisual() {
            // Clear existing weapon models
            while (weaponMeshGroup.children.length > 0) {
                weaponMeshGroup.remove(weaponMeshGroup.children[0]);
            }

            const wData = WEAPONS[currentWeaponIdx];
            const mat = new THREE.MeshStandardMaterial({ color: wData.color, metalness: 0.3, roughness: 0.5 });

            if (wData.id === 'swatter') {
                // Flyswatter
                const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.02, 0.02, 0.8), mat);
                handle.rotation.x = Math.PI / 4;
                weaponMeshGroup.add(handle);

                const head = new THREE.Mesh(new THREE.BoxGeometry(0.3, 0.4, 0.02), mat);
                head.position.set(0, 0.35, -0.35);
                head.rotation.x = Math.PI / 4;
                weaponMeshGroup.add(head);

            } else if (wData.id === 'basin') {
                // Metal Basin
                const basin = new THREE.Mesh(new THREE.CylinderGeometry(0.4, 0.25, 0.2, 16, 1, true), mat);
                basin.rotation.x = Math.PI / 3;
                weaponMeshGroup.add(basin);

            } else if (wData.id === 'net') {
                // Butterfly Net
                const handle = new THREE.Mesh(new THREE.CylinderGeometry(0.025, 0.025, 1.2), mat);
                handle.rotation.x = Math.PI / 4;
                weaponMeshGroup.add(handle);

                const ring = new THREE.Mesh(new THREE.TorusGeometry(0.3, 0.02, 8, 16), mat);
                ring.position.set(0, 0.5, -0.5);
                ring.rotation.x = Math.PI / 4;
                weaponMeshGroup.add(ring);

            } else if (wData.id === 'hanger') {
                // Wire Hanger
                const hanger = new THREE.Mesh(new THREE.TorusGeometry(0.25, 0.015, 6, 3), mat);
                hanger.rotation.x = Math.PI / 3;
                weaponMeshGroup.add(hanger);

            } else if (wData.id === 'tongs') {
                // Tongs
                const t1 = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.02, 0.7), mat);
                t1.position.x = -0.04;
                const t2 = new THREE.Mesh(new THREE.BoxGeometry(0.03, 0.02, 0.7), mat);
                t2.position.x = 0.04;
                weaponMeshGroup.add(t1);
                weaponMeshGroup.add(t2);

            } else if (wData.id === 'gloves') {
                // Work Gloves
                const hand = new THREE.Mesh(new THREE.BoxGeometry(0.25, 0.1, 0.3), mat);
                weaponMeshGroup.add(hand);

            } else if (wData.id === 'smoker') {
                // Bee Smoker
                const body = new THREE.Mesh(new THREE.CylinderGeometry(0.18, 0.18, 0.4), mat);
                const spout = new THREE.Mesh(new THREE.ConeGeometry(0.1, 0.2, 8), mat);
                spout.position.set(0, 0.25, -0.1);
                spout.rotation.x = -Math.PI / 4;
                weaponMeshGroup.add(body);
                weaponMeshGroup.add(spout);
            }
        }

        function spawnPests() {
            // Remove existing pests
            pests.forEach(p => scene.remove(p.mesh));
            pests = [];

            const types = ['hornet', 'toad', 'dragonfly', 'spider'];

            for (let i = 0; i < TOTAL_PESTS; i++) {
                const type = types[i % types.length];
                const pestMesh = createPestMesh(type);

                // Spawn randomly around map
                const angle = Math.random() * Math.PI * 2;
                const dist = 12 + Math.random() * 25;
                const x = Math.cos(angle) * dist;
                const z = Math.sin(angle) * dist;
                const y = (type === 'toad' || type === 'spider') ? 0.5 : 2 + Math.random() * 3;

                pestMesh.position.set(x, y, z);
                scene.add(pestMesh);

                pests.push({
                    mesh: pestMesh,
                    type: type,
                    hp: type === 'toad' ? 120 : (type === 'hornet' ? 80 : 50),
                    maxHp: type === 'toad' ? 120 : (type === 'hornet' ? 80 : 50),
                    speed: type === 'dragonfly' ? 0.12 : (type === 'hornet' ? 0.09 : 0.05),
                    state: 'hunting',
                    targetBee: null
                });
            }

            updatePestHUD();
        }

        function createPestMesh(type) {
            const group = new THREE.Group();

            if (type === 'hornet') {
                // Giant Hornet (Yellow/Black)
                const bodyMat = new THREE.MeshStandardMaterial({ color: 0xffaa00 });
                const stripeMat = new THREE.MeshStandardMaterial({ color: 0x111111 });

                const head = new THREE.Mesh(new THREE.SphereGeometry(0.3, 8, 8), bodyMat);
                head.position.z = -0.4;
                group.add(head);

                const thorax = new THREE.Mesh(new THREE.SphereGeometry(0.4, 8, 8), stripeMat);
                group.add(thorax);

                const abdomen = new THREE.Mesh(new THREE.ConeGeometry(0.35, 0.9, 8), bodyMat);
                abdomen.rotation.x = -Math.PI / 2;
                abdomen.position.z = 0.6;
                group.add(abdomen);

                // Wings
                const wingMat = new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.6 });
                const wing1 = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.02, 0.3), wingMat);
                wing1.position.set(0.4, 0.3, 0);
                const wing2 = wing1.clone();
                wing2.position.set(-0.4, 0.3, 0);
                group.add(wing1);
                group.add(wing2);

            } else if (type === 'toad') {
                // Toad (Green Ground Pest)
                const toadMat = new THREE.MeshStandardMaterial({ color: 0x3d7a22, roughness: 0.9 });
                const body = new THREE.Mesh(new THREE.SphereGeometry(0.7, 8, 8), toadMat);
                body.scale.set(1, 0.7, 1.2);
                group.add(body);

                const eyeMat = new THREE.MeshBasicMaterial({ color: 0xffcc00 });
                const eye1 = new THREE.Mesh(new THREE.SphereGeometry(0.15, 8, 8), eyeMat);
                eye1.position.set(0.3, 0.4, -0.5);
                const eye2 = eye1.clone();
                eye2.position.set(-0.3, 0.4, -0.5);
                group.add(eye1);
                group.add(eye2);

            } else if (type === 'dragonfly') {
                // Dragonfly (Blue fast pest)
                const dfMat = new THREE.MeshStandardMaterial({ color: 0x00ccff });
                const body = new THREE.Mesh(new THREE.CylinderGeometry(0.1, 0.05, 1.4), dfMat);
                body.rotation.x = Math.PI / 2;
                group.add(body);

                const wingMat = new THREE.MeshBasicMaterial({ color: 0xaaffff, transparent: true, opacity: 0.5 });
                const w1 = new THREE.Mesh(new THREE.BoxGeometry(1.4, 0.01, 0.2), wingMat);
                w1.position.set(0, 0.1, -0.2);
                group.add(w1);

            } else if (type === 'spider') {
                // Spider (Dark red crawling)
                const spMat = new THREE.MeshStandardMaterial({ color: 0x660000 });
                const body = new THREE.Mesh(new THREE.SphereGeometry(0.5, 8, 8), spMat);
                group.add(body);

                for (let i = 0; i < 8; i++) {
                    const leg = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 0.8), spMat);
                    const angle = (i / 8) * Math.PI * 2;
                    leg.position.set(Math.cos(angle) * 0.5, -0.1, Math.sin(angle) * 0.5);
                    leg.rotation.z = Math.PI / 3;
                    group.add(leg);
                }
            }

            return group;
        }

        function setupControls() {
            const startBtn = document.getElementById('start-btn');
            const restartBtn = document.getElementById('restart-btn');

            startBtn.addEventListener('click', () => {
                document.getElementById('start-screen').classList.add('hidden');
                document.getElementById('hud').classList.remove('hidden');
                document.getElementById('crosshair').classList.remove('hidden');
                document.body.requestPointerLock();
                gameActive = true;
            });

            restartBtn.addEventListener('click', () => {
                document.getElementById('end-screen').classList.add('hidden');
                resetGame();
                document.body.requestPointerLock();
                gameActive = true;
            });

            // Pointer lock change listener
            document.addEventListener('pointerlockchange', () => {
                if (document.pointerLockElement !== document.body && gameActive) {
                    // Paused or unlocked
                }
            });

            // Keyboard Listeners
            window.addEventListener('keydown', (e) => {
                if (!gameActive) return;
                const k = e.key.toLowerCase();
                if (k === 'w') keys.w = true;
                if (k === 'a') keys.a = true;
                if (k === 's') keys.s = true;
                if (k === 'd') keys.d = true;
                if (e.code === 'Space' && isGrounded) {
                    velocityY = 0.28;
                    isGrounded = false;
                }
                if (e.key === 'Shift') {
                    // Switch Weapon
                    currentWeaponIdx = (currentWeaponIdx + 1) % WEAPONS.length;
                    updateWeaponVisual();
                    updateWeaponHUD();
                }
            });

            window.addEventListener('keyup', (e) => {
                const k = e.key.toLowerCase();
                if (k === 'w') keys.w = false;
                if (k === 'a') keys.a = false;
                if (k === 's') keys.s = false;
                if (k === 'd') keys.d = false;
            });

            // Mouse Look Listeners
            window.addEventListener('mousemove', (e) => {
                if (document.pointerLockElement !== document.body || !gameActive) return;

                const sensitivity = 0.0025;
                yaw -= e.movementX * sensitivity;
                pitch -= e.movementY * sensitivity;

                // Clamp pitch
                pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, pitch));

                playerContainer.rotation.y = yaw;
                cameraPitchGroup.rotation.x = pitch;
            });

            // Mouse Click Attack
            window.addEventListener('mousedown', (e) => {
                if (document.pointerLockElement === document.body && e.button === 0 && gameActive) {
                    performAttack();
                }
            });
        }

        function performAttack() {
            const now = Date.now();
            const wData = WEAPONS[currentWeaponIdx];

            if (now - lastAttackTime < wData.cooldown) return;
            lastAttackTime = now;

            playSound('swing');

            // Weapon Swing Visual Animation
            weaponMeshGroup.position.z = -0.9;
            weaponMeshGroup.rotation.z = -0.5;
            setTimeout(() => {
                weaponMeshGroup.position.z = -0.6;
                weaponMeshGroup.rotation.z = 0;
            }, 120);

            // Raycast / Distance Check for Pests in front of Camera
            const cameraDir = new THREE.Vector3();
            camera.getWorldDirection(cameraDir);
            const playerPos = playerContainer.position;

            pests.forEach((pest, idx) => {
                const dist = playerPos.distanceTo(pest.mesh.position);
                
                if (dist <= wData.range) {
                    // Check angle relative to crosshair view
                    const toPest = pest.mesh.position.clone().sub(playerPos).normalize();
                    const angle = cameraDir.angleTo(toPest);

                    if (angle < 0.6) { // Within attack cone
                        // HIT PEST!
                        pest.hp -= wData.damage;
                        playSound('hit');

                        // Pop Comic Hit Effect
                        showHitComicPop(wData.name, pest.mesh.position);

                        // Knockback
                        const pushDir = toPest.clone().multiplyScalar(1.5);
                        pest.mesh.position.add(pushDir);

                        // Check Pest Death
                        if (pest.hp <= 0) {
                            scene.remove(pest.mesh);
                            pests.splice(idx, 1);
                            updatePestHUD();
                            checkWinCondition();
                        }
                    }
                }
            });
        }

        function showHitComicPop(weaponName, worldPos) {
            const hits = ["SMACK!", "THWACK!", "BUZZ!", "BAM!", "POW!", "CRASH!"];
            const txt = hits[Math.floor(Math.random() * hits.length)];

            const pop = document.createElement('div');
            pop.className = 'hit-pop text-5xl font-black text-yellow-300';
            pop.innerText = txt;

            document.getElementById('pop-container').appendChild(pop);

            setTimeout(() => {
                pop.remove();
            }, 500);
        }

        function updatePlayer() {
            if (!gameActive) return;

            // Player Jumping & Gravity Physics
            playerContainer.position.y += velocityY;
            if (playerContainer.position.y > 2.0) {
                velocityY += gravity;
            } else {
                playerContainer.position.y = 2.0;
                velocityY = 0;
                isGrounded = true;
            }

            // Directional Movement
            const moveDir = new THREE.Vector3();
            if (keys.w) moveDir.z -= 1;
            if (keys.s) moveDir.z += 1;
            if (keys.a) moveDir.x -= 1;
            if (keys.d) moveDir.x += 1;

            moveDir.normalize();
            moveDir.applyAxisAngle(new THREE.Vector3(0, 1, 0), yaw);

            playerContainer.position.addScaledVector(moveDir, moveSpeed);

            // Boundary clamping
            playerContainer.position.x = Math.max(-48, Math.min(48, playerContainer.position.x));
            playerContainer.position.z = Math.max(-48, Math.min(48, playerContainer.position.z));
        }

        function updateBeesAndPests() {
            if (!gameActive) return;

            // Animate Honeybees Flying
            bees.forEach((bee) => {
                bee.userData.angle += bee.userData.speed;
                bee.position.x += Math.cos(bee.userData.angle) * 0.05;
                bee.position.z += Math.sin(bee.userData.angle) * 0.05;
            });

            // Animate Pests AI
            pests.forEach((pest) => {
                // Wing flapping animation for flying pests
                if (pest.type === 'hornet' || pest.type === 'dragonfly') {
                    pest.mesh.position.y += Math.sin(Date.now() * 0.01) * 0.02;
                }

                // AI move towards nearest hive/bees
                const targetPos = new THREE.Vector3(0, pest.type === 'toad' || pest.type === 'spider' ? 0.5 : 2, -2);
                const dir = targetPos.clone().sub(pest.mesh.position).normalize();
                
                pest.mesh.position.addScaledVector(dir, pest.speed);
                pest.mesh.lookAt(targetPos);

                // Pest eats honeybees if close to center hive
                if (pest.mesh.position.distanceTo(targetPos) < 2.5) {
                    if (Math.random() < 0.02) {
                        beeCount = Math.max(0, beeCount - 1);
                        playSound('bee_eat');
                        updateBeeHUD();
                        checkLoseCondition();
                    }
                }
            });
        }

        function updateWeaponHUD() {
            const w = WEAPONS[currentWeaponIdx];
            document.getElementById('weapon-name').innerText = w.name;
            document.getElementById('weapon-desc').innerText = w.desc;
            document.getElementById('weapon-icon').innerHTML = `<i class="fa-solid ${w.icon}"></i>`;
        }

        function updateBeeHUD() {
            document.getElementById('bee-count-text').innerText = `${beeCount} / ${maxBees}`;
            const pct = (beeCount / maxBees) * 100;
            document.getElementById('bee-health-bar').style.width = `${pct}%`;
        }

        function updatePestHUD() {
            document.getElementById('pest-count-text').innerText = `${pests.length} LEFT`;
            const pct = (pests.length / TOTAL_PESTS) * 100;
            document.getElementById('pest-bar').style.width = `${pct}%`;
        }

        function checkWinCondition() {
            if (pests.length === 0) {
                gameActive = false;
                document.exitPointerLock();
                document.getElementById('end-title').innerText = "VICTORY!";
                document.getElementById('end-title').className = "comic-font text-6xl text-yellow-400 tracking-wider mb-4 drop-shadow-[4px_4px_0px_#000]";
                document.getElementById('end-desc').innerText = "모든 천적을 물리치고 꿀벌을 지켜냈습니다!";
                document.getElementById('end-score').innerText = `최종 남은 꿀벌: ${beeCount}마리`;
                document.getElementById('end-screen').classList.remove('hidden');
            }
        }

        function checkLoseCondition() {
            if (beeCount <= 0) {
                gameActive = false;
                document.exitPointerLock();
                document.getElementById('end-title').innerText = "DEFEAT!";
                document.getElementById('end-title').className = "comic-font text-6xl text-red-600 tracking-wider mb-4 drop-shadow-[4px_4px_0px_#000]";
                document.getElementById('end-desc').innerText = "꿀벌들이 모두 천적에게 잡아먹혔습니다...";
                document.getElementById('end-score').innerText = `퇴치하지 못한 천적: ${pests.length}마리`;
                document.getElementById('end-screen').classList.remove('hidden');
            }
        }

        function resetGame() {
            beeCount = maxBees;
            playerContainer.position.set(0, 2, 15);
            yaw = 0;
            pitch = 0;
            currentWeaponIdx = 0;
            updateWeaponVisual();
            updateWeaponHUD();
            updateBeeHUD();
            spawnPests();
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        function animate() {
            requestAnimationFrame(animate);

            updatePlayer();
            updateBeesAndPests();

            renderer.render(scene, camera);
        }

        // Window Load Initialization
        window.onload = function() {
            init3D();
            setupControls();
            spawnPests();
            animate();
        };
    </script>
</body>
</html>
