# CoT-Pipeline: End-to-End Pipeline for Curating Verified Reasoning Data

An asynchronous, high-throughput data curation pipeline engineered to autonomously extract, generate, validate, and curate structured Chain-of-Thought (CoT) reasoning data for post-training Large Language Models.

## Overview & Impact

Modern Large Language Models (LLMs) require explicit reasoning trajectories (thinking traces) to solve complex, multi-step problems. However, existing datasets often suffer from noisy extraction, missing solutions, and unverified reasoning steps. 

This pipeline was built to curate JEE/NEET-style problem datasets from heterogeneous sources (PDFs, Parquet, CSV). Because competitive-exam questions require strict numerical correctness, option consistency, and symbolic precision, naive LLM data generation introduces hallucinated steps and degrades downstream model quality. We solved this by enforcing rigorous multi-stage verification and a Rejection Sampling loop to keep only reliable problem-solution pairs.

### Key Engineering Achievements:
* **Scale:** Successfully generated, verified, and exported a standardized, training-ready dataset of 83,119 synthetic QA/CoT samples[cite: 2].
* **Performance Optimization:** Implemented concurrent multiprocessing (`--workers 8`) and optimized batching, achieving >40% latency reduction across LLM API endpoints.
* **Reliability:** Maintained a 99% successful response rate during massive generation runs via fail-fast validation and robust error handling.

## Pipeline Architecture

The system utilizes a 9-step automated architecture to transition raw, unstructured data into a verified CoT dataset[cite: 2]:

1. **Data Ingestion:** Reads raw OCR text from PDFs, CSVs, and Parquet files[cite: 2].
2. **Column Mapping:** Aligns schemas across disparate datasets[cite: 2].
3. **Cleaning:** Performs deduplication and basic formatting[cite: 2].
4. **CoT Generation:** The "Teacher Model" generates step-by-step reasoning traces[cite: 2].
5. **Answer Extraction:** Autonomously extracts the final choice based on the reasoning trace[cite: 2].
6. **Verification:** Applies strict validation logic to compare the generated answer against the ground truth[cite: 2].
7. **Rejection Sampling Loop:** Failed verifications are pushed to a rejection log, while successes proceed to the approved dataset[cite: 2].
8. **Final Dataset Assembly:** High-quality, verified data is packed for export[cite: 2].

*(Note: Add your 9-step flowchart image here in the repository)*

## Model Evaluation & Benchmarking

To ensure efficiency at scale, multiple models (DeepSeek-V3.2, GLM-4-FP8, Kimi-K2.5, Minimax-M2.5) were evaluated for this pipeline[cite: 2]. 

**Best Validated Production Model:** GPT-OSS-120B[cite: 2]
* **Reasons for selection:**
  * Handled large-scale, end-to-end verified runs natively within this pipeline configuration[cite: 2].
  * Displayed the strongest throughput/latency profile in our micro-benchmarks (lowest average latency per attempt)[cite: 2].
  * Maintained high output efficiency and stable formatting behavior under strict answer-format constraints[cite: 2].

## Final Dataset Results

The pipeline successfully curated high-quality reasoning traces across the following scientific domains[cite: 2]:

| Domain | Verified Samples Curated[cite: 2] |
| :--- | :--- |
| **Mathematics** | 29,161[cite: 2] |
| **Chemistry** | 22,799[cite: 2] |
| **Biology** | 15,945[cite: 2] |
| **Physics** | 15,041[cite: 2] |
| **Others** | 173[cite: 2] |
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
Add symbolic/programmable verification (e.g., SymPy) for broader numerical and algebraic equivalence[cite: 2].

Improve OCR post-processing for equation-heavy images and scanned PDFs[cite: 2].

Enable automatic validation of complex mathematical problems using Tool-Integrated Reasoning (TIR)[cite: 2].

Academic Acknowledgements
Authors: Piyush Keshri, Priyanshu Raj, Prabhanshu Chauhan, Raghvendra Kumar[cite: 2]

Advisor: Prof. Mayank Singh[cite: 2]

Institution: Indian Institute of Technology Gandhinagar[cite: 2]


---

### How to Update Your Old README

You have two simple ways to update the `README.md` in your GitHub repository.

#### Option 1: The Fast Way (Using the GitHub Website)
1. Go to your repository page on GitHub (`[https://github.com/PrabhanshuChouhan/CoT-Pipeline](https://github.com/PrabhanshuChouhan/CoT-Pipeline)`).
2. Scroll down to where the current README is displayed.
3. Click the **pencil icon** (Edit this file) located in the top-right corner of the README box.
4. Delete the old text and paste the new code block above.
5. Click the green **Commit changes...** button at the top right. 
6. Add a brief commit message (e.g., "Updated README with pipeline architecture and metrics") and click **Commit changes**.

#### Option 2: The Engineer's Way (Using Terminal/VS Code)
1. Open your terminal and navigate to your cloned repository folder.
2. Make sure your local repository is up to date:
   `git pull origin main`
3. Open the `README.md` file in your code editor (like VS Code), replace the contents with the new text, and save the file.
4. Stage the file for your commit:
   `git add README.md`
5. Commit the changes:
   `git commit -m "docs: Update README with comprehensive pipeline documentation"`
6. Push the changes to GitHub:
   `git push origin main`
