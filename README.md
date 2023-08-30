# privateGPT
Interact privately with your documents on Google Colab using the power of GPT, 100% privately, no data leaks - Modified for Google Colab /Cloud Notebooks

I modified this script when I ran into memory limitations on my personal machine, I realised other folks might face this same issue. This repo will guide you on how you can re-create a private LLM using the power of GPT.

Inspired by the works of [imartinez](https://github.com/imartinez/privateGPT)

In the original version, you could ask questions to your documents without an internet connection, using the power of LLMs. 100% private, no data leaves your execution environment at any point. You can ingest documents and ask questions without an internet connection! But this version uses Google Colab.

Built with [LangChain](https://github.com/hwchase17/langchain), [GPT4All](https://github.com/nomic-ai/gpt4all), [LlamaCpp](https://github.com/ggerganov/llama.cpp), [Chroma](https://www.trychroma.com/) and [SentenceTransformers](https://www.sbert.net/).

![image](https://github.com/Tolulade-A/privateGPT/assets/22460844/440ea73d-7723-4cbb-8b80-a459ebe233ca)


# Environment Setup
In order to set your environment up to run the code here, first install all requirements:

```shell
!pip install -r /content/privateGPT/requirements.txt
```

Then, download the LLM model and place it in a directory of your choice (In your google colab temp space- See my notebook for details):
- LLM: default to [ggml-gpt4all-j-v1.3-groovy.bin](https://gpt4all.io/models/ggml-gpt4all-j-v1.3-groovy.bin). If you prefer a different GPT4All-J compatible model, just download it and reference it in your `.env` file.

**Please note that the .env will be hidden in your Google Colab after creating it. If you clone this repo into your personal computer, you'd be able to see it though.**

Copy the `example.env` template into `.env`
*First create the file, after creating it move it into the main folder of the project in Google Colab, in my case privateGPT.*

```shell
!touch env.txt #rename to .env

# Rename the file to .env

import os
os.rename('/content/privateGPT/env.txt', '.env')
```
and edit the variables appropriately in the `.env` file.
```
MODEL_TYPE: supports LlamaCpp or GPT4All
PERSIST_DIRECTORY: is the folder you want your vectorstore in
MODEL_PATH: Path to your GPT4All or LlamaCpp supported LLM
MODEL_N_CTX: Maximum token limit for the LLM model
MODEL_N_BATCH: Number of tokens in the prompt that are fed into the model at a time. Optimal value differs a lot depending on the model (8 works well for GPT4All, and 1024 is better for LlamaCpp)
EMBEDDINGS_MODEL_NAME: SentenceTransformers embeddings model name (see https://www.sbert.net/docs/pretrained_models.html)
TARGET_SOURCE_CHUNKS: The amount of chunks (sources) that will be used to answer a question
```

Note: because of the way `langchain` loads the `SentenceTransformers` embeddings, the first time you run the script it will require internet connection to download the embeddings model itself.

## Test dataset
This repo uses a [state of the union transcript](https://github.com/imartinez/privateGPT/blob/main/source_documents/state_of_the_union.txt) as an example.

## Instructions for ingesting your own dataset

Put any and all your files into the `source_documents` directory

The supported extensions are:

   - `.csv`: CSV,
   - `.docx`: Word Document,
   - `.doc`: Word Document,
   - `.enex`: EverNote,
   - `.eml`: Email,
   - `.epub`: EPub,
   - `.html`: HTML File,
   - `.md`: Markdown,
   - `.msg`: Outlook Message,
   - `.odt`: Open Document Text,
   - `.pdf`: Portable Document Format (PDF),
   - `.pptx` : PowerPoint Document,
   - `.ppt` : PowerPoint Document,
   - `.txt`: Text file (UTF-8),

Run the following command to ingest all the data.

```shell
!python /content/privateGPT/ingest.py
```

Output should look like this:

```shell
Downloading (…)e9125/.gitattributes: 100% 1.18k/1.18k [00:00<00:00, 3.88MB/s]
Downloading (…)_Pooling/config.json: 100% 190/190 [00:00<00:00, 953kB/s]
Downloading (…)7e55de9125/README.md: 100% 10.6k/10.6k [00:00<00:00, 27.9MB/s]
Downloading (…)55de9125/config.json: 100% 612/612 [00:00<00:00, 2.18MB/s]
Downloading (…)ce_transformers.json: 100% 116/116 [00:00<00:00, 439kB/s]
Downloading (…)125/data_config.json: 100% 39.3k/39.3k [00:00<00:00, 9.70MB/s]
Downloading pytorch_model.bin: 100% 90.9M/90.9M [00:00<00:00, 92.2MB/s]
Downloading (…)nce_bert_config.json: 100% 53.0/53.0 [00:00<00:00, 216kB/s]
Downloading (…)cial_tokens_map.json: 100% 112/112 [00:00<00:00, 341kB/s]
Downloading (…)e9125/tokenizer.json: 100% 466k/466k [00:00<00:00, 7.92MB/s]
Downloading (…)okenizer_config.json: 100% 350/350 [00:00<00:00, 1.34MB/s]
Downloading (…)9125/train_script.py: 100% 13.2k/13.2k [00:00<00:00, 35.7MB/s]
Downloading (…)7e55de9125/vocab.txt: 100% 232k/232k [00:00<00:00, 8.82MB/s]
Downloading (…)5de9125/modules.json: 100% 349/349 [00:00<00:00, 1.22MB/s]
Creating new vectorstore
Loading documents from /content/privateGPT/source_documents
Loading new documents: 100%|█████████████████████| 1/1 [00:00<00:00, 135.57it/s]
Loaded 1 new documents from /content/privateGPT/source_documents
Split into 91 chunks of text (max. 500 tokens each)
Creating embeddings. May take some minutes...
Ingestion complete! You can now run privateGPT.py to query your documents
```

It will create a `db` folder containing the local vectorstore. Will take 20-30 seconds per document, depending on the size of the document.
You can ingest as many documents as you want, and all will be accumulated in the local embeddings database.
If you want to start from an empty database, delete the `db` folder.

Note: during the ingest process no data leaves your local environment. You could ingest without an internet connection, except for the first time you run the ingest script, when the embeddings model is downloaded.

## Ask questions to your documents, locally!
In order to ask a question, run a command like:

```shell
!python /content/privateGPT/privateGPT.py
```

And wait for the script to require your input.

```plaintext
> Enter a query:
```

Hit enter. You'll need to wait for seconds-minutes (mine took 2670.53 s) while the LLM model consumes the prompt and prepares the answer. Once done, it will print the answer and the 4 sources it used as context from your documents; you can then ask another question without re-running the script, just wait for the prompt again.

Note: you could turn off your internet connection, and the script inference would still work. No data gets out of your local environment.

Type `exit` to finish the script or simply interrupt in the case of Google Colab.


### CLI
The script also supports optional command-line arguments to modify its behavior. You can see a full list of these arguments by running the command ```python privateGPT.py --help``` in your terminal.


# How does it work?
Selecting the right local models and the power of `LangChain` you can run the entire pipeline locally, without any data leaving your environment, and with reasonable performance.

- `ingest.py` uses `LangChain` tools to parse the document and create embeddings locally using `HuggingFaceEmbeddings` (`SentenceTransformers`). It then stores the result in a local vector database using `Chroma` vector store.
- `privateGPT.py` uses a local LLM based on `GPT4All-J` or `LlamaCpp` to understand questions and create answers. The context for the answers is extracted from the local vector store using a similarity search to locate the right piece of context from the docs.
- `GPT4All-J` wrapper was introduced in LangChain 0.0.162.

# System Requirements

## Python Version
To use this software, you must have Python 3.10 or later installed. Earlier versions of Python will not compile.


# Disclaimer
This is a test project to validate the feasibility of a fully private solution for question answering using LLMs and Vector embeddings. It is not production ready, and it is not meant to be used in production. The models selection is not optimized for performance, but for privacy; but it is possible to use different models and vectorstores to improve performance.
