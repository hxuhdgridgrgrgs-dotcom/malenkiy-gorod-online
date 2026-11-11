Вот код для страницы с поляной, дорогой и домиками:
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Городок</title>
    <style>
        body { margin: 0; background: #2e7d32; display: flex; justify-content: center; align-items: center; height: 100vh; }
        canvas { background: #81c784; }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="600" height="400"></canvas>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        function draw() {
            // Дорога
            ctx.fillStyle = "#555";
            ctx.fillRect(0, 250, 600, 100);

            // Дома
            function drawHouse(x) {
                ctx.fillStyle = "#8d6e63";
                ctx.fillRect(x, 180, 50, 70);
                ctx.fillStyle = "#d32f2f";
                ctx.beginPath();
                ctx.moveTo(x - 10, 180);
                ctx.lineTo(x + 25, 130);
                ctx.lineTo(x + 60, 180);
                ctx.fill();
            }
            drawHouse(100);
            drawHouse(280);
            drawHouse(460);
        }
        draw();
    </script>
</body>
</html>

Вставьте этот код в index.html на GitHub и нажмите «Commit changes».


        function draw() {
            // Дорога
            ctx.fillStyle = "#555";
            ctx.fillRect(0, 250, 600, 100);

            // Дома
            function drawHouse(x) {
                ctx.fillStyle = "#8d6e63";
                ctx.fillRect(x, 180, 50, 70);
                ctx.fillStyle = "#d32f2f";
                ctx.beginPath();
                ctx.moveTo(x - 10, 180);
                ctx.lineTo(x + 25, 130);
                ctx.lineTo(x + 60, 180);
                ctx.fill();
            }
            drawHouse(100);
            drawHouse(280);
            drawHouse(460);
        }
        draw();
    </script>
</body>
</html>

