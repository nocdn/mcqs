<script>
  import Question from "./Question.svelte";
  import Navigation from "./Navigation.svelte";

  let currentQuestionIndex = $state(0);
  let answered = $state(false);
  let selectedOption = $state(null);
  let questionsAndAnswers = $state([]);
  let isLoading = $state(true);
  let error = $state(null);
  let sets = $state({});
  let currentSet = $state(1);

  const lambdaUrl =
    "https://4mu4p3ymqfloak377n6whzywpy0jcfki.lambda-url.eu-west-2.on.aws/";
  const setsUrl =
    "https://4mu4p3ymqfloak377n6whzywpy0jcfki.lambda-url.eu-west-2.on.aws/sets";

  async function fetchSets() {
    try {
      const response = await fetch(setsUrl);
      if (!response.ok) throw new Error("Failed to fetch sets");
      sets = await response.json();
      console.log(sets);
      await loadQuestionsForSet(currentSet);
    } catch (err) {
      console.error("Error fetching sets:", err);
      error = "Failed to load question sets";
      isLoading = false;
    }
  }

  async function loadQuestionsForSet(setNumber) {
    try {
      console.log(`Loading questions for set ${setNumber}`);
      isLoading = true;
      resetQuizState();

      const response = await fetch(setsUrl);
      if (!response.ok) throw new Error("Failed to fetch questions");

      let setsResponse = await response.json();
      questionsAndAnswers = setsResponse[`set_${setNumber}`];
      isLoading = false;
      console.log(`Questions loaded for set ${setNumber}`);
    } catch (err) {
      console.error("Error loading questions:", err);
      error = "Failed to load questions";
      isLoading = false;
    }
  }

  function handleSetChange(setNumber) {
    console.log(`Changing set to ${setNumber}`);
    currentSet = setNumber;
    loadQuestionsForSet(setNumber);
  }

  function resetQuizState() {
    currentQuestionIndex = 0;
    answered = false;
    selectedOption = null;
  }

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

  // Initial fetch
  fetchSets();
</script>

<div>
  <div
    role="radiogroup"
    aria-required="false"
    dir="ltr"
    class="flex flex-wrap gap-2"
    tabindex="0"
    style="outline: none;"
  >
    {#if sets}
      {#each Object.keys(sets) as setKey}
        {@const setNumber = parseInt(setKey.replace("set_", ""))}
        <div
          class="relative flex flex-col items-start gap-4 rounded-lg border border-input p-3 shadow-sm shadow-black/5 has-[[data-state=checked]]:border-ring"
          style="border-opacity: 0.5; background-color: white;"
        >
          <div class="flex items-center gap-2">
            <button
              type="button"
              role="radio"
              aria-checked={currentSet === setNumber}
              data-state={currentSet === setNumber ? "checked" : "unchecked"}
              value={setNumber}
              class="aspect-square size-4 rounded-full border border-input shadow-sm shadow-black/5 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:border-primary data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground after:absolute after:inset-0"
              onclick={() => handleSetChange(setNumber)}
              style="border-opacity: 0.5; background-color: white;"
            >
              {#if currentSet === setNumber}
                <span class="flex items-center justify-center text-current">
                  <svg
                    width="6"
                    height="6"
                    viewBox="0 0 6 6"
                    fill="currentcolor"
                    xmlns="http://www.w3.org/2000/svg"
                  >
                    <circle cx="3" cy="3" r="3" />
                  </svg>
                </span>
              {/if}
            </button>
            <label
              class="text-sm font-medium leading-4 text-foreground peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
            >
              Set {setNumber}
            </label>
          </div>
        </div>
      {/each}
    {/if}
  </div>
</div>

<div class="bg-white p-6 rounded-lg shadow-lg w-full max-w-2xl mb-5">
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
