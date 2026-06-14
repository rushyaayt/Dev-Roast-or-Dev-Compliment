<div align="center">

  # Dev-Roast or Dev-Compliment

</div>

Here is the complete, **highly detailed, step-by-step guide** in English to build, run locally, and upload your **"Dev-Roast or Dev-Compliment" AI** project to GitHub.

---

## Phase 1: Prerequisites & Setup on Your Computer

Before you write any code, you need to make sure your computer has the right tools installed.

### Step 1: Install Node.js and Git

1. **Download Node.js:** Go to [nodejs.org](https://nodejs.org/) and download the **LTS (Long Term Support)** version. Follow the installer steps.
2. **Download Git:** Go to [git-scm.com](https://git-scm.com/) and download Git for your operating system (Windows/Mac/Linux).
3. **Verify Installation:** Open your terminal (Command Prompt on Windows or Terminal on Mac) and type:
```bash
node -v
git --version

```



```
   If both show version numbers, you are good to go!

### Step 2: Get a Free Gemini API Key
1. Go to [Google AI Studio](https://aistudio.google.com/).
2. Log in with your Google account.
3. Click on **"Get API Key"** and create a new key.
4. Copy this key somewhere safe (like a Notepad file). We will need it soon.

---

## Phase 2: Create and Code the Project Locally

We will use **Next.js** (a modern React framework) because it handles both the frontend web page and the backend API server in one single project.

### Step 1: Initialize the Project
Open your Terminal/Command Prompt, navigate to the folder where you want to save your project (e.g., `cd Desktop`), and run this command:

```bash
npx create-next-app@latest dev-roaster

```

While creating, the terminal will ask you some questions. **Select these exact answers:**

* Would you like to use TypeScript? › **Yes**
* Would you like to use ESLint? › **Yes**
* Would you like to use Tailwind CSS? › **Yes**
* Would you like to use src/ directory? › **Yes**
* Would you like to use App Router? (recommended) › **Yes**
* Would you like to customize the default import alias (@/*)? › **No**

Now, enter your project folder and install the official Google AI package:

```bash
cd dev-roaster
npm install @google/generative-ai

```

### Step 2: Open the Project in a Code Editor

Download and open **Visual Studio Code (VS Code)**. Click `File > Open Folder` and select the `dev-roaster` folder.

### Step 3: Secure Your API Key

In the root (main) folder of your project, create a new file and name it exactly:
`.env.local`

Inside this file, paste your Gemini API key like this:

```env
NEXT_PUBLIC_GEMINI_API_KEY=your_actual_gemini_api_key_here

```

> *Note: Next.js automatically hides `.env.local` from being uploaded to public GitHub, so your key stays completely safe!*

### Step 4: Write the Backend Logic (GitHub API + AI Integration)

Inside your VS Code file explorer, navigate to `src/app/`. Create a new folder structure inside it like this: `api/roast/`. Inside `roast/`, create a file named `route.ts`.

The full path will be: `src/app/api/roast/route.ts`

Paste the following code inside it:

```typescript
import { GoogleGenAI } from "@google/generative-ai";
import { NextResponse } from "next/server";

const ai = new GoogleGenAI(process.env.NEXT_PUBLIC_GEMINI_API_KEY || "");

export async function POST(req: Request) {
  try {
    const { username, mode } = await req.json(); // mode is either 'roast' or 'compliment'

    // 1. Fetch Basic GitHub User Profile Data
    const userRes = await fetch(`https://api.github.com/users/${username}`);
    if (!userRes.ok) {
      return NextResponse.json({ error: "GitHub user not found!" }, { status: 404 });
    }
    const userData = await userRes.json();

    // 2. Fetch User's Recent Repositories
    const repoRes = await fetch(`https://api.github.com/users/${username}/repos?sort=updated&per_page=8`);
    const repos = await repoRes.json();
    const repoNames = repos.map((r: any) => r.name).join(", ");

    // 3. Craft a strict, funny prompt for Gemini AI
    const prompt = `
      You are a hilarious, witty senior developer roasting or praising a junior developer. 
      Analyze this GitHub profile:
      Username: ${userData.login}
      Bio: ${userData.bio || "No bio written (classic lazy dev)"}
      Public Repos: ${userData.public_repos}
      Followers: ${userData.followers}
      Recent Projects: ${repoNames || "No public projects found"}

      The user explicitly requested a ${mode.toUpperCase()}. 
      Write a sharp, funny, and highly relatable ${mode} in a mix of Marathi and English (Marathish/Hinglish format).
      Make it deeply relevant to coding bugs, endless errors, commit messages, and developer lifestyle. 
      Keep it strictly under 4-5 sentences. Be savage if it's a roast, and wholesome yet witty if it's a compliment.
    `;

    // 4. Generate the content using Gemini
    const model = ai.getGenerativeModel({ model: "gemini-1.5-flash" });
    const result = await model.generateContent(prompt);
    const generatedText = result.response.text();

    return NextResponse.json({ result: generatedText });

  } catch (error) {
    console.error(error);
    return NextResponse.json({ error: "Something went wrong internally." }, { status: 500 });
  }
}

```

### Step 5: Design the Frontend Webpage

Open the existing file `src/app/page.tsx`, delete everything inside it, and paste this modern, clean developer-themed dark UI:

```tsx
'use client';
import { useState } from 'react';

export default function Home() {
  const [username, setUsername] = useState('');
  const [loading, setLoading] = useState(false);
  const [response, setResponse] = useState('');
  const [currentMode, setCurrentMode] = useState('');

  const handleAction = async (mode: 'roast' | 'compliment') => {
    if (!username.trim()) return alert('Please enter a GitHub username!');
    setLoading(true);
    setResponse('');
    setCurrentMode(mode);

    try {
      const res = await fetch('/api/roast', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ username, mode }),
      });
      const data = await res.json();
      if (data.error) {
        setResponse(`Error: ${data.error}`);
      } else {
        setResponse(data.result);
      }
    } catch (err) {
      setResponse('Failed to connect to the server.');
    } finally {
      setLoading(false);
    }
  };

  const shareOnTwitter = () => {
    const text = encodeURIComponent(
      `AI just completely ${currentMode}ed my GitHub profile! 😂 Check out what it says about your code here: `
    );
    window.open(`https://twitter.com/intent/tweet?text=${text}&url=${window.location.href}`, '_blank');
  };

  return (
    <main className="min-h-screen bg-gray-950 text-gray-100 flex flex-col items-center justify-center p-6 selection:bg-teal-500 selection:text-black">
      <div className="max-w-2xl w-full text-center space-y-8">
        <h1 className="text-4xl md:text-6xl font-extrabold tracking-tight bg-gradient-to-r from-red-500 via-purple-400 to-teal-400 bg-clip-text text-transparent animate-pulse">
          Git-Roaster AI 🛠️
        </h1>
        <p className="text-gray-400 text-lg">
          Enter your GitHub username to get roasted or complimented by AI in pure Marathish style.
        </p>

        <div className="flex flex-col sm:flex-row gap-4 justify-center items-center">
          <input
            type="text"
            placeholder="e.g., torvalds"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            className="w-full sm:w-80 px-4 py-3 bg-gray-900 border border-gray-800 rounded-lg focus:outline-none focus:border-teal-500 text-white placeholder-gray-600 transition-colors"
          />
          <div className="flex gap-2 w-full sm:w-auto">
            <button
              onClick={() => handleAction('roast')}
              disabled={loading}
              className="flex-1 sm:flex-none px-6 py-3 bg-red-600 hover:bg-red-700 font-bold rounded-lg transition-all duration-200 transform active:scale-95 disabled:opacity-50"
            >
              🔥 Roast Me
            </button>
            <button
              onClick={() => handleAction('compliment')}
              disabled={loading}
              className="flex-1 sm:flex-none px-6 py-3 bg-teal-600 hover:bg-teal-700 font-bold rounded-lg transition-all duration-200 transform active:scale-95 disabled:opacity-50"
            >
              💖 Praise Me
            </button>
          </div>
        </div>

        {loading && (
          <div className="text-teal-400 font-mono text-xl animate-bounce pt-6">
            Analyzing your messy commits... Hang tight! 🧑‍💻
          </div>
        )}

        {response && (
          <div className="mt-8 p-6 bg-gray-900 border border-gray-800 rounded-xl text-left shadow-2xl relative overflow-hidden animate-fadeIn">
            <div className="absolute top-0 left-0 w-2 h-full bg-gradient-to-b from-teal-500 to-purple-600" />
            <h3 className="text-xs font-mono tracking-widest text-gray-500 uppercase mb-2">
              AI Judgment for @{username}
            </h3>
            <p className="text-xl font-medium leading-relaxed text-gray-200 whitespace-pre-line">
              {response}
            </p>

            {!response.startsWith('Error:') && (
              <button
                onClick={shareOnTwitter}
                className="mt-6 inline-flex items-center gap-2 text-sm text-teal-400 hover:text-teal-300 font-semibold transition-colors"
              >
                🐦 Share this roast on X (Twitter) & humiliate yourself
              </button>
            )}
          </div>
        )}
      </div>
    </main>
  );
}

```

---

## Phase 3: How to Run and Test It Locally

1. Go back to your terminal window (ensure you are inside the `dev-roaster` directory).
2. Start the local Next.js development server by typing:
```bash

```



npm run dev

```
3. Look at the terminal output. It will tell you something like:
   ` Ready in 1200ms on http://localhost:3000`
4. Open your web browser (Chrome/Edge/Safari) and type **`http://localhost:3000`** in the address bar.
5. Your application is live locally! Type any active GitHub username, hit **Roast Me**, and see the AI work.

---

## Phase 4: How to Upload the Project to GitHub

Once everything is working perfectly on your local machine, it's time to show it to the world.

### Step 1: Create a Repository on GitHub Website
1. Go to [github.com](https://github.com/) and log in.
2. Click the **"+"** icon in the top right corner and choose **"New repository"**.
3. Repository name: `dev-roaster`
4. Description: `An AI-powered application that analyzes GitHub profiles and roasts or compliments developers in a fun Marathish/Hinglish style!`
5. Keep it **Public** (so everyone can see and star your work).
6. **Do NOT** check "Add a README file", "Add .gitignore", or "Choose a license" (Next.js already created these for you).
7. Click **"Create repository"**.

### Step 2: Push Your Local Code to GitHub
On the very next page, GitHub will display a set of terminal commands under the heading **"…or push an existing repository from the command line"**.

Go back to your local computer's terminal, stop your server (press `Ctrl + C`), and run these commands one by one:

```bash
# 1. Stage all your changed code files for saving
git add .

# 2. Save your staged files locally with a descriptive message
git commit -m "Initial commit: Set up Gemini AI backend and dark-themed UI"

# 3. Rename your main local branch to 'main'
git branch -M main

# 4. Link your local project directory to the remote GitHub repository
# (Copy-paste the EXACT URL line from your unique GitHub page)
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/dev-roaster.git

# 5. Upload your code to GitHub
git push -u origin main

```

Refresh your GitHub browser page, and boom! All your code is now live on GitHub for the world to explore, fork, and star.
