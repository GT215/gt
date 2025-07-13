<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>語音跳動的立方體</title>
  <style>
    body { margin: 0; overflow: hidden; background: #000; }
    canvas { display: block; }
    #info {
      position: absolute;
      top: 10px;
      left: 10px;
      color: white;
      font-family: sans-serif;
    }
  </style>
</head>
<body>
  <div id="info">🎤 請對著麥克風說話，立方體會跳動！</div>
  <script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
  <script>
    let scene, camera, renderer, cube;

    async function init() {
      scene = new THREE.Scene();

      camera = new THREE.PerspectiveCamera(
        75, window.innerWidth / window.innerHeight, 0.1, 1000
      );
      camera.position.z = 5;

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      document.body.appendChild(renderer.domElement);

      const geometry = new THREE.BoxGeometry();
      const material = new THREE.MeshPhongMaterial({ color: 0xadd8e6 });
      cube = new THREE.Mesh(geometry, material);
      scene.add(cube);

      const light = new THREE.PointLight(0xffffff, 1, 100);
      light.position.set(10, 10, 10);
      scene.add(light);

      await setupMicrophone();
      animate();
    }

    async function setupMicrophone() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      const audioContext = new AudioContext();
      const source = audioContext.createMediaStreamSource(stream);
      const analyser = audioContext.createAnalyser();
      source.connect(analyser);

      analyser.fftSize = 256;
      const dataArray = new Uint8Array(analyser.frequencyBinCount);

      function updateFromAudio() {
        analyser.getByteFrequencyData(dataArray);
        const avg = dataArray.reduce((a, b) => a + b, 0) / dataArray.length;

        const scale = 1 + avg / 150;
        cube.scale.set(scale, scale, scale);

        requestAnimationFrame(updateFromAudio);
      }

      updateFromAudio();
    }

    function animate() {
      requestAnimationFrame(animate);
      renderer.render(scene, camera);
    }

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    init();
  </script>
</body>
</html>
