# Funniest‐Joke Generation Pipeline
A research‐grade pipeline that generates, ranks, and filters creative jokes using PlanSearch‐style prompts and Gemini ("LLM‐as‐Judge") scoring, with embedding‐based novelty detection.

## Overview
This project demonstrates a complete end‐to‐end system for generating the "funniest joke imaginable" given any input context. Key innovations include:
- **PlanSearch‐Style Generation**: Decompose "joke creation" into multiple "plans" (setups/angles), followed by rollout of concise punchlines.
- **Gemini as Judge**: Use Google’s Gemini 2.0 Flash model to score jokes on humor, ensuring a consistent, automated ranking.
- **Novelty Detection**: Leverage SentenceTransformer embeddings and FAISS to flag near‐duplicate or memorized content.
- **Human Evaluation Integration**: Export top candidates for crowd annotation, compute Spearman correlations, and visualize LLM vs. human ratings.

Whether for academic exploration or production deployment, this pipeline balances creativity, rigor, and reproducibility.

## Features
- **Modular Generation**
  - `generate_candidate_jokes(topic)`: Produces multiple jokes per "plan" for diverse outputs.
  - Adjustable parameters: number of plans, rollouts per plan, sampling temperature.
- **Automated Scoring**
  - `score_jokes_with_llm(jokes, topic)`: Queries Gemini to assign 1–10 humor scores.
  - Bias mitigation via shuffled inputs and reference anchors.
- **Novelty Filtering**
  - check_novelty(joke_list, faiss_index, threshold): Labels jokes as "Novel" vs. "Possibly Memorized."
  - Build FAISS index over known joke corpus for semantic‐similarity checks.
- **End‐to‐End Pipeline**
  - `generate_top_jokes(context)`: Orchestrates generation, scoring, ranking, and novelty annotation.
  - Returns a Pandas DataFrame sorted by humor score and novelty flag.
- **Human Evaluation Suite**
  - Exports top_jokes_for_human_eval.csv for annotators.
  - Computes Spearman correlation and displays scatter plots comparing LLM vs. human funniness ratings.

## Installation
- **Clone the Repository**
  ```
  git clone https://github.com/Vishal-sys-code/funniest-joke-pipeline.git
  cd funniest-joke-pipeline
  ```
- **Create & Activate a Virtual Environment**
  ```
  python3 -m venv venv
  source venv/bin/activate
  ```
- **Install Dependencies**
  ```
  pip install \
  google-generativeai \
  sentence-transformers \
  faiss-cpu \
  pandas \
  matplotlib \
  ace_tools
  ```
- **Set Environment Variables**
  ```
  export GEMINI_API_KEY="YOUR_GEMINI_API_KEY"
  ```

## Pipeline Architecture
- Plan Generation
  - Prompt Gemini: "List 5 unique joke ideas for ."
  - Parse response as a Python‐list of "plans."
- Rollout (Joke Creation)
  - For each plan, prompt Gemini: "Write a short joke based on this plan."
  - Repeat per plan to introduce sampling diversity.
- Automated Scoring
  - Aggregate all candidate jokes.
  - Shuffle and prompt Gemini: "Rate each joke 1–10 (funny only)."
  - Parse JSON list of scores; rank descending.
- Novelty Check
  - Embed each top joke via SentenceTransformer.
  - Compare against FAISS index of known jokes.
  - Label as "Novel" if nearest‐neighbor distance > threshold (e.g., 0.25).
- Final Output
  - DataFrame with columns: `joke | score | novelty`.
  - Ready for export, further filtering, or API serving.
 
## Evaluation & Metrics

- **LLM‐vs‐Human Correlation**
  - Export top 10 jokes to top_jokes_for_human_eval.csv.
  - After human annotation (1–10 funniness), load human_ratings.csv.
  - Compute Spearman ρ between LLM scores and human scores; visualize scatter plot.

- **Novelty Coverage**
  - Track percentage of jokes flagged as "Novel" vs. "Possibly Memorized."
  - Perform held‐out prompt experiments to verify true creativity.

- **Latency & Cost**
  - Measure end‐to‐end latency: ~2–3 seconds per context with Gemini‐2.0.
  - Token usage: ~150 tokens/plan, ~100 tokens/rollout, ~50 tokens/scoring per batch.
