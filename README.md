Homology-assisted Convolutional Neural Network for image classification
=============
by Shizuo KAJI (skaji@imi.kyushu-u.ac.jp)

Given a set of labelled images, this algorithm solves the common classification problem by convolutional neural network (CNN).
The algorithm first computes "homology images" using _persistent homology_.
Persistent homology is a popular tool in _Topological Data Analysis_ (TDA), which captures the topology or the _shape_ of data.
A homology image is a greyscale image of the same dimension as the input, with the generators of 1-cycles drawn with the intensity according to their lifetime (or length). 

![homology image]("https://github.com/shizuo-kaji/HomologyCNN/blob/master/homology.jpg?raw=true")

The homology image will be bundled (as an extra channel) with the original image and fed into a CNN for classification.

This is a kind of feature engineering, where the CNN is supplemented with homology features which encode global information.
In general, CNNs are better at learning local features than global features. 
The idea is to _teach_ global information to CNN in the form of homology image.

This _mariage_ of machine learning and mathematics generally performs better than CNN alone.
The technique is independent of network structure, and can be used straightforwardly in conjunction with existing systems.

## Licence
MIT Licence

## Requirements
- Python 3: [Anaconda](https://www.anaconda.com/download/) is recommended
- Python libraries: Chainer, cupy chainercv, chainerui:  `pip install cupy chainer chainerui chainercv`
- R: [Microsoft R Open](https://mran.microsoft.com/open) is recommended
- R libraries: TDA,imager,ggplot2: `install.packages(c("ggplot2","TDA","imager"))` from R
- CUDA supported GPU is highly recommended. Without one, it takes ages to do anything with CNN.

# How to use
The usage will be demonstrated with a texture classification problem using the
[KTH-TIPS2-b](http://www.nada.kth.se/cvap/databases/kth-tips/index.html) dataset.

Download the KTH-TIPS2-b dataset and extract the archive.
Copy all png files into a single directory, say, named "dataset/texture".

First, we need one text file for each train/test dataset, containing lines with
```
    "ImageFileName   class"
    "ImageFileName   class"
    ...
```
I have included sample files for KTH-TIPS2-b under "datatxt".

Homology images should be computed in advance. We use R for this part.
This procedure takes a bit of time.
```
Rscript compute_PH.R dir png
```
produces persistent homology images from image files under "dir" with filename extension "png".
Note that "dir" should be the full path for the directory containing the images.
Homology images will be put under "dir" with some suffix like "_Hsup1_life" in the file names.
Modify the beginning of the R script "compute_PH.R" to tune parameters, if you wish.

Other tasks will be done by the python script. 
```
python train.py -h
```
gives a brief description of command line arguments.

A typical training is performed by
```
python train.py -t datatxt/kth-abc.txt --val datatxt/kth-d.txt -R dataset/texture -a nin -e 200 -op Adam --num_class 11 -hi Hsup1_life
```
Logs and learnt model files will be placed under "result" directory. 

You can also train a CNN without using homology.
```
python train.py -t datatxt/kth-abc.txt --val datatxt/kth-d.txt -R dataset/texture -a nin -e 200 -op Adam --num_class 11
```
Compare the performances.

Inference using a learnt model is done by
```
python train.py --val datatxt/kth-d.txt -R dataset/texture -a nin --num_class 11 -hi Hsup1_life -p model_epoch_100
```
"model_epoch_100" is the model file produced by training. 