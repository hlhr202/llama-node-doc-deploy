# llama-node

Large Language Model LLaMA on node.js

This project is in an early stage, the API for nodejs may change in the future, use it with caution.

[中文文档](./README-zh-CN.md)

<img src="./doc/assets/llama.png" width="300px" height="300px" alt="LLaMA generated by Stable diffusion"/>

<sub>Picture generated by stable diffusion.</sub>


![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/hlhr202/llama-node/llama-build.yml)
![NPM](https://img.shields.io/npm/l/llama-node)
[<img alt="npm" src="https://img.shields.io/npm/v/llama-node">](https://www.npmjs.com/package/llama-node)
![npm type definitions](https://img.shields.io/npm/types/llama-node)
[<img alt="twitter" src="https://img.shields.io/twitter/url?url=https%3A%2F%2Ftwitter.com%2Fhlhr202">](https://twitter.com/hlhr202)

---

- [llama-node](#llama-node)
  - [Introduction](#introduction)
  - [Install](#install)
  - [Getting the weights](#getting-the-weights)
    - [Model versioning](#model-versioning)
  - [Usage (llama.cpp backend)](#usage-llamacpp-backend)
    - [Inference](#inference)
    - [Tokenize](#tokenize)
    - [Embedding](#embedding)
  - [Usage (llama-rs backend)](#usage-llama-rs-backend)
    - [Inference](#inference-1)
    - [Tokenize](#tokenize-1)
    - [Embedding](#embedding-1)
  - [Performance related](#performance-related)
    - [Manual compilation (from node\_modules)](#manual-compilation-from-node_modules)
    - [Manual compilation (from source)](#manual-compilation-from-source)
  - [Future plan](#future-plan)

---

## Introduction

This is a nodejs client library for llama (or llama based) LLM built on top of [llama-rs](https://github.com/rustformers/llama-rs) and [llm-chain-llama-sys](https://github.com/sobelio/llm-chain/tree/main/llm-chain-llama/sys) which generate bindings for [llama.cpp](https://github.com/ggerganov/llama.cpp). It uses [napi-rs](https://github.com/napi-rs/napi-rs) for channel messages between node.js and llama thread.

From v0.0.21, both llama-rs and llama.cpp backends are supported!

Currently supported platforms:
- darwin-x64
- darwin-arm64
- linux-x64-gnu
- linux-x64-musl
- win32-x64-msvc


I do not have hardware for testing 13B or larger models, but I have tested it supported llama 7B model with both ggml llama and ggml alpaca.

<!-- Download one of the llama ggml models from the following links:
- [llama 7B int4 (old model for llama.cpp)](https://huggingface.co/hlhr202/llama-7B-ggml-int4/blob/main/ggml-model-q4_0.bin)
- [alpaca 7B int4](https://huggingface.co/hlhr202/alpaca-7B-ggml-int4/blob/main/ggml-alpaca-7b-q4.bin) -->

---

## Install

- Install main package
```bash
npm install llama-node
```

- Install llama-rs backend
```bash
npm install @llama-node/core
```

- Install llama.cpp backend
```bash
npm install @llama-node/llama-cpp
```

---

## Getting the weights

The llama-node uses llama-rs under the hook and uses the model format derived from llama.cpp. Due to the fact that the meta-release model is only used for research purposes, this project does not provide model downloads. If you have obtained the original **.pth** model, please read the document [Getting the weights](https://github.com/rustformers/llama-rs#getting-the-weights) and use the convert tool provided by llama-rs for conversion.

### Model versioning

There are now 3 versions from llama.cpp community:

- GGML: legacy format, oldest ggml tensor file format
- GGMF: also legacy format, newer than GGML, older than GGJT
- GGJT: mmap-able format

The llama-rs backend now only supports GGML/GGMF models, and llama.cpp backend only supports GGJT models.

---

## Usage (llama.cpp backend)

The current version supports only one inference session on one LLama instance at the same time

If you wish to have multiple inference sessions concurrently, you need to create multiple LLama instances

### Inference

```typescript
import { LLama } from "llama-node";
import { LLamaCpp, LoadConfig } from "llama-node/dist/llm/llama-cpp.js";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-vicuna-7b-4bit-rev1.bin");

const llama = new LLama(LLamaCpp);

const config: LoadConfig = {
    path: model,
    enableLogging: true,
    nCtx: 1024,
    nParts: -1,
    seed: 0,
    f16Kv: false,
    logitsAll: false,
    vocabOnly: false,
    useMlock: false,
    embedding: false,
};

llama.load(config);

const template = `How are you`;

const prompt = `### Human:

${template}

### Assistant:`;

llama.createCompletion(
    {
        nThreads: 4,
        nTokPredict: 2048,
        topK: 40,
        topP: 0.1,
        temp: 0.2,
        repeatPenalty: 1,
        stopSequence: "### Human",
        prompt,
    },
    (response) => {
        process.stdout.write(response.token);
    }
);

```

### Tokenize

```typescript
import { LLama } from "llama-node";
import { LLamaCpp, LoadConfig } from "llama-node/dist/llm/llama-cpp.js";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-vicuna-7b-4bit-rev1.bin");

const llama = new LLama(LLamaCpp);

const config: LoadConfig = {
    path: model,
    enableLogging: true,
    nCtx: 1024,
    nParts: -1,
    seed: 0,
    f16Kv: false,
    logitsAll: false,
    vocabOnly: false,
    useMlock: false,
    embedding: false,
};

llama.load(config);

const content = "how are you?";

llama.tokenize({ content, nCtx: 2048 }).then(console.log);

```

### Embedding
```typescript
import { LLama } from "llama-node";
import { LLamaCpp, LoadConfig } from "llama-node/dist/llm/llama-cpp.js";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-vicuna-7b-4bit-rev1.bin");

const llama = new LLama(LLamaCpp);

const config: LoadConfig = {
    path: model,
    enableLogging: true,
    nCtx: 1024,
    nParts: -1,
    seed: 0,
    f16Kv: false,
    logitsAll: false,
    vocabOnly: false,
    useMlock: false,
    embedding: false,
};

llama.load(config);

const prompt = `Who is the president of the United States?`;

const params = {
    nThreads: 4,
    nTokPredict: 2048,
    topK: 40,
    topP: 0.1,
    temp: 0.2,
    repeatPenalty: 1,
    prompt,
};

llama.getEmbedding(params).then(console.log);

```

---

## Usage (llama-rs backend)

The current version supports only one inference session on one LLama instance at the same time

If you wish to have multiple inference sessions concurrently, you need to create multiple LLama instances

### Inference

```typescript
import { LLama } from "llama-node";
import { LLamaRS } from "llama-node/dist/llm/llama-rs.js";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLama(LLamaRS);

llama.load({ path: model });

const template = `how are you`;

const prompt = `Below is an instruction that describes a task. Write a response that appropriately completes the request.

### Instruction:

${template}

### Response:`;

llama.createCompletion(
    {
        prompt,
        numPredict: 128,
        temp: 0.2,
        topP: 1,
        topK: 40,
        repeatPenalty: 1,
        repeatLastN: 64,
        seed: 0,
        feedPrompt: true,
    },
    (response) => {
        process.stdout.write(response.token);
    }
);
```

### Tokenize

Get tokenization result from LLaMA

```typescript
import { LLama } from "llama-node";
import { LLamaRS } from "llama-node/dist/llm/llama-rs.js";
import path from "path";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLama(LLamaRS);

llama.load({ path: model });

const content = "how are you?";

llama.tokenize(content).then(console.log);

```

### Embedding

Preview version, embedding end token may change in the future. Do not use it in production!

```typescript
import { LLama } from "llama-node";
import { LLamaRS } from "llama-node/dist/llm/llama-rs.js";
import path from "path";
import fs from "fs";

const model = path.resolve(process.cwd(), "./ggml-alpaca-7b-q4.bin");

const llama = new LLama(LLamaRS);

llama.load({ path: model });

const getWordEmbeddings = async (prompt: string, file: string) => {
    const data = await llama.getEmbedding({
        prompt,
        numPredict: 128,
        temp: 0.2,
        topP: 1,
        topK: 40,
        repeatPenalty: 1,
        repeatLastN: 64,
        seed: 0,
    });

    console.log(prompt, data);

    await fs.promises.writeFile(
        path.resolve(process.cwd(), file),
        JSON.stringify(data)
    );
};

const run = async () => {
    const dog1 = `My favourite animal is the dog`;
    await getWordEmbeddings(dog1, "./example/semantic-compare/dog1.json");

    const dog2 = `I have just adopted a cute dog`;
    await getWordEmbeddings(dog2, "./example/semantic-compare/dog2.json");

    const cat1 = `My favourite animal is the cat`;
    await getWordEmbeddings(cat1, "./example/semantic-compare/cat1.json");
};

run();
```

---

## Performance related

We provide prebuild binaries for linux-x64, win32-x64, apple-x64, apple-silicon. For other platforms, before you install the npm package, please install rust environment for self built.

Due to complexity of cross compilation, it is hard for pre-building a binary that fits all platform needs with best performance.

If you face low performance issue, I would strongly suggest you do a manual compilation. Otherwise you have to wait for a better pre-compiled native binding. I am trying to investigate the way to produce a matrix of multi-platform supports.

### Manual compilation (from node_modules)

The following steps will allow you to compile the binary with best quality on your platform

- Pre-request: install rust

- Under node_modules/@llama-node/core folder

    ```shell
    npm run build
    ```

### Manual compilation (from source)

The following steps will allow you to compile the binary with best quality on your platform

- Pre-request: install rust

- Under root folder, run

    ```shell
    npm install && npm run build
    ```

- Under packages/core folder, run
    ```shell
    npm run build
    ```

- You can use the dist under root folder

---

## Future plan
- [ ] prompt extensions
- [ ] more platforms and cross compile (performance related)
- [ ] tweak embedding API, make end token configurable
- [ ] cli and interactive
- [ ] support more open source models as llama-rs planned https://github.com/rustformers/llama-rs/pull/85 https://github.com/rustformers/llama-rs/issues/75
- [ ] more backends (eg. rwkv) supports!