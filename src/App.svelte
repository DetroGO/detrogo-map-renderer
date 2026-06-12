<script>
    import SchematicRoute from "./lib/SchematicRoute.svelte";
    import { onMount } from "svelte";

    let routeData = null;
    let themeMode = "dark"; // Default

    // This handles both the initial load and real-time updates
    const syncState = (data) => {
        if (data.route) {
            routeData = data.route;
        }
        if (data.theme) {
            themeMode = data.theme;
            // This is the magic part: it toggles the :root class for your CSS variables
            document.documentElement.className = data.theme;
        }
    };

    onMount(() => {
        // Universal message handler for React Native
        const handleMessage = (event) => {
            try {
                const parsedData = JSON.parse(event.data);
                // Handle the raw route data OR the SYNC_STATE wrapper
                if (parsedData.type === "SYNC_STATE") {
                    syncState(parsedData.payload);
                } else {
                    syncState({ route: parsedData });
                }
            } catch (e) {
                console.error("Bridge Error:", e);
            }
        };

        // Standard WebView listeners
        window.addEventListener("message", handleMessage);
        document.addEventListener("message", handleMessage);

        // Expose to window for injectJavaScript calls
        window.updateScene = (data) => syncState(data.payload);

        if (window.ReactNativeWebView) {
            window.ReactNativeWebView.postMessage("READY");
        }
    });
</script>

<main class={themeMode}>
    {#if routeData}
        <SchematicRoute route={routeData} />
    {:else}
        <div class="loading-state">
            <span>Awaiting Route Data...</span>
        </div>
    {/if}
</main>

<style>
    :global(html),
    :global(body) {
        margin: 0;
        padding: 0;
        overflow: hidden;
        background-color: transparent !important;
    }

    /* Target the text color specifically for the "Awaiting" state */
    .loading-state {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        color: #888;
        font-family: "DM Sans", sans-serif;
    }
</style>
