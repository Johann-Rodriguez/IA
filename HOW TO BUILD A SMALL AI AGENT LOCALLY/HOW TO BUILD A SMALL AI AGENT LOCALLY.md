# HOW TO BUILD A SMALL AI AGENT LOCALLY

This is a personal project in which I will attemp to build an AI agent locally, using a physical server with Docker containers installed and measure its performance. 

## BASE TECHNICAL DEFINITIONS

### Server Hardware:

| PLATFORM  |  DELL PowerEdge R720 | 
|-----------|---------------------:|
| OS        | Ubuntu 24.04.4 LTS   | 
| CPU       | Intel Xeon E5-2690 - 2x 10-core/20 Threads   | 
| GPU       | N/A   | 
| MEMORY    | 768 GiB (DDR3)   | 
| HDD       | 893.75 GiB (HDD)   | 



### Ollama:

Ollama will serve as the “backend” for this project, hosting all the LLMs. I’ve found Ollama to be exceptionally easy to use and highly flexible, making it a great choice for development and experimentation. Plus, it’s open-source and free.

### Open Web-UI:

For the “frontend” or chat portion of this setup, I will be using Open WebUI, which is also open-source and free. Open WebUI is the platform for running AI on your own terms. Connect to any model; local or cloud.

### Training Strategies:
Because training AI models is expensive and there is no GPU (what with GPU is seconds, with CPU is minutes), RAG should be used, as it allows information to be added instantly, personality can be modified and update the information.

RAG (Retrieval-Augmented Generation) is an AI framework that optimizes Large Language Models (LLMs) by connecting them to external knowledge bases before generating a response.  Instead of relying solely on static training data, RAG allows LLMs to retrieve relevant information from authoritative sources, augment the user prompt with that context, and then generate a more accurate, up-to-date, and fact-grounded answer. 

To use RAG, we must use nomic-embed-text, it is an open-source text encoding (embedding) model, designed to generate semantic vector representations of documents and queries. It is mainly used for Semantic Search (RAG), clustering, text classification and information retrieval, working for both text and images (in their multimodal versions).


### INFRASTRUCTURE DEPLOYMENT: 

The easiest way to use and connect Ollama and Open Web-UI is using Docker, so after installing Docker, let’s create two containers; one for Ollama and the other for Open Web-UI and create a Docker virtual network (ollama-rag-net) to communicate Ollama with Open Web-UI.

Later, we downloaded the nomic-embed-text vector model in Ollama to take care of processing and fragmenting the text of the PDF, Docx, etc.

                docker exec -it ollama ollama pull nomic-embed-text

The Docker containers:

# ![](images/1.png)

They are all the AI models installed in Ollama. Gemma4:12b is the LLM model chosen for been lightweight and powerful.

# ![](images/2.png)

After all of this, on a web browser, let’s type the IP address of your physical server with port 3000, like this:

# ![](images/3.png)


The first user that you will create is the “admin” user, this will be the user to manage all the OPEN WEB-UI platform.

### VOICE PROCESSING:
Considering, there are different ways to process audio, but our goal is to guarantee privacy and local processing, so in this case we should use Whisper, it is a model 100% local on your server. 

Before using Whisper, let’s use the Browser Web API option, it performs speech recognition directly on the client's device (computer or mobile phone) using the browser's native speech engine. It requires no API keys and completely relieves the processing load on the server.

Since we use the HTTP protocol, most modern browsers don’t allow the use of the microphone by default because of security purposes. 

To solve this, On the client computer, you can tell the browser to specifically trust the IP address where the AI is installed.

Open your browser on the client’s computer.
In the address bar, type the following (depending on your browser):
Chrome: chrome://flags/#unsafely-treat-insecure-origin-as-secure
Edge: edge://flags/#unsafely-treat-insecure-origin-as-secure
Find the option called "Insecure origins treated as secure."
Change it from Disabled to Enabled.
In the text box that appears below, enter the exact IP address and port of your Open WebUI. For example: http://192.168.1.100:3000
Click the Relaunch button in the bottom right corner of the browser to apply the changes.

Done! When you reopen Open WebUI, the browser will allow you to grant microphone permissions and you will be able to use both the browser's Web API and your local Speeches server.


### TECHNICAL ERRORS WHILE DEPLOYING:
Troubleshooting Technical Errors (Step-by-Step)

While loading the PDFs and during our initial interactions with Gemma 4:12b, three critical configuration issues arose and were resolved:

***Error 1:*** Missing embedding model (No embedding model is loaded)

Cause: Open WebUI was attempting to download a default model from Hugging Face but failed due to network restrictions.

Solution: We configure the Embedding Engine to use Ollama and point the Embedding Model to our downloaded “nomic-embed-text:latest”.


***Error 2:*** Dimension conflict in the database (dimension of 384, got 768)

Cause: The interface's vector database had previously been initialized with a 384-dimension format, and when the new 768-dimension model from Ollama was added, the configurations conflicted.

Solution: We deleted the document causing the error and clicked “Reset Vector Database” in the administration settings to initialize a clean database with 768 dimensions.

***Error 3:*** Gemma was silent (did not return any response)

Cause: When requesting a summary, the text sent exceeded Ollama’s default limit (4,095 tokens), causing the message to be truncated (truncating input prompt), which removed the user’s question.

Solution: We edited the Gemma model in Open WebUI and expanded the context parameter “num_ctx” to 65,536 (64k). We avoided setting it to an excessively high value (such as 10 million) to prevent an out-of-memory (OOM) error.

