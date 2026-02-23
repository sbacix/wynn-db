/**
 * Imperial Archive Utility: Data Refresh Script (Streamlined)
 * Merges official Wynncraft API v3 data with the 32k line community_coords.json.
 */
const fs = require('fs/promises');
const path = require('path');

const WYNN_API_URL = 'https://api.wynncraft.com/v3/item/database?fullResult';

async function run() {
    console.log("🚀 Imperial Bot Engaged (Streamlined Mode)...");

    try {
        // 1. Fetch official stats (Tier, Level, IDs)
        console.log("📥 Fetching official Wynncraft v3 database...");
        const itemRes = await fetch(WYNN_API_URL);
        if (!itemRes.ok) throw new Error(`API error: ${itemRes.status}`);
        const itemsMap = await itemRes.json();

        // 2. Load Community Coords (The 32k line file)
        let communityData = {};
        try {
            const commFile = await fs.readFile(path.join(__dirname, '../data/community_coords.json'), 'utf8');
            communityData = JSON.parse(commFile);
            console.log("📦 Community intel loaded successfully.");
        } catch (e) {
            console.warn("⚠️ data/community_coords.json not found. Check your file path.");
        }

        const ingredients = [];

        // 3. Process and Merge
        for (const [internalName, item] of Object.entries(itemsMap)) {
            // Check if item is an ingredient
            const isIngredient = (item.requirements?.skills?.length > 0) || 
                                 item.consumableOnlyIDs || 
                                 item.ingredientPositionModifiers;
            
            if (!isIngredient) continue;

            const itemName = item.name || internalName;

            // Priority: Use Community JSON intel, fallback to basic API dropMeta
            const communitySources = communityData[itemName] || communityData[internalName] || null;

            ingredients.push({
                internalName: internalName,
                name: itemName,
                tier: item.tier || "Normal",
                requirements: item.requirements || {},
                identifications: item.identifications || {},
                communitySources: communitySources, 
                dropMeta: item.dropMeta || null
            });
        }

        // 4. Save Final Asset Package
        const outDir = path.join(__dirname, '..', 'public', 'data');
        await fs.mkdir(outDir, { recursive: true });
        await fs.writeFile(path.join(outDir, 'ingredients.json'), JSON.stringify(ingredients));
        
        console.log(`✅ Success. ${ingredients.length} assets logged in the Imperial Archive.`);
    } catch (error) {
        console.error("❌ Critical failure during data synchronization:", error);
        process.exit(1);
    }
}

run();
