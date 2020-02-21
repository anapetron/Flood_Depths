# Extracting Flood Depths from Ground-Based Imagery
## Ana Petronijevic, Ze'ev Gebler, Ben Peck


## Contents:
- Problem Statement
- Guide to Files
- Proposed Process
- Methods and Complications
- Conclusions


## Problem Statement:

Our team was given the following problem statement from the client:

"Floods cause damage to infrastructure and homes. The depth of flood waters is a good indicator of the severity of damage. Floods are incredibly difficult to model, and while model outputs are useful to emergency managers, it is crucial to know the actual depth. Social media and news outlets often present pictures of floods. How can this imagery be used to estimate the depth of water in a given area?"

In order to identify a more defined problem to address, we increased the specificity of the problem statement to the following:

"Given a ground-based image of a flooded area, estimate the depth of flooding present in that image."

As such, collection of images from original sources, classification to determine whether flooding is present, and comparisons involving multiple images of the same area are outside the immediate scope of this analysis.


## Guide to Files:

https://drive.google.com/drive/u/0/folders/1XChxhJE18X-cMlWFAeOQvY2CEexgt_aW?ths=true

## Proposed Process:

In order to estimate damage, we researched how to use convolutional neural networks to predict the flood depth, which could help determine the amount of damage that a place has incurred. With no accompanying image of the target area without inundation, a process using "reference objects" was chosen. The concept of this approach is to identify objects of known average or standard dimensions in the image, measure the extent to which said objects are obscured by floodwater, and infer the depth of the floodwater as the obscured portions of the objects' known heights. This process can be subdivided into two clearly separate steps: identification and detection of a set of reference objects, and defining which portions of an image consist of floodwater in order to obtain an accurate measurement.

1. Reference Objects: Neural networks can be extremely effective at classifying objects within an image, but need to be trained on specified classes for which to search. Datasets like MS COCO can be utilized to train the neural network on recognizing a large set of objects, such as houses, people, bicycles, cars, and other objects commonly found in pictures. Recent literature indicates that convolutional neural networks (aka CNNs) are able to perform the computer vision tasks required for this problem in the most effective way.
2. Defining Floodwater: Image segmentation and edge detection are also valuable tools for this analysis. In order to predict the actual flood, the model would have to be able to recognize the physical boundaries of the flooded area. Distinguishing where the flood meets the trained objects would allow the CNN to separate the two. One method for this is edge-detection - looking for lines of water in an image distinct from expected lines such as sidewalks, streets, and horizons. Another is segmentation via thresholding or clustering based on pixel values. In tandem, the two should make it possible to determine where a reference object enters floodwaters, if at all.


## Methods and Complications

Data Gathering:
Images of floods are quickly shared through social media, news outlets, and various other sources. This provides an opportunity to estimate flood damage in the area a photo was taken. To simulate images of flooding gathered from social media, we used a set of images selected from the [European Flood 2013 Dataset from Jena University](https://www.inf-cv.uni-jena.de/Research/Datasets/European+Flood+2013.html). For a discussion of potential data collection methods for a finished project, see the “Conclusions and Further Steps” section.

For a proof of concept, the reference object we chose to focus on was traffic signs. Traffic signs are present in many flood images, of standard sizes, and datasets of traffic sign images and neural networks pre-trained on said datasets already exist. Traffic signs also have the major benefit of being vertical, immobile pieces of street furniture which tend to emerge recognizably from even very deep floodwater. If the general methodology works, the process can be expanded by simply adding more reference objects to those known by the CNN.

Our CNN (based on this framework by [Sanket Doshi](https://towardsdatascience.com/traffic-sign-detection-using-convolutional-neural-network-660fb32fe90e)) performed fairly well identifying signs, with a 93% accuracy rating. Moving further, however, proved problematic. A number of issues were encountered, including the following:

- Heavily submerged objects - depending on how obscured an object is, the CNN might not be able to detect it accurately. When an area is so heavily inundated that no reference objects protrude above the surface, no estimate can be made
- Natural objects - trees and other natural objects vary greatly in size and the obscured portion is accordingly difficult to estimate.
- Diffuse reflection - highly varied intensity caused by light reflecting off the water’s surface makes images more difficult to segment
- Specular reflection - segmentation could potentially segment the water from the object but also its reflection, which is actually the floodwater
- Depth variation - floodwater depths can vary across a single image, with geographic features, slopes, and angles of the picture making it likely that flood waters will appear as different depths
- Natural bodies of water - in some photos where river banks have overflown, the natural boundary of the water is unclear

Diffuse and specular reflection posed the greatest challenges for defining the region occupied by floodwater, as standard edge detection and image segmentation methods tended to separate on the intensity variations and phantom objects these created. After some experimentation, it was discovered that the relationship between the red and green channels in an RGB image stays very consistent across a region of floodwater, with both moving in lockstep across even extreme intensity variations. The blue channel was more sensitive to intensity variations, especially when diffuse reflection caused all three channels to spike suddenly. In order to control for diffuse reflection while identifying flooded areas, a version of the target image was created consisting of a grayscale representation of the ratio between the red and green channels. No such solution could be found to limit the interference caused by specular reflection. In many cases even the human eye struggled to define boundaries in images affected by specular reflection.

## MASK R-CNN

P. Chaudhary, et al. in “FLOOD-WATER LEVEL ESTIMATION FROM SOCIAL MEDIA IMAGES,” propose a methodology of using a Convolutional Neural Network (CNN) to identify objects within an image, and to calculate the relative depths of each image based on its submersion levels. Then a global depth is calculated across all objects in the image.

The MASK R-CNN architecture has demonstrated success in detecting and classifying objects within an image. After performing its convolutions, the MASK R-CNN segments and identifies objects based on weights from the training phase. Once an object is identified, the model then calculates how much of that object is obscured by the floodwater, based on pretrained flood level weights.

Once Flood objects are detected, we want to train our NN network on the depths of floods, for use in later testing. Chaudhary, et al. use the method of training their model on a dataset of flood images that are manually annotated with flood levels, calculated relative to reference objects in the image. So for example, if a human was totally submerged, that would be a level 10 flood, if a bike were submerged, a level 6 flood. This trains the model to calculate the amount each identified object is submerged.

To obtain a global water level for the image, each object's relative flood depth can be combined using a trimmed mean, or other aggregation methods.

## Conclusions

This approach has potential, but a number of unresolved issues, further steps which could be taken, and alternate approaches which could be used remain.

Unresolved Issues:

- Significant engineering to create pipeline data collection
- Development of synthetic weight algorithm to pre-classify images
- Mask R-CNN needs to optimized to ensure real-time flood-depth estimations
- Better for more suburban and metropolitan areas versus rural areas
- Unable to limit interference from specular reflection
- A single output from images displaying varying depths may be unclear in practice

Further Steps:
- Data Gathering:
  - In order to effectively estimate flood depths, a large amount of images from the area in question should be gathered. Social-media pictures, taken on the ground from people in the affected area, provide the most current, real-time data on flooding
  - Twitter, Snapchat, and Instagram stories provide the richest source of data, but have different levels of access available for downloading images. Regional stories can be searched on Snapchat and Instagram, but there are not currently automated APIs for the large scale automated collection of images. Twitter provides a developer API that streamlines the process
  - Ideally, other sources can be leveraged, such as local media outlets, traffic and security cameras, as well as traditional satellite images to create a region of interest in which to search for ground-based images of flooding
- Addition of further reference objects
  - The more reference objects the model is trained on, the more images it will be able to effectively analyze
- Incorporate Metadata
  - Precise geographic or chronological metadata would allow for a more complete profile of a flood
  - Sufficient automatic input scraping would allow for near-real-time output of metadata and depth estimates in a user-friendly format

Alternative Approaches:

- Leverage Google Earth and Streetview to create reference dataset of dry-images, so that comparisons of images at a shared location can create a flood depth calculation, minimizing the amount of time spent preprocessing images.
- Limit data gathering to fixed-location cameras such as traffic and security cameras to streamline estimation and allow rapid reporting on changes in depth over time
