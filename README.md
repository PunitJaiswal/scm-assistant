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

| File | Description |
|---|---|
| `

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

