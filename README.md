Markdown
# CoT-Pipeline: End-to-End Pipeline for Curating Verified Reasoning Data

An asynchronous, high-throughput data curation pipeline engineered to autonomously extract, generate, validate, and curate structured Chain-of-Thought (CoT) reasoning data for post-training Large Language Models.

## Overview & Impact

Modern Large Language Models (LLMs) require explicit reasoning trajectories (thinking traces) to solve complex, multi-step problems. However, existing datasets often suffer from noisy extraction, missing solutions, and unverified reasoning steps. 

This pipeline was built to curate JEE/NEET-style problem datasets from heterogeneous sources (PDFs, Parquet, CSV). Because competitive-exam questions require strict numerical correctness, option consistency, and symbolic precision, naive LLM data generation introduces hallucinated steps and degrades downstream model quality. We solved this by enforcing rigorous multi-stage verification and a Rejection Sampling loop to keep only reliable problem-solution pairs.

### Key Engineering Achievements:
* **Scale:** Successfully generated, verified, and exported a standardized, training-ready dataset of 83,119 synthetic QA/CoT samples.
* **Performance Optimization:** Implemented concurrent multiprocessing (`--workers 8`) and optimized batching, achieving >40% latency reduction across LLM API endpoints.
* **Reliability:** Maintained a 99% successful response rate during massive generation runs via fail-fast validation and robust error handling.

## Pipeline Architecture

The system utilizes a 9-step automated architecture to transition raw, unstructured data into a verified CoT dataset:

1. **Data Ingestion:** Reads raw OCR text from PDFs, CSVs, and Parquet files.
2. **Column Mapping:** Aligns schemas across disparate datasets.
3. **Cleaning:** Performs deduplication and basic formatting.
4. **CoT Generation:** The "Teacher Model" generates step-by-step reasoning traces.
5. **Answer Extraction:** Autonomously extracts the final choice based on the reasoning trace.
6. **Verification:** Applies strict validation logic to compare the generated answer against the ground truth.
7. **Rejection Sampling Loop:** Failed verifications are pushed to a rejection log, while successes proceed to the approved dataset.
8. **Final Dataset Assembly:** High-quality, verified data is packed for export.

## Model Evaluation & Benchmarking

To ensure efficiency at scale, multiple models (DeepSeek-V3.2, GLM-4-FP8, Kimi-K2.5, Minimax-M2.5) were evaluated for this pipeline. 

**Best Validated Production Model:** GPT-OSS-120B
* **Reasons for selection:**
  * Handled large-scale, end-to-end verified runs natively within this pipeline configuration.
  * Displayed the strongest throughput/latency profile in our micro-benchmarks (lowest average latency per attempt).
  * Maintained high output efficiency and stable formatting behavior under strict answer-format constraints.

## Final Dataset Results

The pipeline successfully curated high-quality reasoning traces across the following scientific domains:

| Domain | Verified Samples Curated |
| :--- | :--- |
| **Mathematics** | 29,161 |
| **Chemistry** | 22,799 |
| **Biology** | 15,945 |
| **Physics** | 15,041 |
| **Others** | 173 |
| **Total** | **83,119** |

## Quick Start (Fresh Clone)

### 1. Clone repository
```bash
git clone [https://github.com/PrabhanshuChouhan/CoT-Pipeline.git](https://github.com/PrabhanshuChouhan/CoT-Pipeline.git)
cd CoT-Pipeline
2. Create Python environment
Bash
# Using venv:
python3 -m venv .venv
source .venv/bin/activate

# Using conda (optional):
conda create -n cot-pipeline python=3.10 -y
conda activate cot-pipeline
3. Install dependencies
Bash
pip install -r requirements.txt
4. Configure model endpoints
Edit config.yaml and set your model values:

model.url, model.model, model.api_key

If you use pipeline.py, also set: mapper_model.url, mapper_model.model, mapper_model.api_key

Usage
Step 1: Segmentation Pipeline (From OCR Output)
Reads raw OCR text from OCR_output/ and segments text into {question, answer} pairs.

Bash
# Run segmentation on demo input:
python segmentation_pipeline.py \
  --step segment \
  --input OCR_output/demo_ocr_output.parquet \
  --output-dir segmented_output/demo_ocr_output_segmented

# Run segmentation on all OCR files using worker pools:
python segmentation_pipeline.py --step segment --input-dir OCR_output --workers 8
Step 2: CoT Generation & Verification (pipeline.py)
Takes the segmented ground truth datasets, generates verified reasoning traces, and applies the validation loop.

Bash
# Demo run (Dry Run)
python pipeline.py Datasets/demo_dataset.parquet --config config.yaml --dry-run

# Full run with explicit column mapping
python pipeline.py Datasets/demo_dataset.parquet \
  --config config.yaml \
  --column-map question=<question_col> answer=<answer_col>
Future Work
Add symbolic/programmable verification (e.g., SymPy) for broader numerical and algebraic equivalence.

Improve OCR post-processing for equation-heavy images and scanned PDFs.

Enable automatic validation of complex mathematical problems using Tool-Integrated Reasoning (TIR).

Academic Acknowledgements
Authors: Piyush Keshri, Priyanshu Raj, Prabhanshu Chauhan, Raghvendra Kumar

Advisor: Prof. Mayank Singh

Institution: Indian Institute of Technology Gandhinagar