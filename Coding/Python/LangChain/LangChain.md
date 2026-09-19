---
tags: [coding, python, llm, langchain, ai]
type: cheatsheet
source: compiled reference (LangChain)
last-verified: 2026-08-27
---

# LangChain

## Up
- [[Python]]

LangChain is a framework for building applications on top of LLMs — composing prompts, models, tools, retrieval, memory, and agents. Modern LangChain centres on **LCEL** (LangChain Expression Language) and the `Runnable` interface, with modular packages. Install: `pip install langchain langchain-openai langchain-community`.

> The LangChain API has changed substantially across versions; this reflects the LCEL-era (`langchain` 0.1+/0.2+) idioms. Always check the version you're on.

---

## Package Layout

| Package | Contains |
|---|---|
| `langchain-core` | Base abstractions: Runnable, messages, prompts, output parsers |
| `langchain` | Chains, agents, retrieval orchestration |
| `langchain-community` | Third-party integrations (vector stores, loaders, tools) |
| `langchain-openai`, `langchain-anthropic`, … | Provider-specific model classes |
| `langgraph` | Stateful, graph-based agent/workflow orchestration |

---

## Chat Models & Messages

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
# llm = ChatAnthropic(model="claude-...", temperature=0)

from langchain_core.messages import SystemMessage, HumanMessage, AIMessage
resp = llm.invoke([
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="Explain vectors in one line."),
])
print(resp.content)

llm.invoke("just a string works too")
llm.stream("stream tokens")          # generator of chunks
llm.batch(["q1", "q2"])              # parallel
await llm.ainvoke("async")           # async variants: ainvoke/astream/abatch
```

---

## Prompt Templates

```python
from langchain_core.prompts import ChatPromptTemplate, PromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an expert in {domain}."),
    ("human", "{question}"),
])
prompt.invoke({"domain": "physics", "question": "What is entropy?"})

# with a running message history placeholder
from langchain_core.prompts import MessagesPlaceholder
prompt = ChatPromptTemplate.from_messages([
    ("system", "Be concise."),
    MessagesPlaceholder("history"),
    ("human", "{input}"),
])
```

---

## LCEL — Composing with `|`

The core idea: chain `Runnable`s with the pipe operator. Output of each step feeds the next.

```python
from langchain_core.output_parsers import StrOutputParser

chain = prompt | llm | StrOutputParser()
chain.invoke({"domain": "physics", "question": "What is entropy?"})

# every Runnable supports the same interface
chain.stream({...}); chain.batch([{...}, {...}]); await chain.ainvoke({...})
```

```python
# parallel branches + passthrough
from langchain_core.runnables import RunnableParallel, RunnablePassthrough

chain = RunnableParallel(
    joke=(ChatPromptTemplate.from_template("Joke about {topic}") | llm | StrOutputParser()),
    fact=(ChatPromptTemplate.from_template("Fact about {topic}") | llm | StrOutputParser()),
)
chain.invoke({"topic": "cats"})    # → {"joke": ..., "fact": ...}
```

---

## Structured Output

```python
from pydantic import BaseModel, Field

class Person(BaseModel):
    name: str = Field(description="full name")
    age: int

structured_llm = llm.with_structured_output(Person)
structured_llm.invoke("John Smith is 42 years old")   # → Person(name=..., age=42)
```

Or parse from text with an output parser:

```python
from langchain_core.output_parsers import JsonOutputParser
parser = JsonOutputParser(pydantic_object=Person)
chain = prompt | llm | parser
```

---

## Retrieval-Augmented Generation (RAG)

```python
# 1. Load + split documents
from langchain_community.document_loaders import TextLoader, PyPDFLoader, WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter

docs = PyPDFLoader("doc.pdf").load()
chunks = RecursiveCharacterTextSplitter(
    chunk_size=1000, chunk_overlap=200).split_documents(docs)

# 2. Embed + store in a vector DB
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS   # or Chroma, Pinecone, PGVector

vectorstore = FAISS.from_documents(chunks, OpenAIEmbeddings())
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

# 3. RAG chain
from langchain_core.runnables import RunnablePassthrough
from langchain_core.output_parsers import StrOutputParser

rag_prompt = ChatPromptTemplate.from_template(
    "Answer using only this context:\n{context}\n\nQuestion: {question}")

def format_docs(docs): return "\n\n".join(d.page_content for d in docs)

rag = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt | llm | StrOutputParser()
)
rag.invoke("What does the document say about X?")
```

---

## Tools & Agents

```python
from langchain_core.tools import tool

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b

# bind tools so the model can request them
llm_with_tools = llm.bind_tools([multiply])
resp = llm_with_tools.invoke("what is 12 times 7?")
resp.tool_calls        # [{'name': 'multiply', 'args': {'a': 12, 'b': 7}, ...}]

# Agent loop (modern path uses LangGraph)
from langgraph.prebuilt import create_react_agent
agent = create_react_agent(llm, tools=[multiply])
agent.invoke({"messages": [("human", "what is 12 * 7?")]})
```

---

## Memory / Conversation History

```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

store = {}
def get_history(session_id):
    return store.setdefault(session_id, InMemoryChatMessageHistory())

chat = RunnableWithMessageHistory(chain, get_history,
        input_messages_key="input", history_messages_key="history")
chat.invoke({"input": "hi"}, config={"configurable": {"session_id": "abc"}})
```

---

## Key Concepts Recap

| Concept | What it is |
|---|---|
| **Runnable** | The uniform unit — everything supports `invoke/stream/batch` + async |
| **LCEL (`\|`)** | Pipe operator that composes Runnables into chains |
| **Prompt template** | Parameterised messages/text |
| **Output parser** | Turns model text into str / JSON / Pydantic objects |
| **Retriever / Vector store** | Semantic search backing RAG |
| **Tool** | A typed function the model can call |
| **Agent** | LLM loop that chooses tools until done (via LangGraph) |
| **Memory** | Persisted conversation history injected into prompts |

---

## Tips

- Set API keys via environment (`OPENAI_API_KEY`, `ANTHROPIC_API_KEY`); don't hard-code.
- Prefer `with_structured_output` over hand-parsing when you need typed results.
- Tune RAG quality with chunk size/overlap, `k`, and better embeddings before blaming the LLM.
- Use `.stream()` for responsive UIs; `.batch()` for throughput.
- For anything stateful or multi-step (agents, human-in-the-loop, branching), reach for **LangGraph** rather than legacy `AgentExecutor`.
- Add **LangSmith** (`LANGCHAIN_TRACING_V2=true`) for tracing/debugging chains.
