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
conda create --name mlops-ds python=3.8 -y
```


3. Activate the conda environment

```bash
conda activate mlops-ds
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

## Task 2. Run Kedro pipelines on a Dataproc cluster in a YARN-client mode

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

3. Run the Kedro pipeline using `yarn-dev` (ensure that `DEV_BUCKET` environment variable is set correctly)

```bash
kedro run --env=yarn-dev
```
4. Check the experiment runs in MLflow 
5. Setup tunneling to the Dataproc cluster and port forwarding for YARN console and
Spark Application UI (VSCode automatically detects 4040 port).
Hint: Workshop 1
![img.png](doc/figures/kedro-pyspark-yarn.png)
![img.png](doc/figures/spark-ui-yarn.png)


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

Important: Take the screenshot of the Kedro pipeline visualization to the documentation.

## Task 4. Create a new Kedro environment for the Dataproc cluster

1. Create an additional bucket for new Kedro environment

```bash
export PRD_MLOPS_ENV=yarn-prd
export PRD_BUCKET=gs://ds-mlops-${PRD_MLOPS_ENV}-${USER_ID}
gsutil mb -l europe-west1 $PRD_BUCKET
```

2. Copy data from the `yarn-dev` bucket to the `yarn-prd` bucket

Before executing this command check if the `DEV_BUCKET` and `PRD_BUCKET` environment variables are set correctly.
```bash
gsutil cp -r ${DEV_BUCKET}/data/01_raw/* ${PRD_BUCKET}/data/01_raw/
```

3. Create a new configuration for `yarn-prd` environment

- in the `adac-kedro-psypark/conf` create new `yarn-prd` directory
- copy the content of the `yarn-dev` directory to the `yarn-prd` directory
- change the parameters in files of the `yarn-prd` directory to:
  - point to the new bucket
  - increase the number of executors (to 2)
  - increase the driver memory (to 4GB) and executor memory (to 2GB)
  - change the `random_state` to the dirfferent value and the `test_size` to 0.15

Hint: Configuration files in new directory inherit after the `base` directory. You can overwrite the parameters in the new directory.

Important: Gather changed code snippets to the documentation

## Task 5. Run model training and inference on the Dataproc cluster

1. Run the Kedro pipeline using `yarn-prd` environment

```bash
kedro run --env=yarn-prd
```

2. Check the experiment runs in MLflow Experiment tracking UI

To open the MLflow UI in the JupyterLab environment, click on the MLFlow card in the Launcher tab.

Important: Take the screenshot of all runs to the documentation.

3. Register the production model in MLflow Model Registry UI

To register the model, you need to click the *Register* button in the detailed model view, in the MLflow UI.

Name of the model: `ds-kedro-model`

4. Prepare the batch inference pipeline

- create a new pipeline folder in the `src/adac_kedro_pyspark` directory called `batch_inference`
- copy the code snippet for Pandas predictions from the MLflow UI (*Artifacts* tab) to the `inference_model()` function in `nodes.py`
- in the configuration of used environment (can be for example `yarn-prd`), add the parameters of the new pipeline to `parameters_batch_inference.yml`
- create 3 nodes in the `batch_inference` pipeline:
  - `get_inference_data` - to get X array for making predictions from the `01_raw` directory in a randomized way
  - `inference_model` - to predict the target values using the loaded model
  - `save_to_file` - to save the predictions to the `07_model_output` directory
- connect the nodes in the `batch_inference` pipeline in the `pipeline.py` file (you can use the input variables from outputs of other pipelines)
- run the batch inference using the `kedro run --env=yarn-prd --pipeline=batch_inference` command

Important: Take the screenshot of the batch inference run to the documentation.

## Task 6. Send the prepared documentation

Please send the prepared PDF documentation in the Microsoft Teams assignment.

## Task 7. Destroy the infrastructure

Important step!