<script lang="ts">
    let quoteType: "student" | "teacher" = "student";

    function parseCSV(csv) {
        const match = csv.match(/^"""([^"]*)""",\s*"""([^"]*)"""$/);

        if (match) {
            return {
                quote: match[1],
                author: match[2],
            };
        } else {
            console.error("Failed to parse CSV");
            console.log(csv);
            console.log(match);
            return null;
        }
    }

    let quote = "";
    let author = "";
    const cacheKey = "tegquote:quote";
    const cacheDateKey = "tegquote:quoteTime";

    function shouldUpdateQuote() {
        const now = new Date();
        const todayDate = now.toISOString().split("T")[0];
        const cachedDate = localStorage.getItem(cacheDateKey);

        return (
            !localStorage.getItem(cacheKey) ||
            !cachedDate ||
            cachedDate !== todayDate
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
        let cachedQuote: any;
        try {
            cachedQuote = JSON.parse(localStorage.getItem(cacheKey));
        } catch (error) {
            console.error("Error parsing cached quote:", error);
            cachedQuote = null;
        }

        if (shouldUpdateQuote()) {
            await loadText();
        } else if (cachedQuote && cachedQuote.quote && cachedQuote.author) {
            quote = cachedQuote.quote;
            author = cachedQuote.author;
        } else {
            console.warn("No valid cached quote found. Loading new quote...");
            await loadText();
        }
    }
</script>

<main>
    <div class="absolute top-4 left-4">
        <button
            on:click={() => (quoteType = "teacher")}
            class={`p-2 rounded ${quoteType === "teacher" ? "bg-palette-2" : "bg-palette-3/50"}`}
            >Lärar Quotes</button
        >
        <button
            on:click={() => (quoteType = "student")}
            class={`p-2 rounded ${quoteType === "student" ? "bg-palette-2" : "bg-palette-3/50"}`}
            >Student Quotes</button
        >
    </div>

    <div class="container">
        {#await main()}
            <h1 class="text-center text-palette-2 text-3xl md:text-6xl">
                Loading...
            </h1>
        {:then}
            <h1 class="text-center text-palette-2 text-3xl md:text-6xl mb-7">
                "{quote}"
            </h1>
            <h2 class="text-center text-palette-3 text-2xl md:text-4xl">
                - {author}
            </h2>
        {/await}
    </div>
</main>
