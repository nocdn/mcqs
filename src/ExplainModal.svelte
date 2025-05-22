<script>
  import { X } from "lucide-svelte";
  import Spinner from "./Spinner.svelte";
  import NumberFlow from "@number-flow/svelte";

  let { onDismiss = () => {}, explanation } = $props();

  let dismissing = $state(false);
  function dismissModal() {
    dismissing = true;
    setTimeout(() => {
      onDismiss();
    }, 300);
  }

  let secondsLeft = $state(10);
  const interval = setInterval(() => {
    console.log(secondsLeft);
    secondsLeft--;
    if (secondsLeft <= 0) {
      clearInterval(interval);
      console.log("Countdown finished!");
    }
  }, 1000);
</script>

<overlay
  class="fixed top-0 left-0 w-full h-full bg-black/50 z-30 flex items-center justify-center {dismissing
    ? 'animate-bg-fade-out'
    : 'animate-bg-fade-in'}"
  onmousedown={dismissModal}
  role="dialog"
  tabindex="0"
>
  <modal
    role="button"
    tabindex="0"
    onmousedown={(e) => {
      e.stopPropagation();
    }}
    class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-xl w-1/2 max-w-2xl min-w-2xl flex flex-col gap-3 overflow-y-scroll {dismissing
      ? 'animate-settings-modal-down'
      : 'animate-settings-modal-up'}"
  >
    <p
      class="text-md font-medium font-geist-mono flex justify-between opacity-75"
    >
      EXPLANATION <X
        size={16}
        class="cursor-pointer"
        onmousedown={() => {
          dismissModal();
        }}
      />
    </p>
    {#if explanation === ""}
      <div class="inline-flex gap-2">
        <Spinner />
        <div>
          <p>Loading explanation</p>
          <p class="opacity-50 font-medium text-sm mt-1">
            This will take around <NumberFlow
              value={secondsLeft}
              class="font-mono font-semibold"
            /> seconds
          </p>
        </div>
      </div>
    {:else}
      <p class="motion-opacity-in-0">
        {explanation}
      </p>
    {/if}
  </modal>
</overlay>
