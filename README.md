# SCM Assistant Bot

A Retrieval-Augmented Generation (RAG) chatbot built on [Flowise](https://flowiseai.com) that answers questions about the BQBYTE Technologies supplier network using live supplier performance data and the internal governance policy.

---

## 🔗 Public Chatbot URL

> **https://cloud.flowiseai.com/chatbot/27b38bdd-d867-40f5-8570-1b17c9921bd8**
>

---

## 🛠️ Stack

| Component | Choice | Reason |
|---|---|---|
| **LLM** | `llama-3.3-70b-versatile` (OpenAI) | Cost-efficient, strong reasoning for structured data QA |
| **Embeddings** | `sentence-transformers/all-mpnet-base-v2` (HuggingFace) | Fast, open-source, high-quality semantic search |
| **Vector Store** | Pinecone | Sufficient for ~2000 PO rows + 8-page PDF in a single session |
| **Chain** | Conversational Retrieval QA Chain | Supports chat history + source document citation |
| **Text Splitter** | Recursive Character Text Splitter | Best general-purpose splitter for mixed CSV/PDF content |

---

## 📂 Data Sources

| File | Description |
|---|---|
| `SupplyChain_Governance_Policy_v3.2.pdf` | 10-section supplier governance policy — tier thresholds, SLAs, penalties, audit rules, disruption response procedures |
| `supplier_performance_data.csv` | 2,000 purchase orders · 116 suppliers · 27 columns (OTD rate, defect rate, compliance score, risk level, disruption flags, PO value, etc.) |

---

## Preprocess CSV File
After Preprocessing the csv file i got some new json file which has some useful information about the csv data. These file extracts the relevant information from the existing csv file such as aggregation, grouping and classification of supplier function using the crieteria given.

| File Name | Description |
|---|---|
| `supplier_tiers_rag.jsonl` | Section 2 tier classification (Tier-1/2/3 or Below Tier-3) per supplier, based on spend, compliance, OTD, and defect rate. |
| `otd_by_supplier_rag.jsonl` | Section 3.1 on-time delivery rate per supplier, evaluated against the supplier's assigned-tier OTD floor (93/84/75%). |
| `defect_by_supplier_rag.jsonl` | Section 3.2 defect rate per supplier vs. tier maximum, including the >8% single-shipment hold/RCA flag. |
| `leadtime_by_supplier_rag.jsonl` | Section 3.3 lead-time compliance per supplier, flagging suppliers above the 50-day ELTRP review threshold. |
| `compliance_by_supplier_rag.jsonl` | Section 3.4 compliance-score floor checks per supplier, flagging SWL (<60) and follow-up-audit (<70) status. |
| `otd_penalties_rag.jsonl` | Section 4.1 OTD penalty assessments per supplier-quarter, with tier-specific penalty rates, dollar amounts, and CAP triggers. |
| `volume_rebate_rag.jsonl` | Section 4.2 Tier-1 volume-rebate eligibility per supplier-year, with the 2.5% rebate amount or the specific failing criteria. |
| `defect_escalation_rag.jsonl` | Section 4.3 defect penalty escalation per supplier-quarter (4% consecutive-quarter surcharge and $15k Quality Containment Fees). |
| `risk_levels_rag.jsonl` | Section 5.1 risk-level classification (Low/Medium/High) per supplier from disruption flags and compliance, with computed vs. reported risk. |
| `escalations_rag.jsonl` | Section 5.2 CPO escalation events per supplier-quarter, triggered by two consecutive High-risk quarters. |
| `concentration_risk_rag.jsonl` | Section 5.3 portfolio-level region (>45%) and country (>25%) spend-concentration checks, plus a summary record. |
| `sustainability_rag.jsonl` | Section 6.1 sustainability-score floor checks per supplier vs. tier minimum (80/60/45), flagging SIP requirements. |
| `certifications_rag.jsonl` | Section 6.2 per-supplier certification compliance, listing held, required, and missing mandatory certs (plus advisory end-use certs). |
| `certification_matrix_rag.jsonl` | Section 8 reference records for each certification in the Approved Certification Matrix (definition, scope, holder count). |
| `audit_frequency_rag.jsonl` | Section 7.1 audit-cadence and overdue status per supplier, with the evaluation anchor date and disruption spot-audit flags. |
| `audit_scores_rag.jsonl` | Section 7.2 compliance-score records per supplier plus the weighted-composite scoring methodology and follow-up-audit triggers. |
| `disruption_response_rag.jsonl` | Section 9 disruption response level (1/2/3) per supplier with safety-stock and alternate-supplier actions, plus a levels-reference record. |
| `alternate_suppliers_rag.jsonl` | Section 10 alternate-supplier eligibility per supplier against the four activation conditions, plus the policy-vs-data discrepancy note. |

## ⚙️ Chunk Configurations Tested

### Config A — Fine-grained (Final choice)
| Parameter | Value |
|---|---|
| Chunk Size | **500** |
| Chunk Overlap | **100** |
| CSV chunks produced | ~4158 |
| JSON chunks produced | ~1845 |
| PDF chunks produced | ~35 |
| **Total chunks** | **~6038** |

**Observations:** Smaller chunks preserved individual supplier rows intact, leading to more precise retrieval for single-supplier lookups and exact metric comparisons. Policy sections stayed self-contained.

### Config B — Coarse
| Parameter | Value |
|---|---|
| Chunk Size | **1000** |
| Chunk Overlap | **200** |
| CSV chunks produced | ~2124 |
| JSON chunks produced | ~1838 |
| PDF chunks produced | ~19 |
| **Total chunks** | **~3981** |

**Observations:** Larger chunks caused multiple unrelated supplier rows to land in the same chunk, which sometimes confused the LLM into mixing up metrics across suppliers. Policy retrieval was slightly better since full sections loaded together, but data precision suffered. Config A chosen as final.

---

## 💬 Sample Q&A — Verified Answers

### Q1 — Tier-3 Suppliers with Active Disruption Flag

**Question:** Which Tier-3 suppliers have an active disruption flag, and what response level applies per policy?

**Answer:**

There are 11 Tier-3 suppliers with an active disruption flag, all of which are classified as High Risk. These suppliers are: Dravex Components India, Plataforma Metales SA, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Quetzal Textiles, Sibertek Molding, Archipelago PCB Corp, Varna Electronics EAD, and Deltaforge Vietnam. Under Policy §9, this triggers a Level 3 Activate response, which calls for CPO escalation together with an alternate supplier covering at least 40% of volume.

---

### Q2 — Volume Rebate Program Qualifiers

**Question:** Which suppliers qualify for the annual Volume Rebate Program and how many are there?

**Answer:** None of the suppliers mentioned in the documents qualify for the annual Volume Rebate Program, as they all fail to meet at least one of the required criteria: OTD at or above 93.0%, Defect Rate below 0.5%, and Sustainability Score at or above 85. Therefore, there are O suppliers that qualify for the annual Volume Rebate Program.

---

### Q3 — Region with Highest PO Value & Concentration Limit

**Question:** Which region has the highest total PO value, and does it breach the concentration limit?

**Answer:** The region with the largest total PO value is EMEA, at $193,987,179.91, which works out to roughly 48.5% of the overall spend of $399,563,494.10. This does breach the concentration limit, as it exceeds the 45% regional concentration cap set in Policy 55.3, and a Diversification Plan must be produced within 60 days.

---

### Q4 — Supplier Watch List (SWL) Status

**Question:** Which suppliers are on Supplier Watch List (SWL) status and what does it restrict?

**Answer:**
There are 11 suppliers on SWL status, each having a Compliance Score below 60: Deltaforge Vietnam, Maghreb Castworks, Helios Pack Greece, Cerromax Mineria, Orinoco Pack SAPI, Varna Electronics EAD, Quetzal Textiles, Plataforma Metales SA, Archipelago PCB Corp, Dravex Components India, and Sibertek Molding. As specified in Policy §3.4, SWL status caps new PO issuance at 20% of the prior quarter's volume.

---

### Q5 — Product Category with Highest Average Defect Rate

**Question:** Which product category has the highest average defect rate and does it exceed the Tier-2 limit?

**Answer:**
The product category with the highest average defect rate is Mechanical Components, with an average defect rate of 2.12% measured across 360 POs. This remains under the Tier-2 ceiling of 2.50% defined in Policy §3.2, so there is no breach, though it is approaching the threshold.

---

