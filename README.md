# 🤖 Smart RAG Chatbot - AI That Actually Knows Your Docs

A simple, intelligent chatbot that answers questions using your own documents. Built with AWS Bedrock and ChromaDB.

## 🎯 What Does This Do?

Imagine having a smart assistant that:
- Reads all your documentation
- Understands what you're asking (not just keywords)
- Gives you accurate answers based on your actual docs
- Remembers your conversation

That's exactly what this chatbot does!

## 🌟 Why Is This Cool?

**Traditional Chatbots**: "I don't know" or gives generic answers

**This RAG Chatbot**: Searches your documents, finds relevant info, and gives you accurate, specific answers

### Real Example:
- **You ask**: "What models are available in Bedrock?"
- **Bot thinks**: "Let me search the Bedrock documentation..."
- **Bot finds**: Relevant FAQ entries about available models
- **Bot answers**: "Based on the documentation, here are the available models..." ✅

## 🚀 Quick Start

### Prerequisites
- Python 3.8 or higher
- AWS Account (with Bedrock access)
- Basic command line knowledge

### Installation

1. **Clone this repository**
```bash
git clone <your-repo-url>
cd rag-chatbot
```

2. **Install required packages**
```bash
pip install streamlit boto3 chromadb
```

3. **Set up AWS credentials**
```bash
aws configure
```
Enter your:
- AWS Access Key ID
- AWS Secret Access Key
- Region (e.g., us-east-1)

4. **Run the chatbot**
```bash
streamlit run rag_chatbot_app.py
```

5. **Open your browser**
Go to `http://localhost:8501` and start chatting!

## 📁 Project Structure

```
rag-chatbot/
│
├── rag_chatbot_app.py          # The web interface (what you see)
├── rag_chatbot_lib.py          # The brain (AI logic)
├── data/
│   └── chroma/                 # Where documents are stored
├── README.md                   # You are here!
└── TECH_BLOG_POST.md          # Detailed technical explanation
```

## 🧠 How It Works (Simple Explanation)

Think of it like a smart librarian:

1. **You ask a question** → "What is AWS Bedrock?"

2. **The librarian (AI) thinks** → "Do I need to check the books?"

3. **Searches the library** → Looks through indexed documents

4. **Finds relevant pages** → Gets the 4 most relevant answers

5. **Reads and understands** → Combines info from multiple sources

6. **Gives you an answer** → Clear, accurate response based on your docs

### The Magic Behind It

```
Your Question
    ↓
AI Brain (Claude)
    ↓
"I need more info!" → Searches Document Database
    ↓
Finds Relevant Docs
    ↓
AI Brain (Claude) + Your Docs
    ↓
Smart Answer!
```

## 🛠️ Technology Stack (What's Under the Hood)

| Technology | What It Does | Why We Use It |
|------------|--------------|---------------|
| **AWS Bedrock (Claude)** | The AI brain that understands and responds | Super smart, can use tools, great at conversation |
| **AWS Bedrock (Titan)** | Converts text to numbers for searching | Helps find similar documents by meaning, not just words |
| **ChromaDB** | Stores and searches documents | Fast, easy to use, perfect for small to medium projects |
| **Streamlit** | Creates the web interface | Makes beautiful UIs with just Python code |

## 💡 Key Features

✅ **Smart Search**: Finds answers by meaning, not just matching words
✅ **Conversation Memory**: Remembers what you talked about
✅ **Accurate Answers**: Uses your actual documents, not made-up info
✅ **Easy to Use**: Simple chat interface, no technical knowledge needed
✅ **Customizable**: Add your own documents and knowledge base

## 📚 How to Add Your Own Documents

Want the chatbot to answer questions about YOUR documents? Here's how:

```python
import chromadb
import boto3
from chromadb.utils.embedding_functions import AmazonBedrockEmbeddingFunction

# Connect to the database
client = chromadb.PersistentClient(path="./data/chroma")

# Set up embeddings (converts text to searchable format)
embedding_function = AmazonBedrockEmbeddingFunction(
    session=boto3.Session(),
    model_name="amazon.titan-embed-text-v2:0"
)

# Create or get your collection
collection = client.get_or_create_collection(
    name="my_documents",
    embedding_function=embedding_function
)

# Add your documents
collection.add(
    documents=[
        "Your first document text here",
        "Your second document text here",
        "And so on..."
    ],
    ids=["doc1", "doc2", "doc3"]  # Unique ID for each document
)
```

Then update `rag_chatbot_lib.py` line 88 to use your collection name:
```python
collection = get_collection("../../data/chroma", "my_documents")
```

## 🎓 Learning Resources

New to these concepts? Here are simple explanations:

### What is RAG (Retrieval-Augmented Generation)?
Think of it as giving the AI a textbook during an exam. Instead of relying only on what it memorized, it can look up facts in real-time.

### What are Embeddings?
Converting text into numbers so computers can understand similarity. Like how "car" and "automobile" are different words but similar meanings.

### What is a Vector Database?
A special database that stores these number representations and can quickly find similar items.

## 🔧 Troubleshooting

### "Fatal: not a git repository"
Run: `git init` in your project folder

### "AWS credentials not found"
Run: `aws configure` and enter your credentials

### "Module not found"
Run: `pip install streamlit boto3 chromadb`

### Chatbot gives generic answers
Make sure your documents are properly indexed in ChromaDB

### Slow responses
Normal! AI processing + document search takes 2-5 seconds

## 📈 What's Next?

Want to improve this project? Here are ideas:

- [ ] Add more document types (PDFs, Word docs)
- [ ] Create a document upload interface
- [ ] Add user authentication
- [ ] Deploy to the cloud (AWS, Heroku)
- [ ] Add response caching for faster answers
- [ ] Support multiple languages
- [ ] Add voice input/output

## 🤝 Contributing

Found a bug? Have an idea? Contributions are welcome!

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📝 License

This project is open source and available under the MIT License.

## 🙋 FAQ

**Q: Do I need to pay for AWS?**
A: Yes, AWS Bedrock has usage-based pricing. Check AWS pricing page for details.

**Q: Can I use this for my company's internal docs?**
A: Absolutely! That's exactly what it's designed for.

**Q: How many documents can I add?**
A: ChromaDB can handle thousands of documents. For millions, consider upgrading to a managed vector database.

**Q: Is my data secure?**
A: Data stays in your AWS account and local ChromaDB. Nothing is shared externally.

**Q: Can I use a different AI model?**
A: Yes! Just change the `modelId` in the code to any Bedrock-supported model.

## 📞 Support

Need help? 
- Read the [detailed technical blog post](TECH_BLOG_POST.md)
- Check [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- Open an issue on GitHub

## ⭐ Show Your Support

If this project helped you, give it a ⭐️!

---

**Built with ❤️ using AWS Bedrock, ChromaDB, and Streamlit**

*Making AI accessible, one chatbot at a time* 🚀
