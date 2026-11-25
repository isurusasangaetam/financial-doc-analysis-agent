# Financial Document Analysis Agent

A RAG-powered, state-machine AI agent built with **LangGraph**, **FAISS**, and **Qwen2.5-1.5B-Instruct** to analyze Apple Inc.'s SEC Form 10-K annual reports (FY 2023, FY 2024, and FY 2025).

Instead of relying on an LLM for arithmetic, the agent separates semantic document retrieval from deterministic Python financial calculations, then generates structured analytical reports, charts, and a combined PDF deliverable.

---

## Architecture & Workflow

The agent operates as a cyclic state graph orchestrated by a Planner node:

```text
User Question
      │
      ▼
  [Planner] ──► FINISH ──► END
   ▲  ▲  ▲
   │  │  │
   ├──┼──┴──────────────────────────┐
   │  │                             │
   ▼  ▼                             ▼
[Retrieve] ──► [Calculate] ──► [Report]
```

### Core Agentic Components

| Component | Implementation |
| :--- | :--- |
| **Goal** | Answer multi-year financial and segment-level questions grounded strictly in Apple Inc. 10-K filings (2023–2025). |
| **State** | `FinancialAgentState` (`TypedDict`) tracking the question, target years, retrieved context, extracted values, calculation results, final report, and completed steps. |
| **Planner** | `planner_node` using `Qwen2.5-1.5B-Instruct` with state guardrails to select `RETRIEVE`, `CALCULATE`, `REPORT`, or `FINISH`. |
| **Tools** | `financial_retrieval_tool`, `retrieve_within_year`, `extract_financial_metric`, `extract_category_revenues`, `calculate_percent_change`, `analyze_category_contributions`, and `generate_financial_report`. |
| **Action Nodes** | `retrieve_node`, `calculate_node`, and `report_node`. |
| **Routing** | Conditional edges from `planner` via `route_from_planner`, looping back after each action node. |
| **Termination** | Stops when the planner outputs `FINISH` (routed to `END`) or hits the graph `recursion_limit`. |

---

## Tech Stack

* **Orchestration:** `LangGraph`
* **LLM:** `Qwen/Qwen2.5-1.5B-Instruct` (`transformers`, `torch`, `accelerate`)
* **Embeddings & Vector Store:** `sentence-transformers` (`all-MiniLM-L6-v2`) + `FAISS` (`IndexFlatIP`)
* **PDF Parsing:** `PyMuPDF` (`pymupdf` / `fitz`)
* **Visualization & Export:** `matplotlib`, `fpdf2`

---

## Repository Structure

```text
financial-doc-analysis-agent/
├── Financial_Doc_Agent.ipynb      # Complete Google Colab / Jupyter notebook
├── financial_analysis_report.pdf  # Generated PDF report for Q1, Q2, and Q3
├── charts/
│   ├── q1_revenue_trend.png       # Total Net Sales (2023–2025) bar chart
│   └── q2_category_share.png      # FY 2025 Product Category share pie chart
└── README.md
```

*(Note: SEC Form 10-K PDFs can be downloaded directly from Apple Investor Relations or the SEC EDGAR database and placed in the project directory.)*

---

## Getting Started (Google Colab)

1. **Upload the Dataset:** Create a folder in Google Drive (e.g., `CSC3173_PA02_Financial_Agent`) and upload Apple's three Form 10-K PDFs:
   * `10-K-2023-As-Filed.pdf`
   * `10-K-2024-As-Filed.pdf`
   * `10-K-2025-As-Filed.pdf`
2. **Open the Notebook:** Upload `Financial_Doc_Agent.ipynb` to Google Colab.
3. **Enable GPU:** Go to **Runtime → Change runtime type → T4 GPU**.
4. **Set Directory Path:** Update `PROJECT_DIR` in Cell 1 to match your Google Drive folder path.
5. **Run All Cells:** Execute the notebook sequentially to build the FAISS index, run the agent on Q1–Q3, render the charts, and export `financial_analysis_report.pdf`.

---

## Sample Evaluation Questions & Verified Results

* **Q1 — Total Revenue Change (2023–2025):**
  * FY 2023: `$383,285M` → FY 2024: `$391,035M` → FY 2025: `$416,161M`
  * **Change:** `+$32,876M` (`+8.58%`)
* **Q2 — Top Product Category (FY 2025):**
  * **iPhone:** `$209,586M` (`50.36%` of total revenue), followed by Services (`$109,158M`, `26.23%`).
* **Q3 — Services Revenue Growth (2023–2025):**
  * FY 2023: `$85,200M` → FY 2025: `$109,158M` (`+28.12%`).

---

## License

This project was developed for academic coursework (`CSC3173: Artificial Intelligence`) and is available for educational use.
