---
layout:     post
title:      LangChain 学习
subtitle:   
date:       2023-12-1
author:     sq
header-img: 
catalog: true
tags:
    - LangChain
---
## 例子
### Prompt + LLM
```javascript
import { PromptTemplate } from "langchain/prompts";
import { ChatOpenAI } from "langchain/chat_models/openai";

const model = new ChatOpenAI({});
const promptTemplate = PromptTemplate.fromTemplate(
  "Tell me a joke about {topic}"
);

const chain = promptTemplate.pipe(model);

const result = await chain.invoke({ topic: "bears" });

console.log(result);

/*
  AIMessage {
    content: "Why don't bears wear shoes?\n\nBecause they have bear feet!",
  }
*/
```

```javascript
import { PromptTemplate } from "langchain/prompts";
import { ChatOpenAI } from "langchain/chat_models/openai";

const prompt = PromptTemplate.fromTemplate(`Tell me a joke about {subject}`);

const model = new ChatOpenAI({});

const functionSchema = [
  {
    name: "joke",
    description: "A joke",
    parameters: {
      type: "object",
      properties: {
        setup: {
          type: "string",
          description: "The setup for the joke",
        },
        punchline: {
          type: "string",
          description: "The punchline for the joke",
        },
      },
      required: ["setup", "punchline"],
    },
  },
];

const chain = prompt.pipe(
  model.bind({
    functions: functionSchema,
    function_call: { name: "joke" },
  })
);

const result = await chain.invoke({ subject: "bears" });

console.log(result);

/*
  AIMessage {
    content: "",
    additional_kwargs: {
      function_call: {
        name: "joke",
        arguments: '{\n  "setup": "Why don\'t bears wear shoes?",\n  "punchline": "Because they have bear feet!"\n}'
      }
    }
  }
*/
```

```javascript
const model = new ChatOpenAI();
const content = 'Is there any errors in the following code? function () {console.log(1);;}';
const messages = [new HumanMessage({ content: content })];
chatModel.predictMessages(messages).then(result => {
});
```

```javascript
import { PromptTemplate } from "langchain/prompts";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { RunnableSequence } from "langchain/schema/runnable";
import { StringOutputParser } from "langchain/schema/output_parser";

const model = new ChatOpenAI({});
const promptTemplate = PromptTemplate.fromTemplate(
  "Tell me a joke about {topic}"
);
const outputParser = new StringOutputParser();

const chain = RunnableSequence.from([promptTemplate, model, outputParser]);

const result = await chain.invoke({ topic: "bears" });

console.log(result);

/*
  "Why don't bears wear shoes?\n\nBecause they have bear feet!"
*/
```

### Cookbook Multiple chains
`RunnableSequence` can be used to combine multiple Chains together.

The following RunnableSequence coerces the object into a RunnableMap. Each property in the map receives the same parameters.
The runnable (`chain` in this example) or function (which can get the input as an object in parameter) set as the value 
of that property is invoked with those parameters, and the return value populates an object which is then passed onto the next runnable in the sequence.

```javascript
import { PromptTemplate } from "langchain/prompts";
import { RunnableSequence } from "langchain/schema/runnable";
import { StringOutputParser } from "langchain/schema/output_parser";
import { ChatAnthropic } from "langchain/chat_models/anthropic";

const prompt1 = PromptTemplate.fromTemplate(
  `What is the city {person} is from? Only respond with the name of the city.`
);
const prompt2 = PromptTemplate.fromTemplate(
  `What country is the city {city} in? Respond in {language}.`
);

const model = new ChatAnthropic({});

const chain = prompt1.pipe(model).pipe(new StringOutputParser());

const combinedChain = RunnableSequence.from([
  {
    city: chain,
    language: (input) => input.language,
  },
  prompt2,
  model,
  new StringOutputParser(),
]);

const result = await combinedChain.invoke({
  person: "Obama",
  language: "German",
});

console.log(result);

/*
  Chicago befindet sich in den Vereinigten Staaten.
*/
```

### RAG

```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { HNSWLib } from "langchain/vectorstores/hnswlib";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { PromptTemplate } from "langchain/prompts";
import {
  RunnableSequence,
  RunnablePassthrough,
} from "langchain/schema/runnable";
import { StringOutputParser } from "langchain/schema/output_parser";
import { formatDocumentsAsString } from "langchain/util/document";

const model = new ChatOpenAI({});

// It creates a new Document instance for each text and metadata, then calls the fromDocuments method to create the 
// HNSWLib instance.
const vectorStore = await HNSWLib.fromTexts(
  // texts used to create document
  ["mitochondria is the powerhouse of the cell"],
  // metadatas used to create document
  [{ id: 1 }],
  // embeddings used by the HNSWLib instance
  new OpenAIEmbeddings()
);
// retriever Can perform similarity search or maximal marginal relevance search.
const retriever = vectorStore.asRetriever();

const prompt =
  PromptTemplate.fromTemplate(`Answer the question based only on the following context:
{context}

Question: {question}`);

const chain = RunnableSequence.from([
  {
    context: retriever.pipe(formatDocumentsAsString),
    // RunnablePassthrough pass the string that invoke received, so that invoke can receive a string rather than object
    question: new RunnablePassthrough(),
  },
  prompt,
  model,
  new StringOutputParser(),
]);

const result = await chain.invoke("What is the powerhouse of the cell?");

console.log(result);

/*
  "The powerhouse of the cell is the mitochondria."
*/
```

#### Conversational Retrieval Chain

```typescript
import { PromptTemplate } from "langchain/prompts";
import {
  RunnableSequence,
  RunnablePassthrough,
} from "langchain/schema/runnable";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { HNSWLib } from "langchain/vectorstores/hnswlib";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { StringOutputParser } from "langchain/schema/output_parser";
import { formatDocumentsAsString } from "langchain/util/document";

const model = new ChatOpenAI({});

const condenseQuestionTemplate = `Given the following conversation and a follow up question, rephrase the follow up question to be a standalone question, in its original language.

Chat History:
{chat_history}
Follow Up Input: {question}
Standalone question:`;
const CONDENSE_QUESTION_PROMPT = PromptTemplate.fromTemplate(
  condenseQuestionTemplate
);

const answerTemplate = `Answer the question based only on the following context:
{context}

Question: {question}
`;
const ANSWER_PROMPT = PromptTemplate.fromTemplate(answerTemplate);

const formatChatHistory = (chatHistory: [string, string][]) => {
  const formattedDialogueTurns = chatHistory.map(
    (dialogueTurn) => `Human: ${dialogueTurn[0]}\nAssistant: ${dialogueTurn[1]}`
  );
  return formattedDialogueTurns.join("\n");
};

const vectorStore = await HNSWLib.fromTexts(
  [
    "mitochondria is the powerhouse of the cell",
    "mitochondria is made of lipids",
  ],
  [{ id: 1 }, { id: 2 }],
  new OpenAIEmbeddings()
);
const retriever = vectorStore.asRetriever();

type ConversationalRetrievalQAChainInput = {
  question: string;
  chat_history: [string, string][];
};

const standaloneQuestionChain = RunnableSequence.from([
  {
    question: (input: ConversationalRetrievalQAChainInput) => input.question,
    chat_history: (input: ConversationalRetrievalQAChainInput) =>
      formatChatHistory(input.chat_history),
  },
  CONDENSE_QUESTION_PROMPT,
  model,
  new StringOutputParser(),
]);

const answerChain = RunnableSequence.from([
  {
    context: retriever.pipe(formatDocumentsAsString),
    question: new RunnablePassthrough(),
  },
  ANSWER_PROMPT,
  model,
]);

const conversationalRetrievalQAChain =
  standaloneQuestionChain.pipe(answerChain);

const result1 = await conversationalRetrievalQAChain.invoke({
  question: "What is the powerhouse of the cell?",
  chat_history: [],
});
console.log(result1);
/*
  AIMessage { content: "The powerhouse of the cell is the mitochondria." }
*/

const result2 = await conversationalRetrievalQAChain.invoke({
  question: "What are they made out of?",
  chat_history: [
    [
      "What is the powerhouse of the cell?",
      "The powerhouse of the cell is the mitochondria.",
    ],
  ],
});
console.log(result2);
/*
  AIMessage { content: "Mitochondria are made out of lipids." }
*/
```


### querying a SQL DB
```typescript
import { DataSource } from "typeorm";
import { SqlDatabase } from "langchain/sql_db";
import {
  RunnablePassthrough,
  RunnableSequence,
} from "langchain/schema/runnable";
import { PromptTemplate } from "langchain/prompts";
import { StringOutputParser } from "langchain/schema/output_parser";
import { ChatOpenAI } from "langchain/chat_models/openai";

const datasource = new DataSource({
  type: "sqlite",
  database: "Chinook.db",
});

const db = await SqlDatabase.fromDataSourceParams({
  appDataSource: datasource,
});

const prompt =
  PromptTemplate.fromTemplate(`Based on the table schema below, write a SQL query that would answer the user's question:
{schema}

Question: {question}
SQL Query:`);

const model = new ChatOpenAI();

// The `RunnablePassthrough.assign()` is used here to passthrough the input from the `.invoke()`
// call (in this example it's the question), along with any inputs passed to the `.assign()` method.
// In this case, we're passing the schema.
const sqlQueryGeneratorChain = RunnableSequence.from([
  RunnablePassthrough.assign({
    schema: async () => db.getTableInfo(),
  }),
  prompt,
  model.bind({ stop: ["\nSQLResult:"] }),
  new StringOutputParser(),
]);

const result = await sqlQueryGeneratorChain.invoke({
  question: "How many employees are there?",
});

console.log({
  result,
});

/*
  {
    result: "SELECT COUNT(EmployeeId) AS TotalEmployees FROM Employee"
  }
*/

const finalResponsePrompt =
  PromptTemplate.fromTemplate(`Based on the table schema below, question, sql query, and sql response, write a natural language response:
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}`);

const fullChain = RunnableSequence.from([
  RunnablePassthrough.assign({
    query: sqlQueryGeneratorChain,
  }),
  {
    schema: async () => db.getTableInfo(),
    question: (input) => input.question,
    query: (input) => input.query,
    response: (input) => db.run(input.query),
  },
  finalResponsePrompt,
  model,
]);

const finalResponse = await fullChain.invoke({
  question: "How many employees are there?",
});

console.log(finalResponse);

/*
  AIMessage {
    content: 'There are 8 employees.',
    additional_kwargs: { function_call: undefined }
  }
*/
```

### Adding memory

```typescript
import { ChatPromptTemplate, MessagesPlaceholder } from "langchain/prompts";
import { RunnableSequence } from "langchain/schema/runnable";
import { ChatAnthropic } from "langchain/chat_models/anthropic";
import { BufferMemory } from "langchain/memory";

const model = new ChatAnthropic();
const prompt = ChatPromptTemplate.fromMessages([
  ["system", "You are a helpful chatbot"],
  new MessagesPlaceholder("history"),
  ["human", "{input}"],
]);

// Default "inputKey", "outputKey", and "memoryKey values would work here
// but we specify them for clarity.
const memory = new BufferMemory({
  returnMessages: true,
  inputKey: "input",
  outputKey: "output",
  memoryKey: "history",
});

console.log(await memory.loadMemoryVariables({}));

/*
  { history: [] }
*/

const chain = RunnableSequence.from([
  {
    input: (initialInput) => initialInput.input,
    memory: () => memory.loadMemoryVariables({}),
  },
  {
    input: (previousOutput) => previousOutput.input,
    history: (previousOutput) => previousOutput.memory.history,
  },
  prompt,
  model,
]);

const inputs = {
  input: "Hey, I'm Bob!",
};

const response = await chain.invoke(inputs);

console.log(response);

/*
  AIMessage {
    content: " Hi Bob, nice to meet you! I'm Claude, an AI assistant created by Anthropic to be helpful, harmless, and honest.",
    additional_kwargs: {}
  }
*/

await memory.saveContext(inputs, {
  output: response.content,
});

console.log(await memory.loadMemoryVariables({}));

/*
  {
    history: [
      HumanMessage {
        content: "Hey, I'm Bob!",
        additional_kwargs: {}
      },
      AIMessage {
        content: " Hi Bob, nice to meet you! I'm Claude, an AI assistant created by Anthropic to be helpful, harmless, and honest.",
        additional_kwargs: {}
      }
    ]
  }
*/

const inputs2 = {
  input: "What's my name?",
};

const response2 = await chain.invoke(inputs2);

console.log(response2);

/*
  AIMessage {
    content: ' You told me your name is Bob.',
    additional_kwargs: {}
  }
*/
```

### Using tools
```typescript
import { SerpAPI } from "langchain/tools";
import { ChatAnthropic } from "langchain/chat_models/anthropic";
import { PromptTemplate } from "langchain/prompts";
import { StringOutputParser } from "langchain/schema/output_parser";

const search = new SerpAPI();

const prompt =
  PromptTemplate.fromTemplate(`Turn the following user input into a search query for a search engine:

{input}`);

const model = new ChatAnthropic({});

const chain = prompt.pipe(model).pipe(new StringOutputParser()).pipe(search);

const result = await chain.invoke({
  input: "Who is the current prime minister of Malaysia?",
});

console.log(result);
/*
  Anwar Ibrahim
*/
```

### Agents
// todo
https://js.langchain.com/docs/expression_language/cookbook/agents

## Modules
LangChain provide Prompts, Language Models and Output parsers.

* Prompts: Templatize, dynamically select, and manage model inputs
* Language models: Make calls to language models through common interfaces
* Output parsers: Extract information from model outputs

### prompt templates: Parametrize model inputs
Language models take text as input - that text is commonly referred to as a prompt.

A prompt template can contain:
* instructions to the language model,
* a set of few shot examples to help the language model generate a better response,
* a question to the language model.

#### normal prompt template
create a prompt template:

```typescript
import { PromptTemplate } from "langchain/prompts";

const oneInputPrompt = new PromptTemplate({
    inputVariables: ["adjective"],
    template: "Tell me a {adjective} joke.",
});
const prompt = await oneInputPrompt.format({
    adjective: "funny",
});
```

```typescript
import { PromptTemplate } from "langchain/prompts";

const template = "Tell me a {adjective} joke about {content}.";

const promptTemplate = PromptTemplate.fromTemplate(template);
console.log(promptTemplate.inputVariables);
// ['adjective', 'content']
const prompt = await promptTemplate.format({
    adjective: "funny",
    content: "chickens",
});
```

#### chat prompt template
Chat Models take a list of chat messages as input, which is differ from raw string in that every chat message associated
to a role.

For example, in OpenAI Chat Completion API, a chat message can be associated with an AI, human or system role.

We should use chat related templates provided by LangChain when interacting with chat models. So to create a message 
template associated with a role, you would use the **corresponding <ROLE>MessagePromptTemplate**.

```typescript
import {
  ChatPromptTemplate,
  PromptTemplate,
  SystemMessagePromptTemplate,
  AIMessagePromptTemplate,
  HumanMessagePromptTemplate,
} from "langchain/prompts";
import { AIMessage, HumanMessage, SystemMessage } from "langchain/schema";
```

##### create

```typescript
const systemTemplate =
  "You are a helpful assistant that translates {input_language} to {output_language}.";
const humanTemplate = "{text}";

const chatPrompt = ChatPromptTemplate.fromMessages([
  ["system", systemTemplate],
  ["human", humanTemplate],
]);

// Format the messages
const formattedChatPrompt = await chatPrompt.formatMessages({
  input_language: "English",
  output_language: "French",
  text: "I love programming.",
});

console.log(formattedChatPrompt);

/*
  [
    SystemMessage {
      content: 'You are a helpful assistant that translates English to French.'
    },
    HumanMessage {
      content: 'I love programming.'
    }
  ]
*/
```

```typescript
const template =
  "You are a helpful assistant that translates {input_language} to {output_language}.";
const systemMessagePrompt = SystemMessagePromptTemplate.fromTemplate(template);
```

```typescript
const prompt = new PromptTemplate({
  template:
    "You are a helpful assistant that translates {input_language} to {output_language}.",
  inputVariables: ["input_language", "output_language"],
});
const systemMessagePrompt2 = new SystemMessagePromptTemplate({
  prompt,
});
```

with typescript:

```typescript
const chatPrompt = ChatPromptTemplate.fromMessages<{
  input_language: string;
  output_language: string;
  text: string;
}>([systemMessagePrompt, humanMessagePrompt]);
```

#### Few Shot Prompt Templates
Few shot prompting is a prompting technique which provides the Large Language Model (LLM) with a list of examples, and then asks the LLM to generate some text following the lead of the examples provided.

Say you want your LLM to respond in a **specific format**. You can few shot prompt the LLM with a list of question answer pairs so it knows what format to respond in.

`FewShotChatMessagePromptTemplate` and `ChatPromptTemplate`:

```typescript
import {
  ChatPromptTemplate,
  FewShotChatMessagePromptTemplate,
} from "langchain/prompts";

const examples = [
  {
    input: "Could the members of The Police perform lawful arrests?",
    output: "what can the members of The Police do?",
  },
  {
    input: "Jan Sindel's was born in what country?",
    output: "what is Jan Sindel's personal history?",
  },
];
const examplePrompt = ChatPromptTemplate.fromTemplate(`Human: {input}
AI: {output}`);
const fewShotPrompt = new FewShotChatMessagePromptTemplate({
  examplePrompt,
  examples,
  inputVariables: [], // no input variables
});
const formattedPrompt = await fewShotPrompt.format({});
console.log(formattedPrompt);
```

```json
[
  HumanMessage {
    lc_namespace: [ 'langchain', 'schema' ],
    content: 'Human: Could the members of The Police perform lawful arrests?\n' +
      'AI: what can the members of The Police do?',
    additional_kwargs: {}
  },
  HumanMessage {
    lc_namespace: [ 'langchain', 'schema' ],
    content: "Human: Jan Sindel's was born in what country?\n" +
      "AI: what is Jan Sindel's personal history?",
    additional_kwargs: {}
  }
]
```

```typescript
const model = new ChatOpenAI({});
const examples = [
  {
    input: "Could the members of The Police perform lawful arrests?",
    output: "what can the members of The Police do?",
  },
  {
    input: "Jan Sindel's was born in what country?",
    output: "what is Jan Sindel's personal history?",
  },
];
const examplePrompt = ChatPromptTemplate.fromTemplate(`Human: {input}
AI: {output}`);
const fewShotPrompt = new FewShotChatMessagePromptTemplate({
  prefix:
    "Rephrase the users query to be more general, using the following examples",
  suffix: "Human: {input}",
  examplePrompt,
  examples,
  inputVariables: ["input"],
});
const formattedPrompt = await fewShotPrompt.format({
  input: "What's France's main city?",
});

const response = await model.invoke(formattedPrompt);
console.log(response);
```

```json
AIMessage {
  lc_namespace: [ 'langchain', 'schema' ],
  content: 'What is the capital of France?',
  additional_kwargs: { function_call: undefined }
}
```

##### Few shot with Function
When you want to pass a variable not along with the other input variables, you can call `partial` function.

```typescript
const getCurrentDate = () => {
  return new Date().toISOString();
};

const prompt = new FewShotChatMessagePromptTemplate({
  template: "Tell me a {adjective} joke about the day {date}",
  inputVariables: ["adjective", "date"],
});

const partialPrompt = await prompt.partial({
  date: getCurrentDate,
});

const formattedPrompt = await partialPrompt.format({
  adjective: "funny",
});

console.log(formattedPrompt);

// Tell me a funny joke about the day 2023-07-13T00:54:59.287Z
```

##### Few Shot vs Chat Few Shot

```typescript
import {
  FewShotPromptTemplate,
  FewShotChatMessagePromptTemplate,
} from "langchain/prompts";

const examples = [
  {
    input: "Could the members of The Police perform lawful arrests?",
    output: "what can the members of The Police do?",
  },
  {
    input: "Jan Sindel's was born in what country?",
    output: "what is Jan Sindel's personal history?",
  },
];
const prompt = `Human: {input}
AI: {output}`;
const examplePromptTemplate = PromptTemplate.fromTemplate(prompt);
const exampleChatPromptTemplate = ChatPromptTemplate.fromTemplate(prompt);
const chatFewShotPrompt = new FewShotChatMessagePromptTemplate({
  examplePrompt: exampleChatPromptTemplate,
  examples,
  inputVariables: [], // no input variables
});
const fewShotPrompt = new FewShotPromptTemplate({
  examplePrompt: examplePromptTemplate,
  examples,
  inputVariables: [], // no input variables
});
```

```typescript
console.log("Chat Few Shot: ", await chatFewShotPrompt.formatMessages({}));
/**
Chat Few Shot:  [
  HumanMessage {
    lc_namespace: [ 'langchain', 'schema' ],
    content: 'Human: Could the members of The Police perform lawful arrests?\n' +
      'AI: what can the members of The Police do?',
    additional_kwargs: {}
  },
  HumanMessage {
    lc_namespace: [ 'langchain', 'schema' ],
    content: "Human: Jan Sindel's was born in what country?\n" +
      "AI: what is Jan Sindel's personal history?",
    additional_kwargs: {}
  }
]
 */

console.log("Few Shot: ", await fewShotPrompt.formatPromptValue({}));
/**
Few Shot:

Human: Could the members of The Police perform lawful arrests?
AI: what can the members of The Police do?

Human: Jan Sindel's was born in what country?
AI: what is Jan Sindel's personal history?
 */
```

### Example selectors: Dynamically select examples to include in prompts
