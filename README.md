# Script to convert file names to slug value

```py
import os
import json

# Path to your folder
folder_path = "./" 

for filename in os.listdir(folder_path):
    if filename.endswith(".json"):
        file_path = os.path.join(folder_path, filename)
        
        # Get slug from filename (without extension)
        slug = os.path.splitext(filename)[0]
        
        # Read JSON file
        with open(file_path, "r", encoding="utf-8") as f:
            try:
                data = json.load(f)
            except json.JSONDecodeError as e:
                print(f"⚠️ Skipping {filename}, invalid JSON: {e}")
                continue
        
        # Update slug
        data["slug"] = slug
        
        # Write JSON back
        with open(file_path, "w", encoding="utf-8") as f:
            json.dump(data, f, indent=4, ensure_ascii=False)
        
        print(f"✅ Updated slug for {filename} -> {slug}")
```
---
# Script to convert html title tag to title value in jSON
```py
import os
import re
import json

# Path to your folder
folder_path = "./" 

# Regex to grab text inside <title> tags
title_pattern = re.compile(r"<title>(.*?)</title>", re.IGNORECASE | re.DOTALL)

for filename in os.listdir(folder_path):
    if filename.endswith(".html"):
        html_path = os.path.join(folder_path, filename)

        # Remove the random differentiator from the HTML filename
        # e.g. pakistani-culture-night-1b504a8494.html -> pakistani-culture-night
        base_name = re.sub(r"-[a-f0-9]{8,}$", "", os.path.splitext(filename)[0])

        json_filename = f"{base_name}.json"
        json_path = os.path.join(folder_path, json_filename)

        if not os.path.exists(json_path):
            print(f"⚠️ No JSON found for {filename}")
            continue

        # Extract title from HTML
        with open(html_path, "r", encoding="utf-8") as f:
            html_content = f.read()

        match = title_pattern.search(html_content)
        if not match:
            print(f"⚠️ No <title> found in {filename}")
            continue

        html_title = match.group(1).strip()

        # Update JSON
        with open(json_path, "r", encoding="utf-8") as f:
            try:
                data = json.load(f)
            except json.JSONDecodeError as e:
                print(f"⚠️ Skipping {json_filename}, invalid JSON: {e}")
                continue

        data["title"] = html_title

        with open(json_path, "w", encoding="utf-8") as f:
            json.dump(data, f, indent=4, ensure_ascii=False)

        print(f"✅ Updated title for {json_filename} -> {html_title}")
```
---
# Script to extract remaining data from index.json and warn for missing ones
```py
import os
import json

# Path to folder
folder_path = "./" 
index_path = os.path.join(folder_path, "index.json")

# Load index.json
with open(index_path, "r", encoding="utf-8") as f:
    try:
        index_data = json.load(f)
    except json.JSONDecodeError as e:
        print(f"❌ Invalid JSON in index.json: {e}")
        exit(1)

articles = index_data.get("articles", [])

for article in articles:
    slug = article.get("slug")
    if not slug:
        continue

    json_filename = f"{slug}.json"
    json_path = os.path.join(folder_path, json_filename)

    if not os.path.exists(json_path):
        print(f"⚠️ No JSON file found for slug '{slug}'")
        continue

    # Load the individual file
    with open(json_path, "r", encoding="utf-8") as f:
        try:
            data = json.load(f)
        except json.JSONDecodeError as e:
            print(f"⚠️ Skipping {json_filename}, invalid JSON: {e}")
            continue

    # Preserve content, update everything else except excerpt
    content_value = data.get("content", "")

    # Copy over fields from index.json (ignore excerpt)
    for key in ["title", "slug", "date", "type", "duration", "src"]:
        if key in article:
            data[key] = article[key]

    # Restore content
    data["content"] = content_value

    # Save updated JSON file
    with open(json_path, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=4, ensure_ascii=False)

    print(f"✅ Updated {json_filename} with metadata from index.json")
```
---
# Script for extracting contents from `<div class="article-content"> ... </div>` into content value
```py
import os
import re
import json
import unicodedata

# Path to folder
folder_path = "./"  

# Regex to grab <div class="article-content">...</div>
content_pattern = re.compile(
    r'<div\s+class="article-content">(.*?)</div>',
    re.IGNORECASE | re.DOTALL
)

# Function to normalize Turkish chars → ASCII for matching
def normalize_filename(name: str) -> str:
    # Decompose unicode chars (e.g., ş -> ş)
    normalized = unicodedata.normalize("NFD", name)
    # Filter out diacritics
    normalized = "".join(c for c in normalized if unicodedata.category(c) != "Mn")
    # Lowercase, keep it safe
    return normalized.lower()

for filename in os.listdir(folder_path):
    if filename.endswith(".html"):
        html_path = os.path.join(folder_path, filename)

        # Base name without extension & random differentiator
        base_name = re.sub(r"-[a-f0-9]{8,}$", "", os.path.splitext(filename)[0])

        # Normalize for matching
        normalized_base = normalize_filename(base_name)

        # Find matching JSON
        json_filename = f"{normalized_base}.json"
        json_path = os.path.join(folder_path, json_filename)

        if not os.path.exists(json_path):
            print(f"⚠️ No JSON found for HTML file {filename} (expected {json_filename})")
            continue

        # Extract content from HTML
        with open(html_path, "r", encoding="utf-8") as f:
            html_content = f.read()

        match = content_pattern.search(html_content)
        if not match:
            print(f"⚠️ No <div class='article-content'> found in {filename}")
            continue

        inner_html = match.group(1).strip()

        # Update JSON
        with open(json_path, "r", encoding="utf-8") as f:
            try:
                data = json.load(f)
            except json.JSONDecodeError as e:
                print(f"⚠️ Skipping {json_filename}, invalid JSON: {e}")
                continue

        data["content"] = inner_html

        with open(json_path, "w", encoding="utf-8") as f:
            json.dump(data, f, indent=4, ensure_ascii=False)

        print(f"✅ Updated content for {json_filename}")
```
---
# The rest of the editing and checking was done manually
