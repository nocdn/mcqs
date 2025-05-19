<script>
  import Quiz from "./Quiz.svelte";

  let last_update = $state("");
  function fetchModifiedTime() {
    fetch(
      "https://4mu4p3ymqfloak377n6whzywpy0jcfki.lambda-url.eu-west-2.on.aws/modified"
    )
      .then((response) => response.json())
      .then((data) => {
        last_update = data.last_update;
      })
      .catch((error) => {
        console.error("Error:", error);
      });
  }
  fetchModifiedTime();

  let width = $state(0);
  width = window.innerWidth;
</script>

<div
  style="height: 100dvh"
  class="flex flex-col gap-4 items-center justify-center bg-gray-100 p-4 motion-opacity-in-0 motion-blur-in-md"
>
  {#if width > 700}
    <Quiz />
  {:else}
    <div class="text-center">
      <h1 class="text-3xl font-bold">Mobile view not yet supported</h1>
      <p class="text-gray-500 font-mono">
        This quiz is best viewed on a wider-screen device.
      </p>
    </div>
  {/if}
</div>
