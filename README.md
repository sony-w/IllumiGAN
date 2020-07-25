# IllumiGAN: a low-light video enhancement system  

### **W251-Summer 2020: Final Project**  
### *Team: [Lina Gurevich](mailto:lgurevich@berkeley.edu), [William Casey King](mailto:caseyking@berkeley.edu), [Neha Kumar](mailto:neha.kumar@berkeley.edu), [Sony Wicaksono](mailto:aji.wicaksono@berkeley.edu)*

![](/assets/earth_540_gan.gif)
## Introduction and Project Motivation  

Images and videos captured in low-light conditions suffer from low contrast, poor visibility, and noise contamination. Those issues challenge both human visual perception that prefers high-contrast images, and numerous intelligent systems relying on computer vision algorithms.  

Concretely, analyzing security camera footage that was recorded during nighttime or in a poorly lit area for the purposes of crime scene reconstruction could greatly benefit from enhancements that present the video in a better light.  
Another potential application is a real-time enhanced rendering of the environment during nighttime driving. A dashboard camera with such functionality can both aid drivers better navigate in poor visibility conditions and generate better quality input signals for automatic processing in a self-driving vehicle.  

This would be a challenging task for traditional contrast enhancement algorithms since nighttime images usually contain both high-intensity and very low-intensity regions that often cause detrimental artifacts in the enhanced image. In the last decade, we have seen a paradigm shift in image reconstruction methods changing from analytic to iterative and now to machine learning based methods. These data-driven algorithms either learn to transfer raw sensory inputs directly to output images or serve as a post processing step for reducing image noise and removing artifacts. Generative Adversarial Networks (GANs), one of the recent breakthroughs in the field of deep learning, have gained attention in both academia and industry due to their versatility in inter-domain image translation and opened a wide range of possibilities in the field of computer vision.

The goal of this project is to assess the feasibility and build a proof-of-concept prototype of a GAN-based low-light video enhancement system that can be deployed on an edge device. 

## Background   

State-of-the-art image restoration and enhancement deep learning algorithms heavily rely on either synthesized or captured corrupted and clean image pairs to train the network. In practice, it is very difficult to simultaneously acquire both corrupted and ground truth images of the same visual scene (e.g., low-light and normal-light image pairs at the same time). Synthesizing corrupted images from clean images could sometimes help, but such synthesized results are usually not realistic enough, leading to various artifacts when the trained model is applied to real-world low-light images. Specifically for the low-light enhancement problem, there may be no unique or well-defined high-light ground truth given a low-light image. An alternative architecture that utilizes unpaired image datasets ([CycleGAN](https://arxiv.org/pdf/1703.10593.pdf)) usually takes a very long time to train and might not be suitable for generating high-fidelity details. 

To mitigate these shortcomings, we propose a solution based on the [EnlightenGAN Architecture](https://arxiv.org/abs/1906.06972) whose main features are summarized below:

* Lightweight one-path GAN for unsupervised image-to-image translation that learns a mapping between low light and normal light image spaces without relying on perfectly paired training images  

* Avoids overfitting any specific data generation protocol or imaging device, which leads to notably improved real-world generalization  

* Utilizes a global-local discriminator structure that handles spatially-varying light conditions in the input image  

* Employs self-regularization by using the feature preserving loss and attention mechanism that utilize information directly extracted from the input  

* Easily adaptable to enhancing low-light images from different domains  

* Consistently outperforms state-of-the-art algorithms across different qualitative and quantitative image quality metrics 


![architecture](/assets/arch.png)  
**EnlightenGAN Network Architecture Diagram**. EnlightenGAN adopts an attention-guided U-Net as the generator and uses the dual discriminator to direct the global and local information. A single, image-level discriminator often fails on spatially-varying light images; to enhance local regions adaptively in addition to improving the light globally, the network uses a novel global-local discriminator structure. A local discriminator randomly crops local patches from both output and real normal light images, and learns to distinguish whether they are real (from real images) or fake (from enhanced outputs). The Attention Map is generated by retrieving the illumination channel, *I*, from the input RGB image, normalizing it to [0,1], and then using *(1 − I)* (element-wise difference) as a self-regularized attention map.

The EnightenGAN's loss function consists of four parts:
<br/><br/>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp;<img src="https://render.githubusercontent.com/render/math?math=\Large \textit{Loss} =  L^{Global}_{SFP}%2BL^{Local}_{SFP}%2BL^{Global}_{G}%2BL^{Local}_{G}">

where:  

* <img src="https://render.githubusercontent.com/render/math?math=L^{Global}_{SFP}"> is a global self-feature preserving loss measured as a distance between a latent feature space of the input low-light and its enhanced normal-light output version (the feature vectors are extracted using VGG-16 model pretrained on ImageNet dataset)  
* <img src="https://render.githubusercontent.com/render/math?math=L^{Local}_{SFP}">  is equivalent to <img src="https://render.githubusercontent.com/render/math?math=L^{Global}_{SFP}"> but applied to randomly selected local patches cropped from input and output images  

* <img src="https://render.githubusercontent.com/render/math?math=L^{Global}_{G}"> is the global adversarial loss estimated as the probability that real image is more realistic than fake image  

* <img src="https://render.githubusercontent.com/render/math?math=L^{Local}_{G}"> is equivalent to <img src="https://render.githubusercontent.com/render/math?math=L^{Global}_{G}"> but applied to randomly selected local patches cropped from input and output images

The paper's authors have also released their [PyTorch model implemetation](https://github.com/TAMU-VITA/EnlightenGAN) which we used as a code base for our project repository.

## System Architecture  

A high-level system architecture diagram is shown on the figure below:

![](/assets/cloud_edge_diagram.png)  

First, the EnlightenGAN model is trained on IBM Cloud using 2 P100 GPU servers. {TODO: Casey and Sony, please review and update if necessary}  
The trained model is then downloaded to Jetson TX2 module where it is used to perform frame-by-frame inference on pre-recorded video files. 

The following two sections provide detailed description of the Cloud and Edge components.


### **Cloud: Model Training & Evaluation**  

{TODO: Casey and Sony, please add your description here}

System setup, training/test data, visdom screenshots, training and fine-tuning process, quality evaluation metrics description, quantitative/qualitative scores comparison table for different hyper-parameters. 

Describe any changes you made to the original model's code. To keep our main README high-level, you can provide links to other documents that deal with the setup and changes details (similar to what I've done for Jetson). 

Please check in your images into the [assets](assets) directory and embed them in this document using the following syntax: `![](/assets/image_filename)`.

Make sure to provide links to the training/test datasets. For example,  

Training data [[Google Drive]](https://drive.google.com/drive/folders/1fwqz8-RnTfxgIIkebFG2Ej3jQFsYECh0?usp=sharing) (unpaired images collected from multiple datasets)

Testing data [[Google Drive]](https://drive.google.com/open?id=1PrvL8jShZ7zj2IC3fVdDxBY1oJR72iDf) (including LIME, MEF, NPE, VV, DICP)  

Provide a link to the best trained model (shared Google dive?)

### **Edge: Deployment on Jetson TX2**  

In the original implementation of EnlightenGAN, the inference is performed by running predictions on a batch of images that are copied to a pre-defined folder. The inference results are stored on disk, similarly using a pre-configured location.  

To repurpose the EnlightenGAN model for video processing, we modified the DataLoader ([image_folder.py](/data/image_folder.py), [unaligned_dataset.py](/data/unaligned_dataset.py)) so that it can read frames directly from a video file (.mp4 or .avi) and feed them to the trained model for inference. We also changed the [predict.py](predict.py) script that now runs inference on each frame supplied by the DataLoader and displays both the input and ouput frames side-by-side as a video stream. The new script also supports saving of the two video streams as an .mp4 file.

The detailed description of how to build and run the docker container can be found in [README_Jetson.md](README_Jetson.md), while the animation below demonstrates a sample run of the IllumiGAN application on Jetson TX2:

![](/assets/screen-recording-outdoor.gif)

Here we can see the print-out of the network configuration within the container shell on the left and the data folder containing an input low-light video in the bottom right. The window in the top right corner shows the input and output video streams displayed side-by-side. 

We observed that without any optimization the best inference speed we could achieve on low-resolution videos (e.g., 352x288) was about 5 FPS. 

To improve the performance, we tried to convert our trained model to TensorRT using NVIDIA's open-source library [Torch2TRT](https://github.com/NVIDIA-AI-IOT/torch2trt), which provides a high-level Python interface for PyTorch-to-TensorRT conversion. Unfortunately, while we were able to convert some state-of-the-art models (i.e., ResNet) to TensorRT successfully, any attempts to apply the library to the EnlightenGAN model resulted in failure. Further details about our experimentation with `torch2trt` converter can be found [here](testtrt).

## Results & Discussion  

To test the performance of our model in terms of the output quality, we ran inference on a set of videos with various degrees of illumination. We used two sources of input data:  

* Royalty-free dataset of [City At Night Stock Video Footage](https://www.videezy.com/free-video/city-at-night)  

* Homemade recordings captured with iPhone 6 and converted to .avi format using a [free online converter](https://www.zamzar.com/convert/mp4-to-avi/). 

These recordings were uploaded to Jetson TX2 and used as an input to our IllimiGAN application.  

Below are a few samples demonstrating how the model performed under different lighting conditions.


### **Outdoor (artificial light)** 

This video is a representative example of an urban night scene, with both brightly-lit and dark areas present in the field of view. Such scenes are particularly challenging for traditional image enhancement algorithms since they struggle to brighten the dark areas without introducing distortions to the well-lit regions.  

Input &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp;Output  
![](/assets/taxi-768-inference.gif)  

As we can see from the side-by-side videos, the input video has a brightly-lit spot at its center while the rest of the objects and the people in front of the building fade into darkness. On the other hand, the video on the right shows as a uniformly-lit scene where colors and details are restored with high fidelity.

### **Outdoor (low-light)**  

The following two samples demonstrate the model's performance under very challenging conditions where the input scene is mostly dark with some occasional bright spots due to artificial lighting. 

The first sample is a nighttime scene in Chicago where the only sources of illumination are distant building lights and occasional blinking bike lights from passing bikers.   
While none of the objects or people can be easily detected in the input video (the bikers' presence is only evident by moving shadows), the output video clearly shows the buildings, the fence, the red-and-white construction pole, as well as the two groups of bikers moving in opposite directions. We can even discern the color of their clothes and the shapes of backpacks they're wearing.    

Input &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp;Output
![](/assets/chicago-768-inference.gif)  

<br/><br/>
The next video shows our teammate's backyard captured at nighttime. Similar to the Chicago scene above, the original video looks completely dark with some occasional bright spots due to ambient lighting.  
The output video, on the other hand, managed to faithfully recreate the flowers, grass and leaves, as well as the stone structures in the yard. The model sometimes struggles, however, with enhancement of extremely dark areas.  

Input &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp;Output  
![](/assets/outdoor-infer.gif)



### **Indoor (low-light)**  
The last example was intended as a stress test to see how well our model can handle the input with no evident external lighting.  
The original video was shot inside our teammate's house during nighttime. It even has a short story line: an intruder is lurking inside the house, looking for a valuable painting, and feeling comfortable since no security camera can detect her presence due to complete darkness. That turned out to be a false hope: our IllumiGAN application was not only able to detect the intruder's presence but was also able to recreate her clothes (and, as a bonus, has revealed the masterpiece painting itself hanging on the wall).  
While the recreated video is very noisy, from the daylight reference image we can infer that the model was able to recontruct the hallway layout along with the content of the painting.   

Input &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp;Output&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Daylight Reference  
![](/assets/mysterious-hallway-inference.gif)![](./assets/hallway_daylight.jpg)

## Conclusion & Future Developments  

{TODO: to be populated after the model training part is complete}  


## References  & Tools

* Yifan Jiang and Xinyu Gong and Ding Liu and Yu Cheng and Chen Fang and Xiaohui Shen and Jianchao Yang and Pan Zhou and Zhangyang Wang. **EnlightenGAN: Deep Light Enhancement without Paired Supervision.** 2019. https://arxiv.org/abs/1906.06972  

* **Video Converter**: https://www.onlineconverter.com

* **NVIDIA PyTorch to TensorRT Converter**: https://github.com/NVIDIA-AI-IOT/torch2trt
