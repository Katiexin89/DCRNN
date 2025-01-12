# Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting

![Diffusion Convolutional Recurrent Neural Network](figures/model_architecture.jpg "Model Architecture")

This is a TensorFlow implementation of Diffusion Convolutional Recurrent Neural Network in the following paper: \
Yaguang Li, Rose Yu, Cyrus Shahabi, Yan Liu, [Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting](https://arxiv.org/abs/1707.01926), ICLR 2018.

## Requirements
- scipy>=0.19.0
- numpy>=1.12.1
- pandas>=0.19.2
- pyaml
- statsmodels
- tensorflow>=1.3.0


Dependency can be installed using the following command:
```bash
pip install -r requirements.txt
```

## Data Preparation
The traffic data files for Los Angeles (METR-LA) and the Bay Area (PEMS-BAY), i.e., `metr-la.h5` and `pems-bay.h5`, are available at [Google Drive](https://drive.google.com/open?id=10FOTa6HXPqX8Pf5WRoRwcFnW9BrNZEIX) or [Baidu Yun](https://pan.baidu.com/s/14Yy9isAIZYdU__OYEQGa_g), and should be
put into the `data/` folder.
The `*.h5` files store the data in `panads.DataFrame` using the `HDF5` file format. Here is an example:

|                     | sensor_0 | sensor_1 | sensor_2 | sensor_n |
|:-------------------:|:--------:|:--------:|:--------:|:--------:|
| 2018/01/01 00:00:00 |   60.0   |   65.0   |   70.0   |    ...   |
| 2018/01/01 00:05:00 |   61.0   |   64.0   |   65.0   |    ...   |
| 2018/01/01 00:10:00 |   63.0   |   65.0   |   60.0   |    ...   |
|         ...         |    ...   |    ...   |    ...   |    ...   |


Here is an article about [Using HDF5 with Python](https://medium.com/@jerilkuriakose/using-hdf5-with-python-6c5242d08773).

Run the following commands to generate train/test/val dataset at  `data/{METR-LA,PEMS-BAY}/{train,val,test}.npz`.
```bash
# Create data directories (linux)
mkdir -p data/{METR-LA,PEMS-BAY} 
# Create data directories (windows)
cd data
md METR-LA
md PEMS-BAY

# METR-LA
python -m scripts.generate_training_data --output_dir=data/METR-LA --traffic_df_filename=data/metr-la.h5
# output
Generating training data
x shape:  (34249, 12, 207, 2) , y shape:  (34249, 12, 207, 2)
train x:  (23974, 12, 207, 2) y: (23974, 12, 207, 2)
val x:  (3425, 12, 207, 2) y: (3425, 12, 207, 2)
test x:  (6850, 12, 207, 2) y: (6850, 12, 207, 2)

# PEMS-BAY
python -m scripts.generate_training_data --output_dir=data/PEMS-BAY --traffic_df_filename=data/pems-bay.h5
# output
Generating training data
x shape:  (52093, 12, 325, 2) , y shape:  (52093, 12, 325, 2)
train x:  (36465, 12, 325, 2) y: (36465, 12, 325, 2)
val x:  (5209, 12, 325, 2) y: (5209, 12, 325, 2)
test x:  (10419, 12, 325, 2) y: (10419, 12, 325, 2)

```

## Graph Construction
 As the currently implementation is based on pre-calculated road network distances between sensors, it currently only
 supports sensor ids in Los Angeles (see `data/sensor_graph/sensor_info_201206.csv`).
```bash
python -m scripts.gen_adj_mx  --sensor_ids_filename=data/sensor_graph/graph_sensor_ids.txt --normalized_k=0.1\
    --output_pkl_filename=data/sensor_graph/adj_mx.pkl
```
Besides, the locations of sensors in Los Angeles, i.e., METR-LA, are available at [data/sensor_graph/graph_sensor_locations.csv](https://github.com/liyaguang/DCRNN/blob/master/data/sensor_graph/graph_sensor_locations.csv), and the locations of sensors in PEMS-BAY are available at [data/sensor_graph/graph_sensor_locations_bay.csv](https://github.com/liyaguang/DCRNN/blob/master/data/sensor_graph/graph_sensor_locations_bay.csv).

## Run the Pre-trained Model on METR-LA

```bash
# METR-LA
python run_demo.py --config_filename=data/model/pretrained/METR-LA/config.yaml
# output
Total number of trainable parameters: 372352
- INFO - Horizon 01, MAE: 2.17, MAPE: 0.0517, RMSE: 3.76
- INFO - Horizon 02, MAE: 2.47, MAPE: 0.0612, RMSE: 4.59
- INFO - Horizon 03, MAE: 2.66, MAPE: 0.0683, RMSE: 5.16
- INFO - Horizon 04, MAE: 2.82, MAPE: 0.0742, RMSE: 5.61
- INFO - Horizon 05, MAE: 2.95, MAPE: 0.0792, RMSE: 5.98
- INFO - Horizon 06, MAE: 3.06, MAPE: 0.0836, RMSE: 6.28
- INFO - Horizon 07, MAE: 3.16, MAPE: 0.0875, RMSE: 6.54
- INFO - Horizon 08, MAE: 3.25, MAPE: 0.0910, RMSE: 6.77
- INFO - Horizon 09, MAE: 3.33, MAPE: 0.0941, RMSE: 6.97
- INFO - Horizon 10, MAE: 3.40, MAPE: 0.0970, RMSE: 7.15
- INFO - Horizon 11, MAE: 3.47, MAPE: 0.0996, RMSE: 7.31
- INFO - Horizon 12, MAE: 3.54, MAPE: 0.1024, RMSE: 7.48
Predictions saved as data/dcrnn_predictions.npz.

# PEMS-BAY
python run_demo.py --config_filename=data/model/pretrained/PEMS-BAY/config.yaml
# output
Total number of trainable parameters: 372352
- INFO - Horizon 01, MAE: 0.85, MAPE: 0.0163, RMSE: 1.53
- INFO - Horizon 02, MAE: 1.12, MAPE: 0.0225, RMSE: 2.22
- INFO - Horizon 03, MAE: 1.31, MAPE: 0.0273, RMSE: 2.75
- INFO - Horizon 04, MAE: 1.45, MAPE: 0.0312, RMSE: 3.17
- INFO - Horizon 05, MAE: 1.56, MAPE: 0.0343, RMSE: 3.49
- INFO - Horizon 06, MAE: 1.64, MAPE: 0.0369, RMSE: 3.74
- INFO - Horizon 07, MAE: 1.72, MAPE: 0.0392, RMSE: 3.94
- INFO - Horizon 08, MAE: 1.78, MAPE: 0.0411, RMSE: 4.11
- INFO - Horizon 09, MAE: 1.83, MAPE: 0.0426, RMSE: 4.25
- INFO - Horizon 10, MAE: 1.88, MAPE: 0.0440, RMSE: 4.37
- INFO - Horizon 11, MAE: 1.92, MAPE: 0.0451, RMSE: 4.47
- INFO - Horizon 12, MAE: 1.96, MAPE: 0.0462, RMSE: 4.55
Predictions saved as data/dcrnn_predictions.npz.

```
The generated prediction of DCRNN is in `data/results/dcrnn_predictions`.


## Model Training

Here are commands for training the model on `METR-LA` and `PEMS-BAY` respectively. 

```bash
# METR-LA
python dcrnn_train.py --config_filename=data/model/dcrnn_la.yaml

# PEMS-BAY
python dcrnn_train.py --config_filename=data/model/dcrnn_bay.yaml
```
### Training details and tensorboard links
With a single GTX 1080 Ti, each epoch takes around 5min for `METR-LA`, and 13 min for `PEMS-BAY` respectively. Here are example tensorboard links for [DCRNN on METR-LA](https://tensorboard.dev/experiment/ijwg04waSOWQ2Pj4mZ3tAg), [DCRNN on PEMS-BAY](https://tensorboard.dev/experiment/QzJtnMfgQJCQ7vc7wNJjxg), including training details and metrics over time.

Note that, there is a chance of training loss explosion, one temporary workaround is to restart from the last saved model before the explosion, or to decrease the learning rate earlier in the learning rate schedule. 

### Metric for different horizons and datasets
The following table summarizes the performance of DCRNN on two dataset with regards to different metrics and horizons (numbers are better than those reported in the paper due to bug fix in commit [2e4b8c8](https://github.com/liyaguang/DCRNN/commit/2e4b8c868fd410a1fb4a469f0995de6616115e03) on Oct 1, 2018).

| Dataset  | Metric | 5min  | 15min | 30min | 60min  |
|----------|--------|-------|-------|-------|--------|
| METR-LA  | MAE    | 2.18  | 2.67  | 3.08  | 3.56   |
|          | MAPE   | 5.17% | 6.84% | 8.38% | 10.30% |
|          | RMSE   | 3.77  | 5.17  | 6.3   | 7.52   |
| PEMS-BAY | MAE    | 0.85  | 1.31  | 1.66  | 1.98   |
|          | MAPE   | 1.63% | 2.74% | 3.76% | 4.74%  |
|          | RMSE   | 1.54  | 2.76  | 3.78  | 4.62   |


## Eval baseline methods
```bash
# METR-LA
python -m scripts.eval_baseline_methods --traffic_reading_filename=data/metr-la.h5
```
More details are being added ...


## Deploying DCRNN on Large Graphs with graph partitioning

With graph partitioning, DCRNN has been successfully deployed to forecast the traffic of the entire California highway network with **11,160** traffic sensor locations simultaneously. The general idea is to partition the large highway network into a number of small networks, and trained them with a share-weight DCRNN simultaneously. The training process takes around 3 hours in a moderately sized GPU cluster, and the real-time inference can be run on traditional hardware such as CPUs.

See the [paper](https://arxiv.org/pdf/1909.11197.pdf "GRAPH-PARTITIONING-BASED DIFFUSION CONVOLUTION RECURRENT NEURAL NETWORK FOR LARGE-SCALE TRAFFIC FORECASTING"), [slides](https://press3.mcs.anl.gov/atpesc/files/2019/08/ATPESC_2019_Track-8_11_8-9_435pm_Mallick-DCRNN_for_Traffic_Forecasting.pdf), and [video](https://www.youtube.com/watch?v=liJNNtJGTZU&list=PLGj2a3KTwhRapjzPcxSbo7FxcLOHkLcNt&index=10) by Tanwi Mallick et al. from Argonne National Laboratory for more information.

## DCRNN Applications 
In addition to vehicle traffic forecasting, DCRNN and its variants have been applied in many important domains, including:
* Neuroimaging: causal inference in brain networks. [S. Wein et al. A graph neural network framework for causal inference in brain networks. Scientific Reports, 2021](https://www.nature.com/articles/s41598-021-87411-8), GitHub [Repo](https://github.com/simonvino/DCRNN_brain_connectivity).
* Air quality forecasting: [Y Lin et al. Exploiting spatiotemporal patterns for accurate air quality forecasting using deep learning. ACM SIGSPATIAL 2018](https://dl.acm.org/doi/10.1145/3274895.3274907).
* Internet traffic forecasting: [D. Andreoletti et al. Network traffic prediction based on diffusion convolutional recurrent neural networks, INFOCOM 2019](https://ieeexplore.ieee.org/document/8845132).

## Third-party re-implementations
The Pytorch implementaion by [chnsh@](https://github.com/chnsh/) is available at [DCRNN-Pytorch](https://github.com/chnsh/DCRNN_PyTorch).


## Citation

If you find this repository, e.g., the code and the datasets, useful in your research, please cite the following paper:
```
@inproceedings{li2018dcrnn_traffic,
  title={Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting},
  author={Li, Yaguang and Yu, Rose and Shahabi, Cyrus and Liu, Yan},
  booktitle={International Conference on Learning Representations (ICLR '18)},
  year={2018}
}
```
