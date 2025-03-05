# 🚀 Windows Log Processing & Automation System

## 📌 Project Overview

This project automates the extraction, processing, and storage of Windows logs using:

- **PowerShell** → Extract logs from Windows Event Viewer.
- **Python** → Process logs and insert them into PostgreSQL.
- **PostgreSQL** → Store logs in a structured database.
- **Task Scheduler** → Automate the entire process daily.

---

## 🛠️ Technologies Used

- **Windows PowerShell**
- **Python (psycopg2)**
- **PostgreSQL**
- **Windows Task Scheduler**

---

## 📜 Steps Performed

### **1️⃣ Extracting Logs Using PowerShell**

We use PowerShell to extract logs from Windows Event Viewer and save them to a text file.

📌 **PowerShell Script (`extract_logs.ps1`)**:

```powershell
# PowerShell script to extract logs from Windows Event Viewer
$logFile = "C:\logs\system_logs.txt"

# Get logs from Event Viewer and save to file
Get-EventLog -LogName System -Newest 1000 | Format-Table TimeGenerated, EntryType, Message -AutoSize | Out-File -Encoding utf8 $logFile

Write-Host "✅ Logs extracted and saved to $logFile"
```
### 📌 What this does? (PowerShell Script)

- Extracts System logs (latest 1000 logs).
- Saves logs in a structured format in `C:\logs\system_logs.txt`.

### 2️⃣ Storing Logs in PostgreSQL

First, install PostgreSQL and create a database to store logs.

### 📌 Create a PostgreSQL Database (`windows_logs`):

```sql
CREATE DATABASE windows_logs;
```

### 📌 Create a PostgreSQL Database (`windows_logs`):

```sql
CREATE TABLE logs (
    id SERIAL PRIMARY KEY,
    log_time TIMESTAMP DEFAULT NOW(),
    log_level TEXT,
    log_message TEXT,
    source_file TEXT
);
```

### 📌 What this does? (PostgreSQL Table)

- Stores log level, message, timestamp, and source file for every log entry.

### 3️⃣ Processing Logs Using Python

We write a Python script to read logs from `system_logs.txt` and insert them into PostgreSQL.

### 📌 Python Script (`process_logs.py`):

```python
import psycopg2
import os

# PostgreSQL connection details
DB_NAME = "windows_logs"
DB_USER = "postgres"
DB_PASSWORD = "your_password"
DB_HOST = "localhost"
DB_PORT = "5432"

# Log file path
LOG_FILE_PATH = "C:\\logs\\system_logs.txt"

# Check if log file exists
if not os.path.exists(LOG_FILE_PATH):
    print(f"❌ Log file not found: {LOG_FILE_PATH}")
    exit(1)

try:
    print("🔄 Connecting to PostgreSQL...")
    conn = psycopg2.connect(
        database=DB_NAME,
        user=DB_USER,
        password=DB_PASSWORD,
        host=DB_HOST,
        port=DB_PORT
    )
    cur = conn.cursor()
    print("✅ Connection successful!")

    # Read logs from file and insert into database
    with open(LOG_FILE_PATH, "r", encoding="utf-8") as file:
        for line in file:
            if "Error" in line:
                log_level = "ERROR"
            elif "Warning" in line:
                log_level = "WARNING"
            elif "Information" in line:
                log_level = "INFO"
            else:
                log_level = "GENERAL"

            cur.execute(
                "INSERT INTO logs (log_time, log_level, log_message, source_file) VALUES (NOW(), %s, %s, %s)",
                (log_level, line.strip(), "system_logs.txt")
            )

    # Commit changes and close connection
    conn.commit()
    cur.close()
    conn.close()
    print("✅ Logs inserted into PostgreSQL successfully!")
except Exception as e:
    print(f"❌ Error: {e}")
```
### 📌 What this does? (Python Script)

- Reads logs from `system_logs.txt`.
- Categorizes logs into ERROR, WARNING, INFO, and GENERAL.
- Inserts logs into PostgreSQL `logs` table.

### 4️⃣ Automating with Windows Task Scheduler

We automate both PowerShell & Python scripts to run daily.

**Step 1: Schedule PowerShell Script (`extract_logs.ps1`)**

1.  Open Task Scheduler (Win + R → `taskschd.msc`).
2.  Click Create Basic Task → Name: Extract Windows Logs.
3.  Set Trigger → Daily → Pick a Time.
4.  Set Action → Start a Program → Enter:

    ```mathematica
    powershell.exe -ExecutionPolicy Bypass -File C:\logs\extract_logs.ps1
    ```

5.  Click Finish.

**Step 2: Schedule Python Script (`process_logs.py`)**

1.  Create Another Task in Task Scheduler.
2.  Name: Process Logs into PostgreSQL.
3.  Set Trigger → Daily (or after PowerShell script).
4.  Set Action → Start a Program → Enter:

    ```mathematica
    python.exe C:\logs\process_logs.py
    ```

5.  Click Finish.

### 5️⃣ Checking Logs in PostgreSQL

To check if logs are stored correctly, run:

```sql
SELECT * FROM logs;
```
###📌 What this does? (SQL Query)
- Shows the stored log entries along with their timestamps and categories.

###✅ Final Outcome
- Windows logs are extracted, processed, and stored in PostgreSQL automatically.
- Task Scheduler runs everything daily, ensuring an automated workflow.
- All log levels (INFO, WARNING, ERROR, GENERAL) are captured.

###🔥 Future Improvements
- Store logs in cloud storage (AWS, Google Cloud, Azure).
- Build a dashboard using Flask/React.js for log visualization.
- Send email alerts for critical logs.

###🎯 Summary
This project automates Windows log extraction & storage using PowerShell, Python, PostgreSQL, and Task Scheduler.
