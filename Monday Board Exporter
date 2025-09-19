#!/usr/bin/env python
# coding: utf-8

# In[ ]:


import requests
import json
import csv
import os
from collections import Counter

# === CONFIGURATION ===
# Replace with your actual Monday.com API token
API_KEY = os.getenv("MONDAY_API_KEY", "your_api_key_here")
# Replace with your Monday.com Board ID
BOARD_ID = int(os.getenv("MONDAY_BOARD_ID", "0000000000"))

API_URL = "https://api.monday.com/v2"
HEADERS = {
    "Authorization": API_KEY,
    "Content-Type": "application/json"
}

# === Required columns in the specified order ===
required_columns = [
    "Group", "Name", "ID Client (PK)", "ETC................"
]

# === Function to fetch all items using pagination ===
def fetch_all_items(board_id):
    all_items = []
    cursor = None
    while True:
        query = {
            "query": f"""
            {{
              boards(ids: {board_id}) {{
                items_page(limit: 500, cursor: {json.dumps(cursor) if cursor else "null"}) {{
                  cursor
                  items {{
                    id
                    name
                    group {{
                      title
                    }}
                    column_values {{
                      id
                      text
                      value
                      column {{
                        title
                      }}
                      ... on BoardRelationValue {{
                        linked_item_ids
                        linked_items {{
                          id
                          name
                          updated_at
                        }}
                      }}
                    }}
                  }}
                }}
              }}
            }}
            """
        }
        response = requests.post(API_URL, headers=HEADERS, json=query)
        data = response.json()
        try:
            page = data["data"]["boards"][0]["items_page"]
            items = page["items"]
            all_items.extend(items)
            cursor = page["cursor"]
            if not cursor:
                break
        except Exception as e:
            print("❌ Error processing API response:", e)
            print(json.dumps(data, indent=2))
            break
    return all_items

# === Fetch all items from the board ===
items = fetch_all_items(BOARD_ID)

# === Process items into flat rows for CSV ===
flat_rows = []
id_client_pk_list = []

for item in items:
    row = {col: "" for col in required_columns}
    row["Name"] = item["name"]
    row["Group"] = item["group"]["title"] if "group" in item and item["group"] else ""
    row["Item ID"] = item["id"]

    for col in item["column_values"]:
        title = col["column"]["title"]
        if title in required_columns:
            if "linked_items" in col and col["linked_items"]:
                linked_names = [linked["name"] for linked in col["linked_items"]]
                value = ", ".join(linked_names)
            elif col["text"]:
                value = col["text"]
            else:
                value = col.get("value") or ""
            row[title] = value
            if title == "ID Client (PK)":
                id_client_pk_list.append(value)
    flat_rows.append(row)

# === Save main CSV file ===
output_path_main = os.path.join(os.getcwd(), "monday_board.csv")
with open(output_path_main, "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=required_columns)
    writer.writeheader()
    for row in flat_rows:
        writer.writerow(row)

# === Identify duplicates in 'ID Client (PK)' ===
duplicates = {item for item, count in Counter(id_client_pk_list).items() if count > 1}

# === Create second CSV with duplicates only ===
duplicate_rows = []
for row in flat_rows:
    if row["ID Client (PK)"] in duplicates:
        duplicate_rows.append({
            "ID Client (PK)": row["ID Client (PK)"],
            "Name": row["Name"],
            "Group": row["Group"]
        })

# === Save duplicates CSV file ===
output_path_duplicates = os.path.join(os.getcwd(), "monday_board_duplicates.csv")
with open(output_path_duplicates, "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["ID Client (PK)", "Name", "Group"])
    writer.writeheader()
    for row in duplicate_rows:
        writer.writerow(row)

print(f"✅ Main CSV file saved to: {output_path_main}")
print(f"✅ Duplicates CSV file saved to: {output_path_duplicates}")

