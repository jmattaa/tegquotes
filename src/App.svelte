<script>
    import { onMount } from "svelte";

    let quote = "";
    const cacheKey = "tegquote:quote";
    const cacheDateKey = "tegquote:quoteTime";
    const targetHourUTC = 8;

    function shouldUpdateQuote() {
        const now = new Date();
        const utcHour = now.getUTCHours();
        const todayDate = now.toISOString().split("T")[0];
        const cachedDate = localStorage.getItem(cacheDateKey);

        const hasReachedTargetTime = utcHour >= targetHourUTC;

        return (
            !localStorage.getItem(cacheKey) || 
            !cachedDate || 
            (cachedDate !== todayDate && hasReachedTargetTime)
        );
    }

    async function loadText() {
        try {
            const response = await fetch("./today.txt");
            if (!response.ok) {
                throw new Error("Failed to fetch quote");
            }
            const newQuote = await response.text();

            if (newQuote !== quote) {
                quote = newQuote;
                const todayDate = new Date().toISOString().split("T")[0];
                localStorage.setItem(cacheKey, newQuote);
                localStorage.setItem(cacheDateKey, todayDate);
            }
        } catch (error) {
            console.error("Error fetching quote:", error);
            quote = "Could not load quote. Try again later.";
        }
    }

    onMount(() => {
        const cachedText = localStorage.getItem(cacheKey);

        if (shouldUpdateQuote()) {
            loadText();
        } else {
            quote = cachedText || "Welcome! Today's quote will load soon.";
        }
    });
</script>

<main>
    <h1 class="text-palette-2 text-3xl md:text-6xl">{quote}</h1>
</main>

