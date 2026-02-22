/**
 * Wynncraft Ingredient Database Refresh Script
 * Fetches data from official Wynncraft API and Wynntils, normalizes it, and saves as static JSON.
 * Run via Node.js v18+
 */
const fs = require('fs/promises');
const path = require('path');

const WYNN_API_URL = 'https://api.wynncraft.com/v3/item/database?fullResult';
const WYNNTILS_PLACES_URL = 'https://raw.githubusercontent.com/Wynntils/Reference/main/locations/places.json';

// Helper: Calculate 2D Euclidean distance (X and Z coordinates in Minecraft)
function calculateDistance(x1, z1, x2, z2) {
    return Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(z2 - z1, 2));
}

async function run() {
    console.log("🚀 Starting data refresh...");

    try {
        // 1. Fetch official items
        console.log(`📥 Fetching Wynncraft items from ${WYNN_API_URL}...`);
        const itemRes = await fetch(WYNN_API_URL);
        if (!itemRes.ok) throw new Error(`Wynncraft API returned ${itemRes.status}`);
        const itemsMap = await itemRes.json();

        // 2. Fetch Wynntils places for location enrichment
        console.log(`📥 Fetching Wynntils places...`);
        let places = [];
        try {
            const placesRes = await fetch(WYNNTILS_PLACES_URL);
            if (placesRes.ok) {
                const placesData = await placesRes.json();
                // Normalize Wynntils places data structure (usually an array of objects)
                places = Array.isArray(placesData) ? placesData : Object.values(placesData);
            }
        } catch (e) {
            console.warn("⚠️ Could not fetch Wynntils places. Location enrichment will be skipped.", e.message);
        }

        // 3. Process and filter ingredients
        console.log("⚙️ Processing ingredients...");
        const ingredients = [];

        for (const [internalName, item] of Object.entries(itemsMap)) {
            // Robust identification of ingredients
            const isIngredient = 
                (item.requirements && item.requirements.skills && item.requirements.skills.length > 0) ||
                item.consumableOnlyIDs || 
                item.ingredientPositionModifiers;

            if (!isIngredient) continue;

            // Enrich with nearest place if it drops from a mob/entity with coordinates
            let nearestPlace = null;
            if (item.dropMeta && item.dropMeta.coordinates && places.length > 0) {
                const [dropX, dropY, dropZ] = item.dropMeta.coordinates;
                let minDistance = Infinity;

                for (const place of places) {
                    if (place.x !== undefined && (place.z !== undefined || place.y !== undefined)) {
                        const placeZ = place.z !== undefined ? place.z : place.y; // Sometimes Y is used as Z in older formats
                        const dist = calculateDistance(dropX, dropZ, place.x, placeZ);
                        if (dist < minDistance) {
                            minDistance = dist;
                            nearestPlace = { name: place.name, distance: Math.round(dist) };
                        }
                    }
                }
            }

            // Construct normalized ingredient object
            ingredients.push({
                internalName: internalName,
                name: item.name || internalName,
                tier: item.tier || "Normal",
                requirements: item.requirements || {},
                identifications: item.identifications || {},
                consumableOnlyIDs: item.consumableOnlyIDs || {},
                ingredientPositionModifiers: item.ingredientPositionModifiers || {},
                dropMeta: item.dropMeta || null,
                nearestPlace: nearestPlace,
                icon: item.icon || null
            });
        }

        // 4. Save to static JSON file
        const outDir = path.join(__dirname, '..', 'public', 'data');
        await fs.mkdir(outDir, { recursive: true });
        
        const outFile = path.join(outDir, 'ingredients.json');
        await fs.writeFile(outFile, JSON.stringify(ingredients, null, 2));
        
        console.log(`✅ Successfully saved ${ingredients.length} ingredients to ${outFile}`);

    } catch (error) {
        console.error("❌ Data refresh failed:", error);
        process.exit(1);
    }
}

run();
