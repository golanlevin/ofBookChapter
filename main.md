1. Introduction 

[This outline is based on workshops Golan taught with Kyle.]
Some previous reference materials here:
-- http://futuretheater.net/wiki/Vision_Workshop
-- http://piratepad.net/resonate-cv 
-- http://www.flong.com/texts/essays/essay_cvad/

Unedited copy-paste from Arturo email (not yet integrated):
looks great to me, something i always find useful to explain as introduction in CV workshops before going into software details is the differences between contour finding (manual input sessions / delicate baoundaries) / motion detection (memo's gold / damian's wind) / blob tracking (chris o'shea's audience) and how contour finding *usually* gives more "precise" information than motion detection while it requires a much more controlled environment (ie: lighting) and how blob tracking techniques are usually some combination of contour finding + motion detection. all of it seems to be in the different subsections of your outline, i find it useful to explain it as an introduction with examples so people get a general idea but perhaps for the book is better to explain it along with the different techniques
also explaining the physical setup of such installations, IR lighting, filters... how to make the computer see some things and humans others and in general how all the work you can do in the physical setup will make your live easier when coding: the first time zach explained to me how in mesa di voce the screen is flooded with IR so the computer has to do almost nothing in order to detect the performers, it totally changed my way of thinking about this kind of problems

========================================================
## 2. Preliminaries


### 2.1. Digital image acquisition and data structures 

This chapter introduces techniques for manipulating (and extracting certain kinds of information from) *raster images*. Such images are also known as *bitmap images* or *pixmap images*, though, for the purposes of this chapter, we'll just use the generic term **image** to refer to any array (or *buffer*) of numbers that represent the color values of a rectangular grid of *pixels* ("picture elements"). 

#### 2.1.1. Pixels in Memory
We first review of pixels and how they are stored in computer memory. Below is a simple illustration of an 8-bit grayscale image buffer, a low-resolution portrait of Abraham Lincoln. Each pixel's brightness is represented by a number, whose range is from 0 (black) to 255 (white): 

![Pixel data diagram](https://dl.dropboxusercontent.com/u/10137599/ofbook/lincoln.png)

In point of fact, pixel values are almost universally stored, at the hardware level, in a *one-dimensional array*. For example, with respect to how it is stored in memory, the data from the image above probably appears something like this list of unsigned chars:

```Java
{157, 153, 174, 168, 150, 152, 129, 151, 172, 161, 155, 156, 155, 182, 163, 74, 75, 62, 33, 17, 110, 210, 180, 154, 180, 180, 50, 14, 34, 6, 10, 33, 48, 106, 159, 181, 206, 109, 5, 124, 131, 111, 120, 204, 166, 15, 56, 180, 194, 68, 137, 251, 237, 239, 239, 228, 227, 87, 71, 201, 172, 105, 207, 233, 233, 214, 220, 239, 228, 98, 74, 206, 188, 88, 179, 209, 185, 215, 211, 158, 139, 75, 20, 169, 189, 97, 165, 84, 10, 168, 134, 11, 31, 62, 22, 148, 199, 168, 191, 193, 158, 227, 178, 143, 182, 106, 36, 190, 205, 174, 155, 252, 236, 231, 149, 178, 228, 43, 95, 234, 190, 216, 116, 149, 236, 187, 86, 150, 79, 38, 218, 241, 190, 224, 147, 108, 227, 210, 127, 102, 36, 101, 255, 224, 190, 214, 173, 66, 103, 143, 96, 50, 2, 109, 249, 215, 187, 196, 235, 75, 1, 81, 47, 0, 6, 217, 255, 211, 183, 202, 237, 145, 0, 0, 12, 108, 200, 138, 243, 236, 195, 206, 123, 207, 177, 121, 123, 200, 175, 13, 96, 218};
```
This way of storing image data may run counter to your expectations, since the data certainly *appears* to be two-dimensional when it is displayed. Yet, this is the case, since computer memory consists simply of an ever-increasing linear list of address spaces. *(Observe how this data includes no details about the image's width and height. Should these gray values be interpreted as an image which is 12 pixels wide and 16 pixels tall, or 8x24, or 3x64? This must be specified elsewhere, as we shall see.)*

Dan Shiffman has made this helpful diagram to underscore how pixel data is stored, in his [tutorial on Images and Pixels](http://www.processing.org/tutorials/pixels/) at [Processing.org](http://processing.org):

![From Processing](http://www.processing.org/tutorials/pixels/imgs/pixelarray.jpg)

#### 2.1.2. Image data types and channels.
The Lincoln portrait above shows an 8-bit, 1-channel image. Each pixel uses a single round number (technically, a byte or unsigned char) to represent a single luminance value. But other data types and formats are possible. 

For example, it is common for color images to be represented by 8-bit, *3-channel* images. In this case, each pixel brings together 3 bytes' worth of information: one byte each for red, green and blue intensities. In computer memory, it is common for these values to be interleaved R-G-B. As you can see, color images necessarily contain three times as much data. 

![Not mine](https://dl.dropboxusercontent.com/u/10137599/ofbook/interleaved.jpg)

Take a very close look at your LCD screen, and you'll see how this way of storing the data is directly motivated by the layout of your display's phosphors:

![Not mine](https://dl.dropboxusercontent.com/u/10137599/ofbook/rgb-screen.jpg)

8-bit 1-channel and 8-bit 3-channel images are the most common image formats you'll find. Around the world of image processing algorithms, however, you'll sometimes encounter a variety of others, including:
- 16-bit (unsigned short) images, in which each channel uses two bytes to store each pixel's data, with a number that ranges from 0-65535;
- 32-bit (float) images, in which each channel's data is represented by floating point numbers. 

For example, the original Microsoft Kinect sensor produces a depth image whose values range from 0 to 1090 -- or approximately 11 bits of resolution. The `ofxKinect` addon uses a 16-bit image to store this information without losing precision. 

You'll also find
- 2-Channel images (commonly, for luminance plus Alpha/transparency)
- 3-Channel images (generally for RGB data, but occasionally used to store images in other color spaces, such as HSB or YUV). 
- 4-Channel images (commonly for RGBA images, but occasionally for CMYK)

It gets even more exotic. ["Hyperspectral" imagery from the Landsat 8 satellite](https://www.mapbox.com/blog/putting-landsat-8-bands-to-work/), for example, has 11 channels, including bands for ultraviolet, near infrared, and thermal (deep) infrared.

#### 2.1.2. Where images come from
In openFrameworks, raster images can come from a wide variety of sources, including (but not limited to): 
- a real-time image stream from a video camera (using `ofVideoGrabber`), 
- a JPEG, PNG, TIFF or other commonly-formatted image file, loaded and decompressed from your hard drive (using `ofImage::loadImage()`) or from the Internet (using `ofLoadURL()`);
- a synthetic rendering, perhaps obtained from an `ofFBO` or stored in an `ofPixels` object; 
- a buffer of pixels captured from the screen, captured with `ofImage::grabScreen()`;
- perhaps more exotically, a *depth image*, in which pixel values represent distances instead of colors. Depth images can be captured from real-world scenes with a Microsoft Kinect using `ofxKinect`, or extracted from synthetic CGI scenes using (for example) `ofFBO::getDepthTexture()`.

![We don't have the rights to this image, we need something similar](https://dl.dropboxusercontent.com/u/10137599/ofbook/kinect_depth_image.png) *An example of a depth image (left) and a corresponding RGB color image (right), captured with a Kinect. In the depth image, the lightness of a pixel represents its distance to the camera.* 

In openFrameworks, such images can be stored in a variety of different containers, which allow their data to be used (captured, displayed, manuipulated, stored) in different ways and contexts. These containers include: 

- **unsigned char*: ** 
- **ofPixels** The 
- **ofImage** The ofImage is the most common object for loading, saving and displaying images. Loading a file into the ofImage allocates an (internal) ofPixels object to store the image data.
- **ofxCvImage**
- **IplImage**
- **Mat**


To the greatest extent possible, the designers of openFrameworks have made it simple to exchange data between these containers.

`[Code to load and display an image]`

`[Extract obtain its unsigned char array]`

`[Code to load an image, and obtain its unsigned char array]`

- padding, byte boundaries, step
- Bayer images
- Back and forth between ofxCV, ofxOpenCV, unsigned char*, ofPixels.



#### 2.1.4 RGB, grayscale, and other color space conversions

Many computer vision algorithms (though not all!) are commonly performed on grayscale or monochome images. Converting color images to grayscale can significantly improve the speed of many image processing routines by reducing both the number of calculations as well as the amount of memory required to process the data. Except where stated otherwise, *all of the examples in this chapter assume that you're working with monochrome images*. Here's some simple code to convert a color image (e.g. captured from a webcam) into a grayscale version: 

`[Code to fetch an RGB image from a real-time camera stream]`

`[Code to convert RGB to grayscale using openFrameworks] `

`[Code to convert RGB to grayscale using ofxCV]` 

`[Code to convert RGB to grayscale using OpenCV]` 

Of course, there are times when 

#### SIDEBAR
> *TIP: There are several ways to derive a grayscale image from a color one.*
> - *The fastest (but undeniably trashiest) method for converting a color image into a grayscale one — suitable only for imagery which is already predominantly black-and-white — is simply to extract a single color channel from an image, as a proxy for its luminance. For example, one might fetch only the green values as an approximation to an image's luminance, discarding its red and blue data. For a typical 8u color image whose data is interleaved R-G-B, this can be done by fetching every 3rd byte.* 
> - *A slower but more perceptually accurate method approximates luminance (often written `Y`) as a straight average of the red, green and blue values for every pixel:* `Y = (R+G+B)/3;`. *This not only produces a better representation of the image's luminance (across the visible color spectrum), but it also diminishes the influence of noise in any one color channel.*
> - *The most perceptually accurate methods for computing grayscale from color images employ a specially-weighted "colorimetric" average of the RGB color data. These methods are marginally more expensive to compute, as each color channel must be multiplied by its own weighting factor. The CCIR 601 imaging specification, which is used in the OpenCV [cvtColor](http://docs.opencv.org/modules/imgproc/doc/miscellaneous_transformations.html#cvtcolor) function, employs the formula* `Y = 0.299*R + 0.587*G + 0.114*B;` *(with the assumption that the RGB values have been gamma-corrected). The CIE 1931 luminance specification, on the other hand, assumes linear behavior in the signals and defines the luminance as:* `Y = 0.2126*R + 0.7152*G + 0.072*B;`. *According to [Wikipedia](http://en.wikipedia.org/wiki/Grayscale), "these coefficients represent the measured intensity perception of typical trichromat humans [...]; in particular, human vision is most sensitive to green and least sensitive to blue." For more information, see http://en.wikipedia.org/wiki/Grayscale , http://en.wikipedia.org/wiki/Luma_(video), and the OpenCV [cvtColor](http://docs.opencv.org/modules/imgproc/doc/miscellaneous_transformations.html#cvtcolor) documentation.*

### 2.2. Image arithmetic: mathematical operations on images

A core part of the workflow of computer vision is *image arithmetic*. These are the basic mathematical operations we all know -- addition, subtraction, multiplication, and division -- but interpreted in the image domain. Here are two very simple examples:

`[Code to add a constant value to an image]` 

`[Code to multiply an image by a constant]` 

TIP: Watch out for blowing out the range of your data types. 

When  assume that these operations are performed *pixelwise* -- meaning, for every pixel in an image. When 



In the examples presented here, for the sake of simplicity, we'll assume that the images upon which we'll perform these operations are all the same size -- for example, 640x480 pixels, a typical capture size for many webcams. We'll also assume that these images are monochrome or grayscale. 

-- 
- adding two images together
- subtracting two images
- multiplying an image by a constant
- mentioning ROI 
- Example: creating an average of several images (e.g. Jason Salavon)
- Example: creating a running average
- Example: creating a circular alpha-mask from a computed Blinn spot
   
### 2.3. Filtering and Noise Removal Convolution Filtering
- Blurring an image
- Edge detection 
- Median filtering
- Advanced sidebar: dealing with boundary conditions

### 2.4. Suggestions for Further Experimentation

I sometimes assign my students the project of copying a well-known work of interactive new-media art. Reimplementing projects such as the ones below can be highly instructive, and test the limits of your attention to detail. As Gerald King [writes](http://www.geraldking.com/Copying.htm), such copying "provides insights which cannot be learned from any other source." *I recommend you build...*

#### 2.4.1. A Slit-Scanner. 
*Slit-scanning* — a type of "time-space imaging" — has been a common trope in interactive video art for more than twenty years. Interactive slit-scanners have been developed by some of the most revered pioneers of new media art (Toshio Iwai, Paul de Marinis, Steina Vasulka) as well as by [literally dozens](http://www.flong.com/texts/lists/slit_scan/) of other highly regarded practitioners. The premise remains an open-ended format for seemingly limitless experimentation, whose possibilities have yet to be exhausted. It is also a good exercise in managing image data, particularly in extracting and copying pixel ROIs. In digital slit-scanning, thin slices are extracted from a sequence of video frames, and concatenated into a new image. The result is an image which succinctly reveals the history of movements in a video or camera stream. 

![Daniel Rozin, Time Scan Mirror (2004)](http://www.flong.com/storage/images/texts/slit_scan/rozin_timescan.jpg)

#### 2.4.2. *[Text Rain](http://camilleutterback.com/projects/text-rain/)* by Camille Utterback and Romy Achituv (1999).<br />
*Text Rain* is a now-classic work of interactive art in which virtual letters appear to "fall" on the visitor's "silhouette". Utterback writes: "In the Text Rain installation, participants stand or move in front of a large projection screen. On the screen they see a mirrored video projection of themselves in black and white, combined with a color animation of falling letters. Like rain or snow, the letters appears to land on participants’ heads and arms. The letters respond to the participants’ motions and can be caught, lifted, and then let fall again. The falling text will 'land' on anything darker than a certain threshold, and 'fall' whenever that obstacle is removed."

![Camille Utterback and Romy Achituv, Text Rain (1999)](http://golancourses.net/2013/wp-content/uploads/2012/12/text-rain.jpg)

- Toy version of *L.A.S.E.R. Tag* by the GRL: Finding the brightest point, tracing a path.
- Toy version of Memo Akten's *Gold*: Optical flow + particle system

========================================================
3. Scenario I. Basic Blobs (e.g. Manual Input Sessions) 

3.1. The Why 
- Some examples of projects that use blob-tracking
- and some scenarios that call for it. 
   
### 3.2. Detecting and Locating Presence and Motion 

#### 3.2.1. Detecting presence with Background subtraction
sfdflkj
#### 3.2.2. Detecting motion with frame-differencing
sfdflkj
#### 3.2.3. Binarization, blob detection and contour extraction
sfdflkj
- Area thresholds for contour extraction (min plausible area, max plausible area, as % of capture size)
- Finding negative vs. positive contours
   
### 3.3. Image Processing Refinements
#### 3.3.1. Using a running average of background
#### 3.3.2. Erosion, dilation, median to remove noise after binarization
#### 3.3.3. Combining presence and motion in a weighted average
#### 3.3.4. Compensating for perspectival distortion and lens distortion
   
### 3.4. Thresholding Refinements
   -- Some techniques for automatic threshold detection 
   -- Dynamic thresholding (per-pixel thresholding)
   
3.5. The Vector space: Extracting information from Blob Contours
   -- Area, Perimeter, Centroids, Bounding box
   -- Calculcating blob orientation (central axis)
   -- Locating corners in contours, estimating local curvature
   -- 1D Filtering of contours to eliminate noise, i.e local averaging. 
   -- Convexity defects, contourFinder.getConvexityDefects()
   -- Other shape metrics; shape recognition
   
3.6. Using Kinect depth images
   -- Finding the "fore-point" (foremost point)
   -- Background subtraction with depth images
   -- Hole-filling in depth images
   -- Computing normals from depth gradients
   
3.7. Suggestions for further experimentation: 
   -- Tracking multiple blobs with ofxCv.tracker
   -- Box2D polygons using OpenCV contours, e.g. https://vimeo.com/9951522
   
========================================================   
4. Scenario II. Face Tracking.

4.1. Overview
Some examples of projects that use face-tracking
- Golan & Zach's Re:Face
- Mary Huang's face typography
- *[Face Substitution](https://vimeo.com/29348533)* by Kyle McDonald & Arturo Castro (2011). The classic 
- *[Google Faces](http://www.onformative.com/lab/googlefaces/)* by Onformative (2012). This project, which identifies face-like features in Google Earth satellite imagery, explores what Greg Borenstein has called *[machine pareidolia](http://urbanhonking.com/ideasfordozens/2012/01/14/machine-pareidolia-hello-little-fella-meets-facetracker/)* -- the possibility that computer algorithms might "hallucinate" faces in everyday images. 
   
### 4.2. A basic face detector. 
Let's get to it. In this section we'll implement face detection using the classic Viola-Jones detector that comes with OpenCV. 
- Face detection with classic OpenCV viola-Jones detector
- How it works, and considerations when using it. 
- cvDazzle; 
 
#### SIDEBAR
> *Orientation-dependence in the OpenCV face detector: Bug or Feature?*
> - Kyle & Aram ("How to Avoid Facial Recognition"
   
4.3. Advanced face analysis with the Saraghi FaceTracker

### 4.4. Suggestions for Further Experimentation
Now that you can locate faces in images and video, consider using the following exercises as starting-points for further exploration: 

- Make a face-controlled puppet
- Mine an image database for faces
- Make a kinetic sculpture that points toward a visitor's face.

   
========================================================  
5. A Computer-Vision lexicon, and where to find out more information 

Computer vision is a huge field and we can't possibly cover all useful examples here. 
Sometimes people lack the terminology to know what to google for. 

-- Camera calibration. 
-- Homography transforms and re-projection. 

