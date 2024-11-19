<script>
    function parseCSV(csv) {
        const rows = csv.split("\n");

        const regex = /^"([^"]*)" *, *([^",\n]+)/;
        const match = rows[0].match(regex);

        if (match) {
            return {
                quote: match[1], 
                author: match[2],
            };
        } else {
            console.error("Failed to parse CSV");
            return null;
        }
    }

    let quote = "";
    let author = "";
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
            const sheetID =
                "2PACX-1vReXV2ZHAVsvPMPl49jvC6gM3puYm9bNWSYRN6aSOpZOWu-QXIawE8tYFXlGBueygBqSC-7vBlMQiXG";

            const response = await fetch(
                `https://docs.google.com/spreadsheets/d/e/${sheetID}/pub?output=csv`,
            );
            if (!response.ok) {
                throw new Error("Failed to fetch quote");
            }

            const csv = await response.text();
            const newQuote = parseCSV(csv);

            localStorage.setItem(cacheKey, JSON.stringify(newQuote));
            localStorage.setItem(
                cacheDateKey,
                new Date().toISOString().split("T")[0],
            );

            quote = newQuote.quote;
            author = newQuote.author;
        } catch (error) {
            console.error("Error fetching quote:", error);
            quote = "Could not load quote. Try again later.";
        }
    }

    async function main() {
        const cachedQuote = JSON.parse(localStorage.getItem(cacheKey));

        if (shouldUpdateQuote()) {
            await loadText();
        } else {
            quote = cachedQuote.quote;
            author = cachedQuote.author;
        }
    }
</script>

<main class="container">
    {#await main()}
        <h1 class="text-center text-palette-2 text-3xl md:text-6xl">
            Loading...
        </h1>
    {:then}
        <h1 class="text-center text-palette-2 text-3xl md:text-6xl mb-7">"{quote}"</h1>
        <h2 class="text-center text-palette-3 text-2xl md:text-4xl">
            - {author}
        </h2>
    {/await}
</main>

