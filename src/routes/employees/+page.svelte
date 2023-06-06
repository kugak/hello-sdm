<script>
    import { PUBLIC_CODA_KEY } from '$env/static/public';
    import { onMount, beforeUpdate } from 'svelte';
    import axios from 'axios';
    import { fade, blur, fly, slide, scale } from "svelte/transition";
    import { quintOut } from "svelte/easing";
    import { page } from '$app/stores';

    const pageId = $page.url.searchParams.get('pid') || 'JPDW7_bTYb';
    const tableId = $page.url.searchParams.get('tid') || 'grid-h5ed1eQGwZ';
    console.log(pageId);
    console.log(tableId);
    // console.log(CODA_KEY);
    let employees = [];
    let error = null;
    let lastUpdated = null;



    async function fetchData() {
        try {
            const response = await axios.get(`https://coda.io/apis/v1/docs/${pageId}/tables/${tableId}/rows?useColumnNames=true&valueFormat=rich&query=isPlaying:"true"`, {
                headers: {
                    'Authorization': `${PUBLIC_CODA_KEY}`
                },
            });
            employees = response.data.items;
            lastUpdated = new Date().toLocaleTimeString(); // Update last updated timestamp
            // logger.info('Data fetched successfully', { employees });
        } catch (e) {
            error = e;
            console.log(e);
            // logger.error('Error fetching data from Coda API', { error: e });
        }
        console.log(employees);
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
    onMount(() => {
        return () => {
            clearInterval(interval);
        };
    });
</script>
<style>
    .container {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 20px;
    }
    
    .service-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
    }
    
    img {
        height: 300px;
    }
</style>

<div>
    <p>Last Updated: {lastUpdated || 'Not available'}</p>
    {#if employees.length > 0}
        <!-- Display the fetched data -->
        <div class="container">
            {#each employees as service}
            <div class="service-item" transition:fade={{ duration: 300, easing: quintOut }}>
                <img src="{service.values.Image[0].url}" alt="">
                <p>Name: {(service.values.Name).slice(3,-3)}</p>
                <p>Title: {(service.values.Title).slice(3,-3)}</p>
                <p>
                    Days:
                    {#each service.values.Weekdays as weekday}
                        {weekday.name}{" "}
                    {/each}
                </p>
            </div>            {/each}
        </div>
    {:else if error !== null}
        <!-- Display error message -->
        <p>Error fetching data: {error.message}</p>
    {:else}
        <!-- Display loading message or spinner -->
        <p>Loading...</p>
    {/if}
</div>
