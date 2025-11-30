<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>お面トラッキング by 服部平次</title>
  <style>
    body {
      margin: 0;
      background-color: #222;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden;
      font-family: sans-serif;
    }
    .container {
      position: relative;
      width: 640px;
      height: 480px;
    }
    video {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: scaleX(-1); /* 鏡みたいに反転 */
    }
    canvas {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      transform: scaleX(-1); /* キャンバスも反転 */
    }
    .loading {
      position: absolute;
      color: white;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      font-size: 20px;
      z-index: 10;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>
</head>
<body>

<div class="container">
  <div class="loading" id="loadingMsg">読み込み中や...ちょっと待ち！</div>
  <video id="input_video" autoplay playsinline></video>
  <canvas id="output_canvas" width="640" height="480"></canvas>
</div>

<script>
  const videoElement = document.getElementById('input_video');
  const canvasElement = document.getElementById('output_canvas');
  const canvasCtx = canvasElement.getContext('2d');
  const loadingMsg = document.getElementById('loadingMsg');

  // お面の画像を読み込むで！
  // 自分の好きな画像URLに変えてな。とりあえずフリー素材の例を入れとくわ。
  const maskImage = new Image();
  maskImage.src = 'https://cdn-icons-png.flaticon.com/512/1077/1077114.png'; // 仮のマスク画像

  function onResults(results) {
    // 読み込み完了したら文字を消す
    loadingMsg.style.display = 'none';

    // キャンバスをクリアして、カメラ映像を描画
    canvasCtx.save();
    canvasCtx.clearRect(0, 0, canvasElement.width, canvasElement.height);
    canvasCtx.drawImage(results.image, 0, 0, canvasElement.width, canvasElement.height);

    if (results.multiFaceLandmarks) {
      for (const landmarks of results.multiFaceLandmarks) {
        // ここがミソや！
        // 鼻の頭（インデックス1）とおでこの中心（インデックス10）とかを使って位置を決めるで。
        
        const nose = landmarks[1]; // 鼻の頭
        const leftTemple = landmarks[234]; // 左こめかみ付近
        const rightTemple = landmarks[454]; // 右こめかみ付近

        // 座標をキャンバスサイズに合わせる
        const x = nose.x * canvasElement.width;
        const y = nose.y * canvasElement.height;

        // 顔の幅を計算して、お面のサイズを決める
        // こめかみの距離を使うと、顔が近づいた時にマスクも大きくなるで！
        const faceWidth = Math.hypot(
          (rightTemple.x - leftTemple.x) * canvasElement.width,
          (rightTemple.y - leftTemple.y) * canvasElement.height
        );

        const size = faceWidth * 2.5; // 倍率は画像の余白に合わせて調整や

        // お面を描画（中心を鼻の位置に合わせる）
        canvasCtx.drawImage(
          maskImage,
          x - size / 2, // 画像の左上のX座標
          y - size / 2, // 画像の左上のY座標
          size,         // 幅
          size          // 高さ
        );
      }
    }
    canvasCtx.restore();
  }

  // MediaPipe FaceMeshの設定や
  const faceMesh = new FaceMesh({locateFile: (file) => {
    return `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`;
  }});

  faceMesh.setOptions({
    maxNumFaces: 1, // 一人だけ追跡
    refineLandmarks: true,
    minDetectionConfidence: 0.5,
    minTrackingConfidence: 0.5
  });

  faceMesh.onResults(onResults);

  // カメラを起動させるで
  const camera = new Camera(videoElement, {
    onFrame: async () => {
      await faceMesh.send({image: videoElement});
    },
    width: 640,
    height: 480
  });
  camera.start();
</script>

</body>
</html>
