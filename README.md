# Ego — Local AI Assistant

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux-0078D6?logo=windows&logoColor=white)
![GUI](https://img.shields.io/badge/GUI-CustomTkinter-1f6feb)
![Backends](https://img.shields.io/badge/Backends-llama.cpp%20%7C%20OpenRouter-2ea043)

Ego is a desktop AI chat assistant built with Python and CustomTkinter. It runs
entirely on your own computer and can talk to an AI model in **two ways**:

- **llama.cpp** — a fully **offline, local** model running on your own PC.
- **OpenRouter** — an **online** API that gives you access to many hosted models.

Ego also has long-term memory, permanent conversation history, and a `/search`
command that looks things up on Wikipedia (when the internet is available) and
summarises the answer.

---

## Table of Contents

- [About](#about)
- [Features](#features)
- [How the two backends work](#how-the-two-backends-work)
- [Commands](#commands)
- [Project files](#project-files)
- [Functions reference](#functions-reference)
- [Installation](#installation)
  - [Windows](#windows)
  - [Linux](#linux)
- [Setting up the AI backend](#setting-up-the-ai-backend)
  - [Option A — llama.cpp (offline, local)](#option-a--llamacpp-offline-local)
  - [Option B — OpenRouter (online)](#option-b--openrouter-online)
- [Troubleshooting](#troubleshooting)
- [License](#license)
- [Credits](#credits)

---

## About

Ego is a single-file Python application with a modern dark GUI. It keeps three
kinds of data:

1. **Temporary context** — the recent messages used to keep the current reply
   coherent. Cleared when the app closes.
2. **Permanent conversation history** — every chat is saved and shown in the
   sidebar so you can reopen it later.
3. **Long-term memory** — small, stable facts about you (your name, preferences,
   projects, etc.) that Ego learns and reuses in future chats.

All data is stored as simple JSON files **next to `main.py`**. These files are
created automatically the first time you launch the app, so a fresh download
needs nothing pre-made.

---

## Features

- Modern CustomTkinter dark interface with chat bubbles.
- **Two backends**: local `llama.cpp` server, or online `OpenRouter`.
- Switch backends at any time from the in-app **Settings** window.
- **Long-term memory** — Ego decides what is worth remembering and can forget
  things on request.
- **Permanent conversation history** with a sidebar (open, rename, delete chats).
- **`/search` command** — resolves your question using the chat context,
  fetches the matching Wikipedia article, and summarises it.
- Command autocomplete (type `/` to see available commands).
- Fullscreen toggle with **F11**.

---

## How the two backends work

Ego sends every request through one function, `request_ai()`, which chooses the
backend based on your settings:

| Backend | Type | Endpoint | Needs internet? |
|---|---|---|---|
| `llama.cpp` | Local | `http://127.0.0.1:8080/v1/chat/completions` | No |
| `openrouter` | Online | `https://openrouter.ai/api/v1/chat/completions` | Yes |

You choose which one to use in the app's **Settings** window, or by editing
`settings.json`:

```json
{
  "provider": "llama.cpp",
  "llama_model": "qwen2.5-3b",
  "openrouter_api_key": "",
  "openrouter_model": "openrouter/free"
}
```

- `provider` — `"llama.cpp"` or `"openrouter"`.
- `llama_model` — any name you like; llama.cpp serves whatever model you loaded.
- `openrouter_api_key` — your OpenRouter key (only needed for the online backend).
- `openrouter_model` — an OpenRouter model id, e.g. `openrouter/free`.

---

## Commands

Type these in the message box:

| Command | What it does |
|---|---|
| `/search <question>` | Look up the topic on Wikipedia and summarise it. |
| `/clear` | Clear the current conversation. |
| `/memory` | Show everything Ego has saved in long-term memory. |
| `/forget <fact>` | Delete a specific saved memory. |

Typing `/` shows a dropdown of the available commands.

---

## Project files

| File | Purpose | Created when |
|---|---|---|
| `main.py` | The whole application. | You download it |
| `requirements.txt` | Python dependencies. | You download it |
| `settings.json` | Backend + API-key settings. | First launch (or shipped) |
| `chat_history.json` | Temporary context (cleared on close). | First launch |
| `conversation_history.json` | All saved conversations. | First launch |
| `memory.json` | Long-term memory. | First launch |
| `ego_logo.png` | Window icon. | You download it |
| `Ego.sh` | Linux/macOS launcher script. | You download it |

---

## Functions reference

Ego is organised into clear groups. Below is every function and what it does.

### Settings

- `load_settings()` — reads `settings.json` and fills in defaults for any
  missing values.
- `save_settings()` — writes settings to `settings.json` atomically.
- `open_settings()` — opens the Settings window (backend choice, OpenRouter key
  and model). Contains a nested `save_and_close()`.

### Utilities

- `generate_conversation_id()` — makes a unique id for a new conversation.
- `current_timestamp()` — returns the current time as a readable string.

### Temporary context

- `load_temp_context()` — loads the recent working context from
  `chat_history.json`.
- `save_temp_context()` — saves the current context.
- `clear_temp_context()` — empties the temporary context when the app closes.

### Long-term memory

- `load_memory()` — loads saved facts from `memory.json`.
- `save_memory()` — saves the memory list (keeps the most recent 50).
- `add_memory(fact)` — adds a new fact (ignores duplicates and very long text).
- `remove_memory(fact)` — deletes a fact by value.
- `memory_text()` — formats all memories for the system prompt.
- `ai_memory_extract(user_text)` — asks the AI whether a user message contains a
  stable fact worth remembering, and saves it if so.

### Permanent conversation history

- `load_conversation_history()` — loads all saved conversations.
- `save_conversation_history()` — writes all conversations to disk.
- `find_conversation(conversation_id)` — looks up a conversation by id.
- `make_conversation_title(messages)` — builds a short title from the first
  user message.
- `create_conversation()` — starts a new conversation.
- `save_current_conversation()` — saves the currently open conversation.
- `update_current_conversation()` — convenience wrapper around the above.
- `render_conversation()` — draws all messages of the open conversation.
- `open_conversation(conversation_id)` — opens a saved conversation.
- `new_chat()` — saves the current chat and starts a fresh one.
- `delete_conversation(conversation_id)` — deletes a conversation (with a
  confirmation dialog; nested `cancel_delete()` and `confirm_delete()`).
- `rename_conversation(conversation_id)` — renames a conversation.
- `refresh_history_sidebar()` — rebuilds the sidebar conversation list.

### Sidebar menus

- `show_history_menu(conversation_id, event)` — the small ⋮ menu (Rename/Delete).
- `rename_from_menu(menu, conversation_id)` — handles "Rename" from that menu.
- `delete_from_menu(menu, conversation_id)` — handles "Delete" from that menu.

### Talking to the AI

- `get_system_prompt()` — builds the system prompt (current date/time + saved
  memories).
- `request_ai(messages, temperature, max_tokens, timeout)` — the single place
  that sends a request to the chosen backend (llama.cpp or OpenRouter) and
  returns the result or an error.
- `send_message_to_llama(user_message)` — sends a normal chat message and
  returns the assistant's reply.
- `add_message_to_history(role, content)` — appends a message to the current
  conversation and saves it.
- `get_bot_response(user_text)` — worker that runs a normal message off the UI
  thread and shows the reply (nested `update_gui()`).

### Wikipedia `/search`

- `resolve_search_query(search_query)` — rewrites your `/search` request into a
  complete query using the recent chat (so "when was he born" becomes "when was
  Albert Einstein born").
- `wikipedia_search(query)` — finds the matching Wikipedia article and returns
  its title, intro text and URL.
- `summarize_wikipedia_result(query, wiki_result)` — asks the AI to summarise
  the article for your question.
- `search_wikipedia_and_answer(query)` — runs the full `/search` pipeline
  (resolve → fetch → summarise).
- `get_search_response(search_query)` — worker that runs `/search` off the UI
  thread (nested `update_gui()`).

### UI building (bubbles & status)

- `calculate_bubble_width(message, max_width)` — sizes a chat bubble to its text.
- `scroll_to_bottom()` — scrolls the chat to the newest message.
- `insert_user_message(message, scroll)` — draws a user message bubble.
- `insert_ego_message(message, scroll)` — draws an assistant message bubble.
- `show_thinking()` / `remove_thinking()` — show/hide the "Thinking…" bubble
  (nested `create_thinking()`).
- `show_searching()` / `remove_searching()` — show/hide the "Searching…" bubble
  (nested `create_searching()`).

### Commands & memory UI

- `clear_conversation()` — clears the current conversation (used by `/clear`).
- `show_memory()` — prints saved memories into the chat.
- `select_memory_to_forget(fact)` — fills `/forget <fact>` into the input box.
- `update_memory_suggestions()` — shows the list of memories to forget.
- `use_command(command)` — inserts the chosen command into the input box.
- `update_command_suggestions(event)` — shows matching commands as you type.
- `hide_command_suggestions()` — hides that dropdown.
- `handle_escape(event)` — closes the dropdown on Escape.

### Main flow

- `send_message(event)` — the heart of the app. Reads the input, then routes it:
  `/clear`, `/memory`, `/forget`, `/search`, or a normal message.
- `toggle_fullscreen(event)` — F11 fullscreen.
- `on_closing()` — saves everything and closes the app cleanly.
- `startup()` — runs once at launch: creates a fresh conversation, clears the
  temporary context, and draws the interface.

---

## Installation

Follow the section for your operating system.

### Windows

**1. Install Python**

- Download Python 3.10 or newer from <https://www.python.org/downloads/>.
- During install, **tick "Add Python to PATH"**.
- Verify in Command Prompt:
  ```bat
  python --version
  ```

**2. Get the app**

- Put all the files (`main.py`, `requirements.txt`, `ego_logo.png`,
  `settings.json`) in one folder, e.g. `C:\Ego`.

**3. Create a virtual environment**

Open Command Prompt in that folder and run:
```bat
python -m venv .venv
```
This creates a `.venv` folder inside your project.

**4. Activate the virtual environment**

```bat
.venv\Scripts\activate
```
Your prompt now starts with `(.venv)`. (In PowerShell, if activation is
blocked, run `.venv\Scripts\activate` once with
`Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.)

**5. Install the dependencies**

```bat
pip install -r requirements.txt
```

**6. Run Ego**

```bat
python main.py
```
The Ego window should open. The first launch creates the JSON files
automatically.

### Linux

**1. Install Python and Tk**

- Most distros ship Python 3. Check it:
  ```bash
  python3 --version
  ```
- Install the Tk library (needed by CustomTkinter):
  ```bash
  sudo apt install python3-tk        # Debian / Ubuntu
  sudo dnf install python3-tkinter   # Fedora
  ```

**2. Get the app**

- Put all the files (`main.py`, `requirements.txt`, `ego_logo.png`,
  `settings.json`, `Ego.sh`) in one folder, e.g. `~/Ego`.

**3. Create a virtual environment**

Open a terminal in that folder and run:
```bash
python3 -m venv .venv
```

**4. Activate the virtual environment**

```bash
source .venv/bin/activate
```
Your prompt now starts with `(.venv)`.

**5. Install the dependencies**

```bash
pip install -r requirements.txt
```

**6. Run Ego**

Either:
```bash
python main.py
```
or use the included launcher script (make it executable once):
```bash
chmod +x Ego.sh
./Ego.sh
```
The first launch creates the JSON files automatically.

---

## Setting up the AI backend

You only need **one** of the two options below.

### Option A — llama.cpp (offline, local)

This runs a model on your own PC. Nothing leaves your machine.

**A1. Download the llama.cpp server**

- Get a build from the official releases page:
  <https://github.com/ggml-org/llama.cpp/releases>
- **Windows:** download a `llama-*-bin-win-*.zip`, unzip it, and use
  `llama-server.exe`.
- **Linux:** either build from source (`git clone` + `cmake`) or download a
  matching prebuilt archive, then use `llama-server`.

**A2. Download a model (GGUF)**

Ego works with **GGUF** models. Good places to get them:

- **Hugging Face** — <https://huggingface.co/models?search=gguf>
  - `bartowski` — huge library of GGUF quantisations.
  - `lmstudio-community` — clean GGUF builds.
  - `Qwen` — official Qwen GGUF models.
  - `unsloth`, `mradermacher`, `TheBloke` — more options.
- **LM Studio** — <https://lmstudio.ai> (a friendly GUI that can also search and
  download GGUF models).

**Which size for your PC?** Pick a quantisation (Q4_K_M is a good default) that
fits in your RAM (CPU) or VRAM (GPU):

| Model size (Q4) | Rough memory needed | Good for |
|---|---|---|
| 0.5B – 1.5B | ~1 – 1.5 GB | Very low-end PCs, CPU-only |
| 3B | ~2 – 2.5 GB | 4 GB RAM |
| 7B – 8B | ~5 – 6 GB | 8 GB RAM (a sweet spot) |
| 13B – 14B | ~8 – 9 GB | 12 – 16 GB RAM |
| 30B – 32B | ~18 – 20 GB | 32 GB RAM |
| 70B | ~40 GB | 64 GB RAM / multi-GPU |

If you have a GPU, the model should fit in **VRAM** for full speed; otherwise
part of it runs on the CPU (slower). Start small and go bigger if it feels
sluggish.

**A3. Start the server**

Ego expects the server on **port 8080**. Start it before opening Ego.

- **Windows** (Command Prompt, adjust the path):
  ```bat
  llama-server.exe -m "C:\models\qwen2.5-3b-instruct-q4_k_m.gguf" -c 4096 --port 8080
  ```
- **Linux**:
  ```bash
  ./llama-server -m ~/models/qwen2.5-3b-instruct-q4_k_m.gguf -c 4096 --port 8080
  ```

`-c 4096` sets the context length (you can raise it if you have memory to
spare). Leave this terminal open while using Ego.

**A4. Point Ego at llama.cpp**

In Ego's **Settings**, choose **llama.cpp (Offline)** and save.

### Option B — OpenRouter (online)

This uses hosted models through an API and needs an internet connection.

1. Create a free account at <https://openrouter.ai>.
2. Generate an API key at <https://openrouter.ai/keys>.
3. In Ego's **Settings**:
   - Choose **OpenRouter (Online)**.
   - Paste your API key.
   - Set a model id (e.g. `openrouter/free`, or any id from
     <https://openrouter.ai/models>).
   - Save.

That's it — no local model or server needed.

---

## Troubleshooting

- **"llama.cpp is not running."** — Start `llama-server` first (Option A3) and
  make sure it is on port `8080`.
- **"OpenRouter API key is not configured."** — Add your key in Settings.
- **Window opens but the GUI looks broken / import error for `tkinter`** —
  install Tk (Linux: `sudo apt install python3-tk`).
- **`pip` says "Can not perform a '--user' install"** — you are inside a venv;
  just run `pip install -r requirements.txt` normally (make sure the venv is
  active).
- **Replies are slow** — the model is too big for your hardware; try a smaller
  quantisation or a smaller model.

---

## License

This project is licensed under the **MIT License**.

Copyright (c) 2026 <Nikunj>

See the [LICENSE](LICENSE) file for the full license text.

## Credits

Built with [CustomTkinter](https://github.com/TomSchimansky/CustomTkinter),
[llama.cpp](https://github.com/ggml-org/llama.cpp),
[OpenRouter](https://openrouter.ai) and the
[Wikipedia API](https://www.mediawiki.org/wiki/API:Main_page).
