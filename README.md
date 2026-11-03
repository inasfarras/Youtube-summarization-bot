# YouTube Transcript Summarizer

A Jupyter notebook that fetches a YouTube video transcript and sends the text to the OpenAI Completions API.

## Features

- `get_transcript` in `yt-sum.ipynb` takes the substring after `v=` as the video id, calls `YouTubeTranscriptApi.get_transcript`, and joins each snippet's `text` field into one string.
- When the transcript request fails, `get_transcript` returns the exception message as a string.
- `get_summary` calls `openai.Completion.create` with model `gpt-4o-mini`, the prompt `Summarize the following transcript: {transcript}`, and `max_tokens=1000`, then returns `response.choices[0].text`.

## Tech stack

- Python 3.12.3, recorded in `yt-sum.ipynb` (`language_info.version`). The kernel display name is `env`.
- Jupyter. `requirements.txt` pins `notebook==7.2.2` and `jupyterlab==4.2.6`.
- `youtube-transcript-api==1.0.1`
- `openai==1.65.4` (the same version printed in the notebook's install-cell output)
- `python-dotenv==1.0.0`, imported as `from dotenv import load_dotenv`

## Setup

```bash
git clone https://github.com/inasfarras/Youtube-summarization-bot.git
cd Youtube-summarization-bot
python3.12 -m venv venv
```

Activate the environment:

```bash
# Linux or macOS
source venv/bin/activate

# Windows
venv\Scripts\activate
```

Install the packages the notebook imports, plus Notebook so you can open `yt-sum.ipynb`. These versions are the pins in `requirements.txt`.

```bash
pip install "youtube-transcript-api==1.0.1" "openai==1.65.4" "python-dotenv==1.0.0" "notebook==7.2.2"
```

The notebook's first code cell runs `pip install youtube-transcript-api`, and the second runs `!pip install openai`. Those cells leave the versions unpinned and leave `python-dotenv` uninstalled.

`requirements.txt` is a freeze of a Windows Python 3.12 environment. Alongside the packages above it lists many libraries the notebook never imports, including `pywin32`, `pywinpty`, and `torch==2.4.0+cu118`.

Create a file named `.env` in the repository root:

```
OPENAI_API_KEY=your_openai_api_key_here
```

The summary cell calls `load_dotenv()` and `os.getenv("OPENAI_API_KEY")`. Start Jupyter from the repository root so that lookup finds this file. `.gitignore` ignores the directories `env/`, `venv/`, and `.env/`. A file named `.env` sits outside that directory rule, so keep the key file out of version control.

## Usage

From the repository root:

```bash
python -m notebook yt-sum.ipynb
```

Run the cells from top to bottom.

The transcript cell calls `get_transcript` with this hardcoded URL and prints the result:

```python
video_url = "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
print(get_transcript(video_url))
```

Replace `video_url` with another `watch?v=` link to fetch a different video. The id is the entire substring after the first `v=`, so a later query parameter such as `&t=` stays attached to the id. The saved output of this cell is the transcript text for that example URL.

The summary cell calls `get_summary(transcript)`. The transcript cell only prints its return value, so assign the name before the summary call:

```python
transcript = get_transcript(video_url)
```

`youtube-transcript-api` 1.0.1 still provides `YouTubeTranscriptApi.get_transcript` and marks it deprecated in favor of `fetch`. That method returns a list of dictionaries with `text`, `start`, and `duration`, which is the shape the notebook reads.

`openai` 1.65.4 exposes `openai.Completion` as a removed v0 stub. Calling `openai.Completion.create`, which is what `get_summary` does, raises `APIRemovedInV1`. That error points to the OpenAI Python 1.0 interface, or to installing `openai==0.28`. The summary cell stored in the notebook ends at `ModuleNotFoundError: No module named 'openai'`.

## License

Released under the MIT License. Copyright (c) 2026 Farras Inas. See [LICENSE](LICENSE).
