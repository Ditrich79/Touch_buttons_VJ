<template>
  <div class="app">
    <h1>Этап III: "Терминология"</h1>
    <div class="header">
      <div class="stages">
        <div class="stage">I Этап</div>
        <div class="stage">II Этап</div>
        <div class="stage">III Этап</div>
        <div class="stage">Всего</div>
      </div>
      <div class="scores">
        <div class="score">0</div>
        <div class="score">0</div>
        <div class="score">0</div>
        <div class="score">0</div>
      </div>
      <div class="quality-questions">
        <div class="number-question">1</div>
        <div class="number-question">2</div>
        <div class="number-question">3</div>
        <div class="number-question">4</div>
        <div class="number-question">5</div>
        <div class="number-question">6</div>
        <div class="number-question">7</div>
        <div class="number-question">8</div>
        <div class="number-question">9</div>
        <div class="number-question">10</div>
      </div>
    </div>
    
    <div class="questions-container">
      <h3 class="main-label">Сопоставьте термин и определение</h3>
      <div 
        class="question" 
        v-for="(question, index) in questions" 
        :key="index" 
        :class="{ correct: questionStatus[question.text] === true, wrong: questionStatus[question.text] === false }"
      >
        <div class="answers-dropzone" 
             :class="{ 'dropzone-active': question.droppedAnswer !== null }"
             @touchend="handleAnswerDrop(question)"
        >
          <div class="dropzone-label">{{ question.droppedAnswer || 'Перетащите ответ сюда' }}</div>
        </div>
        <div class="question-text" style="font-size: 14px; font-weight: bold;">{{ question.text }}</div>

      </div>
    </div>

    <div class="answers-container" ref="answersContainer">
      <div 
        class="answer" 
        v-for="(answer, index) in answers" 
        :key="index" 
        @touchstart="startDrag(answer, $event)" 
        @touchend="endDrag"
        :class="{ dragging: isDragging }"
        :style="getAnswerStyle(answer)"
      >
        {{ answer.text }}
      </div>
      
      <button v-if="isAnswerPlaced" @click="resetAnswers" :disabled="isChecked" class="reset-button">
        Сбросить
      </button>

      <button @click="checkAllAnswers" :disabled="!allAnswersDropped" class="check-button">
        Ответить
      </button>
      
    </div>

    <div 
      class="dragging-answer" 
      v-if="isDragging" 
      :style="draggingAnswerStyle"
    >
      {{ draggedAnswerText }}
    </div>


  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [
        { text: 'Файл с кодом, выполняющий определённые команды.', droppedAnswer: null, hasDroppedAnswer: false, dropzoneStyle: {} },
        { text: 'Тег, который содержит название (заголовок) страницы.', droppedAnswer: null, hasDroppedAnswer: false, dropzoneStyle: {} },
        { text: 'Структура данных, работающая по принципу "последним пришёл, первым вышел".', droppedAnswer: null, hasDroppedAnswer: false, dropzoneStyle: {} }
      ],
      answers: [
        { text: 'Скрипт', question: 'Файл с кодом, выполняющий определённые команды.' },
        { text: 'Тайтл', question: 'Тег, который содержит название (заголовок) страницы.' },
        { text: 'Стек', question: 'Структура данных, работающая по принципу "последним пришёл, первым вышел".' }
      ],
      draggedAnswerText: '',
      questionStatus: {}, 
      isDragging: false,
      draggingAnswerStyle: {},
      isChecked: false,
      isAnswerPlaced: false,
    };
  },
  computed: {
    allAnswersDropped() {
      return this.questions.every(q => q.droppedAnswer !== null);
    }
  },
  methods: {
    handleAnswerDrop(question) {
      const droppedAnswer = this.draggedAnswerText;

      if (droppedAnswer) {
        this.markAnswerAsPlaced(question, droppedAnswer);
        this.removeDroppedAnswer(droppedAnswer);

        question.dropzoneStyle = { active: true };

        this.isAnswerPlaced = true;
      }
    },

    resetAnswers() {
      // Сбрасываем все droppedAnswers и hasDroppedAnswer для вопросов
      this.questions.forEach(question => {
        question.droppedAnswer = null;
        question.hasDroppedAnswer = false;
      });

      // Восстанавливаем все ответы
      this.answers = [
        { text: 'Скрипт', question: 'Файл с кодом, выполняющий определённые команды.' },
        { text: 'Тайтл', question: 'Тег, который содержит название (заголовок) страницы.' },
        { text: 'Стек', question: 'Структура данных, работающая по принципу "последним пришёл, первым вышел".' }
      ];

      // Сбрасываем статус правильности ответов
      this.questionStatus = {};
      this.isChecked = false; // Сбрасываем состояние проверки

      this.isAnswerPlaced = false;
    },

    resetDropzone(question) {
      question.droppedAnswer = null; // Сбрасываем ответ
      question.hasDroppedAnswer = false; // Убираем отметку о сброшенном ответе
      question.dropzoneStyle = {}; // Сбрасываем стили
    },

    checkIfAnyAnswersPlaced() {
      // Проверяем, есть ли хотя бы один ответ, размещенный на дроп-зонах
      this.isAnswerPlaced = this.questions.some(question => question.hasDroppedAnswer);
    },


    removeDroppedAnswer(answerText) {
      this.answers = this.answers.filter(a => a.text !== answerText);
    },
    
    startDrag(answer, event) {
      this.draggedAnswerText = answer.text;
      this.isDragging = true;

      document.body.style.overflow = 'hidden';      

      const touch = event.changedTouches[0];
      this.updateDraggingPosition(touch.clientX, touch.clientY);

      document.addEventListener('touchmove', this.onDrag);
      document.addEventListener('touchmove', this.preventDefaultScroll, { passive: false });
      document.addEventListener('touchend', this.endDrag);
    },
    
    onDrag(event) {
      const touch = event.changedTouches[0];
      this.updateDraggingPosition(touch.clientX, touch.clientY);
    },

    updateDraggingPosition(x, y) {
      this.draggingAnswerStyle = {
        position: 'absolute',
        left: `${x - 130}px`,
        top: `${y - 15}px`,
        pointerEvents: 'none',
      };
    },

    endDrag(event) {
      const touch = event.changedTouches ? event.changedTouches[0] : null;
      if (!touch) return; // Проверяем, есть ли touch

      const dropzoneUnderTouch = document.elementFromPoint(touch.clientX, touch.clientY);
      const dropzone = dropzoneUnderTouch ? dropzoneUnderTouch.closest('.answers-dropzone') : null;

      if (dropzone) {
        const questionElement = dropzone.closest('.question');
        if (questionElement) {
          const questionText = questionElement.querySelector('.question-text').innerText;
          const question = this.questions.find(q => q.text === questionText);
          if (question && !question.hasDroppedAnswer) {
            question.droppedAnswer = this.draggedAnswerText; // Обновляем droppedAnswer
            question.hasDroppedAnswer = true; // Отмечаем, что ответ был сброшен
            this.removeDroppedAnswer(this.draggedAnswerText); // Удаляем ответ из доступных

            // Устанавливаем состояние isAnswerPlaced в true
            this.isAnswerPlaced = true;
          } else {
            console.log('Ответ уже сброшен на эту дроп-зону');
          }
        }
      }

      // Сбрасываем состояние перетаскивания
      this.isDragging = false;
      this.draggedAnswerText = '';
      this.draggingAnswerStyle = {};
      document.body.style.overflow = '';

      // Удаляем обработчики событий
      document.removeEventListener('touchmove', this.onDrag);
      document.removeEventListener('touchmove', this.preventDefaultScroll);
      document.removeEventListener('touchend', this.endDrag);

      this.checkIfAnyAnswersPlaced();
    },
    
    resetDropzoneStyles() {
      this.questions.forEach(question => {
        question.dropzoneStyle = {}; // Сбрасываем стиль на пустой объект
        question.dropzoneStyle = { backgroundColor: 'white' };
      });
    },

    preventDefaultScroll(event) {
      event.preventDefault();
    },

    markAnswerAsPlaced(question, answerText) {
      question.droppedAnswer = answerText; 
    },

    checkAllAnswers() {
      this.questionStatus = {}; // Сброс статуса перед проверкой
      this.isChecked = true; // Устанавливаем состояние проверки в true

      const answers = [
        { text: 'Скрипт', question: 'Файл с кодом, выполняющий определённые команды.' },
        { text: 'Тайтл', question: 'Тег, который содержит название (заголовок) страницы.' },
        { text: 'Стек', question: 'Структура данных, работающая по принципу "последним пришёл, первым вышел".' }
      ];

      this.questions.forEach(question => {
        const correctAnswer = answers.find(a => a.question.trim() === question.text.trim());
        
        if (correctAnswer) {
          if (question.droppedAnswer) {
            this.questionStatus[question.text] = (question.droppedAnswer.trim() === correctAnswer.text.trim());
          } else {
            this.questionStatus[question.text] = false; // Если ответ не был выбран
          }
        } else {
          this.questionStatus[question.text] = false; // Если правильный ответ не найден
        }

        question.dropzoneStyle = {
          backgroundColor: 'lightblue',
          color: 'black',
          border: '2px solid blue'
        };
      });
    },

    getAnswerStyle(answer) {
      return {
        opacity: answer.placedInQuestion ? 1 : 1,
        transition: 'opacity 0.5s',
        width: answer.placedInQuestion ? '100%' : 'auto', 
        display: answer.placedInQuestion ? 'none' : 'block', 
      };
    },
  }
};
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  display: flex;
  flex-direction: column;
  margin: 0;
  height: 100vh;
  width: 100vw; 
  overflow: auto;
  background-image: url("@/assets/img/background-img.png");
  background-size: 100% 100%;           
  background-position: center center; 
  background-repeat: no-repeat;  
}

.questions-container {
  display: flex;
  flex-direction: column; 
  justify-content: flex-start;
  width: 87%;
  height: auto;
  border-radius: 15px;
  /* border: 2px gray solid; */
  font-family: 'KelsonSansBG-Normal', sans-serif;
  transform: translateX(7%);
  background-color: white;
}

.question {
  padding: 0; 
  margin: 2px;
  text-align: center;
  transition: background-color 0.3s;
  display: flex; 
  flex-direction: column; 
  align-items: center; 
}

.question-text {
  /* font-family: 'KelsonSansBG-Normal', sans-serif; */
  padding: 5px 10px;
  color: #4a5f8a;
  hyphens: auto;
  -webkit-hyphens: auto;
  -moz-hyphens: auto;
  -ms-hyphens: auto;
}

.correct {
  background-color: #6DD97D; /* Зеленый цвет для правильных ответов */
  /* border: 2px #151044 solid; */
  color: white;
  border-radius: 15px;
}

.wrong {
  background-color: #FF6161; /* Красный цвет для неправильных ответов */
  /* border: 2px #151044 solid; */
  color: white;
  border-radius: 15px;
}

.answers-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  left: 50%;
  transform: translateX(21%);
  width: 70%;
  margin-top: 20px;
  margin-bottom: 15px;
  min-height: 175px;
}

.answer {
  background-color: white;
  color: #4a5f8a;
  border-radius: 25px;
  padding: 8px;
  margin: 5px;
  /* width: 100px; */
  text-align: center;
  cursor: grab;
  user-select: none;
  font-size: 14px;
  transition: transform 0.2s, opacity 0.5s;
  border: 1px #151044 solid;
  font-family: 'KelsonSansBG-Normal', sans-serif;
}

.dragging-answer {
  background-color: white;
  color: #4a5f8a;
  border-radius: 15px;
  padding: 5px 5px;
  margin: 5px;
  width: 238px;
  height: 26px;
  text-align: center;
  font-size: 14px;
  border: 1px #151044 solid;
  user-select: none;
  pointer-events: none;
  font-family: 'KelsonSansBG-Normal', sans-serif;
}

.dropzone-label {
  color: rgba(74, 95, 138, 0.7);
  font-size: 15px;
}

.dropzone-active {
  background-color: white !important;
  color: #4a5f8a;
  border-radius: 15px;
  padding: 6px;
  width: 100px;
  text-align: center;
  font-size: 14px;
  transition: transform 0.2s, opacity 0.5s;
  border: 1px #151044 solid;
  font-family: 'KelsonSansBG-Normal', sans-serif;
}

.answers-dropzone {
  background-color: #e9ecef;
  color: #4a5f8a;
  min-height: 31px;
  margin-top: 10px; 
  border-radius: 25px;
  width: 77%;
  transform: translateX(0%);
  display: flex; 
  align-items: center; 
  justify-content: center; 
}

h1 {
  text-align: center;
  font-family: 'KelsonSansBG-Bold', sans-serif;
  font-size: 20px;
  color: #4a5f8a;
  margin-bottom: 10px;
}

.stages, .scores, .quality-questions {
  display: flex;
  justify-content: center;
  font-family: 'KelsonSansBG-Bold', sans-serif;
  align-items: center;
  color: #4a5f8a;
}

.stages {
  padding: 0;
  font-size: 14px;
  font-weight: bold;
  margin-bottom: 5px;
}

.stage, .number-question {
  margin: 5px;
}

.scores {
  border: 1px solid #4a5f8a;
  border-radius: 25px;
  width: 70%;
  transform: translateX(20%);
  background-color: white;
}

.score {
  margin: 2px 20px;
  font-size: 28px;
  font-weight: bold;
  /* width: 80%; */
}

.number-question {
  margin-top: 12px;
  margin-bottom: 7px;
  border: 1px solid #4a5f8a;
  border-radius: 50%;
  display: flex;               
  justify-content: center;      
  align-items: center;  
  width: 22px;
  height: 22px;     
  font-weight: bold;   
}

.main-label {
  text-align: center;
  font-family: 'KelsonSansBG-Bold', sans-serif;
  font-size: 16px;
  color: #4a5f8a;
  margin-bottom: 0;
  margin-top: 10px;
}

.number-question {
  background-color: white;
}

html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}

.check-button {
  padding: 10px 15px;
  background-color: #4a5f8a; 
  color: white; 
  border: none; 
  cursor: pointer; 
  border-radius: 15px; 
  margin-top: 10px; 
  transition: background-color 0.3s; 
  z-index: 1000;
  width: 283px;
  margin-left: -12px;
  /* transform: translateX(-2%); */
}

.check-button:disabled {
  background-color: #6c757d;
  border: #151044;
  border-radius: 15px;
}

.reset-button {
  padding: 10px 15px;
  background-color: #ff4d4d;
  color: white; 
  border: none; 
  cursor: pointer; 
  border-radius: 15px; 
  /* margin-bottom: 5px; */
  margin-top: auto;
  transition: background-color 0.3s; 
  z-index: 1000;
  width: 283px;
  margin-left: -12px;
}

.reset-button:hover {
  background-color: #ff1a1a;
}

</style>
