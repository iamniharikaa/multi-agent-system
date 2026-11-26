# ResearchMind: Multi-Agent Research System

ResearchMind researches any topic and writes a report. Four AI agents work in order:

1. **Search Agent** finds recent, reliable information on the topic.
2. **Reader Agent** picks the most relevant page and scrapes its full content.
3. **Writer** drafts a research report from the search results and scraped content.
4. **Critic** reviews the draft and gives feedback.

Built with [LangChain](https://www.langchain.com/) and [Streamlit](https://streamlit.io/). You can use the web interface or run the pipeline in the terminal.

**Live demo:** [https://researchmind04.streamlit.app/](https://researchmind04.streamlit.app/)

## Features

- Enter any research topic and get a full report
- Watch each agent's progress step by step
- View the raw search results and scraped content
- Read the critic's feedback on the report
- Download the final report as a Markdown file

## Project structure

```
├── app.py            # Streamlit web interface
├── agents.py         # Agent and chain definitions
├── tools.py          # Tools the agents use
├── pipeline.py       # Runs the pipeline from the terminal
├── requirements.txt  # Python dependencies
└── .gitignore
```

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/iamniharikaa/multi-agent-system.git
cd multi-agent-system
```

### 2. Create a virtual environment

Windows (PowerShell):

```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

macOS / Linux:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Add your API keys

Create a file named `.env` in the project folder and add the keys your agents use:

```
TAVILY_API_KEY = your-key-here
OPENAI_API_KEY = your-key-here
```

Never commit `.env` to GitHub. It's already listed in `.gitignore`.

## Usage

### Web app

```bash
streamlit run app.py
```

Open the local link it prints (usually http://localhost:8501), enter a topic, and click **Run Research Pipeline**.

### Terminal

```bash
python pipeline.py
```

Type a topic when prompted. The results print in the terminal.

## Deployment

The app runs on Streamlit Community Cloud:

1. Sign in at [share.streamlit.io](https://share.streamlit.io) with GitHub.
2. Click **Create app**, select this repository, set the branch to `main`, and set the main file path to `app.py`.
3. Under **Advanced settings → Secrets**, add your API keys in TOML format:
   ```toml
   TAVILY_API_KEY = "your-key-here"
   OPENAI_API_KEY = "your-key-here"
   ```
4. Click **Deploy**.

On Streamlit Cloud, the app reads keys from the Secrets settings instead of `.env`.

## Limitations

- Each run makes several LLM and web requests. It can take a few minutes and uses API credits.
- Some websites block scrapers, so the Reader Agent may not always get the full page.
- Report quality depends on the language model and the search results.

