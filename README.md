
# Monday.com Board Exporter

A Python script to **export data from a Monday.com board** into CSV files.  
It fetches all items (with pagination) and generates:

- `monday_board.csv` → full dataset with all required columns  
- `monday_board_duplicates.csv` → only rows with duplicate **Client IDs (PK)**  

---

## 🚀 Features
- Connects to the Monday.com GraphQL API
- Supports pagination (large boards)
- Exports clean CSV with consistent column structure
- Detects duplicates in a key column (`ID Client (PK)`)

---

## 🔑 Configuration

You need a Monday.com API token and your Board ID.

Set them as environment variables:

- export MONDAY_API_KEY="your_api_key_here"
- export MONDAY_BOARD_ID="1234567890"
