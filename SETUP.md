# 🛠️ Complete Setup Guide

Everything you need to get the Intelligent RAG system running on your machine or in the cloud.

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Local Installation](#local-installation)
3. [Configuration Guide](#configuration-guide)
4. [Running the System](#running-the-system)
5. [Cloud Deployment](#cloud-deployment)
6. [Optimization Tips](#optimization-tips)
7. [Common Issues](#common-issues)

---

## ✅ Prerequisites

### Hardware Requirements

**For Development:**
- CPU: 4+ cores
- RAM: 8GB minimum (16GB recommended)
- Storage: 20GB free space
- Network: Broadband internet

**For Production:**
- CPU: 8+ cores (or GPU for Ollama)
- RAM: 16GB minimum
- Storage: 50GB+ SSD
- Network: Low latency connection

### Software Requirements

- **Python 3.8+** - [Download](https://www.python.org/downloads/)
- **Git** - [Download](https://git-scm.com/downloads)
- **Ollama** - [Download](https://ollama.ai/)
- **Docker** (optional) - [Download](https://www.docker.com/)

---

## 🏠 Local Installation

### 1. Install Ollama

Ollama runs the AI model locally on your machine.

**On Linux:**
```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

**On macOS:**
```bash
# Download from https://ollama.ai/download/mac
# Or install via Homebrew:
brew install ollama
```

**On Windows:**
```bash
# Download installer from https://ollama.ai/download/windows
# Or use WSL2 and follow Linux instructions
```

**Verify installation:**
```bash
ollama --version
```

### 2. Download AI Model

```bash
# Download Llama 3 (8B parameters, ~4.7GB)
ollama pull llama3

# Verify download
ollama list
```

**Alternative models:**
```bash
# Smaller (faster, less accurate)
ollama pull phi

# Larger (slower, more accurate)  
ollama pull llama3:70b

# Different architecture
ollama pull mistral
```

### 3. Clone Repository

```bash
git clone https://github.com/nobitanobi22/CorrectiveRAG.git
cd CorrectiveRAG
```

### 4. Setup Python Environment

**Create virtual environment:**
```bash
python3 -m venv venv
```

**Activate it:**
```bash
# Linux/Mac:
source venv/bin/activate

# Windows CMD:
venv\Scripts\activate.bat

# Windows PowerShell:
venv\Scripts\Activate.ps1
```

**Install packages:**
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

**Verify installation:**
```bash
python -c "import langchain; import streamlit; print('✅ All good!')"
```

---

## ⚙️ Configuration Guide

### Get API Keys

**1. Google Custom Search (Free)**

Create a Custom Search Engine:
1. Go to [Google CSE](https://cse.google.com/cse/all)
2. Click "Add"
3. Name it anything (e.g., "RAG Search")
4. Sites to search: Leave empty or add "*"
5. Click "Create"
6. Copy your **Search engine ID** (CSE ID)

Get API Key:
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Enable "Custom Search API"
3. Create credentials → API key
4. Copy your **API key**

**2. OpenAI (Optional - Only for Evaluation)**

1. Go to [OpenAI Platform](https://platform.openai.com/)
2. Create API key
3. Copy key and organization ID

### Configure application.yaml

```yaml
# Required: Google Search API
GOOGLE_CSE_ID: "paste_your_search_engine_id_here"
GOOGLE_API_KEY: "paste_your_google_api_key_here"

# Required: Ollama URL
ollama_url: "http://localhost:11434"

# Optional: OpenAI (only needed for evaluate.py)
OPENAI_API_KEY: "paste_your_openai_key_here"
OPENAI_ORGANIZATION: "paste_your_org_id_here"
```

**Security Note:** Never commit this file with real keys to GitHub!

### Customize Prompts (Optional)

Edit `prompts.yaml` to change how agents behave:

```yaml
# Make document validator more strict
document_grader: |
  You assess document relevance with high standards.
  Only mark as relevant if document DIRECTLY answers the question.
  
  Document: {document}
  Question: {question}
  
  Return: {"score": "yes"} or {"score": "no"}
```

---

## 🚀 Running the System

### Start Ollama

```bash
# Ollama should auto-start, but if not:
ollama serve
```

**Verify it's running:**
```bash
curl http://localhost:11434/api/version
```

### Launch the App

```bash
# Make sure you're in the project directory
cd CorrectiveRAG

# Activate virtual environment if not already
source venv/bin/activate  # Linux/Mac
# or
venv\Scripts\activate  # Windows

# Run Streamlit app
streamlit run app.py
```

### Access the Interface

Browser should open automatically at:
```
http://localhost:8501
```

If not, open manually.

### Using the App

**Step 1:** Upload documents
- Click "Upload" in sidebar
- Select PDF, TXT, or DOCX files
- Or paste URLs

**Step 2:** Ask questions
- Type your question in the input box
- Press Enter or click "Submit"

**Step 3:** Get validated answers
- System retrieves relevant content
- Validates quality automatically
- Returns fact-checked response

---

## ☁️ Cloud Deployment

### AWS Deployment

**Architecture:**
```
┌─────────────────┐      ┌──────────────────┐
│  App Server     │─────▶│  Ollama Server   │
│  t3.medium      │      │  g4dn.xlarge+GPU │
│  Streamlit      │      │  Llama 3 Model   │
└─────────────────┘      └──────────────────┘
```

**Step 1: Launch Ollama Server**

```bash
# EC2 Instance Settings:
# - Type: g4dn.xlarge (GPU)
# - AMI: Ubuntu 22.04 LTS
# - Storage: 50GB
# - Security: Port 11434 open

# SSH into instance
ssh ubuntu@<ollama-server-ip>

# Install Ollama
curl -fsSL https://ollama.ai/install.sh | sh

# Download model
ollama pull llama3

# Make it accessible from network
export OLLAMA_HOST=0.0.0.0:11434
ollama serve
```

**Step 2: Launch App Server**

```bash
# EC2 Instance Settings:
# - Type: t3.medium
# - AMI: Ubuntu 22.04 LTS
# - Storage: 20GB
# - Security: Ports 8501, 8080 open

# SSH into instance
ssh ubuntu@<app-server-ip>

# Install Docker
curl -fsSL https://get.docker.com | sh

# Clone your repo
git clone https://github.com/nobitanobi22/CorrectiveRAG.git
cd CorrectiveRAG

# Update application.yaml
nano application.yaml
# Change: ollama_url: "http://<ollama-private-ip>:11434"

# Run with Docker
docker build -t intelligent-rag .
docker run -d -p 8501:8501 intelligent-rag
```

**Step 3: Setup Load Balancer (Optional)**

- Create Application Load Balancer
- Target port 8501
- Add health checks
- Point domain to ALB

---

## ⚡ Optimization Tips

### Speed Up Responses

**1. Use GPU for Ollama**
```bash
# Check GPU availability
nvidia-smi

# Ollama will automatically use GPU if available
```

**2. Reduce iterations**
```yaml
# In application.yaml
workflow:
  max_iterations: 1  # Instead of 3
```

**3. Retrieve fewer documents**
```yaml
workflow:
  retrieval_k: 2  # Instead of 4
```

**4. Use smaller model**
```bash
ollama pull llama3:8b  # Smaller than default
```

### Improve Accuracy

**1. Increase iterations**
```yaml
workflow:
  max_iterations: 5
```

**2. Retrieve more context**
```yaml
workflow:
  retrieval_k: 6
```

**3. Use larger chunks**
```yaml
document_settings:
  chunk_size: 1500  # Instead of 1000
```

### Save Costs

**1. Use local models only**
- Don't use OpenAI API
- Ollama is completely free

**2. Optimize cloud instances**
- Use spot instances for Ollama
- Scale down when not in use

**3. Cache responses**
```python
# Add to rag_app.py
from functools import lru_cache

@lru_cache(maxsize=50)
def cached_answer(question):
    # Cache frequent questions
    pass
```

---

## 🐛 Common Issues

### Issue: "Ollama connection refused"

**Solution:**
```bash
# Check if Ollama is running
ollama list

# Start Ollama
ollama serve

# Check port
lsof -i :11434  # Linux/Mac
netstat -an | find "11434"  # Windows
```

### Issue: "Google API quota exceeded"

**Solution:**
```bash
# Check quota at: https://console.cloud.google.com/
# Free tier: 100 queries/day
# Either wait or upgrade to paid tier
```

### Issue: "Out of memory"

**Solution:**
```bash
# Close other applications
# Use smaller model:
ollama pull phi

# Reduce chunk size in application.yaml
```

### Issue: "ChromaDB errors"

**Solution:**
```bash
# Delete and recreate database
rm -rf chroma_db/

# Restart app
streamlit run app.py
```

### Issue: "Slow responses"

**Solutions:**
1. Enable GPU for Ollama
2. Reduce `max_iterations`
3. Use smaller model
4. Decrease `retrieval_k`
5. Reduce `chunk_size`

### Issue: "Dependencies won't install"

**Solution:**
```bash
# Upgrade pip
pip install --upgrade pip

# Install one by one to find culprit
pip install langchain
pip install streamlit
# ... etc

# Or use specific versions
pip install langchain==0.1.0
```

---

## 🔒 Security Best Practices

### Protect API Keys

```bash
# Never commit application.yaml with real keys
echo "application.yaml" >> .gitignore

# Use environment variables instead
export GOOGLE_API_KEY="your-key"
export GOOGLE_CSE_ID="your-id"
```

### Secure Cloud Deployment

1. **Use Security Groups**
   - Only allow necessary ports
   - Restrict IP ranges

2. **Enable HTTPS**
   - Use SSL certificate
   - Redirect HTTP to HTTPS

3. **Rotate Keys Regularly**
   - Change API keys monthly
   - Use AWS Secrets Manager

4. **Monitor Access**
   - Enable CloudWatch logs
   - Set up alerts

---

## 📊 Running Evaluations

```bash
# Make sure you have OpenAI API key set
python evaluate.py
```

Output shows:
- Faithfulness score
- Answer relevancy
- Context precision
- Context recall
- Comparison with baseline

---

## 🆘 Getting Help

If you're stuck:

1. **Check this guide** - Most issues are covered
2. **Read error messages** - They usually explain the problem
3. **Search GitHub issues** - Someone may have had same issue
4. **Open new issue** - [Create issue](https://github.com/nobitanobi22/CorrectiveRAG/issues)

---

<div align="center">

**Ready to build amazing RAG systems!** 🚀

[← Back to README](README.md)

</div>
