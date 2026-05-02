<script>
    import SchematicRoute from "./lib/SchematicRoute.svelte";
    import { onMount } from "svelte";

    let routeData = null;

    onMount(() => {
        // Listen for the route JSON from React Native
        const handleMessage = (event) => {
            try {
                const parsedData = JSON.parse(event.data);
                if (parsedData && parsedData.route) {
                    routeData = parsedData;
                }
            } catch (e) {
                console.error("Failed to parse message from React Native:", e);
            }
        };

        window.addEventListener("message", handleMessage);
        document.addEventListener("message", handleMessage);

        // Tell React Native we are ready!
        if (window.ReactNativeWebView) {
            window.ReactNativeWebView.postMessage("READY");
        }
    });
</script>

<main>
    {#if routeData}
        <SchematicRoute route={routeData} />
    {:else}
        <div class="loading-state">
            <span>Awaiting Route Data...</span>
        </div>
    {/if}
</main>

<style>
    :global(body) {
        margin: 0;
        padding: 0;
        overflow: hidden;
        background-color: transparent;
    }

    .loading-state {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        color: white;
        font-family: sans-serif;
    }
</style>
