
  
##STILL WORKING ON IT!!!

1. Introduction 
  
 [This outline is based on workshops Golan taught with Kyle.]
 Some previous reference materials here:
- http://futuretheater.net/wiki/Vision_Workshop
- http://piratepad.net/resonate-cv 
- http://www.flong.com/texts/essays/essay_cvad/
  
Unedited copy-paste from Arturo email (not yet integrated):
looks great to me, something i always find useful to explain as introduction in CV workshops before going into software details is the differences between contour finding (manual input sessions / delicate baoundaries) / motion detection (memo's gold / damian's wind) / blob tracking (chris o'shea's audience) and how contour finding *usually* gives more "precise" information than motion detection while it requires a much more controlled environment (ie: lighting) and how blob tracking techniques are usually some combination of contour finding + motion detection. all of it seems to be in the different subsections of your outline, i find it useful to explain it as an introduction with examples so people get a general idea but perhaps for the book is better to explain it along with the different techniques
also explaining the physical setup of such installations, IR lighting, filters... how to make the computer see some things and humans others and in general how all the work you can do in the physical setup will make your live easier when coding: the first time zach explained to me how in mesa di voce the screen is flooded with IR so the computer has to do almost nothing in order to detect the performers, it totally changed my way of thinking about this kind of problems

  
========================================================
## 1. Preliminaries to Image Processing
  
### 1.1. Digital image acquisition and data structures 

  
This chapter introduces techniques for manipulating (and extracting certain kinds of information from) *raster images*. Such images are also known as *bitmap images* or *pixmap images*, although, for the purposes of this chapter, we'll just use the generic term **image** to refer to any array (or *buffer*) of numbers that represent the color values of a rectangular grid of *pixels* ("picture elements"). In openFrameworks, such buffers come in a variety of flavors, and are used within and managed by a wide variety of convenient container objects, as we shall see.
  
#### 1.1.1. Loading and Displaying an Image 
  
Image processing begins with, well, *an image*. Happily, loading and displaying an image is very straightforward in OF. Let's start with this tiny, low-resolution (12x16 pixel) grayscale portrait of Abraham Lincoln: ![Small Lincoln image](https://dl.dropboxusercontent.com/u/10137599/ofbook/lincoln.png)

Below is a simple application for loading and displaying an image, similar to the *imageLoaderExample* in the OF examples collection. The header file for our program, *testApp.h*, declares an instance of an `ofImage` object, *myImage*:
  
```cpp
// This is testApp.h
#pragma once
#include "ofMain.h"

class testApp : public ofBaseApp{
	public:
		void setup();
		void draw();
		ofImage myImage;
};
```
And here's our complete *testApp.cpp* file. The Lincoln image is loaded from our hard drive (once) in the `setup()` function, and then we display it (many times per second) in our `draw()` function. As you can see from the filepath provided to the `loadImage()` function, the program assumes that the image *lincoln.png* can be found in a directory called "images" alongside your executable: 
 
 ```cpp
 #include "testApp.h"
 
 void testApp::setup(){
 	myImage.loadImage("images/lincoln.png");
 	myImage.setImageType(OF_IMAGE_GRAYSCALE);
 }
 
 void testApp::draw(){	
 	ofBackground(255);
	ofSetColor(255);
 
 	int imgW = myImage.width;
 	int imgH = myImage.height;
 	myImage.draw(10, 10, imgW*10, imgH*10);
 }
 ```
Compiling and running the above program displays the following canvas, in which this tiny image is rendered, scaled up by a factor of 10. *(Note that the image appears "blurry" because, by default, openFrameworks uses linear interpolation to scale displayed images.)*
 
 ![Pixel data diagram](https://dl.dropboxusercontent.com/u/10137599/ofbook/lincoln-displayed.jpg)
 
 If you're new to working with images in OF, it's worth pointing out that you should avoid loading images in the `draw()` or `update()` functions, if possible. Reading data from disk is one of the slowest things you can ask a computer to do, and in many circumstances, you can load all the images you'll need when your program is first initialized, in `setup()`. If you're loading an image in your `draw()` loop — the same image, again and again, 60 times per second — you're hurting the performance of your app, and risking damage to your hard disk.  
 
 #### 1.1.2. Where (Else) Images Come From
 In OF, raster images can come from a wide variety of sources, including (but not limited to): 
 a commonly-used image file (in a format like JPEG, PNG, TIFF, or GIF), loaded and decompressed from your hard drive into an `ofImage`; 
 a real-time image stream from a webcam or other video camera (using an `ofVideoGrabber`); 
 a sequence of frames loaded from a digital video file (using an `ofVideoPlayer`); 
 a buffer of pixels grabbed from the screen, captured with `ofImage::grabScreen()`;
 a synthetic rendering, perhaps obtained from an `ofFBO` or stored in an `ofPixels` or `ofTexture` object; 
 a real-time video from a more specialized variety of camera, such as a 1394b Firewire camera (via `ofxLibdc`), a networked Ethernet camera (via `ofxIpCamera`), a Canon DSLR (using `ofxCanonEOS`), or with the help of a variety of other community-contributed addons like `ofxQTKitVideoGrabber`, `ofxRPiCameraVideoGrabber`, etc.
 perhaps more exotically, a *depth image*, in which pixel values represent *distances* instead of colors. Depth images can be captured from real-world scenes with special cameras (such as a Microsoft Kinect via the `ofxKinect` addon), or extracted from synthetic CGI scenes using (for example) `ofFBO::getDepthTexture()`.
 
 ![We don't have the rights to this image, we need something similar](https://dl.dropboxusercontent.com/u/10137599/ofbook/kinect_depth_image.png) *An example of a depth image (left) and a corresponding RGB color image (right), captured with a Kinect. In the depth image, the lightness of a pixel represents its distance to the camera.* 

Incidentally, OF makes it easy to **load images from the Internet**, by using a URL as the filename argument, as in `myImage.loadImage("http://blah.com/img.jpg");`. Keep in mind that doing this will load the remote image *synchronously*, meaning your program will "block" (or freeze) while it waits for all of the data to download. For an improved user experience, you can also load Internet images *asynchronously* (in a background thread), using the response provided by `ofLoadURLAsync()`; a  sample implementation of this can be found in the openFrameworks *imageLoaderWebExample* example. Now that you can load images stored on the Internet, you can fetch images computationally using fun APIs (like Temboo, Instagram or Flickr), or from dynamic online sources like live traffic cameras. 

#### 1.1.3. Acquiring and Displaying a Webcam Image

The procedure for **acquiring a video stream** from a live webcam or digital movie file is no more difficult than loading an `ofImage`. The main conceptual difference is that the image data contained within an `ofVideoGrabber` or `ofVideoPlayer` is continually refreshed, usually about 30 times per second. 

The following program (which you can find elaborated in the *videoGrabberExample*) shows the basic procedure. In this example, for some added fun, we also fetch the buffer of data that contain the `ofVideoGrabber`'s pixels, then "invert" this data and display it with an `ofTexture`.

The header file declares an `ofVideoGrabber`, which will be used to acquire video data from our computer's default webcam. We also declare a buffer of unsigned chars to store the inverted video frame, and the `ofTexture` which we'll use to display it:

```cpp
#pragma once
#include "ofMain.h"

class testApp : public ofBaseApp{
	public:
		
		void setup();
		void update();
		void draw();
		
		ofVideoGrabber  	myVideoGrabber;
		ofTexture           myTexture;
	
		unsigned char*      invertedVideoData;
		int 				camWidth;
		int 				camHeight;
};

```
Below is the complete code of our webcam-grabbing .cpp file. 

As you might expect, the `ofVideoGrabber` object provides many more options and settings, not shown here. These allow you to do things like listing and selecting from available camera devices; choosing capture dimensions and frame rate; and (depending on your hardware and drivers) adjusting parameters like camera exposure and contrast.

```cpp
#include "testApp.h"

void testApp::setup(){
	
	// Set capture dimensons of 320x240, a common video size.
	camWidth  = 320;
	camHeight = 240;
	
	// Open an ofVideoGrabber for the default camera
	myVideoGrabber.initGrabber (camWidth,camHeight);
	
	// Create resources to store and display another copy of the data
	invertedVideoData = new unsigned char [camWidth*camHeight*3];
	myTexture.allocate (camWidth,camHeight, GL_RGB);
}

void testApp::update(){
	
	// Ask the grabber to refresh its data.
	myVideoGrabber.update();
	
	// If the grabber indeed has fresh data,
	if (myVideoGrabber.isFrameNew()){
		
		// Obtain a pointer to the grabber's image data.
		unsigned char* pixelData = myVideoGrabber.getPixels();
		
		// For every byte of the RGB image data,
		int nTotalBytes = camWidth*camHeight*3;
		for (int i=0; i<nTotalBytes; i++){
			
			// Subtract it from 255, to make a "photo negative"
			invertedVideoData[i] = 255 - pixelData[i];
		}
		
		// Now stash the inverted data in an ofTexture
		myTexture.loadData (invertedVideoData, camWidth,camHeight, GL_RGB);
	}
}

void testApp::draw(){
	ofBackground(100,100,100);
	ofSetColor(255,255,255);
	
	// Draw the grabber, and next to it, the "negative" ofTexture.
	myVideoGrabber.draw(10,10);
	myTexture.draw(340, 10);
}
```
Here's the result, using my laptop's webcam:

![Video grabber screenshot](https://dl.dropboxusercontent.com/u/10137599/ofbook/videograbber.png)

Acquiring frames from a Quicktime movie or other digital video file stored on disk is an almost identical procedure; see the OF *videoPlayerExample* implementation or `ofVideoGrabber` documentation for details. 

A common pattern among computer vision practitioners is to switch between a pre-stored "sample" video of your scene, and a live camera grabber. That way, you can refine your processing algorithms from the comfort of your own room. A hacky if effective example of this pattern can be found in the *opencvExample* in the addons example directory, where the switch is built using a `#define`:

```cpp
    //...
	#ifdef _USE_LIVE_VIDEO
        myVideoGrabber.initGrabber(320,240);
	#else
        myVideoPlayer.loadMovie("pedestrians.mov");
        myVideoPlayer.play();
	#endif
	//...
```

#### 2.1.1. Pixels in Memory
To begin our study of image processing and computer vision, we'll need to do more than just load and display images; we'll need to *access, manipulate and analyze the numeric data represented by their pixels*. It's therefore worth reviewing how pixels are stored in computer memory. Below is a simple illustration of the grayscale image buffer which stores our image of Abraham Lincoln. Each pixel's brightness is represented by a single 8-bit number, whose range is from 0 (black) to 255 (white): 

![Pixel data diagram](https://dl.dropboxusercontent.com/u/10137599/ofbook/lincoln_pixel_values.png)

In point of fact, pixel values are almost universally stored, at the hardware level, in a *one-dimensional array*. For example, the data from the image above is stored in a manner similar to this long list of unsigned chars:

```cpp
{157, 153, 174, 168, 150, 152, 129, 151, 172, 161, 155, 156, 155, 182, 163, 74, 75, 62, 33, 17, 110, 210, 180, 154, 180, 180, 50, 14, 34, 6, 10, 33, 48, 106, 159, 181, 206, 109, 5, 124, 131, 111, 120, 204, 166, 15, 56, 180, 194, 68, 137, 251, 237, 239, 239, 228, 227, 87, 71, 201, 172, 105, 207, 233, 233, 214, 220, 239, 228, 98, 74, 206, 188, 88, 179, 209, 185, 215, 211, 158, 139, 75, 20, 169, 189, 97, 165, 84, 10, 168, 134, 11, 31, 62, 22, 148, 199, 168, 191, 193, 158, 227, 178, 143, 182, 106, 36, 190, 205, 174, 155, 252, 236, 231, 149, 178, 228, 43, 95, 234, 190, 216, 116, 149, 236, 187, 86, 150, 79, 38, 218, 241, 190, 224, 147, 108, 227, 210, 127, 102, 36, 101, 255, 224, 190, 214, 173, 66, 103, 143, 96, 50, 2, 109, 249, 215, 187, 196, 235, 75, 1, 81, 47, 0, 6, 217, 255, 211, 183, 202, 237, 145, 0, 0, 12, 108, 200, 138, 243, 236, 195, 206, 123, 207, 177, 121, 123, 200, 175, 13, 96, 218};
```
This way of storing image data may run counter to your expectations, since the data certainly *appears* to be two-dimensional when it is displayed. Yet, this is the case, since computer memory consists simply of an ever-increasing linear list of address spaces. *(Note how this data includes no details about the image's width and height. Should this list of values be interpreted as a grayscale image which is 12 pixels wide and 16 pixels tall, or 8x24, or 3x64? Could it be interpreted as a color image? Such 'meta-data' is specified elsewhere — generally in a container object like an `ofImage`.)*

#### 2.1.2. Grayscale Pixels and Array Indices
Dan Shiffman has made this helpful diagram to illustrate how pixel data is stored, in his highly recommended [tutorial on Images and Pixels](http://www.processing.org/tutorials/pixels/) at [Processing.org](http://processing.org):

![From Processing](http://www.processing.org/tutorials/pixels/imgs/pixelarray.jpg)

Note how the (one-dimensional) list of values have been distributed to successive (two-dimensional) pixel locations in the image — wrapping over the right edge just like English text. 

There are two ways to access the individual 

It frequently happens that you'll need to find the array-index of a given pixel *(x,y)* in an image. This little task comes up often enough that it's worth committing the following pattern to memory: 

```cpp
// Given: 
// unsigned char *buffer, an array storing a one-channel image
// int x, the horizontal coordinate (column) of your query pixel
// int y, the vertical coordinate (row) of your query pixel
// int imgWidth, the width of the image

int arrayIndex = y*imgW + x;

// Now you can get values at location (x,y), e.g.:
unsigned char pixelValueAtXY = buffer[arrayIndex]; 
// And you can also set values at that location, e.g.:
buffer[arrayIndex] = pixelValueAtXY;
```
Reciprocally, you can also fetch the x and y locations of a pixel corresponding to a given array index: 

```cpp
// Given: 
// A one-channel (e.g. grayscale) image
// int arrayIndex, an index in that image's array of pixels
// int imgW, the width of the image

int y = arrayIndex / imgW; // NOTE, this is integer division!
int x = arrayIndex % imgW; 
```

#### 2.1.3. Image Data Types and Channels.
The Lincoln portrait above shows an 8-bit, 1-channel image. Each pixel uses a single round number (technically, a byte or unsigned char) to represent a single luminance value. But other data types and formats are possible. 

For example, it is common for color images to be represented by 8-bit, *3-channel* images. In this case, each pixel brings together 3 bytes' worth of information: one byte each for red, green and blue intensities. In computer memory, it is common for these values to be interleaved R-G-B. As you can see, color images necessarily contain three times as much data. 

![Not mine](https://dl.dropboxusercontent.com/u/10137599/ofbook/interleaved.jpg)

Take a very close look at your LCD screen, and you'll see how this way of storing the data is directly motivated by the layout of your display's phosphors:

![Not mine](https://dl.dropboxusercontent.com/u/10137599/ofbook/rgb-screen.jpg)

Accessing pixel values in buffers containing RGB data is a little more complex. Here's how you can retrieve the values representing the individual red, green and blue components of pixel at a given (x,y) location: 

```cpp
// Given: 
// unsigned char *buffer, an array storing an RGB image
// (assuming interleaved data in RGB order!)
// int x, the horizontal coordinate (column) of your query pixel
// int y, the vertical coordinate (row) of your query pixel
// int imgWidth, the width of the image

int rArrayIndex = (y*imgWidth*3) + (x*3);
int gArrayIndex = (y*imgWidth*3) + (x*3) + 1;
int bArrayIndex = (y*imgWidth*3) + (x*3) + 2;

// Now you can get and set values at location (x,y), e.g.:
unsigned char redValueAtXY   = buffer[rArrayIndex]; 
unsigned char greenValueAtXY = buffer[gArrayIndex]; 
unsigned char blueValueAtXY  = buffer[bArrayIndex]; 
```

8-bit 1-channel and 8-bit 3-channel images are the most common image formats you'll find. Around the world of image processing algorithms, however, you'll sometimes encounter a variety of others, including:
- 8-bit *palettized* images, in which each pixel stores an index into an array of (up to ) 256 possible colors;
- 16-bit (unsigned short) images, in which each channel uses two bytes to store each pixel's data, with a number that ranges from 0-65535;
- 32-bit (float) images, in which each channel's data is represented by floating point numbers. 

For example, the original Microsoft Kinect sensor produces a depth image whose values range from 0 to 1090. That's approximately 11 bits of resolution. The `ofxKinect` addon uses a 16-bit image to store this information without losing precision. 

You'll also find
- 2-Channel images (commonly, for luminance plus Alpha/transparency)
- 3-Channel images (generally for RGB data, but occasionally used to store images in other color spaces, such as HSB or YUV). 
- 4-Channel images (commonly for RGBA images, but occasionally for CMYK)

It gets even more exotic. ["Hyperspectral" imagery from the Landsat 8 satellite](https://www.mapbox.com/blog/putting-landsat-8-bands-to-work/), for example, has 11 channels, including bands for ultraviolet, near infrared, and thermal (deep) infrared.



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



#### 2.1.5 RGB, grayscale, and other color space conversions

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
   - Some techniques for automatic threshold detection 
   - Dynamic thresholding (per-pixel thresholding)
   
3.5. The Vector space: Extracting information from Blob Contours
   - Area, Perimeter, Centroids, Bounding box
   - Calculcating blob orientation (central axis)
   - Locating corners in contours, estimating local curvature
   - 1D Filtering of contours to eliminate noise, i.e local averaging. 
   - Convexity defects, contourFinder.getConvexityDefects()
   - Other shape metrics; shape recognition
   
3.6. Using Kinect depth images
   - Finding the "fore-point" (foremost point)
   - Background subtraction with depth images
   - Hole-filling in depth images
   - Computing normals from depth gradients
   
3.7. Suggestions for further experimentation: 
   - Tracking multiple blobs with ofxCv.tracker
   - Box2D polygons using OpenCV contours, e.g. https://vimeo.com/9951522
   
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
## A Computer-Vision lexicon, and where to find out more information 

Computer vision is a huge field and we can't possibly cover all useful examples here. 
Sometimes people lack the terminology to know what to google for. 

-- Camera calibration. 
-- Homography transforms and re-projection. 

