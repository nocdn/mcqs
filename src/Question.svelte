<script>
  let {
    question,
    options,
    correctAnswer,
    answered,
    selectedOption,
    onCheckAnswer,
    wasAnsweredBefore,
    onExplain,
  } = $props();

  import { Sparkles } from "lucide-svelte";
</script>

<div class="text-lg font-medium mb-5 leading-relaxed">
  {question}
</div>

<div class="flex flex-col gap-2">
  <!-- svelte-ignore a11y_click_events_have_key_events -->
  <!-- svelte-ignore a11y_no_static_element_interactions -->
  {#each options as option}
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <div
      class="p-4 border border-gray-300 rounded-lg cursor-pointer transition-colors"
      class:bg-green-100={answered && option === correctAnswer}
      class:border-green-300={answered && option === correctAnswer}
      class:bg-red-100={answered &&
        selectedOption === option &&
        option !== correctAnswer}
      class:border-red-300={answered &&
        selectedOption === option &&
        option !== correctAnswer}
      onclick={() => onCheckAnswer(option)}
    >
      {option}
    </div>
  {/each}
  <button
    onclick={onExplain}
    class="text-left inline-flex items-center gap-2 opacity-60 hover:opacity-100 transition-opacity cursor-pointer text-sm font-medium ml-1 mt-2"
    ><Sparkles size={14} /> Explain this</button
  >
</div>
