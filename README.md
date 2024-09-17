<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Live Camera Clothing Try-On</title>
  <style>
    body {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 20px;
      font-family: Arial, sans-serif;
    }
    #video {
      display: block;
      width: 80%;
      height: auto;
      border: 1px solid #ddd;
      background: #000;
    }
    #canvas {
      position: absolute;
      top: 0;
      left: 0;
      pointer-events: none;
    }
    input[type="file"], #clothing-style {
      margin: 10px;
    }
    .container {
      position: relative;
      width: 80%;
      height: auto;
    }
  </style>
  <!-- Add MediaPipe -->
  <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose"></script>
</head>
<body>
  <h1>Live Camera Clothing Try-On</h1>

  <div class="container">
    <video id="video" autoplay></video>
    <canvas id="canvas"></canvas>
  </div>

  <select id="clothing-style">
    <option value="clothing1.png">Style 1</option>
    <option value="clothing2.png">Style 2</option>
    <option value="clothing3.png">Style 3</option>
    <option value="clothing4.png">Style 4</option>
    <option value="clothing5.png">Style 5</option>
  </select>
  
  <input type="file" id="upload" accept="image/*">
  
  <script>
    let clothingImage = new Image();
    let clothingPosition = { x: 50, y: 50, width: 200, height: 200 };
    let isDragging = false;

    // Access the camera and display the video feed
    async function setupCamera() {
      const video = document.getElementById('video');
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;

        video.onloadedmetadata = () => {
          const canvas = document.getElementById('canvas');
          canvas.width = video.videoWidth;
          canvas.height = video.videoHeight;

          console.log('Camera initialized.');
          drawOverlay(); // Start drawing once video is ready
        };
      } catch (error) {
        console.error('Error accessing camera:', error);
        alert('Camera access denied or unavailable.');
      }
    }

    // Setup MediaPipe for body landmark detection
    function setupPoseDetection() {
      const video = document.getElementById('video');
      const canvas = document.getElementById('canvas');
      const context = canvas.getContext('2d');

      const pose = new Pose({
        locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${file}`,
      });

      pose.setOptions({
        modelComplexity: 1,
        smoothLandmarks: true,
        enableSegmentation: false,
        minDetectionConfidence: 0.5,
        minTrackingConfidence: 0.5,
      });

      pose.onResults((results) => {
        if (!results.poseLandmarks) return;

        context.clearRect(0, 0, canvas.width, canvas.height);
        context.drawImage(video, 0, 0, canvas.width, canvas.height);

        // Assuming clothing is to be drawn on the torso
        const shoulderLeft = results.poseLandmarks[11];
        const shoulderRight = results.poseLandmarks[12];

        const centerX = (shoulderLeft.x + shoulderRight.x) / 2 * canvas.width;
        const centerY = (shoulderLeft.y + shoulderRight.y) / 2 * canvas.height;
        const width = Math.abs(shoulderRight.x - shoulderLeft.x) * canvas.width * 1.5;
        const height = width * 1.5;  // Adjust proportionately for dress length

        // Draw the clothing image based on torso landmarks
        if (clothingImage.src) {
          context.drawImage(clothingImage, centerX - width / 2, centerY - height / 3, width, height);
        }
      });

      // Run pose detection continuously
      const runDetection = () => {
        pose.send({ image: video });
        requestAnimationFrame(runDetection);
      };
      runDetection();
    }

    // Handle clothing style change
    document.getElementById('clothing-style').addEventListener('change', function(event) {
      const selectedStyle = event.target.value;
      clothingImage.src = selectedStyle;
    });

    // Handle file upload for custom clothing
    document.getElementById('upload').addEventListener('change', function(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = function(e) {
          clothingImage.src = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    });

    // Start everything
    setupCamera();
    setupPoseDetection();
  </script>
</body>
</html>
