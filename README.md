
# Airlines Incremental Data Processing Pipeline with CI/CD via Azure DevOps

[![Azure Data Factory](https://img.shields.io/badge/Azure-Data%20Factory-Blue?style=flat-square&logo=microsoft-azure)](https://azure.microsoft.com/en-us/products/data-factory/)
[![Azure DevOps](https://img.shields.io/badge/Azure-DevOps-0078D4?style=flat-square&logo=azure-devops)](https://azure.microsoft.com/en-us/products/devops/)
[![GitHub](https://img.shields.io/badge/Version%20Control-GitHub-181717?style=flat-square&logo=github)](https://github.com/)

An enterprise-grade, end-to-end data engineering and DataOps project demonstrating incremental data ingestion, transformation, and automated multi-environment deployment (`Dev` $\rightarrow$ `Prod`).

This project establishes a robust data pipeline using **Azure Data Factory (ADF)** and **Azure Data Lake Storage Gen2 (ADLS)**, fully integrated with a **CI/CD infrastructure** managed by **Azure DevOps** using a self-hosted automation runner.



## 📌 Architecture Overview

```

[ ADLS Gen2 Source ] 

                 │

                 ▼

     [ ADF: Get Metadata ]

      (get_checkFlightDataExits)

                 │

                 ▼

       [ If_checkFileExistence ]

                 │

        ┌────────┴────────┐

        ▼                 ▼

     [ True ]          [ False ]

        │                 │

        ▼                 ▼

 [ ADF Data Flow ]   [ Fail Activity ]

(df_transformFlights)  (fail_forFile)

        │

        ▼

[ ADLS Gen2 Target ]

```

### 1. Data Pipeline Workflow (`pl_flightDataIngestion`)
* **Validation Stage (`get_checkFlightDataExits`)**: A `Get Metadata` activity verifies the presence of incoming dataset drops in the source storage account.
* **Conditional Routing (`If_checkFileExistence`)**: Evaluates the output of the metadata check.
    * **True Path**: Triggers the Mapping Data Flow (`df_transformFlightsData`) to process and transform raw airport and flight log metrics, writing optimized outputs to `ds_targetTransformedData`.
    * **False Path**: Routes to a web/fail activity (`fail_forFile`) to log missing file execution frames and block downstream dependencies cleanly.

### 2. CI/CD Release Architecture
* **Source Control**: Native Git orchestration tracks JSON templates inside the `dev` branch, while the public `main` repository houses production deployment assets and proof-of-concept visual documentation.
* **Continuous Deployment (CD)**: Automated release triggers capture changes upon artifact changes/merges into the Azure DevOps repository (`Airlines-ADF-CICD`), invoking an ARM deployment stage (`ProdADFDeployment`).
* **Compute Runner**: Deployments are executed deterministically on a dedicated self-hosted agent pool (`AgentOnMyLaptop`) using an authenticated Windows/Mac background service runner.

---

## 🛠️ Repository & Directory Structure

The codebase is organized as follows, keeping production pipeline logic distinct from visual execution proofs:

```directory
airlines-dataops-adf-cicd/
├── Images/                # Proof of concept execution screenshots and UI architecture validations
├── .gitignore             # Specifies intentionally untracked files to ignore from Git tracking
└── README.md              # Project documentation and architectural overview

```

* * * * *

🚀 Environment Configurations & Infrastructure
----------------------------------------------

### Multi-Environment Infrastructure Parity

The deployment landscape maintains mirror configurations between environments inside the`Poland Central` Azure geographies:

-   **Development Instance** : `airlines-adf-dev-poland`(Linked directly to Git for collaborative active workspace authoring)

-   **Production Instance** : `airlines-adf-prod-poland`(Locked workspace; state modifications are completely managed by Azure DevOps release gates)

### Self-Hosted Execution Agent Setup

To securely decouple external execution permissions from Microsoft-hosted clouds, deployments leverage a local background runner initialized under `vsts-agent-win-x64-4.273.0`:

PowerShell

```
# Agent Authentication & Lifecycle Registration
.\config.cmd --url [https://dev.azure.com/manishkumarrai3389](https://dev.azure.com/manishkumarrai3389) --auth pat --pool AgentOnMyLaptop --agent MANISH --runAsService

```

> **Note** : The agent runs continuously as a system daemon ( `vstsagent.manishkumarrai3389.AgentOnMyLaptop.MANISH`) with a delayed auto-start mode to process incoming build/release tasks on-demand.

* * * * *

🔄 DevOps CI/CD Automation Workflow
-----------------------------------

### 1\. Continuous Integration (CI)

-   Developers implement transformations and test logic inside individual feature branches or directly via the`dev` branch collaboration canvas.

-   Upon validation, changes are pushed to the Azure DevOps central repository. This synchronizes the live factory configurations into clear, modular ARM template JSON components.

### 2\. Continuous Deployment (CD Trigger)

-   **Trigger Filter** : Configured automated continuous deployment triggers track modifications targeting the root `dev`branch.

-   **Automated Release Actions** : The generation of fresh commits instantly spins up an automated pipeline release instance inside `AirlineCICDPipeline`.
