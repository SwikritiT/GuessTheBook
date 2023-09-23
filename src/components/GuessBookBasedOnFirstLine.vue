<template>
  <BookItem>
    <template #score>
      <span v-if="!showTotalScore" class="score">
        Score: {{ counter }}
      </span>
      <span v-if="!showTotalScore" class="remaining">
        {{iteration}}/20
      </span>
      <span v-if="showTotalScore" class="total-score">
        Your total score is {{ counter }} !
      </span>
      <button v-if="showTotalScore" id="restart" class="hidden" @click="restartQuiz">Restart</button>
    </template>
    <template v-if="!showTotalScore" #heading>
      <div>
        “{{ description }}”
      </div>
    </template>
    <template v-if="!showTotalScore" #options>
      <div  class="options" :class="{ 'input-error': showErrorMessage }" v-for="(book, index) in selectedBooks" :key="index">
        <input type="text"
               class="book-titles"
               :value="book"
               name="bookTitles"
               @click="checkSelectedOption(book)"
               @blur="resetBorder" readonly/>
      </div>
<!--      <span v-if="showErrorMessage" class="error-msg">:( Wrong answer. The right answer is <em>{{ actualBook }}</em> </span>-->
      <IconRightArrow v-if="showErrorMessage" class="next" @click="nextQuestion"/>
<!--      <button v-if="showErrorMessage" class="submit" @click="nextQuestion">Next</button>-->
    </template>
  </BookItem>
</template>

<script setup lang="ts">
import books from '../util/books.json'
import bookOption from '../util/booktitles.json'
import { ref, computed, onMounted } from 'vue'
import BookItem from "@/components/BookItem.vue"
import IconRightArrow from "@/components/icons/IconRightArrow.vue"

const selectedBook = ref<string | null>(null)
const showNext = ref(false)
const description = ref<string | null>(null)
const optionBooks = ref<string[] | null>(null)
const actualBook = ref<string | null>(null)
const counter = ref(0)
const titlesToGuess = ref<string[] | null>(null)
const showErrorMessage = ref(false)
const showTotalScore = ref (false)
const  iteration = ref(1)
const isInputClicked = ref(false)


function setArrayForQuiz() {
  const entries = Object.entries(books)
  titlesToGuess.value = shuffleArray(entries)
}

function resetBorder() {
 isInputClicked.value = false
}
function reset() {
  selectedBook.value = null
  showNext.value = false
  description.value = null
  optionBooks.value = null
  actualBook.value = null
  counter.value = 0
  titlesToGuess.value = null
  showErrorMessage.value = false
  showTotalScore.value = false
  iteration.value = 1
}
function displayDescription(){
  if(iteration.value > 20){
    showTotalScore.value = true
    return;
  }
  const[title,firstLine] = titlesToGuess.value[iteration.value];
  description.value = firstLine
  optionBooks.value = getRandomElementsWithExclusion(title)
  actualBook.value = title
}

function getRandomElementsWithExclusion(exclusion: string): string[] {
  // Filter out elements equal to the exclusion string i.e book title
  const filteredArray = bookOption.filter(item => item !== exclusion);

  if (filteredArray.length < 3) {
    throw new Error("Array size is less than 3 after exclusion.");
  }

  const result: string[] = [];
  while (result.length < 3) {
    const randomIndex = Math.floor(Math.random() * filteredArray.length);
    const randomElement = filteredArray.splice(randomIndex, 1)[0];
    result.push(randomElement);
  }

  result.push(exclusion); // Add the excluded string to the result
  return shuffleArray(result);
}

// Fisher-Yates Algorithm for shuffling
function shuffleArray<T>(array: T[]): T[] {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]]; // Swap elements
  }
  return array;
}

function checkSelectedOption(book){
  isInputClicked.value = true
  showErrorMessage.value = false
  console.log(book)
  showNext.value = book === actualBook.value
  iteration.value = iteration.value + 1
  if(showNext.value){
    counter.value = counter.value + 1
    displayDescription()
  } else{
    showErrorMessage.value = true
  }
}

function nextQuestion() {
  showErrorMessage.value = false
  displayDescription()
}

function restartQuiz () {
  reset()
  setArrayForQuiz()
  displayDescription()
}

const selectedBooks = computed(() => optionBooks.value)

onMounted(() => {
  setArrayForQuiz()
  displayDescription()
})

</script>
<style scoped>
.input-error {
  border: 1px solid red;
  animation-name: skew-x-shaking;
  animation-duration: 3s;
}

@keyframes skew-x-shaking{
  0% { transform: skewX(-15deg); }
  5% { transform: skewX(15deg); }
  10% { transform: skewX(-15deg); }
  15% { transform: skewX(15deg); }
  20% { transform: skewX(0deg); }
  100% { transform: skewX(0deg); }
}

/* Define a class for the button */
.book-titles {
  width:100%;
  background-color: transparent;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  padding: 5px 20px;
  margin-top: 15px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 5px;
  color: #9FE2BF;
  font-weight: bold;
  text-decoration: none;
  line-height: 1.5;
  letter-spacing: 1px;
  display: flex;
  flex-wrap: wrap;
  //transition: background-color 0.2s, box-shadow 0.2s;
  cursor: pointer;
}

.next {
  margin-top: 10px;
  float: right;
}

.options {
 display: flex;
 align-items: center;
  justify-content: center;
}

.error-msg {
  font-size: 14px;
  line-height: 1;
  font-weight: bold;
  color: var(--vt-c-error);
  padding: 0 5px;
}

.score{
  color: #9FE2BF;
}
.remaining {
  color: #9FE2BF;
  float: right;
}

input {
  margin-top: 5px;
}

.total-score {
  text-align: center;
  animation-name: fade-in;
  animation-duration: 4s;
}
@keyframes fade-in {
  from {
    opacity: 0;
    font-size: 12px;
    text-align: center;
  }
  to {
    opacity: 1;
    font-size: 24px;
    text-align: center;
  }
}

#restart {
  background-color: transparent;
  box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  padding: 5px 20px;
  margin-top: 15px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 5px;
  color: var(--color-text);
  font-weight: bold;
  text-decoration: none;
  line-height: 1.5;
  letter-spacing: 1px;
  animation-name: fade-in-restart;
  animation-duration: 6s;
  display:flex;
}

@keyframes fade-in-restart {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.book-titles:hover{
  background-color: #A9B2AC;
  color: var(--vt-c-white);
  cursor: pointer;
}

#restart:hover {
  background-color: #f0f0f0;
  box-shadow: 4px 4px 6px rgba(0, 0, 0, 0.3);
  color: var( --vt-c-text-light-2);
  cursor: pointer;
}
</style>
