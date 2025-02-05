# FemtoDet
Official codes of ICCV2023 paper: <<Femtodet: an object detection baseline for energy versus performance tradeoffs>>

### Dependencies
* Python 3.8
* Torch 1.9.1+cu111
* Torchvision 0.10.1+cu111
* mmcv-full 1.4.2
* mmdet 2.23.0

### Installation
Do it as [mmdetection](https://github.com/open-mmlab/mmdetection/tree/v2.23.0) had done.

### Preparation
1. Download the dataset.
   
   We mainly train FemtoDet on [Pascal VOC 0712](http://host.robots.ox.ac.uk/pascal/VOC/), you should firstly download the datasets. By default, we assume the dataset is stored in ./data/.

2. Dataset preparation.
   
   Then, you can move all images to ./data/voc2coco/jpeg/*;you can use our converted coco format [annotation files](https://pan.baidu.com/s/1SLgZd_2cLhLFC54lLM3sHg?pwd=umbz)(umbz) and put these files to ./data/voc2coco/annotations/*; finally, the directory structure is

```
*data/voc2coco
    *jpeg
        *2008_003841.jpg
        *...
    *annotations
        *trainvoc_annotations.json
        *testvoc_annotations.json
```

3. Download the initialized models.
   
   We trained our designed backbone on ImageNet 1k, and used it for [the inite weights](https://pan.baidu.com/s/1JGsvlvzPkb5nxGBaRSD7ng?pwd=hx8k))(hx8k) of FemtoDet.

```
FemtoDet/weights/*
```

### Training
```
bash ./tools/train_femtodet.sh 4
```

### Results (trained on VOC) and Models

[trained model and logs download](https://pan.baidu.com/s/1IBpYyhhGA-kg71S_1OJpjA?pwd=7aok) (7aok)
```
|  Detector  | Params | box AP50 |              Config                    | 
---------------------------------------------------------------------------
|            |        |   37.1   | ./configs/femtoDet/femtodet_0stage.py  |
                      -----------------------------------------------------
|  FemtoDet  | 68.77k |   40.4   | ./configs/femtoDet/femtodet_1stage.py  |
                      -----------------------------------------------------
|            |        |   44.4   | ./configs/femtoDet/femtodet_2stage.py  |
                      -----------------------------------------------------
|            |        |   46.5   | ./configs/femtoDet/femtodet_3stage.py  |
---------------------------------------------------------------------------
```
### Deployment
- Support convert model to onnx.
- You can convert onnx using the following command, but there are many useless ops that affect performance.
```
python tools/deployment/pytorch2onnx.py femtodet_3stage.py ./best_bbox_mAP_epoch_300.pth --simplify --skip-postprocess --output-file work_dirs/femtodet_0stage/out.onnx
```
- We support a reparameterized way to transform the model(recommend).
1. Convert stage 3 model use 
```
python tools/deployment/femtodet_deploy_reparam.py ./best_bbox_mAP_epoch_300.pth work_dirs/femtodet_0stage/rep.pth.
```
2. This model will have same precision with before.You can use the following command to compare.
```
python tools/test.py femtodet_3stage.py ./best_bbox_mAP_epoch_300.pth --eval bbox
python tools/test.py femtodet_deploy.py ./rep.pth --eval bbox
```
3. Convert reparameterized model to onnx.
```
python tools/deployment/pytorch2onnx.py ./femtodet_deploy.py ./rep.pth --simplify --skip-postprocess --output-file work_dirs/out_rep.onnx
```
On the left is the model structure before reparameterization, and on the right is the model structure after reparameterization.If you want to deploy to the device, you need to write your own post-processing code.
![compare](demo/compare.png)

### References
If you find the code useful for your research, please consider citing:
```bib
@InProceedings{Tu_2023_ICCV,
    author    = {Tu, Peng and Xie, Xu and Ai, Guo and Li, Yuexiang and Huang, Yawen and Zheng, Yefeng},
    title     = {FemtoDet: An Object Detection Baseline for Energy Versus Performance Tradeoffs},
    booktitle = {Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)},
    month     = {October},
    year      = {2023},
    pages     = {13318-13327}
}
@misc{tu2023femtodet,
      title={FemtoDet: An Object Detection Baseline for Energy Versus Performance Tradeoffs}, 
      author={Peng Tu and Xu Xie and Guo AI and Yuexiang Li and Yawen Huang and Yefeng Zheng},
      year={2023},
      eprint={2301.06719},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```
