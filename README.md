# meusgames
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lebre vs Falcão: Speed Run Pós-Páscoa</title>
    <style>
        body { 
            margin: 0; 
            overflow: hidden; 
            font-family: Arial, sans-serif;
        }
        #info {
            position: absolute;
            top: 10px;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 24px;
            text-shadow: 1px 1px 2px black;
            z-index: 100;
        }
        #score {
            position: absolute;
            color: white;
            font-size: 20px;
            text-shadow: 1px 1px 2px black;
            padding: 10px;
            top: 10px;
            right: 10px;
            z-index: 100;
        }
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 36px;
            text-shadow: 2px 2px 4px black;
            text-align: center;
            z-index: 200;
            display: none;
            background-color: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 10px;
        }
        button {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 18px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        button:hover {
            background-color: #45a049;
        }
        #instructions {
            position: absolute;
            bottom: 10px;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 16px;
            text-shadow: 1px 1px 2px black;
            z-index: 100;
        }
    </style>
</head>
<body>
    <div id="info">Lebre vs Falcão: Speed Run Pós-Páscoa</div>
    <div id="score">Ovos: 0</div>
    <div id="instructions">Use as setas ou WASD para mover | Espaço para pular | Colete ovos e evite o falcão!</div>
    <div id="gameOver">
        <h2>Fim de Jogo!</h2>
        <p id="finalScore">Você coletou 0 ovos!</p>
        <button id="restartButton">Jogar Novamente</button>
    </div>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script>
        // Configuração
        let scene, camera, renderer, clock;
        let rabbit, falcon, ground, eggs = [];
        let trees = [];
        let score = 0;
        let gameActive = true;
        let moveSpeed = 0.15;
        let falconSpeed = 0.08;
        let jumpForce = 0.3;
        let gravity = 0.01;
        let isJumping = false;
        let jumpVelocity = 0;
        let rabbitHeight = 1;
        
        // Mapa de posições ocupadas para evitar sobreposição
        let occupiedPositions = new Set();

        // Controles
        const keysPressed = {};
        
        function init() {
            // Criar cena
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87CEEB); // Céu azul
            
            // Criar câmera
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(0, 4, 8);
            
            // Criar renderer
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);
            
            // Criar luz
            const light = new THREE.DirectionalLight(0xffffff, 1);
            light.position.set(5, 10, 5);
            scene.add(light);
            
            const ambientLight = new THREE.AmbientLight(0x404040);
            scene.add(ambientLight);
            
            // Criar relógio
            clock = new THREE.Clock();
            
            // Criar terreno
            createGround();
            
            // Criar personagens
            createRabbit();
            createFalcon();
            
            // Criar árvores
            createTrees();
            
            // Criar ovos iniciais
            for(let i = 0; i < 5; i++) {
                createEgg();
            }
            
            // Adicionar eventos
            window.addEventListener('keydown', (e) => keysPressed[e.key] = true);
            window.addEventListener('keyup', (e) => keysPressed[e.key] = false);
            window.addEventListener('resize', onWindowResize);
            
            // Iniciar animação
            animate();
        }
        
        function createGround() {
            // Chão verde (grama)
            const groundGeometry = new THREE.PlaneGeometry(100, 100);
            const groundMaterial = new THREE.MeshLambertMaterial({ color: 0x7CFC00 });
            ground = new THREE.Mesh(groundGeometry, groundMaterial);
            ground.rotation.x = -Math.PI / 2;
            ground.position.y = 0;
            scene.add(ground);
        }
        
        function createRabbit() {
            // Corpo da lebre
            const rabbitBody = new THREE.Group();
            
            // Corpo principal
            const bodyGeometry = new THREE.SphereGeometry(0.5, 16, 16);
            const bodyMaterial = new THREE.MeshLambertMaterial({ color: 0xf0f0f0 });
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            rabbitBody.add(body);
            
            // Cabeça
            const headGeometry = new THREE.SphereGeometry(0.3, 16, 16);
            const headMaterial = new THREE.MeshLambertMaterial({ color: 0xf0f0f0 });
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.set(0, 0.5, 0.3);
            rabbitBody.add(head);
            
            // Orelhas
            const earGeometry = new THREE.CylinderGeometry(0.08, 0.05, 0.5, 12);
            const earMaterial = new THREE.MeshLambertMaterial({ color: 0xffcccc });
            
            const leftEar = new THREE.Mesh(earGeometry, earMaterial);
            leftEar.position.set(0.15, 0.8, 0.3);
            leftEar.rotation.x = -Math.PI / 6;
            rabbitBody.add(leftEar);
            
            const rightEar = new THREE.Mesh(earGeometry, earMaterial);
            rightEar.position.set(-0.15, 0.8, 0.3);
            rightEar.rotation.x = -Math.PI / 6;
            rabbitBody.add(rightEar);
            
            // Rabo fofinho
            const tailGeometry = new THREE.SphereGeometry(0.15, 16, 16);
            const tailMaterial = new THREE.MeshLambertMaterial({ color: 0xffffff });
            const tail = new THREE.Mesh(tailGeometry, tailMaterial);
            tail.position.set(0, 0.1, -0.5);
            rabbitBody.add(tail);
            
            // Posicionar lebre
            rabbitBody.position.set(0, rabbitHeight, 0);
            rabbitBody.rotation.y = Math.PI;
            
            rabbit = rabbitBody;
            scene.add(rabbit);
            
            // Adicionar posição da lebre às posições ocupadas
            markPositionAsOccupied(0, 0);
        }
        
        function createFalcon() {
            // Corpo do falcão
            const falconBody = new THREE.Group();
            
            // Corpo principal
            const bodyGeometry = new THREE.ConeGeometry(0.3, 1, 8);
            const bodyMaterial = new THREE.MeshLambertMaterial({ color: 0x8B4513 });
            const body = new THREE.Mesh(bodyGeometry, bodyMaterial);
            body.rotation.x = Math.PI / 2;
            falconBody.add(body);
            
            // Asas
            const wingGeometry = new THREE.PlaneGeometry(1.2, 0.5);
            const wingMaterial = new THREE.MeshLambertMaterial({ color: 0x654321, side: THREE.DoubleSide });
            
            const leftWing = new THREE.Mesh(wingGeometry, wingMaterial);
            leftWing.position.set(0.6, 0, 0);
            leftWing.rotation.y = Math.PI / 2;
            falconBody.add(leftWing);
            
            const rightWing = new THREE.Mesh(wingGeometry, wingMaterial);
            rightWing.position.set(-0.6, 0, 0);
            rightWing.rotation.y = Math.PI / 2;
            falconBody.add(rightWing);
            
            // Cabeça
            const headGeometry = new THREE.SphereGeometry(0.2, 16, 16);
            const headMaterial = new THREE.MeshLambertMaterial({ color: 0x8B4513 });
            const head = new THREE.Mesh(headGeometry, headMaterial);
            head.position.set(0, 0, 0.5);
            falconBody.add(head);
            
            // Bico
            const beakGeometry = new THREE.ConeGeometry(0.05, 0.2, 8);
            const beakMaterial = new THREE.MeshLambertMaterial({ color: 0xFFD700 });
            const beak = new THREE.Mesh(beakGeometry, beakMaterial);
            beak.position.set(0, -0.1, 0.7);
            beak.rotation.x = Math.PI / 2;
            falconBody.add(beak);
            
            // Posicionar falcão
            falconBody.position.set(10, 5, 10);
            
            falcon = falconBody;
            scene.add(falcon);
        }
        
        // Função auxiliar para verificar se uma posição está ocupada
        function isPositionOccupied(x, z, radius = 2) {
            for (let i = -radius; i <= radius; i++) {
                for (let j = -radius; j <= radius; j++) {
                    const key = `${Math.round(x + i)},${Math.round(z + j)}`;
                    if (occupiedPositions.has(key)) {
                        return true;
                    }
                }
            }
            return false;
        }

        // Função para marcar uma posição como ocupada
        function markPositionAsOccupied(x, z) {
            const key = `${Math.round(x)},${Math.round(z)}`;
            occupiedPositions.add(key);
        }

        // Função para encontrar uma posição não ocupada
        function findFreePosition() {
            let x, z;
            let attempts = 0;
            const maxAttempts = 50;
            
            do {
                // Gerar posição aleatória dentro de um círculo ao redor do ponto 0,0
                const radius = Math.random() * 35 + 5; // Entre 5 e 40
                const angle = Math.random() * Math.PI * 2;
                x = Math.cos(angle) * radius;
                z = Math.sin(angle) * radius;
                
                attempts++;
                if (attempts > maxAttempts) {
                    // Evitar loop infinito
                    return { x: x, z: z };
                }
            } while (isPositionOccupied(x, z));
            
            markPositionAsOccupied(x, z);
            return { x: x, z: z };
        }
        
        function createEgg() {
            // Ovo de páscoa com forma mais oval
            const eggGeometry = new THREE.SphereGeometry(0.25, 16, 16);
            eggGeometry.scale(1, 1.3, 1);
            
            // Cores aleatórias para os ovos
            const colors = [0xFF69B4, 0x00BFFF, 0x7CFC00, 0xFFD700, 0xFFA500, 0x9370DB, 0xFF6347];
            const randomColor = colors[Math.floor(Math.random() * colors.length)];
            
            // Material com brilho
            const eggMaterial = new THREE.MeshPhongMaterial({ 
                color: randomColor,
                specular: 0xffffff,
                shininess: 100
            });
            
            const egg = new THREE.Mesh(eggGeometry, eggMaterial);
            
            // Encontrar posição livre no mapa
            const pos = findFreePosition();
            
            // Posicionar ovo ACIMA do chão
            egg.position.set(pos.x, 0.4, pos.z);
            
            // Adicionar rotação para parecer mais natural
            egg.rotation.x = Math.random() * 0.5;
            egg.rotation.z = Math.random() * 0.5;
            
            // Adicionar pequena animação
            egg.userData = {
                originalY: 0.4,
                phase: Math.random() * Math.PI * 2
            };
            
            scene.add(egg);
            eggs.push(egg);
        }
        
        function createTrees() {
            for(let i = 0; i < 15; i++) {
                const pos = findFreePosition();
                
                // Tronco
                const trunkGeometry = new THREE.CylinderGeometry(0.2, 0.3, 1.5, 8);
                const trunkMaterial = new THREE.MeshLambertMaterial({ color: 0x8B4513 });
                const trunk = new THREE.Mesh(trunkGeometry, trunkMaterial);
                
                // Copa
                const leafGeometry = new THREE.ConeGeometry(1, 2, 8);
                const leafMaterial = new THREE.MeshLambertMaterial({ color: 0x228B22 });
                const leaves = new THREE.Mesh(leafGeometry, leafMaterial);
                leaves.position.y = 1.7;
                
                // Árvore completa
                const tree = new THREE.Group();
                tree.add(trunk);
                tree.add(leaves);
                
                tree.position.set(pos.x, 0.75, pos.z);
                
                scene.add(tree);
                trees.push(tree);
            }
        }
        
        function gameOver() {
            gameActive = false;
            document.getElementById('gameOver').style.display = 'block';
            document.getElementById('finalScore').textContent = `Você coletou ${score} ovos!`;
            
            document.getElementById('restartButton').addEventListener('click', () => {
                location.reload();
            });
        }
        
        function updateRabbit() {
            // Movimento da lebre
            if(keysPressed['ArrowUp'] || keysPressed['w']) {
                rabbit.position.z -= moveSpeed;
                rabbit.rotation.y = Math.PI;
            }
            if(keysPressed['ArrowDown'] || keysPressed['s']) {
                rabbit.position.z += moveSpeed;
                rabbit.rotation.y = 0;
            }
            if(keysPressed['ArrowLeft'] || keysPressed['a']) {
                rabbit.position.x -= moveSpeed;
                rabbit.rotation.y = Math.PI / 2;
            }
            if(keysPressed['ArrowRight'] || keysPressed['d']) {
                rabbit.position.x += moveSpeed;
                rabbit.rotation.y = -Math.PI / 2;
            }
            
            // Pulo
            if((keysPressed[' '] || keysPressed['Spacebar']) && !isJumping) {
                isJumping = true;
                jumpVelocity = jumpForce;
            }
            
            // Aplicar física de pulo
            if(isJumping) {
                rabbit.position.y += jumpVelocity;
                jumpVelocity -= gravity;
                
                if(rabbit.position.y <= rabbitHeight) {
                    rabbit.position.y = rabbitHeight;
                    isJumping = false;
                }
            }
            
            // Limitar área de jogo
            if(rabbit.position.x > 40) rabbit.position.x = 40;
            if(rabbit.position.x < -40) rabbit.position.x = -40;
            if(rabbit.position.z > 40) rabbit.position.z = 40;
            if(rabbit.position.z < -40) rabbit.position.z = -40;
            
            // Atualizar câmera para seguir lebre
            camera.position.x = rabbit.position.x;
            camera.position.z = rabbit.position.z + 8;
            camera.lookAt(rabbit.position);
        }
        
        function updateFalcon() {
            // Movimento do falcão em direção à lebre
            const direction = new THREE.Vector3();
            direction.subVectors(rabbit.position, falcon.position).normalize();
            
            falcon.position.x += direction.x * falconSpeed;
            falcon.position.z += direction.z * falconSpeed;
            
            // Manter altura do falcão
            falcon.position.y = 5 + Math.sin(clock.getElapsedTime()) * 0.5;
            
            // Rotacionar falcão para olhar para a lebre
            falcon.lookAt(rabbit.position);
            
            // Verificar colisão com a lebre
            const distance = falcon.position.distanceTo(rabbit.position);
            if(distance < 1.5 && !isJumping) {
                gameOver();
            }
        }
        
        function checkEggCollisions() {
            for(let i = eggs.length - 1; i >= 0; i--) {
                const distance = rabbit.position.distanceTo(eggs[i].position);
                
                if(distance < 1) {
                    // Coletar ovo
                    scene.remove(eggs[i]);
                    eggs.splice(i, 1);
                    
                    // Atualizar pontuação
                    score++;
                    document.getElementById('score').textContent = `Ovos: ${score}`;
                    
                    // Criar novo ovo
                    createEgg();
                    
                    // Aumentar velocidade da lebre (bônus por coletar ovos)
                    moveSpeed += 0.005;
                    // Aumentar velocidade do falcão para manter desafio
                    falconSpeed += 0.003;
                }
            }
        }
        
        function updateEggs() {
            // Aplicar pequena animação flutuante aos ovos
            const time = clock.getElapsedTime();
            
            eggs.forEach(egg => {
                if (egg.userData) {
                    egg.position.y = egg.userData.originalY + Math.sin(time * 2 + egg.userData.phase) * 0.05;
                    egg.rotation.y += 0.01; // Rotação suave
                }
            });
        }
        
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        function animate() {
            if(!gameActive) return;
            
            requestAnimationFrame(animate);
            
            if(gameActive) {
                updateRabbit();
                updateFalcon();
                updateEggs();
                checkEggCollisions();
            }
            
            renderer.render(scene, camera);
        }
        
        // Iniciar jogo
        init();
    </script>
</body>
</html>
