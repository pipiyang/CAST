CAST-multi-scale-CNN-for-segmentation-of-hippocamal-subfields
===============================================================
### Instruction

### 1. Required libraries
- [Python](https://www.python.org/downloads/): Python 3 by default, the script is tested on Python 3.6.8.
- [Tensorflow](https://www.https://www.tensorflow.org/): Python 3 by default, the script is tested on Python 3.6.
- [deepmedic](https://github.com/pipiyang/deepmedic): The deepmedic is revised based on https://github.com/deepmedic/deepmedic. When we run the original code on a multiple GPU workstation, it will exhaust all the memories but without reducing computational time. I have modified the code to enable multiple GPUs train a model separately.
### 2. Training and segmentation
Both training and segmentation pipelines is carried out by running the command CastRun.
CastRun [options]
* -path [pathdir] 
  directory of the model configuration. 
* -model [modelconffile] 
  directory or filenames of the configuration file for the deep neural network. The parameters used to setup the network architecture are included in this file.
* -train [trainconffile] 
  directory or filenames of the configuration file for the training pipeline. The parameters used to setup the training scheme are included in this file.
* -test [testconffile]
  directory or filenames of the configuration file for the testing pipeline. The parameters used to setup the testing scheme are included in this file.
* -load [trainedparametersfile]
  directory or filenames of the trained parameters.
* -dev [cudaname]
  cudaname
  
  If [pathdir] is specified, then only the filenames should be specified for option [modelconffile], [trainconffile], [testconffile] and [trainedparametersfile]. [pathdir] can be absolute directory or relative directory. In this case, model configuratopm file is under the directory [pathdir]/configFiles/deepMedic/model/[modelconffile], training configuration file is under the directory [pathdir]/configFiles/deepMedic/train/[trainconffile], testing configuration file is under the directory [pathdir]/configFiles/deepMedic/test/[testconffile], trained parameters is under the directory [pathdir]/output/saved_models/trainSessionDm/[trainedparametersfile]
  
  If [pathdir] is not specified, then the relative or absolute directory for [modelconffile], [trainconffile], [testconffile] and [trainedparametersfile] should be included.
  
  #### 2.1. Training pipeline
  
  In the training pipeline, besides [trainconffile] files, the paths to the input images and the manual segmentation maps should be specified. In the mni_seg model, since there are two input imaging modalities for hippocampal subfield segmentation, there are two files (trainChannels_t1w.cfg and trainChannels_t2w.cfg) to specify the paths to these images. In addition, the paths of manual segmentation maps are saved in trainGtLabels.cfg. Similar files are also required for validation. The names of these files are in trainConfig.cfg, thus the algorithm can find the data to train the network. These files can be easily revised for another dataset. 
  
  Training a new model by running 
```cshell
./CastRun -model ./mni_seg/configFiles/deepMedic/model/modelConfig.cfg \
          -train ./mni_seg/configFiles/deepMedic/train/trainConfig.cfg -dev cuda0
```
or
```cshell
./CastRun -path ./mni_seg -model modelConfig.cfg \
          -train trainConfig.cfg -dev cuda0
```
cuda0 can be revised to the names of other available GPUs in your workstation, such as cuda1, cuda2, etc. GPU is required becase of the intense computation in the training pipeline.
-load option can also be used for training pipeline, which is useful if training was previously interrupted or you want to fine tune the network. 

  #### 2.2. Segmentation pipeline
  Similar to the training pipeline, a text file having the paths to the input images are required and the filename is included in testConfig.cfg file. The text file for the manual segmentation is not required, if it does exist, CAST outputs the probability map for each label and the hard segmentation map. The dice coefficient for each label will also be calculated when performing segmentation. If it does not exist, then certainly  segmentation pipeline does not calculate dice coefficient.
  
  Segmenting new subject(s) by running
  ```cshell
./CastRun -model ./mni_seg/configFiles/deepMedic/model/modelConfig.cfg \
          -train ./mni_seg/configFiles/deepMedic/test/testConfig.cfg -load [trainedparametersfile] -dev cuda0
```
or
```cshell
./CastRun -path ./mni_seg -model modelConfig.cfg \
          -test testConfig.cfg -load [trainedparametersfile] -dev cuda0
```
  GPU (e.g. -dev cuda0) is not required for the segmentation pipeline. With GPU disabled, CAST segments a new subject in MNI dataset in one minute. With GPU enabled, CAT segments a new subject in MNI dataset in 10 seconds (for Tesla K40c GPU card). 
