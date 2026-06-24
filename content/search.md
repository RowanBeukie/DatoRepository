---
title: "Search Repository"
type: "docs"
---

## Search the repository

<input type="text" id="custom-search-input" placeholder="Search..." style="width: 100%; padding: 12px; margin: 20px 0; border: 2px solid #ccc; border-radius: 4px; font-size: 16px; background: #fff; color: #000;">

<div id="custom-search-results" style="margin-top: 20px;"></div>

<script>
(function() {
    let searchIndex = [];

    // Step A: Fetch Hugo's internal search library file automatically 
    // This matches the exact index file Hugo Book creates natively!
    fetch('/index.json')
        .then(response => response.json())
        .then(data => {
            searchIndex = data;
            console.log("Local documentation database loaded successfully. Pages indexed:", searchIndex.length);
        })
        .catch(err => console.error("Error loading index database:", err));

    // Step B: Listen to keystrokes and instantly filter matches down the page
    const input = document.getElementById("custom-search-input");
    const resultsContainer = document.getElementById("custom-search-results");

    if (input && resultsContainer) {
        input.addEventListener("input", function() {
            const query = this.value.toLowerCase().trim();
            resultsContainer.innerHTML = ""; // Clear old display view

            if (query.length < 2) return;

            // Match terms against page content or title strings
            const matches = searchIndex.filter(page => {
                return (page.title && page.title.toLowerCase().includes(query)) || 
                       (page.content && page.content.toLowerCase().includes(query));
            });

            if (matches.length === 0) {
                resultsContainer.innerHTML = '<p style="color: #666;">No matching topics found.</p>';
                return;
            }

            // Build out clean result clickable elements list layout cards
            matches.forEach(page => {
                const resultCard = document.createElement("div");
                resultCard.style.cssText = "padding: 15px; margin-bottom: 10px; border-left: 4px solid #0055bb; background: #f9f9f9; border-radius: 0 4px 4px 0;";

                // Create a snippet preview string around the matched query text phrase
                let snippet = "";
                if (page.content) {
                    const idx = page.content.toLowerCase().indexOf(query);
                    const start = Math.max(0, idx - 60);
                    const end = Math.min(page.content.length, idx + 100);
                    snippet = (start > 0 ? "..." : "") + page.content.substring(start, end).trim() + "...";
                }

                // Append the search parameter value to the URL tail query dynamically (?hl=keyword)
                const destinationUrl = page.permalink || page.uri || page.relpermalink;
                const finalUrl = `${destinationUrl}?hl=${encodeURIComponent(query)}`;

                resultCard.innerHTML = `
                    <h3 style="margin: 0 0 5px 0;"><a href="${finalUrl}" style="color: #0055bb; text-decoration: none; font-weight: bold;">${page.title || "Untitled Document"}</a></h3>
                    <p style="margin: 0; color: #444; font-size: 14px; font-style: italic;">${snippet}</p>
                `;
                resultsContainer.appendChild(resultCard);
            });
        });
    }
})();
</script>