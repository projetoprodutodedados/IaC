## Instructions:
* At the `/docker-compose` directory, create directories for Airflow DAGs, logs, and plugins:
```bash
mkdir -p ./dags  ./plugins ./logs./config
```
* Create a `.env` file on the `/docker-compose` directory:
```bash
vim .env
```

Then, press `esc` and type `:wq` , then press `enter` to save and close the file.

* Generate a Fernet key for Airflow and place it as `AIRFLOW__CORE__FERNET_KEY` environment variable at the `.env` file:
```bash
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

The `.env` file has to look like this:
```
AIRFLOW__CORE__FERNET_KEY='{{GeneratedFernetKey}}'
```

* Run the Docker Compose setup:
```bash
docker-compose up
```

This setup will bring up the services and initialize the Airflow database. You can access the Airflow web interface at http://localhost:8080 with the username airflow and password airflow. Let me know if you need any further instructions!