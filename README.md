# Modeling Swiss farmer's attitudes about climate change

[Dataset](https://www.sciencedirect.com/science/article/pii/S2352340920303048)


1. Get dataset from [Dataset](https://www.sciencedirect.com/science/article/pii/S2352340920303048) with python
    ``` bash
    python get_data.py
    ```
    ![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/Images/get_data.png)

2. Process data with python
    ``` bash
    python process_data.py
    ```
    ![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/Images/process_data.png)

3. Train
    ``` bash
    python train.py
    ```
    ![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/Images/train_result.png)

4. Build Pipeline
    ``` bash
    dvc init
    dvc run -n get_data -d get_data.py -o data_raw.csv --no-exec python get_data.py
    ```
    ![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/Images/dvc_run.png)

    Edit `dvc.yaml`
    ``` yaml:dvc.yaml
    stages:
        get_data:
            cmd: python get_data.py
            deps:
            - get_data.py
            outs:
            - data_raw.csv

        process:
            cmd: python process_data.py
            deps:
            - process_data.py
            - data_raw.csv
            outs:
            - data_processed.csv

        train:
            cmd: python train.py
            deps:
            - train.py
            - data_processed.csv
            outs:
            - by_region.png
            metrics:
            - metrics.json:
                cache: false
    ```
    
    dvc repro
    ``` bash
    dvc repro
    ```
    ![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/Images/dvc_repro.png)

    git add, commit & push
    ```bash
    git add .
    git commit -m "setup dvc pipelines"
    git push
    ```

5. Create .github/workflows/train.yaml
   ```yaml
   name: farmer
    on: [push]
    jobs:
    run:
        runs-on: [ubuntu-latest]
        container: docker://dvcorg/cml-py3:latest
        steps:
        - uses: actions/checkout@v2
        - name: cml_run
            env:
            repo_token: ${{ secrets.GITHUB_TOKEN }}
            run: |
            pip install -r requirements.txt
            dvc repro 
            git fetch --prune
            dvc metrics diff --show-md main > report.md
            
            # Add figure to the report
            echo "## Validating results by region"
            cml-publish by_region.png --md >> report.md
            cml-send-comment report.md
    ```

    git add, commit & push
    ```bash
    git add .
    git commit -m "Make CML workflow"
    git push
    ```

## Result 
You can see [here](https://github.com/fiqgant/MLOps-Farmer/commit/e217475ec059857fbad059f81d8d62b1d5353739#commitcomment-81294769)
| Path         | Metrics     | LogisticRegression | QuadraticDiscriminantAnalysis | Change   |
|--------------|-------------|--------------------|-------------------------------|----------|
| metrics.json | accuracy    | 0.86667            | 0.70476                       | -0.1619  |
| metrics.json | sensitivity | 0.95506            | 0.78652                       | -0.16854 |
| metrics.json | specificity | 0.375              | 0.25                          | -0.125   |

## Plot
### LogisticRegression
![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/by_region_lr.png)
### QuadraticDiscriminantAnalysis
![](https://raw.githubusercontent.com/fiqgant/MLOps-Farmer/main/by_region_qda.png)
