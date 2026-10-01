# CV Quick Revision — Core Notes till Page 108

<aside>
🎯

These are the **core revision notes for everything covered up to Sir's page 108**. They deliberately leave out the supplementary sections we decided not to study. The wording is meant to bring the idea back quickly, not just give isolated keywords.

</aside>

## 1. Digital image basics: sampling and quantization

A digital image is a finite grid of pixels. In a grayscale image, each pixel stores one intensity value; in a colour image, a pixel can store several channel values. The pixel coordinates tell us **where** we are in the image, while the stored value tells us the measured brightness or colour at that location.

The easiest way to separate **sampling** and **quantization** is to ask two different questions. Sampling decides **where we measure the image**, so it makes the spatial coordinates discrete. Quantization decides **which intensity values we are allowed to store**, so it makes brightness values discrete. More samples improve spatial resolution; more quantization levels improve intensity resolution. One does not automatically increase the other.

For a $b$-bit grayscale image,

$$
L=2^b
$$

possible gray levels are available, numbered from $0$ to $L-1$. Therefore an 8-bit image has 256 possible values: 0 through 255.

For simple storage questions, use

$$
\text{storage in bits}=\text{width}\times\text{height}\times\text{bits per pixel}.
$$

For example, a $512\times512$ 8-bit grayscale image uses $512\times512\times8$ bits, which is 256 KiB if headers and compression are ignored.

---

## 2. Image formation and scattering

A useful image-formation model separates **illumination** from **reflectance**:

$$
f(x,y)=i(x,y)r(x,y).
$$

The recorded intensity is therefore not determined only by the object. A pixel can look dark because the object reflects little light, because the illumination is weak, or because both effects occur together.

For underwater or hazy imaging, keep the three light paths distinct. **Direct light** travels from the object to the camera and carries useful scene detail. **Forward or front scatter** happens when object light is deflected by particles on its way to the camera; this spreads the object's light and mainly causes blur or loss of fine detail. **Backscatter** is different: illumination is scattered by particles toward the camera without first carrying the object's appearance, so it adds a veil and reduces contrast.

A compact way to remember the distinction is:

```
Object → particle → camera        = forward/front scatter → blur
Illumination → particle → camera  = backscatter → veil / contrast loss
```

Sir also shows the simplified model

$$
I_c(x,y)=J_c(x,y)t(x,y)+[1-t(x,y)]b_c.
$$

Here $J_c$ is the clear scene, $t$ is transmission and $b_c$ is surrounding light. When transmission is high, the camera receives more of the actual scene. When transmission is low, the surrounding-light contribution becomes more dominant.

---

## 3. What digital image processing is trying to do

Digital image processing manipulates image data so that it becomes more useful for viewing, analysis, storage or transmission. A useful distinction from computer vision is that **image processing changes the image**, while **computer vision tries to understand what the image contains**. Removing noise is image processing; deciding that the image contains a car or an abnormality is a computer-vision task.

Sir's major stages are not a compulsory pipeline that every system must follow in exactly the same order. They are better understood by their roles. **Acquisition** captures and digitizes the scene. **Enhancement** makes useful information easier to see or use. **Restoration** tries to estimate the original image from a degraded observation. **Morphological processing** works with structures and shapes. **Segmentation** separates meaningful regions or objects. **Representation and description** turn those regions into useful features, and **recognition** assigns an identity or class. Compression and colour processing support the overall system where needed.

---

## 4. Enhancement: where the processing happens

Image enhancement can be classified by **where** the operation is performed.

In the **spatial domain**, we work directly on pixel values. Point processing, histogram processing and neighbourhood filtering all belong here. In the **frequency domain**, the image is first transformed into a frequency representation, processed there, and then transformed back.

The important thing is not to mix this classification with the goal of the operation. **Spatial/frequency tells us where we process; enhancement/restoration tells us why we process.**

### Point processing

In point processing, each output pixel depends only on the corresponding input pixel:

$$
s=T(r).
$$

This is why the same input intensity gets mapped to the same output intensity wherever it appears in the image.

### Negative transformation

For gray levels from $0$ to $L-1$,

$$
s=(L-1)-r.
$$

This reverses brightness order. In an 8-bit image, an input value of 50 becomes $255-50=205$.

### Log and inverse-log behaviour

A log transform has the form

$$
s=c\log(1+r).
$$

It expands lower intensities relatively more than high intensities, so darker details can become easier to see while a large dynamic range is compressed. The inverse-log curve has the opposite general behaviour.

### Power-law / gamma transformation

The power-law form is

$$
s=cr^\gamma.
$$

For the normalized curve, $\gamma<1$ raises intermediate intensities and usually brightens the image, while $\gamma>1$ lowers them and usually darkens the image. $\gamma=1$ gives the identity case. As a shape mnemonic only, $\gamma<1$ resembles the effect of a log curve and $\gamma>1$ resembles the opposite direction; they are not the same formula.

### Contrast stretching

Contrast stretching spreads a useful intensity range so differences become easier to distinguish. Sir's emphasis is on the graph and the control points rather than on memorising a long piecewise formula. If the input and output control points coincide, the mapping becomes the identity. If the transition becomes extremely steep, the behaviour approaches thresholding.

### Gray-level slicing

Gray-level slicing highlights a chosen range of intensities. One version suppresses the background and shows mainly the selected range; the other highlights the selected range while keeping the background information.

### Bit-plane slicing

An 8-bit pixel can be separated into eight one-bit planes. Plane 0 is the least significant bit and plane 7 is the most significant bit. Higher-order planes usually contain most of the visible structure, while lower-order planes contain finer intensity detail. The highest bit plane is equivalent to separating values below and above 128.

---

## 5. Histograms and histogram equalisation

A grayscale histogram tells us how often each gray level occurs. If the counts are divided by the total number of pixels, we get a **normalized histogram**, whose values behave like probabilities and sum to 1.

The shape gives a quick idea of image appearance. A histogram concentrated toward lower intensities usually corresponds to a darker image; one concentrated toward higher intensities usually corresponds to a brighter image. A narrow concentration suggests that much of the image occupies only a small intensity range and may therefore have low contrast.

Histogram equalisation remaps intensities using the cumulative distribution:

$$
CDF(r_k)=\sum_{j=0}^{k}p(r_j),
$$

followed by a discrete mapping such as

$$
s_k=\operatorname{round}\left[(L-1)CDF(r_k)\right].
$$

Do not treat the CDF as a mysterious extra formula. It simply accumulates the probability from the darkest level up to the current level, and the equalisation mapping uses that accumulated value to redistribute intensities over the available output range.

For a numerical, the procedure is more important than memorising the final matrix: **count each gray level → normalize the counts → build the cumulative distribution → convert each input level to its new output level → replace the pixels**.

One important point from Sir's example is that a discrete equalized histogram does **not** have to become perfectly flat. With a finite number of pixels and discrete gray levels, several input levels may still end up sharing output levels.

---

## 6. Spatial filtering and convolution

Neighbourhood filtering differs from point processing because the output pixel now depends on nearby pixels as well. A small matrix called a **kernel** or **mask** is moved across the image.

For the convolution convention used by Sir, the kernel is first flipped horizontally and vertically, which is the same as rotating it by $180^\circ$. Then place it over the neighbourhood, multiply overlapping entries, add the products, and slide to the next position. Padding may be used at the boundary when the output size needs to remain unchanged.

The important procedure is therefore:

```
Flip kernel by 180° → place it → multiply corresponding entries → sum → slide
```

The flip does **not** mean “change the signs.” The values are simply moved to their rotated positions. Symmetric kernels such as the mean, Gaussian and Laplacian masks look unchanged after this rotation.

### Smoothing masks

The $3\times3$ mean filter is

$$
\frac{1}{9}
\begin{bmatrix}
1&1&1\\
1&1&1\\
1&1&1
\end{bmatrix}.
$$

It replaces a pixel by an average of its neighbourhood, reducing local fluctuations but also tending to blur edges.

The Gaussian mask shown by Sir is

$$
\frac{1}{16}
\begin{bmatrix}
1&2&1\\
2&4&2\\
1&2&1
\end{bmatrix}.
$$

It also smooths the image, but gives more importance to pixels near the centre than to the corners.

### Prewitt and Sobel

Prewitt and Sobel are directional edge operators. One orientation responds to left-right intensity changes and therefore helps reveal vertical boundaries; the other responds to top-bottom changes and helps reveal horizontal boundaries.

Prewitt uses equal weights along the relevant rows or columns, while Sobel gives weight 2 to the centre row or column. The purpose is more important than the derivative theory: both are masks for detecting directional intensity change.

### Laplacian and sharpening

Sir's four-neighbour Laplacian mask is

$$
\begin{bmatrix}
0&1&0\\
1&-4&1\\
0&1&0
\end{bmatrix}.
$$

A perfectly constant neighbourhood produces zero response, while rapid local intensity changes produce a stronger response. The related sharpening mask shown in the notes is

$$
\begin{bmatrix}
0&-1&0\\
-1&5&-1\\
0&-1&0
\end{bmatrix}.
$$

For quick recall: **mean/Gaussian smooth; Prewitt/Sobel detect directional edges; Laplacian responds to rapid local changes and is used in sharpening-related processing.**

---

## 7. Restoration and degradation

Image enhancement and image restoration can both improve an image, but they are not trying to do the same thing. **Enhancement** asks whether the result is more useful or visually suitable. **Restoration** asks whether we can recover an estimate of the original image from a degraded observation, usually by assuming that the degradation process is known or can be estimated.

Sir's degradation model is

$$
g(x,y)=h(x,y)*f(x,y)+\eta(x,y),
$$

where $f$ is the original image, $h$ is the degradation function, $\eta$ is additive noise and $g$ is the degraded observation. Restoration tries to produce an estimate $\hat f$ of the original.

In the simplest noise-only case, noise is added to the image and a suitable noise-removal filter is applied afterward. Restoration can also be performed in the frequency domain:

```
Degraded image → frequency transformation → filtering / processing
→ inverse transformation → restored image
```

This frequency-domain route is not a new type of degradation. It is simply another domain in which the restoration operation can be carried out.

---

## 8. Noise models: what actually needs to be remembered

A noise model describes the possible noise values using a probability distribution. Sir's main point is that the correct model should be chosen using knowledge of the physical source of the noise, rather than by blindly picking a convenient curve.

The easiest way to revise the six models is to attach each one to its **shape and association**. **Gaussian** is the symmetric bell-shaped curve and is associated with poor illumination. **Rayleigh** rises to a peak and then has a long right-hand tail; Sir associates it with range images. **Gamma** is a positively skewed hump and **exponential** starts high and falls continuously; both are associated with laser imaging. **Uniform** stays flat over an interval and is described as the least-used of these examples. **Impulse or salt-and-pepper noise** produces isolated very dark and very bright values, giving two spike-like components, and is associated with faulty switching during imaging.

A useful grouping is therefore:

```
Gaussian → bell → poor illumination
Rayleigh → hump + right tail → range image
Gamma + Exponential → laser imaging
Uniform → flat → least used
Impulse → two extreme spikes → faulty switching / salt-and-pepper
```

Sir also emphasizes estimating noise parameters from the histogram of a **small flat region**. The reason is important: in a flat area the true scene intensity changes very little, so the observed variation is much more likely to come from noise. A whole-image histogram can contain several peaks simply because the image contains several different objects or background intensities.

---

## 9. Digital video as a spatiotemporal signal

A digital video is an ordered sequence of image frames. Each individual frame is still a 2D image described by spatial coordinates $x$ and $y$. What makes the complete video different is that the frames are arranged over time, so we also need a time or frame index $t$:

$$
V(x,y,t).
$$

This is why Sir calls video a **3D spatiotemporal signal**. The third dimension is **time**, not physical depth. Saying “a video is a sequence of 2D images” and saying “a video is a 3D signal” are therefore both correct: each frame is 2D, while the complete stack varies over $x$, $y$ and $t$.

A $1920\times1080$ video with 100 frames can be thought of as an array of size $1920\times1080\times100$ for one value per spatial-temporal sample location. That tells us the number of samples, not the final file size; channels, bit depth and compression still affect storage.

---

## 10. Video processing: pipeline and kinds of operations

Sir shows two different ideas here, and they should not be mixed.

The first is the **video-processing pipeline**:

```
Acquisition → Sampling → Quantization → Compression
→ Storage / Transmission → Display / Playback
```

Acquisition captures the video. Sampling turns the continuous signal into discrete spatial and temporal measurements. Quantization restricts pixel intensities to finite levels. Compression reduces the amount of data, after which the video can be stored or transmitted and later reconstructed for playback.

The second idea is the **type of operation performed on the video**. **Spatial processing** works inside individual frames, so familiar image operations such as filtering, enhancement and edge detection fit here. **Temporal processing** looks across frames over time, so frame differencing and motion estimation belong here. Motion analysis then uses these changes to detect or track moving objects. Video enhancement improves visual quality, while video segmentation separates meaningful objects or foreground regions.

For compression, keep one distinction clear: **intra-frame** means redundancy is reduced within one frame; **inter-frame** means redundancy is reduced by exploiting similarity between different frames over time.

---

## 11. Video processing vs video analytics

This distinction is easier if you focus on the output.

**Video processing** changes the signal itself. It works mainly with pixels and frames and includes operations such as filtering, enhancement, compression, resizing, stabilization and frame extraction. Its output is usually another video or processed frames.

**Video analytics** tries to understand what is happening. It focuses on objects, events and behaviour, using tasks such as object detection, tracking, classification and activity recognition. Its output is information: counts, events, alerts, statistics or decisions. Analytics also frequently uses AI/ML/DL, whereas ordinary processing does not necessarily require it.

The clean distinction from Sir's table is:

> **Processing asks “How can I process or modify this video?” Analytics asks “What is happening in this video?”**
> 

Removing noise or resizing a 4K video to HD is processing. Detecting a person entering a restricted area or counting vehicles is analytics.

---

## 12. Visual surveillance and the intelligent-surveillance pipeline

A basic visual-surveillance system combines **object detection** and **object tracking**. Detection tells us where an object is in a frame. Tracking keeps the identity or association of that object across later frames. A CCTV sequence therefore becomes useful surveillance information by first finding objects and then following them over time.

Sir's intelligent-surveillance pipeline can be remembered as a six-stage story rather than as a large diagram:

```
Video capture
→ Pre-processing + feature extraction
→ Spatiotemporal modelling
→ Anomaly / intrusion detection
→ Alert generation
→ Monitoring + response
```

The logic is straightforward. First capture the video and clean/extract useful information. Then model appearance and motion across time. Compare the observed behaviour with what is considered normal, detect an anomaly or intrusion, generate an alert, and finally support monitoring or a response action. The many example bullets inside Sir's diagram are less important than this overall flow.

---

## 13. Object detection by background modelling

Background modelling learns what the **normal scene/background** looks like, usually with a static camera. A new frame is compared with that background model. Regions that differ enough are treated as possible foreground objects.

If $I_t(x,y)$ is the current frame and $B_t(x,y)$ is the background estimate, the difference image is

$$
D_t(x,y)=|I_t(x,y)-B_t(x,y)|.
$$

If the difference is larger than a threshold, that location is marked as foreground. The resulting binary mask is then cleaned to remove small noise or fill holes, after which the object can be localized.

The pipeline is therefore:

```
Video frames → build/update background → current frame − background
→ threshold → clean mask → detect object
```

Sir also shows a running-average background update:

$$
B_t=(1-\alpha)B_{t-1}+\alpha I_t.
$$

Do not treat this as a formula to fear. It simply says: **new background = mostly old background + some amount of the current frame**. A small $\alpha$ adapts slowly; a larger $\alpha$ adapts faster.

The main idea to retain is that background modelling asks:

> **“What is different from the normal scene?”**
> 

---

## 14. Object detection by object modelling

Object modelling takes the opposite approach. Instead of learning the background, it builds a representation of the **target object** and searches for regions in a new image that match that representation.

The logical sequence is:

```
Collect examples of target → extract useful features → build object model/template
→ search candidate regions → compare candidates with model
→ best match above threshold → detect and localize object
```

For Sir's car example, the system learns a representation of the car, checks candidate regions in a new road image, gives them similarity scores, and selects a sufficiently strong match.

The feature names and similarity measures shown on the slide are implementation examples; the core concept is the representation-search-match-localize pipeline.

The distinction worth keeping clear is:

```
Background modelling → learn the scene → detect what differs
Object modelling     → learn the target → detect what matches
```

---

## 15. Frame differencing for moving-object detection

Frame differencing is simpler than background modelling because it does not maintain a persistent background model. It compares **two consecutive frames** and looks for locations whose intensities changed.

For the current frame $I_t$ and previous frame $I_{t-1}$,

$$
D_t(x,y)=|I_t(x,y)-I_{t-1}(x,y)|.
$$

The absolute value matters because we care about the **size of the change**, whether the pixel became brighter or darker.

The difference image is thresholded:

$$
B_t(x,y)=
\begin{cases}
1,&D_t(x,y)\ge T,\\
0,&\text{otherwise}.
\end{cases}
$$

Large changes become foreground/motion; small changes remain background. The binary mask is then cleaned and used for object detection.

The full flow is:

```
Consecutive frames → absolute frame difference → threshold
→ binary foreground mask → post-processing → object detection
```

A useful visual intuition is that when an object moves, **both its old position and its new position have changed**. Therefore the raw difference image can show two changed regions or a ghost-like outline instead of a perfect single object silhouette.

Frame differencing is simple and fast and works well when the camera/background is static and the object moves. Its weakness is that illumination changes or camera noise can also create frame-to-frame differences and therefore appear as false motion.

The distinction from background modelling is especially important:

```
Frame differencing  → current frame compared with previous frame
Background modelling → current frame compared with learned background
```

<aside>
✅

**Stopping point for this revision sheet: Sir's page 108.** The remaining video-analysis material starts after this point and is intentionally not included yet.

</aside>