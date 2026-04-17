# SemanticTextSplitter — Spring AI

## Problem
Spring AI has no native semantic text chunking — only TokenTextSplitter which breaks semantic boundaries at fixed token counts, degrading RAG retrieval quality.

## Solution
Implement `SemanticTextSplitter` extending Spring AI's `TextSplitter` base class.

## Implementation

### Algorithm
1. Split text into sentences using a simple sentence boundary detector
2. Compute embeddings for each sentence using `EmbeddingModel`
3. Compute cosine similarity between consecutive sentence embeddings
4. Split when similarity drops below `similarityThreshold`
5. Respect `maxChunkSize` by starting a new chunk when exceeded

### API
```
SemanticTextSplitter(
    EmbeddingModel embeddingModel,
    double similarityThreshold = 0.5,
    int maxChunkSize = 1000  // chars
)
```

### Output
- Returns `List<TextChunk>` — each chunk has `String content` and `List<Integer> sentenceIndices`

### Tests
- Normal case: paragraph splits into semantic chunks
- Single sentence: returns as single chunk
- Empty list: returns empty list
- Threshold boundaries: splitting at exact threshold
- Mixed: small threshold (many chunks) vs large threshold (few chunks)
