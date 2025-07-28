# UrbanCanvasAI
Urban Canvas AI turns Google's map into a time machine. It's an immersive app where you can fly through a city's 3D landscape and discover 20 years of crowd-sourced memories—first dates, lost landmarks, personal stories.  We use AI to help anyone instantly craft a beautiful narrative from just an old photo.
The Concept: A platform that transforms Google Maps into a 4D time capsule of human experience. We're not just mapping places; we're mapping the memories, stories, and emotions tied to them over the past 20 years. It’s a collaborative, living atlas of personal history, enhanced with AI and immersive 3D.

Imagine looking at a 3D model of your current location in HCMC, like the area around the Opera House. You could filter by "2008" and see ghostly, glowing pins appear on the map. Tapping one reveals a photo of a couple on their first date at a now-closed cafe, with an AI-generated narrative that captures the feeling of that moment.

Primary Categories:

Art of the Map: The map is our canvas for emotional data visualization.

Travel: It offers tourists and locals an authentic, profound way to discover a city's soul.

AI: We're using AI not as a gimmick, but to fundamentally enhance storytelling.

Immersive: We will use 3D to make exploring these memories feel like a cinematic journey.

Building the Winning Features (Based on Judging Criteria)

1. The Immersive Canvas (Immersive & Art of the Map)

To create a stunning user experience and visualization, we'll build on a foundation of Google's most immersive products.

Photorealistic 3D Tiles: The entire experience will be built on the Maps JavaScript API's 3D Tiles. When a user explores, they're not panning a flat map; they're flying through a photorealistic 3D model of the city.

Cinematic Camera: Clicking a memory pin triggers a smooth, cinematic flight to that location using map.moveCamera(), swooping down to the exact building or spot. For parks or landmarks, we can trigger the Aerial View API for a breathtaking establishing shot.

Custom Styling: The base map will use custom styling to evoke a sense of nostalgia—slightly desaturated, with artistic fonts, inspired by vintage cartography like the Terrain Highlighting winner.

2. The AI Memory Weaver (AI & Content)

To make content creation seamless and beautiful, we integrate AI at the core. This addresses technical execution.

Gemini-Powered Narratives: A user can drop a pin, upload a photo from 2010, and provide a few simple keywords like "Rainy evening," "Best phở of my life," "Nguyễn Trãi street." The Gemini model via the Places API will analyze the photo, understand the location's context, and draft a short, poetic narrative.

Example Output: "On a rain-slicked evening in 2010, the neon lights of Nguyễn Trãi street blurred into watercolor. Here, at a small, steamy stall, was a bowl of phở so perfect it became a memory, a warm anchor in the heart of a bustling Saigon."

Thematic Clustering: Our backend AI will analyze all memories and automatically tag them with themes ("First Dates," "Street Food Discoveries," "Lost Landmarks"). This allows users to explore the map thematically, providing deep, insightful content visualization.

3. The Time-Traveler View (Travel & Functionality)

This is the core feature that directly celebrates the 20-year anniversary and drives the purpose of the app.

Historical Street View Integration: When viewing a memory, if the date is old enough, a "Then & Now" button appears. This opens a split-screen view showing the current Street View next to the historical Street View imagery from the time of the memory. This is a powerful, direct use of Google Maps' historical data archive.

Data-Driven Heatmaps: Using the Deck.gl integration, we'll render a dynamic heatmap showing the emotional pulse of the city—the most "cherished" locations. A timeline slider will animate this heatmap, showing how the city's centers of memory have shifted over 20 years. This offers scalable, region-agnostic functionality.

{
  
  "canisters": {
    "urban_canvas_ai": {
      "type": "custom",
      "candid": "urban_canvas_ai.did",
      "wasm": "urban_canvas_ai.wasm",
      "build": "cargo build --target wasm32-unknown-unknown --release --package urban_canvas_ai",
      "root": "src/urban_canvas_ai",
      "dependencies": []
    }
  },
  "defaults": {
    "build": {
      "packtool": "mops"
    }
  },
  "output_env_file": ".env",
  "version": 1
}
