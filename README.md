# 💼 Auto Jobs: AI Job Application Assistant

> Paste a job description and Auto Jobs gives you an ATS keyword match score, specific suggestions to tailor your resume, and a cover letter written by Google Gemini. It also files every application in your tracker and sets up a ready-to-edit folder for each company.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-Web%20App-FF4B4B?logo=streamlit&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-0.3-1C3C3C?logo=langchain&logoColor=white)
![Gemini](https://img.shields.io/badge/Google-Gemini-4285F4?logo=googlegemini&logoColor=white)
![Codespaces](https://img.shields.io/badge/Dev%20Container-ready-2088FF?logo=github&logoColor=white)

🎥 **[Watch the demo](https://youtu.be/xXHOI-Lyxgw)**

---

## Table of Contents

- [Why Auto Jobs?](#why-auto-jobs)
- [Features](#features)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Option A: Web App (Streamlit)](#option-a-web-app-streamlit)
- [Option B: Command-Line Workflow](#option-b-command-line-workflow)
- [Configuration](#configuration)
- [Project Structure](#project-structure)
- [Roadmap](#roadmap)
- [Contributing](#contributing)

---

## Why Auto Jobs?

Applying for jobs means doing the same things over and over for every posting: tweaking your resume, writing a new cover letter, guessing whether you'll get past the ATS, and keeping a spreadsheet up to date. Auto Jobs turns that into **a single command or a single click**.

---

## Features

| | Feature | What you get |
|---|---|---|
| 🎯 | **ATS match score** | Keywords from the job description, the ones missing from your resume, and a percentage match |
| ✍️ | **Resume tailoring** | Specific, bullet-by-bullet edits that line your resume up with the job, plus an honest verdict on whether it would pass screening |
| 📝 | **AI cover letter** | A tailored cover letter based on the role's real challenges, downloadable as `.docx` |
| 📁 | **Folder per company** | Copies your resume and cover-letter templates for the role type into a new `roles/<Company>/` folder (command line) |
| 📊 | **Application tracker** | Adds each application (company, role, date, job description) to `assets/Job Tracker.csv` (command line) |
| 🧑‍💻 | **Two ways to use it** | A Streamlit web app or a `make`-based command-line workflow |

### How the cover letter is written

Auto Jobs doesn't just ask for "a cover letter". It builds one in steps, the way a career coach would:

1. **Find the challenges.** Work out the main day-to-day problems the role is meant to solve, and what causes them.
2. **Write the hook (≈70 words).** Show you understand those problems, with evidence from your resume.
3. **Write the body (≈70 words).** Connect specific results, leadership and skills to the job description.
4. **Write the close (≈50 words).** Restate your interest in the role.

The command-line tool runs these as a four-step LangChain `SequentialChain`. The web app uses a single combined prompt that follows the same structure.

---

## How It Works

```
                    ┌──────────────── Inputs ────────────────┐
                    │  Resume (PDF) · Job description ·       │
                    │  Company name · Role type (CLI only)    │
                    └───────────────────┬────────────────────┘
                                        ▼
                         pypdf ─► resume text
                                        ▼
            ┌─────────────── LangChain + Gemini ───────────────┐
            │  ATS match prompt ───────► keywords, missing, %   │
            │  Resume eval prompt ─────► tailoring suggestions  │
            │  Cover letter chain ─────► challenges → hook →    │
            │                            body → close           │
            └───────────────────────┬─────────────────────────┘
                                    ▼
      Web: show results + download .docx    CLI: roles/<Company>/ folder
                                                 + cover letter .docx
                                                 + row in Job Tracker.csv
```

---

## Tech Stack

| Area | Tools |
|---|---|
| LLM | Google Gemini Flash through `langchain-google-genai` |
| Orchestration | LangChain (`LLMChain`, `SequentialChain`, `PromptTemplate`) |
| UI | Streamlit |
| Documents | pypdf (reading resumes), python-docx (writing cover letters) |
| Tracking | pandas (CSV tracker) |
| Developer setup | Makefile, Dev Container / GitHub Codespaces |

---

## Getting Started

### Prerequisites

- Python 3.11 or later
- A free **Google Gemini API key** from [Google AI Studio](https://aistudio.google.com/app/apikey)

### Install

```bash
git clone <your-repo-url> auto-jobs
cd auto-jobs
make install                 # creates venv/ and installs requirements
```

Then create your `.env` file from the example:

```bash
cp .env.example .env
# open .env and set:  GOOGLE_API_KEY="your-key-here"
```

> ☁️ **Prefer the cloud?** Open the repo in **GitHub Codespaces**. The dev container installs everything and starts the web app on port 8501 automatically.

---

## Option A: Web App (Streamlit)

The quickest way to try it.

```bash
source venv/bin/activate
streamlit run app.py
```

Open http://localhost:8501, then:

1. **Upload** your resume as a PDF.
2. **Paste** the job description and enter the company name.
3. Click one of these:
   - **Analyze Resume based on Job description** for tailoring suggestions
   - **ATS Match** for the keyword match and score
   - **Generate Cover Letter** to see the letter and download it as `.docx`

---

## Option B: Command-Line Workflow

Best if you apply to many jobs and want everything organized on disk.

### 1. Set up your profile and role types

Edit `config/config.py`:

```python
FName = "Your"           # your first name, used in file names and the letter sign-off
LName = "Name"           # your last name

roleMap = {              # one entry per kind of role you apply for
    "Role1": f"{template_dir}/Backend",
    "Role2": f"{template_dir}/Data_Analysis",
    "Role3": f"{template_dir}/Machine_Learning",
}

class rolePath(Enum):    # short codes you'll type on the command line
    Role1 = "BE"
    Role2 = "DS"
    Role3 = "ML"
```

Also put your contact details into `assets/cover_letter.docx`, which is the cover-letter template.

### 2. Create the folder layout

```bash
make setup
```

This creates `templates/<Role>/` for each role type, each with a copy of the cover-letter template and empty placeholder resume files, plus an empty `roles/` folder.

### 3. Add your resumes

Replace the placeholders in each `templates/<Role>/` folder with your real resume for that kind of role, as **both `.pdf` and `.docx`**, named `<FName>_<LName>_Resume.pdf` and `<FName>_<LName>_Resume.docx`.

### 4. Apply

```bash
make run r="ML" c="Acme Corp"
```

Paste the job description, then press **Ctrl+D** (macOS/Linux) or **Ctrl+Z, Enter** (Windows). Auto Jobs will:

- create `roles/Acme Corp/` containing your ML resume and cover-letter template
- print the ATS match score and tailoring suggestions
- write the cover letter into `roles/Acme Corp/<FName>_<LName>_cover_letter.docx`
- add the application to `assets/Job Tracker.csv`

---

## Configuration

| Setting | Where | Purpose |
|---|---|---|
| `GOOGLE_API_KEY` | `.env` | Gemini API access |
| `FName`, `LName` | `config/config.py` | File names and the cover-letter signature |
| `roleMap`, `rolePath` | `config/config.py` | Role types and their command-line codes |
| `*_prompt` strings | `config/config.py` | All LLM prompts. Edit these to change tone, length or strictness |
| Gemini model | `src/google_smart.py` | Set to a current Gemini model (for example `gemini-2.5-flash`). `gemini-1.5-flash` has been retired |
| Tracker columns | `assets/Job Tracker.csv` | Company, title, location, link, source, referral, salary, benefits, date, interviews |

---

## Project Structure

```
auto-jobs/
├── app.py                    # Streamlit web app
├── makefile                  # make install | make setup | make run
├── config/
│   └── config.py             # Your name, role types and all LLM prompts
├── src/
│   ├── google_smart.py       # Gemini chains: ATS match, evaluation, cover letters
│   ├── Make_files.py         # Command-line workflow: folder, analysis, letter, tracker
│   ├── setup.py              # Creates the templates/ and roles/ folders
│   └── CSV_saving.py         # Adds rows to the job tracker CSV
├── assets/
│   ├── cover_letter.docx     # Cover-letter template
│   └── Job Tracker.csv       # Application tracker
├── .devcontainer/            # Codespaces / Dev Container setup
├── .env.example
└── requirements.txt
```

`templates/` and `roles/` are created by `make setup` and git-ignored, so your resumes stay private.

---

## Roadmap

- [ ] Package as a proper CLI (`pip install auto-jobs`) with Poetry
- [ ] Better prompts for resume suggestions, and cover letters that sound less AI-written
- [ ] Support more models, including local LLMs through LM Studio
- [ ] Chat with your career history through a RAG pipeline
- [ ] Track applications in the web app too

---

## Contributing

Contributions are welcome. Fork the repo, create a branch, and open a pull request. See `TODO.md` for open ideas.