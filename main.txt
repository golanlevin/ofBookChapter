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


### 2.1. Image acquisition and data structures 
- 8u, 32f, etc.
- 1C, 3C, 4C images, etc.
- Bayer images
- Back and forth between ofxCV, ofxOpenCV, unsigned char*, ofPixels.
- color space conversions; converting RGB to Gray.

#### 2.1.4 RGB, grayscale, and other color space conversions

Many computer vision algorithms (though not all!) are commonly performed on grayscale or monochome images. Converting color images to grayscale can significantly improve the speed of many image processing routines by reducing both the number of calculations as well as the amount of memory required to process the data. Except where stated otherwise, *all of the examples in this chapter assume that you're working with monochrome images*. Here's some simple code to convert a color image (e.g. captured from a webcam) into a grayscale version: 

`[Code to convert RGB to grayscale using openFrameworks] `

`[Code to convert RGB to grayscale using ofxCV]` 

`[Code to convert RGB to grayscale using OpenCV]` 

#### SIDEBAR
> *TIP: There are several ways to derive a grayscale image from a color one.*
> - *The fastest (but undeniably trashiest) method for converting a color image into a grayscale one — suitable only for imagery which is already predominantly black-and-white — is simply to extract a single color channel from an image, as a proxy for its luminance. For example, one might fetch only the green values as an approximation to an image's luminance, discarding its red and blue data. For a typical 8u color image whose data is interleaved R-G-B, this can be done by fetching every 3rd byte.* 
> - *A slower but more perceptually accurate method approximates luminance (often written `Y`) as a straight average of the red, green and blue values for every pixel:* `Y = (R+G+B)/3;`. *This not only produces a better representation of the image's luminance (across the visible color spectrum), but it also diminishes the influence of noise in any one color channel.*
> - *The most perceptually accurate methods for computing grayscale from color images employ a specially-weighted "colorimetric" average of the RGB color data. These methods are marginally more expensive to compute, as each color channel must be multiplied by its own weighting factor. The CCIR 601 imaging specification, which is used in the OpenCV [cvtColor](http://docs.opencv.org/modules/imgproc/doc/miscellaneous_transformations.html#cvtcolor) function, employs the formula* `Y = 0.299*R + 0.587*G + 0.114*B;` *(with the assumption that the RGB values have been gamma-corrected). The CIE 1931 luminance specification, on the other hand, assumes linear behavior in the signals and defines the luminance as:* `Y = 0.2126*R + 0.7152*G + 0.072*B;`. *According to [Wikipedia](http://en.wikipedia.org/wiki/Grayscale), "these coefficients represent the measured intensity perception of typical trichromat humans [...]; in particular, human vision is most sensitive to green and least sensitive to blue." For more information, see http://en.wikipedia.org/wiki/Grayscale , http://en.wikipedia.org/wiki/Luma_(video), and the OpenCV [cvtColor](http://docs.opencv.org/modules/imgproc/doc/miscellaneous_transformations.html#cvtcolor) documentation.*

Of course, there are times when 

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

