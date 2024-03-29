# Fast Semantic Image Segmentation Using Multi-Task Deep Neural Networks (CNN-I2I)

This is an official pytorch implementation of [Autonomous UAV Safety by Visual Human Crowd Detection Using Multi-Task Deep Neural Networks](https://ieeexplore.ieee.org/abstract/document/9560830). This implementation can be also easily adapted for any binary/multi-class image segmentation task (see instructions below).

## Prerequisits
1. PyTorch (torch and torchvision)
* `pip install torch torchvision`
2. Easydict
* `pip install easydict`
3. tqdm
* `pip install tqdm`
4. OpenCV
* `pip install opencv-python`
5. Scipy
* `pip install scipy`
6. Pandas
* `pip install pandas`

## Prepare dataset fot crowd segmentation
The dataset used for the crowd detetction is provided in the [link](https://aiia.csd.auth.gr/open-multidrone-datasets/) under the name `DroneCrowd`.

The dataset used for the training and evaluation of the models must have the following structure:
```
.
+-- RGB
|   +-- train
|       +-- subdirs
|   +-- val
|       +-- subdirs
+-- annotations
|   +-- train
|       +-- subdirs
|   +-- val
|       +-- subdirs
+-- test.txt
+-- train.txt
+-- val.txt
```
Each subfolder of the `annotations` folder contains a segmentation ground truth and an RGB segmentation mask of each image. (WARNING: the RGB segmentation mask must have non-zero values on all three channels in order to train the model efficiently.) A script [`configure_dataset_annotations.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/configure_dataset_annotations.py) is provided to create the segmentation ground truth or the RGB segmentation mask.
To create the test, train and val txt files use the script [`create_trainvaltest_files.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/create_trainvaltest_files.py).

(Crowd: create the dataset folder and run the scripts: [`configure_dataset_annotations.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/configure_dataset_annotations.py) and [`create_trainvaltest_files.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/create_trainvaltest_files.py))

## Training for Crowd segmentation
1. download the pre-trained model [ResNet18](https://drive.google.com/file/d/19xf-d76FFiJfCm6a9QtZrc0xLsw8GMDG/view?usp=sharing) and save it inside a folder (train from scratch is possible however accuracy will decrease).
2. inside the [`config.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/config.py) change the repo name (according to the folder name in which the project is saved), the paths corresponding to the location of the dataset and the resnet18 and optionally provide the desired setteings for the hyperparameters, batch size and image configuartion.
3. train a network using the [`train_i2i_crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/train_i2i_crowd.py).

## Evaluation for Crowd segmentation
In case the training phase is skipped the following steps must be followed:
1. create the directory `./CNN-I2I/log/crowd_detection/snapshot` and inside place the pretrained model epoch-XX.pth (the pretrained model provided has an accuracy of 82%)
2. provide the dataset path in the [`config.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/config.py)

The following steps must be followed independently of the model used (pre-trained/trained)
1. in argument parser of the file [`eval_i2i_crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/eval_i2i_crowd.py) define:
* the model indice (single number or range of numbers), corresponding to an epoch, that will be evaluated (line 119).
* the save path for the predicted segmentation masks (line 124) (optional, default None).
* (Optional) by setting True or False (default) the show_image parameter each image and the corresponding segmentation mask is shown.
2. evaluate the model using the [`eval_i2i_crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/eval_i2i_crowd.py).

## Custom detection
In order to train the network on a custom dataset first a dataset folder must be created with the same structure presented earlier and then run the [`configure_dataset_annotations.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/configure_dataset_annotations.py) to create th RGB segmentation masks and the [`create_trainvaltest_files.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/scripts/create_trainvaltest_files.py) to create the configuration files `test.txt`, `train.txt` and `val.txt`.

### Training
For the training of the network on a custom dataset follow these steps.

1. download the pre-trained model [ResNet18](https://drive.google.com/file/d/19xf-d76FFiJfCm6a9QtZrc0xLsw8GMDG/view?usp=sharing) and save it inside a folder (train from scratch is possible however accuracy will decrease).
2. inside the folder [`./furnace/datasets/`](https://github.com/AIIALaboratory/fast_image_segmentation/tree/main/CNN-I2I/furnace/datasets) change according to the new custom dataset the [`__init__.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/furnace/datasets/__init__.py) file and create a new folder of the custom dataset with two files [`__init__.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/furnace/datasets/crowd/__init__.py) and `custom_dataset.py` (example file from the crowd dataset [`crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/furnace/datasets/crowd/crowd.py)
3. inside the folder [`./furnace/seg_opr/`](https://github.com/AIIALaboratory/fast_image_segmentation/tree/main/CNN-I2I/furnace/seg_opr) change accordingly the file [`loss_opr.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/furnace/seg_opr/loss_opr.py) for the custom dataset and set the new weights of the classes (depending on the number of pixels that belong in each class) inside the class `ProbOhemCrossEntropy2d`. 
4. create a new folder inside the [`./model/`](https://github.com/AIIALaboratory/fast_image_segmentation/tree/main/CNN-I2I/model/) for the training, evaluation, network, dataloader and configuration files. RECOMMENDED (normally the network and dataloader files should be the same)
5. inside the folder of step 4 create a new `config_custom.py` file by changing properly the already existing [`config.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/config.py). 
* change the repo name (according to the folder name in which the project is saved)
* define the correct dataset path (line 37)
* define the correct path (step 2) for the pre-trained model ResNet18 (line 77) 
* configure image parameters (lines 58-68)
* hyperparameters train
6. in case the name of the [`config.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/config.py) was change, inside the file [`dataloader.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/dataloader.py), [`pix2pix_networks.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/pix2pix_networks.py) and the [`network.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/network.py) the name of the config module (line 6) must be changed with the name of the new created config file
7. inside the folder of step 4 create a new training python file `train_i2i_custom.py` for the custom dataset with these changes to the corresponding [`train_i2i_crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/train_i2i_crowd.py) of the Crowd dataset:
* import the new configuration file `config_custom.py` and and the custom dataset class from the file `custom_dataset.py` (line 14 and 17 accordingly). Change the Crowd class name with the new custom dataset class name in line 38
* (Optional) create a new folder with same name as the created folder in step 4 inside the `./CNN-I2I/log/` for the custom dataset with two subfolders Imgs and snapshot. Set the path for the Imgs folder to the filename variable of the `train_i2i_custom.py` to save the images of each epoch.
8. train a network.

### Evaluation
For the evaluation of a model on a custom dataset follow these steps.
1. inside the folder created in step 5 of the training process create a new evaluation file `eval_i2i_custom.py` with the following changes to the corresponding [`eval_i2i_crowd.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/eval_i2i_crowd.py) of the Crowd dataset.
* import the the custom dataset class from the file `custom_dataset.py` (line 18) and change the Crowd class name with the new custom dataset class name in line 142
* in argument parser define the model indice, corresponding to an epoch, that will be evaluated (line 119).
* in argument parser define the save path for the predicted segmentation masks (line 124).
* (Optional) by setting True or False (default) the show_image parameter each image and the corresponding segmentation mask is shown
2. in case the name of the new [`config.py`](https://github.com/AIIALaboratory/fast_image_segmentation/blob/main/CNN-I2I/model/crowd_detection/config.py) is different, inside the file `eval_i2i_custom.py` use the name of the new created config file
3. evaluate the model.

## Citation
If you use our code or models in your research, please cite with:
```
@inproceedings{papaioannidis2021autonomous,
  title={Autonomous UAV safety by visual human crowd detection using multi-task deep neural networks},
  author={Papaioannidis, Christos and Mademlis, Ioannis and Pitas, Ioannis},
  booktitle={2021 IEEE International Conference on Robotics and Automation (ICRA)},
  pages={11074--11080},
  year={2021},
  organization={IEEE}
}

@inproceedings{papaioannidis2022fast,
  title={Fast Semantic Image Segmentation for Autonomous Systems},
  author={Papaioannidis, Christos and Mademlis, Ioannis and Pitas, Ioannis},
  booktitle={2022 IEEE International Conference on Image Processing (ICIP)},
  pages={2646--2650},
  year={2022},
  organization={IEEE}
}
```
