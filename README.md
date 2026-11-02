# Jarvis (local-first Windows starter)

**Version:** `0.2.0`

A voice-controlled desktop assistant for Windows. Speech recognition and language-model requests run on this PC. Google is contacted only when you request a Gmail or Calendar action; message and event data are summarized by the local model.

## What this starter can do

- Keep an offline wake-word listener active while Jarvis is running. Say â€œJarvisaâ€ followed by a command; â€œJarvisâ€ and â€œHey Jarvisâ€ are accepted as recognition variants.
- Open a small allowlist of apps and control media playback/volume.
- Search file names in Documents, Desktop, and Downloads; summarize a selected TXT, Markdown, CSV, PDF, or DOCX file.
- Read unread Gmail messages, search Gmail, and list upcoming Google Calendar events.
- Create Gmail drafts and add Google Calendar events after an on-screen confirmation.
- Keep a local to-do list, set countdown reminders, and ask for the current time and basic PC/storage status.
- Speak responses using Windows' local speech voices.

Jarvis can read Gmail and Calendar, create Gmail drafts, and add Calendar events. It never sends email, edits or deletes calendar events, deletes files, or runs arbitrary commands. Every Google write requires a Windows confirmation dialog. The microphone is active while Jarvis is running; press **M** in its console to pause or resume it, or **Q** to quit.

Tasks and reminders are stored in `%LOCALAPPDATA%\Jarvis\jarvis.db`. Reminders show a Windows popup while Jarvis is running; reminders that become due while Jarvis is closed appear the next time it starts.

Example requests:

- â€œJarvisa, add a task to pay the electricity bill.â€
- â€œJarvisa, what are my open tasks?â€
- â€œJarvisa, mark task 2 complete.â€
- â€œJarvisa, remind me in 20 minutes to stretch.â€
- â€œJarvisa, what reminders are pending?â€
- â€œJarvisa, cancel reminder 2.â€
- â€œJarvisa, how much free disk space do I have?â€
- â€œJarvisa, draft an email to alex@example.com saying I will be ten minutes late.â€
- â€œJarvisa, add a dentist appointment tomorrow at 3 PM.â€

## Requirements

- Windows 10 or later.
- Python 3.10.7 or newer.
- [Ollama for Windows](https://ollama.com/download/windows), with a local model. The starter defaults to `qwen3:4b`; Ollama lists this model at about 2.5 GB, and it needs additional memory while running. If it is too slow, choose a smaller model in `JARVIS_OLLAMA_MODEL`.
- A microphone and speakers/headphones.
- An internet connection for the first-time downloads of the offline speech models and local Ollama model.
- A Google Cloud project with Gmail API and Google Calendar API enabled, and OAuth desktop credentials (only if you want Google integration).

## Setup

1. Install Python 3.10.7 or newer and Ollama for Windows.
2. Open PowerShell in this folder and install the Python packages:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   python -m pip install --upgrade pip
   python -m pip install -r requirements.txt
   ```

3. Download a local Ollama model:

   ```powershell
   ollama pull qwen3:4b
   ```

   To use a different local model, set its name before launching Jarvis, for example:

   ```powershell
   $env:JARVIS_OLLAMA_MODEL = "qwen3:1.7b"
   ```

   Do not use a model name ending in `:cloud` if you want all AI processing to stay local.

4. (Optional) Set up Google access:
   - In Google Cloud Console, create a project and enable Gmail API and Google Calendar API.
   - Configure the OAuth consent screen for personal testing and add your Google account as a test user if Google asks for one.
   - Create an OAuth client ID for a **Desktop app** and save its downloaded JSON file in this folder as `credentials.json`.
   - The first Gmail or Calendar request opens Google's sign-in/consent page. Jarvis requests read access plus Gmail draft creation and Calendar event creation. If you already authorized the earlier read-only version, Google will ask you to review the expanded permissions again. Its OAuth token is stored under your Windows local app data folder (`%LOCALAPPDATA%\Jarvis`), not in this project.

5. Start Jarvis:

   ```powershell
   python jarvis.py
   ```

   To check setup before starting the microphone listener, run:

   ```powershell
   python jarvis.py --doctor
   ```

   Jarvis begins listening for â€œJarvisaâ€ as soon as it starts. Leave this console window and process running; it does not yet start automatically when you sign in to Windows. The small Vosk wake-word model downloads automatically to `%LOCALAPPDATA%\Jarvis\models` on first launch. The faster-whisper command model downloads locally the first time you say the wake word. These first downloads need internet access; microphone audio is processed locally and held only in memory.

## Privacy and permissions

- Ollama is called only at `http://127.0.0.1:11434`; Jarvis does not call a hosted AI API.
- The wake-word detector and speech-to-text process microphone audio locally. Jarvis does not save microphone recordings. The mic stream runs only while the Jarvis process is open, and **M** stops the stream until resumed.
- Google APIs receive requests only when you use their mail/calendar actions. Gmail and Calendar contents are passed to Ollama on this PC for summaries.
- Draft and event creation require a confirmation dialog. Confirmed drafts and calendar events are sent to Google. Googleâ€™s `gmail.compose` scope includes the API ability to send mail, but Jarvis only creates drafts; `calendar.events` permits event changes, but Jarvis only creates events. Review those permissions before approving.
- File search looks at names only in Documents, Desktop, and Downloads. Summarizing a file reads its contents locally and supports TXT, Markdown, CSV, PDF, and DOCX.
- App launching uses a fixed allowlist. The assistant never executes text as a shell command.
- To-do items and reminders remain on this PC. The reminder scheduler runs only while Jarvis is open.
- This is an early starter. Review the requested Google permissions before approving them, and do not add credentials or token files to source control.

## Settings

Environment variables:

- `JARVIS_OLLAMA_MODEL` â€” local Ollama model; default `qwen3:4b`.
- `JARVIS_WHISPER_MODEL` â€” local faster-whisper model; default `base`.

To change searchable folders or allowed apps, edit `DOCUMENT_ROOTS` or `APP_TARGETS` in `jarvis.py`.

## References

- [Ollama local API](https://github.com/ollama/ollama/blob/main/docs/api.md)
- [Google Gmail API Python quickstart](https://developers.google.com/workspace/gmail/api/quickstart/python)
- [Google Calendar API Python quickstart](https://developers.google.com/workspace/calendar/api/quickstart/python)
- [Google OAuth scopes](https://developers.google.com/identity/protocols/oauth2/scopes)
- [faster-whisper](https://github.com/SYSTRAN/faster-whisper)
- [Vosk models](https://alphacephei.com/vosk/models)
- [Vosk API](https://github.com/alphacep/vosk-api)

