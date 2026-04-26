<template>
  <div class="min-h-screen bg-base-200 flex flex-col items-center p-4">
    
    <div class="w-full max-w-md mb-8">
      <div class="flex justify-between mb-2 px-1">
        <span class="text-sm font-bold">進捗: {{ currentIndex + 1 }} / {{ quizData.length }}</span>
        <span class="text-sm font-bold">スコア: {{ score }}</span>
      </div>
      <progress 
        class="progress progress-primary w-full" 
        :value="currentIndex + 1" 
        :max="quizData.length"
      ></progress>
    </div>

    <div class="card w-full max-w-md bg-base-100 shadow-xl">
      <div class="card-body items-center text-center">
        <h2 class="card-title text-4xl mb-4">{{ currentQuestion.word }}</h2>
        <p class="text-gray-500 mb-6">この単語の意味を選んでください</p>
        
        <div class="card-actions flex flex-col w-full gap-3">
          <button 
            v-for="(choice, index) in currentQuestion.choices" 
            :key="index"
            @click="checkAnswer(choice)"
            class="btn btn-outline btn-primary w-full"
          >
            {{ choice }}
          </button>
        </div>
      </div>
    </div>

    <div v-if="feedback" class="toast toast-top toast-center mt-4">
      <div :class="['alert', isCorrect ? 'alert-success' : 'alert-error']">
        <span>{{ feedback }}</span>
      </div>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      currentIndex: 0,
      score: 0,
      feedback: '',
      isCorrect: false,
      quizData: [
        { word: 'Significant', answer: '重要な', choices: ['重要な', '些細な', '速い', '静かな'] },
        { word: 'Efficiency', answer: '効率', choices: ['効果', '効率', '欠陥', '努力'] },
        { word: 'Implement', answer: '実行する', choices: ['想像する', '提案する', '実行する', '中断する'] }
      ]
    }
  },
  computed: {
    currentQuestion() {
      return this.quizData[this.currentIndex];
    }
  },
  methods: {
    checkAnswer(choice) {
      if (this.feedback) return; // 連続クリック防止

      if (choice === this.currentQuestion.answer) {
        this.score += 10;
        this.feedback = '正解！ 🎉';
        this.isCorrect = true;
      } else {
        this.feedback = `不正解... 正解は「${this.currentQuestion.answer}」`;
        this.isCorrect = false;
      }

      // 1.5秒後に次の問題へ
      setTimeout(() => {
        this.feedback = '';
        if (this.currentIndex < this.quizData.length - 1) {
          this.currentIndex++;
        } else {
          alert(`終了！最終スコア: ${this.score}`);
          this.currentIndex = 0;
          this.score = 0;
        }
      }, 1500);
    }
  }
}
</script>