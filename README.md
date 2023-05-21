# A New Perspective for Shuttlecock Hitting Event Detection




## Introduction
According to global statistics, there are approximately 2.2 billion badminton players worldwide and more than 3 million in Taiwan. This single sport is ranked second in terms of national popularity. In recent years, badminton players have achieved outstanding performances in international competitions, gradually increasing the public attention.

In terms of badminton skills and tactics analysis, our team has proposed a match shuttlecock recording format and developed a computer vision assisted quick shuttlecock labeling program to initiate the research of badminton big data. Although many computer-assisted techniques have been used, manual shuttlecock labeling still requires manpower and time, especially for technical data identification, which requires badminton experts to perform. Through this competition, we hope to invite machine learning, image processing, and sports science expertise to develop an automatic shuttlecock labeling model with a high recognition rate, making the massive badminton information collection possible, and thus popularizing the research and application of badminton tactics analysis.




## Environmental Setup


<details>

<summary>Hardware Information</summary>

- CPU: i7-11700F
- GPU: GeForce GTX 1660 SUPER™ VENTUS XS OC (6G)
  
</details>


<details>

<summary>Create Conda Environments</summary>

###
```bash
# TrackNetv2
$ conda create -n tracknetv2 python=3.9 -y
# SwingNet
$ conda create -n golfdb python=3.8 -y
# ViT
$ conda create -n ViT_j python==3.9 -y
# YOLOv5
$ conda create -n yolov5 python=3.7 -y
```
  
</details>


<details>

<summary>Install Required Packages</summary>

```bash
# TrackNetv2
$ conda activate tracknetv2
$ git clone https://nol.cs.nctu.edu.tw:234/lukelin/TrackNetV2_pytorch.git
$ sudo apt-get install git
$ sudo apt-get install python3-pip
$ pip3 install pandas
$ pip3 install opencv-python
$ pip3 install matplotlib
$ pip3 install -U scikit-learn
$ pip3 install torch
$ pip3 install torchvision
# SwingNet
$ conda activate golfdb
$ git clone https://github.com/wmcnally/golfdb.git
$ pip3 install opencv-python
$ pip3 install scipy
$ pip3 install pandas
$ pip3 install torch
$ pip3 install torchvision
$ pip3 install torchaudio
# ViT
$ conda activate ViT_j
$ git clone https://github.com/jeonsworld/ViT-pytorch.git
$ cd ViT-pytorch/
$ pip3 install -r requirements.txt
$ mkdir checkpoint/
$ cd checkpoint/
$ wget https://storage.googleapis.com/vit_models/imagenet21k+imagenet2012/ViT-B_16.npz
$ git clone https://github.com/NVIDIA/apex    # A PyTorch Extension
$ cd apex/
$ python3 setup.py install
# YOLOv5
$ conda activate yolov5
$ git clone https://github.com/ultralytics/yolov5.git
$ cd yolov5/
$ pip install -r requirements.txt
```
  
</details>





## Reproduce Details


<details>

<summary>VideoName, ShotSeq, HitFrame</summary>

1. put Badminton/data/part2/test/00170/ .. /00399/ into Badminton/data/part1/val/
    ```bash
    → Badminton/data/part1/val/00001/ .. /00399/    # 1280x720
    ```
2. convert val/+test/ to val_test_xgg/
    ```bash
    $ conda activate golfdb
    $ cd Badminton/src/preprocess/
    $ mkdir val_test_xgg
    $ python3 rt_conversion_datasets.py
    → Badminton/src/preprocess/val_test_xgg/    # 1280x720
    ```
3. upload val_test_xgg/ to google drive Teaching_Computer_to_Watch_Badminton_Matches_Taiwan_first_competition_combining_AI_and_sports/datasets/part1/
    ```bash
    → Teaching_Computer_to_Watch_Badminton_Matches_Taiwan_first_competition_combining_AI_and_sports/datasets/part1/val_test_xgg/
    → execute golfdb_xgg_inference_best.ipynb
    → src/Notebook/golfdb/golfdb_G3_fold5_iter3000_val_test_X.csv    # 0.0426
    ```
  
</details>


<details>

<summary>Hitter</summary>

4. put golfdb_G3_fold5_iter3000_val_test_X.csv into Badminton/src/postprocess/
    ```bash
    → Badminton/src/postprocess/golfdb_G3_fold5_iter3000_val_test_X.csv
    ```
5. extract hitframe from csv file
    ```bash
    $ cd Badminton/src/postprocess/
    $ mkdir HitFrame
    $ mkdir HitFrame/1
    $ python3 get_hitframe.py
    >> len(vns), len(hits), len(os.listdir(savePath)) = 4007, 4007, 4007
    → Badminton/src/postprocess/HitFrame/1/    # 720x720, 4007
    ```
6. execute hitter inference
    ```bash
    $ conda activate ViT_j
    $ cd Badminton/src/ViT-pytorch_Hitter/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_Hitter_ViT-B_16_checkpoint.bin","output/fold2_Hitter_ViT-B_16_checkpoint.bin","output/fold3_Hitter_ViT-B_16_checkpoint.bin","output/fold4_Hitter_ViT-B_16_checkpoint.bin","output/fold5_Hitter_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    → Badminton/src/ViT-pytorch_Hitter/golfdb_G3_fold5_iter3000_val_test_hitter_vote.csv    # 0.0494
    → Badminton/src/ViT-pytorch_Hitter/golfdb_G3_fold5_iter3000_val_test_hitter_mean.csv    # 0.0494
    ```
  
</details>


<details>

<summary>RoundHead</summary>

7. execute roundhead inference
    ```bash
    $ cd Badminton/src/ViT-pytorch_RoundHead/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_RoundHead_ViT-B_16_checkpoint.bin","output/fold2_RoundHead_ViT-B_16_checkpoint.bin","output/fold3_RoundHead_ViT-B_16_checkpoint.bin","output/fold4_RoundHead_ViT-B_16_checkpoint.bin","output/fold5_RoundHead_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    → Badminton/src/ViT-pytorch_Hitter/golfdb_G3_fold5_iter3000_val_test_hitter_vote_roundhead_vote.csv    # 0.0527
    → Badminton/src/ViT-pytorch_Hittergolfdb_G3_fold5_iter3000_val_test_hitter_mean_roundhead_mean.csv    # 0.0527
    ```
  
</details>


<details>

<summary>Backhand</summary>

8. execute backhand inference
    ```bash
    $ cd Badminton/src/ViT-pytorch_Backhand/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_Backhand_ViT-B_16_checkpoint.bin","output/fold2_Backhand_ViT-B_16_checkpoint.bin","output/fold3_Backhand_ViT-B_16_checkpoint.bin","output/fold4_Backhand_ViT-B_16_checkpoint.bin","output/fold5_Backhand_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    ```
  
</details>


<details>

<summary>BallHeight</summary>


  
</details>


<details>

<summary>VideoName, ShotSeq, HitFrame</summary>

9. execute ballheight inference
    ```bash
    $ cd Badminton/src/ViT-pytorch_BallHeight/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_BallHeight_ViT-B_16_checkpoint.bin","output/fold2_BallHeight_ViT-B_16_checkpoint.bin","output/fold3_BallHeight_ViT-B_16_checkpoint.bin","output/fold4_BallHeight_ViT-B_16_checkpoint.bin","output/fold5_BallHeight_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    ```
  
</details>


<details>

<summary>LandingX</summary>

10. get trajectory
    ```bash
    $ conda activate tracknetv2
    $ cd Badminton/src/TrackNetV2_pytorch/10-10Gray/
    $ mkdir output
    $ python3 predict10_custom.py
    $ mkdir denoise
    $ python3 denoise10_custom.py
    ```
11. execute landingx inference
    ```bash
    $ cd Badminton/src/TrackNetV2_pytorch/10-10Gray/
    $ (mkdir event
    $ cd Badminton/src/TrackNetV2_pytorch/
    $ python3 event_detection_custom.py
    $ python3 HitFrame.py)
    $ python3 LandingX.py
    ```
  
</details>



<details>

<summary>LandingY, HitterLocationX, HitterLocationY, DefenderLocationX, DefenderLocationY</summary>

12. extract hitframe for yolo from csv
    ```bash
    $ cd Badminton/src/postprocess/
    $ mkdir HitFrame_yolo
    $ python3 get_hitframe_yolo.py
    → Badminton/src/postprocess/HitFrame_yolo/    # 1280x720, 4007
    ```
13. execute yolov5 inference
    ```bash
    $ conda activate yolov5
    $ cd Badminton/src/yolov5/
    $ python3 detect.py --weights runs/train/exp/weights/best.pt --source /home/yuhsi/Badminton/src/postprocess/HitFrame_yolo/ --conf-thres 0.3 --iou-thres 0.3 --save-txt --imgsz 2880 --agnostic-nms --augment
    → Badminton/src/yolov5/runs/detect/exp/    # 4007
    ```
14. execute landingy inference
    ```bash
    $ mkdir runs/detect/exp_draw
    $ mkdir runs/detect/exp_draw/case1
    $ python3 LandingY_Hitter_Defender_Location.py
    ```
  
</details>



<details>

<summary>BallType</summary>

15. execute balltype inference
    ```bash
    $ conda activate ViT_j
    $ cd Badminton/src/ViT-pytorch_BallType/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_BallType_ViT-B_16_checkpoint.bin","output/fold2_BallType_ViT-B_16_checkpoint.bin","output/fold3_BallType_ViT-B_16_checkpoint.bin","output/fold4_BallType_ViT-B_16_checkpoint.bin","output/fold5_BallType_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    ```
  
</details>



<details>

<summary>Winner</summary>

16. execute winner inference
    ```bash
    $ cd Badminton/src/Vit-pytorch_Winner/
    $ python3 submit.py --model_type ["ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16","ViT-B_16"] --checkpoint ["output/fold1_Winner_ViT-B_16_checkpoint.bin","output/fold2_Winner_ViT-B_16_checkpoint.bin","output/fold3_Winner_ViT-B_16_checkpoint.bin","output/fold4_Winner_ViT-B_16_checkpoint.bin","output/fold5_Winner_ViT-B_16_checkpoint.bin"] --img_size [480,480,480,480,480]
    ```
  
</details>
