<div align="center">   
  
# You Only :eyes: One Sequence
</div>

* **TL;DR:** We study the transferability of the vanilla ViT pre-trained on mid-sized ImageNet-1k to the more challenging COCO object detection benchmark.

* Code and model weights will be released soon, please stay tuned :)

<br>

> [**You Only Look at One Sequence: Rethinking Transformer in Vision through Object Detection**](http://arxiv.org/abs/yolos.yolos)
>
> by [Yuxin Fang](https://scholar.google.com/citations?user=_Lk0-fQAAAAJ&hl=en)<sup>1</sup> \*, Bencheng Liao<sup>1</sup> \*, [Xinggang Wang](https://xinggangw.info/)<sup>1 :email:</sup>, [Jiemin Fang](https://jaminfong.cn)<sup>2, 1</sup>, Jiyang Qi<sup>1</sup>, [Rui Wu](https://scholar.google.com/citations?hl=en&user=Z_ZkkbEAAAAJ&view_op=list_works&citft=1&email_for_op=2yuxinfang%40gmail.com&gmla=AJsN-F6AJfvX_wN_jDDdJOp33cW5LrvrAwATh1FFyrUxKD8H354RTN7gMFIXi4NTozHvdj1ITW1q5sNS3ED-3htZJpnUA9BraZa8Wnc_XSfCR37MriE77bh9KHFTKml-qPSgNTPdxwFl8KHxIgOWc_ZuJdvo8cbBWc_Ec3SBL6n7wsYYS2E1Wzm4kWwXQybOJCGjI8_EwHwwipOfkQR9I2C_Riq1gk1Y_JG3BQ3xrTy2fN_plPE37StUe_nOnrTjUz919wcMXKqW)<sup>3</sup>, Jianwei Niu<sup>3</sup>, [Wenyu Liu](http://eic.hust.edu.cn/professor/liuwenyu/)<sup>1</sup>.
> 
> <sup>1</sup> [School of EIC, HUST](http://eic.hust.edu.cn/English/Home.htm), <sup>2</sup> Institute of AI, HUST, <sup>3</sup> [Horizon Robotics](https://en.horizon.ai).
> 
> (\*) equal contribution, (<sup>:email:</sup>) corresponding author.
> 
> *arXiv technical report ([arXiv yolos.yolos](http://arxiv.org/abs/yolos.yolos))*

<br>

## You Only Look at One Sequence (YOLOS)

### The Illustration of YOLOS
![yolos](yolos.png)

### Highlights

Directly inherited from [ViT](https://arxiv.org/abs/2010.11929) ([DeiT](https://arxiv.org/abs/2012.12877)), YOLOS is not designed to be yet another high-performance object detector, but to unveil the versatility and transferability of Transformer from image recognition to object detection.
Concretely, our main contributions are summarized as follows:

* We use the mid-sized `ImageNet-1k` as the sole pre-training dataset, and show that a vanilla [ViT](https://arxiv.org/abs/2010.11929) ([DeiT](https://arxiv.org/abs/2012.12877)) can be successfully transferred to perform the challenging object detection task and produce competitive `COCO` results with the fewest possible modifications, _i.e._, by only looking at one sequence (YOLOS).

* We demonstrate that 2D object detection can be accomplished in a pure sequence-to-sequence manner by taking a sequence of fixed-sized non-overlapping image patches as input. Among existing object detectors, YOLOS utilizes minimal 2D inductive biases. Moreover, it is feasible for YOLOS to perform object detection in any dimensional space unaware the exact spatial structure or geometry.

* For [ViT](https://arxiv.org/abs/2010.11929) ([DeiT](https://arxiv.org/abs/2012.12877)), we find the object detection results are quite sensitive to the pre-train scheme and the detection performance is far from saturating. Therefore the proposed YOLOS can be used as a challenging benchmark task to evaluate different pre-training strategies for [ViT](https://arxiv.org/abs/2010.11929) ([DeiT](https://arxiv.org/abs/2012.12877)).

* We also discuss the impacts as wel as the limitations of prevalent pre-train schemes and model scaling strategies for Transformer in vision through transferring to object detection.

## Requirement
This codebase has been developed with python version 3.6, PyTorch 1.5+ and torchvision 0.6+:
```setup
conda install -c pytorch pytorch torchvision
```
Install pycocotools (for evaluation on COCO) and scipy (for training):
```setup
conda install cython scipy
pip install -U 'git+https://github.com/cocodataset/cocoapi.git#subdirectory=PythonAPI'
```

## Data preparation
Download and extract COCO 2017 train and val images with annotations from http://cocodataset.org. We expect the directory structure to be the following:
```
path/to/coco/
  annotations/  # annotation json files
  train2017/    # train images
  val2017/      # val images
```
## Training
To train the YOLOS-Ti model(s) in the paper, run this command:

```train
python -m torch.distributed.launch \
    --nproc_per_node=8 \
    --use_env main.py \
    --coco_path /path/to/coco
    --batch_size 2 \
    --lr 5e-5 \
    --epochs 300 \
    --backbone_name tiny \
    --pre_trained \
    --eval_size 512 \
    --init_pe_size 800 1333 \
    --output_dir /output/path/box_model

```


To train the YOLOS-S model(s) in the paper, run this command:

```train
python -m torch.distributed.launch \
    --nproc_per_node=8 \
    --use_env main.py \
    --coco_path /path/to/coco
    --batch_size 1 \
    --lr 2.5e-5 \
    --epochs 150 \
    --backbone_name small \
    --pre_trained \
    --eval_size 800 \
    --init_pe_size 512 864 \
    --mid_pe_size 512 864 \
    --output_dir /output/path/box_model

```
To train the YOLOS-B model(s) in the paper, run this command:

```train
python -m torch.distributed.launch \
    --nproc_per_node=8 \
    --use_env main.py \
    --coco_path /path/to/coco
    --batch_size 1 \
    --lr 2.5e-5 \
    --epochs 150 \
    --backbone_name base \
    --pre_trained \
    --eval_size 800 \
    --init_pe_size 800 1344 \
    --mid_pe_size 800 1344 \
    --output_dir /output/path/box_model

```

## Evaluation

To evaluate YOLOS-Ti model on coco, run:

```eval
python main.py --coco_path /path/to/coco --batch_size 2 --backbone_name tiny --eval --eval_size 512 --init_pe_size 800 1333 --resume /path/to/YOLOS-Ti
```
To evaluate YOLOS-S model on coco, run:
```eval
python main.py --coco_path /path/to/coco --batch_size 1 --backbone_name small --eval --eval_size 800 --init_pe_size 512 864 --mid_pe_size 512 864 --resume /path/to/YOLOS-S
```
To evaluate YOLOS-B model on coco, run:
```eval
python main.py --coco_path /path/to/coco --batch_size 1 --backbone_name small --eval --eval_size 800 --init_pe_size 800 1344 --mid_pe_size 800 1344 --resume /path/to/YOLOS-B
```

## Results
| Model | Pretrained Epochs |Backbone|Finetuned Epochs| eval_size |#Params (M) | FLOPS(G) | URL | AP|
| --- | --- | ---|--- | --- | --- |--- |---| ---|
|YOLOS-Ti|300|[Deit-Ti](https://dl.fbaipublicfiles.com/deit/deit_tiny_patch16_224-a1311bcf.pth)|300|512|6.5|???|[yolos_ti.pth]()|28.7
|YOLOS-S|300|[Deit-S](https://dl.fbaipublicfiles.com/deit/deit_small_patch16_224-cd65a155.pth)|150|800|30.7|200.2|[yolos_s_300_pre.pth]()|36.1
|YOLOS-S|200|[Deit-S]()|150|800|30.7|200.2|[yolos_s_300_pre.pth]()|36.1
|YOLOS-S(dWr)|300|[Deit-S(dWr Scale)]()|150|800|27.9|179|[yolos_s_dWr.pth]()|37.6
|YOLOS-B|300|[Deit-B distilled](https://dl.fbaipublicfiles.com/deit/deit_base_distilled_patch16_384-d0272ac0.pth)|150|800|127|537|[yolos_base.pth]()|42.0
## Citation

If you find our paper and code useful in your research, please consider giving a star :star: and citation :pencil: :

```BibTeX
@article{YOLOS,
  title={You Only Look at One Sequence: Rethinking Transformer in Vision through Object Detection},
  author={All YOLOS Authors},
  journal={arXiv preprint arXiv:yolos.yolos},
  year={2021}
}
```
