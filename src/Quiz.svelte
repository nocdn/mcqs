<script>
  import Question from "./Question.svelte";
  import Navigation from "./Navigation.svelte";

  let currentQuestionIndex = $state(0);
  let answered = $state(false);
  let selectedOption = $state(null);
  let questionsAndAnswers = $state([]);
  let isLoading = $state(true);
  let error = $state(null);

  const lambdaUrl =
    "https://4mu4p3ymqfloak377n6whzywpy0jcfki.lambda-url.eu-west-2.on.aws/";

  fetch(lambdaUrl)
    .then((response) => {
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      return response.json();
    })
    .then((data) => {
      questionsAndAnswers = data;
      isLoading = false;
    })
    .catch((error) => {
      console.error("Error fetching data:", error);
      error = "Failed to load questions. Please try again later.";
      isLoading = false;
    });

  function handlePrev() {
    if (currentQuestionIndex > 0) {
      currentQuestionIndex--;
      resetAnswer();
    }
  }

  function handleNext() {
    if (currentQuestionIndex < questionsAndAnswers.length - 1) {
      currentQuestionIndex++;
      resetAnswer();
    }
  }

  function checkAnswer(option) {
    if (answered) return;
    answered = true;
    selectedOption = option;
  }

  function resetAnswer() {
    answered = false;
    selectedOption = null;
  }
</script>

<div class="bg-white p-6 rounded-lg shadow-lg w-full max-w-xl mb-5">
  {#if isLoading}
    <div class="flex justify-center items-center h-40">
      <div
        class="animate-spin rounded-full h-8 w-8 border-b-2 border-gray-900"
      ></div>
    </div>
  {:else if error}
    <div class="text-red-500 text-center p-4">
      {error}
    </div>
  {:else if questionsAndAnswers.length > 0}
    <Question
      question={questionsAndAnswers[currentQuestionIndex].question}
      options={questionsAndAnswers[currentQuestionIndex].options}
      correctAnswer={questionsAndAnswers[currentQuestionIndex].answer}
      {answered}
      {selectedOption}
      onCheckAnswer={checkAnswer}
    />

    <Navigation
      {currentQuestionIndex}
      totalQuestions={questionsAndAnswers.length}
      onPrev={handlePrev}
      onNext={handleNext}
    />
  {/if}
</div>
