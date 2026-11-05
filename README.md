<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Truy Bắt Khuyển Tặc - Mini Game</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background: #1a1a1a;
            color: #fff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        #game-container {
            position: relative;
            width: 800px;
            height: 500px;
            background: #2b2b2b;
            border: 4px solid #f1c40f;
            box-shadow: 0 0 20px rgba(241, 196, 15, 0.5);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
        }
        canvas {
            background: #333;
            display: none;
        }
        .screen {
            text-align: center;
            padding: 20px;
        }
        h1 { color: #f1c40f; margin-bottom: 10px; text-transform: uppercase; }
        p { color: #ccc; margin-bottom: 20px; }
        .btn {
            background: #e74c3c;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 5px;
            margin: 5px;
            transition: 0.2s;
        }
        .btn:hover { background: #c0392b; transform: scale(1.05); }
        .btn-police { background: #3498db; }
        .btn-police:hover { background: #2980b9; }
        .btn-dog { background: #2ecc71; }
        .btn-dog:hover { background: #27ae60; }
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            right: 10px;
            display: flex;
            justify-content: space-between;
            font-size: 16px;
            font-weight: bold;
            background: rgba(0,0,0,0.6);
            padding: 10px;
            border-radius: 5px;
            display: none;
        }
    </style>
</head>
<body>

    <div id="game-container">
        <!-- Màn hình Menu Chọn Vai Trò -->
        <div id="menu-screen" class="screen">
            <h1>Truy Bắt Khuyển Tặc 🐕‍🦺</h1>
            <p>Chọn vai trò của bạn trong cuộc chiến xóm nhỏ đêm khuya:</p>
            <button class="btn" onclick="startGame('thief')">1. 🛵 Trộm Chó (Săn mồi & Chạy trốn)</button><br>
            <button class="btn btn-police" onclick="startGame('police')">2. 👮 Công An (Truy quét tội phạm)</button><br>
            <button class="btn btn-dog" onclick="startGame('dog')">3. 🐕 Chó Cỏ (Chạy trốn, Cắn trả & Tiến Hóa K9)</button>
        </div>

        <!-- Thanh trạng thái trong game -->
        <div id="ui">
            <span id="ui-role">Vai trò: -</span>
            <span id="ui-score">Điểm/EXP: 0</span>
            <span id="ui-status">Trạng thái: Bình thường</span>
        </div>

        <!-- Màn chơi chính (Canvas) -->
        <canvas id="gameCanvas" width="800" height="500"></canvas>
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        let currentRole = "";
        let gameRunning = false;
        let score = 0;
        let level = 1;

        // Đối tượng người chơi và kẻ địch
        let player = { x: 400, y: 250, size: 25, speed: 4, color: "#fff" };
        let target = { x: 100, y: 100, size: 20, speed: 2, color: "#e74c3c" };

        let keys = {};

        // Lắng nghe bàn phím
        window.addEventListener("keydown", (e) => keys[e.key] = true);
        window.addEventListener("keyup", (e) => keys[e.key] = false);

        function startGame(role) {
            currentRole = role;
            document.getElementById("menu-screen").style.display = "none";
            canvas.style.display = "block";
            document.getElementById("ui").style.display = "flex";
            
            gameRunning = true;
            score = 0;
            level = 1;

            // Khởi tạo vị trí dựa trên vai trò
            if (role === 'thief') {
                player.color = "#e74c3c"; // Đỏ (Trộm)
                document.getElementById("ui-role").innerText = "Vai trò: 🛵 Trộm Chó";
            } else if (role === 'police') {
                player.color = "#3498db"; // Xanh dương (Công an)
                document.getElementById("ui-role").innerText = "Vai trò: 👮 Công An Xã";
            } else if (role === 'dog') {
                player.color = "#2ecc71"; // Xanh lá (Chó Cỏ Lv.1)
                document.getElementById("ui-role").innerText = "Vai trò: 🐕 Chó Cỏ (Lv.1)";
            }

            loop();
        }

        function update() {
            if (!gameRunning) return;

            // Điều khiển bàn phím (W, A, S, D hoặc Mũi tên)
            if ((keys["ArrowUp"] || keys["w"]) && player.y > 50) player.y -= player.speed;
            if ((keys["ArrowDown"] || keys["s"]) && player.y < canvas.height - 20) player.y += player.speed;
            if ((keys["ArrowLeft"] || keys["a"]) && player.x > 20) player.x -= player.speed;
            if ((keys["ArrowRight"] || keys["d"]) && player.x < canvas.width - 20) player.x += player.speed;

            // Logic di chuyển của mục tiêu (Target)
            let dx = player.x - target.x;
            let dy = player.y - target.y;
            let dist = Math.sqrt(dx * dx + dy * dy);

            if (currentRole === 'dog') {
                // Chó chạy trốn hoặc đuổi cắn trộm chó
                if (level < 3) {
                    // Trốn tránh khi chưa lên cấp cao
                    target.x += (Math.random() - 0.5) * 4;
                    target.y += (Math.random() - 0.5) * 4;
                } else {
                    // Cấp cao (K9): Chủ động truy đuổi trộm chó
                    target.x += (dx / dist) * target.speed;
                    target.y += (dy / dist) * target.speed;
                }

                // Va chạm giữa Chó và Kẻ trộm
                if (dist < player.size + target.size) {
                    score += 10;
                    // Tiến hóa qua từng mốc điểm
                    if (score >= 50 && level === 1) {
                        level = 2;
                        player.speed = 5;
                        document.getElementById("ui-role").innerText = "Vai trò: 🐕 Chó Nghiệp Vụ (Lv.2)";
                    } else if (score >= 120 && level === 2) {
                        level = 3;
                        player.color = "#f1c40f"; // Đổi màu vàng (Chó Cảnh Sát K9)
                        player.speed = 6;
                        document.getElementById("ui-role").innerText = "Vai trò: 🛡️ Chó K9 Đội Cảnh Sát";
                    }
                    // Reset vị trí mục tiêu
                    target.x = Math.random() * 700 + 50;
                    target.y = Math.random() * 400 + 50;
                }
            } else if (currentRole === 'thief' || currentRole === 'police') {
                // Trộm hoặc Công an đi săn bắt mục tiêu khác
                if (dist < player.size + target.size) {
                    score += 1;
                    target.x = Math.random() * 700 + 50;
                    target.y = Math.random() * 400 + 50;
                }
            }

            document.getElementById("ui-score").innerText = `Điểm/EXP: ${score}`;
        }

        function draw() {
            // Xóa màn hình
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Vẽ đường biên xóm nhỏ
            ctx.strokeStyle = "#444";
            ctx.lineWidth = 2;
            ctx.strokeRect(10, 40, canvas.width - 20, canvas.height - 50);

            // Vẽ Người chơi
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.arc(player.x, player.y, player.size, 0, Math.PI * 2);
            ctx.fill();

            // Vẽ Mục tiêu (Kẻ trộm hoặc Con mồi)
            ctx.fillStyle = target.color;
            ctx.beginPath();
            ctx.arc(target.x, target.y, target.size, 0, Math.PI * 2);
            ctx.fill();
        }

        function loop() {
            update();
            draw();
            if (gameRunning) {
                requestAnimationFrame(loop);
            }
        }
    </script>
</body>
</html>
