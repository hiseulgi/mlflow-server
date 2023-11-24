# MLflow On-Premise Deployment using Docker Compose
Easily deploy an MLflow tracking server with 1 command.

MinIO S3 is used as the artifact store and MySQL server is used as the backend store.

## How to run

1. Clone (download) this repository

    ```bash
    git clone https://github.com/sachua/mlflow-docker-compose.git
    ```

2. `cd` into the `mlflow-docker-compose` directory

3. Build and run the containers with `docker-compose`

    ```bash
    docker-compose up -d --build
    ```

4. Access MLflow UI with http://localhost:5000

5. Access MinIO UI with http://localhost:9000

## Containerization

The MLflow tracking server is composed of 4 docker containers:

* MLflow server
* MinIO object storage server
* MySQL database server

## Example

1. Install [conda](https://conda.io/projects/conda/en/latest/user-guide/install/index.html)

2. Install MLflow with extra dependencies, including scikit-learn

    ```bash
    pip install mlflow[extras]
    ```

3. Set environmental variables

    ```bash
    export MLFLOW_TRACKING_URI=http://localhost:5000
    export MLFLOW_S3_ENDPOINT_URL=http://localhost:9000
    ```
4. Set MinIO credentials

    ```bash
    cat <<EOF > ~/.aws/credentials
    [default]
    aws_access_key_id=minio
    aws_secret_access_key=minio123
    EOF
    ```

5. Train a sample MLflow model

    ```bash
    mlflow run https://github.com/mlflow/mlflow-example.git -P alpha=0.42
    ```

    * Note: To fix ModuleNotFoundError: No module named 'boto3'

        ```bash
        #Switch to the conda env
        conda env list
        conda activate mlflow-3eee9bd7a0713cf80a17bc0a4d659bc9c549efac #replace with your own generated mlflow-environment
        pip install boto3
        ```

 6. Serve the model (replace with your model's actual path)
    ```bash
    mlflow models serve -m S3://mlflow/0/98bdf6ec158145908af39f86156c347f/artifacts/model -p 1234
    ```

 7. You can check the input with this command
    ```bash
    curl -X POST -H "Content-Type:application/json; format=pandas-split" --data '{"columns":["alcohol", "chlorides", "citric acid", "density", "fixed acidity", "free sulfur dioxide", "pH", "residual sugar", "sulphates", "total sulfur dioxide", "volatile acidity"],"data":[[12.8, 0.029, 0.48, 0.98, 6.2, 29, 3.33, 1.2, 0.39, 75, 0.66]]}' http://127.0.0.1:1234/invocations
    ```

## Personal Note

>This note is based on changes to the `.env` and `docker-compose.yml` files. The changes to the Minio Access Key must first be made in the Minio Console.

### Jupyter Notebook Usage

1. Make Minio Access Keys on [Minio](http://localhost:9001/access-keys), then save access key id and secret access key.

2. Import environment on notebook

``` python
%env MLFLOW_TRACKING_URI=http://localhost:5000
%env MLFLOW_S3_ENDPOINT_URL=http://localhost:9000
%env AWS_ACCESS_KEY_ID=2vSrPs21nZYaUQvovgRL
%env AWS_SECRET_ACCESS_KEY=yCqF29KU1qbykEnsceWMDRNvPelgGAVBmyD6PeU5
```

3. Check it again and setup experiment name

``` python
import os
import mlflow

assert "MLFLOW_TRACKING_URI" in os.environ
assert "MLFLOW_S3_ENDPOINT_URL" in os.environ
assert "AWS_ACCESS_KEY_ID" in os.environ
assert "AWS_SECRET_ACCESS_KEY" in os.environ

# you can also use this method for set tracking uri, instead using environment
mlflow.set_tracking_uri("http://localhost:5000/")
mlflow.set_experiment("nyc-taxi")
```

4. Use with statement in trainer code

``` python
with mlflow.start_run():
    # your trainer code
```