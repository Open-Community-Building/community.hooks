# Cognee Sprint at PyCon PyData 2025

This is a quick summary of what my installation looked like on our server.

Learn about Cognee here:

https://www.cognee.ai/

I was interested in this sprint topic because I like the idea of building a knowledge graph for documents
along the way.
The hope here is that by setting up a stage for a query, a knowledge graph can help selecting
the relevant documents for queries and respond accurately. 

During the sprint I could get it to work on on our GEX44 Hetzner servera for a minimal project.
I want to revisit my notes here to try it out on the Heidelberg city council documents.

Council Analytics:

https://github.com/Open-Community-Building/hackathon-council-analytics/

The sprint was managed by Hande Kafkas from Cognee. 
She quickly got us set up and answered any questions we had, so we could get things done.
I worked a bit with Gabriel Thiem to debug some errors, and got valuable input from Jan Müller who is from the area of Heidelberg and works for a big company there.

## Feedback for Cognee

My feedback to the Cognee project was that to get things running, it was necessary to also run

```
pip install transformers
```

so that Hugging Face works.

Another point I made was that sometimes you need to adapt the embedding dimension setting when you get this kind of error:

```
List should have at most 3072 items after validation, not 4096 [type=too_long, 
input_value=[6.078067302703857, 4.588..., -0.043787240982055664], input_type=list]
    For further information visit https://errors.pydantic.dev/2.10/v/too_long
```

This can be easily fixed in the .env:

EMBEDDING_DIMENSIONS=4096

So let's dive in to see how far I came:

## Installation

I logged in to our server with user maik, and and cloned the repository in my home folder.

```shell
ssh maik@altstadt.openheidelberg.de
ssh-add .ssh/gh/id_rsa
git clone git@github.com:topoteretes/cognee.git
mkdir venv
cd venv/
virtualenv .
  creator CPython3Posix(dest=/home/maik/venv, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, via=copy, app_data_dir=/home/maik/.local/share/virtualenv)
    added seed packages: pip==24.0
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
source bin/activate
cd
cd cognee
pip install .
```

## Using the local LLM server Ollama

Here are the tutorials I looked at while doing the installation:

https://ollama.com/download/linux
https://docs.cognee.ai/tutorials/setup-ollama

## Ollama installation

Installing ollama:

```shell
curl -fsSL https://ollama.com/install.sh | sh
>>> Cleaning up old version at /usr/local/lib/ollama
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle
######################################################################## 100.0%
>>> Adding ollama user to render group...
>>> Adding ollama user to video group...
>>> Adding current user to ollama group...
>>> Creating ollama systemd service...
>>> Enabling and starting ollama service...
>>> NVIDIA GPU installed.
```

I started with phi4:

```shell
ollama pull phi4
```

This is the fill list now:

```
ollama list
NAME                        ID              SIZE      MODIFIED      
phi4:latest                 ac896e5b8b34    9.1 GB    4 minutes ago    
granite3.2-vision:latest    3be41a661804    2.4 GB    14 hours ago     
llama3.2:latest             a80c4f17acd5    2.0 GB    15 hours ago     
mxbai-embed-large:latest    468836162de7    669 MB    15 hours ago     
```

Runnung it:

```
ollama run phi4:latest
>>> 
Use Ctrl + d or /bye to exit.
```
Check that it runs:

```
ollama ps
NAME           ID              SIZE     PROCESSOR    UNTIL              
phi4:latest    ac896e5b8b34    12 GB    100% GPU     4 minutes from now    
```

## Install phi4 and mistral

Try it:

```
curl http://localhost:11434/api/embeddings -d '{
    "model": "phi4:latest",
    "prompt": "Tell me about NLP"
}'
```

Install mistral:

```
ollama run avr/sfr-embedding-mistral
```

Check that it works

```
curl http://localhost:11434/api/embeddings -d '{
    "model": "avr/sfr-embedding-mistral:latest",
    "prompt": "Tell me about NLP"
}'
```

## Creat a Python script

I added a simple_test.py file:

``` 
import asyncio
import cognee
from cognee.shared.logging_utils import get_logger, ERROR
from cognee.api.v1.search import SearchType
 
# Prerequisites:
# 1. Copy `.env.template` and rename it to `.env`.
# 2. Add your OpenAI API key to the `.env` file in the `LLM_API_KEY` field:
#    LLM_API_KEY = "your_key_here"
 
 
async def main():
    # Create a clean slate for cognee -- reset data and system state
    await cognee.prune.prune_data()
    await cognee.prune.prune_system(metadata=True)
 
    # cognee knowledge graph will be created based on this text
    text = """
    Natural language processing (NLP) is an interdisciplinary
    subfield of computer science and information retrieval.
    """
 
    # Add the text, and make it available for cognify
    await cognee.add(text)
 
    # Run cognify and build the knowledge graph using the added text
    await cognee.cognify()
 
    # Query cognee for insights on the added text
    query_text = "Tell me about NLP"
    search_results = await cognee.search(query_type=SearchType.INSIGHTS, query_text=query_text)
    for result_text in search_results:
        print(result_text)
 
 
if __name__ == "__main__":
    logger = get_logger(level=ERROR)
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    try:
        loop.run_until_complete(main())
    finally:
        loop.run_until_complete(loop.shutdown_asyncgens())
```

## Environment variables

Use these environment variables in your .env file:

```
ENV="local"
TOKENIZERS_PARALLELISM="false"

# Default User Configuration
DEFAULT_USER_EMAIL=""
DEFAULT_USER_PASSWORD=""

# LLM Configuration
LLM_API_KEY = "ollama"
LLM_MODEL = "phi4:latest"
LLM_PROVIDER = "ollama"
LLM_ENDPOINT = "http://localhost:11434/v1"
LLM_API_VERSION=""
LLM_MAX_TOKENS="16384"

GRAPHISTRY_USERNAME=
GRAPHISTRY_PASSWORD=

SENTRY_REPORTING_URL=

# Embedding Configuration
EMBEDDING_PROVIDER = "ollama"
EMBEDDING_MODEL = "avr/sfr-embedding-mistral:latest"
EMBEDDING_ENDPOINT = "http://localhost:11434/api/embeddings"
EMBEDDING_DIMENSIONS = 4096
EMBEDDING_API_KEY=""
EMBEDDING_API_VERSION=""
EMBEDDING_DIMENSIONS=4096
EMBEDDING_MAX_TOKENS=8191
HUGGINGFACE_TOKENIZER = "Salesforce/SFR-Embedding-Mistral"

# "neo4j", "networkx" or "kuzu"
GRAPH_DATABASE_PROVIDER="networkx"
# Only needed if using neo4j
GRAPH_DATABASE_URL=
GRAPH_DATABASE_USERNAME=
GRAPH_DATABASE_PASSWORD=

# "qdrant", "pgvector", "weaviate", "milvus", "lancedb" or "chromadb"
VECTOR_DB_PROVIDER="lancedb"
# Not needed if using "lancedb" or "pgvector"
VECTOR_DB_URL=
VECTOR_DB_KEY=

# Relational Database provider "sqlite" or "postgres"
DB_PROVIDER="sqlite"

# Database name
DB_NAME=cognee_db

# Postgres specific parameters (Only if Postgres or PGVector is used). Do not use for cognee default simplest setup of SQLite-NetworkX-LanceDB
# DB_HOST=127.0.0.1
# DB_PORT=5432
# DB_USERNAME=cognee
# DB_PASSWORD=cognee


# Params for migrating relational database data to graph / Cognee ( PostgreSQL and SQLite supported )
# MIGRATION_DB_PATH="/path/to/migration/directory"
# MIGRATION_DB_NAME="migration_database.sqlite"
# MIGRATION_DB_PROVIDER="sqlite"
# Postgres specific parameters for migration
# MIGRATION_DB_USERNAME=cognee
# MIGRATION_DB_PASSWORD=cognee
# MIGRATION_DB_HOST="127.0.0.1"
# MIGRATION_DB_PORT=5432

# LITELLM Logging Level. Set to quiten down logging
LITELLM_LOG="ERROR"
```

## Run the script

Running the script gives:

```
python3 simple_test.py 

HTTP Request: GET https://raw.githubusercontent.com/BerriAI/litellm/main/model_prices_and_context_window.json "HTTP/1.1 200 OK"

2025-04-22T15:10:00.368480 [warning  ] Ontology file 'None' not found. Using fallback ontology at http://example.org/empty_ontology [OntologyAdapter]
2025-04-22T15:10:00.368709 [info     ] Lookup built: 0 classes, 0 individuals [OntologyAdapter]
2025-04-22T15:10:00.402547 [info     ] Graph deleted successfully.    [cognee.shared.logging_utils]None of PyTorch, TensorFlow >= 2.0, or Flax have been found. Models won't be available and only tokenizers, configuration and file/data utilities can be used.
2025-04-22T15:10:00.964682 [info     ] Database deleted successfully. [cognee.shared.logging_utils]
2025-04-22T15:10:00.992812 [info     ] Model not found in LiteLLM's model_cost. [cognee.shared.logging_utils]
HTTP Request: POST http://localhost:11434/v1/chat/completions "HTTP/1.1 200 OK"
EmbeddingRateLimiter initialized: enabled=False, requests_limit=60, interval_seconds=60User 4a183d79-b6d3-4c29-a8f7-dbbfb688a663 has registered.

2025-04-22T15:10:01.631930 [info     ] Pipeline run started: `4b84e400-23fc-5976-bbb4-f8ee303eed81` [run_tasks(tasks: [Task], data)]
2025-04-22T15:10:01.791205 [info     ] Coroutine task started: `resolve_data_directories` [run_tasks_base]
2025-04-22T15:10:01.957053 [info     ] Coroutine task started: `ingest_data` [run_tasks_base]/home/maik/venv/lib/python3.12/site-packages/dlt/destinations/impl/sqlalchemy/merge_job.py:194: SAWarning: Table 'file_metadata' already exists within the given MetaData - not copying.
  staging_table_obj = table_obj.to_metadata(
/home/maik/venv/lib/python3.12/site-packages/dlt/destinations/impl/sqlalchemy/merge_job.py:229: SAWarning: implicitly coercing SELECT object to scalar subquery; please use the .scalar_subquery() method to produce a scalar subquery.
  order_by=order_dir_func(order_by_col),

2025-04-22T15:10:02.305887 [info     ] Coroutine task completed: `ingest_data` [run_tasks_base]
2025-04-22T15:10:02.452267 [info     ] Coroutine task completed: `resolve_data_directories` [run_tasks_base]
2025-04-22T15:10:02.635528 [info     ] Pipeline run completed: `4b84e400-23fc-5976-bbb4-f8ee303eed81` [run_tasks(tasks: [Task], data)]
2025-04-22T15:10:02.796872 [info     ] Model not found in LiteLLM's model_cost. [cognee.shared.logging_utils]
2025-04-22T15:10:02.816312 [warning  ] Ontology file 'None' not found. Using fallback ontology at http://example.org/empty_ontology [OntologyAdapter]
2025-04-22T15:10:02.816438 [info     ] Lookup built: 0 classes, 0 individuals [OntologyAdapter]
2025-04-22T15:10:02.824925 [info     ] Pipeline run started: `5a8d7dd2-fd20-53c3-a8f7-45edf0f84677` [run_tasks(tasks: [Task], data)]
2025-04-22T15:10:02.971563 [info     ] Coroutine task started: `classify_documents` [run_tasks_base]
2025-04-22T15:10:03.117116 [info     ] Coroutine task started: `check_permissions_on_documents` [run_tasks_base]
2025-04-22T15:10:03.284699 [info     ] Async Generator task started: `extract_chunks_from_documents` [run_tasks_base]
2025-04-22T15:10:03.442357 [info     ] Coroutine task started: `extract_graph_from_data` [run_tasks_base]
2025-04-22T15:10:03.591203 [info     ] Model not found in LiteLLM's model_cost. [cognee.shared.logging_utils]
HTTP Request: POST http://localhost:11434/v1/chat/completions "HTTP/1.1 200 OK"
2025-04-22T15:10:14.611020 [warning  ] File /home/maik/venv/lib/python3.12/site-packages/cognee/.cognee_system/databases/cognee_graph.pkl not found. Initializing an empty graph. [cognee.shared.logging_utils]
2025-04-22T15:10:14.611933 [info     ] No close match found for 'area of study' in category 'classes' [OntologyAdapter]
2025-04-22T15:10:14.612088 [info     ] No close match found for 'nlp' in category 'individuals' [OntologyAdapter]
2025-04-22T15:10:14.612187 [info     ] No close match found for 'field' in category 'classes' [OntologyAdapter]
2025-04-22T15:10:14.612262 [info     ] No close match found for 'computer science' in category 'individuals' [OntologyAdapter]
2025-04-22T15:10:14.612333 [info     ] No close match found for 'subfield' in category 'classes' [OntologyAdapter]
2025-04-22T15:10:14.612397 [info     ] No close match found for 'information retrieval' in category 'individuals' [OntologyAdapter]
2025-04-22T15:10:14.918902 [info     ] Coroutine task started: `summarize_text` [run_tasks_base]
2025-04-22T15:10:15.078329 [info     ] Model not found in LiteLLM's model_cost. [cognee.shared.logging_utils]
HTTP Request: POST http://localhost:11434/v1/chat/completions "HTTP/1.1 200 OK"
2025-04-22T15:10:16.666512 [info     ] Coroutine task started: `add_data_points` [run_tasks_base]
2025-04-22T15:10:17.182823 [info     ] Coroutine task completed: `add_data_points` [run_tasks_base]
2025-04-22T15:10:17.444762 [info     ] Coroutine task completed: `summarize_text` [run_tasks_base]
2025-04-22T15:10:17.598445 [info     ] Coroutine task completed: `extract_graph_from_data` [run_tasks_base]
2025-04-22T15:10:17.744878 [info     ] Async Generator task completed: `extract_chunks_from_documents` [run_tasks_base]
2025-04-22T15:10:17.891488 [info     ] Coroutine task completed: `check_permissions_on_documents` [run_tasks_base]
2025-04-22T15:10:18.044973 [info     ] Coroutine task completed: `classify_documents` [run_tasks_base]
2025-04-22T15:10:18.196892 [info     ] Pipeline run completed: `5a8d7dd2-fd20-53c3-a8f7-45edf0f84677` [run_tasks(tasks: [Task], data)]({'created_at': 1745327403432, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 3, 432000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['text']}, 'type': 'DocumentChunk', 'belongs_to_set': None, 'text': '\n    Natural language processing (NLP) is an interdisciplinary\n    subfield of computer science and information retrieval.\n    ', 'chunk_size': 52, 'chunk_index': 0, 'cut_type': 'paragraph_end', 'id': UUID('db036125-214d-534a-a647-78497b269f0e')}, {'source_node_id': UUID('db036125-214d-534a-a647-78497b269f0e'), 'target_node_id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447'), 'relationship_name': 'contains', 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 17, 69140, tzinfo=datetime.timezone.utc)}, {'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'Entity', 'belongs_to_set': None, 'name': 'information retrieval', 'description': 'A branch of computer science concerning the retrieval of information from sources relevant to an informational need such as a query.', 'id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447')})
({'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'Entity', 'belongs_to_set': None, 'name': 'nlp', 'description': 'Interdisciplinary subfield primarily linking computer science and information retrieval.', 'id': UUID('bc338a39-64d6-549a-acec-da60846dd90d')}, {'relationship_name': 'connected_with', 'source_node_id': UUID('bc338a39-64d6-549a-acec-da60846dd90d'), 'target_node_id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447'), 'ontology_valid': False, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 916910, tzinfo=datetime.timezone.utc)}, {'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'Entity', 'belongs_to_set': None, 'name': 'information retrieval', 'description': 'A branch of computer science concerning the retrieval of information from sources relevant to an informational need such as a query.', 'id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447')})
({'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'Entity', 'belongs_to_set': None, 'name': 'information retrieval', 'description': 'A branch of computer science concerning the retrieval of information from sources relevant to an informational need such as a query.', 'id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447')}, {'source_node_id': UUID('02bdab9a-0981-518c-a0d4-1684e0329447'), 'target_node_id': UUID('d46662b0-93ea-5811-b78d-800c3265f97c'), 'relationship_name': 'is_a', 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 17, 69142, tzinfo=datetime.timezone.utc)}, {'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'EntityType', 'belongs_to_set': None, 'name': 'subfield', 'description': 'subfield', 'id': UUID('d46662b0-93ea-5811-b78d-800c3265f97c')})
({'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'Entity', 'belongs_to_set': None, 'name': 'nlp', 'description': 'Interdisciplinary subfield primarily linking computer science and information retrieval.', 'id': UUID('bc338a39-64d6-549a-acec-da60846dd90d')}, {'source_node_id': UUID('bc338a39-64d6-549a-acec-da60846dd90d'), 'target_node_id': UUID('66443a9d-b00a-5632-bdf0-72758f4f3b53'), 'relationship_name': 'is_a', 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 17, 69147, tzinfo=datetime.timezone.utc)}, {'created_at': 1745327414612, 'updated_at': datetime.datetime(2025, 4, 22, 13, 10, 14, 612000, tzinfo=datetime.timezone.utc), 'ontology_valid': False, 'version': 1, 'topological_rank': 0, 'metadata': {'index_fields': ['name']}, 'type': 'EntityType', 'belongs_to_set': None, 'name': 'area of study', 'description': 'area of study', 'id': UUID('66443a9d-b00a-5632-bdf0-72758f4f3b53')})
```

That worked nicely, but how about going bigger and using more recent tools.

Also, we were told that llama 3.2 is too weak to generate graph. They recomment llama3.3 in the documentation as well as deepseek-r1:32b:

```
DISCLAIMER: Models below 32b parameters are unable to create the proper graph structure sometimes so we suggest to use models like ‘deepseek-r1:32b’ or ‘llama3.3’.
```

I followed the advice of Jan Müller to use Gemma3. He told me that Google worked closely with Ollama to get Gemma3 working smoothly, and this is my setup:

```
ollama run gemma3:27b-it-qat
ollama pull mxbai-embed-large
```

This is the .env, I used:

```
ENV="local"
TOKENIZERS_PARALLELISM="false"

# Default User Configuration
DEFAULT_USER_EMAIL=""
DEFAULT_USER_PASSWORD=""

# LLM Configuration
LLM_API_KEY = "ollama"
LLM_MODEL = "gemma3:27b-it-qat"
LLM_PROVIDER = "ollama"
LLM_ENDPOINT = "http://localhost:11434/v1"
LLM_API_VERSION=""
LLM_MAX_TOKENS="16384"

GRAPHISTRY_USERNAME=
GRAPHISTRY_PASSWORD=

SENTRY_REPORTING_URL=

# Embedding Configuration
EMBEDDING_PROVIDER = "ollama"
EMBEDDING_MODEL = "mxbai-embed-large"
EMBEDDING_ENDPOINT = "http://localhost:11434/api/embeddings"
EMBEDDING_API_KEY=""
EMBEDDING_API_VERSION=""
EMBEDDING_DIMENSIONS=1024
EMBEDDING_MAX_TOKENS=8191
HUGGINGFACE_TOKENIZER = "mixedbread-ai/mxbai-embed-large-v1"

# "neo4j", "networkx" or "kuzu"
GRAPH_DATABASE_PROVIDER="networkx"
# Only needed if using neo4j
GRAPH_DATABASE_URL=
GRAPH_DATABASE_USERNAME=
GRAPH_DATABASE_PASSWORD=

# "qdrant", "pgvector", "weaviate", "milvus", "lancedb" or "chromadb"
VECTOR_DB_PROVIDER="lancedb"
# Not needed if using "lancedb" or "pgvector"
VECTOR_DB_URL=
VECTOR_DB_KEY=

# Relational Database provider "sqlite" or "postgres"
DB_PROVIDER="sqlite"

# Database name
DB_NAME=cognee_db

# Postgres specific parameters (Only if Postgres or PGVector is used). Do not use for cognee default simplest setup of SQLite-NetworkX-LanceDB
# DB_HOST=127.0.0.1
# DB_PORT=5432
# DB_USERNAME=cognee
# DB_PASSWORD=cognee


# Params for migrating relational database data to graph / Cognee ( PostgreSQL and SQLite supported )
# MIGRATION_DB_PATH="/path/to/migration/directory"
# MIGRATION_DB_NAME="migration_database.sqlite"
# MIGRATION_DB_PROVIDER="sqlite"
# Postgres specific parameters for migration
# MIGRATION_DB_USERNAME=cognee
# MIGRATION_DB_PASSWORD=cognee
# MIGRATION_DB_HOST="127.0.0.1"
# MIGRATION_DB_PORT=5432

# LITELLM Logging Level. Set to quiten down logging
LITELLM_LOG="ERROR"
```

That's it for the moment. I hope this documentation can help me get started on some real data.
