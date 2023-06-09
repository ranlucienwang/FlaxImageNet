## ImageNet classification

Trains a ResNet50 model ([He *et al.*, 2016]) for the ImageNet classification task
([Russakovsky *et al.*, 2015]).

This example uses linear learning rate warmup and cosine learning rate schedule.

[He *et al.*, 2016]: https://arxiv.org/abs/1512.03385
[Russakovsky *et al.*, 2015]: https://arxiv.org/abs/1409.0575

You can run this code and even modify it directly in Google Colab, no
installation required:

https://colab.research.google.com/github/google/flax/blob/main/examples/imagenet/imagenet.ipynb

The Colab also demonstrates how to load pretrained checkpoints from Cloud
storage at
[gs://flax_public/examples/imagenet/](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet)

Table of contents:

- [Requirements](#requirements)
- [Example runs](#example-runs)
- [Running locally](#running-locally)
  - [Overriding parameters on the command line](#overriding-parameters-on-the-command-line)
- [Running on Cloud](#running-on-cloud)
  - [Preparing the dataset](#preparing-the-dataset)
  - [Google Cloud TPU](#google-cloud-tpu)
  - [Google Cloud GPU](#google-cloud-gpu)

### Requirements

* TensorFlow dataset `imagenet2012:5.*.*`
* `≈180GB` of RAM if you want to cache the dataset in memory for faster IO

### Example runs

While the example should run on a variety of hardware,
we have tested the following GPU and TPU configurations:

|          Name           | Steps  | Walltime | Top-1 accuracy |                                                                       Metrics                                                                        |                                                                               Workdir                                                                                |
| :---------------------- | -----: | :------- | :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TPU v3-32                | 125100 | 2.1h     | 76.54%         | [tfhub.dev](https://tensorboard.dev/experiment/GhPHRoLzTqu7c8vynTk6bg/)                                                                              | [gs://flax_public/examples/imagenet/tpu_v3_32](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet/tpu_v3_32)                                         |
| TPU v2-32                | 125100 | 2.5h     | 76.67%         | [tfhub.dev](https://tensorboard.dev/experiment/qBJ7T9VPSgO5yeb0HAKbIA/)                                                                              | [gs://flax_public/examples/imagenet/tpu_v2_32](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet/tpu_v2_32)                                         |
| TPU v3-8                | 125100 | 4.4h     | 76.37%         | [tfhub.dev](https://tensorboard.dev/experiment/JwxRMYrsR4O6V6fnkn3dmg/)                                                                              | [gs://flax_public/examples/imagenet/tpu](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet/tpu)                                         |
| v100_x8                 | 250200 | 13.2h    | 76.72%         | [tfhub.dev](https://tensorboard.dev/experiment/venzpsNXR421XLkvvzSkqQ/#scalars&_smoothingWeight=0&regexInput=%5Eimagenet/v100_x8%24)                 | [gs://flax_public/examples/imagenet/v100_x8](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet/v100_x8)                                 |
| v100_x8_mixed_precision |  62500 | 4.3h     | 76.27%         | [tfhub.dev](https://tensorboard.dev/experiment/venzpsNXR421XLkvvzSkqQ/#scalars&_smoothingWeight=0&regexInput=%5Eimagenet/v100_x8_mixed_precision%24) | [gs://flax_public/examples/imagenet/v100_x8_mixed_precision](https://console.cloud.google.com/storage/browser/flax_public/examples/imagenet/v100_x8_mixed_precision) |


### Running locally

```shell
python main.py --workdir=./imagenet --config=configs/v100_x8.py
```

#### Overriding parameters on the command line

Specify a hyperparameter configuration by the means of setting `--config` flag.
Configuration flag is defined using
[config_flags](https://github.com/google/ml_collections/tree/master#config-flags).
`config_flags` allows overriding configuration fields. This can be done as
follows:

```shell
python main.py --workdir=./imagenet_default --config=configs/default.py \
--config.num_epochs=100
```

### Running on Cloud

#### Preparing the dataset

For running the ResNet50 model on imagenet dataset,
you first need to prepare the `imagenet2012` dataset.
Download the data from http://image-net.org/ as described in the
[tensorflow_datasets catalog](https://www.tensorflow.org/datasets/catalog/imagenet2012).
Then point the environment variable `$IMAGENET_DOWNLOAD_PATH`
to the directory where the downloads are stored and prepare the dataset
by running

```shell
python -c "
import tensorflow_datasets as tfds
tfds.builder('imagenet2012').download_and_prepare(
    download_config=tfds.download.DownloadConfig(
        manual_dir="/root/tensorflow_datasets"))
"
```

The contents of the directory `~/tensorflow_datasets` should be copied to your
gcs bucket. Point the environment variable `GCS_TFDS_BUCKET` to your bucket and
run the following command:

```shell
gsutil cp -r ~/tensorflow_datasets gs://$GCS_TFDS_BUCKET/datasets
```

#### To Run the Program on Google Cloud TPU VM

1. Create VM using the following command 
```
gcloud compute tpus tpu-vm create tpu-name \
--zone=europe-west4-a \
--accelerator-type=v3-8 \
--version=tpu-vm-tf-2.12.0
```
2. SSH into the TPU VM
```
gcloud compute tpus tpu-vm ssh tpu-name \
  --zone europe-west4-a
```
3. Install requirements using _requirements_tpu.txt_
4. Perform additional installation steps
```
pip3 install numpy --upgrade
pip3 install jax[tpu] -f https://storage.googleapis.com/jax-releases/libtpu_releases.html
```
5. Run the main command
```
python3 main.py --workdir=./imagenet_default --config=configs/tpu.py 
```