<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>楽しい広場</title>
  <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.15.0/firebase-database-compat.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #333;
      color: #fff;
      margin: 0;
      padding: 0;
    }
    .container {
      width: 90%;
      max-width: 600px;
      margin: 0 auto;
      text-align: left;
    }
    #whiteboard {
      border: 2px solid #444;
      background-color: white;
      width: 100%;
      height: 300px;
      cursor: crosshair;
    }
    .color-buttons {
      display: flex;
      justify-content: center;
      margin: 10px 0;
    }
    .color-button {
      width: 30px;
      height: 30px;
      margin: 0 5px;
      border-radius: 50%;
      cursor: pointer;
      border: 2px solid #fff;
    }
    .red { background-color: red; }
    .green { background-color: green; }
    .blue { background-color: blue; }
    .yellow { background-color: yellow; }
  </style>
</head>
<body>
  <div class="container">
    <h1>楽しい広場</h1>

    <!-- ホワイトボード -->
    <canvas id="whiteboard"></canvas>
    <div class="color-buttons">
      <div class="color-button red" onclick="setColor('red')"></div>
      <div class="color-button green" onclick="setColor('green')"></div>
      <div class="color-button blue" onclick="setColor('blue')"></div>
      <div class="color-button yellow" onclick="setColor('yellow')"></div>
    </div>
    <button onclick="setEraser()">消しゴム</button>
    <button onclick="clearCanvas()">全消去</button>
  </div>

  <script>
    // Firebase 初期化
    const firebaseConfig = {
      apiKey: "AIzaSyDdLAHqEKekLAxKTiC2dTefIVZnabWdW5I",
      authDomain: "yauto108-72184.firebaseapp.com",
      databaseURL: "https://yauto108-72184-default-rtdb.asia-southeast1.firebasedatabase.app",
      projectId: "yauto108-72184",
      storageBucket: "yauto108-72184.firebasestorage.app",
      messagingSenderId: "783689974461",
      appId: "1:783689974461:web:e35b650705cfa8ab08edd4",
      measurementId: "G-SVFHC5KKHB"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // ホワイトボードの初期化
    const canvas = document.getElementById('whiteboard');
    const ctx = canvas.getContext('2d', { willReadFrequently: true });
    let drawing = false;
    let color = '#000000';
    let eraser = false;
    let lastPosition = null;

    canvas.width = canvas.offsetWidth;
    canvas.height = canvas.offsetHeight;

    // リサイズ時にキャンバスを保持
    window.addEventListener('resize', () => {
      const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      canvas.width = canvas.offsetWidth;
      canvas.height = canvas.offsetHeight;
      ctx.putImageData(imgData, 0, 0);
    });

    // マウスイベント
    canvas.addEventListener('mousedown', (event) => startDrawing(event));
    canvas.addEventListener('mouseup', stopDrawing);
    canvas.addEventListener('mousemove', (event) => draw(event));

    // タッチイベント
    canvas.addEventListener('touchstart', (event) => {
      event.preventDefault();
      startDrawing(event.touches[0]);
    });
    canvas.addEventListener('touchend', (event) => {
      event.preventDefault();
      stopDrawing();
    });
    canvas.addEventListener('touchmove', (event) => {
      event.preventDefault();
      draw(event.touches[0]);
    });

    function startDrawing(event) {
      drawing = true;
      lastPosition = getMousePosition(event);
    }

    function stopDrawing() {
      drawing = false;
      ctx.beginPath();
      lastPosition = null;
      saveDrawingData({ end: true });
    }

    function draw(event) {
      if (!drawing) return;
      const { x, y } = getMousePosition(event);

      ctx.lineWidth = eraser ? 20 : 2;
      ctx.lineCap = 'round';
      ctx.strokeStyle = eraser ? 'white' : color;

      if (lastPosition) {
        ctx.beginPath();
        ctx.moveTo(lastPosition.x, lastPosition.y);
        ctx.lineTo(x, y);
        ctx.stroke();
        ctx.closePath();
      }

      lastPosition = { x, y };
      saveDrawingData({ x, y, color: eraser ? 'white' : color });
    }

    function getMousePosition(event) {
      const rect = canvas.getBoundingClientRect();
      return {
        x: event.clientX - rect.left,
        y: event.clientY - rect.top
      };
    }

    function setColor(newColor) {
      color = newColor;
      eraser = false;
    }

    function setEraser() {
      eraser = true;
    }

    function clearCanvas() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      db.ref('drawing').remove(); // Firebaseからも削除
    }

    // 描画データの保存
    function saveDrawingData(data) {
      db.ref('drawing').push(data);
    }

    // 描画データのリアルタイム同期
    db.ref('drawing').on('child_added', (snapshot) => {
      const data = snapshot.val();

      if (data.end) {
        ctx.beginPath();
        return;
      }

      ctx.lineWidth = 2;
      ctx.lineCap = 'round';
      ctx.strokeStyle = data.color;
      ctx.lineTo(data.x, data.y);
      ctx.stroke();
      ctx.beginPath();
      ctx.moveTo(data.x, data.y);
    });
  </script>
</body>
</html>
