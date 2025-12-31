# Dynamic Chunking Strategy for RFC Documents
## Chief Scientist Design Document

### Executive Summary

This document presents a novel dynamic chunking strategy specifically designed for RFC (Request for Comments) documents. Unlike fixed-size chunking, our approach adaptively determines optimal chunk size and overlap based on content type, semantic density, and structural characteristics of each RFC section.

---

## 1. RFC Content Type Classification

### 1.1 Content Categories

We identify 7 primary content types in RFC documents:

| Content Type | Characteristics | Optimal Chunk Size | Optimal Overlap |
|-------------|----------------|-------------------|-----------------|
| **Protocol Specification** | Dense technical definitions, formal syntax, state machines | 1200-1800 tokens | 400-600 tokens (30-35%) |
| **Message Formats** | Packet structures, field definitions, bit layouts | 800-1200 tokens | 300-400 tokens (30-35%) |
| **Examples** | Code snippets, protocol exchanges, use cases | 200-400 tokens | 80-120 tokens (30-40%) |
| **Terminology/Definitions** | Glossary entries, term definitions | 300-500 tokens | 100-150 tokens (30-35%) |
| **Introduction/Overview** | High-level descriptions, background | 400-600 tokens | 120-180 tokens (30%) |
| **Procedures/Algorithms** | Step-by-step processes, state transitions | 600-1000 tokens | 200-300 tokens (30-35%) |
| **Appendix/References** | Bibliographic info, supplementary material | 500-800 tokens | 150-200 tokens (25-30%) |

### 1.2 Content Type Detection Features

#### Feature Set F = {f₁, f₂, ..., fₙ}

1. **Lexical Density (f₁)**
   - Ratio of technical terms to total words
   - Technical terms: protocol names, field names, acronyms
   - Formula: `density = (technical_terms / total_words) * 100`
   - Threshold: >15% → Protocol Spec; <5% → Introduction

2. **Structural Markers (f₂)**
   - Presence of specific patterns:
     - `+---+` or ASCII art → Message Format
     - `MUST/SHOULD/MAY` keywords → Protocol Spec
     - Code blocks (indented, monospace) → Examples
     - Numbered lists → Procedures
     - `"term": "definition"` → Terminology

3. **Section Depth (f₃)**
   - Hierarchical level: `section_id.count('.')`
   - Deeper sections (3.2.1) → More specific → Smaller chunks
   - Top-level sections (1, 2) → Broader context → Larger chunks

4. **Content Length (f₄)**
   - Section length in tokens
   - Very short (<200 tokens) → Examples/Terminology
   - Very long (>2000 tokens) → Protocol Spec

5. **Code/Format Density (f₅)**
   - Ratio of code blocks, tables, diagrams to text
   - High density → Message Format/Examples
   - Low density → Narrative text

6. **Reference Density (f₆)**
   - Number of RFC citations per 100 words
   - High density → Protocol Spec (builds on other RFCs)
   - Low density → Introduction/Overview

7. **Formal Language Indicators (f₇)**
   - Presence of formal syntax (BNF, ABNF, state diagrams)
   - Strong indicator → Protocol Spec → Large chunks

---

## 2. Dynamic Chunk Size Algorithm

### 2.1 Base Chunk Size Calculation

For each section `s` with features `F(s) = {f₁, f₂, ..., f₇}`:

```
base_chunk_size(s) = α × content_type_base_size(type(s)) + 
                     β × length_factor(s) + 
                     γ × depth_factor(s)
```

Where:
- `type(s)` = classified content type (from Section 1.1)
- `content_type_base_size(type)` = optimal size from table above
- `length_factor(s)` = adjustment based on section length
- `depth_factor(s)` = adjustment based on section depth
- `α, β, γ` = weighting coefficients (α=0.6, β=0.25, γ=0.15)

### 2.2 Length Factor

```
length_factor(s) = {
    if length(s) < 500 tokens:  -200  (prefer smaller chunks)
    if length(s) < 1500 tokens:   0   (neutral)
    if length(s) < 3000 tokens:  +300  (allow larger chunks)
    else:                        +500  (very long sections)
}
```

### 2.3 Depth Factor

```
depth_factor(s) = {
    if depth(s) == 1:        +200  (top-level, need context)
    if depth(s) == 2:        +100  (second level)
    if depth(s) == 3:         0   (neutral)
    if depth(s) >= 4:        -150  (deep nesting, smaller chunks)
}
```

### 2.4 Final Chunk Size with Constraints

```
chunk_size(s) = clamp(
    base_chunk_size(s),
    min=100,   # OpenAI minimum
    max=4096   # OpenAI maximum
)
```

---

## 3. Dynamic Overlap Size Algorithm

### 3.1 Base Overlap Calculation

Overlap is calculated as a percentage of chunk size, adjusted by content characteristics:

```
base_overlap_ratio(s) = base_overlap_ratio(type(s)) × 
                        continuity_factor(s) × 
                        boundary_factor(s)
```

Where:
- `base_overlap_ratio(type)` = from table in Section 1.1 (e.g., 0.30-0.35)
- `continuity_factor(s)` = adjustment for semantic continuity needs
- `boundary_factor(s)` = adjustment for section boundary importance

### 3.2 Continuity Factor

Determines how much overlap is needed to maintain semantic coherence:

```
continuity_factor(s) = {
    if has_formal_syntax(s):     1.2  (formal specs need more overlap)
    if has_state_machine(s):      1.3  (state transitions critical)
    if is_procedure(s):           1.1  (step-by-step needs continuity)
    if is_example(s):             0.8  (examples are self-contained)
    if is_terminology(s):         0.7  (definitions are independent)
    else:                         1.0  (default)
}
```

### 3.3 Boundary Factor

Adjusts overlap based on section boundaries:

```
boundary_factor(s) = {
    if crosses_major_section(s):  1.2  (preserve section context)
    if crosses_subsection(s):      1.1  (preserve subsection context)
    else:                          1.0  (within same section)
}
```

### 3.4 Final Overlap Size

```
overlap_size(s) = clamp(
    chunk_size(s) × base_overlap_ratio(s),
    min=0,
    max=chunk_size(s) / 2  # OpenAI constraint
)
```

---

## 4. Content Type Classification Algorithm

### 4.1 Multi-Feature Classifier

```python
def classify_content_type(section):
    """
    Classifies a section into one of 7 content types.
    Returns: content_type, confidence_score
    """
    features = extract_features(section)
    scores = {}
    
    # Protocol Specification
    scores['protocol_spec'] = (
        0.3 * lexical_density_score(features) +
        0.25 * formal_language_score(features) +
        0.2 * reference_density_score(features) +
        0.15 * length_score(features, min=1000) +
        0.1 * structural_marker_score(features, ['MUST', 'SHOULD', 'state'])
    )
    
    # Message Formats
    scores['message_format'] = (
        0.4 * structural_marker_score(features, ['+---+', 'bit', 'field', 'octet']) +
        0.3 * code_density_score(features) +
        0.2 * table_diagram_score(features) +
        0.1 * lexical_density_score(features)
    )
    
    # Examples
    scores['examples'] = (
        0.5 * code_block_score(features) +
        0.3 * indentation_density_score(features) +
        0.2 * length_score(features, max=500)
    )
    
    # Terminology/Definitions
    scores['terminology'] = (
        0.4 * definition_pattern_score(features) +
        0.3 * glossary_marker_score(features) +
        0.2 * length_score(features, max=500) +
        0.1 * reference_density_score(features, low=True)
    )
    
    # Introduction/Overview
    scores['introduction'] = (
        0.4 * lexical_density_score(features, low=True) +
        0.3 * section_depth_score(features, shallow=True) +
        0.2 * reference_density_score(features, low=True) +
        0.1 * narrative_text_score(features)
    )
    
    # Procedures/Algorithms
    scores['procedures'] = (
        0.4 * numbered_list_score(features) +
        0.3 * step_marker_score(features) +
        0.2 * state_transition_score(features) +
        0.1 * length_score(features, min=400, max=1500)
    )
    
    # Appendix/References
    scores['appendix'] = (
        0.5 * section_title_score(features, ['Appendix', 'Reference']) +
        0.3 * bibliography_pattern_score(features) +
        0.2 * reference_density_score(features, high=True)
    )
    
    # Select best match
    best_type = max(scores, key=scores.get)
    confidence = scores[best_type]
    
    # Fallback to default if confidence too low
    if confidence < 0.4:
        best_type = 'introduction'  # Safe default
    
    return best_type, confidence
```

### 4.2 Feature Extraction Functions

```python
def extract_features(section):
    """Extracts all features from a section."""
    return {
        'text': section['content'],
        'title': section['title'],
        'identifier': section['identifier'],
        'depth': section['identifier'].count('.'),
        'length_tokens': estimate_tokens(section['content']),
        'has_code_blocks': detect_code_blocks(section['content']),
        'has_tables': detect_tables(section['content']),
        'has_ascii_art': detect_ascii_art(section['content']),
        'technical_terms': extract_technical_terms(section['content']),
        'rfc_references': extract_rfc_references(section['content']),
        'formal_syntax': detect_formal_syntax(section['content']),
        'numbered_lists': detect_numbered_lists(section['content']),
        'must_should_may': count_requirements(section['content']),
    }
```

---

## 5. Implementation Strategy

### 5.1 Per-Section Chunking

For each section in the RFC:

1. **Extract features** from section content
2. **Classify content type** using multi-feature classifier
3. **Calculate optimal chunk_size** using dynamic algorithm
4. **Calculate optimal overlap_size** using dynamic algorithm
5. **Apply chunking** with calculated parameters

### 5.2 Section Boundary Preservation

Critical constraint: **Never split chunks across major section boundaries**

- Major sections: Top-level (1, 2, 3, ...)
- Minor sections: Subsections (1.1, 1.2, ...)
- Strategy: If a chunk would cross a major boundary, end it at the boundary and start a new chunk

### 5.3 Chunking Execution

```python
def dynamic_chunk_section(section, chunk_size, overlap_size):
    """
    Chunks a section with dynamic size and overlap.
    Preserves semantic boundaries.
    """
    chunks = []
    content = section['content']
    current_pos = 0
    
    while current_pos < len(content):
        # Calculate chunk end position
        chunk_end = min(current_pos + chunk_size, len(content))
        
        # Check for semantic boundaries (sentence, paragraph)
        chunk_end = adjust_to_boundary(content, chunk_end)
        
        # Extract chunk
        chunk = content[current_pos:chunk_end]
        chunks.append({
            'text': chunk,
            'section_id': section['identifier'],
            'chunk_size': len(chunk),
            'overlap_size': overlap_size if current_pos > 0 else 0
        })
        
        # Move to next chunk (with overlap)
        current_pos = chunk_end - overlap_size
    
    return chunks
```

---

## 6. Optimization Heuristics

### 6.1 Adaptive Refinement

After initial chunking, apply refinement rules:

1. **Merge tiny chunks**: If chunk < 50 tokens and next chunk < 200 tokens, merge them
2. **Split oversized chunks**: If chunk > max_size, split at paragraph boundaries
3. **Balance chunk sizes**: If adjacent chunks differ by >50%, adjust boundaries

### 6.2 Context Window Optimization

For sections that are frequently referenced together:

- Increase overlap between related sections
- Use larger chunks for sections with high cross-references
- Maintain semantic coherence across section boundaries

---

## 7. Evaluation Metrics

### 7.1 Retrieval Quality Metrics

- **Precision@K**: Relevance of top-K retrieved chunks
- **Recall@K**: Coverage of relevant information
- **MRR (Mean Reciprocal Rank)**: Average rank of first relevant chunk
- **Semantic Coherence**: Measure of chunk boundary quality

### 7.2 Efficiency Metrics

- **Average chunk size**: Should match content type distribution
- **Overlap ratio**: Should optimize for continuity without waste
- **Chunk count**: Fewer chunks = lower storage cost
- **Query latency**: Time to retrieve relevant chunks

### 7.3 Cost Metrics

- **Storage cost**: Total tokens stored (including overlaps)
- **Embedding cost**: Number of chunks to embed
- **Query cost**: Tokens processed per query

---

## 8. Expected Performance Improvements

Based on theoretical analysis and similar work:

| Metric | Improvement | Rationale |
|--------|------------|------------|
| **Retrieval Precision** | +25-40% | Optimal chunk sizes for each content type |
| **Context Preservation** | +30-50% | Dynamic overlap maintains semantic continuity |
| **Storage Efficiency** | +15-25% | Reduced redundant chunks, better boundaries |
| **Query Latency** | -10-20% | Fewer irrelevant chunks to process |
| **Cost Reduction** | -20-30% | Optimized chunk sizes reduce token waste |

---

## 9. Implementation Roadmap

### Phase 1: Core Algorithm (2-3 weeks)
- [ ] Feature extraction functions
- [ ] Content type classifier
- [ ] Dynamic chunk size calculator
- [ ] Dynamic overlap calculator

### Phase 2: Integration (1-2 weeks)
- [ ] Integrate with existing RFC processing pipeline
- [ ] Update vector store upload functions
- [ ] Add chunking strategy parameter to API calls

### Phase 3: Evaluation (2-3 weeks)
- [ ] Benchmark on 50+ RFC documents
- [ ] Compare with fixed-size chunking
- [ ] Measure retrieval quality improvements
- [ ] Analyze cost/performance trade-offs

### Phase 4: Optimization (1-2 weeks)
- [ ] Tune feature weights based on evaluation
- [ ] Implement adaptive refinement
- [ ] Add caching for feature extraction

---

## 10. Future Enhancements

1. **Learning-based Classification**: Train ML model on labeled RFC sections
2. **Query-Adaptive Chunking**: Adjust chunk sizes based on query patterns
3. **Cross-RFC Optimization**: Optimize chunking for frequently co-referenced RFCs
4. **Real-time Adaptation**: Adjust chunking based on retrieval feedback

---

## Appendix: Feature Extraction Details

### A.1 Technical Term Detection

```python
def extract_technical_terms(text):
    """Extracts technical terms using patterns."""
    patterns = [
        r'\b[A-Z]{2,}\b',  # Acronyms (TCP, HTTP, TLS)
        r'\b[A-Z][a-z]+-[A-Z][a-z]+\b',  # Hyphenated terms
        r'\b[a-z]+-[a-z]+(-[a-z]+)*\b',  # Kebab-case terms
    ]
    # ... implementation
```

### A.2 Formal Syntax Detection

```python
def detect_formal_syntax(text):
    """Detects BNF, ABNF, state diagrams."""
    patterns = [
        r'::=',  # BNF production
        r'<[A-Z][a-z]+>',  # Non-terminals
        r'\[.*?\]',  # Optional elements
        r'\{.*?\}',  # Repetition
    ]
    # ... implementation
```

---

**Document Version**: 1.0  
**Author**: Chief Scientist  
**Date**: 2024  
**Status**: Design Phase

