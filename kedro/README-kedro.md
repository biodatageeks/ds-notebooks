# TBD Workshop 2. Kedro
[Spaceflights tutorial](https://docs.kedro.org/en/stable/tutorial/spaceflights_tutorial.html)

## Workshop goals

1. Learn how to use Kedro for building reproducible, maintainable, and modular data pipelines.
2. Learn how to use PySpark (local mode and deployed on Kubernetes) for data processing together with Kedro.
3. Learn how to use MLflow for tracking experiments and managing machine learning models.
4. Learn how to use Kedro-Viz for visualizing the data pipeline.

## Prerequisites
* Access to the GitHub account
* Web browser, Google Chrome preferred
* Run labs in the VSCode server available in the Launchpad of your Jupyter Lab in lab.biodatageeks.com


## Task 1. Setup Kedro project, run the pipeline locally 

1. Initialize Anaconda in terminal

```bash
conda init
```
Restart the bash session after running the above command.

2. Create Anaconda environment (Python 3.8 because of the compatibility with Python running on Dataproc cluster)

```bash 
conda create -p /home/jovyan/envs/mlops-ds python=3.8 -y
```


3. Activate the conda environment

```bash
conda activate /home/jovyan/envs/mlops-ds
```

4. Install Kedro

```bash
pip install kedro==0.19.3
```

5. Create a new Kedro project (we specify the starter template from the GitHub repository and the directory name from a specific branch)

```bash
kedro new --starter  https://github.com/mwiewior/kedro-starters/ --directory spaceflights-pyspark-mlflow --checkout spaceflights-pyspark-mlflow
```

Set the name for your new project: `ds-kedro-pyspark`

6. Change the directory to the newly created project

```bash
cd ds-kedro-pyspark
```

7. Configure required environment variables

```bash
# workaround for the issue with PySpark and threading for the local mode an logging MLflow runs
export PYSPARK_PIN_THREAD=false
```

8. Install the project dependencies

```bash
pip install -r requirements.txt
```

This step will last for ~10 minutes. Meanwhile, you can explore the kedro project structure and content, and continue with the next steps.

9. Run the MLflow UI

For running the Mlflow instance, you need to click the *MLflow* card in the Launcher tab in the JupyterLab environment.

10. Run the Kedro pipeline

```bash
kedro run
```

11. Check experiment runs in MLflow

12. What Kedro environment is used by default? How to change it?

## Task 2. Run Kedro pipelines on a Kubernetes cluster in a K8S-client mode

1. Create a new bucket in the same region as the rest of the infrastructure

```bash
# change the USER_ID to your username
export USER_ID=mwiewior
export DEV_MLOPS_ENV=k8s-dev
export DEV_BUCKET=gs://ds-mlops-${DEV_MLOPS_ENV}-${USER_ID}
gsutil mb -l europe-west1 $DEV_BUCKET
```
2. Copy the raw data to the newly created bucket

```bash
gsutil cp -r data/01_raw/* ${DEV_BUCKET}/data/01_raw/
```

3. Run the Kedro pipeline using `k8s-dev` (ensure that `DEV_BUCKET` environment variable is set correctly)

```bash
kedro run --env=k8s-dev
```
4. Check the experiment runs in MLflow 


## Task 3. Visualize the Kedro pipeline
1. Run from the command line
```bash
kedro viz
```
2. After around 1 minute, you should see a pop-up window from VSCode prompting you allow port forwarding for port `4141`.
3. Click on the `Open Browser` button in the pop-up window.

![img.png](doc/figures/kedro-viz-popup.png)

Hint: there might another pop-up window from VSCode asking you to allow the port forwarding for port 4040 (Spar Application UI). 

5. To stop kedro viz press `Ctrl+C` in the terminal where you run the `kedro viz` command.

## Task 4. Create a new Kedro environment for the Kubernetes cluster

1. Create an additional bucket for new Kedro environment

```bash
export PRD_MLOPS_ENV=k8s-prd
export PRD_BUCKET=gs://ds-mlops-${PRD_MLOPS_ENV}-${USER_ID}
gsutil mb -l europe-west1 $PRD_BUCKET
```

2. Copy data from the `k8s-dev` bucket to the `k8s-prd` bucket

Before executing this command check if the `DEV_BUCKET` and `PRD_BUCKET` environment variables are set correctly.
```bash
gsutil cp -r ${DEV_BUCKET}/data/01_raw/* ${PRD_BUCKET}/data/01_raw/
```

3. Create a new configuration for `k8s-prd` environment

- in the `ds-kedro-psypark/conf` create new `k8s-prd` directory
- copy the content of the `k8s-dev` directory to the `k8s-prd` directory
- change the parameters in files of the `k8s-prd` directory to:
  - point to the new bucket
  - increase the number of executors (to 2)
  - change the `random_state` to the dirfferent value and the `test_size` to 0.15

Hint: Configuration files in new directory inherit after the `base` directory. You can overwrite the parameters in the new directory.

## Task 5. Run model training and inference on the Dataproc cluster

1. Run the Kedro pipeline using `k8s-prd` environment

```bash
kedro run --env=k8s-prd
```

2. Check the experiment runs in MLflow Experiment tracking UI

To open the MLflow UI in the JupyterLab environment, click on the MLFlow card in the Launcher tab.

1. Register the production model in MLflow Model Registry UI

To register the model, you need to click the *Register* button in the detailed model view, in the MLflow UI.

Name of the model: `ds-kedro-model`
