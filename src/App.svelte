<script>
  import Quiz from "./Quiz.svelte";
  import { createClient } from "@supabase/supabase-js";

  const supabase = createClient(
    "https://qncicczuoqawbufxevjm.supabase.co",
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InFuY2ljY3p1b3Fhd2J1ZnhldmptIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Mzc2MzMzMDMsImV4cCI6MjA1MzIwOTMwM30.IPkFgtY1OP0lqzvsMv1zAqTu3AmMeI5in9HtctSOR5Q"
  );

  function getUserAgent() {
    return navigator.userAgent;
  }

  async function getAddresses() {
    const { data, error } = await supabase.from("users").select("ip_address");
    if (error) {
      console.error("Error fetching addresses:", error);
      return [];
    }
    return data.map((item) => item.ip_address);
  }

  async function isIPUnique(ipToCheck) {
    const existingIPs = await getAddresses();
    return !existingIPs.includes(ipToCheck);
  }

  async function logSupabase(ipAddress, restOfData) {
    const shouldLog = await isIPUnique(ipAddress);

    if (!shouldLog) {
      console.log("IP already exists in database");
      return;
    }

    const { error } = await supabase.from("users").insert({
      user_agent_string: getUserAgent(),
      ip_address: ipAddress,
      ip_info_rest: restOfData,
    });

    if (error) {
      console.error("Error:", error);
    }
  }

  function getIPAndLogSupabase() {
    fetch("https://api.ipify.org?format=json")
      .then((response) => response.json())
      .then((data) => {
        const ipAddress = data.ip;
        const restOfData = data;
        logSupabase(ipAddress, restOfData);
      })
      .catch((error) => {
        console.error("Error:", error);
      });
  }

  getIPAndLogSupabase();
</script>

<div
  style="height: 100dvh"
  class="flex flex-col gap-4 items-center justify-center bg-gray-100 p-4"
>
  <div
    role="radiogroup"
    aria-required="false"
    dir="ltr"
    class="flex flex-wrap gap-2"
    tabindex="0"
    style="outline: none;"
  >
    <div
      class="relative flex flex-col items-start gap-4 rounded-lg border border-input p-3 shadow-sm shadow-black/5 has-[[data-state=checked]]:border-ring"
    >
      <div class="flex items-center gap-2">
        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
          type="button"
          role="radio"
          aria-checked="false"
          data-state="unchecked"
          value="1"
          class="aspect-square size-4 rounded-full border border-input shadow-sm shadow-black/5 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:border-primary data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground after:absolute after:inset-0"
          id=":Ss:-1"
          tabindex="-1"
          data-radix-collection-item=""
        ></button><label
          class="text-sm font-medium leading-4 text-foreground peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          for=":Ss:-1">Set 1</label
        >
      </div>
    </div>
    <div
      class="relative flex flex-col items-start gap-4 rounded-lg border border-input p-3 shadow-sm shadow-black/5 has-[[data-state=checked]]:border-ring"
    >
      <div class="flex items-center gap-2">
        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
          type="button"
          role="radio"
          aria-checked="true"
          data-state="checked"
          value="2"
          class="aspect-square size-4 rounded-full border border-input shadow-sm shadow-black/5 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:border-primary data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground after:absolute after:inset-0"
          id=":Ss:-2"
          tabindex="0"
          data-radix-collection-item=""
          ><span
            data-state="checked"
            class="flex items-center justify-center text-current"
            ><svg
              width="6"
              height="6"
              viewBox="0 0 6 6"
              fill="currentcolor"
              xmlns="http://www.w3.org/2000/svg"
              ><circle cx="3" cy="3" r="3"></circle></svg
            ></span
          ></button
        ><label
          class="text-sm font-medium leading-4 text-foreground peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          for=":Ss:-2">Set 2</label
        >
      </div>
    </div>
    <div
      class="relative flex flex-col items-start gap-4 rounded-lg border border-input p-3 shadow-sm shadow-black/5 has-[[data-state=checked]]:border-ring"
    >
      <div class="flex items-center gap-2">
        <!-- svelte-ignore a11y_consider_explicit_label -->
        <button
          type="button"
          role="radio"
          aria-checked="false"
          data-state="unchecked"
          value="3"
          class="aspect-square size-4 rounded-full border border-input shadow-sm shadow-black/5 outline-offset-2 focus-visible:outline focus-visible:outline-2 focus-visible:outline-ring/70 disabled:cursor-not-allowed disabled:opacity-50 data-[state=checked]:border-primary data-[state=checked]:bg-primary data-[state=checked]:text-primary-foreground after:absolute after:inset-0"
          id=":Ss:-3"
          tabindex="-1"
          data-radix-collection-item=""
        ></button><label
          class="text-sm font-medium leading-4 text-foreground peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
          for=":Ss:-3">Set 3</label
        >
      </div>
    </div>
  </div>
  <Quiz />
</div>

<div
  class="absolute bottom-0 left-0 right-0 w-fit px-3 py-1 opacity-35 font-mono text-sm font-semibold"
>
  Questions updated at: 23/01/2025 18:26
</div>
