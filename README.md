<style>
* {
  box-sizing: border-box;
}

/* Create three equal columns that floats next to each other */
.column {
  float: left;
  width: 33.33%;
  padding: 10px;
  height: 300px; /* Should be removed. Only for demonstration */
}

/* Clear floats after the columns */
.row:after {
  content: "";
  display: table;
  clear: both;
}
</style>


## Making React Apps Smarter with AI-Powered Search
Beyond Keywords: Leveraging AI for Intelligent Retrieval
[React Meetup Talk Notes](https://github.com/kurtzace/diary-2025/tree/main/seminars/reactmeetup88)

---

## The Need for Smarter Search

*   Traditional keyword search has limitations.
*   It often fails to understand the **user's true intent** if exact keywords aren't used.
*   Users expect more intuitive ways to find information.
*   AI-powered search aims for **intent-based search**.
*   This allows systems to understand the meaning behind queries, even with different phrasing.

---

## Introducing RAG: Retrieval Augmented Generation

*   **RAG** provides Large Language Models (LLMs) with **specific information relevant to the prompt**.
*   This information can be outside the LLM's original training data, such as **proprietary data**.
*   **Why is RAG important?** LLMs are limited by their training data cutoff date.
*   RAG solves this by fetching relevant external information and passing it to the model as **context**.

---

## How RAG Works: Components & Process

*   **Process Overview:**
    1.  User query is received.
    2.  Query is embedded (converted to a vector representation of its meaning).
    3.  Relevant source material ("chunks") is retrieved from a knowledge base based on semantic similarity to the query embedding.
    4.  The retrieved chunks are passed to the LLM alongside the original query as context.
    5.  The LLM generates a response based on the query and the provided context.
--
*   **Key Components:**
    *   **Embeddings:** Vector representations of words and phrases capturing their semantic meaning.
    *   **Chunking:** Breaking down large documents into smaller, meaningful pieces before embedding.
    *   **Vector Databases:** Specialized databases designed to store and search embeddings efficiently (e.g., Postgres with pgvector, Milvus). Support semantic similarity search using metrics like cosine distance.
    *   **LLM (Language Model):** Generates the final response using the retrieved context.

---

## RAG in Action: The Vercel AI SDK Example

*   The **Vercel AI SDK** provides a unified interface for interacting with various LLMs and building AI applications, including RAG chatbots.
*   It makes it easier to incorporate generative AI and streaming interfaces in React apps.
*   The **`ai-sdk-rag-starter`** repository demonstrates building a RAG chatbot with Next.js.
*   This example uses Next.js 14, AI SDK, OpenAI, Drizzle ORM, and Postgres with pgvector.
*   The goal is a chatbot that **only responds with information from its knowledge base**.

---

## Vercel RAG Example: Code Structure

*   The project involves:
    *   Database schema for `resources` (source material) and `embeddings`.
    *   Server Actions (`lib/actions/resources.ts`) to manage data.
    *   Embedding logic (`lib/ai/embedding.ts`) using `embedMany` or `embed` functions from the AI SDK to generate embeddings and `findRelevantContent` to search the database.
    *   Frontend (`app/page.tsx`) using the `@ai-sdk/react` hooks like `useChat`.
    *   API Route handler (`app/api/chat/route.ts`) for processing chat requests and defining tools

---

## Vercel RAG Example: Tools & Retrieval

*   The AI SDK allows defining **tools** that the LLM can call.
*   Tools extend the LLM's capabilities, like searching a knowledge base.
*   The `getInformation` tool takes the user's `question` and uses `findRelevantContent` to query the vector database.
--
```typescript
import { z } from 'zod';
import { findRelevantContent } from '@/lib/ai/embedding';
export async function POST(req: Request) {
  const { messages } = await req.json();
  const result = streamText({ model: openai('gpt-4o'), 
    system: `Use your knowledge base...`, messages,
    tools: {
      getInformation: tool({
        description: `get information from your knowledge base to answer questions.`,
        parameters: z.object({ question: z.string().describe('the users question'),  }),
        execute: async ({ question }) => findRelevantContent(question),
      }),
      // ... other tools like addResource
    },
  });
  return result.toDataStreamResponse();
}
```

---

## Real-World Challenge: Internal Docs & Storybooks

*   Applying RAG concepts to internal, unstructured data presents challenges.
*   **Problem:** Internal company design library documentation (React Storybooks) wasn't easily accessible to AI tools like Copilot for coding completion.
*   Need an AI pipeline to access this specific knowledge.

---

## Our LLM Pipeline for Internal Docs

*   An LLM pipeline was created to read the React storybooks.
*   **Initial Setup:**
    *   An unstructured HTML parser to extract text content.
    *   Embeddings generation from the parsed content.
    *   Storing embeddings in a vector store (Chroma mentioned).
    *   Calling a coding LLM (from Hugging Face) for QA based on retrieved context.
*   This allows querying the Storybook content using natural language.

---

## The Parser Problem & Potential Solutions

*   **Challenge:** The unstructured HTML parser might be **feeding "garbage" to the LLM**.
*   Unstructured parsing may pull irrelevant or poorly formatted text, hindering embedding quality and retrieval accuracy.
*   **Potential Solution:**
    *   **Fine-tune the parser.**
    *   Consider using libraries like **Beautiful Soup** (external concept) to target and extract **specific components** and relevant documentation text.
    *   This ensures cleaner, more focused input for embeddings, leading to better retrieval.

---

## Adding More Knowledge: PDFs to S3 & Bedrock

*   Expanding the internal knowledge base for RAG.
*   Plan involves adding **PDF documents to S3**.
*   These documents can then be processed (chunked, embedded).
*   Using **Bedrock** (AWS service, external concept) for RAG processes with this new data source.
*   This extends the AI's knowledge to include information from internal documents.
--
<video controls width="720">
  <source src="assets/BedrockRagSteps.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
--
Lambda code - [refer](assets/lambda_function.txt)
![image](https://cdn-0.plantuml.com/plantuml/png/XLHTRzf047pthrZb2LII7u2KAaxQf6b8YWXINv3GuTiu9yvxw3uWgEf_xzfSp5aoz2doVipip4suSLvO7ogTUUKrGlxx5MvHMbEiOPO_m4HKAodWFb3XbI6ursQNPHlUKONQ2F9myVt3eoMXOMvLMQ9Tq02logiHvDD7U10UTs8_l03XFWUwwymOMNXnklVlXbgwPYrskrl9dGlAI-JQB91N5TQqpLfiDytOEuFnH6QdKPO8jy9X0z2Mk79kzTb8r32OJ9w7AN5Jph8e6Yw_HY5ZUAIjMICCS5lVNHZoGf6Y4XHjWbzw_F33B1DimLPu_DH_-9FYiFEMfv8rUBEcWWOtInPwh1Z3dT0QB7ghU7ufI2wvZPOP1mk20-Zr4NfK5_dMDrLU_tcwNJv-6q4ZSPdFNlukABcur_gU6f_MDNaCitWFMDIuhRsMfWWLxpYEnufd91W2h5Oe9SgEOorzz-4wNt1voWvc3fQEl2orSNHGl0TQkz7na64cyGJNAPLf9CRm47rcChSnv92b1TSD07nftqR6MnA9tOob7_yLM4Po2SshGg6yyCeGAqHVNB6sVf9-QKaQ8tvlebgb37ys9Jqfd_8GEfUQXJ7u9lXDw6eeWx3I2gNpmoz83-2ziPNmDLxG7eI3dw-_NexCF1d_PrjDF-RpASoE_8xfWhip4aUmCd9_NID4m3GTaJBBRf1agFG_)
--
#### How about N8N Workflow Bedrock and RAG
<video controls width="720">
  <source src="assets/n8n-bedrock-rag.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
---

## Future Vision: From QA to Coding Completion

*   The current pipeline uses the Hugging Face LLM for **QA-based responses**.
*   **Goal:** Fine-tune the process to use **coding completion** instead.
*   This would enable the AI to generate code snippets or component usage examples directly from the Storybook documentation [Hugging Face coding completion blog mentioned].
*   Moving from answering "What does this component do?" to "Show me how to use this component with these props."

---

## Credits & Resources

Presented by Karan Bhandari

Special thanks to **Ajay Lakshman** for insights on Making React Apps Smarter with AI-Powered Search - React Meetup #88.
--
**Resources:**
*   **Vercel AI SDK Documentation (Tool Calling, RAG Guides):**,
*   **LinkedIn Learning - LLM Foundations: Vector Databases for Caching and RAG:**
*   **React meetup #88 notes (kurtzace/diary-2025):**
*   **GitHub - thatbeautifuldream/ai-for-react-developers:**,
    *   Repository: [https://github.com/thatbeautifuldream/ai-for-react-developers](https://github.com/thatbeautifuldream/ai-for-react-developers) & Slides: [https://milindmishra.com/slide/ai-for-react-developers](https://milindmishra.com/slide/ai-for-react-developers)

