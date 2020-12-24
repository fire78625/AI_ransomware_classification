# PE_malware - AI_ransomware_classification
Introduce the cencept of our model which was used to attend the AI competition and won 2nd place

Moreover, showing how we applied it to classify ransomware samples with our own defined categories and the results 

Authors: Owen Chen, Ping-Chung Chen, Chih-Wei Chen

Source category:

(1) Source code of training and testing (/source_code)

(2) Example samples for testing (/testsample)

(3) Figures of execution results (/result_showing)

## Introduction

The original model attends the AI competition whose name is "2019 SecBuzzer AI UP! AI Information Security Challenge" which was hold by III (Institute For Information Industry)

Index page of AI competition: https://competition.secbuzzer.co/

The evaluation standard of competition is the accuracy of classification, and our model holds at least 50% accuracy to each category of malware

### Analysis process

Use feature extracting techniques: Select Hex code, OpCode, and Hex image to extract features.
Then an AI model which is trained by these features will be used to identify the category of malware.

![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/malware_analysis.JPG)

(1) Hex code

Find features from Hex file, and use model to find relation between features and functions
![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/hex_code_analysis.JPG)

(2) OpCode

Using IDA Pro to convert the .asm file of OpCode, then use keywords analysis to transfer Opcodes to Vector
![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/opcode_analysis.JPG)

(3) Hex image

Transfer Hex code into gray image, then use Fourier transform to describe better features of image
![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/hex_image_analysis.JPG)

### Diagram of model

Using stacking model to training and testing, then merging the results of different models to balance result
![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/stack_model.JPG)

### Classification result

Accuracy > 50% in each category of malware
![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/accuracy.JPG)

## Experimental environment setting
### Operation System: Ubuntu 16.04 LTS Client (or at least Linux-based system recommended)
### install steps:

(1) Install anaconda from Offical website: https://docs.anaconda.com/anaconda/install/windows/

(2) Construct virtual environment by anaconda: https://medium.com/python4u/%E7%94%A8conda%E5%BB%BA%E7%AB%8B%E5%8F%8A%E7%AE%A1%E7%90%86python%E8%99%9B%E6%93%AC%E7%92%B0%E5%A2%83-b61fd2a76566 [python version >= 3]

(3) Check tools version from environment.txt in directory source_code. Most tools should have been installed when virtual environment construct success
!!! We recommend to install the same version of tools to avoid unexcepted conflicts

You can check by open example.ipynb and execute the first 4 cell to confirm that do confilcts exist or not 

### Categories (self-definition) & samples amount in training set

We manually classify the ransomware into 10 categories according to the names from Kaspersky in VirusTotal

Category name                      |          amount

Trojan-Ransom.Win32.Blocker.xxx       |       1103

Trojan-Ransom.Win32.GandCrypt.xxx     |       911

Trojan-Ransom.Win32.Foreign.xxx       |       317

Trojan-Ransom.Win32.PornoBlocker.xxx  |       63

Trojan-Ransom.Win32.Wanna.xxx         |       163

HEUR:Trojan-Ransom.xxx.Blocker.gen    |       110

HEUR:Trojan-Ransom.xxx.Gen.gen        |       14

Trojan-Ransom.PHP.xxx                 |       1

HEUR:Trojan-Ransom.Win32.xxx.vho      |       65

Others                                |       4886  (72.3%)

 
## Usage

Extract souce code from the zip file 'ML_model.zip' first 

### Training phase
First, each original binary executable in training set needs to use tools (ex: IDA Pro) or commands (ex: xxd) to generate 2 category files: .asm file and .byte file

Afterwards, the inputs of training phase are binary executable(PE), .asm file, and .byte file 

The outputs of training phase are 5 models whose file format is .pickle

The categories of output model: Random Forest(RF), XGBoost(XGB), LightGBM(LGB), Pytorch, stack

#### Training steps:

If anaconda and others packages install completely, you can type "jupter notebook" command in command line.
The next steps will process in jupter notebook.

(1) Feature generate stage [File format: csv]

Using extract_feature.ipynb to generate a file which record the extracted features of binary files (1st ~ 6th cell). First parameter of sample should be filename(without extname) and the others are the extracted features in order [Needed inputs are binary executable, .asm file, and .byte file. Output is a csv file]

!!! However, you need to modify several paths first

1. In 2nd cell, you have to modify the paths of your binary files, asm files, and byte files directory

2. In 5th cell, you have to modify the errormsg saving path and its filename [There are 2 error msg file, one records error in predict function, the other is the generate features amount does not match what we want.]

3. In 6th cell, you need to modify the output file [features file] location

Executing filterbyte.py to generate bytefile. Before executing, please modify the targetdir to your binary file save location and bytedir to the location you want to save bytefile. [Notice it used Linux command xxd, so need to execute in Linux-based environment]
    
(2) Label generate stage [File format: csv]

Using Samplelabel.py to generate a file which record the filename and its label. In this file, each sample whose first parameter is filename, and second is label. [Input is binary executable, and output is a csv file]

!!! Need to modify the path in python file first

1. modify VirusTotal report path (reportpath), report name(targetfile), and the save position of new label file directory (labelfilespath)

2. modify label files save path in logfile variable

3. modify path of open VirusTotal report

(3) Open model_training.ipynb and execute the cells (from 1st to 5th) by order then you can acquire the 5 output models in specific path that you have written. [Inputs are the 2 csv file, and the outputs are 5 models]

!!!Before execute the cells of model_training.ipynb, you need to modify some paths first.

1. In second cell, you need to modify the path of csv files which record the features and the labels of training samples.

2. In third cell, you need to modify the integer in each np.zeros() from the original integer 10 to the amount of your own classification.

3. In fourth and fifth cell, the save path of generated models should be modify to location of your machine

### Training sample list

We provide the md5 list of our training samples which download from VirusTotal in directory source_code

### Testing phase
Similarly, each binary executable needs to generate .asm file (ex: IDA Pro) and .byte (ex: xxd) file at first.

Inputs are the same as training phase. That is, binary executable(PE), .asm file, and .byte file

Output format: ndarray

#### Testing steps:

Execute the command: python3 Example.py <bytesfile_path> <asmfile_path> [output is a ndarray]

### Testing sample

We provide a zip file testsample.zip which contains 10 testing samples in directory testsample. 

#### Testing result

If you follow the testing steps, and you can acquire a response whose format is Dataframe. The visualization of response is displayed below.

The left column list the names of category, the right column are the values of possibility of prediction. The largest value is the prediction result

![](https://github.com/fire78625/AI_ransomware_classification/blob/main/result_showing/ai_single.jpg)

