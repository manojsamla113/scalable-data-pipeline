Scalable Data Pipeline

A runnable Python project demonstrating chunked CSV and JSON Lines ingestion, data-quality validation, customer reference checks, duplicate detection, SQLite publishing, rejected-record quarantine, reconciliation metrics, and performance benchmarking.

Requirements
Python 3.10+
Windows, macOS, or Linux
Setup in VS Code

Open the project folder in VS Code. In the integrated terminal, run the following commands.

Windows PowerShell
python -m venv .venv

.\.venv\Scripts\Activate.ps1

python -m pip install -r requirements.txt

If PowerShell blocks activation, run python -m pip install -r requirements.txt using the Python interpreter selected in VS Code.

Run CSV Example
python -m src.pipeline --input data/input/transactions.csv --format csv --chunk-size 3
Run JSONL Example
python -m src.pipeline --input data/input/transactions.jsonl --format jsonl --chunk-size 2
Generate a Larger CSV

Generate 100,000 transaction records:

python -m src.generator --rows 100000 --output data/input/transactions.csv

The generator uses customer IDs C0001–C1000. Before running the generated data, regenerate the customer reference file with IDs C0001–C1000:

@("customer_id") + (1..1000 | ForEach-Object { "C{0:D4}" -f $_ }) | Set-Content -Encoding utf8 data/input/customers.csv
Run Tests
python -m pytest -v
Benchmark

Compare performance using different chunk sizes:

python -m src.benchmark --rows 100000 --chunk-sizes 1000 5000 10000

Benchmark results are saved to:

reports/benchmark_results.json
reports/benchmark_results.csv
Outputs
Output	Description
data/output/transactions.db	SQLite target database and pipeline run metadata
data/rejects/rejected_records.csv	Invalid and malformed records, with rejection details where supported
data/rejects/rejected_records.xlsx	Formatted Excel report of rejected records, if generated using the Excel exporter
reports/reconciliation.json	Pipeline metrics and reconciliation result
Validation Rules

The pipeline applies the following validation rules:

Required fields: transaction_id, customer_id, amount, status, and event_timestamp.
Amount must be numeric and greater than zero.
Status must be SUCCESS, PENDING, or FAILED.
Timestamp must parse as a valid date/time.
Customer ID must exist in customers.csv.
Duplicate transaction IDs within the same input run are rejected.
The target database's primary key prevents inserting an existing transaction ID twice.
Memory Behavior and Limitations
CSV input is read in chunks using pandas chunksize.
JSONL input is read line by line.
Duplicate IDs for the current run are tracked in SQLite rather than a Python set.
Current-chunk DataFrames still use memory proportional to the configured chunk size.
Malformed JSONL lines are quarantined individually.
Malformed CSV parsing errors currently fail the run rather than being quarantined individually.
SQLite writes are committed by chunk. If a later chunk fails, previously committed chunks remain in the target database.
Benchmark RSS values are sampled and approximate; they are not a formal peak-memory guarantee.
Reconciliation checks that input_records = accepted_records + rejected_records.
published_records counts new database inserts and can be lower on reruns because existing records are not inserted again.
Idempotency

The pipeline is designed to prevent duplicate transaction IDs from being inserted into the target database when the same input is processed again.

On a rerun, published_records may be zero even when valid records are processed, because those records may already exist in the database.

Rejected-Record Reporting

Rejected records are written to:

data/rejects/rejected_records.csv

The rejection report includes source information, the original record, and the rejection reason when available.

To generate the formatted Excel report, run:

python -m src.export_rejects_excel

The Excel report is saved to:

data/rejects/rejected_records.xlsx

To open it in Windows:

Invoke-Item data\rejects\rejected_records.xlsx
Project Goals

This project demonstrates:

Chunked data ingestion.
Data validation and quality checks.
Customer reference validation.
Duplicate detection and idempotent publishing.
Rejected-record quarantine.
SQLite storage.
Reconciliation metrics.
Automated tests.
Performance benchmarking.
CSV and Excel reporting.
