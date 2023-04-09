# Training
## Centernet Baseline
```shell
# centernet r18
python3 tools/train.py  configs/mva2023_baseline/centernet_resnet18_140e_coco.py
python3 tools/train.py  configs/mva2023_baseline/centernet_resnet18_140e_coco_finetune.py
python3 hard_neg_example_tools/test_hard_neg_example.py \
    --config configs/mva2023_baseline/centernet_resnet18_140e_coco_sample_hard_negative.py \
    --checkpoint work_dirs/centernet_resnet18_140e_coco_finetune/latest.pth \
    --launcher none \
    --generate-hard-negative-samples True \
    --hard-negative-file work_dirs/centernet_resnet18_140e_coco_finetune/train_coco_hard_negative.json \
    --hard-negative-config num_max_det=10 pos_iou_thr=1e-5 score_thd=0.05 nms_thd=0.05
python3 tools/train.py configs/mva2023_baseline/centernet_resnet18_140e_coco_hard_negative_training.py

# centernet r50
python3 tools/train.py  configs/mva2023_baseline/centernet_resnet50_140e_coco.py
python3 tools/train.py  configs/mva2023_baseline/centernet_resnet50_140e_coco_finetune.py
python3 hard_neg_example_tools/test_hard_neg_example.py \
    --config configs/mva2023_baseline/centernet_resnet50_140e_coco_sample_hard_negative.py \
    --checkpoint work_dirs/centernet_resnet50_140e_coco_finetune/latest.pth \
    --launcher none \
    --generate-hard-negative-samples True \
    --hard-negative-file work_dirs/centernet_resnet50_140e_coco_finetune/train_coco_hard_negative.json \
    --hard-negative-config num_max_det=10 pos_iou_thr=1e-5 score_thd=0.05 nms_thd=0.05
python3 tools/train.py configs/mva2023_baseline/centernet_resnet50_140e_coco_hard_negative_training.py

# centernet intern image xl
python3 tools/train.py  configs/mva2023_baseline/centernet_internimage_xl_140e_coco.py
python3 tools/train.py  configs/mva2023_baseline/centernet_internimage_xl_140e_coco_finetune.py
python3 hard_neg_example_tools/test_hard_neg_example.py \
    --config configs/mva2023_baseline/centernet_internimage_xl_140e_coco_sample_hard_negative.py \
    --checkpoint work_dirs/centernet_internimage_xl_140e_coco_finetune/latest.pth \
    --launcher none \
    --generate-hard-negative-samples True \
    --hard-negative-file work_dirs/centernet_internimage_xl_140e_coco_finetune/train_coco_hard_negative.json \
    --hard-negative-config num_max_det=10 pos_iou_thr=1e-5 score_thd=0.05 nms_thd=0.05
python3 tools/train.py configs/mva2023_baseline/centernet_internimage_xl_140e_coco_hard_negative_training.py
```
## Cascade RCNN
```shell
# Cascade RCNN r18
python3 tools/train.py  configs/mva2023/cascade_rcnn_r18_fpn_140e_coco_nwd.py
python3 tools/train.py  configs/mva2023/cascade_rcnn_r18_fpn_20e_coco_nwd_finetune.py

# Cascade RCNN r50
python3 tools/train.py  configs/mva2023/cascade_rcnn_r50_fpn_140e_coco_nwd.py
python3 tools/train.py  configs/mva2023/cascade_rcnn_r50_fpn_20e_coco_nwd_finetune.py

# Cascade intern image xl
python3 tools/train.py  configs/mva2023/cascade_rcnn_internimage_xl_fpn_140e_coco_nwd.py
python3 tools/train.py  configs/mva2023/cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune.py
```

# Inferencing
```shell
python3 tools/test.py configs/mva2023_baseline/centernet_resnet18_140e_coco_inference.py work_dirs/centernet_resnet18_140e_coco_hard_negative_training/latest.pth --format-only --eval-options jsonfile_prefix=centernet_resnet18_140e_coco_hard_negative_training
python3 tools/test.py configs/mva2023_baseline/centernet_resnet50_140e_coco_inference.py work_dirs/centernet_resnet50_140e_coco_hard_negative_training/latest.pth --format-only --eval-options jsonfile_prefix=centernet_resnet50_140e_coco_hard_negative_training
python3 tools/test.py configs/mva2023_baseline/centernet_internimage_xl_140e_coco_inference.py work_dirs/centernet_internimage_xl_140e_coco_hard_negative_training/latest.pth --format-only --eval-options jsonfile_prefix=centernet_internimage_xl_140e_coco_hard_negative_training

mv centernet_resnet18_140e_coco_hard_negative_training.bbox.json submit/centernet_resnet18_140e_coco_hard_negative_training.json 
mv centernet_resnet50_140e_coco_hard_negative_training.bbox.json submit/centernet_resnet50_140e_coco_hard_negative_training.json
mv centernet_internimage_xl_140e_coco_hard_negative_training.bbox.json submit/centernet_internimage_xl_140e_coco_hard_negative_training.json

python3 tools/sahi_evaluation.py configs/mva2023/cascade_rcnn_r18_fpn_20e_coco_nwd_finetune.py \
    work_dirs/cascade_rcnn_r18_fpn_20e_coco_nwd_finetune/latest.pth \
    data/mva2023_sod4bird_private_test/images/ \
    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
    --out-file-name cascade_rcnn_r18_fpn_20e_coco_nwd_finetune.json
python3 tools/sahi_evaluation.py configs/mva2023/cascade_rcnn_r50_fpn_20e_coco_nwd_finetune.py \
    work_dirs/cascade_rcnn_r50_fpn_20e_coco_nwd_finetune/latest.pth \
    data/mva2023_sod4bird_private_test/images/ \
    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
    --out-file-name cascade_rcnn_r50_fpn_20e_coco_nwd_finetune.json
python3 tools/sahi_evaluation.py configs/mva2023/cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune.py \
    work_dirs/cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune/latest.pth \
    data/mva2023_sod4bird_private_test/images/ \
    data/mva2023_sod4bird_private_test/annotations/private_test_coco_empty_ann.json \
    --out-file-name cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune.json

mv work_dirs/cascade_rcnn_r18_fpn_20e_coco_nwd_finetune.json submit/cascade_rcnn_r18_fpn_20e_coco_nwd_finetune.json
mv work_dirs/cascade_rcnn_r50_fpn_20e_coco_nwd_finetune.json submit/cascade_rcnn_r50_fpn_20e_coco_nwd_finetune.json
mv work_dirs/cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune.json submit/cascade_rcnn_internimage_xl_fpn_20e_coco_nwd_finetune.json
```

# Ensemble
```shell
# TODO: ensemble results from submit/ and output to results.json
zip results.zip results.json
```