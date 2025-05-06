---
title: "FinOps 101 - Saving Databricks Cost"
date: 2025-05-06
description: "FinOps for Data Engineers: Stop Wasting Money by Using Databricks Smarter"
ogimage: assets/images/tech/finops101-databricks.png
tags: 
- databricks
- finops
- cloud
categories:
- tech
---
FinOps for Data Engineers: Stop Wasting Money by Using Databricks Smarter
---
![Kubernetes](assets/images/tech/finops101-databricks.png)

Databricks has revolutionized how data teams handle big data, machine learning, and analytics by offering a unified cloud platform with powerful distributed computing. But here’s the tough truth:

Many companies are burning cloud money unnecessarily because developers are using Databricks notebooks as an all-in-one development environment — even for tasks that don’t need cluster power.

This article explores why this behavior is costly, how to shift towards local-first development, and how to use the Databricks CLI to push only the final, distributed workloads to the cloud — saving your company potentially thousands of dollars per month.

### The Problem: Notebooks as IDE = Unnecessary Cloud Costs
Notebooks are popular because they’re interactive, visual, and easy to use. But here’s what’s happening across many teams:
✅ A developer wants to write a small Spark script.
✅ They open a Databricks notebook.
✅ The cluster spins up.
✅ They write some test code, fix syntax errors, retry, debug, iterate.
✅ They run small test datasets (which could easily fit on a laptop).
✅ They hit an error or need to fix logic, rerun, reconsume cluster time.

> Every minute the cluster runs, the company is paying for cloud compute, often for work that doesn’t need distributed resources. Multiply that by dozens or hundreds of engineers, and you’re staring at runaway Databricks bills.

### The FinOps Shift: Local-First Development + Cloud-Only Final Runs
Here’s the FinOps-aware workflow we recommend:

✅ Step 1: Develop locally
Write, debug, and unit-test your Spark/PySpark/Scala code on your laptop. Use tools like:
	•	Local Jupyter Notebooks or JupyterLab
	•	VS Code or PyCharm with Spark dependencies
	•	Small local sample datasets

This costs you zero cloud dollars.

✅ Step 2: Use Databricks CLI for Final Cloud Runs
Once your code works locally, push it to Databricks clusters only when you need distributed power — i.e., when running on full datasets, leveraging Spark parallelism, or integrating with cloud-specific tools.

### Databricks CLI Setup and Usage
Here’s how to set up the Databricks CLI and integrate it into your local-to-cloud workflow.

1️⃣ Install the Databricks CLI

First, install the CLI:
```zsh
pip install databricks-cli
```

2️⃣ Configure the CLI

You need a Databricks personal access token. Generate it in your Databricks workspace (Account Settings → User Settings → Access Tokens).

Then configure the CLI:
```zsh
databricks configure --token
```
You’ll be prompted for:
	•	Databricks host (e.g., https://.databricks.com)
	•	Access token

3️⃣ Use the CLI to Manage Jobs and Notebooks

✅ Upload local notebook or script
```zsh
databricks workspace import ./my_notebook.py /Users/yourname/my_notebook.py
```

✅ Run a job on Databricks
You can define a job JSON file (job-config.json) like:
```json
{
  "name": "my-test-job",
  "new_cluster": {
    "spark_version": "13.3.x-scala2.12",
    "node_type_id": "i3.xlarge",
    "num_workers": 2
  },
  "notebook_task": {
    "notebook_path": "/Users/yourname/my_notebook.py"
  }
}
```

Then trigger the job:
```zsh
databricks jobs create --json-file job-config.json
```

✅ Monitor job runs
```zsh
databricks runs list
databricks runs get --run-id <run-id>
```

✅ Export results or logs
```zsh
databricks fs cp dbfs:/path/to/results ./local_results --recursive
```


### Benefits of Local + CLI Workflow

✅ Cost Savings
You avoid paying cloud cluster costs for development/debugging iterations.

✅ Faster Feedback
Local testing gives you immediate feedback without cluster spin-up delays.

✅ Cleaner Code
You push only working, tested code to the cluster, reducing failed runs.

✅ FinOps Culture
Your team builds a culture of cloud cost awareness — a key competitive advantage as cloud spend grows.


### Final FinOps Advice - The Conclusion
Databricks is powerful — but with great power comes great responsibility (and costs).

By adopting a local + CLI workflow, your team can cut waste on iterations, accelerate development, and build a FinOps-aware engineering culture that respects cloud budgets.

Start today: install the Databricks CLI, shift development locally, and watch your cloud bills drop — without sacrificing innovation or speed.

> If you want, I can also help draft a companion GitHub repo with sample scripts and CLI configs to share with your readers. Would you like that?