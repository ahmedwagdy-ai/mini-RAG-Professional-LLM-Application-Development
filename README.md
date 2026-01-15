# mini-RAG-Professional-LLM-Application-Development

## 📌 About the Project (Video 01)
**mini-RAG** is a practical educational series designed to bridge the gap between "Experimental Data Science" (Jupyter Notebooks) and "Production-ready Software Engineering." 

The project focuses on building a **Retrieval-Augmented Generation (RAG)** application from scratch. This type of application allows users to query their own documents and receive accurate, context-aware answers extracted directly from those files using Large Language Models (LLMs).

## 🎯 The Core Philosophy
The main challenge in AI today is not just writing a model, but building the software around it. This course addresses:
* **Escaping the Notebook Bubble:** Transitioning from `.ipynb` scripts to a structured Python web application.
* **Software Engineering for AI:** Implementing best practices that make AI projects deliverable to real-world clients.
* **Production Readiness:** Focus on environment setup, library orchestration, and UI/UX integration.

## 🛠 Tech Stack
* **Language:** Python
* **LLM Orchestration:** LangChain
* **Web Framework:** Python-based Web Framework (to be detailed in subsequent stages).
* **Concept:** RAG (Retrieval-Augmented Generation).

## 📂 Repository Structure (How to use this Repo)
To make it easy to follow along with the video series, this repository is organized using **Checkpoints**:

1.  **`main` Branch:** Contains the latest, most stable, and final version of the application.
2.  **Tutorial Branches:** Each video in the series has a dedicated branch (e.g., `tutorial-02`, `tutorial-03`). 
    * *To see the exact code written in a specific video, switch to its corresponding branch.*

## 🤝 Contribution
This is an open-source educational project. Contributions are highly encouraged! 
If you wish to add a feature:
1. Fork the repo.
2. Create your feature branch.
3. Submit a **Pull Request** with a brief explanation (video or text) of the changes made.

## 👨‍🏫 Credits
This project is based on the "mini-RAG" course by **Abu Bakr Soliman**. It serves as a template for developers looking to build modern AI applications.

# mini-RAG: Technical Roadmap & Architecture (Video 02)

## 🏗 Project Architecture
The project is built using **FastAPI**, focusing on a clean and scalable professional directory structure. This phase covers essential configuration management including `.env` files and modular project organization.

## 🚀 Phase 1: Core Data Pipeline
The application implements the standard RAG lifecycle through four main stages, exposed as API endpoints:

### 1. Upload & Text Extraction
* **Goal:** Handle various file formats (PDF, Text, etc.).
* **Process:** Files are uploaded, and text is extracted and prepared for processing.
* **Output:** A unique `Reference ID` for each uploaded document.

### 2. Chunking & Indexing
* **Goal:** Transform raw text into searchable units.
* **Process:** * Large texts are split into smaller **Chunks** (to fit LLM context windows).
    * Chunks are converted into **Vectors** (Embeddings).
    * Vectors are stored in a **Vector Database** (to be specified in the implementation).

### 3. Semantic Search (Retrieval)
* **Goal:** Find the most relevant information based on meaning.
* **Process:** The system compares the user's question against the stored vectors.
* **Output:** A list of relevant document snippets with a **Similarity Score**.

### 4. Answer Generation (Augmentation)
* **Goal:** Provide a final human-like response.
* **Process:** The system sends the user's question + the retrieved context to the **LLM**.
* **Output:** A concise, accurate answer based strictly on the provided documents.

## 🛣 API Endpoints Roadmap
In this stage, the following endpoints will be developed:
* `POST /upload`: Upload documents and receive a reference ID.
* `POST /process`: Trigger text extraction, chunking, and indexing.
* `GET /search`: Query the vector database for relevant snippets.
* `GET /answer`: Perform the full RAG cycle to generate a final answer.

## 🛠 Next Steps
Following the **Checkpoints** method:
* Switch to the branch corresponding to each tutorial to see the incremental implementation of these features.
* Pay attention to the **Logic Flow** explained in the videos to understand how FastAPI interacts with LangChain and the Vector DB.

# 📑 Exhaustive Technical Setup & Architecture (Video 03)


## 1. Project Philosophy & Introduction
In this educational series, we aim to build an **Open Source** project (meaning the code is free and available for anyone to modify). The goal is to use **Large Language Models (LLMs)** like GPT to answer user questions based on specific documents.

* **RAG (Retrieval Augmented Generation):** A technique that combines retrieving information from your documents with the smart generation of AI models. This ensures answers are accurate and based on real data, preventing "Hallucinations" (AI making up facts).
* **From Notebooks to Production:** We are moving from "Notebooks" (Jupyter files used for quick experiments) to "Production" (the actual operation of the app in a stable, scalable, and real-world environment).
* **Repo (Repository):** The first rule is that you *must* have a Repo. It’s the storehouse for your code on platforms like GitHub or Bitbucket for storage and version control. {ts:1}

---

## 2. GitHub Repository Setup
* **Step 1:** Go to GitHub and click **Create New Repo**.
* **Step 2:** Name it (e.g., `mini-RAG-app`).
* **Step 3:** Choose **Private** (visible only to you and your collaborators) or **Public** (visible to the whole world).
* **Note:** Get into the habit of having a Git Repo for every project, even personal ones.

---

## 3. Installing the Git Client
To manage the repo on your "Machine" (your computer), you need a **Git Client**.
* **Windows:** Visit [git-scm.com](https://git-scm.com/), search for "Download for Windows," install it, and you're set.
* **Linux (Ubuntu):** Simply run `sudo apt install git`.
* **Verification:** Open your **Terminal** or **CMD** and type:
    ```bash
    git --version
    ```
    If you see a version number (e.g., 2.45.2), Git is successfully installed.

---

## 4. Cloning and Authentication (Tokens)
Now, we perform a **Clone** (copying the repo from the web to your local machine).
1.  On GitHub, click the **Code** button and copy the **HTTPS Link**.
2.  Open your terminal, navigate to a location (e.g., `cd Desktop`), and type `git clone` followed by the link.
3.  **Authentication (Token):** GitHub will likely ask for a password. You cannot use your account password here; you need a **Token**.
    * Go to your GitHub **Profile** > **Settings** > **Developer Settings** > **Personal Access Tokens**.
    * Generate a new token, name it (e.g., `mini-RAG-access`), and copy it.
    * This token is your "password" for terminal operations.

---

## 5. IDE: Visual Studio Code (VS Code)
The **Editor** is where you write the code. We use **VS Code** because it's free, popular, and supports Python/Git extensions.
* **Download:** Get it from `code.visualstudio.com`.
* **Integrated Terminal:** In VS Code, go to `File > View > Terminal`. This opens a tab at the bottom to run commands directly within the editor.

---

## 6. Python Setup: Miniconda
To manage Python on your device, the instructor prefers **Miniconda**. It is a "Bundle" that includes Python and a package manager called **Conda**.
* **Why Conda?** It handles **Environments** (isolated setups for each project) to avoid library conflicts.
* **Windows Installation Warning:** When installing, you **must** select the option **"Add to PATH."** If you skip this, the `conda` command won't work in your terminal.

---

## 7. Creating a Conda Environment
In the terminal, you might see `(base)`. This is the default environment. **Do not use it.** We need a specific "Env" for our project.
1.  Run the command: `conda create -n mini-rag-app python=3.8` (This creates an env named `mini-rag-app` with Python 3.8).
2.  Press **Enter** and type **Yes (y)** when prompted.
3.  **Activation:** To "enter" this world, type:
    ```bash
    conda activate mini-rag-app
    ```
4.  The prefix in your command prompt will change from `(base)` to `(mini-rag-app)`. You can verify by typing `python --version`.

---

## 8. WSL (Windows Subsystem for Linux)
Even if you are a Windows user, it is highly recommended to use **WSL**.
* **The Idea:** Run a native Linux system inside Windows without a full Virtual Machine.
* **The Reason:** Most **Cloud** platforms (Railway, Vercel, etc.) run on Linux. Developing in a Linux environment now prevents deployment "surprises" later.

---

## 9. Installing WSL (Step-by-Step)
1.  **Check Version:** Ensure you have Windows 10 (version 2004+) or Windows 11. Check this in **System Information**.
2.  **PowerShell:** Open Windows PowerShell as **Administrator** and run:
    ```powershell
    wsl --install
    ```
3.  **Default Version:** Ensure you are using version 2: `wsl --set-default-version 2`.
4.  **Distro:** Search for **Ubuntu 22.04 LTS** in the Microsoft Store and install it.

---

## 10. Configuring Ubuntu
1.  Launch Ubuntu from the Start Menu.
2.  Set a **Username** and **Password** (This password will be required for any `sudo` command).
3.  Run your first update:
    ```bash
    sudo apt update
    ```

---

## 11. Accessing Windows Files from Ubuntu (WSL)
Your Windows files are accessible within Linux under the `/mnt/` (mount) folder.
* To go to your Windows Desktop:
    ```bash
    cd /mnt/c/Users/<Your_Username>/Desktop
    ```
* **Opening VS Code from WSL:** Type `code .` in the Ubuntu terminal. It will install the VS Code client and open the folder. The status bar will show "WSL: Ubuntu."

---

## 12. Installing Miniconda inside WSL (Linux)
You need a separate installation for the Linux side:
1.  Copy the **Linux 64-bit** link from the Miniconda website.
2.  In Ubuntu, type `wget` followed by the link.
3.  Make the file executable: `chmod +x Miniconda3-latest-Linux-x86_64.sh`.
4.  Run the installer: `./Miniconda3-latest-Linux-x86_64.sh`.
5.  **Critical Step:** When asked to initialize or add to PATH, type **Yes**.
6.  **Refresh:** Run `source ~/.bashrc` or restart the terminal to activate Conda.

---

## 13. Terminal Types & Troubleshooting
* **VS Code Tip:** You can run VS Code on Windows but use the **Ubuntu WSL terminal** inside it. Click the `+` icon in the terminal window and select **WSL / Ubuntu**.
* **Missing Environment?** If `conda activate` doesn't find your env, run `conda info --envs` to find the path, then activate it using the full path:
    ```bash
    conda activate /home/user/miniconda3/envs/mini-rag-app
    ```

---
**Summary of the Workflow:** Open VS Code -> Open WSL Terminal -> Activate your Conda Env -> Start Coding!


# 📑 Professional Project Architecture (Video 04) - Deep Technical Dive


## I. Project Introduction & RAG Philosophy
In this tutorial series, we are building a step-by-step **Open-Source** project. "Open-source" means the code is available for everyone to modify, use, and contribute to for free.

* **The Goal:** We want to exploit **Large Language Models (LLMs)** like GPT to answer user questions based on specific documents. This technique is known as **Retrieval Augmented Generation (RAG)**. 
* **What is RAG?** It is a technology that bridges the gap between retrieving specific information from documents and the creative generation of AI. This ensures answers are accurate, grounded in real data, and free from "hallucinations" (AI making up facts).
* **The Path to Production:** We are moving our work from simple **Notebooks** (Jupyter files for quick testing) to applications capable of reaching the **Production** stage—where the app is stable, scalable, and ready for actual users. {ts:1}

---

## II. Development Environment Setup (The Essentials)
Before writing code, we must set up the "Machine" (your computer).

### 1. Version Control (Git & GitHub)
You must have a **Repo** (Repository), which is a storage space for your code on GitHub, GitLab, or Bitbucket. 
* **Recommendation:** Always use a Git Repo, even for small personal projects.
* **Git Client:** You need the Git software installed locally.
    * **Windows:** Download from `git-scm.com`.
    * **Linux (Ubuntu):** Run `sudo apt install git`.
    * **Verification:** Run `git --version`. If it shows a number (e.g., 2.45.2), you are ready.
* **Authentication (Tokens):** GitHub requires a **Personal Access Token (PAT)** instead of a password. Go to **Settings > Developer Settings > Personal Access Tokens** to generate one.

### 2. Editor (VS Code)
We use **Visual Studio Code** as our primary editor. It has an integrated terminal (`View > Terminal`) that opens a "Tab" at the bottom for running commands without leaving the code.

### 3. Python Management (Miniconda)
**Miniconda** is a lightweight version of Anaconda that includes Python and the `conda` tool.
* **Why Conda?** It manages **Environments** (isolated spaces). This prevents "Dependency Hell" where libraries from one project conflict with another.
* **Windows Setup:** You **must** check the option **"Add to PATH"** during installation so the commands work in any terminal.

---

## III. Professional Workflow: WSL (Windows Subsystem for Linux)
For Windows users, using **WSL** is a professional standard.
* **Why WSL?** Most **Cloud** platforms (Railway, Vercel, AWS) run on Linux. Developing in a Linux environment (Ubuntu) inside Windows ensures that what works on your machine will work on the server.
* **Setup:** Run `wsl --install` in PowerShell as Administrator. Use **Ubuntu 22.04 LTS**.
* **Accessing Files:** Your Windows files are located under `/mnt/c/Users/<Username>/Desktop`.
* **Installing Miniconda in WSL:** Right-click the Linux 64-bit installer link from the site, use `wget <link>`, make it executable with `chmod +x`, and run `./`. Always say **"Yes"** to initialize.

---

## IV. Git Branching & Project Cleanup
"Peace be upon you (سلام عليكم). Since we finished setup, let's start coding."

### 1. Branching Strategy
We never work directly on the `main` branch. Every tutorial happens on a new **Branch** to preserve the code.
* **In VS Code:** Go to Source Control and click **Create Branch**.
* **Naming Conventions:**
    * **Feature branches:** Start with `feature/` (e.g., `feature/document-loader`).
    * **Bugfix:** For fixing errors.
    * **Task IDs:** If working in a team using Jira, name the branch after the Task ID (e.g., `Jira-101`) to link everything together.
    * **Tutorial:** For this series, the branch name is `tutorial`.

### 2. Cleaning the Directory
Delete everything except the core project structure. We keep only:
* **LICENSE:** Defines usage rights (e.g., Apache/MIT).
* **.gitignore:** A template that tells GitHub which files to ignore (cache, data, secrets). You can search for the "GitHub Python .gitignore template" and copy it.

---

## V. The Project Gateway: README.md
Every project must have a **README** in **MARKDOWN** format (all uppercase).
* **The Goal:** It is the gateway for anyone exploring your project.
* **Rules:** Put yourself in the shoes of a stranger. Explain how to run the project without contacting you. Even if a detail seems "trivial," include it. Assume the developer knows nothing about Python. 
* **Quality:** A well-organized README is a primary indicator of high-quality work.

---

## VI. Managing Dependencies: requirements.txt
This file lists ALL packages needed for the project in one place.
* **FastAPI (0.115.0):** The most famous web framework for Python. It enforces standard code structure and better longevity.
* **Uvicorn (0.30.6):** An **ASGI server**. FastAPI needs it to send requests and get responses.
    * *Production Note:* For the cloud, we might later use Gunicorn or Nginx.

### The Rule of Pinned Versions
**Never** write a package without a version. 
* **Bad:** `pip install fastapi` (Installs the latest, which might cause conflicts).
* **Good:** `fastapi==0.115.0` (Ensures an exact match and a reproducible project).
* **Git Diff Tip:** Always leave an **empty line** at the end of the file to prevent false Git differences.

---

## VII. Folder Structure & Environment Config
* **assets/ folder:** Create this to group downloads/creations. Since Git ignores empty folders, add an empty file named **.gitkeep** to force tracking.
* **Configuration (.env):** Hardcoding secrets in code is "bad practice."
    * **.env (Ignored):** Contains actual secrets like `OPENAI_API_KEY`.
    * **.env.example (Committed):** A template for others. It should include the `APP_NAME` and `APP_VERSION` (using **SemVer** like `v0.1.0`).
    * **Instruction:** The README should tell users to `cp .env.example .env` and fill in their keys.

---

## VIII. Installation & Final Checklist
In your WSL Terminal, run:
1.  `conda activate mini-rag-app`
2.  `pip install -r requirements.txt`

**The Boilerplate is now ready:**
- [x] Git Repo & Tutorial Branch created.
- [x] .gitignore & LICENSE added.
- [x] README.md (Comprehensive documentation).
- [x] requirements.txt (Pinned FastAPI & Uvicorn).
- [x] assets/ folder with .gitkeep.
- [x] .env.example for configuration management.
