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

## Prompts
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
Main difference is input and output values.

`FewShotChatMessagePromptTemplate` works by taking in a list of `ChatPromptTemplate` for examples, and its output is a list of instances of `BaseMessage`.

On the other hand, `FewShotPromptTemplate` works by taking in a `PromptTemplate` for examples, and its output is a string.

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

##### With non chat model
LangChain also provides a class for few shot prompt formatting for non chat models: `FewShotPromptTemplate`.

###### Partial with function

```typescript
import {
  ChatPromptTemplate,
  FewShotChatMessagePromptTemplate,
} from "langchain/prompts";

const examplePrompt = PromptTemplate.fromTemplate("{foo}{bar}");
const prompt = new FewShotPromptTemplate({
  prefix: "{foo}{bar}",
  examplePrompt,
  inputVariables: ["foo", "bar"],
});
const partialPrompt = await prompt.partial({
  foo: () => Promise.resolve("boo"),
});
const formatted = await partialPrompt.format({ bar: "baz" });
console.log(formatted);
// boobaz\n
```

###### With Functions and Example Selector

```typescript
import {
  ChatPromptTemplate,
  FewShotChatMessagePromptTemplate,
} from "langchain/prompts";

const examplePrompt = PromptTemplate.fromTemplate("An example about {x}");
const exampleSelector = await LengthBasedExampleSelector.fromExamples(
    [{ x: "foo" }, { x: "bar" }],
    { examplePrompt, maxLength: 200 }
);
const prompt = new FewShotPromptTemplate({
  prefix: "{foo}{bar}",
  exampleSelector,
  examplePrompt,
  inputVariables: ["foo", "bar"],
});
const partialPrompt = await prompt.partial({
  foo: () => Promise.resolve("boo"),
});
const formatted = await partialPrompt.format({ bar: "baz" });
console.log(formatted);
// boobaz
// An example about foo
// An example about bar
```

#### Partial prompt template
For example, you have a prompt using two variable. You get value of one variable early and the other later. Don't need to 
wait until the later one with `partial`.

```typescript
import { PromptTemplate } from "langchain/prompts";

const prompt = new PromptTemplate({
  template: "{foo}{bar}",
  inputVariables: ["foo", "bar"],
});

const partialPrompt = await prompt.partial({
  foo: "foo",
});

const formattedPrompt = await partialPrompt.format({
  bar: "baz",
});

console.log(formattedPrompt);

// foobaz
```

For partial with function, example is same as [FewShotChatMessagePromptTemplate](#Few-shot-with-Function). `partial` is common function of prompt template.

Or you can initialize the prompt with partialed varible:
```typescript
const prompt = new PromptTemplate({
  template: "{foo}{bar}",
  inputVariables: ["bar"],
  partialVariables: {
    // value can be function which return value
    foo: "foo",
  },
});

const formattedPrompt = await prompt.format({
  bar: "baz",
});

console.log(formattedPrompt);

// foobaz
```

#### Composition
This part goes over how to compose multiple prompts together with PipelinePrompt.

A PipelinePrompt consists of two main parts:
- Final prompt: This is the final prompt that is returned
- Pipeline prompts: This is a list of tuples, consisting of a string name and a prompt template. Each prompt template will
be formatted and then passed to future prompt templates as a variable with the same name.

```typescript
import { PromptTemplate, PipelinePromptTemplate } from "langchain/prompts";

const fullPrompt = PromptTemplate.fromTemplate(`{introduction}

{example}

{start}`);

const introductionPrompt = PromptTemplate.fromTemplate(
  `You are impersonating {person}.`
);

const examplePrompt =
  PromptTemplate.fromTemplate(`Here's an example of an interaction:
Q: {example_q}
A: {example_a}`);

const startPrompt = PromptTemplate.fromTemplate(`Now, do this for real!
Q: {input}
A:`);

const composedPrompt = new PipelinePromptTemplate({
  pipelinePrompts: [
    {
      name: "introduction",
      prompt: introductionPrompt,
    },
    {
      name: "example",
      prompt: examplePrompt,
    },
    {
      name: "start",
      prompt: startPrompt,
    },
  ],
  finalPrompt: fullPrompt,
});

const formattedPrompt = await composedPrompt.format({
  person: "Elon Musk",
  example_q: `What's your favorite car?`,
  example_a: "Telsa",
  input: `What's your favorite social media site?`,
});

console.log(formattedPrompt);

/*
  You are impersonating Elon Musk.

  Here's an example of an interaction:
  Q: What's your favorite car?
  A: Telsa

  Now, do this for real!
  Q: What's your favorite social media site?
  A:
*/
```

### Example selectors: Dynamically select examples to include in prompts
The base interface of the example selector is defined as below:

```typescript
class BaseExampleSelector {
  addExample(example: Example): Promise<void | string>;

  selectExamples(input_variables: Example): Promise<Example[]>;
}
```

It needs to expose a selectExamples - this takes in the input variables and then returns a list of examples method - and
an addExample method, which saves an example for later selection. It is up to each specific implementation as to how 
those examples are saved and selected.

#### Select by length
This example selector selects which examples to use based on length. This is useful when you are worried about 
constructing a prompt that will go over the length of the context window.

```typescript
import {
  LengthBasedExampleSelector,
  PromptTemplate,
  FewShotPromptTemplate,
} from "langchain/prompts";

export async function run() {
  // Create a prompt template that will be used to format the examples.
  const examplePrompt = new PromptTemplate({
    inputVariables: ["input", "output"],
    template: "Input: {input}\nOutput: {output}",
  });

  // Create a LengthBasedExampleSelector that will be used to select the examples.
  const exampleSelector = await LengthBasedExampleSelector.fromExamples(
    [
      { input: "happy", output: "sad" },
      { input: "tall", output: "short" },
      { input: "energetic", output: "lethargic" },
      { input: "sunny", output: "gloomy" },
      { input: "windy", output: "calm" },
    ],
    {
      examplePrompt,
      maxLength: 25,
    }
  );

  // Create a FewShotPromptTemplate that will use the example selector.
  const dynamicPrompt = new FewShotPromptTemplate({
    // We provide an ExampleSelector instead of examples.
    exampleSelector,
    examplePrompt,
    prefix: "Give the antonym of every input",
    suffix: "Input: {adjective}\nOutput:",
    inputVariables: ["adjective"],
  });

  // An example with small input, so it selects all examples.
  console.log(await dynamicPrompt.format({ adjective: "big" }));
  /*
   Give the antonym of every input

   Input: happy
   Output: sad

   Input: tall
   Output: short

   Input: energetic
   Output: lethargic

   Input: sunny
   Output: gloomy

   Input: windy
   Output: calm

   Input: big
   Output:
   */
  
  // TODO why
  // 如果 maxlength 为 4，显示：
  // Give the antonym of every input
  //
  // Input: big
  // Output:
  
  // 如果 maxlength 为 5，显示：
  // Give the antonym of every input
  //
  // Input: happy
  // Output: sad
  //
  // Input: big
  // Output:

  // An example with long input, so it selects only one example.
  const longString =
    "big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else";
  console.log(await dynamicPrompt.format({ adjective: longString }));
  /*
   Give the antonym of every input

   Input: happy
   Output: sad

   Input: big and huge and massive and large and gigantic and tall and much much much much much bigger than everything else
   Output:
   */
}
```

#### Select by similarity
This object selects examples based on similarity to the inputs.

The fields of the examples object will be used as parameters to format the examplePrompt passed to the 
FewShotPromptTemplate. Each example should therefore contain **all required fields** for the example prompt you are using.

```typescript
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import {
  SemanticSimilarityExampleSelector,
  PromptTemplate,
  FewShotPromptTemplate,
} from "langchain/prompts";
import { HNSWLib } from "langchain/vectorstores/hnswlib";

// Create a prompt template that will be used to format the examples.
const examplePrompt = PromptTemplate.fromTemplate(
  "Input: {input}\nOutput: {output}"
);

// Create a SemanticSimilarityExampleSelector that will be used to select the examples.
const exampleSelector = await SemanticSimilarityExampleSelector.fromExamples(
  [
    { input: "happy", output: "sad" },
    { input: "tall", output: "short" },
    { input: "energetic", output: "lethargic" },
    { input: "sunny", output: "gloomy" },
    { input: "windy", output: "calm" },
  ],
  new OpenAIEmbeddings(),
  HNSWLib,
  { k: 1 }
);

// Create a FewShotPromptTemplate that will use the example selector.
const dynamicPrompt = new FewShotPromptTemplate({
  // We provide an ExampleSelector instead of examples.
  exampleSelector,
  examplePrompt,
  prefix: "Give the antonym of every input",
  suffix: "Input: {adjective}\nOutput:",
  inputVariables: ["adjective"],
});

// Input is about the weather, so should select eg. the sunny/gloomy example
console.log(await dynamicPrompt.format({ adjective: "rainy" }));
/*
  Give the antonym of every input

  Input: sunny
  Output: gloomy

  Input: rainy
  Output:
*/

// Input is a measurement, so should select the tall/short example
console.log(await dynamicPrompt.format({ adjective: "large" }));
/*
  Give the antonym of every input

  Input: tall
  Output: short

  Input: large
  Output:
*/
```

### Prompt selectors: programmatically select a prompt
https://js.langchain.com/docs/modules/model_io/prompts/prompt_selectors/

## Language Model
LangChain provides interfaces and integrations for two types of models:
* LLMs: Models that take a text string as input and return a text string
* Chat models: Models that are backed by a language model but take a list of Chat Messages as input and return a Chat Message

LLMs in LangChain refer to pure text completion models. The APIs they wrap take a string prompt as input and output a 
string completion. OpenAI's GPT-3 is implemented as an LLM.

Chat models are often backed by LLMs but tuned specifically for having conversations. And, crucially, their provider 
APIs expose a different interface than pure text completion models. Instead of a single string, they take **a list of 
chat messages** as input. Usually these messages are **labeled** with the speaker (usually one of "System", "AI", and 
"Human"). And they return a ("AI") chat message as output. GPT-4 and Anthropic's Claude are both implemented as Chat Models.

To make it possible to swap LLMs and Chat Models, both implement the Base Language Model interface. This exposes common 
methods "predict", which takes a string and returns a string, and "predict messages", which takes messages and returns a message.
If you are using a specific model it's recommended you use the methods specific to that model class, but if you're 
creating an application that should work with different types of models the shared interface can be helpful.

### LLMs
LangChain does not serve its own LLMs, but rather provides a standard interface for interacting with many different LLMs.

#### Get started
1. install npm package such as openai
2. setup api key by command or passing to constructor

#### call
The simplest way to use an LLM is the `.call` method: pass in a string, get a string completion.

`generate` lets you can call the model with a list of strings, getting back a more complete response than just the text.
Including things like multiple top responses and other LLM provider-specific information

```typescript
const llmResult = await llm.generate(
  ["Tell me a joke", "Tell me a poem"],
  ["Tell me a joke", "Tell me a poem"]
);

console.log(llmResult.generations[0]);
/*
  [
    {
      text: "\n\nQ: What did the fish say when it hit the wall?\nA: Dam!",
      generationInfo: { finishReason: "stop", logprobs: null }
    }
  ]
*/

console.log(llmResult.llmOutput);

/*
  {
    tokenUsage: { completionTokens: 46, promptTokens: 8, totalTokens: 54 }
  }
*/
```

Here's an example with additional parameters, which sets -1 for max_tokens to turn on token size calculations:

```typescript
import { OpenAI } from "langchain/llms/openai";

export const run = async () => {
  const model = new OpenAI({
    // customize openai model that's used, `gpt-3.5-turbo-instruct` is the default
    modelName: "gpt-3.5-turbo-instruct",

    // `max_tokens` supports a magic -1 param where the max token length for the specified modelName
    //  is calculated and included in the request to OpenAI as the `max_tokens` param
    maxTokens: -1,

    // use `modelKwargs` to pass params directly to the openai call
    // note that they use snake_case instead of camelCase
    modelKwargs: {
      user: "me",
    },

    // for additional logging for debugging purposes
    verbose: true,
  });

  const resA = await model.call(
    "What would be a good company name a company that makes colorful socks?"
  );
  console.log({ resA });
  // { resA: '\n\nSocktastic Colors' }
};
```

#### Advanced info
Both LLMs and Chat Models are built on top of the BaseLanguageModel class. This class provides a common interface for all models, and allows us to easily swap out models in chains without changing the rest of the code.
LLM和聊天模型都是建立在 BaseLanguageModel 类之上，这个类为所有模型提供了一个公共接口，并允许我们在不更改其余代码的情况下轻松地交换链中的模型。

The BaseLanguageModel class has two abstract methods: generatePrompt and getNumTokens, which are implemented by BaseChatModel and BaseLLM respectively.
BaseLanguageModel  类有两个抽象方法： generatePrompt  和  getNumTokens ，分别由  BaseChatModel  和  BaseLLM  实现。

BaseLLM is a subclass of BaseLanguageModel that provides a common interface for LLMs while BaseChatModel is a subclass of BaseLanguageModel that provides a common interface for chat models.
BaseLLM  是  BaseLanguageModel  的子类，为 LLM 提供公共接口，而  BaseChatModel  是  BaseLanguageModel  的子类，为聊天模型提供公共接口。




#### Cancelling requests
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/cancelling_requests

#### maxRetries: Dealing with API Errors
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/dealing_with_api_errors

#### maxConcurrency: Dealing with Rate Limits
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/dealing_with_rate_limits

#### Caching: speed up your application
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/llm_caching

#### Streaming: processing it as soon as it's available
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/streaming_llm

#### Subscribing to events
https://js.langchain.com/docs/modules/model_io/models/llms/how_to/subscribing_events

#### Adding a timeout
By default, LangChain will wait indefinitely for a response from the model provider.

```typescript
import { OpenAI } from "langchain/llms/openai";

const model = new OpenAI({ temperature: 1 });

const resA = await model.call(
  "What would be a good company name a company that makes colorful socks?",
  { timeout: 1000 } // 1s timeout
);

console.log({ resA });
// '\n\nSocktastic Colors' }
```

### Chat models

#### Messages
The chat model interface is based around messages rather than raw text. The types of messages currently supported in 
LangChain are AIMessage, HumanMessage, SystemMessage, FunctionMessage, and ChatMessage -- ChatMessage takes in an 
arbitrary role parameter. Most of the time, you'll just be dealing with HumanMessage, AIMessage, and SystemMessage

#### call
You can generate LLM responses by calling `.invoke` and passing in whatever inputs you defined in the `Runnable`.

```typescript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { PromptTemplate } from "langchain/prompts";

const chat = new ChatOpenAI({});
// Create a prompt to start the conversation.
const prompt =
  PromptTemplate.fromTemplate(`You're a dog, good luck with the conversation.
Question: {question}`);
// Define your runnable by piping the prompt into the chat model.
const runnable = prompt.pipe(chat);
// Call .invoke() and pass in the input defined in the prompt template.
const response = await runnable.invoke({ question: "Who's a good boy??" });
console.log(response);
// AIMessage { content: "Woof woof! Thank you for asking! I believe I'm a good boy! I try my best to be a good dog and 
// make my humans happy. Wagging my tail happily here! How can I make your day better?" }
```

You can get chat completions by passing one or more messages to the chat model. The response will be a message.

```typescript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { HumanMessage } from "langchain/schema";

const chat = new ChatOpenAI({});
// Pass in a list of messages to `call` to start a conversation. In this simple example, we only pass in one message.
const response = await chat.call([
  new HumanMessage(
    "What is a good name for a company that makes colorful socks?"
  ),
]);
console.log(response);
// AIMessage { text: '\n\nRainbow Sox Co.' }
```

```typescript
const response2 = await chat.call([
  new SystemMessage(
    "You are a helpful assistant that translates English to French."
  ),
  new HumanMessage("Translate: I love programming."),
]);
console.log(response2);
// AIMessage { text: "J'aime programmer." }
```

You can generate completions for multiple sets of messages using `generate`. This returns an `LLMResult` with an additional 
`message` parameter.

```typescript
const response3 = await chat.generate([
  [
    new SystemMessage(
      "You are a helpful assistant that translates English to French."
    ),
    new HumanMessage(
      "Translate this sentence from English to French. I love programming."
    ),
  ],
  [
    new SystemMessage(
      "You are a helpful assistant that translates English to French."
    ),
    new HumanMessage(
      "Translate this sentence from English to French. I love artificial intelligence."
    ),
  ],
]);
console.log(response3);
/*
  {
    generations: [
      [
        {
          text: "J'aime programmer.",
          message: AIMessage { text: "J'aime programmer." },
        }
      ],
      [
        {
          text: "J'aime l'intelligence artificielle.",
          message: AIMessage { text: "J'aime l'intelligence artificielle." }
        }
      ]
    ]
  }
*/
```

You can recover things like token usage from this LLMResult:

```typescript
console.log(response3.llmOutput);
/*
  {
    tokenUsage: { completionTokens: 20, promptTokens: 69, totalTokens: 89 }
  }
*/
```

#### Cancelling requests
#### maxRetries: Dealing with API Errors
#### maxConcurrency: Dealing with Rate Limits
#### Caching: speed up your application
#### Streaming: processing it as soon as it's available
#### Subscribing to events
#### Adding a timeout
#### LLM Chain
#### OpenAI function calling
There are two main ways to apply functions to your OpenAI calls.

The first and most simple is by attaching a function directly to the .invoke({}) method:

```typescript
/* Define your function schema */
const extractionFunctionSchema = {...}

/* Instantiate ChatOpenAI class */
const model = new ChatOpenAI({ modelName: "gpt-4" });

/**
 * Call the .invoke method on the model, directly passing
 * the function arguments as call args.
 */
const result = await model.invoke([new HumanMessage("What a beautiful day!")], {
  functions: [extractionFunctionSchema],
  function_call: { name: "extractor" },
});

console.log({ result });
```

The second way is by calling the `.bind({})` method attaches any call arguments passed in to all future calls to the model.
It is useful when you want to call the same function twice.

```typescript
/* Define your function schema */
const extractionFunctionSchema = {...}

/* Instantiate ChatOpenAI class and bind function arguments to the model */
const model = new ChatOpenAI({ modelName: "gpt-4" }).bind({
  functions: [extractionFunctionSchema],
  function_call: { name: "extractor" },
});

/* Now we can call the model without having to pass the function arguments in again */
const result = await model.invoke([new HumanMessage("What a beautiful day!")]);

console.log({ result });
```

Specifying the function_call argument will force the model to return a response using the specified function. This is 
useful if you have multiple schemas you'd like the model to pick from.

Example:
```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { HumanMessage } from "langchain/schema";
// Example function schema:
const extractionFunctionSchema = {
  name: "extractor",
  description: "Extracts fields from the input.",
  parameters: {
    type: "object",
    properties: {
      tone: {
        type: "string",
        enum: ["positive", "negative"],
        description: "The overall tone of the input",
      },
      word_count: {
        type: "number",
        description: "The number of words in the input",
      },
      chat_response: {
        type: "string",
        description: "A response to the human's input",
      },
    },
    required: ["tone", "word_count", "chat_response"],
  },
};

const model = new ChatOpenAI({
  modelName: "gpt-4",
}).bind({
  functions: [extractionFunctionSchema],
  function_call: { name: "extractor" },
});

const result = await model.invoke([new HumanMessage("What a beautiful day!")]);

console.log(result);
/*
AIMessage {
  lc_serializable: true,
  lc_kwargs: { content: '', additional_kwargs: { function_call: [Object] } },
  lc_namespace: [ 'langchain', 'schema' ],
  content: '',
  name: undefined,
  additional_kwargs: {
    function_call: {
      name: 'extractor',
      arguments: '{\n' +
        '  "tone": "positive",\n' +
        '  "word_count": 4,\n' +
        `  "chat_response": "I'm glad you're enjoying the day! What makes it so beautiful for you?"\n` +
        '}'
    }
  }
}
*/
```

> [Usage with Zod](https://js.langchain.com/docs/modules/model_io/models/chat/how_to/function_calling#usage-with-zod)

#### Promopts
https://js.langchain.com/docs/modules/model_io/models/chat/how_to/prompts



## Output parsers
Language models just output text. Output parsers are classes that help structure language model responses.

There are two main methods an output parser must implement:
- "Get format instructions": A method which returns a string containing instructions for how the output of a language model should be formatted.
- "Parse": A method which takes in a string (assumed to be the response from a language model) and parses it into some structure.

And one optional:

"Parse with prompt": A method which takes in a string (assumed to be the response from a language model) and a prompt 
(assumed to the prompt that generated such a response) and parses it into some structure. The prompt is largely provided 
in the event the OutputParser wants to retry or fix the output in some way, and needs information from the prompt to do so.

“Parse with prompt”：一种方法，它接受一个字符串(假定是来自语言模型的响应)和一个提示符(假定是生成此响应的提示符)，并将其解析为某种结构。
提示符主要在OutputParser想要以某种方式重试或修复输出时提供，并需要从提示符中获取信息来这样做。

### Structured Output Parser
This output parser can be used when you want to return multiple fields. If you want complex schema returned 
(i.e. a JSON object with arrays of strings), use the Zod Schema detailed below.

// TODO Model I/O Output parsers

## Retrieval
Retrieval Augmented Generation (RAG).

Key modules:

Source(data) => Load => Transform => Embed(?) => store => Retrieve

### Document loaders
LangChain provides many different document loaders to load all types of documents (html, PDF, code) from all types 
of locations.

Document loaders expose a "`load`" method for loading data as documents from `.txt`, web page and many other source.

#### Creating Document
A Document is a piece of text and associated metadata. The piece of text is what we interact with the language model, 
while the optional metadata is useful for keeping track of metadata about the document (such as the source).

```typescript
import { Document } from "langchain/document";

const doc = new Document({ pageContent: "foo", metadata: { source: "1" } });
```

#### CSV
https://js.langchain.com/docs/modules/data_connection/document_loaders/how_to/csv

#### Custom document loaders
https://js.langchain.com/docs/modules/data_connection/document_loaders/how_to/custom

#### Load file in directory
https://js.langchain.com/docs/modules/data_connection/document_loaders/how_to/file_directory

#### JSON and PDF
https://js.langchain.com/docs/modules/data_connection/document_loaders/how_to/json
https://js.langchain.com/docs/modules/data_connection/document_loaders/how_to/pdf

### Document transformers
A key part of retrieval is fetching only the relevant parts of documents. One of the primary ones here is splitting 
(or chunking) a large document into smaller chunks.

#### Split by character
`RecursiveCharacterTextSplitter`. This will split documents recursively by different characters - starting with `\n\n`, 
then `\n`, then `" "`.

There are two important parameters chunkSize and chunkOverlap. `chunkSize` controls the max size of the final documents.
`chunkOverlap` specifies how much overlap there should be between chunks.

```typescript
const text = `Hi.\n\nI'm Harrison.\n\nHow? Are? You?\nOkay then f f f f.
This is a weird text to write, but gotta test the splittingggg some how.\n\n
Bye!\n\n-H.`;
const splitter = new RecursiveCharacterTextSplitter({
    chunkSize: 10,
    chunkOverlap: 1,
});
// we can pass docuemnt or text
const docOutput = await splitter.splitDocuments([
    new Document({ pageContent: text }),
]);
const output = await splitter.createDocuments([text]);
```

#### Split code and markup
LangChain supports a variety of different markup and programming language-specific text splitters to split your text.

`SupportedTextSplitterLanguages` is an Array of supported languages.

Using `RecursiveCharacterTextSplitter.fromLanguage` to split text.

example: https://js.langchain.com/docs/modules/data_connection/document_transformers/text_splitters/code_splitter

#### Contextual chunk headers
在矢量存储中存储大量任意文档，并对其执行问答任务时，简单拆分文档可能无法为 LLM 提供足够上下文来确定是否多个块引用了相同信息或解决不同来源的矛盾信息。

如果知道如何过滤，直接设置好 document 的 metadata 可以解决，但在 vector store 处理前你不一定知道。可以添加 contextual information 
在 headers 上，有助于处理 vector store 的查询。

```typescript
import { OpenAI } from "langchain/llms/openai";
import { RetrievalQAChain, loadQAStuffChain } from "langchain/chains";
import { CharacterTextSplitter } from "langchain/text_splitter";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { HNSWLib } from "langchain/vectorstores/hnswlib";

const splitter = new CharacterTextSplitter({
  chunkSize: 1536,
  chunkOverlap: 200,
});

const jimDocs = await splitter.createDocuments(
  [`My favorite color is blue.`],
  [],
  {
    chunkHeader: `DOCUMENT NAME: Jim Interview\n\n---\n\n`,
    appendChunkOverlapHeader: true,
  }
);

const pamDocs = await splitter.createDocuments(
  [`My favorite color is red.`],
  [],
  {
    chunkHeader: `DOCUMENT NAME: Pam Interview\n\n---\n\n`,
    appendChunkOverlapHeader: true,
  }
);

const vectorStore = await HNSWLib.fromDocuments(
  jimDocs.concat(pamDocs),
  new OpenAIEmbeddings()
);

const model = new OpenAI({ temperature: 0 });

const chain = new RetrievalQAChain({
  combineDocumentsChain: loadQAStuffChain(model),
  retriever: vectorStore.asRetriever(),
  returnSourceDocuments: true,
});
const res = await chain.call({
  query: "What is Pam's favorite color?",
});

console.log(JSON.stringify(res, null, 2));

/*
  {
    "text": " Red.",
    "sourceDocuments": [
      {
        "pageContent": "DOCUMENT NAME: Pam Interview\n\n---\n\nMy favorite color is red.",
        "metadata": {
          "loc": {
            "lines": {
              "from": 1,
              "to": 1
            }
          }
        }
      },
      {
        "pageContent": "DOCUMENT NAME: Jim Interview\n\n---\n\nMy favorite color is blue.",
        "metadata": {
          "loc": {
            "lines": {
              "from": 1,
              "to": 1
            }
          }
        }
      }
    ]
  }
*/
```

#### Custom text splitters
https://js.langchain.com/docs/modules/data_connection/document_transformers/text_splitters/custom_text_splitter

#### Recursively split by character
https://js.langchain.com/docs/modules/data_connection/document_transformers/text_splitters/recursive_text_splitter

#### TokenTextSplitter

### Text embedding models
Creating embeddings for documents is a important part. Embeddings capture the semantic meaning of text, allowing you to 
quickly and efficiently find other pieces of text that are similar in vector store.

The Embeddings class is a class designed for interfacing with text embedding models with standard interface.
The embedding class exposes a `embedQuery` and `embedDocuments` method for queries and documents.

#### maxRetries: Dealing with api error

#### Cache: without recompute

#### Dealing with rate limits

#### timeout
https://js.langchain.com/docs/modules/data_connection/text_embedding/how_to/timeouts

### Vector stores 矢量存储
LangChain provides integrations with many different vector stores so that LangChain support efficient storage and 
searching of these embeddings.

A vector store takes care of storing embedded data and performing vector search for you.

A key part of working with vector stores is creating the vector to put in them, which is usually created via embeddings.

#### creating index from texts or loader
Use `fromTexts` or `fromDocuments` to do so, then we can create vector store from index, document or texts.

```typescript
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";

const vectorStore = await MemoryVectorStore.fromTexts(
  ["Hello world", "Bye bye", "hello nice world"],
  [{ id: 2 }, { id: 1 }, { id: 3 }],
  new OpenAIEmbeddings()
);

const resultOne = await vectorStore.similaritySearch("hello world", 1);

// from loader
import { TextLoader } from "langchain/document_loaders/fs/text";

// Create docs with a loader
const loader = new TextLoader("src/document_loaders/example_data/example.txt");
const docs = await loader.load();

// Load the docs into the vector store
const vectorStore = await MemoryVectorStore.fromDocuments(
    docs,
    new OpenAIEmbeddings()
);

// Search for the most similar document
const resultOne = await vectorStore.similaritySearch("hello world", 1);
```

#### which vector store to pick
https://js.langchain.com/docs/modules/data_connection/vectorstores/#which-one-to-pick

### Retrievers
Once the data is in the vector store, you still need to retrieve it. LangChain supports many different retrieval algorithms.

A retriever does not need to be able to store documents, only to return (or retrieve) it.

#### A example showcases question answering over documents.
Question answering over documents consists of four steps:

1. Create an index
2. Create a Retriever from that index
3. Create a question answering chain
4. Ask questions!

```shell
npm install -S hnswlib-node
```

```typescript
import { HNSWLib } from "langchain/vectorstores/hnswlib";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import * as fs from "fs";
import {
  RunnablePassthrough,
  RunnableSequence,
} from "langchain/schema/runnable";
import { StringOutputParser } from "langchain/schema/output_parser";
import {
  ChatPromptTemplate,
  HumanMessagePromptTemplate,
  SystemMessagePromptTemplate,
} from "langchain/prompts";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { formatDocumentsAsString } from "langchain/util/document";

// Initialize the LLM to use to answer the question.
const model = new ChatOpenAI({});
const text = fs.readFileSync("state_of_the_union.txt", "utf8");
const textSplitter = new RecursiveCharacterTextSplitter({ chunkSize: 1000 });
// split it into small document
const docs = await textSplitter.createDocuments([text]);
// Create a vector store from the documents.
// also embeds the documents using the passed OpenAIEmbeddings instance
const vectorStore = await HNSWLib.fromDocuments(docs, new OpenAIEmbeddings());

// Initialize a retriever wrapper around the vector store
const vectorStoreRetriever = vectorStore.asRetriever();

// Create a system & human prompt for the chat model
const SYSTEM_TEMPLATE = `Use the following pieces of context to answer the question at the end.
If you don't know the answer, just say that you don't know, don't try to make up an answer.
----------------
{context}`;
const messages = [
  SystemMessagePromptTemplate.fromTemplate(SYSTEM_TEMPLATE),
  HumanMessagePromptTemplate.fromTemplate("{question}"),
];
const prompt = ChatPromptTemplate.fromMessages(messages);

const chain = RunnableSequence.from([
  {
    context: vectorStoreRetriever.pipe(formatDocumentsAsString),
    question: new RunnablePassthrough(),
  },
  prompt,
  model,
  new StringOutputParser(),
]);

const answer = await chain.invoke(
  "What did the president say about Justice Breyer?"
);

console.log({ answer });

/*
{
  answer: 'The president thanked Justice Stephen Breyer for his service and honored him for his dedication to the country.'
}
*/
```

#### Contextual compression
The information most relevant to a query may be buried in a document with a lot of irrelevant text. Contextual compression is meant to fix this.

https://js.langchain.com/docs/modules/data_connection/retrievers/how_to/contextual_compression

// todo retriever 这部分开始往后。
// todo 后面有记的只代表看了那一章

### experimental

## chain

### Conversational Retrieval QA（会话关键）
https://js.langchain.com/docs/modules/chains/popular/chat_vector_db

```javascript
const { ChatOpenAI } = require("langchain/chat_models/openai");
const { HNSWLib } = require("langchain/vectorstores/hnswlib");
const { OpenAIEmbeddings } = require("langchain/embeddings/openai");
const { RecursiveCharacterTextSplitter } = require("langchain/text_splitter");
const fs = require("fs");
const { PromptTemplate } = require("langchain/prompts");
const { RunnableSequence } = require("langchain/schema/runnable");
const { StringOutputParser } = require("langchain/schema/output_parser");
const { formatDocumentsAsString }= require("langchain/util/document");

/* Initialize the LLM to use to answer the question */
const model = new ChatOpenAI({
// todo
});
/* Load in the file we want to do question answering over */
const text = fs.readFileSync('state_of_the_union.txt', 'utf-8');
/* Split the text into chunks */
const textSplitter = new RecursiveCharacterTextSplitter({ chunkSize: 1000 });

textSplitter.createDocuments([text]).then((docs) => {
  /* Create the vectorstore */
  HNSWLib.fromDocuments(docs, new OpenAIEmbeddings({
    // todo
  })).then((vectorStore) => {
    const retriever = vectorStore.asRetriever();

    const formatChatHistory = (
      human,
      ai,
      previousChatHistory
    ) => {
      const newInteraction = `Human: ${human}\nAI: ${ai}`;
      if (!previousChatHistory) {
        return newInteraction;
      }
      return `${previousChatHistory}\n\n${newInteraction}`;
    };

    const questionPrompt = PromptTemplate.fromTemplate(
      `Use the following pieces of context to answer the question at the end. If you don't know the answer, just say that you don't know, don't try to make up an answer.
  ----------------
  CONTEXT: {context}
  ----------------
  CHAT HISTORY: {chatHistory}
  ----------------
  QUESTION: {question}
  ----------------
  Helpful Answer:`
    );

    const chain = RunnableSequence.from([
      {
        question: (input) => input.question,
        chatHistory: (input) => input.chatHistory ?? "",
        context: async (input) => {
          const relevantDocs = await retriever.getRelevantDocuments(input.question);
          return formatDocumentsAsString(relevantDocs);
        },
      },
      questionPrompt,
      model,
      new StringOutputParser(),
    ]);

    const questionOne = "What did the president say about Justice Breyer?";

    chain.invoke({
      question: questionOne,
    }).then((resultOne) => {
      console.log({ resultOne });

      chain.invoke({
        chatHistory: formatChatHistory(resultOne, questionOne),
        question: "was it nice?",
      }).then((resultTwo) => {
        console.log({ resultTwo });
      });
    });
  });
});
```

### SQL（会话关键）
https://js.langchain.com/docs/modules/chains/popular/sqlite

## Memory
Chains and Agents are stateless, meaning that they treat each incoming query independently. In some applications, like 
chatbots, it is essential to remember previous interactions, both in the short and long-term. The Memory class does exactly that.

### ChatMessageHistory, ChatHistory
`ChatMessageHistory` is a core memory module,  which exposes convenience methods for saving Human messages, AI messages, and then fetching them all.

> Do not share the same history or memory instance between two different chains, a memory instance represents the history of a single conversation

```javascript
import { ChatMessageHistory } from "langchain/memory";

const history = new ChatMessageHistory();

await history.addUserMessage("Hi!");

await history.addAIChatMessage("What's up?");

const messages = await history.getMessages();

console.log(messages);

/*
  [
    HumanMessage {
      content: 'Hi!',
    },
    AIMessage {
      content: "What's up?",
    }
  ]
*/
```

```javascript
import { BufferMemory, ChatMessageHistory } from "langchain/memory";
import { HumanChatMessage, AIChatMessage } from "langchain/schema";

const pastMessages = [
  new HumanMessage("My name's Jonas"),
  new AIMessage("Nice to meet you, Jonas!"),
];

const memory = new BufferMemory({
  chatHistory: new ChatMessageHistory(pastMessages),
});
```

### BufferMemory
`BufferMemory`, a wrapper around `ChatMessageHistory` that extracts the messages into an input variable.

This memory allows for storing of messages, then later formats the messages into a prompt input variable.

```javascript
const { ChatOpenAI } = require("langchain/chat_models/openai");
const { BufferMemory } = require("langchain/memory");
const { ConversationChain } = require("langchain/chains");

const model = new ChatOpenAI({
// todo
})
const memory = new BufferMemory();
// This chain is preconfigured with a default prompt
const chain = new ConversationChain({ llm: model, memory: memory });

chain.call({ input: "Hi! I'm Jim." })
  .then((res1) => {
    console.log({ res1 });
    chain.call({ input: "What's my name?" }).then(res2 => {
      console.log({res2});
    });
  });

```

### Creating your own memory class
https://js.langchain.com/docs/modules/memory/#creating-your-own-memory-class

### Using Buffer Memory with Chat Models
https://js.langchain.com/docs/modules/memory/how_to/buffer_memory_chat

### BufferWindowMemory
`BufferWindowMemory` save last `k` interaction with chat models. so the buffer does not get too large.

```javascript
const { ChatOpenAI } = require("langchain/chat_models/openai");
const { BufferWindowMemory } = require("langchain/memory");
const { ConversationChain } = require("langchain/chains");

const model = new ChatOpenAI({
});
// check the difference of result when k is 1 and 2
const memory = new BufferWindowMemory({ k: 2 });
const chain = new ConversationChain({ llm: model, memory: memory });

chain.call({ input: "Hi! I'm Jim." })
  .then((res1) => {
    console.log({ res1 });

    // Call the chain again after the first call
    return chain.call({ input: "What's my name?" });
  })
  .then((res2) => {
    console.log({ res2 });
    return chain.call({ input: "What can you do?" });
  })
  .then((res2) => {
    console.log({ res2 });
    return chain.call({ input: "What's my name?" });
  })
  .then((res2) => {
    console.log({ res2 });
  })
  .catch((error) => {
    console.error("Error:", error);
  });
```

### Entity memory
Entity Memory remembers given facts about specific entities in a conversation. It extracts information on entities 
(using an LLM) and builds up its knowledge about that entity over time (also using an LLM).

```javascript
import { OpenAI } from "langchain/llms/openai";
import {
  EntityMemory,
  ENTITY_MEMORY_CONVERSATION_TEMPLATE,
} from "langchain/memory";
import { LLMChain } from "langchain/chains";

export const run = async () => {
  const memory = new EntityMemory({
    llm: new OpenAI({ temperature: 0 }),
    chatHistoryKey: "history", // Default value
    entitiesKey: "entities", // Default value
  });
  const model = new OpenAI({ temperature: 0.9 });
  const chain = new LLMChain({
    llm: model,
    prompt: ENTITY_MEMORY_CONVERSATION_TEMPLATE, // Default prompt - must include the set chatHistoryKey and entitiesKey as input variables.
    memory,
  });

  const res1 = await chain.call({ input: "Hi! I'm Jim." });
  console.log({
    res1,
    memory: await memory.loadMemoryVariables({ input: "Who is Jim?" }),
  });

  const res2 = await chain.call({
    input: "I work in construction. What about you?",
  });
  console.log({
    res2,
    memory: await memory.loadMemoryVariables({ input: "Who is Jim?" }),
  });
};
```

### How to use multiple memory classes in the same chain

### Conversation summary memory
`ConversationSummaryMemory`. This type of memory creates a summary of the conversation over time. This memory is most 
useful for longer conversations, where keeping the past message history in the prompt verbatim would take up too many tokens.

with llm:
```javascript
import { OpenAI } from "langchain/llms/openai";
import { ConversationSummaryMemory } from "langchain/memory";
import { LLMChain } from "langchain/chains";
import { PromptTemplate } from "langchain/prompts";

export const run = async () => {
  const memory = new ConversationSummaryMemory({
    memoryKey: "chat_history",
    llm: new OpenAI({ modelName: "gpt-3.5-turbo", temperature: 0 }),
  });

  const model = new OpenAI({ temperature: 0.9 });
  const prompt =
    PromptTemplate.fromTemplate(`The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

  Current conversation:
  {chat_history}
  Human: {input}
  AI:`);
  const chain = new LLMChain({ llm: model, prompt, memory });

  const res1 = await chain.call({ input: "Hi! I'm Jim." });
  console.log({ res1, memory: await memory.loadMemoryVariables({}) });
  /*
  {
    res1: {
      text: " Hi Jim, I'm AI! It's nice to meet you. I'm an AI programmed to provide information about the environment around me. Do you have any specific questions about the area that I can answer for you?"
    },
    memory: {
      chat_history: 'Jim introduces himself to the AI and the AI responds, introducing itself as a program designed to provide information about the environment. The AI offers to answer any specific questions Jim may have about the area.'
    }
  }
  */

  const res2 = await chain.call({ input: "What's my name?" });
  console.log({ res2, memory: await memory.loadMemoryVariables({}) });
  /*
  {
    res2: { text: ' You told me your name is Jim.' },
    memory: {
      chat_history: 'Jim introduces himself to the AI and the AI responds, introducing itself as a program designed to provide information about the environment. The AI offers to answer any specific questions Jim may have about the area. Jim asks the AI what his name is, and the AI responds that Jim had previously told it his name.'
    }
  }
  */
};
```

with chat models:
```javascript
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationSummaryMemory } from "langchain/memory";
import { LLMChain } from "langchain/chains";
import { PromptTemplate } from "langchain/prompts";

export const run = async () => {
  const memory = new ConversationSummaryMemory({
    memoryKey: "chat_history",
    llm: new ChatOpenAI({ modelName: "gpt-3.5-turbo", temperature: 0 }),
  });

  const model = new ChatOpenAI();
  const prompt =
    PromptTemplate.fromTemplate(`The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

  Current conversation:
  {chat_history}
  Human: {input}
  AI:`);
  const chain = new LLMChain({ llm: model, prompt, memory });

  const res1 = await chain.call({ input: "Hi! I'm Jim." });
  console.log({ res1, memory: await memory.loadMemoryVariables({}) });
  /*
  {
    res1: {
      text: "Hello Jim! It's nice to meet you. My name is AI. How may I assist you today?"
    },
    memory: {
      chat_history: 'Jim introduces himself to the AI and the AI greets him and offers assistance.'
    }
  }
  */

  const res2 = await chain.call({ input: "What's my name?" });
  console.log({ res2, memory: await memory.loadMemoryVariables({}) });
  /*
  {
    res2: {
      text: "Your name is Jim. It's nice to meet you, Jim. How can I assist you today?"
    },
    memory: {
      chat_history: 'Jim introduces himself to the AI and the AI greets him and offers assistance. The AI addresses Jim by name and asks how it can assist him.'
    }
  }
  */
};
```

### ConversationSummaryBufferMemory（会话关键）
`ConversationSummaryBufferMemory` combines the ideas behind `BufferMemory` and `ConversationSummaryMemory`. It keeps 
recent interactions in memory and summarize older interactions into summary.  It uses token length rather than number 
of interactions to determine when to flush interactions.

```javascript
import { OpenAI } from "langchain/llms/openai";
import { ChatOpenAI } from "langchain/chat_models/openai";
import { ConversationSummaryBufferMemory } from "langchain/memory";
import { ConversationChain } from "langchain/chains";
import {
  ChatPromptTemplate,
  HumanMessagePromptTemplate,
  MessagesPlaceholder,
  SystemMessagePromptTemplate,
} from "langchain/prompts";

// summary buffer memory
const memory = new ConversationSummaryBufferMemory({
  llm: new OpenAI({ modelName: "gpt-3.5-turbo-instruct", temperature: 0 }),
  maxTokenLimit: 10,
});

await memory.saveContext({ input: "hi" }, { output: "whats up" });
await memory.saveContext({ input: "not much you" }, { output: "not much" });
const history = await memory.loadMemoryVariables({});
console.log({ history });
/*
  {
    history: {
      history: 'System: \n' +
        'The human greets the AI, to which the AI responds.\n' +
        'Human: not much you\n' +
        'AI: not much'
    }
  }
*/

// We can also get the history as a list of messages (this is useful if you are using this with a chat prompt).
const chatPromptMemory = new ConversationSummaryBufferMemory({
  llm: new ChatOpenAI({ modelName: "gpt-3.5-turbo", temperature: 0 }),
  maxTokenLimit: 10,
  returnMessages: true,
});
await chatPromptMemory.saveContext({ input: "hi" }, { output: "whats up" });
await chatPromptMemory.saveContext(
  { input: "not much you" },
  { output: "not much" }
);

// We can also utilize the predict_new_summary method directly.
const messages = await chatPromptMemory.chatHistory.getMessages();
const previous_summary = "";
const predictSummary = await chatPromptMemory.predictNewSummary(
  messages,
  previous_summary
);
console.log(JSON.stringify(predictSummary));
// 可以修改 buffer 属性来实现初始化聊天记录
// chatPromptMemory.buffer = "predictSummary"

// Using in a chain
// Let's walk through an example, again setting verbose to true so we can see the prompt.
const chatPrompt = ChatPromptTemplate.fromMessages([
  SystemMessagePromptTemplate.fromTemplate(
    "The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know."
  ),
  new MessagesPlaceholder("history"),
  HumanMessagePromptTemplate.fromTemplate("{input}"),
]);

const model = new ChatOpenAI({ temperature: 0.9, verbose: true });
const chain = new ConversationChain({
  llm: model,
  memory: chatPromptMemory,
  prompt: chatPrompt,
});

const res1 = await chain.predict({ input: "Hi, what's up?" });
console.log({ res1 });
/*
  {
    res1: 'Hello! I am an AI language model, always ready to have a conversation. How can I assist you today?'
  }
*/

const res2 = await chain.predict({
  input: "Just working on writing some documentation!",
});
console.log({ res2 });
/*
  {
    res2: "That sounds productive! Documentation is an important aspect of many projects. Is there anything specific you need assistance with regarding your documentation? I'm here to help!"
  }
*/

const res3 = await chain.predict({
  input: "For LangChain! Have you heard of it?",
});
console.log({ res3 });
/*
  {
    res3: 'Yes, I am familiar with LangChain! It is a blockchain-based language learning platform that aims to connect language learners with native speakers for real-time practice and feedback. It utilizes smart contracts to facilitate secure transactions and incentivize participation. Users can earn tokens by providing language learning services or consuming them for language lessons.'
  }
*/

const res4 = await chain.predict({
  input:
    "That's not the right one, although a lot of people confuse it for that!",
});
console.log({ res4 });

/*
  {
    res4: "I apologize for the confusion! Could you please provide some more information about the LangChain you're referring to? That way, I can better understand and assist you with writing documentation for it."
  }
*/
```

### Vector store-backed memory
`VectorStoreRetrieverMemory` stores memories in a VectorDB and queries the top-K most "salient" docs every time it is called.

This differs from most of the other Memory classes in that it doesn't explicitly track the order of interactions.

In this case, the "docs" are previous conversation snippets. This can be useful to refer to relevant pieces of 
information that the AI was told earlier in the conversation.

// TODO 如果只是忽略顺序，好像没啥区别？

```javascript
import { OpenAI } from "langchain/llms/openai";
import { VectorStoreRetrieverMemory } from "langchain/memory";
import { LLMChain } from "langchain/chains";
import { PromptTemplate } from "langchain/prompts";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";

const vectorStore = new MemoryVectorStore(new OpenAIEmbeddings());
const memory = new VectorStoreRetrieverMemory({
  // 1 is how many documents to return, you might want to return more, eg. 4
  vectorStoreRetriever: vectorStore.asRetriever(1),
  memoryKey: "history",
});

// First let's save some information to memory, as it would happen when
// used inside a chain.
await memory.saveContext(
  { input: "My favorite food is pizza" },
  { output: "thats good to know" }
);
await memory.saveContext(
  { input: "My favorite sport is soccer" },
  { output: "..." }
);
await memory.saveContext({ input: "I don't the Celtics" }, { output: "ok" });

// Now let's use the memory to retrieve the information we saved.
console.log(
  await memory.loadMemoryVariables({ prompt: "what sport should i watch?" })
);
/*
{ history: 'input: My favorite sport is soccer\noutput: ...' }
*/

// Now let's use it in a chain.
const model = new OpenAI({ temperature: 0.9 });
const prompt =
  PromptTemplate.fromTemplate(`The following is a friendly conversation between a human and an AI. The AI is talkative and provides lots of specific details from its context. If the AI does not know the answer to a question, it truthfully says it does not know.

Relevant pieces of previous conversation:
{history}

(You do not need to use these pieces of information if not relevant)

Current conversation:
Human: {input}
AI:`);
const chain = new LLMChain({ llm: model, prompt, memory });

const res1 = await chain.call({ input: "Hi, my name is Perry, what's up?" });
console.log({ res1 });
/*
{
  res1: {
    text: " Hi Perry, I'm doing great! I'm currently exploring different topics related to artificial intelligence like natural language processing and machine learning. What about you? What have you been up to lately?"
  }
}
*/

const res2 = await chain.call({ input: "what's my favorite sport?" });
console.log({ res2 });
/*
{ res2: { text: ' You said your favorite sport is soccer.' } }
*/

const res3 = await chain.call({ input: "what's my name?" });
console.log({ res3 });
/*
{ res3: { text: ' Your name is Perry.' } }
*/
```

## Agents

### Conversational（会话关键）
https://js.langchain.com/docs/modules/agents/agent_types/chat_conversation_agent

### SQL Agent Toolkit（会话关键）
Chains are a sequence of predetermined steps, so they are good to get started with as they give you more control. While 
agent is more complex and powerful, which allows you to use them on larger databases and more complex schemas.

https://js.langchain.com/docs/integrations/toolkits/sql

## Callbacks

## Experimental

## QA and Chat over Documents（会话关键）
Steps to make chat app:
1. Get document from loader. [langchain supported loader](https://js.langchain.com/docs/integrations/document_loaders/)
2. break document into small pieces by splitter
3. Storage(often vector store) house and embed the splits
4. Retrieval: The app retrieves splits from storage (e.g., often with similar embeddings to the input question)
5. output

```javascript
import { CheerioWebBaseLoader } from "langchain/document_loaders/web/cheerio";
import { RecursiveCharacterTextSplitter } from "langchain/text_splitter";
import { OpenAIEmbeddings } from "langchain/embeddings/openai";
import { MemoryVectorStore } from "langchain/vectorstores/memory";
import { RetrievalQAChain } from "langchain/chains";
import { ChatOpenAI } from "langchain/chat_models/openai";

// load documents
const loader = new CheerioWebBaseLoader(
  "https://lilianweng.github.io/posts/2023-06-23-agent/"
);
const data = await loader.load();

// split into chunks
const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 500,
  chunkOverlap: 0,
});

const splitDocs = await textSplitter.splitDocuments(data);

// embed and store the splits
const embeddings = new OpenAIEmbeddings();

const vectorStore = await MemoryVectorStore.fromDocuments(
  splitDocs,
  embeddings
);

// retrieve for any question using similarity_search.
const relevantDocs = await vectorStore.similaritySearch(
  "What is task decomposition?"
);

console.log(relevantDocs.length);

// Distill the retrieved documents into an answer
const model = new ChatOpenAI({ modelName: "gpt-3.5-turbo" });

const template = `Use the following pieces of context to answer the question at the end.
If you don't know the answer, just say that you don't know, don't try to make up an answer.
Use three sentences maximum and keep the answer as concise as possible.
Always say "thanks for asking!" at the end of the answer.
{context}
Question: {question}
Helpful Answer:`;

// the third arguments is optional
const chain = RetrievalQAChain.fromLLM(model, vectorStore.asRetriever(), {
  prompt: PromptTemplate.fromTemplate(template),
  // get all retrieved documents used for answer distillation
  returnSourceDocuments: true,
});

const response = await chain.call({
  query: "What is task decomposition?",
});
console.log(response);

// all retrieved documents
console.log(response.sourceDocuments[0]);
```

Retrieved documents can be fed to an LLM for answer distillation in a few different ways.

`stuff`, `refine`, and `map-reduce` chains for passing documents to an LLM prompt are well summarized [here](https://js.langchain.com/docs/modules/chains/document/).

`stuff` is commonly used because it simply "stuffs" all retrieved documents into the prompt.

The `loadQAChain` methods are easy ways to pass documents to an LLM using these various approaches.

```javascript
import { loadQAStuffChain } from "langchain/chains";

// Distill the retrieved documents into an answer using loadQAChain
const stuffChain = loadQAStuffChain(model);

const stuffResult = await stuffChain.call({
  input_documents: relevantDocs,
  question: "What is task decomposition?",
});
```

To keep chat history, we use a variant of the previous chain called a `ConversationalRetrievalQAChain`.

First, specify a `Memory buffer` to track the conversation inputs / outputs.

```javascript
import { ConversationalRetrievalQAChain } from "langchain/chains";
import { BufferMemory } from "langchain/memory";
import { ChatOpenAI } from "langchain/chat_models/openai";

const memory = new BufferMemory({
  memoryKey: "chat_history",
  returnMessages: true,
});

const model = new ChatOpenAI({ modelName: "gpt-3.5-turbo" });
const chain = ConversationalRetrievalQAChain.fromLLM(
  model,
  vectorStore.asRetriever(),
  {
    memory,
  }
);

const result = await chain.call({
  question: "What are some of the main ideas in self-reflection?",
});
console.log(result);

```

Azure Blob Storage Container
https://js.langchain.com/docs/integrations/document_loaders/web_loaders/azure_blob_storage_container
