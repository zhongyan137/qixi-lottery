# qixi-lottery
七夕抽奖网页
[1.html](https://github.com/user-attachments/files/21965951/1.html)
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>七夕限定 | 心跳抽奖池</title>
  <!-- 引入 Tailwind CSS（CDN 版，方便直接使用） -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- 字体图标（可选，用于装饰） -->
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bootstrap-icons@1.11.2/font/bootstrap-icons.min.css"
  />
  <style>
    /* 自定义动画：卡片翻转 */
    .card-flip {
      perspective: 1200px;
    }
    .card-inner {
      transition: transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
      transform-style: preserve-3d;
    }
    .card-front,
    .card-back {
      backface-visibility: hidden;
    }
    .card-back {
      transform: rotateY(180deg);
    }
    .flipped .card-inner {
      transform: rotateY(180deg);
    }

    /* 浪漫粒子动画（简化版，适配移动端） */
    .particle {
      position: absolute;
      width: 6px;
      height: 6px;
      background: #ff69b4;
      border-radius: 50%;
      opacity: 0.8;
      pointer-events: none;
      animation: float 5s infinite linear;
    }
    @keyframes float {
      0% {
        transform: translateY(0) rotate(0deg);
      }
      50% {
        transform: translateY(-60px) rotate(180deg);
      }
      100% {
        transform: translateY(-120px) rotate(360deg);
        opacity: 0;
      }
    }

    /* 适配手机端点击区域 */
    .btn {
      touch-action: manipulation;
    }
  </style>
</head>
<body class="bg-gradient-to-b from-pink-50 to-pink-100 min-h-screen relative">
  <!-- 浪漫粒子背景（动态生成） -->
  <div id="particle-container" class="absolute inset-0 z-0"></div>

  <!-- 页面主体 -->
  <div class="container mx-auto px-4 py-8 relative z-1">
    <!-- 标题区 -->
    <header class="text-center mb-8">
      <h1 class="text-3xl md:text-4xl font-bold text-pink-600">
        🎀 七夕限定卡池 · 心跳开启 🎀
      </h1>
      <p class="mt-2 text-gray-700">
        发送「咱俩试试？」密语，抽取专属缘分<br />
        截止日期：2025年8月29日 💕
      </p>
    </header>

    <!-- 抽奖控制区 -->
    <div class="flex flex-col md:flex-row justify-center items-center gap-4 mb-8">
      <button
        id="btn-single"
        class="btn bg-pink-500 hover:bg-pink-600 text-white font-bold py-2 px-6 rounded shadow"
      >
        <i class="bi bi-gift-fill mr-2"></i> 单抽一次
      </button>
      <button
        id="btn-ten"
        class="btn bg-orange-400 hover:bg-orange-500 text-white font-bold py-2 px-6 rounded shadow"
      >
        <i class="bi bi-star-fill mr-2"></i> 10连抽（必出特殊奖励）
      </button>
    </div>

    <!-- 抽奖结果区 -->
    <div class="flex flex-col items-center">
      <!-- 单抽卡片 -->
      <div
        id="single-card"
        class="card-flip w-64 md:w-80 h-64 md:h-80 bg-white rounded-lg shadow-lg mb-6 hidden"
      >
        <div class="card-inner h-full">
          <!-- 卡片正面（未抽状态） -->
          <div
            class="card-front h-full flex flex-col items-center justify-center p-4"
          >
            <i class="bi bi-heart fs-1 text-pink-500 animate-bounce mb-4"></i>
            <h3 class="text-xl font-bold text-gray-800">点击开始邂逅</h3>
            <button
              id="btn-draw-single"
              class="mt-4 bg-pink-500 hover:bg-pink-600 text-white py-2 px-4 rounded"
            >
              立即抽奖
            </button>
          </div>
          <!-- 卡片背面（结果展示） -->
          <div
            class="card-back h-full flex flex-col items-center justify-center p-4 bg-pink-50"
          >
            <div id="single-result-icon" class="text-4xl mb-4"></div>
            <h3 id="single-result-text" class="text-xl font-bold mb-2"></h3>
            <p id="single-result-rarity" class="text-sm text-gray-500"></p>
            <button
              id="btn-reset-single"
              class="mt-4 bg-white hover:bg-gray-50 text-pink-500 py-2 px-4 rounded border"
            >
              再抽一次
            </button>
          </div>
        </div>
      </div>

      <!-- 10连抽结果区 -->
      <div
        id="ten-result-container"
        class="grid grid-cols-2 md:grid-cols-5 gap-2 mb-6 hidden"
      >
        <!-- 动态生成10连抽卡片 -->
      </div>
      <button
        id="btn-reset-ten"
        class="bg-white hover:bg-gray-50 text-orange-400 py-2 px-4 rounded border hidden"
      >
        再来10连
      </button>
    </div>

    <!-- 概率公示 & 规则说明 -->
    <div
      class="bg-white rounded-lg shadow p-4 mb-8"
    >
      <h2 class="text-xl font-bold text-pink-600 mb-2">概率公示</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-2 text-sm">
        <!-- 动态插入概率项 -->
      </div>
      <p class="mt-4 text-sm text-gray-600">
        📌 抽满100次未中「好呀宝宝」？别慌！<br />
        联系页面底部客服，领取「坚持浪漫奖」安慰礼包～
      </p>
    </div>
  </div>

  <!-- 脚本逻辑 -->
  <script>
    // 抽奖配置（概率、奖励内容）
    const rewards = [
      {
        name: "滚你妈的",
        prob: 10,
        icon: "🖕",
        rarity: "普通",
      },
      {
        name: "去你妈的",
        prob: 10,
        icon: "🤬",
        rarity: "普通",
      },
      {
        name: "你也配？",
        prob: 10,
        icon: "👎",
        rarity: "普通",
      },
      {
        name: "傻逼二次元",
        prob: 10,
        icon: "🤢",
        rarity: "普通",
      },
      {
        name: "癔症又犯了？",
        prob: 10,
        icon: "🤪",
        rarity: "普通",
      },
      {
        name: "死男同真恶心",
        prob: 5,
        icon: "🤮",
        rarity: "稀有",
      },
      {
        name: "现在是幻想时刻",
        prob: 5,
        icon: "💭",
        rarity: "稀有",
      },
      {
        name: "柚子厨滚出去",
        prob: 5,
        icon: "🚪",
        rarity: "稀有",
      },
      {
        name: "这是最后通牒",
        prob: 5,
        icon: "📜",
        rarity: "稀有",
      },
      {
        name: "以后不要再和我扯上关系",
        prob: 5,
        icon: "🚫",
        rarity: "稀有",
      },
      {
        name: "被拉黑+屏蔽",
        prob: 5,
        icon: "🔒",
        rarity: "稀有",
      },
      {
        name: "小男娘",
        prob: 5,
        icon: "👦",
        rarity: "稀有",
      },
      {
        name: "随机的网图",
        prob: 2.5,
        icon: "🖼️",
        rarity: "特殊",
      },
      {
        name: "随机的贴吧圣经",
        prob: 2.5,
        icon: "📖",
        rarity: "特殊",
      },
      {
        name: "下次再说吧",
        prob: 2.5,
        icon: "⌛",
        rarity: "特殊",
      },
      {
        name: "你是个好人",
        prob: 2.5,
        icon: "👍",
        rarity: "特殊",
      },
      {
        name: "我一直把你当好朋友",
        prob: 2.5,
        icon: "👫",
        rarity: "特殊",
      },
      {
        name: "滚出去",
        prob: 2.49,
        icon: "🚶",
        rarity: "普通",
      },
      {
        name: "好呀宝宝",
        prob: 0.01,
        icon: "💖",
        rarity: "SSR",
      },
      {
        name: "其实我也喜欢你好久了",
        prob: 0.001,
        icon: "💘",
        rarity: "隐藏款",
      },
    ];

    // DOM 元素缓存
    const $particleContainer = document.getElementById("particle-container");
    const $singleCard = document.getElementById("single-card");
    const $btnDrawSingle = document.getElementById("btn-draw-single");
    const $btnResetSingle = document.getElementById("btn-reset-single");
    const $singleResultIcon = document.getElementById("single-result-icon");
    const $singleResultText = document.getElementById("single-result-text");
    const $singleResultRarity = document.getElementById("single-result-rarity");
    const $tenResultContainer = document.getElementById("ten-result-container");
    const $btnResetTen = document.getElementById("btn-reset-ten");
    const $probContainer = document.querySelector(".grid");
    const $btnSingle = document.getElementById("btn-single");
    const $btnTen = document.getElementById("btn-ten");

    // 状态管理
    let drawCount = 0; // 累计抽奖次数
    let hasFinalReward = false; // 是否已获得 SSR 或隐藏款

    // 初始化：生成粒子、概率列表
    function init() {
      createParticles();
      renderProbList();
    }

    // 生成浪漫粒子（适配手机性能）
    function createParticles() {
      for (let i = 0; i < 30; i++) {
        const particle = document.createElement("div");
        particle.className = "particle";
        particle.style.left = Math.random() * 100 + "%";
        particle.style.animationDelay = Math.random() * 3 + "s";
        $particleContainer.appendChild(particle);
      }
    }

    // 渲染概率列表
    function renderProbList() {
      rewards.forEach((item) => {
        const div = document.createElement("div");
        div.className = "flex justify-between items-center p-2 hover:bg-pink-50";
        div.innerHTML = `
          <div class="flex items-center gap-2">
            <span>${item.icon}</span>
            <span>${item.name}</span>
          </div>
          <span class="text-gray-500">${item.prob}%</span>
        `;
        $probContainer.appendChild(div);
      });
    }

    // 核心：抽奖逻辑（通用）
    function draw() {
      const totalProb = rewards.reduce((sum, item) => sum + item.prob, 0);
      const random = Math.random() * totalProb;
      let cumulative = 0;

      for (const item of rewards) {
        cumulative += item.prob;
        if (random <= cumulative) {
          // 标记特殊奖励
          if (item.rarity === "SSR" || item.rarity === "隐藏款") {
            hasFinalReward = true;
          }
          return item;
        }
      }
      return rewards[0]; // 兜底
    }

    // 单抽流程
    function handleSingleDraw() {
      const result = draw();
      showSingleResult(result);
      updateDrawCount(1);
    }

    // 显示单抽结果
    function showSingleResult(result) {
      $singleCard.classList.remove("hidden");
      $singleCard.classList.add("flipped");
      $singleResultIcon.textContent = result.icon;
      $singleResultText.textContent = result.name;
      $singleResultRarity.textContent = `稀有度：${result.rarity}`;
    }

    // 10连抽流程
    function handleTenDraw() {
      $tenResultContainer.innerHTML = "";
      $tenResultContainer.classList.remove("hidden");
      $btnResetTen.classList.remove("hidden");

      let hasSpecial = false;
      for (let i = 0; i < 10; i++) {
        const result = draw();
        renderTenResultCard(result);
        if (result.rarity === "特殊" || result.rarity === "SSR") {
          hasSpecial = true;
        }
      }

      // 确保10连必出特殊奖励（补逻辑）
      if (!hasSpecial) {
        const special = rewards.find((item) => item.rarity === "特殊");
        renderTenResultCard(special);
      }

      updateDrawCount(10);
    }

    // 渲染10连抽卡片
    function renderTenResultCard(result) {
      const card = document.createElement("div");
      card.className =
        "bg-white rounded-lg shadow flex flex-col items-center justify-center p-2";
      card.innerHTML = `
        <span class="text-2xl mb-1">${result.icon}</span>
        <p class="text-xs truncate">${result.name}</p>
        <span class="text-xs text-gray-500">${result.rarity}</span>
      `;
      $tenResultContainer.appendChild(card);
    }

    // 更新抽奖次数 & 百抽安慰检测
    function updateDrawCount(amount) {
      drawCount += amount;
      if (drawCount >= 100 && !hasFinalReward) {
        alert("百抽未中？别难过～联系客服领安慰礼包！");
      }
    }

    // 绑定事件
    $btnSingle.addEventListener("click", () => {
      $singleCard.classList.remove("hidden");
      $singleCard.classList.remove("flipped");
      $tenResultContainer.classList.add("hidden");
      $btnResetTen.classList.add("hidden");
    });

    $btnTen.addEventListener("click", handleTenDraw);
    $btnDrawSingle.addEventListener("click", handleSingleDraw);
    $btnResetSingle.addEventListener("click", () => {
      $singleCard.classList.remove("flipped");
    });
    $btnResetTen.addEventListener("click", handleTenDraw);

    // 初始化执行
    init();
  </script>
</body>
</html>
