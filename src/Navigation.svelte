<script>
  let { currentQuestionIndex, totalQuestions, onPrev, onNext } = $props();

  let disablePrev = $derived(currentQuestionIndex === 0);
  let disableNext = $derived(currentQuestionIndex === totalQuestions - 1);

  import { ArrowRight } from "lucide-svelte";
  import { ArrowLeft } from "lucide-svelte";
</script>

<div
  class="flex justify-between item-center w-full max-w-xl bg-white p-5 rounded-lg"
>
  <button
    onclick={onPrev}
    class="px-5 py-2.5 font-mono text-sm font-semibold text-gray-900 bg-white border border-gray-200 rounded-lg flex items-center gap-4
             hover:bg-gray-100 hover:text-blue-700 focus:outline-none disabled:cursor-not-allowed disabled:opacity-70 disabled:bg-gray-200 cursor-pointer"
    disabled={disablePrev}
  >
    <ArrowLeft size={16} />
    Previous
  </button>

  <button
    onclick={onNext}
    class="px-5 py-2.5 font-mono text-sm font-semibold text-gray-900 bg-white border border-gray-200 rounded-lg flex items-center gap-4
             hover:bg-gray-100 hover:text-blue-700 focus:outline-none disabled:cursor-not-allowed disabled:opacity-70 disabled:bg-gray-200 cursor-pointer"
    disabled={disableNext}
  >
    Next
    <ArrowRight size={16} />
  </button>
</div>
