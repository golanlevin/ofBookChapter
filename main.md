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
2. Preliminaries

2.1. Image data structures 
   -- 8u, 32f, etc.
   -- 1C, 3C, 4C images, etc.
   -- Back and forth between ofxCV, ofxOpenCV, unsigned char*, ofPixels.
   -- Example: color space conversions; converting RGB to Gray.

2.2. Image Arithmetic: mathematical operations on images
   -- adding two images together
   -- subtracting two images
   -- multiplying an image by a constant
   -- mentioning ROI 
   -- Example: creating an average of several images (e.g. Jason Salavon)
   -- Example: creating a circular alpha-mask from a computed Blinn spot
   
2.3. Convolution Filtering
   -- Blurring an image
   -- Edge detection 
   -- Advanced sidebar: dealing with boundary conditions

2.4. Suggestions for Further Experimentation:
   -- Slit-Scanner
   -- Toy version of Text Rain. Fun with edge detection.
   -- Toy version of LASER Tag. Finding the brightest point, tracing a path.
   -- Toy version of Memo Akten's Gold: Optical flow + particle system

========================================================
3. Scenario I. Basic Blobs (e.g. Manual Input Sessions) 

3.1. The Why 
   -- Some examples of projects that use blob-tracking
   -- and some scenarios that call for it. 
   
3.2. Detecting and Locating Presence and Motion 
   -- Detecting presence with Background subtraction
   -- Detecting motion with frame-differencing
   -- Binarization, blob detection and contour extraction
      -- Area thresholds for contour extraction (min plausible area, max plausible area, as % of capture size)
      -- Finding negative vs. positive contours
   
3.3. Image Processing Refinements
   -- Using a running average of background
   -- Erosion, dilation, median to remove noise after binarization
   -- Combining presence and motion in a weighted average
   -- Compensating for perspectival distortion and lens distortion
   
3.4. Thresholding Refinements
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
   -- Some examples of projects that use face-tracking
      -- Golan & Zach's Re:Face
      -- Mary Huang's face typography
      -- Kyle & Arturo, face swapping
   
4.2. A basic face detector. 
   -- Face detection with classic OpenCV viola-Jones detector
   -- How it works, and considerations when using it. 
   -- cvDazzle; Kyle & Aram ("How to Avoid Facial Recognition"
   
4.3. Advanced face analysis with the Saraghi FaceTracker

4.4. Suggestions for further experimentation: 
   -- Make a face-controlled puppet
   -- Toy version of David Tinapple's "The Face of TV": Find the average face of a TV channel
   
========================================================  
5. A Computer-Vision lexicon, and where to find out more information 

Computer vision is a huge field and we can't possibly cover all useful examples here. 
Sometimes people lack the terminology to know what to google for. 

-- Camera calibration. 
-- Homography transforms and re-projection. 

