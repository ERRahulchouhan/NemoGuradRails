# Pipeline Flow

```mermaid
flowchart TD
    A[User message] --> B[run_pipeline]
    B --> C[NeMo input rails<br/>ChatGroq guard model]
    C --> D[parse_nemo_response]
    D --> E{Blocked or dialog?}
    E -- Yes --> F[Return rail response and trace]
    E -- No --> G[FAISS retrieval<br/>top 3 chunks]
    G --> H[Build context from chunks]
    H --> I[Answer generation<br/>ChatGroq chat model]
    I --> J[check_output<br/>regex sensitive-pattern scan]
    J --> K{Sensitive pattern found?}
    K -- Yes --> L[Withhold answer<br/>return contact message]
    K -- No --> M[Keep generated answer]
    L --> N[Return reply and trace]
    M --> N

    G -. Retrieval error .-> O[Use no chunks]
    O --> H
    I -. Generation error .-> P[Use LLM failure message]
    P --> J
```

## Notes

- The input guard uses NeMo rails. A blocked request or a dialog response returns before retrieval and answer generation.
- If retrieval fails, the pipeline continues with the fallback context: "No relevant policy excerpts were retrieved."
- If answer generation fails, the pipeline checks the failure message with the output guard like any other answer.
- The output guard is the regex-based `check_output()` scan against `SENSITIVE_OUTPUT_PATTERNS`; matches cause the generated response to be withheld.
- `needs_rag` is parsed from the NeMo response but does not currently control whether retrieval runs.