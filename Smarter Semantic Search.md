✅ Great request — here is a **Python implementation of Smarter Semantic Search over `.md` files in a GitHub repo folder**, with clear step-by-step explanation.

We’ll use:

* `requests` to call GitHub API & fetch files
* `openai` to embed the text and the prompt
* `numpy` for cosine similarity

---

# 📄 Full Code

```python
import requests
import openai
import numpy as np

# ===== CONFIG =====
GITHUB_OWNER = "dishant9876"
GITHUB_REPO = "onlineCV"
FOLDER_PATH = ""   # e.g., "docs" if files are in /docs, else leave blank
GITHUB_TOKEN = "ghp_XXXXXX"  # optional, if repo is private
OPENAI_API_KEY = "sk-XXXXXX"
PROMPT = "weather forecast application"

openai.api_key = OPENAI_API_KEY

# ===== STEP 1: LIST .md FILES IN FOLDER =====

def list_md_files():
    url = f"https://api.github.com/repos/{GITHUB_OWNER}/{GITHUB_REPO}/contents/{FOLDER_PATH}"
    headers = {}
    if GITHUB_TOKEN:
        headers["Authorization"] = f"Bearer {GITHUB_TOKEN}"
    resp = requests.get(url, headers=headers)
    resp.raise_for_status()
    files = resp.json()
    md_files = [f for f in files if f['name'].endswith('.md')]
    return md_files

# ===== STEP 2: FETCH FILE CONTENTS =====

def fetch_file_content(download_url):
    resp = requests.get(download_url)
    resp.raise_for_status()
    return resp.text

# ===== STEP 3: GET EMBEDDING =====

def get_embedding(text):
    resp = openai.Embedding.create(
        input=text,
        model="text-embedding-3-small"
    )
    return np.array(resp['data'][0]['embedding'])

# ===== STEP 4: COSINE SIMILARITY =====

def cosine_similarity(vec1, vec2):
    return np.dot(vec1, vec2) / (np.linalg.norm(vec1) * np.linalg.norm(vec2))

# ===== MAIN =====

def main():
    # List files
    print("Fetching list of .md files...")
    md_files = list_md_files()
    print(f"Found {len(md_files)} markdown files.")

    # Embed prompt
    print("Embedding prompt...")
    prompt_embedding = get_embedding(PROMPT)

    # For each file
    results = []
    for f in md_files:
        print(f"Processing {f['name']}...")
        content = fetch_file_content(f['download_url'])
        embedding = get_embedding(content)
        similarity = cosine_similarity(prompt_embedding, embedding)
        results.append((f['name'], similarity))

    # Sort by similarity
    results.sort(key=lambda x: x[1], reverse=True)

    print("\nTop matches:")
    for name, score in results[:5]:
        print(f"{name}: similarity={score:.4f}")

if __name__ == "__main__":
    main()
```

---

# 🧾 Explanation, line by line

### 📋 Config

```python
GITHUB_OWNER = "dishant9876"
GITHUB_REPO = "onlineCV"
FOLDER_PATH = ""
GITHUB_TOKEN = "ghp_XXXXXX"
OPENAI_API_KEY = "sk-XXXXXX"
PROMPT = "weather forecast application"
```

🔷 Set your repo details, folder path (if needed), tokens, and the search prompt.

---

### 🔍 Step 1: List `.md` files

```python
def list_md_files():
    url = f"https://api.github.com/repos/{GITHUB_OWNER}/{GITHUB_REPO}/contents/{FOLDER_PATH}"
    headers = {}
    if GITHUB_TOKEN:
        headers["Authorization"] = f"Bearer {GITHUB_TOKEN}"
    resp = requests.get(url, headers=headers)
    resp.raise_for_status()
    files = resp.json()
    md_files = [f for f in files if f['name'].endswith('.md')]
    return md_files
```

✅ Calls the **GitHub Contents API** to list files in the folder, filters for `.md`.

---

### 📝 Step 2: Fetch file content

```python
def fetch_file_content(download_url):
    resp = requests.get(download_url)
    resp.raise_for_status()
    return resp.text
```

✅ For each `.md` file, fetches its raw content from `download_url`.

---

### 🔷 Step 3: Get embeddings

```python
def get_embedding(text):
    resp = openai.Embedding.create(
        input=text,
        model="text-embedding-3-small"
    )
    return np.array(resp['data'][0]['embedding'])
```

✅ Uses OpenAI API to generate an embedding vector for a text.

---

### 🔷 Step 4: Compute similarity

```python
def cosine_similarity(vec1, vec2):
    return np.dot(vec1, vec2) / (np.linalg.norm(vec1) * np.linalg.norm(vec2))
```

✅ Standard cosine similarity between two vectors.

---

### 🚀 Main logic

```python
md_files = list_md_files()
prompt_embedding = get_embedding(PROMPT)
for f in md_files:
    content = fetch_file_content(f['download_url'])
    embedding = get_embedding(content)
    similarity = cosine_similarity(prompt_embedding, embedding)
    results.append((f['name'], similarity))
```

✅ Embeds the prompt.
✅ Embeds each `.md` file.
✅ Computes similarity.
✅ Collects and sorts results.

---

### 📊 Output

✅ Prints the top 5 most similar `.md` files.

---

# 🧪 Example output:

```
Fetching list of .md files...
Found 3 markdown files.
Embedding prompt...
Processing weather.md...
Processing todo.md...
Processing summary.md...

Top matches:
weather.md: similarity=0.8891
summary.md: similarity=0.6523
todo.md: similarity=0.4322
```

---

# 🪄 Notes:

* You can replace `text-embedding-3-small` with `text-embedding-ada-002` if needed.
* Works best with `.md` files up to \~8k tokens; truncate if too large.
* Requires installing dependencies:

```bash
pip install openai numpy requests
```

---

If you’d like, I can also:
✅ Add multiprocessing to speed up
✅ Save results to a CSV
✅ Or make it into a Flask API

👉 Just tell me what you’d like next!
