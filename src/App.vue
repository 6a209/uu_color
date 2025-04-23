<template>
  <div class="container max-w-md mx-auto px-4 py-6">
    <!-- 标题栏 -->
    <div class="flex justify-between items-center mb-8">
      <button @click="showInstructionsModal = true" class="text-gray-600 hover:text-gray-800">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.228 9c.549-1.165 2.03-2 3.772-2 2.21 0 4 1.343 4 3 0 1.4-1.278 2.575-3.006 2.907-.542.104-.994.54-.994 1.093m0 3h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
      </button>
      <h1 class="text-3xl font-normal text-center">柚柚猜颜色</h1>
      <button @click="showStatsModal = true" class="text-gray-600 hover:text-gray-800">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" />
        </svg>
      </button>
    </div>

    <!-- 消息提示区 -->
    <div v-if="message.text" :class="messageClasses" class="text-center py-2 px-4 rounded-lg mb-4 font-medium">
      {{ message.text }}
    </div>

    <!-- 游戏区域 -->
    <div class="flex flex-col items-center mb-6">
      <!-- 猜测历史行 -->
      <div v-for="row in maxAttempts" :key="row" class="flex mb-2 justify-center w-full">
        <div class="flex items-center justify-center ml-12">
          <template v-if="guessHistory[row-1]">
            <!-- 已猜测的行 -->
            <div class="flex justify-center">
              <div v-for="(color, cIndex) in guessHistory[row-1].guess" :key="cIndex" 
                   :class="[colorClasses[color], 'color-cell']">
              </div>
            </div>
            <div class="ml-3 hints-grid">
              <div v-for="(hint, hIndex) in guessHistory[row-1].hints" :key="hIndex" :class="hintClasses(hint)"></div>
            </div>
          </template>
          <template v-else-if="row === attempts + 1 && !gameOver">
            <!-- 当前猜测行 -->
            <div class="flex justify-center">
              <div v-for="(color, index) in currentGuess" :key="index" 
                   :class="[color ? colorClasses[color] : 'bg-gray-200', 'color-cell']" 
                   @click="removeColorFromGuess(index)">
              </div>
            </div>
            <div class="ml-3 w-[36px]"></div>
          </template>
          <template v-else>
            <!-- 空行 -->
            <div class="flex justify-center">
              <div v-for="i in 4" :key="i" class="color-cell bg-gray-200"></div>
            </div>
            <div class="ml-3 w-[36px]"></div>
          </template>
        </div>
      </div>

      <!-- 颜色选择器 -->
      <div v-if="!gameOver" class="grid grid-cols-6 gap-2 mt-6 justify-center mx-auto">
        <div v-for="color in availableColors" :key="color" 
             :class="[colorClasses[color], 'color-option']" 
             @click="addColorToGuess(color)">
        </div>
      </div>
    </div>

    <!-- 控制按钮 -->
    <div class="flex justify-center">
      <button @click="initializeGame" class="bg-green-500 hover:bg-green-600 text-white px-6 py-2 rounded-lg">
        新游戏
      </button>
    </div>
  </div>

  <!-- 游戏说明弹窗 -->
  <div v-if="showInstructionsModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-10">
    <div class="bg-white rounded-lg max-w-md mx-4 p-6">
      <h2 class="text-xl font-bold mb-4">如何游戏</h2>
      <ul class="list-disc pl-5 mb-4 text-gray-700">
        <li>游戏开始时，系统会随机生成一个由4个不同颜色组成的隐藏组合</li>
        <li>你有 {{ maxAttempts }} 次机会猜出正确的颜色组合</li>
        <li>从底部选择颜色填充当前行的空位</li>
        <li>填满4个位置后点击"提交"按钮</li>
        <li>系统会给予提示：</li>
        <li>🟢 表示有一个颜色的位置和颜色都正确</li>
        <li>🟠 表示有一个颜色出现在组合中但位置不正确</li>
        <li>⚪ 表示这个颜色不在组合中</li>
        <li>提示点按2x2网格排列</li>
      </ul>
      <button @click="showInstructionsModal = false" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg w-full">
        知道了
      </button>
    </div>
  </div>

  <!-- 胜利弹窗 -->
  <div v-if="showWinModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-10">
    <div class="bg-white rounded-lg max-w-md mx-4 p-6 text-center">
      <h2 class="text-2xl font-bold mb-2">恭喜你赢了！</h2>
      <p class="mb-4 text-gray-700">你用了 <span class="font-bold">{{ attempts }}</span> 次尝试猜出了正确组合！</p>
      <div class="flex justify-center mb-4">
         <div v-for="(color, cIndex) in secretCombination" :key="cIndex" :class="[colorClasses[color], 'color-cell']"></div>
      </div>
      <button @click="initializeGame" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg w-full">
        再玩一次
      </button>
    </div>
  </div>

  <!-- 失败弹窗 -->
  <div v-if="showLoseModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-10">
    <div class="bg-white rounded-lg max-w-md mx-4 p-6 text-center">
      <h2 class="text-2xl font-bold mb-2">游戏结束</h2>
      <p class="mb-4 text-gray-700">很遗憾，你没有猜出正确的组合。</p>
      <p class="mb-2 text-gray-700">正确组合是：</p>
      <div class="flex justify-center mb-4">
        <div v-for="(color, cIndex) in secretCombination" :key="cIndex" :class="[colorClasses[color], 'color-cell']"></div>
      </div>
      <button @click="initializeGame" class="bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-lg w-full">
        再玩一次
      </button>
    </div>
  </div>

  <!-- 统计数据弹窗 -->
  <div v-if="showStatsModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-10">
    <div class="bg-white rounded-lg max-w-md mx-4 p-6">
      <h2 class="text-xl font-bold mb-4">游戏统计</h2>
      <div class="grid grid-cols-3 gap-4 mb-6">
        <div class="text-center">
          <p class="text-2xl font-bold">{{ stats.totalGames }}</p>
          <p class="text-gray-600">游戏总数</p>
        </div>
        <div class="text-center">
          <p class="text-2xl font-bold">{{ stats.wins }}</p>
          <p class="text-gray-600">胜利</p>
        </div>
        <div class="text-center">
          <p class="text-2xl font-bold">{{ stats.losses }}</p>
          <p class="text-gray-600">失败</p>
        </div>
      </div>
      
      <h3 class="font-bold mb-2">尝试次数分布</h3>
      <div class="mb-4">
        <div v-for="i in maxAttempts" :key="i" class="flex items-center mb-1">
          <div class="w-8 text-right mr-2">{{ i }}:</div>
          <div class="bg-green-100 h-5" :style="`width: ${calculateBarWidth(i)}%`"></div>
          <div class="ml-2">{{ stats.attemptsDistribution[i] || 0 }}</div>
        </div>
      </div>
      
      <button @click="showStatsModal = false" class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg w-full">
        关闭
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// --- 响应式状态 --- 
const secretCombination = ref([]);
const currentGuess = ref([null, null, null, null]);
const attempts = ref(0);
const maxAttempts = ref(6);
const gameOver = ref(false);
const guessHistory = ref([]); // 存储 { guess: [], hints: [] }
const message = ref({ text: '', type: '' }); // { text: string, type: 'success' | 'error' | 'info' }

const showInstructionsModal = ref(false);
const showWinModal = ref(false);
const showLoseModal = ref(false);
const showStatsModal = ref(false);

const stats = ref({
  totalGames: 0,
  wins: 0,
  losses: 0,
  attemptsDistribution: {} // 记录每次尝试次数的胜利分布 {1: 次数, 2: 次数...}
});

const availableColors = ref(['red', 'blue', 'green', 'yellow', 'purple', 'pink']);
const colorClasses = ref({
  'red': 'bg-red-500',
  'blue': 'bg-blue-500',
  'green': 'bg-green-500',
  'yellow': 'bg-yellow-400',
  'purple': 'bg-purple-500',
  'pink': 'bg-pink-500'
});

// --- 计算属性 --- 
const isGuessComplete = computed(() => {
  return currentGuess.value.every(color => color !== null);
});

const messageClasses = computed(() => {
  switch (message.value.type) {
    case 'success': return 'bg-green-100 text-green-700';
    case 'error': return 'bg-red-100 text-red-700';
    case 'info': return 'bg-yellow-100 text-yellow-700';
    default: return '';
  }
});

// --- 方法 --- 
function showFlashMessage(text, type) {
  message.value = { text, type };
  setTimeout(() => {
    message.value = { text: '', type: '' };
  }, 3000);
}

function generateSecretCombination() {
  const colors = [...availableColors.value];
  const combination = [];
  for (let i = 0; i < 4; i++) {
    const randomIndex = Math.floor(Math.random() * colors.length);
    combination.push(colors[randomIndex]);
    colors.splice(randomIndex, 1);
  }
  return combination;
}

function checkGuess(guess) {
  const secret = [...secretCombination.value];
  const guessCopy = [...guess];
  const hints = [];

  // 检查 correct (green)
  for (let i = 0; i < 4; i++) {
    if (guessCopy[i] === secret[i]) {
      hints.push('correct');
      guessCopy[i] = null; // Mark as checked
      secret[i] = null;    // Mark as checked
    }
  }

  // 检查 misplaced (orange) 和 absent (gray)
  for (let i = 0; i < 4; i++) {
    if (guessCopy[i] !== null) { // Only check non-correct slots
      const colorIndex = secret.indexOf(guessCopy[i]);
      if (colorIndex !== -1) {
        hints.push('misplaced');
        secret[colorIndex] = null; // Mark as checked to avoid double hints
      } else {
        hints.push('absent');
      }
    }
  }

  // Pad with absent hints if fewer than 4 were generated (handles correct guesses)
  while (hints.length < 4) {
    hints.push('absent');
  }
  
  // 按照优先级排序：correct > misplaced > absent
  return hints.sort((a, b) => {
    const priority = { 'correct': 0, 'misplaced': 1, 'absent': 2 };
    return priority[a] - priority[b];
  });
}

function addColorToGuess(color) {
  const emptyIndex = currentGuess.value.indexOf(null);
  if (emptyIndex !== -1) {
    // Check if color already exists in guess
    if (currentGuess.value.includes(color)) {
       showFlashMessage('不能使用重复的颜色', 'error');
       return;
    }
    currentGuess.value[emptyIndex] = color;

    // 使用标准Vue方式添加动画效果
    // 在真实项目中，可能会使用Vue Transition或CSS类绑定来实现动画
    const cells = document.querySelectorAll('.color-cell');
    const currentRow = attempts.value;
    const targetIndex = currentRow * 4 + emptyIndex;
    
    if (cells[targetIndex]) {
      cells[targetIndex].classList.add('animate-bounce-custom');
      setTimeout(() => {
        cells[targetIndex].classList.remove('animate-bounce-custom');
      }, 500);
    }
    
    // 如果这是最后一个颜色，自动提交
    if (emptyIndex === 3) {
      setTimeout(() => {
        submitGuess();
      }, 600); // 延迟一点提交，让动画先完成
    }
  } else {
    showFlashMessage('已选择4个颜色', 'info');
  }
}

function removeColorFromGuess(index) {
  if (currentGuess.value[index] !== null) {
    currentGuess.value[index] = null;
  }
}

function submitGuess() {
  if (!isGuessComplete.value) {
    showFlashMessage('请填满所有4个颜色格子', 'error');
    return;
  }

  // Double check for duplicates (should be prevented by addColorToGuess, but good safeguard)
  const uniqueColors = new Set(currentGuess.value);
  if (uniqueColors.size !== 4) {
    showFlashMessage('不能使用重复的颜色', 'error');
    return;
  }

  attempts.value++;
  const currentGuessSubmitted = [...currentGuess.value]; // Copy before resetting
  const hints = checkGuess(currentGuessSubmitted);

  guessHistory.value.push({ guess: currentGuessSubmitted, hints: hints });

  // Check win condition
  if (hints.filter(h => h === 'correct').length === 4) {
    gameOver.value = true;
    updateWinStats();
    showWinModal.value = true;
    return;
  }

  // Check lose condition
  if (attempts.value >= maxAttempts.value) {
    gameOver.value = true;
    updateLossStats();
    showLoseModal.value = true;
    return;
  }

  // Reset for next guess
  currentGuess.value = [null, null, null, null];
}

function hintClasses(hint) {
  switch (hint) {
    case 'correct': return 'hint-dot bg-green-500';
    case 'misplaced': return 'hint-dot bg-orange-500';
    case 'absent': return 'hint-dot bg-gray-300';
    default: return 'hint-dot';
  }
}

function initializeGame() {
  secretCombination.value = generateSecretCombination();
  currentGuess.value = [null, null, null, null];
  attempts.value = 0;
  gameOver.value = false;
  guessHistory.value = [];
  message.value = { text: '', type: '' };
  showWinModal.value = false;
  showLoseModal.value = false;
  showInstructionsModal.value = false; // Close instructions if open
  console.log("Secret combination (for debugging):", secretCombination.value);
}

function calculateBarWidth(attemptNumber) {
  if (!stats.value.attemptsDistribution[attemptNumber]) {
    return 0;
  }
  
  // 找出最大值
  const maxValue = Math.max(...Object.values(stats.value.attemptsDistribution));
  if (maxValue === 0) return 0;
  
  // 计算当前值占最大值的百分比（最大75%的宽度）
  return (stats.value.attemptsDistribution[attemptNumber] / maxValue) * 75;
}

function loadStats() {
  const savedStats = localStorage.getItem('colorGameStats');
  if (savedStats) {
    stats.value = JSON.parse(savedStats);
  }
}

function saveStats() {
  localStorage.setItem('colorGameStats', JSON.stringify(stats.value));
}

function updateWinStats() {
  stats.value.totalGames++;
  stats.value.wins++;
  
  // 更新尝试次数分布
  if (!stats.value.attemptsDistribution[attempts.value]) {
    stats.value.attemptsDistribution[attempts.value] = 0;
  }
  stats.value.attemptsDistribution[attempts.value]++;
  
  saveStats();
}

function updateLossStats() {
  stats.value.totalGames++;
  stats.value.losses++;
  saveStats();
}

// 初始化游戏
onMounted(() => {
  loadStats();
  initializeGame();
});
</script> 