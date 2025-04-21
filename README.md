# VONGQUAY
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Vòng Quay May Mắn</title>
  <script src="https://cdn.jsdelivr.net/npm/gsap@3.12.2/dist/gsap.min.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <style>
    #wheelCanvas {
      width: 100%;
      max-width: 500px;
      height: auto;
    }
    .pointer {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -220px);
      width: 0;
      height: 0;
      border-left: 20px solid transparent;
      border-right: 20px solid transparent;
      border-bottom: 40px solid #fff;
      animation: bounce 1s infinite;
      z-index: 10;
    }
    @keyframes bounce {
      0%, 100% {
        transform: translate(-50%, -220px) translateY(0);
      }
      50% {
        transform: translate(-50%, -220px) translateY(-10px);
      }
    }
    .popup {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: white;
      padding: 2rem;
      border-radius: 1rem;
      box-shadow: 0 0 20px rgba(0,0,0,0.3);
      z-index: 50;
      text-align: center;
      display: none;
    }
    .fireworks {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      pointer-events: none;
      z-index: 40;
    }
  </style>
</head>
<body class="bg-gradient-to-br from-purple-500 to-pink-500 min-h-screen flex items-center justify-center relative overflow-hidden">
  <div class="pointer"></div>
  <div class="text-center">
    <canvas id="wheelCanvas" width="500" height="500"></canvas>
    <button id="spinBtn" class="mt-6 px-6 py-3 bg-yellow-400 hover:bg-yellow-300 text-black font-bold rounded-full shadow-lg">QUAY</button>
    <div id="result" class="mt-6 text-white text-xl font-semibold"></div>
  </div>

  <div id="popup" class="popup">
    <h2 class="text-xl font-bold mb-4">🎉 Bạn đã trúng thưởng! 🎉</h2>
    <p id="popupText" class="text-lg"></p>
    <button onclick="document.getElementById('popup').style.display='none'" class="mt-4 px-4 py-2 bg-green-500 text-white rounded">Đóng</button>
  </div>

  <canvas class="fireworks" id="fireworksCanvas"></canvas>
  <audio id="winSound" src="https://www.soundjay.com/human/sounds/applause-8.mp3"></audio>

  <script>
    const rewards = [
      { label: "Airpods Pro 2", chance: 0.01, image: "https://i.imgur.com/ZG6E4Sj.png" },
      { label: "1 THÁNG HỌC PHÍ", chance: 1, image: "https://i.imgur.com/1wYB1ft.png" },
      { label: "1.000.000", chance: 5, image:     "https://i.pinimg.com/736x/7f/79/eb/7f79eb40946e64987220158234c78df4.jpg" },
      { label: "700.000", chance: 10, image: "https://i.imgur.com/7Vmr7i8.png" },
      { label: "500.000", chance: 15, image: "https://i.imgur.com/ZBZESiP.png" },
      { label: "500.000", chance: 15, image: "https://i.imgur.com/ZBZESiP.png" },
      { label: "300.000", chance: 20, image: "https://i.imgur.com/ZBZESiP.png" },
      { label: "300.000", chance: 20, image: "https://i.imgur.com/ZBZESiP.png" },

    ];

    const weights = [5, 10, 20, 65];
    const canvas = document.getElementById("wheelCanvas");
    const ctx = canvas.getContext("2d");
    const centerX = canvas.width / 2;
    const centerY = canvas.height / 2;
    const radius = 200;
    let currentRotation = 0;

    function drawWheel() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      let sliceAngle = 2 * Math.PI / rewards.length;
      let startAngle = 0;
      rewards.forEach((reward, index) => {
        ctx.beginPath();
        ctx.moveTo(centerX, centerY);
        ctx.arc(centerX, centerY, radius, startAngle, startAngle + sliceAngle);
        ctx.fillStyle = index % 2 === 0 ? "#ffeb3b" : "#f44336";
        ctx.fill();
        ctx.stroke();

        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(startAngle + sliceAngle / 2);
        ctx.textAlign = "right";
        ctx.fillStyle = "#000";
        ctx.font = "bold 16px sans-serif";
        ctx.fillText(reward.label, radius - 10, 5);
        ctx.restore();

        startAngle += sliceAngle;
      });
    }

    drawWheel();

    function getRewardByIndex(index) {
      return rewards[index];
    }

    function pickRandomIndex() {
      const total = weights.reduce((a, b) => a + b, 0);
      const rand = Math.random() * total;
      let acc = 0;
      for (let i = 0; i < weights.length; i++) {
        acc += weights[i];
        if (rand < acc) return i;
      }
      return weights.length - 1;
    }

    function showPopup(reward) {
      document.getElementById("popupText").innerText = `Bạn trúng ${reward.label}!`;
      document.getElementById("popup").style.display = 'block';
    }

    function launchFireworks() {
      const canvas = document.getElementById("fireworksCanvas");
      const ctx = canvas.getContext("2d");
      canvas.width = window.innerWidth;
      canvas.height = window.innerHeight;

      const particles = [];
      for (let i = 0; i < 100; i++) {
        particles.push({
          x: canvas.width / 2,
          y: canvas.height / 2,
          angle: Math.random() * 2 * Math.PI,
          speed: Math.random() * 5 + 2,
          radius: Math.random() * 3 + 2,
          alpha: 1
        });
      }

      function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        particles.forEach(p => {
          ctx.beginPath();
          ctx.globalAlpha = p.alpha;
          ctx.arc(p.x, p.y, p.radius, 0, 2 * Math.PI);
          ctx.fillStyle = `hsl(${Math.random() * 360}, 100%, 50%)`;
          ctx.fill();
        });
      }

      function update() {
        particles.forEach(p => {
          p.x += Math.cos(p.angle) * p.speed;
          p.y += Math.sin(p.angle) * p.speed;
          p.alpha -= 0.01;
        });
      }

      function animate() {
        draw();
        update();
        if (particles[0].alpha > 0) {
          requestAnimationFrame(animate);
        } else {
          ctx.clearRect(0, 0, canvas.width, canvas.height);
        }
      }

      animate();
    }

    document.getElementById("spinBtn").addEventListener("click", () => {
      const selectedIndex = pickRandomIndex();
      const slice = 360 / rewards.length;
      const stopAngle = 360 * 5 + (360 - selectedIndex * slice - slice / 2);

      gsap.to(canvas, {
        rotation: currentRotation + stopAngle,
        duration: 5,
        ease: "power4.out",
        onUpdate: function () {
          canvas.style.transform = `rotate(${this.targets()[0]._gsap.rotation}deg)`;
        },
        onComplete: function () {
          currentRotation = (currentRotation + stopAngle) % 360;
          const reward = getRewardByIndex(selectedIndex);
          document.getElementById("result").innerText = `Bạn trúng: ${reward.label}!`;
          document.getElementById("winSound").play();
          showPopup(reward);
          if (reward.label !== "Không trúng") launchFireworks();
        },
      });
    });
  </script>
</body>
</html>

