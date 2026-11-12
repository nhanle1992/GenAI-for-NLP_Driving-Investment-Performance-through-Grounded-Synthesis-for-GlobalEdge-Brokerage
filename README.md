# Financial Intelligence RAG for GlobalEdge Brokerage

A proof-of-concept Retrieval-Augmented Generation (RAG) assistant that helps brokerage professionals turn financial news, market prices, and SEC filings into grounded, evidence-backed answers.

## Project overview

GlobalEdge Brokerage's brokers monitor equities, foreign exchange, commodities, crypto assets, and international markets while managing large client portfolios. The volume and variety of available information make it difficult to identify relevant developments quickly and support recommendations with traceable evidence.

This project builds a searchable financial intelligence layer that lets a broker ask questions in plain English. The system retrieves relevant passages from multiple financial datasets and uses a large language model to synthesize a cited response, surface material risks, and highlight implications for a broker.

## Business objectives

- Reduce the time brokers spend searching across disconnected sources.
- Improve market coverage without increasing workflow complexity.
- Generate grounded answers with source citations rather than unsupported claims.
- Support faster, more consistent investment research and client conversations.
- Identify where the available evidence is insufficient instead of inventing an answer.

## Data sources

The retrieval corpus combines:

- Global financial news articles
- Historical stock and crypto price observations
- SEC filing text
- A 20-question golden benchmark used for development evaluation

The current saved run uses the provided SEC **10-K fallback** file. The assignment called for 10-Q filings; this is documented as a limitation below.

## RAG architecture

```text
Financial news + price data + SEC filing
                  |
        clean, enrich, and chunk
                  |
   OpenAI text-embedding-3-small
                  |
        local Chroma vector store
                  |
      similarity/MMR retrieval
                  |
       optimized broker prompt
                  |
          GPT-4o mini answer
                  |
 citations + risks + broker implications
```

### Core design

- **Vector database:** Chroma with local persistence
- **Embedding model:** `text-embedding-3-small`
- **SEC text splitting:** recursive character splitting with 1,200-character chunks and 200-character overlap
- **Answer model:** `gpt-4o-mini`
- **Prompt strategy:** grounded financial synthesis with explicit source references, risk identification, and broker-oriented implications
- **Evaluation metrics:** Answer Relevance, Broker Actionability, Faithfulness, and Groundedness

The chunk size was chosen to retain enough filing context for financial interpretation, while overlap reduces the risk of losing meaning at section boundaries. Structured metadata makes it possible to trace retrieved evidence back to its dataset, ticker, date, article, or filing type.

## Configuration experiments

Four development configurations were compared using a fixed index and prompt:

| Configuration | Retrieval | Top-k | Temperature | Max tokens | Mean score | Pass rate | Mean latency |
|---|---:|---:|---:|---:|---:|---:|---:|
| `similarity_k5` | Similarity | 5 | 0.1 | 800 | 0.705 | 62.5% | 3.59 s |
| `similarity_k3` | Similarity | 3 | 0.0 | 600 | 0.692 | 60.0% | 3.71 s |
| `mmr_k5` | MMR | 5 | 0.1 | 800 | 0.710 | 65.0% | 3.97 s |
| `similarity_k8` | Similarity | 8 | 0.2 | 1,000 | 0.680 | 58.8% | 3.80 s |

The saved selection policy chose **`similarity_k5`** because it offered the strongest combined quality-and-latency utility. Although `mmr_k5` achieved a slightly higher raw mean score, it had higher latency and a lower utility-adjusted selection score.

### Selected configuration results

| Metric | Mean score | Pass rate |
|---|---:|---:|
| Answer Relevance | 0.731 | 70% |
| Broker Actionability | 0.592 | 45% |
| Faithfulness | 0.794 | 80% |
| Groundedness | 0.704 | 55% |

The selected configuration produced an overall mean score of **0.705**, an overall pass rate of **62.5%**, and an average response latency of **3.59 seconds** on the 20-example development benchmark. Faithfulness was the strongest metric; broker actionability remained the primary improvement opportunity.

## Final use cases

The final pipeline was demonstrated on five broker-style questions covering:

1. Cross-company revenue, income, and growth comparisons
2. Amazon news sentiment and price confirmation
3. Apple leadership-change, performance, and risk analysis
4. Bitcoin price trends and rally drivers
5. Comparative AI investment analysis for Alphabet, Amazon, and Apple

The generated answers and their retrieved source metadata are available in [`final_rag_test_results.csv`](final_rag_test_results.csv) and [`final_rag_test_results.json`](final_rag_test_results.json).

## Evaluation result

The submitted project received **55/58 points (94.8%)**.

| Rubric area | Score |
|---|---:|
| Data preparation for RAG | 9/10 |
| Question answering with RAG | 9/10 |
| Evaluating RAG | 7/8 |
| Question answering with tuned RAG | 18/20 |
| Actionable insights and recommendations | 4/4 |
| Presentation and notebook quality | 8/8 |

Evaluator feedback highlighted the working end-to-end preparation pipeline, systematic configuration comparison, clear notebook structure, heavily commented code, and business recommendations tied to broker workflows, coverage, latency, and evidence quality.

## Known limitations and next steps

The evaluation also identified several improvements needed before broader deployment:

1. **Use the required SEC 10-Q data.** Replace the development 10-K fallback, rebuild the Chroma index, and rerun retrieval and generation.
2. **Align evaluation with the required test set.** Evaluate the baseline and each tuned configuration on all five provided cases, rather than relying primarily on the 20-example development benchmark.
3. **Compare exactly three tuned configurations.** Remove the extra fourth configuration from the assignment-facing comparison.
4. **Add per-answer analysis.** Document concise observations about factual coverage, retrieval quality, limitations, and business usefulness for every test case.
5. **Strengthen model rationale.** Explain why the embedding model, answer model, decoding parameters, and evaluation thresholds fit financial retrieval.
6. **Improve broker actionability.** Refine retrieval and prompting so answers more consistently connect evidence to decisions, risks, and next actions.

## Repository structure

```text
.
├── Financial_Dataset/                 # News, prices, benchmark, and SEC text
├── UTAIGA_Project_2_Full_Code_Notebook.ipynb
├── UTAIGA_Project_2_Full_Code_Notebook.html
├── baseline_evaluation_*.csv          # Baseline metric outputs
├── optimized_evaluation_*.csv         # Optimized prompt evaluation
├── rag_tuning_results/                # Configuration details and tuning outputs
├── rag_configuration_comparison.csv   # Side-by-side experiment results
├── best_rag_configuration.json        # Selected configuration and metrics
└── final_rag_test_results.*           # Final five broker-style answers
```

## Running the notebook

1. Clone the repository and open `UTAIGA_Project_2_Full_Code_Notebook.ipynb` in Jupyter or Google Colab.
2. Install the packages listed in the notebook, including LangChain, ChromaDB, OpenAI, pandas, and DeepEval.
3. Create a local `config.json` containing your OpenAI API key. This file is excluded by `.gitignore`; never commit credentials.
4. Verify the dataset paths in the setup section.
5. Run the notebook from top to bottom to build the vector store, execute the RAG pipeline, evaluate configurations, and export results.

The generated `chroma_db/` directory is intentionally excluded from version control and will be recreated locally.

## Responsible use

This project is an educational proof of concept, not financial advice. Generated answers should be reviewed by a qualified professional and verified against current primary sources before being used in investment decisions or client communications.

## Author

**Nhan Le**  
Generative AI for Natural Language Processing — Project 2
