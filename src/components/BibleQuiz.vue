<template>
	<div class="quiz-container">
		<!-- Question Section (Top 50%) -->
		<div class="question-section">
			<div class="header">
				<h1>聖經問答挑戰</h1>
				<div v-if="!gameOver" class="timer">
					<div
						class="timer-circle"
						:style="{
							background: `conic-gradient(#ffd700 ${timerPercentage}%, transparent 0%)`,
						}"
					>
						<span>{{ timer }}</span>
					</div>
				</div>
			</div>

			<div v-if="!gameOver" class="question-container">
				<div class="question-number">
					問題 {{ currentQuestionIndex + 1 }}/{{ questions.length }}
				</div>
				<div class="question">{{ currentQuestion.question }}</div>
			</div>

			<div v-if="gameOver" class="game-over">
				<h2>遊戲結束!</h2>
				<p>您答對了 {{ correctAnswers }} 題，共 {{ questions.length }} 題</p>
				<button @click="restartGame" class="restart-btn">再玩一次</button>
			</div>
		</div>

		<!-- Options Section (Bottom 50%) -->
		<div v-if="!gameOver" class="options-section">
			<button
				v-for="(option, index) in currentQuestion.options"
				:key="index"
				:class="[
					'option-btn',
					`option-${index}`,
					{
						selected: selectedOption === index,
						correct: showAnswer && index === currentQuestion.correctIndex,
						incorrect:
							showAnswer &&
							selectedOption === index &&
							selectedOption !== currentQuestion.correctIndex,
					},
				]"
				:disabled="showAnswer"
				@click="selectOption(index)"
			>
				<span class="option-label">{{ ['A', 'B', 'C', 'D'][index] }}</span>
				<span class="option-text">{{ option }}</span>
			</button>
		</div>
	</div>
</template>

<script setup>
	import { ref, computed, onMounted, onUnmounted, watch } from 'vue';

	// 聖經問題資料
	const questions = [
		{
			question: '誰建造了方舟？',
			options: ['亞伯拉罕', '挪亞', '摩西', '大衛'],
			correctIndex: 1,
		},
		{
			question: '耶穌出生在哪個城市？',
			options: ['拿撒勒', '耶路撒冷', '伯利恆', '迦百農'],
			correctIndex: 2,
		},
		{
			question: '誰殺死了巨人歌利亞？',
			options: ['大衛', '所羅門', '掃羅', '約拿單'],
			correctIndex: 0,
		},
		{
			question: '耶穌有幾個門徒？',
			options: ['10', '12', '7', '14'],
			correctIndex: 1,
		},
		{
			question: '摩西在哪座山上領受十誡？',
			options: ['西奈山', '橄欖山', '迦密山', '黑門山'],
			correctIndex: 0,
		},
		{
			question: '誰被上帝吩咐獻上自己的兒子為祭？',
			options: ['雅各', '亞伯拉罕', '以撒', '約瑟'],
			correctIndex: 1,
		},
		{
			question: '耶穌行的第一個神蹟是什麼？',
			options: ['使瞎子看見', '使死人復活', '把水變成酒', '平靜風浪'],
			correctIndex: 2,
		},
		{
			question: '誰三次不認主耶穌？',
			options: ['約翰', '彼得', '馬太', '多馬'],
			correctIndex: 1,
		},
		{
			question: '聖經的第一卷書是？',
			options: ['出埃及記', '創世記', '利未記', '約翰福音'],
			correctIndex: 1,
		},
		{
			question: '誰被上帝從死裡復活，他已經死了四天？',
			options: ['拉撒路', '雅各', '以利亞', '施洗約翰'],
			correctIndex: 0,
		},
	];

	// 遊戲狀態
	const currentQuestionIndex = ref(0);
	const selectedOption = ref(null);
	const showAnswer = ref(false);
	const gameOver = ref(false);
	const correctAnswers = ref(0);
	const timer = ref(10);
	const timerInterval = ref(null);

	// 計算屬性
	const currentQuestion = computed(() => questions[currentQuestionIndex.value]);
	const isLastQuestion = computed(
		() => currentQuestionIndex.value === questions.length - 1
	);
	const timerPercentage = computed(() => (timer.value / 10) * 100);

	// 方法
	const selectOption = (index) => {
		if (!showAnswer.value) {
			selectedOption.value = index;
			clearInterval(timerInterval.value);
			checkAnswer();
		}
	};

	const checkAnswer = () => {
		showAnswer.value = true;

		if (selectedOption.value === currentQuestion.value.correctIndex) {
			correctAnswers.value++;
		}

		// 2秒後進入下一題
		setTimeout(() => {
			if (isLastQuestion.value) {
				gameOver.value = true;
			} else {
				currentQuestionIndex.value++;
				selectedOption.value = null;
				showAnswer.value = false;
				startTimer();
			}
		}, 2000);
	};

	const startTimer = () => {
		timer.value = 10;
		clearInterval(timerInterval.value);

		timerInterval.value = setInterval(() => {
			timer.value--;

			if (timer.value <= 0) {
				clearInterval(timerInterval.value);
				// 時間到，自動檢查答案（如果沒選，視為答錯）
				if (selectedOption.value === null) {
					selectedOption.value = -1; // 表示沒選
				}
				checkAnswer();
			}
		}, 1000);
	};

	const restartGame = () => {
		currentQuestionIndex.value = 0;
		selectedOption.value = null;
		showAnswer.value = false;
		gameOver.value = false;
		correctAnswers.value = 0;
		startTimer();
	};

	// 生命週期鉤子
	onMounted(() => {
		startTimer();
	});

	onUnmounted(() => {
		clearInterval(timerInterval.value);
	});

	// 監聽器
	watch(gameOver, (newValue) => {
		if (newValue) {
			clearInterval(timerInterval.value);
		}
	});
</script>

<style scoped>
	.quiz-container {
		display: flex;
		flex-direction: column;
		height: 100vh;
		font-family: Arial, sans-serif;
		color: white;
		background-color: #0d2b45;
		overflow: hidden;
	}

	/* Question Section - Top 50% */
	.question-section {
		height: 50vh;
		display: flex;
		flex-direction: column;
		padding: 20px;
		background-color: #1a4a74;
	}

	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 20px;
	}

	h1 {
		color: #ffd700;
		font-size: 2.5rem;
		text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
		margin: 0;
	}

	.timer {
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.timer-circle {
		width: 60px;
		height: 60px;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		font-size: 1.5rem;
		font-weight: bold;
		color: #ffd700;
		position: relative;
	}

	.timer-circle::before {
		content: '';
		position: absolute;
		width: 54px;
		height: 54px;
		border-radius: 50%;
		background-color: #0d2b45;
		z-index: 1;
	}

	.timer-circle span {
		position: relative;
		z-index: 2;
	}

	.question-container {
		flex-grow: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
	}

	.question-number {
		font-size: 1.2rem;
		margin-bottom: 15px;
		color: #aaa;
	}

	.question {
		display: flex;
		justify-content: center;
		font-size: 2rem;
		padding: 20px;
		height: 20vh;
		background-color: #0d2b45;
		border-radius: 10px;
		text-align: center;
		align-items: center;
		box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
	}

	/* Options Section - Bottom 50% */
	.options-section {
		height: 50vh;
		display: grid;
		grid-template-columns: 1fr 1fr;
		grid-template-rows: 1fr 1fr;
		gap: 10px;
		padding: 10px;
		background-color: #0d2b45;
	}

	.option-btn {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 15px;
		background-color: #2a5a8a;
		border: 2px solid #3a6a9a;
		border-radius: 10px;
		color: white;
		font-size: 1.2rem;
		cursor: pointer;
		transition: all 0.3s;
		text-align: center;
	}

	.option-btn:hover:not(:disabled) {
		background-color: #3a6a9a;
		transform: scale(1.02);
	}

	.option-label {
		display: flex;
		justify-content: center;
		align-items: center;
		width: 40px;
		height: 40px;
		background-color: #0d2b45;
		border-radius: 50%;
		margin-bottom: 10px;
		font-size: 1.5rem;
		font-weight: bold;
	}

	.option-text {
		font-size: 1.2rem;
	}

	.selected {
		background-color: #4a7aaa;
		border-color: #5a8aba;
	}

	.correct {
		background-color: #2e8b57;
		border-color: #3e9b67;
	}

	.incorrect {
		background-color: #8b0000;
		border-color: #9b1010;
	}

	.game-over {
		flex-grow: 1;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		text-align: center;
	}

	.game-over h2 {
		color: #ffd700;
		font-size: 2.5rem;
		margin-bottom: 20px;
	}

	.game-over p {
		font-size: 1.5rem;
		margin-bottom: 30px;
	}

	.restart-btn {
		background-color: #ffd700;
		color: #0d2b45;
		font-size: 1.2rem;
		padding: 15px 30px;
		border: none;
		border-radius: 8px;
		cursor: pointer;
		transition: all 0.3s;
	}

	.restart-btn:hover {
		transform: translateY(-3px);
		box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
	}

	@media (max-width: 768px) {
		.question {
			font-size: 1.5rem;
		}

		.option-text {
			font-size: 1rem;
		}
	}
</style>
