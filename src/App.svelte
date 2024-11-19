<script>
    import { onMount } from "svelte";

    let quote = "";
    const cacheKey = "tegquote:quote";
    const cacheTimeKey = "tegquote:quoteTime";
    const targetHourUTC = 17;

    function shouldUpdateQuote() {
        const now = new Date();
        const utcHour = now.getUTCHours();
        const cachedTime = localStorage.getItem(cacheTimeKey);
        const cachedQuote = localStorage.getItem(cacheKey);

        return (
            utcHour === targetHourUTC ||
            !cachedTime ||
            !cachedQuote
        );
    }

    async function loadText() {
        const response = await fetch("./today.txt");
        quote = await response.text();

        localStorage.setItem(cacheKey, quote);
        localStorage.setItem(cacheTimeKey, Date.now());
    }

    onMount(() => {
        const cachedText = localStorage.getItem(cacheKey);

        if (shouldUpdateQuote()) {
            loadText();
        } else {
            quote = cachedText;
        }
    });
</script>

<main>
    <h1 class="text-palette-2 text-6xl">{quote}</h1>
</main>
