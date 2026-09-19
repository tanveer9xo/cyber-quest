## Quest Summary

**Methodology:**
1. **Data Parsing:** Extracted key fields (`Computer`, `EventCode`, `Image`, `QueryName`, `UtcTime`, `ProcessId`) from raw Sysmon JSON logs using PySpark's `get_json_object` to create a clean, usable dataframe.
2. **Detection Engineering:** Built a detection query targeting Event Code 22 (DNS queries). I filtered the data for Microsoft Office applications (`winword.exe`, `excel.exe`, `powerpnt.exe`) to test the phishing hypothesis. This isolated an anomalous query from `WINWORD.EXE` to `www.mediafire.com`, strongly indicating a malicious document attempting to download a secondary payload.
3. **Normalization & Alerting:** Normalized the resulting dataframe to the Elastic Common Schema (ECS) (e.g., mapping `Computer` to `host.name`). I then utilized a PySpark UDF to simulate Threat Intelligence enrichment, flagging the file-sharing domain as malicious, and packaged the data into an actionable alert with MITRE ATT&CK mapping (T1566.001).

**AI Usage Disclosure:**
I utilized an AI assistant as a pair-programming and troubleshooting reference tool during this exercise.

* **What I used:** LLM Chat Interface.
* **Example Prompts:** 
  * *"Still getting this error after switching to Java 17: WARNING: Using incubator modules..."*
  * *"Getting this at SparkSession.builder step: java.net.BindException: Can't assign requested address: Service 'sparkDriver' failed after 16 retries..."*
  * *"How do I fix this PySpark error: `A column, variable, or function parameter with name 'dns'.'question'.'name' cannot be resolved`?"*
* **How it helped:** It was highly effective for local environment configuration on macOS. It helped diagnose that the Jupyter kernel was caching an older Java version, provided the code to dynamically lock `JAVA_HOME` to Java 17 within the notebook, and resolved a Mac-specific Spark networking bug by overriding `SPARK_LOCAL_IP` to `127.0.0.1`. It also served as a syntax reference for creating PySpark UDFs for the threat intel enrichment.
* **What it got wrong:** 
  1. **Missing Imports:** When providing the initial Python script to lock the `JAVA_HOME` path, the AI forgot to include `import os` and `import subprocess`. This threw a secondary Python exception that masked the actual issue until I prompted it with the error.
  2. **ECS Column Syntax:** During the data normalization phase, it initially suggested selecting the renamed ECS column using `F.col("dns.question.name")`. This crashed PySpark because the engine interpreted the dots as nested JSON structures rather than a literal column string. I had to feed the error traceback back to the AI to get the corrected backtick syntax (``F.col("`dns.question.name`")``).