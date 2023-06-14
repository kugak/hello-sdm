<script>
  import { page } from "$app/stores";
  import axios from "axios";
  import { onDestroy, onMount } from "svelte";

  const pageId = $page.url.searchParams.get("pid") || "JPDW7_bTYb";
  const tableId = $page.url.searchParams.get("tid") || "grid-vfXh3vEkTg";
  console.log(pageId);
  console.log(tableId);
  let services = [];
  let error = null;
  let lastUpdated = null;
  let currentTime = new Date().toLocaleTimeString([], {
    hour: "numeric",
    minute: "2-digit",
  });
  let currentDay = null;

  async function fetchData() {
    try {
      const response = await axios.get(
        `https://coda.io/apis/v1/docs/${pageId}/tables/${tableId}/rows?useColumnNames=true&valueFormat=rich&query=isPlaying:"true"`,
        {
          headers: {
            Authorization: "Bearer c32f30d9-c49f-42cf-b445-ddcb6c950e97",
          },
        }
      );
      services = response.data.items;
      lastUpdated = new Date().toLocaleTimeString([], {
        hour: "numeric",
        minute: "2-digit",
      }); // Update last updated timestamp
      currentDay = new Date().getDay(); // Get the current day index (0-6)
      services = services.filter((service) =>
        service.values.Weekdays.some((weekday) => weekday.name === currentDay)
      );
      console.log("Data fetched successfully", { services });
    } catch (e) {
      error = e;
      console.log(e);
      console.error("Error fetching data from Coda API", { error: e });
    }
    console.log(services);
  }

  onMount(() => {
    fetchData(); // Fetch data immediately on mount
  });

  // Fetch data every minute
  let interval;
  onMount(() => {
    interval = setInterval(fetchData, 60000);
  });

  // Clear interval on component destruction
  onDestroy(() => {
    clearInterval(interval);
  });
</script>

<div>
  <div class="container">
    <h1>Enter the url /services or /employees</h1>
  </div>
</div>

<style>
  :global(body) {
    background-color: #1b4655;
    height: 100%;
    margin: 0;
    color: #fff;
    font-family: "GT Eesti Pro Display";
    font-weight: 100;
  }
</style>
