# Context-Aware Note-Taking System README

## Overview

This system is designed to manage and maintain conversation context for a user interacting with an AI model. It handles short-term in-memory context, long-term summarized context, and specific non-summarized core memories. Additionally, the system allows for dynamic updates to the model and user identity during execution.

## Features

- **Short-Term Context Management**: Stores the immediate conversation history in-memory.
- **Long-Term Memory Summarization**: Summarizes conversations and stores them in a JSON file when the short-term context reaches a certain token limit.
- **Core Memory Storage**: Allows specific facts to be stored without summarization for future retrieval.
- **Model and User Identity**: Stores and allows updates to the identity and traits of both the AI model and the user.

## Commands

### Memory Commands

- `REMEMBERTHIS!:` - Adds the text following this command to the core memory.
- `REMEMBERTHAT!` - Adds the text that came immediately before this command to the core memory.
- `CANYOUREMEMBER!:` - Searches the core memory for text matching the query provided after this command.

### Identity Commands

- `SOMETHINGABOUTME!:` - Updates the user identity with information provided after this command.
- `UPDATEYOURIDENTITY!:` - Updates the model identity with attributes and values provided after this command in the format `UPDATEYOURIDENTITY!:[attribute]:[value]`.

### Initial Startup Context Informer

- This function is called at the script startup to establish the initial context by pulling recent entries from the long-term and core memories, as well as loading the model and user identities.

## Functions

- `load_core_memory()`: Loads the core memory from a JSON file.
- `add_to_core_memory(fact)`: Adds a fact to the core memory.
- `search_core_memory(query)`: Searches the core memory for a specific query.
- `load_long_term_memory()`: Loads the long-term memory from a JSON file.
- `add_to_long_term_memory(summary)`: Adds a summary to the long-term memory.
- `get_long_term_memory_by_id(unique_id)`: Retrieves a long-term memory summary by its unique identifier.
- `update_conversation_history(new_input)`: Updates the conversation history with new input.
- `process_user_input(user_input)`: Processes user input and checks for memory and identity commands.
- `load_model_identity()`: Loads the model identity.
- `update_model_identity(attribute, value)`: Updates the model identity with new attributes or values.
- `handle_identity_input(user_input)`: Processes input related to identity updates.
- `initial_startup_context_informer()`: Establishes the initial startup context.

## Setup and Usage

To set up the system, ensure that Python 3.x is installed and that the necessary JSON files for storing memories and identities are in place. To interact with the system, run the main script and use the commands as described above.

## Additional Information

This README is an evolving document and will be updated as new features are added or existing ones are enhanced. For any queries or contributions, please contact the maintainer of this system.


context-aware-system/
│
├── main.py                                   # Main script to run the context-aware system
├── README.md                                 # The project documentation
│  
│
├── histories/
│     └── <custom history folder>/            # Created when you create the vector store
│       └── ├── core_memory.json              # Stores specific, non-summarized facts (core memory)
│           ├── long_term_memory.json         # Stores summarized conversations (long-term memory)
│           ├── DeepLake Archival memory      # Vector storage for knowledge base (#TODO implement file uploading to database script ingest.py)
│ 
│ 
├── identities/
      ├── model_identity.json                 # Stores model identity attributes                          
      └── user_identity.json                  # Stores user identity attributes  
     # Manages conversation history and context



#### FLOW OF THE APPLICATION/CONVERSATION FLOW

1. Script initiation. Variables loaded. Previous context (last 2 conversations in longterm memory, and corresponding core memories, and initial prompts. See below initial_startup_context_informer for complete details*) sent to model and established.
2. Model responds with a recap of the previous context and is ready to engage with the user.
3. User sends prompt.
4. Previous user prompt(in this case is the initial context text, previous model output, and new user prompt are sent to the model.)
5. Model responds to single prompt which is now the concatenated (initial context+previous model output+new user prompt). 
6. We now have new model output. User decides to respond to that output. Now previous user prompt+previous model output+new user prompt are concatenated and sent to the model. 
7. Model responds with new output, etc. 

Now , there are quite a few variables that could happen instead of the above. The above illustrates a regular, normal conversation flow between the model and the user. In addition to the above regular flow, the following back-end processes should always be happening:
    1. Long Term Memory function - This logic states that after every 500 tokens PAST the models context window (which should be set at 2000 tokens as default) between the user and model, a summary is generated and stored in the long term memory file as a memory entry. That means that at 2500 tokens the first summary should be generated, and then every 500 tokens after that. These memories are stored in a local file called long-term-memory.json
    2. Identity Function - this logic is used as an initial establishment of the models character and the user's character. Also acts as an instruction for the model to behave in a certain way. #TODO different identities and profiles can be loaded upon startup for different personalities/personas/model characters.




**initial_startup_context_informer - Upon initial startup and execution, the following processes should take place:
      1. Initialize program
      2. Initial context informer sends the following data to the model as a "prompt" - Initial prompt, 2 most recently created long-term memories from the long term memory file, and any correlating core memories(determined by a match of unique id numbers in the metadata, model_identity retrieved from model_identity.json and user_identity from user_identity.json). All of this data is sent to the model as an initial context establishment to set the conversation and context between the user and the model.
