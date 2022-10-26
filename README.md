# Image Registration and Warping in OpenCV-Python

  <img src="images/image-registration.jpg" width="1000"/>
  
  ## 1. Objective

The objective of this project is to implement and demonstrate image registration using OpenCV Python.

## 2. Image Registration

Image registration is a digital image processing technique which aligns and overlays two or more images of the same scene, which are acquired from various imaging equipment or sensors taken at different times and angles. 

The need for image registration is encountered in various applications, including:

* Stitching various scenes together to form a continuous panoramic shot.
* Aligning camera images of documents to a standard alignment to create realistic scanned documents.
* Aligning medical images for better observation and analysis.


In many cases, image registration involves aligning a slave image, S, to a reference or master image, M, through simple coordinates transform process, according to the following steps: 

* Convert both S and M images to grayscale.
* Generate features from both images, S and M
* Match features between the slave (S) and master (M) images
* Select the best matches, and filter-out the noisy and potentially erroneous matches.
* Compute the homomorphy matrix between the slave (S) and the master (M) image based on the selected good matches
* Apply this transform to the slave image to align it with the master image.

We shall implement and illustrate this process, step by step in section 4. 

## 3. Data

The master and slave images are illustrated in the next figure, respectively:
* Note the significant difference in the geometry, perspective, and scale between two images.
* Our objective is to register and align the slave image (S: second image) relative to the master image (M: first image). 

 <img src="images/Master+Slave--Images.png" width="1000"/>
 
## 4. Development

* Project: Image Registration and Warping:

  * The objective of this project is to implement and demonstrate image registration using OpenCV Python.
  * In the simplest case, image registration involves aligning a slave image, S, to a reference or master image, M, through simple coordinates transform process, according to the following steps:
    * Convert both S and M images to grayscale.
    * Generate features from both images, S and M
    * Match features between the slave (S) and master (M) images
    * Select the best matches, and filter-out the noisy and potentially erroneous matches.
    * Compute the homomorphy matrix between the slave (S) and the master (M) image based on the selected good matches
    * Apply this transform to the slave image to align it with the master image.
    * We shall implement and illustrate each of these steps below.
    
* Author: Mohsen Ghazel (mghazel)
* Date: April 9th, 2021

### 4.1. Step 1: Imports and global variables:

<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># Python imports and environment setup</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># opencv</span>
<span style="color:#200080; font-weight:bold; ">import</span> cv2
<span style="color:#595979; "># numpy</span>
<span style="color:#200080; font-weight:bold; ">import</span> numpy <span style="color:#200080; font-weight:bold; ">as</span> np
<span style="color:#595979; "># matplotlib</span>
<span style="color:#200080; font-weight:bold; ">import</span> matplotlib<span style="color:#308080; ">.</span>pyplot <span style="color:#200080; font-weight:bold; ">as</span> plt
<span style="color:#200080; font-weight:bold; ">import</span> matplotlib<span style="color:#308080; ">.</span>image <span style="color:#200080; font-weight:bold; ">as</span> mpimg

<span style="color:#595979; "># input/output OS</span>
<span style="color:#200080; font-weight:bold; ">import</span> os 

<span style="color:#595979; "># date-time to show date and time</span>
<span style="color:#200080; font-weight:bold; ">import</span> datetime

<span style="color:#595979; "># to display the figures in the notebook</span>
<span style="color:#44aadd; ">%</span>matplotlib inline

<span style="color:#595979; ">#------------------------------------------</span>
<span style="color:#595979; "># Test imports and display package versions</span>
<span style="color:#595979; ">#------------------------------------------</span>
<span style="color:#595979; "># Testing the OpenCV version</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">"OpenCV : "</span><span style="color:#308080; ">,</span>cv2<span style="color:#308080; ">.</span>__version__<span style="color:#308080; ">)</span>
<span style="color:#595979; "># Testing the numpy version</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Numpy : "</span><span style="color:#308080; ">,</span>np<span style="color:#308080; ">.</span>__version__<span style="color:#308080; ">)</span>

OpenCV <span style="color:#308080; ">:</span>  <span style="color:#008000; ">4.5</span><span style="color:#308080; ">.</span><span style="color:#008c00; ">1</span>
Numpy <span style="color:#308080; ">:</span>  <span style="color:#008000; ">1.19</span><span style="color:#308080; ">.</span><span style="color:#008c00; ">2</span>
</pre>



### 4.2. Step 2: Input data:

* Read and visualize the master and slave images


<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#----------------------------------------------------</span>
<span style="color:#595979; "># 2.1) Read the slave image:</span>
<span style="color:#595979; ">#----------------------------------------------------</span>
<span style="color:#595979; "># the slave file name</span>
slave_img_file_path <span style="color:#308080; ">=</span> <span style="color:#1060b6; ">"../data/test-images/slave.jpg"</span>
<span style="color:#595979; "># check if the slave image file exists</span>
<span style="color:#200080; font-weight:bold; ">if</span><span style="color:#308080; ">(</span>os<span style="color:#308080; ">.</span>path<span style="color:#308080; ">.</span>exists<span style="color:#308080; ">(</span>slave_img_file_path<span style="color:#308080; ">)</span> <span style="color:#44aadd; ">==</span> <span style="color:#008c00; ">0</span><span style="color:#308080; ">)</span><span style="color:#308080; ">:</span>
    <span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'Slave image file name DOES NOT EXIST! = '</span> <span style="color:#44aadd; ">+</span> slave_img_file_path<span style="color:#308080; ">)</span>
<span style="color:#595979; "># Read the slave image </span>
slave_img <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>imread<span style="color:#308080; ">(</span>slave_img_file_path<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>IMREAD_COLOR<span style="color:#308080; ">)</span>

<span style="color:#595979; ">#----------------------------------------------------</span>
<span style="color:#595979; "># 2.2) Read the reference/master image:</span>
<span style="color:#595979; ">#----------------------------------------------------</span>
<span style="color:#595979; "># the master file name</span>
master_img_file_path <span style="color:#308080; ">=</span> <span style="color:#1060b6; ">"../data/test-images/master.jpg"</span>
<span style="color:#595979; "># check if the master image file exists</span>
<span style="color:#200080; font-weight:bold; ">if</span><span style="color:#308080; ">(</span>os<span style="color:#308080; ">.</span>path<span style="color:#308080; ">.</span>exists<span style="color:#308080; ">(</span>master_img_file_path<span style="color:#308080; ">)</span> <span style="color:#44aadd; ">==</span> <span style="color:#008c00; ">0</span><span style="color:#308080; ">)</span><span style="color:#308080; ">:</span>
    <span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'Master image file name DOES NOT EXIST! = '</span> <span style="color:#44aadd; ">+</span> master_img_file_path<span style="color:#308080; ">)</span>
<span style="color:#595979; "># Read the master image </span>
master_img <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>imread<span style="color:#308080; ">(</span>master_img_file_path<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>IMREAD_COLOR<span style="color:#308080; ">)</span>

<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 2.3) Visualize the slave and master images</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># create a figure</span>
plt<span style="color:#308080; ">.</span>figure<span style="color:#308080; ">(</span>figsize<span style="color:#308080; ">=</span><span style="color:#308080; ">(</span><span style="color:#008c00; ">16</span><span style="color:#308080; ">,</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; "># visualize the master image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">121</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Master image"</span><span style="color:#308080; ">,</span> fontsize<span style="color:#308080; ">=</span><span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>master_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; "># visualize the slave image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">122</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Slave image"</span><span style="color:#308080; ">,</span> fontsize<span style="color:#308080; ">=</span><span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>slave_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">;</span>
     
</pre>


 <img src="images/test-images.PNG" width="1000"/>
 
### 4.3. Step 3: Detect ORB features from the master and slave images:
* We need to generate features from each image so we can match them and estimate the Homography matrix

<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.1) Convert each image to grayscale if needed</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># Convert slave image to grayscale, if needed:</span>
<span style="color:#200080; font-weight:bold; ">if</span> <span style="color:#308080; ">(</span> <span style="color:#400000; ">len</span><span style="color:#308080; ">(</span>slave_img<span style="color:#308080; ">.</span>shape<span style="color:#308080; ">)</span> <span style="color:#44aadd; ">&gt;</span> <span style="color:#008c00; ">2</span> <span style="color:#308080; ">)</span><span style="color:#308080; ">:</span>
    img1_gray <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>slave_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2GRAY<span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">else</span><span style="color:#308080; ">:</span> 
    img1_gray <span style="color:#308080; ">=</span> slave_img<span style="color:#308080; ">.</span>copy<span style="color:#308080; ">(</span><span style="color:#308080; ">)</span>
<span style="color:#595979; "># Convert master image to grayscale, if needed</span>
<span style="color:#200080; font-weight:bold; ">if</span> <span style="color:#308080; ">(</span> <span style="color:#400000; ">len</span><span style="color:#308080; ">(</span>master_img<span style="color:#308080; ">.</span>shape<span style="color:#308080; ">)</span> <span style="color:#44aadd; ">&gt;</span> <span style="color:#008c00; ">2</span> <span style="color:#308080; ">)</span><span style="color:#308080; ">:</span>
    img2_gray <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>master_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2GRAY<span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">else</span><span style="color:#308080; ">:</span> 
    img2_gray <span style="color:#308080; ">=</span> master_img<span style="color:#308080; ">.</span>copy<span style="color:#308080; ">(</span><span style="color:#308080; ">)</span>

<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.2) Get the dimensions of the master/reference </span>
<span style="color:#595979; ">#------------------------------------------------------</span>
height_master<span style="color:#308080; ">,</span> width_master <span style="color:#308080; ">=</span> img2_gray<span style="color:#308080; ">.</span>shape

<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.3) Make a copy of each image</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># slave image</span>
img1 <span style="color:#308080; ">=</span> slave_img<span style="color:#308080; ">.</span>copy<span style="color:#308080; ">(</span><span style="color:#308080; ">)</span>

<span style="color:#595979; "># master image</span>
img2 <span style="color:#308080; ">=</span> master_img<span style="color:#308080; ">.</span>copy<span style="color:#308080; ">(</span><span style="color:#308080; ">)</span>
    
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.4) Generate ORB features from the 2 images</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># Create ORB detector with 5000 features.</span>
orb_detector <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>ORB_create<span style="color:#308080; ">(</span><span style="color:#008c00; ">5000</span><span style="color:#308080; ">)</span>
  
<span style="color:#595979; "># Find keypoints and descriptors.</span>
<span style="color:#595979; "># slave image</span>
kp1<span style="color:#308080; ">,</span> d1 <span style="color:#308080; ">=</span> orb_detector<span style="color:#308080; ">.</span>detectAndCompute<span style="color:#308080; ">(</span>img1_gray<span style="color:#308080; ">,</span> <span style="color:#074726; ">None</span><span style="color:#308080; ">)</span>
<span style="color:#595979; "># master image</span>
kp2<span style="color:#308080; ">,</span> d2 <span style="color:#308080; ">=</span> orb_detector<span style="color:#308080; ">.</span>detectAndCompute<span style="color:#308080; ">(</span>img2_gray<span style="color:#308080; ">,</span> <span style="color:#074726; ">None</span><span style="color:#308080; ">)</span>

<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.5) Draw the detected ORB features on the images</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># create a figure</span>
plt<span style="color:#308080; ">.</span>figure<span style="color:#308080; ">(</span>figsize<span style="color:#308080; ">=</span><span style="color:#308080; ">(</span><span style="color:#008c00; ">16</span><span style="color:#308080; ">,</span> <span style="color:#008c00; ">10</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.5.1) Slave image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># draw only keypoints location, not size and orientation on master image</span>
cv2<span style="color:#308080; ">.</span>drawKeypoints<span style="color:#308080; ">(</span>img1<span style="color:#308080; ">,</span> kp1<span style="color:#308080; ">,</span> img1<span style="color:#308080; ">,</span> flags<span style="color:#308080; ">=</span>cv2<span style="color:#308080; ">.</span>DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS<span style="color:#308080; ">)</span>
<span style="color:#595979; "># visualize the detected features for the slave image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">121</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Slave image: ORB Features"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>img1<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.5.2) Master image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># draw only keypoints location, not size and orientation on slave image</span>
cv2<span style="color:#308080; ">.</span>drawKeypoints<span style="color:#308080; ">(</span>img2<span style="color:#308080; ">,</span> kp2<span style="color:#308080; ">,</span> img2<span style="color:#308080; ">,</span> flags<span style="color:#308080; ">=</span>cv2<span style="color:#308080; ">.</span>DRAW_MATCHES_FLAGS_DRAW_RICH_KEYPOINTS<span style="color:#308080; ">)</span>
<span style="color:#595979; "># visualize the detected features for the master image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">122</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Master image: ORB Features"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>img2<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">;</span>
</pre>

 <img src="images/test-images-detected-features.PNG" width="1000"/>
 
### 4.4. Step 4: Match the features between the 2 images using Brute-Force Feature Matching:

* Brute-Force matcher is simple:
  * It takes the descriptor of each query-image feature in first and matches it with all other features in scene-image using some distance calculation.
  * The closest one is returned.
  * This process is repeated for all the features
  * In the end we pick the K features, based on the distances separating the query feature and its matched reference-image feature.
 
 <pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#----------------------------------------------------------------</span>
<span style="color:#595979; "># 4.1) Match the scene and query images features using the </span>
<span style="color:#595979; ">#      Brute-Force Matcher.</span>
<span style="color:#595979; ">#-----------------------------------------------------------------</span>
<span style="color:#595979; "># create BFMatcher object</span>
bf <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>BFMatcher<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>NORM_HAMMING<span style="color:#308080; ">,</span> crossCheck<span style="color:#308080; ">=</span><span style="color:#074726; ">True</span><span style="color:#308080; ">)</span>

<span style="color:#595979; "># Match descriptors:</span>
<span style="color:#595979; "># d1: master image</span>
<span style="color:#595979; "># d2: slave image</span>
matches <span style="color:#308080; ">=</span> bf<span style="color:#308080; ">.</span>match<span style="color:#308080; ">(</span>d1<span style="color:#308080; ">,</span>d2<span style="color:#308080; ">)</span>

<span style="color:#595979; "># Sort them in the order of their distance.</span>
matches <span style="color:#308080; ">=</span> <span style="color:#400000; ">sorted</span><span style="color:#308080; ">(</span>matches<span style="color:#308080; ">,</span> key <span style="color:#308080; ">=</span> <span style="color:#200080; font-weight:bold; ">lambda</span> x<span style="color:#308080; ">:</span>x<span style="color:#308080; ">.</span>distance<span style="color:#308080; ">)</span>

<span style="color:#595979; ">#-----------------------------------------------------------------</span>
<span style="color:#595979; "># 4.2) Visualize the matches</span>
<span style="color:#595979; ">#-----------------------------------------------------------------</span>
<span style="color:#595979; "># visualization preferences parameters</span>
draw_params <span style="color:#308080; ">=</span> <span style="color:#400000; ">dict</span><span style="color:#308080; ">(</span>matchColor <span style="color:#308080; ">=</span> <span style="color:#308080; ">(</span><span style="color:#008c00; ">255</span><span style="color:#308080; ">,</span><span style="color:#008c00; ">0</span><span style="color:#308080; ">,</span><span style="color:#008c00; ">0</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span>  <span style="color:#595979; "># matching-lines color</span>
                   singlePointColor <span style="color:#308080; ">=</span> <span style="color:#308080; ">(</span><span style="color:#008c00; ">0</span><span style="color:#308080; ">,</span><span style="color:#008c00; ">255</span><span style="color:#308080; ">,</span><span style="color:#008c00; ">0</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> <span style="color:#595979; ">#  keypoints color</span>
                   flags <span style="color:#308080; ">=</span> <span style="color:#008c00; ">4</span> <span style="color:#308080; ">)</span> <span style="color:#595979; "># show kepoints and matching lines</span>
<span style="color:#595979; "># Draw first 15 matches.</span>
img3 <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>drawMatches<span style="color:#308080; ">(</span>img1<span style="color:#308080; ">,</span>kp1<span style="color:#308080; ">,</span>img2<span style="color:#308080; ">,</span>kp2<span style="color:#308080; ">,</span>matches<span style="color:#308080; ">[</span><span style="color:#308080; ">:</span><span style="color:#008c00; ">25</span><span style="color:#308080; ">]</span><span style="color:#308080; ">,</span><span style="color:#074726; ">None</span><span style="color:#308080; ">,</span><span style="color:#44aadd; ">**</span>draw_params<span style="color:#308080; ">)</span>
<span style="color:#595979; "># create the figure</span>
plt<span style="color:#308080; ">.</span>figure<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"BFMatcher - Best Match"</span><span style="color:#308080; ">,</span>figsize<span style="color:#308080; ">=</span><span style="color:#308080; ">(</span><span style="color:#008c00; ">16</span><span style="color:#308080; ">,</span><span style="color:#008c00; ">10</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">111</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Brute-Force Matcher: The 25 matches"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>img3<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">;</span>
</pre>

 <img src="images/test-images-detected-matched-features.PNG" width="1000"/>
 
### 4.5. Step 5: Use the good matches to compute the Homography Matrix:

* The good ORB matches are used to compute the Homography matrix between the 2 images.


<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 5.1) Sort matches on the basis of their Hamming </span>
<span style="color:#595979; ">#      distance.</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
matches<span style="color:#308080; ">.</span>sort<span style="color:#308080; ">(</span>key <span style="color:#308080; ">=</span> <span style="color:#200080; font-weight:bold; ">lambda</span> x<span style="color:#308080; ">:</span> x<span style="color:#308080; ">.</span>distance<span style="color:#308080; ">)</span>
 
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 5.2) Good matches: Take the top 90 % matches forward.</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># Take the top 90 % matches forward</span>
matches <span style="color:#308080; ">=</span> matches<span style="color:#308080; ">[</span><span style="color:#308080; ">:</span><span style="color:#400000; ">int</span><span style="color:#308080; ">(</span><span style="color:#400000; ">len</span><span style="color:#308080; ">(</span>matches<span style="color:#308080; ">)</span><span style="color:#44aadd; ">*</span><span style="color:#008c00; ">90</span><span style="color:#308080; ">)</span><span style="color:#308080; ">]</span>
<span style="color:#595979; "># the number of good matches</span>
no_of_matches <span style="color:#308080; ">=</span> <span style="color:#400000; ">len</span><span style="color:#308080; ">(</span>matches<span style="color:#308080; ">)</span>
  
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 5.3) Compute the Homography Matrix:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># Define empty matrices of shape no_of_matches * 2.</span>
p1 <span style="color:#308080; ">=</span> np<span style="color:#308080; ">.</span>zeros<span style="color:#308080; ">(</span><span style="color:#308080; ">(</span>no_of_matches<span style="color:#308080; ">,</span> <span style="color:#008c00; ">2</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
p2 <span style="color:#308080; ">=</span> np<span style="color:#308080; ">.</span>zeros<span style="color:#308080; ">(</span><span style="color:#308080; ">(</span>no_of_matches<span style="color:#308080; ">,</span> <span style="color:#008c00; ">2</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
  
<span style="color:#595979; "># populate the good matches </span>
<span style="color:#200080; font-weight:bold; ">for</span> i <span style="color:#200080; font-weight:bold; ">in</span> <span style="color:#400000; ">range</span><span style="color:#308080; ">(</span><span style="color:#400000; ">len</span><span style="color:#308080; ">(</span>matches<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">:</span>
  p1<span style="color:#308080; ">[</span>i<span style="color:#308080; ">,</span> <span style="color:#308080; ">:</span><span style="color:#308080; ">]</span> <span style="color:#308080; ">=</span> kp1<span style="color:#308080; ">[</span>matches<span style="color:#308080; ">[</span>i<span style="color:#308080; ">]</span><span style="color:#308080; ">.</span>queryIdx<span style="color:#308080; ">]</span><span style="color:#308080; ">.</span>pt
  p2<span style="color:#308080; ">[</span>i<span style="color:#308080; ">,</span> <span style="color:#308080; ">:</span><span style="color:#308080; ">]</span> <span style="color:#308080; ">=</span> kp2<span style="color:#308080; ">[</span>matches<span style="color:#308080; ">[</span>i<span style="color:#308080; ">]</span><span style="color:#308080; ">.</span>trainIdx<span style="color:#308080; ">]</span><span style="color:#308080; ">.</span>pt
  
<span style="color:#595979; "># compute the homography matrix</span>
homography<span style="color:#308080; ">,</span> mask <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>findHomography<span style="color:#308080; ">(</span>p1<span style="color:#308080; ">,</span> p2<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>RANSAC<span style="color:#308080; ">)</span>
<span style="color:#595979; "># print out the estimated Homography matrix</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'------------------------------------------'</span><span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'The estimated Homography Matrix = '</span><span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'------------------------------------------'</span><span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span>homography<span style="color:#308080; ">)</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'------------------------------------------'</span><span style="color:#308080; ">)</span>

<span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span>
The estimated Homography Matrix <span style="color:#308080; ">=</span> 
<span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span>
<span style="color:#308080; ">[</span><span style="color:#308080; ">[</span><span style="color:#44aadd; ">-</span><span style="color:#008000; ">1.12107014e+00</span>  <span style="color:#008000; ">1.18535677e+00</span>  <span style="color:#008000; ">1.06303284e+03</span><span style="color:#308080; ">]</span>
 <span style="color:#308080; ">[</span><span style="color:#44aadd; ">-</span><span style="color:#008000; ">9.49907409e-01</span> <span style="color:#44aadd; ">-</span><span style="color:#008000; ">1.17521447e+00</span>  <span style="color:#008000; ">5.88590119e+03</span><span style="color:#308080; ">]</span>
 <span style="color:#308080; ">[</span> <span style="color:#008000; ">8.24894021e-05</span>  <span style="color:#008000; ">7.28293036e-05</span>  <span style="color:#008000; ">1.00000000e+00</span><span style="color:#308080; ">]</span><span style="color:#308080; ">]</span>
<span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span><span style="color:#44aadd; ">-</span>
</pre>


### 4.6. Step 6: Use Homography matrix to transform the slave image wrt the master image:

* Transform the slave image using the computer Homography matrix.


<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 6.1) Use this matrix to transform the</span>
<span style="color:#595979; ">#      colored image wrt the reference image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
transformed_img <span style="color:#308080; ">=</span> cv2<span style="color:#308080; ">.</span>warpPerspective<span style="color:#308080; ">(</span>slave_img<span style="color:#308080; ">,</span>
                    homography<span style="color:#308080; ">,</span> <span style="color:#308080; ">(</span>width_master<span style="color:#308080; ">,</span> height_master<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
  
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 6.2) Display the final results</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># create a figure</span>
plt<span style="color:#308080; ">.</span>figure<span style="color:#308080; ">(</span>figsize<span style="color:#308080; ">=</span><span style="color:#308080; ">(</span><span style="color:#008c00; ">20</span><span style="color:#308080; ">,</span> <span style="color:#008c00; ">10</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.3.1) Master image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># visualize the slave image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">131</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Slave image"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>slave_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.3.2) master image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># visualize the master image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">132</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Master image"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>master_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">;</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># 3.3.2) The transformed slave image:</span>
<span style="color:#595979; ">#------------------------------------------------------</span>
<span style="color:#595979; "># visualize the transformed slave image</span>
plt<span style="color:#308080; ">.</span>subplot<span style="color:#308080; ">(</span><span style="color:#008c00; ">133</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>title<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"Transformed slave image"</span><span style="color:#308080; ">,</span> fontsize <span style="color:#308080; ">=</span> <span style="color:#008c00; ">12</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>xticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span><span style="color:#308080; ">,</span> plt<span style="color:#308080; ">.</span>yticks<span style="color:#308080; ">(</span><span style="color:#308080; ">[</span><span style="color:#308080; ">]</span><span style="color:#308080; ">)</span>
plt<span style="color:#308080; ">.</span>imshow<span style="color:#308080; ">(</span>cv2<span style="color:#308080; ">.</span>cvtColor<span style="color:#308080; ">(</span>transformed_img<span style="color:#308080; ">,</span> cv2<span style="color:#308080; ">.</span>COLOR_BGR2RGB<span style="color:#308080; ">)</span><span style="color:#308080; ">)</span><span style="color:#308080; ">;</span>
</pre>

 <img src="images/test-images - final-registration-results.PNG" width="1000"/>
 
 ### 4.7. Step 7: Display a successful execution message:

<pre style="color:#000020;background:#e6ffff;font-size:10px;line-height:1.5;"><span style="color:#595979; "># display a final message</span>
<span style="color:#595979; "># current time</span>
now <span style="color:#308080; ">=</span> datetime<span style="color:#308080; ">.</span>datetime<span style="color:#308080; ">.</span>now<span style="color:#308080; ">(</span><span style="color:#308080; ">)</span>
<span style="color:#595979; "># display a message</span>
<span style="color:#200080; font-weight:bold; ">print</span><span style="color:#308080; ">(</span><span style="color:#1060b6; ">'Program executed successfully on: '</span><span style="color:#44aadd; ">+</span> <span style="color:#400000; ">str</span><span style="color:#308080; ">(</span>now<span style="color:#308080; ">.</span>strftime<span style="color:#308080; ">(</span><span style="color:#1060b6; ">"%Y-%m-%d %H:%M:%S"</span><span style="color:#308080; ">)</span> <span style="color:#44aadd; ">+</span> <span style="color:#1060b6; ">"...Goodbye!</span><span style="color:#0f69ff; ">\n</span><span style="color:#1060b6; ">"</span><span style="color:#308080; ">)</span><span style="color:#308080; ">)</span>

Program executed successfully on<span style="color:#308080; ">:</span> <span style="color:#008c00; ">2021</span><span style="color:#44aadd; ">-</span><span style="color:#008c00; ">04</span><span style="color:#44aadd; ">-</span><span style="color:#008c00; ">14</span> <span style="color:#008c00; ">10</span><span style="color:#308080; ">:</span><span style="color:#008c00; ">32</span><span style="color:#308080; ">:</span><span style="color:#008000; ">54.</span><span style="color:#308080; ">.</span><span style="color:#308080; ">.</span>Goodbye!
</pre>

## 5. Analysis

* In view of the presented results, we make the following observations:

  * The implemented simple registration algorithm yielded properly registered slave image with respect to the master image, in spite of the significant difference in the geometry, perspective, and scale of the two images.
  * The implemented algorithm works well because these are non-deformable objects
  * More advanced registration algorithms are generally needed for more complex and multiple images of the scene scene, acquired at different times, perspective, using different sensors, their order is unknown.

## 6. Future Work

* We plan to investigate the following related issues:

  * Test the developed registration scheme using more complex images, such as medical or satellite images
  * Identify the advantages and limitations of the implemented approach
  * Explore more advanced image registration algorithms.


## 7. References

1. Satya Mallick. Feature Based Image Alignment using OpenCV (C++/Python). https://learnopencv.com/tag/image-registration/ 
2. Adrian Rosebrock. Image alignment and registration with OpenCV. https://www.pyimagesearch.com/2020/08/31/image-alignment-and-registration-with-opencv/ 
3. MATLAB. Register Images with Projection Distortion Using Control Points. https://www.mathworks.com/help/images/registering-an-aerial-photo-to-an-orthophoto.html 
 4. J. Tohka. Image Registration. https://www.sciencedirect.com/topics/neuroscience/image-registration 
 5. Emna Kamoun. Image Registration: From SIFT to Deep Learning. https://medium.com/sicara/image-registration-sift-deep-learning-3c794d794b7a 
 6. Guna RakulanImage. Registration Using Homography in OpenCV. https://morioh.com/p/efb8a47942d5 
 7. GeeksforGeeks. Image Registration using OpenCV | Python. https://www.geeksforgeeks.org/image-registration-using-opencv-python/
 
 
 

 
 
