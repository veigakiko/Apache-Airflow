# Apache-Airflow

________________________________________________

Step 1: Stop Running Airflow Services

Stop the Airflow Webserver:
pkill -f "airflow webserver"

Stop the Airflow Scheduler:
pkill -f "airflow scheduler"


Check for Other Airflow Processes: Ensure no other Airflow processes (like workers, if using CeleryExecutor) are running:
pkill -f "airflow"
________________________________________________


Step 2: Remove Apache Airflow 
Deactivate the Virtual Environment:
deactivate



