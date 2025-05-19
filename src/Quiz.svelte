<script>
  import Question from "./Question.svelte";
  import Navigation from "./Navigation.svelte";
  import ExplainModal from "./ExplainModal.svelte";

  import { Check } from "lucide-svelte";
  import NumberFlow from "@number-flow/svelte";

  let currentQuestionIndex = $state(0);
  let answered = $state(false);
  let selectedOption = $state(null);
  let questionsAndAnswers = $state([]);
  let isLoading = $state(true);
  let error = $state(null);
  let sets = $state({});
  let currentSet = $state(1);
  let answeredQuestions = $state([]);

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

  // Modified shuffleArray to be non-mutating
  function shuffleArray(inputArray) {
    // Ensure inputArray is an array before spreading
    if (!Array.isArray(inputArray)) {
      return [];
    }
    const array = [...inputArray]; // Create a shallow copy
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }

  async function loadQuestionsForSet(setNumber) {
    try {
      console.log(`Loading questions for set ${setNumber}`);
      isLoading = true;
      resetQuizState();

      const response = await fetch(setsUrl);
      if (!response.ok) throw new Error("Failed to fetch questions for set");

      let setsResponse = await response.json();
      questionsAndAnswers = setsResponse[`set_${setNumber}`] || [];
      isLoading = false;
      console.log(`Questions loaded for set ${setNumber}`);
    } catch (err) {
      console.error("Error loading questions:", err);
      error = "Failed to load questions";
      isLoading = false;
    }
  }

  // Derived state for the current question
  let currentQuestion = $derived(
    questionsAndAnswers && questionsAndAnswers.length > currentQuestionIndex
      ? questionsAndAnswers[currentQuestionIndex]
      : null
  );

  // Derived state for shuffled options for the current question
  let displayedShuffledOptions = $derived.by(() => {
    if (currentQuestion && currentQuestion.options) {
      return shuffleArray(currentQuestion.options);
    }
    return [];
  });

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
    explanation = "";
  }

  function handleNext() {
    if (currentQuestionIndex < questionsAndAnswers.length - 1) {
      currentQuestionIndex++;
      resetAnswer();
    }
    explanation = "";
  }

  function isQuestionAnswered(questionIndex) {
    // Use currentQuestion if available, otherwise fallback to array access
    // This protects against potential race conditions if currentQuestion is not yet updated
    const questionText = questionsAndAnswers[questionIndex]?.question;
    return answeredQuestions.includes(questionText);
  }

  function setCookie(name, value, days = 365) {
    const date = new Date();
    date.setTime(date.getTime() + days * 24 * 60 * 60 * 1000);
    const expires = `expires=${date.toUTCString()}`;
    document.cookie = `${name}=${JSON.stringify(value)};${expires};path=/`;
  }

  function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) {
      const cookieValue = parts.pop().split(";").shift();
      try {
        return JSON.parse(cookieValue);
      } catch {
        return cookieValue;
      }
    }
    return null;
  }

  function loadAnsweredQuestions() {
    const stored = getCookie("answeredQuestions");
    if (stored) {
      answeredQuestions = stored;
    }
  }

  function saveAnsweredQuestion(questionText) {
    if (!answeredQuestions.includes(questionText)) {
      answeredQuestions = [...answeredQuestions, questionText];
      setCookie("answeredQuestions", answeredQuestions);
    }
  }

  function checkAnswer(option) {
    if (answered) return;
    answered = true;
    selectedOption = option;
    if (currentQuestion) {
      saveAnsweredQuestion(currentQuestion.question);
    }
  }

  function resetAnswer() {
    answered = false;
    selectedOption = null;
  }

  fetchSets();
  loadAnsweredQuestions();

  let showingExplainModal = $state(false);
  let explanation = $state("");
  const geminiApiKey = import.meta.env.VITE_GEMINI_API_KEY;

  async function handleExplain() {
    if (!currentQuestion) {
      explanation = "Cannot explain: No current question loaded.";
      showingExplainModal = true;
      return;
    }
    showingExplainModal = true;
    explanation = "Generating explanation..."; // Provide initial feedback

    const question = currentQuestion.question;
    const answer = currentQuestion.answer;

    const prompt = `Please explain clearly and in simple terms but without being too verbose, why the answer to the question ${question} is ${answer}. Do not use markdown. Just plain text`;

    try {
      const response = await fetch(
        "https://api.perplexity.ai/chat/completions",
        {
          method: "POST",
          headers: {
            "content-type": "application/json",
            accept: "application/json",
            Authorization: `Bearer ${import.meta.env.VITE_PERPLEXITY_API_KEY}`,
          },
          body: JSON.stringify({
            model: "sonar-pro",
            messages: [{ role: "user", content: prompt }],
          }),
        }
      );

      if (!response.ok) {
        const errorData = await response
          .json()
          .catch(() => ({ detail: "failed to parse error response" })); // Reverted error parsing structure
        // Set explanation to the error message instead of throwing, to display in modal
        explanation = `Error: ${errorData.detail || `Proxy request failed with status: ${response.status}`}`;
        console.error(
          "Gemini API error:",
          errorData.detail || response.statusText
        );
        return;
      }
      const data = await response.json();
      // Reverted success response parsing structure
      explanation = data.choices[0].message.content;
      let citations = data.citations || [];
      console.log("citations:", citations);
      console.log("pplx response:", explanation);
    } catch (err) {
      explanation = `An error occurred: ${err.message}`;
      console.error("Error in handleExplain:", err);
    }
  }
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
          class="relative flex flex-col items-start gap-4 rounded-lg border border-gray-300 border-input p-3 shadow-sm shadow-black/5 has-[[data-state=checked]]:border-ring"
          style="border-opacity: 0.5; background-color: white;"
        >
          <div class="flex items-center gap-2">
            <button
              type="button"
              role="radio"
              aria-checked={currentSet === setNumber}
              data-state={currentSet === setNumber ? "checked" : "unchecked"}
              value={setNumber}
              class="aspect-square size-4 rounded-full border border-gray-300 border-input shadow-sm shadow-black/5 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:border-primary data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground after:absolute after:inset-0"
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
            <!-- svelte-ignore a11y_label_has_associated_control -->
            <label
              class="text-sm font-medium leading-4 text-foreground peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
            >
              {setNumber === 6
                ? "Occupational, Clinical and Educational"
                : `Set ${setNumber}`}
            </label>
          </div>
        </div>
      {/each}
    {/if}
  </div>
</div>

<div class="bg-white p-6 rounded-lg shadow-lg w-full max-w-5xl mb-5">
  {#if isLoading}
    <div class="flex justify-center items-center h-96">
      <div
        class="animate-spin rounded-full h-8 w-8 border-b-2 border-gray-900"
      ></div>
    </div>
  {:else if error}
    <div class="text-red-500 text-center p-4">
      {error}
    </div>
  {:else if questionsAndAnswers.length > 0 && currentQuestion}
    <Question
      question={currentQuestion.question}
      options={displayedShuffledOptions}
      correctAnswer={currentQuestion.answer}
      {answered}
      {selectedOption}
      onCheckAnswer={checkAnswer}
      wasAnsweredBefore={isQuestionAnswered(currentQuestionIndex)}
      onExplain={handleExplain}
    />

    <div class="font-mono text-sm text-gray-600 text-center mt-2">
      Question <NumberFlow
        value={currentQuestionIndex + 1}
      />/{questionsAndAnswers.length}
    </div>

    {#if isQuestionAnswered(currentQuestionIndex)}
      <div
        class="motion-opacity-in-0 motion-blur-in-md motion-translate-y-in-100 motion-duration-300 font-mono text-sm text-gray-500 text-center mt-1 flex items-center gap-2 justify-center pl-5"
      >
        Question already answered <Check size={16} color="blue" />
      </div>
    {/if}

    <Navigation
      {currentQuestionIndex}
      totalQuestions={questionsAndAnswers.length}
      onPrev={handlePrev}
      onNext={handleNext}
    />
  {:else if questionsAndAnswers.length === 0 && !isLoading}
    <div class="text-gray-500 text-center p-4">
      No questions available for this set.
    </div>
  {/if}
</div>

{#if showingExplainModal}
  <ExplainModal
    {explanation}
    onDismiss={() => {
      showingExplainModal = false;
    }}
  />
{/if}
