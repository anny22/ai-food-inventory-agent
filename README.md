# ai-food-inventory-agent
An automated household food inventory and meal-planning agent built with n8n (Docker), Google Sheets, Apps Script, and Google Gemini. Generates weekly 7-day dinner plans (~1,700 kcal/day) and shopping lists from pantry priorities and store promotions, deducts stock when meals are cooked, and includes an in-progress receipt scanner.
This project acts as a smart background assistant that uses your pantry stock, dietary preferences, and weekly supermarket promotions to generate a 7-day dinner plan (~1,700 kcal/day target, ~500–600 kcal dinners) and email you a shopping list. When you cook, marking a meal as cooked automatically deducts the ingredients from your inventory; when you shop, scanning a receipt restocks your inventory.

Project Status

Google Sheets & Apps Script: Completed — 7-tab database structure, custom sidebar entry form, and an onEdit trigger that automatically deducts ingredients and logs consumption when a meal status changes to Cooked (Skuhano).

Workflow A (Weekly Meal Planner & Shopping List): Completed & Working — reads pantry priorities, dietary preferences, and weekly promotions, merges and aggregates data, generates a structured 7-day dinner plan with Google Gemini, writes the schedule to Google Sheets, and emails the shopping list via Gmail.

Workflow B (Receipt Scanner & Restocking — Work in Progress): Google Drive folder trigger and binary file download are configured; connecting direct binary image input into Gemini Vision, updating the receipt mapping dictionary, and handling new items are scheduled next.

System Architecture

Google Sheets (Database & UI):

Food Inventory (Inventar hrane): Current stock, units (g, ml, pcs), unit/total prices, expiration dates, and consumption priority flags (Priority 1 = use first).

Recipe Database (Baza receptov): Structured recipes using machine-readable ingredient norms (KON-029: 1, TES-021: 120) and calorie estimates.

Weekly Meal Plan (Tedenski jedilnik): 7-day dinner schedule with status dropdowns (Planned / Cooked).

Preferences (Preference): User-defined rules (daily calorie target, weekly budget, max meat/fish meals per week).

Store Promotions (Eurospin akcije): Weekly promotional catalogue items and prices.

Receipt Dictionary (Slovar_racunov): Lookup table mapping cryptic receipt text to internal ITEM_IDs and package sizes.

Consumption Log (Dnevnik_porabe): Audit log of all PURCHASE and CONSUMPTION transactions.

n8n Workflows:

Workflow A (01-weekly-meal-planner.json): Reads inventory, preferences, and promotions -> Merge + Aggregate -> Gemini LLM chain -> Code parser -> writes 7 days to Weekly Meal Plan -> sends shopping list via Gmail.

Workflow B (02-receipt-scanner-wip.json — WIP): Watches a Google Drive folder for new receipt photos/PDFs -> downloads binary file -> Gemini Vision extraction -> matches against Receipt Dictionary & Food Inventory -> updates stock.

Google Apps Script:

Custom sidebar form for manual item entry with automatic ID generation.

onEdit event handler for automatic ingredient deduction upon cooking.

Repository Structure

ai-food-inventory-agent/
├── README.md
├── .gitignore
├── .env.example
├── apps-script/
│   ├── Code.gs                        # To be added
│   └── Sidebar.html                   # To be added
├── n8n-workflows/
│   ├── 01-weekly-meal-planner.json    # Exported working Workflow A
│   └── 02-receipt-scanner-wip.json    # To be added after testing
└── docs/
    └── sheets-schema.md               # Tab & column reference


Setup & Running Locally

1. Run n8n in Docker

Start n8n locally with a persistent volume so workflows and credentials are preserved across container restarts:

docker run -d --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n docker.n8n.io/n8nio/n8n


Access the editor at http://localhost:5678.

2. Configure Credentials

In Google Cloud Console, enable the Google Sheets API, Google Drive API, and Gmail API, and create OAuth2 credentials. In n8n, configure:

Google Sheets OAuth2

Gmail OAuth2

Google Drive OAuth2

Google Gemini API Key

3. Import Workflows

Import n8n-workflows/01-weekly-meal-planner.json into n8n (exported workflows exclude credentials by default), select your Google Spreadsheet document in the Sheets nodes, and run the workflow.

Security & Privacy

Never commit .env, OAuth client secrets, tokens, personal receipt images, or private Spreadsheet IDs. Always rely on .gitignore and n8n's built-in credential manager.
