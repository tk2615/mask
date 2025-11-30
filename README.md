<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>お面トラッキング by 服部平次 (修正版)</title>
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
      background-color: rgba(0,0,0,0.5);
      padding: 10px;
      border-radius: 5px;
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

  // ▼▼▼ ここが変更点や！ ▼▼▼
  // 拓郎がくれた画像をBase64データに変換して埋め込んだで。
  // これでファイル配置の心配は無用や！
  const maskImage = new Image();
  maskImage.src = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAMAAACahl6sAAADAFBMVEVHcEy8vLzDxMTFxcXGxsbHx8fIyMjJycnKysrLy8vMzMzNzc3Ozs7Pz8/Q0NDR0dHS0tLT09PU1NTV1dXW1tbX19fY2NjZ2dna2trb29vc3Nzd3d3e3t7f39/g4ODh4eHi4uLj4+Pk5OTl5eXm5ubn5+fo6Ojp6enq6urr6+vs7Ozt7e3u7u7v7+/w8PDx8fHy8vLz8/P09PT19fX29vb39/f4+Pj5+fn6+vr7+/v8/Pz9/f3+/v7////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////iE3a8AAAACXBIWXMAAAsTAAALEwEAmpwYAAAE0klEQVR4nO2ba3dTNxCFH24JTYqE5lKa5tJCaYFSoL3Qnre9//8fFx8sWV4c9YwkY2v2t9bK8YxG0q52ZEmR6/UqWZb/G3+17y7+6v9uN87/45Wn613H+e+u4+j5P+50vV7n/w3u3f91eO922v91t18X97/b/r7pP2b7q9H//Lz95fbzR+c/c/z69vHzx50u7v/x3b/j6w/b4+F45/8sP4P7y/H93b/j69vbn28u//Ea//P74912+/3x3fXl5vL/+rX/i/s8b1/f/d+Xq8uP//Fp/8ft8fT93bHjR2f79fD/H/fLzfN/vNZ/9p/Tq+e7H54sXW8O958+P582B8+X/8fr/Oer2+uP73d/tG63j57//vO0/W8vX26uf7y/v18dfn25vDrd746/Pz2fd79+1727PZ2+XN28Pz+cTqfn09Xp4cvp+fTt/vLq/P6+PT1/Od28u328Pz2+X315+vLy/uXy8eZ0dTpdnp6eH76+XF2c/ny8uTy9PZ6fv1x++Pb8+u7+5fH58nK7+eH69eZwe3q6P19dXG++nE6nL49f/9t9d3m9PV19uTvd3h8/3Z+url/v749vT2+P96fT/fnJ6d3p5ubydHt8+vJ2d/J8v7v8/u7d7f3b9eX19nC6f/f65XQ4fXl4vLw6fTm/f3m4vLu8P1xd3/06P3798vby8fJ0ubp6fHt6/rI/PT++3x2/3D1eH24fL6/fH16eLu+P928Xl+fT9eH0cPc4nC4Ot9eb98+n+5er96+/69/vLh9fD5fXt4fT5cvD3fH0/vj2cHn/9f7Xm9P98c3D5dPj6XT/eLq+fn19PD2+f/p/118+7q7f3T6+3N6e79/efj2fLg6397u32+P3X7+en++PLw+P17v79/c/3e2+Pzxefzm/e7z55fTx/unL2/W3H6+76/P989v916fXy4vT+a+/n7++PT88nL7eP7/dXT//t+v4/un0fPd8uXn8uDtcH94fL5/+5+uvH+ef3e+eP//n65/T3fH5/P2fXf//F87/7vK/3e3yF/w/3Tj/42uXv+vG+f/9043z/3vlxvkfXrtlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZlWZZl
