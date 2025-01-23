<script>
  import Question from "./Question.svelte";
  import Navigation from "./Navigation.svelte";

  let currentQuestionIndex = $state(0);
  let answered = $state(false);
  let selectedOption = $state(null);

  const questionsAndAnswers = [
    {
      question:
        "Which of the following statements best describes the difference in how the DSM and ICD approach the classification of mental disorders, based on your notes?",
      options: [
        "The DSM is more qualitative in its descriptions, while the ICD is quantitatively based, and is therefore favoured in research due to its more specific approach.",
        "The DSM is primarily used in Europe and is free, while the ICD is used in the US and is very expensive.",
        "The DSM, compared to the ICD, is often criticized for its detailed symptom descriptions and associated diagnostic criteria, which makes it more suited for research and less accessible to general medical use.",
        "The ICD is less valued in research settings as the descriptions of disorders are general and lack quantitative data, and thus does not update with newer findings as quickly as DSM.",
      ],
      answer:
        "The ICD is less valued in research settings as the descriptions of disorders are general and lack quantitative data, and thus does not update with newer findings as quickly as DSM.",
    },
    {
      question:
        "According to your notes, which of the following best describes the controversy around the rise of DID diagnoses, specifically in the 1980’s?",
      options: [
        "The dramatic rise in cases coincided with the exposure of Sybil as a hoax, leading to questions about the validity of DID.",
        "Despite a peak in diagnoses in the 1980s, this was not a true reflection of increasing cases and was largely due to cult-based hysteria and inappropriate techniques.",
        "The increase of DID cases was mainly due to more cases being found in the general public due to wider awareness of the disorder, with treatment through hypnosis and medication being shown to be successful.",
        "An increase in diagnosis was directly related to a new definition of the disorder being published in the DSM in 1980, making diagnoses easier.",
      ],
      answer:
        "Despite a peak in diagnoses in the 1980s, this was not a true reflection of increasing cases and was largely due to cult-based hysteria and inappropriate techniques.",
    },
    {
      question:
        "Which statement best describes the 'neuroadaptation' process, as detailed in your notes, in relation to addictive behaviours like social media use and computer gaming?",
      options: [
        "It refers to the brain increasing grey matter and dopamine receptor availability in response to excessive stimuli to heighten the reward system.",
        "It involves the brain's attempt to maintain homeostasis by decreasing grey matter and reducing dopamine receptor availability as a result of chronic overstimulation.",
        "It is a short-term reduction in neural activity that occurs directly after using addictive substances or engaging in addictive behaviour, and which then returns to normal levels.",
        "It is the brain's ability to quickly adapt and develop new neural pathways to help people stop addictive behaviours.",
      ],
      answer:
        "It involves the brain's attempt to maintain homeostasis by decreasing grey matter and reducing dopamine receptor availability as a result of chronic overstimulation.",
    },
    {
      question:
        "How does the concept of 'Pavlovian conditioning' contribute to our understanding of addiction according to your notes?",
      options: [
        "It explains that addiction is a result of an over active unconditioned stimulus",
        "It highlights how drug-associated stimuli (CS) come to acquire potent conditioned properties by activating the mesoaccumbens dopamine projection, leading to craving.",
        "It explains how drugs activate mesoaccumbens dopamine projection which are then interpreted as potent stimuli, which is why people relapse months later.",
        "It is a result of reward stimuli not properly activating the mesoaccumbens dopamine projection, which results in an increase in drug use.",
      ],
      answer:
        "It highlights how drug-associated stimuli (CS) come to acquire potent conditioned properties by activating the mesoaccumbens dopamine projection, leading to craving.",
    },
  ];

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
  <Question
    question={questionsAndAnswers[currentQuestionIndex].question}
    options={questionsAndAnswers[currentQuestionIndex].options}
    correctAnswer={questionsAndAnswers[currentQuestionIndex].answer}
    {answered}
    {selectedOption}
    onCheckAnswer={checkAnswer}
  />
</div>

<Navigation
  {currentQuestionIndex}
  totalQuestions={questionsAndAnswers.length}
  onPrev={handlePrev}
  onNext={handleNext}
/>
