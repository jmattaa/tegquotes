<script lang="ts">
    interface QuoteFromCSV {
        quoteStudent: string;
        student: string;
        quoteTeacher: string;
        teacher: string;
    }

    let quoteType: "student" | "teacher" = "student";

    function parseCSV(csv: string): QuoteFromCSV {
        const match = csv.match(
            /^"""([^"]*)""",\s*"""([^"]*)""",\s*"""([^"]*)""",\s*"""([^"]*)"""$/,
        );

        if (match) {
            return {
                quoteStudent: match[1],
                student: match[2],
                quoteTeacher: match[3],
                teacher: match[4],
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
    let quoteTypeKey = "tegquote:quoteType";

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

            if (newQuote) {
                localStorage.setItem(cacheKey, JSON.stringify(newQuote));
                localStorage.setItem(
                    cacheDateKey,
                    new Date().toISOString().split("T")[0],
                );
                quote =
                    newQuote[
                        quoteType === "student"
                            ? "quoteStudent"
                            : "quoteTeacher"
                    ];
                author =
                    newQuote[quoteType === "student" ? "student" : "teacher"];
            }
        } catch (error) {
            console.error("Error fetching quote:", error);
            quote = "Could not load quote. Try again later.";
        }
    }

    let storedQuoteType = localStorage.getItem(quoteTypeKey);
    if (storedQuoteType) {
        quoteType = storedQuoteType as "student" | "teacher";
    }

    function handleBtnClick(type: "student" | "teacher") {
        quoteType = type;
        localStorage.setItem(quoteTypeKey, quoteType);
    }

    async function main() {
        let cachedQuote: QuoteFromCSV | null = null;
        try {
            cachedQuote = JSON.parse(localStorage.getItem(cacheKey));
        } catch (error) {
            console.error("Error parsing cached quote:", error);
            cachedQuote = null;
        }

        if (shouldUpdateQuote()) {
            await loadText();
        } else if (cachedQuote) {
            quote =
                cachedQuote[
                    quoteType === "student" ? "quoteStudent" : "quoteTeacher"
                ];
            author =
                cachedQuote[quoteType === "student" ? "student" : "teacher"];
        } else {
            console.warn("No valid cached quote found. Loading new quote...");
            await loadText();
        }
    }
</script>

<main>
    <div class="absolute top-4 left-4">
        <button
            on:click={() => handleBtnClick("student")}
            class={`p-2 rounded ${quoteType === "student" ? "bg-palette-2" : "bg-palette-3/50"}`}
            >Elev Quotes</button
        >
        <button
            on:click={() => handleBtnClick("teacher")}
            class={`p-2 rounded ${quoteType === "teacher" ? "bg-palette-2" : "bg-palette-3/50"}`}
            >Lärar Quotes</button
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
