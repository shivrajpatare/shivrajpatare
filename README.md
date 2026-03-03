<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>SHIVRAJ PATARE</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #0a0a0f;
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    font-family: 'Courier New', monospace;
    overflow: hidden;
  }

  .container {
    text-align: center;
    position: relative;
  }

  #ascii-art {
    font-size: clamp(4px, 1vw, 11px);
    line-height: 1.2;
    white-space: pre;
    letter-spacing: 0.05em;
    display: inline-block;
    text-align: left;
    position: relative;
  }

  .char {
    display: inline;
    opacity: 0;
    transition: opacity 0.05s;
  }

  .char.visible {
    opacity: 1;
  }

  /* Color gradient for rows */
  .row-0  { color: #ff0080; }
  .row-1  { color: #ff2060; }
  .row-2  { color: #ff6030; }
  .row-3  { color: #ffaa00; }
  .row-4  { color: #ffe000; }
  .row-5  { color: #80ff00; }
  .row-6  { color: #00ffaa; }
  .row-7  { color: #00ccff; }

  /* Scanline overlay */
  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background: repeating-linear-gradient(
      0deg,
      transparent,
      transparent 2px,
      rgba(0,0,0,0.08) 2px,
      rgba(0,0,0,0.08) 4px
    );
    pointer-events: none;
    z-index: 10;
  }

  /* Glow pulse after reveal */
  #ascii-art.done {
    animation: glowPulse 3s ease-in-out infinite;
  }

  @keyframes glowPulse {
    0%, 100% { filter: drop-shadow(0 0 8px rgba(255,0,128,0.6)) drop-shadow(0 0 20px rgba(0,200,255,0.3)); }
    50%       { filter: drop-shadow(0 0 20px rgba(255,200,0,0.8)) drop-shadow(0 0 40px rgba(0,255,170,0.4)); }
  }

  /* Cursor blink */
  .cursor {
    display: inline-block;
    width: 0.6em;
    height: 1em;
    background: #00ffaa;
    vertical-align: bottom;
    animation: blink 0.7s step-end infinite;
    margin-left: 2px;
    box-shadow: 0 0 8px #00ffaa;
  }

  @keyframes blink {
    0%, 100% { opacity: 1; }
    50%       { opacity: 0; }
  }

  .subtitle {
    margin-top: 24px;
    color: #444;
    font-size: 13px;
    letter-spacing: 0.3em;
    text-transform: uppercase;
    opacity: 0;
    transition: opacity 1s ease;
  }
  .subtitle.visible {
    opacity: 1;
    color: #556;
    animation: subtitleGlow 4s ease-in-out infinite;
  }
  @keyframes subtitleGlow {
    0%,100% { color: #445566; }
    50%     { color: #88aacc; text-shadow: 0 0 10px #00ccff55; }
  }

  /* Background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(0,200,255,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,200,255,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
  }
</style>
</head>
<body>
<div class="container">
  <div id="ascii-art"></div>
  <div class="subtitle" id="subtitle">// initializing . . .</div>
</div>

<script>
const art = `███████╗██╗  ██╗██╗██╗   ██╗██████╗  █████╗      ██╗ ██████╗  █████╗ ████████╗ █████╗ ██████╗ ███████╗
██╔════╝██║  ██║██║██║   ██║██╔══██╗██╔══██╗     ██║ ██╔══██╗██╔══██╗╚══██╔══╝██╔══██╗██╔══██╗██╔════╝
███████╗███████║██║██║   ██║██████╔╝███████║     ██║ ██████╔╝███████║   ██║   ███████║██████╔╝█████╗  
╚════██║██╔══██║██║╚██╗ ██╔╝██╔══██╗██╔══██║██   ██║ ██╔═══╝ ██╔══██║   ██║   ██╔══██║██╔══██╗██╔══╝  
███████║██║  ██║██║ ╚████╔╝ ██║  ██║██║  ██║╚█████╔╝ ██║     ██║  ██║   ██║   ██║  ██║██║  ██║███████╗
╚══════╝╚═╝  ╚═╝╚═╝  ╚═══╝  ╚═╝  ╚═╝╚═╝  ╚═╝ ╚════╝  ╚═╝     ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝`;

const container = document.getElementById('ascii-art');
const subtitle  = document.getElementById('subtitle');

// Build spans per character, row-colored
const lines = art.split('\n');
const allSpans = [];

lines.forEach((line, rowIdx) => {
  const rowClass = `row-${rowIdx}`;
  [...line].forEach(ch => {
    const span = document.createElement('span');
    span.className = `char ${rowClass}`;
    span.textContent = ch;
    container.appendChild(span);
    allSpans.push(span);
  });
  // newline
  const nl = document.createElement('span');
  nl.textContent = '\n';
  container.appendChild(nl);
});

// Cursor element
const cursor = document.createElement('span');
cursor.className = 'cursor';
container.appendChild(cursor);

// Typing animation
let i = 0;
const speed = 1; // chars per tick
const interval = 8; // ms per tick

function type() {
  if (i < allSpans.length) {
    const batch = Math.min(speed + Math.floor(i / 200), 6);
    for (let b = 0; b < batch && i < allSpans.length; b++, i++) {
      allSpans[i].classList.add('visible');
    }
    // Move cursor after last visible span
    const lastVisible = allSpans[i - 1];
    if (lastVisible && lastVisible.nextSibling !== cursor) {
      lastVisible.parentNode.insertBefore(cursor, lastVisible.nextSibling);
    }
    setTimeout(type, interval);
  } else {
    // Done — remove cursor, add glow, show subtitle
    cursor.remove();
    container.classList.add('done');
    setTimeout(() => subtitle.classList.add('visible'), 400);
  }
}

// Small delay before starting
setTimeout(type, 300);
</script>
</body>
</html>




<pre align="center">
███████╗██╗  ██╗██╗██╗   ██╗██████╗  █████╗      ██╗ ██████╗  █████╗ ████████╗ █████╗ ██████╗ ███████╗
██╔════╝██║  ██║██║██║   ██║██╔══██╗██╔══██╗     ██║ ██╔══██╗██╔══██╗╚══██╔══╝██╔══██╗██╔══██╗██╔════╝
███████╗███████║██║██║   ██║██████╔╝███████║     ██║ ██████╔╝███████║   ██║   ███████║██████╔╝█████╗  
╚════██║██╔══██║██║╚██╗ ██╔╝██╔══██╗██╔══██║██   ██║ ██╔═══╝ ██╔══██║   ██║   ██╔══██║██╔══██╗██╔══╝  
███████║██║  ██║██║ ╚████╔╝ ██║  ██║██║  ██║╚█████╔╝ ██║     ██║  ██║   ██║   ██║  ██║██║  ██║███████╗
╚══════╝╚═╝  ╚═╝╚═╝  ╚═══╝  ╚═╝  ╚═╝╚═╝  ╚═╝ ╚════╝  ╚═╝     ╚═╝  ╚═╝   ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝
</pre>

<!-- Animated Tagline -->
<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Orbitron&size=28&duration=2500&pause=1000&color=00FF41&center=true&vCenter=true&width=700&lines=Automating+Intelligence;Engineering+AI+Systems;Building+Scalable+ML;From+Models+to+Production;Designing+Intelligent+Automation" />
</p>

Building scalable AI systems that move from research prototypes to real-world deployment.

🎓 Master's Student in Artificial Intelligence & Machine Learning  
⚙️ Focused on LLM Systems, Applied ML, and MLOps  
🚀 Passionate about building intelligent systems that create measurable impact  

---

## 🧠 About Me

I specialize in designing and deploying AI systems that go beyond experimentation.  
My work focuses on bridging the gap between research models and production-ready solutions.

I am particularly interested in:
- Large Language Models & Retrieval-Augmented Generation (RAG)
- ML system design & scalable inference
- End-to-end AI application development
- Automation through intelligent architectures

---

## ⭐ Featured Projects

### 🔹 AI Research Assistant (RAG System)
A document intelligence system powered by retrieval-augmented generation.
- Built with LangChain + Vector Database
- Reduced document retrieval latency by 60%
- Designed modular pipeline for scalable deployment
- Deployed using FastAPI + Docker

---

### 🔹 Time-Series Forecasting Engine
A predictive modeling system for structured temporal data.
- Feature engineering + model ensembling
- Improved baseline accuracy by 18%
- Designed reproducible experimentation workflow

---

### 🔹 End-to-End ML Deployment Pipeline
Automated training and deployment workflow.
- CI/CD integration for retraining
- Containerized model serving
- Cloud-ready architecture
- Monitoring-ready inference setup

---

## 🛠 Technical Stack

### Languages
Python • C++ • SQL

### Machine Learning
PyTorch • TensorFlow • Scikit-learn • XGBoost

### LLM & AI Systems
LangChain • RAG Architectures • Vector Databases • OpenAI APIs

### MLOps & Deployment
Docker • FastAPI • GitHub Actions • CI/CD Pipelines

### Cloud & Infrastructure
AWS • GCP

---

## 🔬 Currently Exploring

- Agentic AI systems
- Multi-modal learning architectures
- Efficient fine-tuning techniques (LoRA, PEFT)
- Scalable inference optimization

---

## 🧩 Engineering Philosophy

I focus on building AI systems that:

- Move beyond notebooks into production  
- Are modular, scalable, and reproducible  
- Solve real-world problems with measurable outcomes  
- Prioritize clarity, efficiency, and maintainability  

AI should not just be intelligent — it should be deployable.

---

## 📈 What I’m Looking For

I’m open to:
- AI research collaborations
- ML engineering roles
- High-impact AI startups
- System-focused AI development opportunities

---

## 🤝 Let’s Connect

If you're building intelligent systems or working on impactful AI problems,  
I'd love to collaborate.

📫 Email: shivrajpatare.work@gmail.com  
💼 LinkedIn: www.linkedin.com/in/shivrajpatare 
🌐 Portfolio: https://shivrajpatare-cli.vercel.app/ 

---
