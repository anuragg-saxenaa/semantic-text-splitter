# Semantic Text Splitter — Spring AI

A native semantic text chunker for [Spring AI](https://github.com/spring-projects/spring-ai) that uses embedding-based similarity detection to split text at natural semantic boundaries.

## Problem

Spring AI's built-in `TokenTextSplitter` breaks text at fixed token boundaries, which can split semantically related content across chunk boundaries. This degrades RAG retrieval quality.

## Solution

`SemanticTextSplitter` extends `TextSplitter` and uses an `EmbeddingModel` to:

1. Split text into sentences using punctuation
2. Compute embeddings for each sentence (batch call)
3. Calculate cosine similarity between consecutive sentence embeddings
4. Split where similarity drops below a configurable threshold
5. Respect `maxChunkSize` by forcing a new chunk when exceeded

## Usage

```java
SemanticTextSplitter splitter = SemanticTextSplitter.builder()
    .embeddingModel(embeddingModel)
    .similarityThreshold(0.5)  // default
    .maxChunkSize(1000)       // default chars
    .build();

List<Document> chunks = splitter.split(document);
```

Or with defaults (threshold=0.5, maxChunkSize=1000):

```java
SemanticTextSplitter splitter = new SemanticTextSplitter(embeddingModel);
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `embeddingModel` | `EmbeddingModel` | — | Required. Used to compute sentence embeddings. |
| `similarityThreshold` | `double` | `0.5` | Cosine similarity threshold [0,1]. Split occurs when similarity **drops below** this value. |
| `maxChunkSize` | `int` | `1000` | Maximum chunk size in characters. Forces a new chunk when exceeded. |

## Algorithm

```
sentences = splitIntoSentences(text)
embeddings = embeddingModel.embed(sentences)  // batch call

for each sentence i:
    add sentence to current chunk
    if length > maxChunkSize: flush chunk, start new
    if i < len-1 and cosineSimilarity(embeddings[i], embeddings[i+1]) < threshold:
        flush chunk, start new
```

## Testing

```bash
./mvnw test
```

Tests cover: normal case, single sentence, null/blank input, threshold boundaries, mixed thresholds, max chunk size enforcement, sentence splitting, cosine similarity edge cases, and document metadata preservation.

## Module

This module extends `spring-ai-commons` and `spring-ai-model` — no new external dependencies beyond what Spring AI already provides.

## Related

- Issue: [spring-projects/spring-ai#5464](https://github.com/spring-projects/spring-ai/issues/5464)
- `TokenTextSplitter`: Spring AI's fixed-token-count splitter (complementary)
- 
