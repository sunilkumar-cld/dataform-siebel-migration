# Dataform CRM Migration Pipeline (Siebel to BigQuery)

Dataform pipeline for legacy Siebel CRM migration to BigQuery. Standardizes customer accounts, inventory, and order mappings via automated STTM rules. Features multi-tier address fallbacks, optimized email contact filters, and a strict Quality Gate validation engine routing data cleanly between production targets and JSON fallout logs.

---

## 🏗️ Architecture & Component Layers

The pipeline processing flow is split into three decoupled operational stages:

### 1. View Tier (`.../source/`)
Lightweight relational layers parsing source operational parameters:
*   **`stg_inventory_vw`**: Exposes master tracking scopes. Enriches customer lines with an inline, performance-optimized corporate switch mapping (`is_colt_internal`).
*   **`stg_addr_vw`**: Builds an active-to-inactive lookup fallback matrix tier prioritizing active flags over legacy operational boundaries.
*   **`stg_con_role_vw`**: Dynamically normalizes variable role labels (`LEGAL`, `KEY`, etc.) into standardized system formats.

### 2. Staging/Transformation Tier (`.../transformation/`)
Heavy transformation engines executing strict cross-regional STTM rule sets:
*   **`CUSTOMER_ORDER_STG`**: Maps core transactional matrices, implementing multi-tier address allocations and contact lookups.
*   **`AGREEMENT_STG`**: Aligns corporate framework agreements using the validated `04899692` master fallback tokens.

### 3. Production Tier (`.../target/` & `.../fallout/`)
Final destination split routers evaluating data sanity tokens via the Quality Gate:
*   **Success Tables (`.../target/`)**: Stores verified records passing all checks.
*   **Fallout Tables (`.../fallout/`)**: Logs broken pipelines into highly structured, stratified JSON error packets.

---

## 📁 Project Directory Layout

Codebase assets are organized cleanly into entity-specific domains within the main `definitions/` directory:

```text
definitions/
├── agreement/                  # Master Agreement & Framework Pipeline
│   ├── fallout/                # AGREEMENT_FO target logging configurations
│   ├── source/                 # Raw/Staging views (stg_con_role_vw, stg_addr_vw, etc.)
│   ├── target/                 # PRODUCTION_READY verified AGREEMENT targets
│   └── transformation/         # AGREEMENT_STG analytical execution layer
│
├── customer_order/             # Transactional Customer Order Pipeline
│   ├── fallout/                # CUSTOMER_ORDER_FO configuration targets
│   ├── target/                 # PRODUCTION_READY verified CUSTOMER_ORDER targets
│   └── transformation/         # CUSTOMER_ORDER_STG analytical execution layer
│
├── service_order/              # Downstream localized execution pipeline track
└── service_order_attachm...    # Attachment lineage mapping blocks
```

---

## 🛡️ Automated Quality Gate Matrix

To prevent downstream data contamination, records are automatically routed based on specific metrics mapping back to the **STTM Specification Codes**:

### Agreement Pipeline Specs

| Target Attribute Code | Monitored Field | Validation Failure Logic |
| :--- | :--- | :--- |
| **NPA_0005** | `legal_party_contact` | Raises fallout if the account OCN anchor breaks structural contact resolution. |
| **NPA_0012** | `account_name` | Fails if the customer master OCN string is null or blank. |
| **NPA_0017** | `colt_contracting_country`| Fails if the structural region value cannot be dynamically resolved. |
| **NPA_0022** | `legal_address` | Fails if standard lookup tiers collapse or if default matrices are missing. |

### Customer Order Pipeline Specs

| Target Attribute Code | Monitored Field | Validation Failure Logic |
| :--- | :--- | :--- |
| **NPA_0028** | `status` | Fails if the underlying transaction status is not marked as `Completed`. |
| **NPA_0031** | `account_executive` | Fails if the dedicated positional team ID (`PR_POSTN_ID`) is missing. |
| **NPA_0032** | `account_number` | Fails if the target account master OCN reference string evaluates to null. |
| **NPA_0034** | `ordering_party` | Fails if the request-submitting entity name mapping context is missing. |

---

## 🚀 Deployment & Development Setup

### Prerequisites
* Google Cloud Project with **BigQuery** activated.
* **Dataform Service Account** permissions (`roles/dataform.editor`, `roles/bigquery.dataEditor`).

### Local Development Flow
1. Clone the repository to your Dataform Development Workspace.
2. Ensure variable configs are configured in `dataform.json`:
   ```json
   {
     "vars": {
       "target_dataset": "YOUR_TARGET_DATASET",
       "eu_dataset": "YOUR_EU_SOURCE_DATASET",
       "asia_dataset": "YOUR_ASIA_SOURCE_DATASET"
     }
   }
   ```
3. Click **Compile** to verify dependency graph continuity.
4. Execute individual tables using the Dataform execution panel or via Dataform CLI framework targets.

---

## 💡 Troubleshooting & Workspace Cache Refreshes

If Dataform throws an `Unrecognized name` compilation error on newly updated columns:
1. Commit your current changes to the development branch.
2. Hard refresh your browser (`Ctrl + Shift + R` or `Cmd + Shift + R`) to drop old session cache contexts.
3. Add a single blank space inside the target query definition file and click **Save** to force the compilation worker to evaluate the schema from scratch.
