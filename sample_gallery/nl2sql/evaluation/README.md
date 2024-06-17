# Evaluating Evaluate Natural Language to SQL Converter

In this flow, we demonstrate how to evaluate the performance of the Natural Language to SQL Converter (NL2SQL) using various metrics such as Exact String Match (EM), Execution Accuracy (EX), Evaluate by LLMs, and Vector Similarity.

## Evaluation metrics

- **Exact String Match (EM)**: This metrics compares the predicted SQL query with the ground truth SQL query by comparing the strings. If the predicted SQL query is exactly the same as the ground truth SQL query, then the EM score is 1. Otherwise, the EM score is 0.

- **Execution Accuracy (EX)**: This metric evaluates the execution accuracy of the predicted SQL query. The predicted SQL query is executed on the database and the results are compared with the ground truth results. The EX score is calculated as the number of correct results divided by the total number of results.

- **Evaluate by LLMs**: This metric evaluates the performance of the model by comparing the predicted SQL query with the ground truth SQL query using the LLMs. The LLMs follows the prompt instruction to calculate the score.

- **Vector Similiarity**: This metric evaluates the performance of the model by comparing the predicted SQL query with the ground truth SQL query using the vectors. The score is calculated by cosine similarity.

## Getting Started

These instructions will get you a copy of the sample up and running using PromptFlow for development and testing purposes.

### Prerequisites

1.  The libraries required to run this project are listed in the `requirements.txt` file. To install them, use the following command:

```bash
    pip  install  -r  requirements.txt
```

2. Deploy an Azure SQL Database with sample (AdventureWorksLT) - [Setup Guide)([Create a single database - Azure SQL Database | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?view=azuresql&tabs=azure-portal)
3. Install and setup the [ODBC Driver for Microsoft SQL Server (17)](https://learn.microsoft.com/en-us/sql/connect/odbc/download-odbc-driver-for-sql-server?view=sql-server-ver16#version-17) to connect to the Azure SQL Database using pyodbc
   - (Local PromptFlow): Install the driver on the development machine
   - (Azure AI Studio): The Automatic Runtime doesn't contain the ODBC Driver, so you need to create a custom runtime with promptflow, pyodbc, and the driver installed. [Customize Runtime](https://learn.microsoft.com/en-us/azure/ai-studio/how-to/create-manage-runtime)
4. (Optional but highly recommended) Install and configure [Prompt flow for VS Code extension](https://marketplace.visualstudio.com/items?itemName=prompt-flow.prompt-flow) follow [Quick Start Guide](https://microsoft.github.io/promptflow/how-to-guides/quick-start.html).
   💡 This extension is optional but highly recommended for flow development and debugging.

5. (Optional) If you use the output of [this sample development flow](../development/), you have to preprocess the output to extract sql queries becuase the output includes not only sql queries but also explanation.

### Connections Setup

We need to setup 2 connections to run this sample. [Manage PromptFlow Connections](https://microsoft.github.io/promptflow/how-to-guides/manage-connections.html)

1.  Azure OpenAI connection: Create a connection to you Azure OpenAI endpoint with a GPT-4 Model deployment. For the evaluation sample, you will also need an text-embedding-ada-002 deployment
2.  Custom Connection: Create a custom connection for the Azure SQL Database. Sample YAML

```YML
$schema: https://azuremlschemas.azureedge.net/promptflow/latest/CustomConnection.schema.json
name: "to_replace_with_connection_name"
type: custom
configs:
  Server_name: "tcp:<server-name>.database.windows.net,1433"
  User_name: "<sql-login-name>"
  Database_name: "<database-name>"
secrets:
# Don't replace the '<user-input>' placeholder. The application will prompt you to enter a value when it runs.
  Password: "<user-input>"
```

**_NOTE:_**

- Ensure connectivity to your Azure SQL Database. Ref: https://learn.microsoft.com/en-us/azure/azure-sql/database/connectivity-settings?view=azuresql&tabs=azure-portal
- Change version of the ODBC driver in get_table_names.py and get_table_schema.py based on the version installed in your machine/runtime.

Once the connections are created, modify the python and LLM nodes in the flow.dag.yaml with the appropriate connections names.

###### For Example -

###### Query_to_database (python tool)

```YAML
- name: Query_to_database
  type: python
  source:
    type: code
    path: Query.py
  inputs:
    sql_groundtruth: ${inputs.sql_groundtruth}
    sql_generated: ${inputs.sql_generated}
    sqlconn: <sql-connection-name>
```

###### Evaluate_by_LLM (LLM tool)

```YAML
- name: Evaluate_by_LLM
  type: llm
  source:
    type: code
    path: evaluate_by_llms.jinja2
  inputs:
    deployment_name: gpt-4
    sql_groundtruth: ${inputs.sql_groundtruth}
    sql_generated: ${inputs.sql_generated}
  connection: azure_openai_connection
  api: chat
```

#### Sample Flow Run (VSCode)

![Sample Flow Run](./media/evaluation_flow_run_sample.png)

#### Batch Evaluation

To run a batch evaluation you need the generated SQL queries from the development flow and corresponding ground truth SQL queries to compare against. We have added the following sample golden dataset files:
- sample-groundtruth-sql.jsonl: user queries from the development flow, and their corresponding groundtruth T-SQL queries for this purpose.
- sample-groundtruth-and-generated-sql.jsonl: user queries from the development flow, their corresponding groundtruth T-SQL queries for this purpose, and generated T-SQL queries from the development flow.

Follow the guidance here - https://microsoft.github.io/promptflow/how-to-guides/run-and-evaluate-a-flow/index.html#run-evaluation-flow-against-run

### Important Notes

- This sample is not intended for production.


###### Credits - 
Sample queries and T-SQL queries - https://github.com/keilorg/sqlzoo-solutions/blob/main/16%20AdventureWorks.sql
