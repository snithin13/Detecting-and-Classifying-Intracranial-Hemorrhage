# Detecting-and-Classifying-Intracranial-Hemorrhage
![alt text](https://github.com/snithin13/Detecting-and-Classifying-Intracranial-Hemorrhage/blob/master/Images/image_1.png)

This project was undertaken as part of the Advanced Predictive Modelling course within the UT MSBA curriculum
## Contributors:
* Jacob Padden
* Nithin Saseendran
* Danyang Zhang
* Kelly Zhang
* Candice Zuo
## Project Goals:
1. Build a Convolutional neural network (CNN) to detect intracranial hemorrhaging, and classify it into the relevant sub-types
2. Leverage Transfer Learning to improve on the detection and classification using one of the pre-trained models
3. Explain how the model we built identifies and classifies intracranial hemorrhaging
## Motivation:
Human eyes can only detect approximately 6% changes in grey scale, meaning there must be at least a 120 [Hounsfield units (HU)](https://en.wikipedia.org/wiki/Hounsfield_scale) change for us to detect a difference. Intracranial hemorrhaging usually occurs within 70 to 80 HU, making these diagnoses impossible 
## Data:
* Our data was provided by the Radiological Society of North America (RSNA®) in collaboration with members of the American Society of Neuroradiology and MD.ai. 
* Data also contains a CSV file containing binary labels corresponding to each of the five hemorrhage sub-types and an additional label for ‘any’, that ignores sub-type. 
* Each DICOM file can be mapped to its labels within the CSV using its SOP Instance UID contained within the metadata. The metadata also includes various patient information. 
* The images within the DICOM files are stored as a 512 x 512 pixel arrays.
* Our data is extremely imbalanced. 94% of the images don’t have hemorrhages. Among the 6% of images depicting hemorrhages, the distribution of sub-types is also imbalanced as shown below:
![alt text](https://github.com/snithin13/Detecting-and-Classifying-Intracranial-Hemorrhage/blob/master/Images/image_4.png)
* After pre-processing, we had about 160K images in the train set and 30K images in the test set.
## Approach:
1. **Rescaling:** Many CT scan images looked different because they were in raw pixel format and had to be converted into the appropriate HU unit format. Below is an example of how images look before and after the correction.
![alt text](https://github.com/snithin13/Detecting-and-Classifying-Intracranial-Hemorrhage/blob/master/Images/image_5.png "original | RescaleIntercept = 0 | RescaleIntercept = -1000")

2. **Windowing:** The point of windowing is to extract important features from the original image. 
* Based on our research, three parts from cranial CT scans may be helpful in determining intracranial hemorrhaging: brain, subdural, and soft.
* After extracting the three layers, we stacked them back together to form a 128 x 128 x 3 array.
![alt text](https://github.com/snithin13/Detecting-and-Classifying-Intracranial-Hemorrhage/blob/master/Images/image_6.png "Using Metadata Original | Stack Layer | Three Channels | Gradient")

3. **Modeling:** 

i) CNN: We first built a CNN from scratch, to serve as a benchmark. 
* We set 3 blocks for the model, each including a convolutional and max-pooling layer. The convolution layers have 32, 64, and 128 filters sequentially, with a size of 3 x 3 and zero padding. The max-pooling layer has a 2 x 2 filter with no padding.
* Finally, we set up 3 fully connected layers: a flatten layer, a dense layer, and an output layer.
* For the activation function, we used LeakyReLu and for the output layer, we used a sigmoid activation function.
* Weighted Log Loss on Test images = 0.2247

ii) InceptionV3: For building our next model, we leveraged transfer learning using Keras. 
* We used the Inception V3 model as the base because it’s known to provide comparatively better results. 
* We pooled all lthe ayers and added a dense 6-output layer with an output corresponding to each of the five hemorrhage subtypes and the “any” type. 
* Finally, we compiled the model. Here, we tried different optimizers and chose the best one which is “Adam”.
* Weighted Log Loss on Test images = 0.2051

4. **Explainable Model using SHAP:** - To explain how our model makes classifications, we leveraged SHAPLEY values. 
* SHAPLEY values quantify how much an input feature affects the confidence of a certain classification (or non-classification). 
* In the context of image classification, each pixel is assigned a SHAPLEY value which can be visualized using a color scale.
* Since SHAPLEY values quantify how each pixel contributes to the classification of an image, we assume that pixels with high SHAPLEY values indicate where hemorrhages are and what type of hemorrhage is present. 
* Using a color scale ranging from blue to red for low to high SHAPLEY values, a proper classification’s hemorrhage area will be colored red. For proper non-classifications, the area where hemorrhage is expected for a given type will be colored blue, indicating the lack of hemorrhage.
Below is an example of this application to our data:
![alt text](https://github.com/snithin13/Detecting-and-Classifying-Intracranial-Hemorrhage/blob/master/Images/image_7.png)
