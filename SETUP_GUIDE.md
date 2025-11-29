# 🚀 Complete Setup Guide - Step by Step

This guide will help you set up the RAG Chatbot from scratch, even if you're new to coding!

## 📋 What You'll Need

- A computer (Windows, Mac, or Linux)
- Internet connection
- 30 minutes of your time
- An AWS account (we'll help you set this up)

## Step 1: Install Python 🐍

### Windows:
1. Go to [python.org/downloads](https://www.python.org/downloads/)
2. Download Python 3.8 or higher
3. Run the installer
4. ⚠️ **IMPORTANT**: Check "Add Python to PATH" during installation
5. Click "Install Now"

### Mac:
```bash
# Open Terminal and run:
brew install python3
```

### Verify Installation:
```bash
python --version
# Should show: Python 3.8.x or higher
```

## Step 2: Set Up AWS Account ☁️

### Create AWS Account:
1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click "Create an AWS Account"
3. Follow the signup process (you'll need a credit card)
4. Complete identity verification

### Enable AWS Bedrock:
1. Log into AWS Console
2. Search for "Bedrock" in the search bar
3. Click on "Amazon Bedrock"
4. Go to "Model access" in the left sidebar
5. Click "Manage model access"
6. Enable these models:
   - ✅ Claude 3.7 Sonnet
   - ✅ Titan Embeddings v2
7. Click "Save changes"
8. Wait 2-5 minutes for access to be granted

### Create Access Keys:
1. In AWS Console, click your name (top right)
2. Click "Security credentials"
3. Scroll to "Access keys"
4. Click "Create access key"
5. Choose "Command Line Interface (CLI)"
6. Check the confirmation box
7. Click "Create access key"
8. **IMPORTANT**: Save both:
   - Access Key ID (looks like: AKIAIOSFODNN7EXAMPLE)
   - Secret Access Key (looks like: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY)
9. ⚠️ Keep these secret! Don't share them!

## Step 3: Install AWS CLI 🔧

### Windows:
1. Download from [AWS CLI Installer](https://awscli.amazonaws.com/AWSCLIV2.msi)
2. Run the installer
3. Follow the prompts

### Mac:
```bash
brew install awscli
```

### Verify Installation:
```bash
aws --version
# Should show: aws-cli/2.x.x
```

### Configure AWS CLI:
```bash
aws configure
```

You'll be asked for:
```
AWS Access Key ID: [paste your Access Key ID]
AWS Secret Access Key: [paste your Secret Access Key]
Default region name: us-east-1
Default output format: json
```

## Step 4: Download the Project 📥

### Option A: Using Git (Recommended)

1. **Install Git**:
   - Windows: Download from [git-scm.com](https://git-scm.com)
   - Mac: `brew install git`

2. **Clone the repository**:
```bash
# Navigate to where you want the project
cd Desktop

# Clone the repository (replace with your actual repo URL)
git clone https://github.com/yourusername/rag-chatbot.git

# Enter the project folder
cd rag-chatbot
```

### Option B: Download ZIP

1. Go to the GitHub repository
2. Click the green "Code" button
3. Click "Download ZIP"
4. Extract the ZIP file
5. Open terminal/command prompt in that folder

## Step 5: Install Python Packages 📦

```bash
# Make sure you're in the project folder
cd rag-chatbot

# Install required packages
pip install -r requirements.txt
```

This will install:
- Streamlit (for the web interface)
- Boto3 (for AWS connection)
- ChromaDB (for document storage)

**If you get an error**, try:
```bash
pip3 install -r requirements.txt
```

## Step 6: Prepare Your Documents 📚

### Create the data folder:
```bash
mkdir -p data/chroma
```

### Add documents to ChromaDB:

Create a file called `setup_documents.py`:

```python
import chromadb
import boto3
from chromadb.utils.embedding_functions import AmazonBedrockEmbeddingFunction

# Connect to ChromaDB
client = chromadb.PersistentClient(path="./data/chroma")

# Set up embeddings
session = boto3.Session()
embedding_function = AmazonBedrockEmbeddingFunction(
    session=session,
    model_name="amazon.titan-embed-text-v2:0"
)

# Create collection
collection = client.get_or_create_collection(
    name="bedrock_faqs_collection",
    embedding_function=embedding_function
)

# Add sample documents
documents = [
    "Amazon Bedrock is a fully managed service that offers a choice of high-performing foundation models.",
    "Claude 3.7 Sonnet is available in Amazon Bedrock with advanced reasoning capabilities.",
    "Titan Embeddings v2 can convert text into vector representations for semantic search.",
    # Add more of your documents here
]

ids = [f"doc_{i}" for i in range(len(documents))]

collection.add(
    documents=documents,
    ids=ids
)

print(f"✅ Successfully added {len(documents)} documents!")
```

Run it:
```bash
python setup_documents.py
```

## Step 7: Run the Chatbot! 🎉

```bash
streamlit run rag_chatbot_app.py
```

You should see:
```
You can now view your Streamlit app in your browser.

Local URL: http://localhost:8501
Network URL: http://192.168.x.x:8501
```

Open your browser and go to `http://localhost:8501`

## Step 8: Test It Out 🧪

Try asking:
- "What is Amazon Bedrock?"
- "Tell me about Claude models"
- "How do embeddings work?"

The chatbot should search your documents and give you answers!

## 🎯 Success Checklist

- [ ] Python installed and working
- [ ] AWS account created
- [ ] Bedrock models enabled
- [ ] AWS CLI configured with credentials
- [ ] Project downloaded
- [ ] Python packages installed
- [ ] Documents added to ChromaDB
- [ ] Chatbot running on localhost:8501
- [ ] Successfully got a response from the chatbot

## 🐛 Common Issues & Solutions

### "Command not found: python"
**Solution**: Try `python3` instead of `python`

### "AWS credentials not configured"
**Solution**: Run `aws configure` again and enter your credentials

### "Access denied to Bedrock"
**Solution**: 
1. Check if you enabled model access in Bedrock console
2. Wait 5 minutes and try again
3. Verify your AWS region is correct

### "Module not found: streamlit"
**Solution**: 
```bash
pip install streamlit boto3 chromadb
```

### "Port 8501 already in use"
**Solution**: 
```bash
streamlit run rag_chatbot_app.py --server.port 8502
```

### Chatbot gives generic answers
**Solution**: Make sure you ran `setup_documents.py` to add documents

### "Connection timeout" errors
**Solution**: Check your internet connection and AWS credentials

## 🚀 Next Steps

Now that it's working:

1. **Add more documents**: Edit `setup_documents.py` with your own content
2. **Customize the UI**: Modify `rag_chatbot_app.py` to change colors, title, etc.
3. **Share with others**: Deploy to Streamlit Cloud (free!)
4. **Learn more**: Read the [Technical Blog Post](TECH_BLOG_POST.md)

## 📞 Need Help?

- Check the [main README](README.md)
- Read the [FAQ section](README.md#-faq)
- Open an issue on GitHub
- Check AWS Bedrock documentation

## 🎓 Understanding What You Built

You now have:
- ✅ A web-based chatbot interface
- ✅ AI-powered question answering
- ✅ Document search capability
- ✅ Conversation memory
- ✅ AWS cloud integration

**Congratulations! You're now running your own AI chatbot!** 🎉

---

*Having trouble? Don't worry! Even experienced developers run into issues. Take it step by step, and you'll get there!*
