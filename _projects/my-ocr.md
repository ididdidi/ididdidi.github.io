---
title: Character Recognition
repository: my-ocr
preview: ./assets/images/projects/my-ocr/segmentation.jpg
excerpt: This program allows you to recognize text characters that are on a bitmap image. The purpose of this work is to study machine vision based on the filters of the theory of active image perception...
date: 22-05-2017
categories: projects
tags: [c++, studies]
layout: project
lang: En
---

## 1. Purpose of the program

This program allows you to recognize text characters that are on a bitmap image.
The purpose of this work is to study machine vision based on the filters of the theory of active image perception.
The program is written in **«C++»** using the technology **«OpenMP 2.0»**, therefore, it is recommended to use the development environment starting from _«Visual Studio 2008»_ and higher.

### 1.1. The task of recognition
The task of recognizing text information can be defined as follows. 
The input is a text image. It is required to determine its encoded electronic representation, i.e. to translate it from a bitmap graphic representation to a text one. The latter means that for each image of the letter of the text, you need to determine the corresponding encoding number. This means that each fragment of the original image corresponding to a letter must be assigned to one of the classes of letters, the set of which is given by the known alphabet. Images of letters of different classes differ in their shape. The output text representation of the manuscript should contain the entire set of letters represented in the original image, in the appropriate order. 
Thus, the task to be solved is the task of multiple image recognition of the letters of the text.

### 1.2. Source data:
*	A bitmap image with alphanumeric or numeric characters.
*	The image size can be any size.
*	The font and the relative position of the characters can be arbitrary.

### 1.3. The application must perform the following tasks:**
*	Uploading an image to the app.
*	Segmentation of content into characters.
*	Recognition of characters similar to the reference standards.

### 1.4. Output data:
*	A table of the found symbols with an indication of their location and the value that characterizes their similarity to the standards.

## 2. Project structure

File name       | The contents of the file
----------------|-------------------------
stdafx.h        | Connecting standard libraries
main.h          | Declaring variables, classes, methods, and functions
main.cpp        | The main project file, contains the **main()** function
Source.cpp      | Description of all methods and functions
MatchBase.dat   | Training sample data

## 3. Theoretical information

Let's consider the process of text recognition as part of a point graphic image on the example of captcha analysis. 

![Pic. 1. Captcha](/assets/images/projects/my-ocr/01.bmp?raw=true "Pic. 1. Captcha") 

A captcha is a computer test used to determine whether a system user is a human or a computer. On the computer monitor, we see an image of the symbols of letters and numbers (a graphic image). But a graphic image is not yet a text document. A person only needs to look at a piece of paper with a text on it to understand what is written on it. From the computer's point of view, an image is a collection of colored dots, not a text document at all. Such tasks are solved with the help of special software tools called image recognition tools. Such systems are called OCR (Optical Character Recognition), and rely on specially designed fonts that facilitate this approach.

Recognition systems have the following typical functional scheme:

![Pic. 2. Functional diagram of the recognition system](/assets/images/projects/my-ocr/Functional_diagram_OCR.jpg?raw=true "Pic. 2. Functional diagram of the recognition system")

### 3.1. Input data
The input data to be recognized is fed to the input of the system and is preprocessed in order to transform it into the form necessary for the next stage or to extract the necessary characteristic features from it. Next, at the decision-making stage, a number of calculations are performed on the processed data array and, based on their results, a response is formed containing the information expected from the system about the input data. The content of the input and output data is determined by the purpose of the system.

### 3.2. System training
The task of training the system is to form in its memory a set of information necessary for recognizing the intended class of input data. Depending on the specifics of the problem being solved learning can be expressed by a single procedure manual setting of parameters of the system by its developer, an automatic procedure for determining the optimal parameter values as a result of the training cycles recognition or process of continuous adjustment of parameters in the analysis generated by the system answers. As a rule, there is a combination of these approaches.

At the preprocessing stage, we solve the problem of creating a formalized description of recognition objects that can be used by the recognition algorithms themselves. Usually, the initial data about the observed objects is presented in a form that is not suitable directly for recognition. These can be bitmaps, audio files, statistics (numeric sets), video recordings, and so on. Some recognition algorithms require a higher-level representation. This leads to the need to perform one or more transformations of the source data, moving from code 0 to code 1, 2, etc. In our case, it is necessary to examine a bitmap image of any available format(JPG, PNG, etc.)

For character recognition, we use the point brightness value. The brightness of the point is determined by the formula, the coefficients of which are determined by the properties of human vision:
**Y:=0.3\*R+0.59\*G+0.11\*B (1)**

The division of the set of objects under consideration into classes can be specified by an enumeration. Each class is defined by directly specifying its members. This approach is used if there is complete a priori information about all possible recognition objects, it is called a _«Reference approach»_. The images presented to the system are compared with the specified descriptions of class representatives and belong to the class that the most similar samples belong to. The decision to assign incoming objects to a particular class is made on the basis of a measure of similarity with the standards. The decision function is based on the analysis of reference objects and a priori information about their belonging to different classes.

### 3.3. The distance function
When constructing classification rules based on distance functions, the assumption is used as a prerequisite that the natural indicator of the similarity of images is the degree of proximity of the points describing these images in Euclidean space.

![Pic. 3. The distance function](/assets/images/projects/my-ocr/Euclidean_metric.jpg?raw=true "Pic. 3. The distance function") 

Considering the example in the figure, we can intuitively refer the object _**«x»**_ to the class _**«ω1»**_, based on the considerations that the point describing it is located closer to the objects of this particular class than to the objects of the class _**«ω2»**_. When using distance functions, feature classes are represented as clusters in a parametric space. At the recognition stage, the criterion of the minimum distance between the point of the recognized object and the cluster of the class to which this object should be assigned is used. Building classification rules is about building clusters in an optimal way. 
It should be noted, however, that this technique is effective only if the clustering properties of the object classes under consideration are present, i.e., when the distance between objects within the classes is significantly less than the distance between the groups of points forming the classes.

Consider the operation of classification rules based on distance functions. In some cases, the objects of each class are grouped around a single image that is typical or representative of that class. In this case, it is said that the class is represented by its only reference. For example, this phenomenon is observed when recognizing printed characters. In this situation, it is effective to apply the criterion of the minimum distance between the recognized object _**«x»**_ and each of the standards of the available classes _**«z1»**_,_**«z2»**_,…, _**«zM»**_.

The distance between the image _**«x»**_ and one of the standards _**«zi»**_ in Euclidean space is defined by the function **1**):

![Pic. 4. Euclidean metric](/assets/images/projects/my-ocr/Euclidean_metric_2.jpg?raw=true "Pic. 4. Euclidean metric")

### 3.4. Filters of the theory of active image perception
The condition for their use for recognition is the preliminary selection of characters from the source image.
To get the boundaries of symbols, we use one of the filters of the theory of active perception (**f1**):

![Pic. 5. Filter f1](/assets/images/projects/my-ocr/f1.jpg?raw=true "Pic. 5. Filter f1")

By applying a filter to an image or part of it, we mean calculating the value of a function **2**:

![Pic. 6. Filter overlay function](/assets/images/projects/my-ocr/f1(x).jpg?raw=true "Pic. 6. Filter overlay function")

Where:
*	**k** - the filter number
*	**mi**,**j** - the total brightness of the corresponding filter area

Based on the obtained data on the position of the extremes of this function, it is possible to build hypotheses about the location of the beginning and end of each character in the image. Each of the segments on the abscissa axis between the extremes can be the location of the symbol.

![Pic. 7. Graph f1 (x)](/assets/images/projects/my-ocr/segmentation.jpg?raw=true "Pic. 7. Graph f1 (x)")

Due to the fact that the number of segments significantly exceeds the number of characters, in order to more specifically and accurately localize the position of the characters, the following parameters were introduced:
*	Minimum interval;
*	Maximum interval;

As a result, we added conditions for comparing the lengths of segments between extremes with the values defined by the user, such as the minimum and maximum width of the symbol. All segments that were less than the minimum or greater than the maximum width of the letters were not considered in the future. This allowed us to filter out a significant part of the intervals from the studied feature space.

The remaining segments are overlaid with 16 filters of the theory of active perception:

![Pic. 8. Filters of the theory of active image perception](/assets/images/projects/my-ocr/filters.jpg?raw=true "Pic. 8. Filters of the theory of active image perception")

Their superposition also implies the calculation of the value of function **2**, discussed above. The result of these calculations is the coordinates of a point in the 16-dimensional feature space. Using these coordinates, we find the Cartesian distance (function **1**) between the intended symbol and the pre-prepared standards, the coordinates of which were obtained in a heuristic way. Then, comparing the obtained distances, we choose the closest standard, the symbolic value of which will be considered the value of the found interval.

Since the found segments between the extremes can overlap, another parameter was introduced – the maximum percentage of overlap. From the two segments, the percentage of which exceeds the maximum allowed, the one that is closest to the standard is selected. As a result, only the values of the segments remain, the percentage of overlap of which is strictly less than what the user sets.

## 4. Compilation

A special feature in the compilation of this project will be the inclusion of OpenMP support. You can read more about this in the article [Параллельные заметки №2 – инструментарий для OpenMP](https://habrahabr.ru/company/intel/blog/83504/)

## 5. Working in the program

После запуска программа просит выбрать один из двух режимов работы.

![Pic. 9. Selecting a mode](/assets/images/projects/my-ocr/mode_selection.jpg?raw=true "Pic. 9. Selecting a mode")

### 5.1. System Training
To perform the training, you need to enter the filter width and the offset step-parameters that affect the number of local extremes found.
The narrower the filter and the smaller the step, the more local minima the program will detect in the image as a result of applying the **f1** filter.

![Pic. 10. Parameters](/assets/images/projects/my-ocr/enter_parameters.jpg?raw=true "Pic. 10. Parameters")

Next, you need to enter the boundaries of the intended character.
If the program finds extremes in these boundaries, it takes them for the exact boundaries of the symbol and offers to save these values to the database, marking them with the necessary symbol.

![Pic. 11. The training process](/assets/images/projects/my-ocr/training.jpg?raw=true "Pic. 11. The training process")

This process is repeated for all characters.

### 5.2. Recognition
In addition to the filter width and offset step length, the recognition quality is affected by parameters such as the minimum character width, maximum character width, and the percentage of overlap. These parameters are necessary in order to cut off as many false recognition hypotheses as possible. It is assumed that the minimum and maximum character width are known in advance. The percentage of overlap is determined by the distance between the characters If the images of individual characters are arranged so that they do not overlap, the percentage of overlap is zero.

![Pic. 12. Recognition parameters](/assets/images/projects/my-ocr/detection_settings.jpg?raw=true "Pic. 12. Recognition parameters")

If all the parameters are selected correctly, the program recognizes all the characters in the image, even if their order is changed.

![Pic. 13. Character Recognition](/assets/images/projects/my-ocr/detection.jpg?raw=true "Pic. 13. Character Recognition")

But if you change the slope of the letters, the font, or add distortions, the program will produce erroneous values.

![Pic. 14. Recognition of cursive](/assets/images/projects/my-ocr/detection_k.jpg?raw=true "Pic. 14. Recognition of cursive")

![Pic. 15. Recognition of distorted characters](/assets/images/projects/my-ocr/detection_f.jpg?raw=true "Pic. 15. Recognition of distorted characters")

In order to improve the quality of recognition, you need to add images to the training sample with the variants of the letters that we are going to recognize.

![Pic. 16. Training on a distorted sample](/assets/images/projects/my-ocr/training_f.jpg?raw=true "Pic. 16. Training on a distorted sample")

![Pic. 17. Recognition of distorted characters-2](/assets/images/projects/my-ocr/detection_f2.jpg?raw=true "Pic. 17. Recognition of distorted characters-2")

## 6. Conclusion

The advantage of this method is the relative ease of implementation. 
The disadvantage is the weak resistance to noise and distortion in the recognized images. This is due to the nature of the recognition process: each recognized object is compared pixel by pixel in turn with all the standards known to the system. In addition, recognizing linear character transformations requires some effort at the preprocessing stage.

Of course, there are many directions for the development of this program:
* It is necessary to train the system by directly specifying images of all recognized characters (i.e. by setting standards): A B C ..... a b c ... 1 2 3....
* It should be noted that if the recognition of italic, bold, or other font characters is assumed, then this approach will require you to present each variant of the font character.

_Thank you for your interest! :)_
