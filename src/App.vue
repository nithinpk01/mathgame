<template>
  <div class="game-container">
    <div v-if="!gameStarted" class="start-screen">
      <h1>Bubble Pop Quiz</h1>
      <button @click="startGame">Start Game</button>
    </div>

    <div v-else-if="gameOver" class="end-screen">
      <h1>⛔ Game Over!</h1>
      <p>You missed the answer!</p>
      <button @click="startGame">Try Again</button>
    </div>

    <div v-else class="game-content">
      <h2 class="question">What is {{ question.text }}?</h2>
      <div class="bubble-area">
        <div v-for="(bubble, index) in bubbles" :key="bubble.id" class="bubble"
          :style="{ top: bubble.top + 'vh', left: bubble.left + 'vw' }" @click="checkAnswer(index)">
          {{ bubble.value }}
        </div>
        <div v-if="popped" class="pop-effect">💥</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const gameStarted = ref(false);
const gameOver = ref(false);
const question = ref({ text: '', answer: 0 });
const bubbles = ref([]);
const popped = ref(false);
let bubbleId = 0;
let interval;

const generateQuestion = () => {
  const operators = ['+', '-', '*', '/'];
  const op = operators[Math.floor(Math.random() * operators.length)];

  let a = Math.floor(Math.random() * 10 + 1);
  let b = Math.floor(Math.random() * 10 + 1);
  let result;

  switch (op) {
    case '+':
      result = a + b;
      break;
    case '-':
      // Ensure non-negative results
      if (a < b) [a, b] = [b, a];
      result = a - b;
      break;
    case '*':
      result = a * b;
      break;
    case '/':
      result = a; // ensure clean division
      result = a;
      b = Math.floor(Math.random() * 9) + 1;
      a = result * b;
      result = a / b;
      break;
  }

  question.value = {
    text: `${a} ${op} ${b}`,
    answer: result,
  };
};

const createBubble = () => {
  const wrongAnswers = [
    question.value.answer + 1,
    question.value.answer - 1,
    question.value.answer + 2,
  ];
  const options = [...wrongAnswers, question.value.answer].sort(() => 0.5 - Math.random());

  bubbles.value = []; // Clear existing before placing new ones

  options.forEach((val) => {
    let newBubble;
    let attempts = 0;
    do {
      newBubble = {
        id: bubbleId++,
        value: val,
        top: 100,
        left: Math.random() * 80 + 5,
      };
      attempts++;
    } while (isOverlapping(newBubble, bubbles.value) && attempts < 20); // avoid infinite loop

    bubbles.value.push(newBubble);
  });
};
const isOverlapping = (newBubble, existingBubbles) => {
  return existingBubbles.some(bubble => {
    const dx = newBubble.left - bubble.left;
    const dy = newBubble.top - bubble.top;
    const distance = Math.sqrt(dx * dx + dy * dy);
    return distance < 15; // adjust based on bubble size
  });
};


const checkAnswer = (index) => {
  const bubble = bubbles.value[index];
  if (bubble.value === question.value.answer) {
    popped.value = true;
    setTimeout(() => (popped.value = false), 500);
    bubbles.value = [];
    generateQuestion();
    createBubble();
  } else {
    gameOver.value = true;
    stopGame();
  }
};

const startGame = () => {
  gameStarted.value = true;
  gameOver.value = false;
  bubbles.value = [];
  bubbleId = 0;
  generateQuestion();
  createBubble();

  if (interval) clearInterval(interval);
  interval = setInterval(() => {
    bubbles.value.forEach((b) => {
      b.top -= 1.5; // Faster movement
    });

    // If correct bubble missed
    const correctMissed = bubbles.value.every((b) => b.top <= 0 || b.value !== question.value.answer);
    if (correctMissed && bubbles.value.length > 0) {
      gameOver.value = true;
      stopGame();
    }

    bubbles.value = bubbles.value.filter((b) => b.top > -10);
  }, 100);
};

const stopGame = () => {
  clearInterval(interval);
  bubbles.value = [];
};
</script>

<style scoped>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: 'Comic Sans MS', cursive, sans-serif;
}

.game-container {
  width: 100%;
  height: 100vh;
  background: linear-gradient(to top, #a6e6ff, #ffffff);
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
}

.start-screen,
.end-screen {
  text-align: center;
}

.start-screen h1,
.end-screen h1 {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.start-screen button,
.end-screen button {
  font-size: 1.5rem;
  padding: 0.7rem 2rem;
  background-color: #00bcd4;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background 0.3s;
}

.start-screen button:hover,
.end-screen button:hover {
  background-color: #0097a7;
}

.game-content {
  position: relative;
  width: 100%;
  height: 100vh;
  padding-top: 3vh;
  text-align: center;
}

.question {
  font-size: 6vw;
  color: #333;
}

.bubble-area {
  position: relative;
  width: 100%;
  height: 100%;
}

.bubble {
  position: absolute;
  width: 12vw;
  height: 12vw;
  max-width: 70px;
  max-height: 70px;
  background: #55cdfc;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 4vw;
  max-font-size: 22px;
  color: white;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  transition: transform 0.2s ease;
  user-select: none;
}

.bubble:hover {
  transform: scale(1.15);
}

.pop-effect {
  position: absolute;
  top: 50%;
  left: 50%;
  font-size: 3rem;
  transform: translate(-50%, -50%);
  animation: pop 0.4s ease-out;
  pointer-events: none;
}

@keyframes pop {
  0% {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }

  100% {
    opacity: 0;
    transform: translate(-50%, -50%) scale(2);
  }
}

@media (min-width: 768px) {
  .question {
    font-size: 2rem;
  }

  .bubble {
    font-size: 1.2rem;
  }
}
</style>
