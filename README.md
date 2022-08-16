Modeling Swiss farmer's attitudes about climate change

[Dataset](https://www.sciencedirect.com/science/article/pii/S2352340920303048)


1. Get dataset from [Dataset](https://www.sciencedirect.com/science/article/pii/S2352340920303048) with python
    ``` bash
    python get_data.py
    ```

2. Process data with python
    ``` bash
    python process_data.py
    ```

3. Train
    ``` bash
    python train.py
    ```

4. Build Pipeline
    ``` bash
    dvc init
    dvc run -n get_data -d get_data.py -o data_raw.csv --no-exec python get_data.py
    ```
5. Edit `dvc.yaml`
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

6. dvc repro
    ``` bash
    dvc repro
    ```