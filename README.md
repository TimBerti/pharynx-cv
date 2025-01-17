# Pharynx CV

This project investigates the muscular activation in the mouth and throat during equalizing. Equalizing is the process in which the pressure in the middle ear is equalized with the pressure in the environment by opening the Eustachian tube. It is a common technique used by scuba divers to prevent Barotrauma.

There are two types of equalizing: Valsalva and Frenzel. Both techniques involve closing the mouth and nose and blowing air into the Eustachian tube. In the Valsalva technique, the air is blown from the lungs, while in the Frenzel technique, the necessary pressure is generated by the tongue and throat muscles while closing the Glottis. The aim is to find out which muscles are used in the Frenzel technique.

To achieve this, pictures of the mouth and throat area are taken during three states:
- Resting (open Glottis)
- Conscious closing of the Glottis
- Frenzel equalizing

Transfer learning was used to train a pretrained ResNet18 Model to classify the images into these three categories, and segmentation and LIME were used to find out which parts of the image were most important for the classification.

## Data
The pictures were taken with a €20 endoscope from Amazon. The endoscope was held in place by a 3D-printed holder, which was designed using bite marks on a piece of paper, that were imported into [FreeCAD](https://www.freecadweb.org/). 

![Sketch of the endoscope holder](images/endoscope_holder_sketch.jpg)
![3D model of the endoscope holder](images/endoscope_holder_model.jpg)

The endoscope holder was printed with an [Anycubic Kobra 2 Neo](https://www.anycubic.com/products/kobra-2-neo) with a 0.4mm nozzle and 0.2mm layer height.

![3D-printed endoscope holder](images/endoscope_holder.jpg) 

In total, about 200 pictures were taken per category. The images were resized to 256x256 pixels, randomly rotated by up to 10 degrees, and randomly cropped to 224x224 pixels to augment the data. They were then normalized with the mean and standard deviation of the [ImageNet](https://www.image-net.org/) dataset.

![Example images](images/transformations.png)

## Training
The model was trained in [notebooks/train.ipynb](notebooks/train.ipynb) on an RTX 3070. Cross Entropy Loss with label smoothing was used as the loss function, and SGD with momentum and weight decay as the optimizer. The model was trained for 10 epochs, with a learning rate reduction of a factor of 10 after 5 epochs.

![Training and validation loss and accuracy](images/epochs.png)

On the validation set, the model achieved an accuracy of 98.86%. 

## Segmentation
To find out which parts of the image were most important for the classification, the images were segmented, and the LIME values of the segments were computed. As segmentation algorithm, the SLIC algorithm from the [scikit-image](https://scikit-image.org/) library was used. 

![Segmentation of an image](images/superpixel.png)

Then the `LimeImageExplainer` from [LIME](https://lime-ml.readthedocs.io/en/latest/index.html) was used to find out which superpixels were most important for the classification. The segmentation and analysis were done in [notebooks/lime_visualization.ipynb](notebooks/lime_visualization.ipynb). 

![LIME explanation of an image](images/lime_pixel.png)

## Analysis
The 3 most important superpixels for each image were computed in [scripts/lime_superpixels.py](scripts/lime_superpixels.py). The mouth and throat area was then divided into the regions shown below, and for each category, the frequency, of how often each region was among the 3 most important superpixels was counted.

![Regions of the pharynx](images/pharynx_segments.jpg)
![Frequency of regions in the 3 most important superpixels](images/segment_frequency.png)

## Conclusion
Comparing the frequencies, it seems like Frenzel equalization is more associated with the relaxed state, but with a higher activation in the regions `B` and `F` to the sides. 

A possible explanation of the closer resemblance to the relaxed state, even though Frenzel equalization involves closing the Glottis, could be that the conscious closing of the Glottis exaggerates the activation of the muscles in the front of the Pharynx, while the Frenzel equalization is more focused on the back of the Pharynx and not as exaggerated. 

The high activations of the side regions `B` and `F` could point to an involvement of the Styloglossus muscle, which attaches to the back and to the side of the tongue, and the Stylohyoideus muscle, which attaches to the back and to the side of the Hyoid bone.

![Styloglossus muscle](images/styloglossus.png)

Source: [https://en.wikipedia.org/wiki/Styloglossus](https://en.wikipedia.org/wiki/Styloglossus)
