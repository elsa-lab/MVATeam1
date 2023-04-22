# Execution

## 0. Hardware Requirement
- Before running our model, please ensure that your GPU has at least **20 GB** of VRAM.
- Our environment has been deployed into a Docker image that requires the NVIDIA Container Toolkit. The Docker image is based on CUDA 11.3, please make sure that your GPU driver version is at least **465.19.01**.

It is recommended to execute our code using V100 32GB or RTX A6000, as we have tested the following scripts and docker images on these devices.

## 0.5 Hardware Requirement (Parallel)
We offer a parallelized version of our inference script, which can significantly reduce the processing time. However, please note that this version requires a higher VRAM space. **Each** GPU should have at least **26 GB** of VRAM.

## 1. Environment Preparation (MVATeam1)

Please clone our GitHub repository:

```bash
git clone https://github.com/Howardkhh/MVATeam1.git
cd MVATeam1
```

Before starting the docker container, please link the data folder to our repository:

```bash
ln -s <absolute path to the data folder> ./data
```

The docker container can be launched by:

```bash
make start
```

The docker image is built with the `Dockerfile` within the root of our repository, and has been uploaded to the Dockerhub. Invoking `make start` will also download the image.

As inferencing using MMDetection requires a larger shared memory space (`/dev/shm`), we set the Docker container to have 32GB of shared memory space. The size could be changed by editing the `SHMEM_SIZE` variable in the `Makefile`.

After launching the Docker container, the working directory should be the `MVATeam1` folder. Next, please install the MMDetection development package inside the container by:

```bash
make post_install
```

## 2. Model Weights

Our model weights are too large to be uploaded to GitHub. Please download the pre-trained weights from Google Drive:

```bash
gdown --folder https://drive.google.com/drive/folders/1zNZTGmlRVsPSpxrVwpik17I2w3XLbqDc?usp=share_link
```

A single folder named `final` should be downloaded to the root of our repository. 

In case of download quota exceeded, we provide another link. Please make sure the tarball is properly extracted.
```bash
gdown 'https://drive.google.com/u/3/uc?id=1fRW-3CVf3t5EQUjYfh6BQN27VvTRTn9a&export=download'
tar zxvf final.tar.gz
```

**Important:** Please ensure that **all files** are fully downloaded (the progress bars should be at 100%). 

The size of the `final` folder should be 36 GB.
```bash
du -sh final
# output: 36G        final
```

## 2.25 (Optional) Download Our Data

We use an auxiliary dataset (https://www.kaggle.com/datasets/nelyg8002000/birds-flying) to augment our data during training. We have uploaded our dataset to the Google Drive. For detailed usage, please refer to the class: `MVAPasteBirds` in `MVATeam1/mmdet/datasets/pipelines/transforms.py`. It can be downloaded with the following commands:

```bash
cd data
gdown --fuzzy https://drive.google.com/file/d/1dcPNszz3h7Ntq0G_OfjEzGMwc9OAv7uQ/view?usp=share_link
unzip birds.zip
rm birds.zip
cd ..
```

## 2.5 Folder Contents 
Please make sure that the files are in the following format (only the most important files are listed):
```bash
MVATeam1
├── configs
├── data # only mva2023_sod4bird_private_test folder is needed for inferencing
│   ├── birds
│   │   ├── BirdsFlying_1.png
│   │   ├── BirdsFlying_2.png
│   │   └── ...
│   ├── drone2021
│   │   ├── annotations
│   │   │   └── ...
│   │   └── images
│   │       └── ...
│   ├── mva2023_sod4bird_private_test
│   │   ├── annotations
│   │   │   └── private_test_coco_empty_ann.json
│   │   └── images
│   │       └── ...
│   ├── mva2023_sod4bird_pub_test
│   │   ├── annotations
│   │   └── images
│   └── mva2023_sod4bird_train
│       ├── annotations
│       └── images
├── ensemble
├── final
│	├── baseline_centernet
│	│   ├── centernet_resnet18_140e_coco_inference.py
│	│   └── latest.pth
│	├── cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train
│	│   └── latest.pth
│	├── cascade_nwd_paste_howard
│	│   ├── cascade_nwd_paste_howard.zip
│	│   └── latest.pth
│	├── cascade_rcnn_r50_fpn_40e_coco_finetune_sticker
│	│   └── latest.pth
│	├── cascade_rcnn_r50_fpn_40e_coco_nwd_finetune
│	│   ├── cascade_rcnn_r50_fpn_40e_coco_finetune_original.py
│	│   └── latest.pth
│	├── internimage_h_nwd
│	│   ├── intern_h_public.json
│	│   ├── intern_h_public_nosahi.json
│	│   ├── intern_h_public_nosahi_randflip.json
│	│   └── latest.pth
│	├── internimage_xl_no_nwd
│	│   └── latest.pth
│	└── internimage_xl_nwd
│		├── intern_xl_public_nosahi_randflip.json
│		└── latest.pth   
├── ...
└── setup.py
```

## 3. Inference
A bash script for **step 3 and 4** has been prepared:
```bash
bash inference_private.sh
```
The paralleled version script can be executed by:
```bash
bash inference_private_parallel.sh <NUM_GPUS>
# E.g.:
bash inference_private_parallel.sh 4
```

The script executes the following commands.
```bash
# cascade_original.json
python tools/test.py configs/mva2023/cascade_rcnn_r50_fpn_40e_coco_nwd_finetune.py final/cascade_rcnn_r50_fpn_40e_coco_nwd_finetune/latest.pth --format-only --eval-options jsonfile_prefix=cascade_original

# intern_h_public_nosahi.json
python tools/test.py configs/mva2023/cascade_mask_internimage_h_fpn_40e_nwd_finetune.py final/internimage_h_nwd/latest.pth --format-only --eval-options jsonfile_prefix=intern_h_public_nosahi

# intern_xl_public_nosahi_randflip.json
python tools/test.py configs/mva2023/cascade_mask_internimage_xl_fpn_40e_nwd_finetune.py final/internimage_xl_nwd/latest.pth --format-only --eval-options jsonfile_prefix=intern_xl_public_nosahi_randflip

# intern_h_public_nosahi_randflip.json
python tools/test.py configs/mva2023/cascade_mask_internimage_h_fpn_40e_nwd_finetune_tta_randflip.py final/internimage_h_nwd/latest.pth --format-only --eval-options jsonfile_prefix=intern_h_public_nosahi_randflip

# centernet_slicing_01.json
python tools/sahi_evaluation.py configs/mva2023_baseline/centernet_resnet18_140e_coco_inference.py \
			final/baseline_centernet/latest.pth \
			data/mva2023_sod4bird_private_test/images/ \
			data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
			--out-file-name centernet_slicing_01.json \
			--score-threshold 0.1 \
			--crop-size 512 \
			--overlap-ratio 0.2

#results_interImage.json
python tools/sahi_evaluation.py configs/mva2023/cascade_mask_internimage_xl_fpn_finetune.py \
			final/internimage_xl_no_nwd/latest.pth \
		    data/mva2023_sod4bird_private_test/images/ \
		    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
		    --out-file-name results_interImage.json

# cascade_nwd_paste_howard_0604.json
python tools/sahi_evaluation.py  configs/cascade_rcnn_mva2023/cascade_rcnn_r50_fpn_20e_coco_finetune_nwd_paste.py \
			final/cascade_nwd_paste_howard/latest.pth \
		    data/mva2023_sod4bird_private_test/images/ \
		    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
		    --out-file-name cascade_nwd_paste_howard_0604.json

# cascade_rcnn_sticker_61_2.json
python tools/sahi_evaluation.py  configs/cascade_rcnn_mva2023/cascade_rcnn_r50_fpn_40e_coco_finetune_sticker.py \
			final/cascade_rcnn_r50_fpn_40e_coco_finetune_sticker/latest.pth \
		    data/mva2023_sod4bird_private_test/images/ \
		    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
		    --out-file-name cascade_rcnn_sticker_61_2.json

# cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train.json
python tools/sahi_evaluation.py  configs/mva2023/cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train.py \
			final/cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train/latest.pth \
		    data/mva2023_sod4bird_private_test/images/ \
		    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
		    --out-file-name cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train.json

# cascade_mask_internimage_h_fpn_40e_nwd_finetune.json
python tools/sahi_evaluation.py  configs/mva2023/cascade_mask_internimage_h_fpn_40e_nwd_finetune.py \
			final/internimage_h_nwd/latest.pth \
		    data/mva2023_sod4bird_private_test/images/ \
		    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
			--crop-size 512 \
		    --out-file-name cascade_mask_internimage_h_fpn_40e_nwd_finetune.json
```

## 4. Ensemble

The following commands are also included in the `inference_private.sh` and `inference_private_parallel.sh`.
```bash
mv cascade_original.bbox.json ensemble/cascade_original.json
mv intern_h_public_nosahi.bbox.json ensemble/intern_h_public_nosahi.json
mv intern_xl_public_nosahi_randflip.bbox.json ensemble/intern_xl_public_nosahi_randflip.json
mv intern_h_public_nosahi_randflip.bbox.json ensemble/intern_h_public_nosahi_randflip.json
mv centernet_slicing_01.json ensemble/centernet_slicing_01.json
mv results_interImage.json ensemble/results_interImage.json
mv cascade_nwd_paste_howard_0604.json ensemble/cascade_nwd_paste_howard_0604.json
mv cascade_rcnn_sticker_61_2.json ensemble/cascade_rcnn_sticker_61_2.json
mv cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train.json ensemble/cascade_mask_internimage_xl_fpn_20e_nwd_finetune_merged_train.json
mv cascade_mask_internimage_h_fpn_40e_nwd_finetune.json ensemble/cascade_mask_internimage_h_fpn_40e_nwd_finetune.json

pushd ensemble
python ensemble.py ../data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json
zip results_team1.zip results.json
popd
cp ensemble/results_team1.zip ./
```

### **The final results to be evaluated is the file `results_team1.zip`!**

# Trouble Shooting
If you encounter 
```KeyError: "CenterNet: 'InternImage is not in the models registry'"```\
or
```KeyError: "CenterNet: 'CustomLayerDecayOptimizerConstructor is not in the models registry'"```,\
please add
```python
import mmdet_custom  # noqa: F401,F403
import mmcv_custom  # noqa: F401,F403
```
to the python script you are running.
