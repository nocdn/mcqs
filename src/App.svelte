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
  <Quiz />
</div>

<div
  class="absolute bottom-0 left-0 right-0 w-fit px-3 py-1 opacity-35 font-mono text-sm font-semibold"
>
  Questions updated at: 23/01/2025 18:26
</div>
