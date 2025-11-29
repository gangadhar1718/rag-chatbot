# Building an Intelligent RAG Chatbot with AWS Bedrock and ChromaDB

## 1. Problem & Solution

### The Problem
Organizations often struggle with information accessibility. Documentation, FAQs, and knowledge bases can be extensive, making it time-consuming for users to find specific answers. Traditional search methods rely on keyword matching, which often misses contextually relevant information. Additionally, generic AI chatbots lack domain-specific knowledge and can provide inaccurate or outdated information.

### The Solution
I built a **Retrieval-Augmented Generation (RAG) chatbot** that combines the power of large language models with a curated knowledge base. This system:

- **Understands natural language queries** using AWS Bedrock's Claude 3.7 Sonnet
- **Retrieves relevant information** from a vector database using semantic search
- **Generates accurate, context-aware responses** by grounding AI outputs in factual documentation
- **Provides a user-friendly interface** through Streamlit for easy interaction

### Who Benefits?
- **End Users**: Get instant, accurate answers without manually searching through documentation
- **Support Teams**: Reduce repetitive queries by enabling self-service support
- **Organizations**: Improve knowledge accessibility while maintaining accuracy and reducing support costs
- **Developers**: Learn AWS Bedrock capabilities through an interactive, intelligent assistant

## 2. Technical Implementation

### Architecture Overview

```
User Query → Streamlit UI → AWS Bedrock (Claude) → Tool Decision
                                    ↓
                            Tool: get_amazon_bedrock_information
                                    ↓
                            ChromaDB Vector Search
                                    ↓
                            Retrieve Top 4 FAQs
                                    ↓
                            AWS Bedrock (Claude) + Context
                                    ↓
                            Final Response → User
```

### AWS Services Used

#### 1. **AWS Bedrock - Claude 3.7 Sonnet**
**Why**: State-of-the-art language model with excellent reasoning capabilities and tool-use functionality.

**How it's used**:
- Primary conversational AI engine
- Decides when to retrieve information from the knowledge base
- Synthesizes retrieved context into coherent responses
- Maintains conversation history for contextual understanding

**Configuration**:
```python
response = bedrock.converse(
    modelId="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    messages=messages,
    inferenceConfig={
        "maxTokens": 2000,      # Response length limit
        "temperature": 0,        # Deterministic responses
        "topP": 0.9,            # Nucleus sampling
        "stopSequences": []
    },
    toolConfig={
        "tools": tool_list      # Enable RAG tool
    }
)
```

#### 2. **AWS Bedrock - Titan Embeddings v2**
**Why**: Converts text into high-dimensional vectors for semantic similarity search.

**How it's used**:
- Embeds user queries into vector space
- Embeds FAQ documents during indexing
- Enables semantic search (finds meaning, not just keywords)

```python
embedding_function = AmazonBedrockEmbeddingFunction(
    session=session, 
    model_name="amazon.titan-embed-text-v2:0"
)
```

### Core Components

#### **Vector Database - ChromaDB**
**Why**: Efficient, open-source vector database for semantic search.

**How it works**:
```python
def get_vector_search_results(collection, question):
    # Semantic search: finds documents similar in meaning to the question
    results = collection.query(
        query_texts=[question],
        n_results=4  # Return top 4 most relevant documents
    )
    return results
```

#### **Tool-Use Pattern**
The system implements AWS Bedrock's tool-use capability, allowing Claude to decide when it needs additional information:

```python
tools = [
    {
        "toolSpec": {
            "name": "get_amazon_bedrock_information",
            "description": "Retrieve information about Amazon Bedrock, a managed service for hosting generative AI models.",
            "inputSchema": {
                "json": {
                    "type": "object",
                    "properties": {
                        "query": {
                            "type": "string",
                            "description": "The retrieval-augmented generation query used to look up information in a repository of FAQs about Amazon Bedrock."
                        }
                    },
                    "required": ["query"]
                }
            }
        }
    }
]
```

**Flow**:
1. User asks: "What models are available in Bedrock?"
2. Claude recognizes it needs specific information
3. Claude invokes `get_amazon_bedrock_information` tool with optimized query
4. System retrieves relevant FAQs from ChromaDB
5. Claude receives context and generates accurate response

#### **Conversation Management**
```python
MAX_MESSAGES = 20

# Maintain conversation history with automatic pruning
if number_of_messages > MAX_MESSAGES:
    # Remove oldest messages (both user and assistant)
    del message_history[0 : (number_of_messages - MAX_MESSAGES) * 2]
```

This prevents context window overflow while maintaining recent conversation context.

### System Workflow

**Step 1: User Input Processing**
```python
# Streamlit captures user input
input_text = st.chat_input("Chat with your bot here")

if input_text:
    # Pass to backend for processing
    glib.chat_with_model(
        message_history=st.session_state.chat_history, 
        new_text=input_text
    )
```

**Step 2: LLM Decision Making**
```python
# Claude decides if it needs to search the knowledge base
response = bedrock.converse(
    modelId="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    messages=messages,
    toolConfig={"tools": tool_list}
)
```

**Step 3: RAG Retrieval (if needed)**
```python
def process_tool(response_message, messages, bedrock, tool_list):
    # Check if Claude wants to use the RAG tool
    for content_block in response_content_blocks:
        if 'toolUse' in content_block:
            # Extract the query Claude generated
            query = tool_use_block['input']['query']
            
            # Search vector database
            search_results = get_vector_search_results(collection, query)
            
            # Flatten and format results
            flattened_results_list = list(itertools.chain(*search_results['documents']))
            rag_content = "\n\n".join(flattened_results_list)
            
            # Return context to Claude
            follow_up_content_blocks.append({
                "toolResult": {
                    "toolUseId": tool_use_block['toolUseId'],
                    "content": [{"text": rag_content}]
                }
            })
```

**Step 4: Final Response Generation**
```python
# Claude generates response using retrieved context
response = bedrock.converse(
    modelId="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
    messages=messages,  # Now includes tool results
    inferenceConfig={...},
    toolConfig={"tools": tool_list}
)
```

## 3. Scaling Strategy

### Current Capacity

**Performance Characteristics**:
- **Response Time**: 2-5 seconds per query (including RAG retrieval)
- **Concurrent Users**: Supports multiple users via Streamlit's session management
- **Knowledge Base**: Currently indexes Amazon Bedrock FAQs (~100-500 documents)
- **Vector Search**: Sub-second retrieval of top 4 relevant documents
- **Conversation History**: Maintains last 20 messages per session

**Current Limitations**:
- Single ChromaDB instance (local persistence)
- Streamlit runs on single server
- No caching layer for repeated queries
- AWS Bedrock API rate limits apply

### Future Growth Plans

#### Phase 1: Performance Optimization (0-3 months)
- **Implement caching**: Redis layer for frequently asked questions
- **Optimize embeddings**: Batch processing for faster indexing
- **Add monitoring**: CloudWatch metrics for latency and error rates
- **Response streaming**: Stream Claude responses for better UX

#### Phase 2: Scalability (3-6 months)
- **Containerization**: Docker + ECS for horizontal scaling
- **Load balancing**: Application Load Balancer for traffic distribution
- **Distributed vector DB**: Migrate to managed vector database (e.g., Amazon OpenSearch with vector engine or Pinecone)
- **Multi-region deployment**: Reduce latency for global users

#### Phase 3: Advanced Features (6-12 months)
- **Multi-domain support**: Expand beyond Bedrock FAQs to multiple knowledge bases
- **Hybrid search**: Combine semantic search with keyword filtering
- **User feedback loop**: Collect ratings to improve retrieval quality
- **Fine-tuning**: Custom embeddings for domain-specific terminology
- **Analytics dashboard**: Track popular queries, response quality, user satisfaction

#### Phase 4: Enterprise Features (12+ months)
- **Multi-tenancy**: Isolated knowledge bases per organization
- **Access control**: Role-based permissions for sensitive information
- **Audit logging**: Compliance tracking for regulated industries
- **API gateway**: RESTful API for integration with other systems
- **Advanced RAG**: Implement re-ranking, query expansion, and citation tracking

### Scaling Architecture (Future State)

```
                    ┌─────────────────┐
                    │  CloudFront CDN │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   ALB (Load     │
                    │   Balancer)     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         ┌────▼────┐    ┌────▼────┐   ┌────▼────┐
         │ ECS     │    │ ECS     │   │ ECS     │
         │ Task 1  │    │ Task 2  │   │ Task 3  │
         └────┬────┘    └────┬────┘   └────┬────┘
              │              │              │
              └──────────────┼──────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         ┌────▼────────┐ ┌──▼──────────┐ ┌─▼──────────┐
         │ ElastiCache │ │ OpenSearch  │ │  Bedrock   │
         │   (Redis)   │ │  (Vectors)  │ │   (LLM)    │
         └─────────────┘ └─────────────┘ └────────────┘
```

## 4. Code & Resources

### Key Code Snippets

#### 1. **Setting Up Vector Database Connection**
```python
def get_collection(path, collection_name):
    """
    Initialize ChromaDB client with AWS Bedrock embeddings
    
    Args:
        path: Local path to ChromaDB persistence directory
        collection_name: Name of the collection to retrieve
    
    Returns:
        ChromaDB collection object with embedding function
    """
    # Create AWS session for Bedrock access
    session = boto3.Session()
    
    # Initialize Titan Embeddings v2 for vector generation
    embedding_function = AmazonBedrockEmbeddingFunction(
        session=session, 
        model_name="amazon.titan-embed-text-v2:0"
    )
    
    # Connect to persistent ChromaDB instance
    client = chromadb.PersistentClient(path=path)
    
    # Get collection with automatic embedding on queries
    collection = client.get_collection(
        collection_name, 
        embedding_function=embedding_function
    )
    
    return collection
```

#### 2. **Semantic Search Implementation**
```python
def get_vector_search_results(collection, question):
    """
    Perform semantic search on vector database
    
    Args:
        collection: ChromaDB collection object
        question: User's natural language query
    
    Returns:
        Dictionary containing matched documents and metadata
    """
    # ChromaDB automatically embeds the query and finds similar vectors
    results = collection.query(
        query_texts=[question],  # Can batch multiple queries
        n_results=4              # Return top 4 most similar documents
    )
    
    return results
```

#### 3. **Tool Definition for Claude**
```python
def get_tools():
    """
    Define tools that Claude can invoke during conversation
    
    Returns:
        List of tool specifications in Bedrock format
    """
    tools = [
        {
            "toolSpec": {
                "name": "get_amazon_bedrock_information",
                "description": "Retrieve information about Amazon Bedrock, a managed service for hosting generative AI models.",
                "inputSchema": {
                    "json": {
                        "type": "object",
                        "properties": {
                            "query": {
                                "type": "string",
                                "description": "The retrieval-augmented generation query used to look up information in a repository of FAQs about Amazon Bedrock."
                            }
                        },
                        "required": ["query"]
                    }
                }
            }
        }
    ]
    return tools
```

#### 4. **Processing Tool Invocations**
```python
def process_tool(response_message, messages, bedrock, tool_list):
    """
    Handle Claude's tool invocations and return results
    
    Args:
        response_message: Claude's response containing potential tool use
        messages: Conversation history
        bedrock: Bedrock client
        tool_list: Available tools
    
    Returns:
        Tuple of (tool_used: bool, response_text: str)
    """
    # Add Claude's response to conversation
    messages.append(response_message)
    
    response_content_blocks = response_message['content']
    follow_up_content_blocks = []
    
    # Check if Claude wants to use any tools
    for content_block in response_content_blocks:
        if 'toolUse' in content_block:
            tool_use_block = content_block['toolUse']
            
            # Handle the RAG tool
            if tool_use_block['name'] == 'get_amazon_bedrock_information':
                # Get vector database collection
                collection = get_collection("../../data/chroma", "bedrock_faqs_collection")
                
                # Extract Claude's optimized query
                query = tool_use_block['input']['query']
                print(f"----QUERY:----\n{query}")
                
                # Perform semantic search
                search_results = get_vector_search_results(collection, query)
                
                # Flatten nested list structure from ChromaDB
                flattened_results_list = list(itertools.chain(*search_results['documents']))
                
                # Combine all retrieved documents
                rag_content = "\n\n".join(flattened_results_list)
                print(f"----RAG CONTENT----\n{rag_content}")
                
                # Format tool result for Claude
                follow_up_content_blocks.append({
                    "toolResult": {
                        "toolUseId": tool_use_block['toolUseId'],
                        "content": [{"text": rag_content}]
                    }
                })
    
    # If tool was used, send results back to Claude
    if len(follow_up_content_blocks) > 0:
        follow_up_message = {
            "role": "user",
            "content": follow_up_content_blocks,
        }
        messages.append(follow_up_message)
        
        # Get final response with RAG context
        response = bedrock.converse(
            modelId="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
            messages=messages,
            inferenceConfig={
                "maxTokens": 2000,
                "temperature": 0,
                "topP": 0.9,
                "stopSequences": []
            },
            toolConfig={"tools": tool_list}
        )
        
        return True, response['output']['message']['content'][0]['text']
    else:
        return False, None
```

#### 5. **Main Chat Function**
```python
def chat_with_model(message_history, new_text=None):
    """
    Main function to handle user messages and generate responses
    
    Args:
        message_history: List of ChatMessage objects
        new_text: User's new message
    """
    # Initialize AWS Bedrock client
    session = boto3.Session()
    bedrock = session.client(service_name='bedrock-runtime')
    
    # Get available tools
    tool_list = get_tools()
    
    # Add user message to history
    new_text_message = ChatMessage('user', text=new_text)
    message_history.append(new_text_message)
    
    # Prune old messages to stay within context limits
    number_of_messages = len(message_history)
    if number_of_messages > MAX_MESSAGES:
        # Remove oldest pairs (user + assistant messages)
        del message_history[0 : (number_of_messages - MAX_MESSAGES) * 2]
    
    # Convert to Bedrock API format
    messages = convert_chat_messages_to_converse_api(message_history)
    
    # Initial call to Claude
    response = bedrock.converse(
        modelId="us.anthropic.claude-3-7-sonnet-20250219-v1:0",
        messages=messages,
        inferenceConfig={
            "maxTokens": 2000,
            "temperature": 0,      # Deterministic for consistency
            "topP": 0.9,
            "stopSequences": []
        },
        toolConfig={"tools": tool_list}
    )
    
    response_message = response['output']['message']
    
    # Process any tool invocations
    tool_used, output = process_tool(response_message, messages, bedrock, tool_list)
    
    # Use original response if no tool was needed
    if not tool_used:
        output = response['output']['message']['content'][0]['text']
    
    print(f"----FINAL RESPONSE----\n{output}")
    
    # Add assistant response to history
    response_chat_message = ChatMessage('assistant', output)
    message_history.append(response_chat_message)
```

#### 6. **Streamlit UI Setup**
```python
import streamlit as st
import rag_chatbot_lib as glib

# Configure page
st.set_page_config(page_title="RAG Chatbot")
st.title("RAG Chatbot")

# Initialize chat history in session state
if 'chat_history' not in st.session_state:
    st.session_state.chat_history = []

# Create container for chat messages
chat_container = st.container()

# Chat input box
input_text = st.chat_input("Chat with your bot here")

# Process new messages
if input_text:
    glib.chat_with_model(
        message_history=st.session_state.chat_history, 
        new_text=input_text
    )

# Render chat history (Streamlit reruns on each interaction)
for message in st.session_state.chat_history:
    with chat_container.chat_message(message.role):
        st.markdown(message.text)
```

### Repository Structure
```
rag-chatbot/
├── rag_chatbot_app.py          # Streamlit frontend
├── rag_chatbot_lib.py          # Backend logic and AWS integration
├── data/
│   └── chroma/                 # ChromaDB persistence directory
│       └── bedrock_faqs_collection/
├── requirements.txt            # Python dependencies
├── README.md                   # Project documentation
└── .env                        # AWS credentials (not in repo)
```

### Dependencies (requirements.txt)
```txt
streamlit==1.29.0
boto3==1.34.0
chromadb==0.4.22
```

### Setup Instructions

1. **Install dependencies**:
```bash
pip install -r requirements.txt
```

2. **Configure AWS credentials**:
```bash
aws configure
# Enter your AWS Access Key ID, Secret Access Key, and region
```

3. **Prepare your knowledge base**:
```python
# Example: Index documents into ChromaDB
import chromadb
from chromadb.utils.embedding_functions import AmazonBedrockEmbeddingFunction

client = chromadb.PersistentClient(path="./data/chroma")
embedding_function = AmazonBedrockEmbeddingFunction(
    session=boto3.Session(),
    model_name="amazon.titan-embed-text-v2:0"
)

collection = client.create_collection(
    name="bedrock_faqs_collection",
    embedding_function=embedding_function
)

# Add documents
collection.add(
    documents=["FAQ content here..."],
    ids=["faq_1"]
)
```

4. **Run the application**:
```bash
streamlit run rag_chatbot_app.py
```

5. **Access the chatbot**:
Open your browser to `http://localhost:8501`


### Additional Resources
- [AWS Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)
- [ChromaDB Documentation](https://docs.trychroma.com/)
- [Streamlit Documentation](https://docs.streamlit.io/)
- [RAG Best Practices](https://www.anthropic.com/index/retrieval-augmented-generation)

---

## Conclusion

This RAG chatbot demonstrates how to build production-ready AI applications by combining:
- **Powerful LLMs** (AWS Bedrock Claude) for natural language understanding
- **Vector databases** (ChromaDB) for semantic search
- **Tool-use patterns** for dynamic information retrieval
- **User-friendly interfaces** (Streamlit) for accessibility

The architecture is designed to scale from prototype to production, with clear paths for optimization and feature expansion. Whether you're building customer support bots, internal knowledge assistants, or domain-specific AI tools, this pattern provides a solid foundation.

**Key Takeaways**:
- RAG significantly improves accuracy by grounding responses in factual data
- Tool-use allows LLMs to decide when they need additional context
- Vector search enables semantic understanding beyond keyword matching
- AWS Bedrock simplifies LLM deployment without infrastructure management

Feel free to adapt this architecture for your specific use case and scale it according to your needs!

---

*Built with ❤️ using AWS Bedrock, ChromaDB, and Streamlit*
