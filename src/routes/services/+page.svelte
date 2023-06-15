<script>
  import { page } from "$app/stores";
  import { PUBLIC_CODA_KEY } from "$env/static/public";
  import axios from "axios";
  import { afterUpdate, onDestroy, onMount } from "svelte";
  import { blur, fade } from "svelte/transition";
  // Images
  import pharmaPlus from "../../images/pharma-plus.png";

  const pageId = $page.url.searchParams.get("pid") || "JPDW7_bTYb";
  const tableId = $page.url.searchParams.get("tid") || "grid-vfXh3vEkTg";

  let services = [];
  let error = null;
  let currentTime = new Date().toLocaleTimeString([], {
    hour: "numeric",
    minute: "2-digit",
  });
  let currentDay = "";
  let timeParts = currentTime.split(" "); // Split the time string by space
  let time = timeParts[0]; // Extract the time without AM/PM
  let amPm = timeParts[1]; // Extract the AM/PM designation
  let showContent = false; // Flag to show/hide content

  console.log(time); // Output: "9:42"
  console.log(amPm); // Output: "AM"

  function updateTime() {
    currentTime = new Date().toLocaleTimeString([], {
      hour: "numeric",
      minute: "2-digit",
    });
    timeParts = currentTime.split(" ");
    time = timeParts[0];
    amPm = timeParts[1];
  }

  async function fetchData() {
    try {
      const response = await axios.get(
        `https://coda.io/apis/v1/docs/${pageId}/tables/${tableId}/rows?useColumnNames=true&valueFormat=rich&query=isPlaying:"true"`,
        {
          headers: {
            Authorization: `${PUBLIC_CODA_KEY}`,
          },
        }
      );
      services = response.data.items;
      localStorage.setItem("services", JSON.stringify(services)); // Store services data in local storage

      console.log("Data fetched successfully", { services });
    } catch (e) {
      error = e;
      console.log(e);
      console.error("Error fetching data from Coda API", { error: e });
      const storedServices = localStorage.getItem("services");
      if (storedServices) {
        services = JSON.parse(storedServices); // Use stored services data
      }
    }
    console.log(services);
    showContent = true; // Set flag to show the content
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

  afterUpdate(() => {
    updateTime(); // Update time after each update
  });
</script>

<section class="pharmacy pharmacy__service">
  <div class="ph__container">
    <div class="ph__wrapper">
      <div class="ph__top-title">
        <h3 class="ph__subtitle">Pharmacy</h3>
        <h2 class="ph__title">Care Clinic</h2>
      </div>
      <div class="services-list">
        {#if services.length > 0}
          <!-- Display the fetched data with fade-in effect -->
          {#each services as service}
            {#if showContent}
              <p in:blur out:blur>{service.name}</p>
            {/if}
          {/each}
        {:else if error !== null}
          <!-- Display error message -->
          <p>Error fetching data: {error.message}</p>
        {:else}
          <!-- Display loading message or spinner -->
          {#if !showContent}
            <p />
          {/if}
        {/if}
      </div>

      <div class="pharma_logo">
        <img src={pharmaPlus} alt="logo" />
      </div>
    </div>
  </div>

  <!-- Time -->
  <div class="service__date" in:blur out:fade>
    <div id="time" in:blur out:fade>
      <span in:blur out:fade>{time}</span>
      <span class="ampm" in:fade out:fade>{amPm}</span>
    </div>
  </div>
</section>

<style>
  @font-face {
    font-family: "GT Eesti Pro Display";
    src: url("/fonts/GTEestiProDisplay-Regular.woff2") format("woff2");
    font-weight: unset;
    font-style: unset;
    font-display: swap;
  }
  :root {
    --primary-font: "GT Eesti Pro Display";
  }
  :global(body) {
    background-color: #1b4655;
    height: 100%;
    margin: 0;
    color: #fff;
    font-family: "GT Eesti Pro Display";
    overflow: hidden;
  }

  .pharmacy {
    max-width: 1080px;
    margin: 0 auto;
    position: relative;
    z-index: 1;
    background: #1b4655;
    max-height: 1920px;
    height: 100vh;
    min-height: 1480px;
  }

  .pharmacy__service {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    max-height: 100vh;
  }

  .ph__container {
    max-width: 870px;
    margin: 0 auto;
    padding: 0px 20px;
    height: 1700px;
  }

  .ph__wrapper p {
    font-size: 2.7rem;
    letter-spacing: 1px;
    line-height: 90px;
    color: #ffffff;
    font-weight: 300;
    text-align: center;
    /* padding: 30px 20px; */
    max-width: 700px;
    margin: 0 auto;
    display: flex;
    flex-direction: column;
    align-content: center;
    align-items: center;
  }

  .ph__top-title {
    position: relative;
  }

  .ph__subtitle {
    margin: 0;
    padding: 0;
    text-align: center;
    -webkit-transform: translate(-122px, 187px);
    -ms-transform: translate(-122px, 187px);
    transform: translate(-122px, 187px);
    font-size: 3.5rem;
    font-weight: 100;
  }

  .ph__title {
    font-size: 9.2rem;
    font-weight: 100;
    margin-bottom: 1rem;
  }

  .services-list {
    height: 950px;
    height: 950px;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .service__date {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: center;
    -ms-flex-pack: center;
    justify-content: center;
    -webkit-box-align: center;
    -ms-flex-align: center;
    align-items: center;
    -webkit-column-gap: 20px;
    -moz-column-gap: 20px;
    column-gap: 20px;
  }

  .pharma_logo {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  #time {
    background-image: url("../../images/bean-blue.png");
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
    width: 450px;
    height: 400px;
    position: absolute;
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-pack: end;
    -ms-flex-pack: end;
    justify-content: flex-end;
    -webkit-box-align: end;
    -ms-flex-align: end;
    align-items: flex-end;
    bottom: 0;
    right: 0;
    padding-right: 50px;
    padding-bottom: 80px;
    font-size: 3rem;
    color: #333;
    transition: transform 0.5s;
  }

  #time,
  #time span {
    font-size: 6.2rem;
    color: #1b4655;
  }
  #time span.ampm {
    font-size: 2.2rem;
    line-height: 72px;
    color: #1b4655;
  }
</style>
