# Airflow Installation
Create a new environment for airflow
```bash
python3 -m venv airflow_venv
source airflow_venv/bin/activate
```

Install Airflow
```bash
export AIRFLOW_VERSION=2.10.1
export PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
export CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

Initialize airflow database
```bash
airflow db init
```

Create airflow user
```bash
airflow users create \
    --username admin \
    --password admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com
```

After the installation, start the webserver and log in with the credentials at `http://localhost:8080`
```bash
airflow werbserver --port 8080
```
