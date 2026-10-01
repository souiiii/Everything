# Computer Vision Midsem — Complete Study Notes

These notes cover **Image Processing and Video Analysis**, with theory and calculations together. Unmarked sections are **CORE**. Sections labelled **SUPPLEMENTARY G1–G9** provide limited exam-safety coverage.

Prepare for a likely mix of two numericals, three theory questions and one 2-mark question; this is not a confirmed mark scheme. **Front/forward scatter vs backscatter is a priority.** Video topics also require preparation, even without a matching PYQ.

PYQ abbreviations: **IT25** = [2025 IT CV](../Sources/PYQs/2025_IT_Computer_Vision_Midsem.md), **A** = [Legacy Fundamentals A](../Sources/PYQs/Legacy_Fundamentals_of_Image_Processing_A.md), **IPCV** = [Legacy Image Processing and CV](../Sources/PYQs/Legacy_Image_Processing_and_Computer_Vision.md), **B** = [Legacy Fundamentals B](../Sources/PYQs/Legacy_Fundamentals_of_Image_Processing_B.md).

For numerical readiness, give early attention to intensity transforms, histogram equalisation, convolution/padding and bit planes, followed by the supplementary Pascal kernel, histogram moments and IT25 geometric-transform example. Learn the surrounding explanations as well: several PYQs ask theory about these same methods.

## 1. Images, digital representation, sampling and quantization

*PYQ: IT25 Q1.1–Q1.4.*

An **image** is a visual representation of an object, scene or information. In image processing, a grayscale image is represented by an intensity function: its coordinates identify a location, and its value describes brightness there. A **digital image** stores a finite array of picture elements, or **pixels**, at discrete locations. A grayscale pixel contains one intensity value; a colour pixel can contain a vector of channel values. Thus an image array contains measured information, not merely a collection of visible squares.

An **analog image representation** varies continuously in position and intensity. A digital representation samples positions and quantizes intensities into discrete values that a computer can store and manipulate numerically. The distinction concerns representation: displaying a digital image smoothly does not make its stored data continuous.

### 1.1 Raster, vector, binary and grayscale

These distinctions answer different questions. Raster/vector describes how image content is represented; binary/grayscale describes the intensity information stored in pixels.

| Representation | Structure and consequence |
| --- | --- |
| **Raster** | A pixel grid, suitable for photographs. Enlargement reveals or interpolates a finite set of samples, so detail cannot increase merely by scaling. |
| **Vector** | Mathematical shapes, lines and curves. Shapes can be redrawn at a new size without the pixelation of an enlarged fixed raster; logos are a common example. |
| **Binary image** | Two possible pixel values, logically 0 and 1, requiring one bit per pixel in an ideal packed representation. A displayed white pixel may be shown as 255 while still representing logical 1. |
| **Grayscale image** | Multiple brightness levels without separate colour channels. With $b$ bits per pixel there are $L=2^b$ possible levels, conventionally numbered 0 to $L-1$. |

For an 8-bit grayscale image, $L=256$, with values **0–255**. There are 256 levels, not a maximum value of 256. This is an 8-bit convention, not a rule that all grayscale images must have 256 levels.

### 1.2 Sampling and quantization

**Sampling** selects discrete spatial positions. If the spacing between positions is $\Delta x$ horizontally and $\Delta y$ vertically, the samples of continuous image $f$ are

$$
f[m,n]=f(m\Delta x,n\Delta y).
$$

**Quantization** maps each sampled intensity to one of the allowed levels. Writing $Q$ for this mapping, the stored image is

$$
g[m,n]=Q\{f(m\Delta x,n\Delta y)\}.
$$

Smaller sampling intervals provide more spatial samples and finer spatial resolution. More quantization levels provide finer intensity resolution. Increasing one does not automatically increase the other: a large image may still have only two intensity levels.

**Diagram to remember:** continuous image → spatial sampling/grid → intensity quantization/staircase → digital array. Label the spatial spacing separately from the intensity levels.

**Worked storage example.** A $512\times512$ grayscale image at 8 bits/pixel has

$$
N=512\times512=262144\text{ pixels},\qquad
\text{storage}=N\times8=2097152\text{ bits}.
$$

Divide by 8 to obtain $262144\text{ bytes}=256\text{ KiB}$, ignoring headers/compression. This is often written approximately as 256 KB; KiB specifies the binary unit. The calculation uses a single grayscale value per pixel; do not add three colour channels unless the question specifies them.

## 2. Image formation and scattering

*PYQ: IPCV Q1.1.*

### 2.1 Illumination and reflectance

An imaging system receives light from a scene and forms an image on its image plane. The image intensity depends on both the light incident on the scene and the fraction reflected toward the imaging system:

$$
f(x,y)=i(x,y)r(x,y),
$$

where $i$ is **illumination** and $r$ is **reflectance**. A dark pixel may therefore result from weak illumination, low reflectance, or both. This product explains why a pixel value is not simply an intrinsic property of the object.

**Diagram to remember:** optical source → physical scene → imaging system → image plane/pixel grid. Put the illumination–reflectance product between scene and recorded image.

### 2.2 Front/forward scatter vs backscatter

In underwater or hazy scenes, light interacts with suspended particles before reaching the camera. There are three relevant light paths. **Direct light** travels from the object to the camera and carries scene detail. **Forward-scattered light** starts from the object but is deflected by particles on its way to the camera; spreading light from an object point blurs its appearance. **Backscattered light** is illumination scattered by particles toward the camera without first conveying the object’s reflected appearance; it adds a veil that reduces contrast.

| Feature | Front/forward scatter | Backscatter |
| --- | --- | --- |
| Light path | Object → particle → camera | Illumination → particle → camera |
| Main image effect | Spreading of object light; blur and loss of fine detail | Added background/veiling light; reduced object–background contrast |
| What to label | Deflected object-light path | Particle-light path entering the camera |

**Front scatter** refers here to **forward scattering**: object light is deflected on its way to the camera.

```
Illumination ──> object ─────────────────────> camera   (direct)
                   └──> particle ──────────> camera   (forward scatter)
Illumination ──────────> particle ──────────> camera   (backscatter)
```

A simplified image-formation model is

$$
I_c(x,y)=J_c(x,y)t(x,y)+[1-t(x,y)]b_c.
$$

Here $I_c$ is the **acquired image**, $J_c$ the **clear image**, $t$ the **transmission map**, and $b_c$ the **surrounding light**, with $c$ denoting a colour channel. The first term is transmitted scene information; the second is the surrounding-light contribution. When $t$ is close to 1, the acquired value is close to the clear value. When $t$ is small, the surrounding-light term dominates. **Caution:** this simplified model has no separate forward-blur kernel; distinguish the physical scattering paths from the terms represented in the equation.

### 2.3 SUPPLEMENTARY G7 — imaging modalities and EM sources

*IT25 Q2(b), IPCV Q1.2, B Q2.*

Images can be formed from different physical signals. For an EM-spectrum question, identify the radiation, what the detector measures, and a representative use; wavelengths and detailed device physics are unnecessary here.

| EM band | Formation principle and representative use |
| --- | --- |
| Gamma rays | Detect high-energy radiation, for example emissions associated with a tracer, to map its distribution in nuclear imaging. |
| X-rays | Measure radiation transmitted through an object; differing attenuation produces contrast, as in bone or baggage imaging. |
| Ultraviolet | Record UV interaction or UV-induced fluorescence, useful for inspecting surface features or fluorescent specimens. |
| Visible light | Record reflected/transmitted visible illumination; ordinary cameras and machine inspection. |
| Infrared | Record reflected IR or emitted thermal IR, depending on the band/system; thermal inspection and night observation. |
| Microwaves | Record emitted radiation or transmitted-signal echoes, as in radar imaging for terrain or weather observation. |
| Radio waves | Detect radio-frequency signals; MRI uses RF excitation and measured responses in a magnetic field to form images. |

**Infrared imaging** uses an IR-sensitive detector to convert incoming infrared radiation into an image. Thermal IR systems represent differences in emitted radiation associated with temperature, allowing observation when visible illumination is poor. Two suitable applications are **night surveillance** and **detecting overheating electrical equipment**. Not all IR images are thermal: some use reflected IR illumination.

**Ultrasound imaging** uses a transducer to send high-frequency sound pulses into a medium and receive echoes from interfaces. Echo travel time indicates depth and echo strength contributes image brightness; repeated measurements build a cross-sectional image. Two applications are **prenatal imaging** and **industrial internal-defect inspection**. Ultrasound is **sound, not electromagnetic radiation**, so do not place it inside the EM-band table.

## 3. Digital image processing: purpose and stages

*PYQ: IT25 Q2(a).*

**Digital image processing (DIP)** is the computer-based representation and manipulation of pictorial information. It may improve an image for human interpretation or prepare information for machine processing, storage and transmission. Image processing **improves/transforms the image**, while computer vision **understands the image**. Removing noise from an X-ray is image processing; identifying an abnormality in it is image analysis/computer vision.

The main processing stages serve the following functions. They are not a requirement that every application execute every stage in a rigid sequence. For example, enhancement may help segmentation, while compression supports storage at a different point in a system.

| Stage | Meaning and contribution |
| --- | --- |
| **Image acquisition** | Obtain the image with a sensor and convert it into usable digital samples. Poor acquisition limits the information available later. |
| **Image enhancement** | Make relevant information easier to see or use, such as by improving contrast. Suitability depends on the application. |
| **Image restoration** | Estimate the original image from a degraded observation using a known or estimated degradation model. |
| **Morphological processing** | Process image shapes/structures, for example cleaning small unwanted features in a mask. |
| **Segmentation** | Separate an image into meaningful regions or objects so subsequent analysis can focus on them. |
| **Object recognition** | Assign meaning or an identity/class to an object, rather than merely marking a region. |
| **Representation and description** | Represent regions or boundaries and describe them through useful properties/features for later interpretation. |
| **Image compression** | Reduce the data needed to store or transmit the image. |
| **Colour image processing** | Use and manipulate colour information when it is relevant to the task. |

**Diagram to remember:** draw acquisition, enhancement, restoration, morphological processing, segmentation, recognition, and representation/description as the main blocks; include compression and colour processing as supporting blocks. Explain each block’s contribution rather than listing names alone.

Applications include medical and biological imaging; remote sensing/GIS and meteorology; robotics and autonomous vision; surveillance, biometrics and forensics; industrial/agricultural quality inspection; entertainment; document/handwriting analysis; and cultural-heritage restoration. In an exam, connect a few examples to a function: segmentation isolates a region, enhancement exposes faint information, and recognition supplies an object label.

### 3.1 SUPPLEMENTARY G9 — cost and input quality

Processing cost grows with the amount of image data and the work performed at each location. For example, increasing image resolution or the size of a sliding filter means more pixel operations. Input noise, blur or poor contrast can hide useful information and produce unreliable segmentation or recognition; preprocessing may help, but it also adds computation and cannot recover information that was never captured. Thus an algorithm must balance the input quality, task requirements and available processing time.

### 3.2 SUPPLEMENTARY G6 — lossy vs lossless compression

*IT25 Q1.5, 2 marks.* **Lossless compression** permits exact recovery of the original pixel data; PNG is an example. **Lossy compression** discards some information to reduce size and cannot generally reconstruct the original exactly; ordinary JPEG compression is an example. The distinction is recoverability, not simply whether a compressed image looks visibly worse.

**Classification map — enhancement vs restoration**

```mermaid
flowchart TD
    DIP["Digital Image Processing"]

    DIP --> ENH["Image Enhancement<br>Goal: make the image more suitable for a task"]
    DIP --> RES["Image Restoration<br>Goal: estimate/recover the original image"]

    ENH --> ES["Spatial Domain<br>operate directly on pixels"]
    ENH --> EF["Frequency Domain<br>transform → process/filter → inverse transform"]

    ES --> PP["Point Processing<br>same input pixel only"]
    ES --> HP["Histogram Processing"]
    ES --> NF["Neighbourhood Filtering<br>kernel/mask + convolution"]

    PP --> PT["Negative • Log/Inverse Log • Power-Law/Gamma<br>Contrast Stretching • Thresholding<br>Gray-Level Slicing • Bit-Plane Slicing"]
    HP --> HE["Histogram Equalisation"]

    NF --> SM["Smoothing"]
    NF --> ED["Edge / Sharpening"]
    SM --> MG["Mean • Gaussian"]
    ED --> ESL["Prewitt • Sobel • Laplacian • Sharpening mask"]

    RES --> RS["Spatial Domain<br>direct pixel-domain restoration/filtering"]
    RES --> RF["Frequency Domain<br>transform → process/filter → inverse transform"]
```

## 4. Enhancement by point processing

*PYQ: IT25 Q3(a); A Q1.2/Q2; B Q3(a)/Q3(b).*

Enhancement produces an image more suitable for a particular application. **Spatial-domain methods** operate directly on pixels; **frequency-domain methods** operate on a transformed representation and then return to the image domain. The following methods operate in the spatial domain.

Let $f(x,y)$ be the input and $g(x,y)$ the output. A spatial operation is written

$$
g(x,y)=T[f(x,y)].
$$

$T$ may use a neighbourhood centred at $(x,y)$. If that neighbourhood is only $1\times1$, the output depends on the input pixel alone, giving **point processing**:

$$
s=T(r),
$$

where $r$ and $s$ are input and output gray levels. The same input value receives the same output wherever it occurs. For an 8-bit input, a **lookup table (LUT)** can store the 256 possible output values once, then use each pixel value as an index.

### 4.1 Negative transformation

For levels 0 through $L-1$, the negative is

$$
s=(L-1)-r.
$$

It reverses brightness order: black becomes white and high input values become low output values. In an 8-bit image, $r=50$ becomes $255-50=205$. It is useful for viewing details whose visibility improves under reversed contrast, including medical images.

**Curve:** a descending straight line from $(0,L-1)$ to $(L-1,0)$. Use $L-1-r$: $L$ is the number of levels, so the maximum value is $L-1$. Using $L-r$ would produce the invalid level $L$ at $r=0$.

### 4.2 Logarithmic and inverse-log behaviour

A log transform is

$$
s=c\log(1+r),\qquad c>0.
$$

The 1 makes the mapping defined at zero. The curve rises rapidly at small $r$ and more slowly at large $r$: relative to the available output range, it separates lower intensities while compressing a large range of high intensities. This helps display data with a large dynamic range. The constant $c$ and log base set the output scale; they must be used consistently. For example, with $c=1$ and log base 2, $r=7$ gives $s=\log_2 8=3$.

**Inverse log** has the opposite shape to log: it compresses the lower part and expands the higher part of the intensity range. Distinguish the two by the shape of their curves.

### 4.3 Power law and gamma correction

The power-law transformation is

$$
s=cr^\gamma,\qquad c>0,\ \gamma>0.
$$

Gamma changes the shape of the curve. To discuss brightness without confusing scaling, use normalized input $r$ in $[0,1]$ and $c=1$: $\gamma<1$ **raises intermediate values**, $\gamma=1$ **is identity**, and $\gamma>1$ **lowers intermediate values**. For $r=0.25$, $\gamma=0.5$ gives 0.5, whereas $\gamma=2$ gives 0.0625. When working with raw gray levels, follow the question’s $c$ and range rather than applying this normalized shortcut blindly.

**Gamma correction** compensates for a display’s nonlinear intensity response. The displayed gradient can differ from the intended gradient; applying a compensating curve before display improves the match. Power-law curves also provide controllable brightening/darkening for enhancement.

| Transformation | Shape/effect to explain in IT25 Q3(a) |
| --- | --- |
| Negative | Decreasing line; reverses intensity ordering. |
| Logarithmic | Concave increasing curve; expands low-level distinctions and compresses high dynamic range with suitable scaling. |
| Power law | Family controlled by $\gamma$; normalized $\gamma<1$ brightens, $\gamma>1$ darkens. |

**Diagram to remember:** plot output $s$ against input $r$, add identity, negative, log, and representative power curves; label the gamma cases and the monitor-compensation idea.

### 4.4 Worked power-law numerical — A Q2 / B Q3(a)

Both papers give the same $4\times4$, 4-bit input and require $g=\operatorname{round}(10\sqrt f)$:

$$
f=\begin{bmatrix}12 & 8 & 4 & 9\\10 & 5 & 3 & 6\\8 & 12 & 9 & 13\\4 & 12 & 9 & 10\end{bmatrix}
$$

This is $c=10$, $\gamma=1/2$. Compute the transform for each distinct input value, round the result to the nearest integer, and replace each occurrence. For example, 12 gives $10\sqrt{12}\approx34.641\to35$; 5 gives $22.361\to22$.

| $f$ | 3 | 4 | 5 | 6 | 8 | 9 | 10 | 12 | 13 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| $10\sqrt f$, approximately | 17.321 | 20 | 22.361 | 24.495 | 28.284 | 30 | 31.623 | 34.641 | 36.056 |
| $g$ after rounding | 17 | 20 | 22 | 24 | 28 | 30 | 32 | 35 | 36 |

$$
g=\begin{bmatrix}35 & 28 & 20 & 30\\32 & 22 & 17 & 24\\28 & 35 & 30 & 36\\20 & 35 & 30 & 32\end{bmatrix}
$$

The question specifies **4-bit input**, not a 4-bit output constraint. It gives no clipping instruction, so do not replace values above 15 by 15. Round after evaluating $10\sqrt f$, not before multiplication.

### 4.5 Contrast stretching and thresholding

Contrast stretching increases the range occupied by useful gray levels. A piecewise-linear stretching curve passes through $(0,0)$, $(r_1,s_1)$, $(r_2,s_2)$, and $(L-1,L-1)$. Each line segment has its own slope: a slope above 1 expands differences in that interval; below 1 compresses them. The positions of the two internal control points determine which intensities receive increased contrast.

For the nondegenerate case $0<r_1<r_2<L-1$, the mapping is

$$
s=\begin{cases}
(s_1/r_1)r,&0\le r\le r_1,\\
s_1+\dfrac{s_2-s_1}{r_2-r_1}(r-r_1),&r_1<r\le r_2,\\
s_2+\dfrac{L-1-s_2}{L-1-r_2}(r-r_2),&r_2<r\le L-1.
\end{cases}
$$

**Worked example:** for $L=256$, $(r_1,s_1)=(50,20)$ and $(r_2,s_2)=(150,220)$, input $r=100$ lies in the middle segment. Its slope is $(220-20)/(150-50)=2$, giving $s=20+2(100-50)=120$. Do not use the first segment’s slope for every pixel.

If $r_1=s_1$ and $r_2=s_2$, the curve becomes identity. The limiting case $r_1=r_2$, $s_1=0$, $s_2=L-1$ becomes **thresholding**, which separates pixels into two classes. With the explicit convention “high when $r\ge T$,”

$$
s=\begin{cases}0,&r<T,\\L-1,&r\ge T.\end{cases}
$$

For $T=100$, inputs 50,100,200 become 0,255,255 in an 8-bit displayed binary image. Treat the threshold as a separate rule; do not divide by $r_2-r_1$ when those values coincide.

**Diagram to remember:** the piecewise rising curve and its limiting vertical threshold step, with $r_1,r_2,s_1,s_2$ labelled.

### 4.6 Gray-level slicing

Gray-level slicing highlights an interval of intensities, for example features occupying a selected gray-level band. There are two variants: make the selected interval bright and suppress everything else, or make it bright while retaining the original background intensities. Unlike thresholding’s single split, slicing selects a band.

For example, for the interval $80\le r\le160$, set selected pixels to 255. Inputs $[50,100,200]$ become $[0,255,0]$ when the background is suppressed, and $[50,255,200]$ when it is preserved.

**Diagram to remember:** a rectangular high band over $[A,B]$ on a zero background; beside it, the same high band interrupting the identity line. State whether the endpoints belong to the selected band if a numerical gives no convention.

### 4.7 Bit-plane slicing

A $b$-bit pixel is the sum of $b$ binary contributions. Let $B_k(x,y)$ be bit $k$, with $k=0$ the least significant bit (LSB):

$$
B_k(x,y)=\left\lfloor\frac{f(x,y)}{2^k}\right\rfloor\bmod2,
\qquad f(x,y)=\sum_{k=0}^{b-1}2^kB_k(x,y).
$$

Each **bit plane** contains that bit from every pixel. Higher bits have larger numerical weights and usually carry the most visible image structure; lower bits contribute finer intensity changes. An 8-bit decomposition places plane 0 at the LSB end and plane 7 at the MSB end. Plane 7 is exactly the logical threshold $f\ge128$. The observation that higher planes are more visually significant does not mean lower planes are always noise.

**Worked B Q3(b), 2 marks.** Use the input $f$ from §4.4, not its transformed output $g$. Write each value as four bits: $12=(1100)_2$, $8=(1000)_2$, $4=(0100)_2$, $9=(1001)_2$, $10=(1010)_2$, $5=(0101)_2$, $3=(0011)_2$, $6=(0110)_2$, $13=(1101)_2$. Reading corresponding bit positions gives the following planes. The weights of $B_3,B_2,B_1,B_0$ are 8, 4, 2 and 1, respectively.

$$
B_3=\begin{bmatrix}1 & 1 & 0 & 1\\1 & 0 & 0 & 0\\1 & 1 & 1 & 1\\0 & 1 & 1 & 1\end{bmatrix},\qquad
B_2=\begin{bmatrix}1 & 0 & 1 & 0\\0 & 1 & 0 & 1\\0 & 1 & 0 & 1\\1 & 1 & 0 & 0\end{bmatrix}
$$

$$
B_1=\begin{bmatrix}0 & 0 & 0 & 0\\1 & 0 & 1 & 1\\0 & 0 & 0 & 0\\0 & 0 & 0 & 1\end{bmatrix},\qquad
B_0=\begin{bmatrix}0 & 0 & 0 & 1\\0 & 1 & 1 & 0\\0 & 0 & 1 & 1\\0 & 0 & 1 & 0\end{bmatrix}
$$

Check a location: the top-left bits reconstruct $8\times1+4\times1+2\times0+1\times0=12$. A bit plane contains **0/1**, not the weighted values 0/8, 0/4, etc. Draw a stack of four or eight 1-bit sheets to explain decomposition visually.

## 5. Histograms and histogram equalisation

*PYQ: IPCV Q1.4/Q3(b); B Q1.2.*

### 5.1 What a histogram represents

For gray level $r_k$, let $n_k$ be its number of occurrences in an image containing $N$ pixels. The **histogram** is $h(r_k)=n_k$. The **normalized histogram** is

$$
p(r_k)=\frac{n_k}{N},\qquad \sum_kp(r_k)=1.
$$

Normalization turns counts into fractions/probabilities without changing the histogram’s shape. A histogram describes the distribution of intensities, not where those intensities occur. Different spatial arrangements may have the same histogram.

The intensity distribution helps explain image appearance: a dark image concentrates values toward low intensities; a bright image toward high intensities; a low-contrast image occupies a relatively narrow band; a higher-contrast image uses a broader range. A broad distribution may be more evenly spread, but a high-contrast histogram need not be flat.

### 5.2 Worked histogram — IPCV Q3(b)

The question supplies the following frequencies. First total them: $N=40+45+55+50+40+35+65+70=400$. Divide each count by 400.

| Gray level $r_k$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| $n_k$ | 40 | 45 | 55 | 50 | 40 | 35 | 65 | 70 |
| $p(r_k)$ | .1000 | .1125 | .1375 | .1250 | .1000 | .0875 | .1625 | .1750 |

The following horizontal bar plots show both distributions; in an exam, put gray level on the horizontal axis and the corresponding count/probability on the vertical axis.

```
Histogram: each # = 5 pixels       Normalized: each # = 0.0125
0 | ########       40             0 | ########       .1000
1 | #########      45             1 | #########      .1125
2 | ###########    55             2 | ###########    .1375
3 | ##########     50             3 | ##########     .1250
4 | ########       40             4 | ########       .1000
5 | #######        35             5 | #######        .0875
6 | #############  65             6 | #############  .1625
7 | ############## 70             7 | ############## .1750
```

**Caution:** these counts are not monotonically increasing. Plot the given frequencies rather than assuming a rising histogram.

### 5.3 SUPPLEMENTARY G2 — mean and standard deviation

The **mean intensity** is a weighted average of gray levels. The **standard deviation** measures their spread about that mean. For the image’s full intensity distribution, use population moments:

$$
\mu=\sum_kr_kp(r_k),\qquad
\sigma^2=\sum_k(r_k-\mu)^2p(r_k)
=\sum_kr_k^2p(r_k)-\mu^2,\qquad
\sigma=\sqrt{\sigma^2}.
$$

Continue the same IPCV numerical:

$$
\sum r_kn_k=0+45+110+150+160+175+390+490=1520,
$$

$$
\sum r_k^2n_k=0+45+220+450+640+875+2340+3430=8000.
$$

Therefore $\mu=1520/400=3.8$, $E[r^2]=8000/400=20$, $\sigma^2=20-3.8^2=5.56$, and $\sigma\approx2.358\text{ gray levels}$. Keep enough precision until the final square root; do not average the eight gray-level labels equally or divide by $N-1$.

For **B Q1.2**, explain that a small standard deviation means intensities are concentrated, whereas a larger one indicates a broader spread and often greater global contrast. It does not by itself establish image quality or spatial detail. The mean describes average brightness, not spread.

### 5.4 Equalisation: why the CDF is used

Histogram equalisation redistributes intensity values to use the available range more effectively. It is useful for many dark, bright or washed-out images. The mapping is based on how much probability has accumulated up to each intensity, called the **cumulative distribution function (CDF)**:

$$
CDF(r_k)=\sum_{j=0}^{k}p(r_j),\qquad
s_k=\operatorname{round}\big[(L-1)CDF(r_k)\big].
$$

The CDF never decreases, so the mapping preserves intensity order, although different input levels may merge. Multiplication by $L-1$ converts the normalized cumulative value into the output gray-level range; rounding selects an integer level.

**Caution:** include the probability of gray level 0 in the cumulative sum. Scale the normalized CDF by $L-1$, not $L$, and round to obtain integer output levels. The worked example below uses this mapping without minimum-CDF subtraction.

### 5.5 Complete equalisation numerical

For $L=8$ and $N=16$, consider the image

$$
f=\begin{bmatrix}3 & 4 & 5 & 6\\5 & 4 & 3 & 5\\6 & 5 & 4 & 5\\5 & 6 & 4 & 3\end{bmatrix}
$$

**Step 1: count. Step 2: normalize. Step 3: accumulate. Step 4: scale and round.**

| $r_k$ | $n_k$ | $n_k/16$ | CDF | $7\times CDF$ | $s_k$ |
| --- | --- | --- | --- | --- | --- |
| 0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 0 | 0 | 0 | 0 | 0 |
| 2 | 0 | 0 | 0 | 0 | 0 |
| 3 | 3 | .1875 | .1875 | 1.3125 | 1 |
| 4 | 4 | .2500 | .4375 | 3.0625 | 3 |
| 5 | 6 | .3750 | .8125 | 5.6875 | 6 |
| 6 | 3 | .1875 | 1.0000 | 7.0000 | 7 |
| 7 | 0 | 0 | 1.0000 | 7.0000 | 7 |

**Step 5: replace each input**, using $3\to1,\ 4\to3,\ 5\to6,\ 6\to7$:

$$
g=\begin{bmatrix}1 & 3 & 6 & 7\\6 & 3 & 1 & 6\\7 & 6 & 3 & 6\\6 & 7 & 3 & 1\end{bmatrix}
$$

| Output level | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Output count | 0 | 3 | 0 | 4 | 0 | 0 | 6 | 3 |

**Step 6: check** that the output still contains 16 pixels and its levels lie in 0–7. The occupied range has spread from 3–6 to 1–7, but the histogram is clearly not flat.

**Continuous vs discrete equalisation (IPCV Q1.4).** Ideal continuous equalisation can redistribute a continuous probability distribution to uniformity under suitable conditions. In a discrete image, whole groups of pixels at the same input level receive one mapped level; integer counts, rounding and possible merging prevent arbitrary redistribution. Consequently equalisation aims to improve range use and contrast, **not guarantee an exactly flat histogram**.

**Diagram to remember:** input histogram → nondecreasing CDF/mapping curve → output histogram; show that gaps and unequal bars may remain.

## 6. Spatial filtering, smoothing and sharpening

*PYQ: IT25 Q3(b); A Q4–Q5; B Q5; IPCV Q4(a).*

### 6.1 Neighbourhood filtering and convolution

A spatial filter computes an output from a small neighbourhood around each pixel. Its **kernel/mask** contains the weights assigned to those neighbours. Weighted averaging suppresses local fluctuations; signed difference weights emphasize changes. Thus the weights determine whether filtering smooths or highlights edges.

For centred offsets $u,v$ and kernel $h$, **convolution** is

$$
g(i,j)=\sum_u\sum_vh(u,v)f(i-u,j-v).
$$

Here $i,j$ are row/column indices. To calculate the output with a sliding window:

1. Flip the supplied kernel in both horizontal and vertical directions, equivalent to a $180^\circ$ rotation.
2. Place its centre over the output location being computed.
3. Multiply the flipped weights by the overlapping input values and add all products.
4. Repeat at every location. Pad undefined input positions to preserve the image size; use the boundary rule supplied by the problem.

Multiplying by an **unflipped** kernel is correlation. Symmetric mean/Gaussian masks hide the difference, but signed derivative masks may reverse the output sign. The worked example below uses zero-valued positions outside the image.

### 6.2 Complete convolution numerical

$$
f=\begin{bmatrix}1 & 2 & 3\\4 & 5 & 6\\7 & 8 & 9\end{bmatrix},\qquad
h=\begin{bmatrix}-1 & -2 & -1\\0 & 0 & 0\\1 & 2 & 1\end{bmatrix},\qquad
h_{\mathrm{flipped}}=\begin{bmatrix}1 & 2 & 1\\0 & 0 & 0\\-1 & -2 & -1\end{bmatrix}
$$

The kernel centre is its middle zero. Pad the $3\times3$ image with one zero border. At the **top-left output**, the neighbourhood is

$$
\begin{bmatrix}0 & 0 & 0\\0 & 1 & 2\\0 & 4 & 5\end{bmatrix}
$$

Multiply by the flipped mask: the first row contributes 0, the middle row contributes 0, and the last row contributes $-0-2\times4-5=\mathbf{-13}$.

At the **centre**, all values are available:

$$
g(1,1)=(1+2\times2+3)-(7+2\times8+9)=8-32=\mathbf{-24}.
$$

At the **top-middle**, only the negative bottom weights contribute: $-4-2\times5-6=-20$. At the **bottom-right**, only the positive top weights contribute: $5+2\times6+0=17$. Repeating the same placement gives the full output:

$$
g=\begin{bmatrix}-13 & -20 & -17\\-18 & -24 & -18\\13 & 20 & 17\end{bmatrix}
$$

Keep the signed filter response; no instruction here requires clipping it into a display range. Check the output dimensions, centre placement and flip before doing all nine sums.

### 6.3 Mean and Gaussian smoothing

The **mean filter** replaces a pixel by its neighbourhood average, giving each location equal weight. It reduces rapid local fluctuations but also smooths genuine boundaries and fine detail. A **Gaussian-weighted mask** gives more influence to the centre and nearby positions, producing a smoother weighted average. Commonly used $3\times3$ smoothing masks are

$$
\text{Mean}=\frac19\begin{bmatrix}1 & 1 & 1\\1 & 1 & 1\\1 & 1 & 1\end{bmatrix},\qquad
\text{Gaussian}=\frac1{16}\begin{bmatrix}1 & 2 & 1\\2 & 4 & 2\\1 & 2 & 1\end{bmatrix}
$$

Both have total weight 1, so a constant patch stays constant away from padding effects. **Worked example using the input patch in §6.2:** the centre mean is $45/9=5$; the Gaussian weighted sum is $1+4+3+8+20+12+7+16+9=80$, giving $80/16=5$. This equality is specific to this patch; the filters generally produce different results. Multiply first, then normalize; omitting $1/9$ or $1/16$ changes brightness rather than computing the intended average.

### 6.4 SUPPLEMENTARY G1 — $5\times5$ Gaussian by Pascal’s triangle

*A Q1.3, IPCV Q1.3, B Q1.3.* The Pascal row with five entries is $v=[1,4,6,4,1]$. Form the outer product $v^T v$, so entry $(i,j)$ equals $v_i v_j$:

$$
K=\frac{1}{256}\begin{bmatrix}1 & 4 & 6 & 4 & 1\\4 & 16 & 24 & 16 & 4\\6 & 24 & 36 & 24 & 6\\4 & 16 & 24 & 16 & 4\\1 & 4 & 6 & 4 & 1\end{bmatrix}
$$

The row sum is 16, so the outer product sums to $16^2=256$; divide by 256 to normalize. For comparison, $[1,2,1]$ has sum 4 and produces the $3\times3$ Gaussian mask above with denominator 16. This **Pascal/binomial Gaussian approximation** constructs the kernel directly from integer coefficients; no continuous Gaussian integration is needed.

### 6.5 SUPPLEMENTARY G4 — why the mean filter is low-pass

*A Q5, proof.* Low spatial frequencies describe slowly varying intensities; high frequencies describe rapid changes. Let $n$ index pixel positions, $\omega$ be angular spatial frequency and $j$ be the imaginary unit ($j^2=-1$). For a centred three-point mean, a frequency component $e^{j\omega n}$ is multiplied by

$$
H(\omega)=\frac{e^{-j\omega}+1+e^{j\omega}}3
=\frac{1+2\cos\omega}{3}.
$$

At zero frequency $H(0)=1$, so averaging preserves the constant/DC component. Near zero, the gain stays close to 1, preserving slow variation. At the rapidly alternating frequency $\pi$, the gain magnitude is only $1/3$. Averaging therefore attenuates rapid changes/high-frequency content relative to DC and slow variation, so it acts as a **low-pass filter**.

### 6.6 Derivative and sharpening masks

Derivative filters respond to changes between nearby intensities. Their weights include positive and negative terms and usually sum to zero, so a constant neighbourhood produces zero response. Prewitt and Sobel each use two directional masks:

$$
\text{Prewitt, column change: }\begin{bmatrix}-1 & 0 & 1\\-1 & 0 & 1\\-1 & 0 & 1\end{bmatrix},\qquad
\text{Prewitt, row change: }\begin{bmatrix}-1 & -1 & -1\\0 & 0 & 0\\1 & 1 & 1\end{bmatrix}
$$

$$
\text{Sobel, column change: }\begin{bmatrix}-1 & 0 & 1\\-2 & 0 & 2\\-1 & 0 & 1\end{bmatrix},\qquad
\text{Sobel, row change: }\begin{bmatrix}-1 & -2 & -1\\0 & 0 & 0\\1 & 2 & 1\end{bmatrix}
$$

Column change responds across left/right regions and helps detect vertical boundaries; row change responds across top/bottom regions and helps detect horizontal boundaries. Sobel gives the central row/column more weight. When computing convolution, flip these masks as in §6.1; do not change their signs independently of that operation.

The **Laplacian** uses second differences and combines changes in both image directions. The four-neighbour Laplacian and its associated sharpening mask are

$$
L=\begin{bmatrix}0 & 1 & 0\\1 & -4 & 1\\0 & 1 & 0\end{bmatrix},\qquad
\text{Sharpening mask}=\begin{bmatrix}0 & -1 & 0\\-1 & 5 & -1\\0 & -1 & 0\end{bmatrix}
$$

The Laplacian’s zero sum suppresses flat background. The sharpening mask has sum 1, retaining a constant background while strengthening local differences.

### 6.7 SUPPLEMENTARY G3 — complete derivative explanations

*A Q4; B Q5; IPCV Q4(a).* For a one-dimensional intensity sequence, a first difference $f(x+1)-f(x)$ measures a local change, while a second difference $f(x+1)-2f(x)+f(x-1)$ measures how that change itself varies. Both vanish on a constant region. Along a constant-slope ramp, the first difference is nonzero but constant, whereas the second is zero except at transitions. Near a sharp step, the second difference gives opposite-signed responses on the two sides.

| First-derivative operators | Second-derivative/Laplacian operator |
| --- | --- |
| Describe strength and direction of change through directional responses. | Combines second differences; the Laplacian alone does not specify an edge direction. |
| Useful for locating boundaries; Prewitt/Sobel combine differencing with averaging in the other direction. | Useful for highlighting fine changes and constructing sharpening. |
| Still affected by noise; averaging can broaden the response. | Usually more sensitive to noise and may create paired responses around a step. |

For the four-neighbour mask,

$$
\nabla^2 f(i,j)=f(i-1,j)+f(i+1,j)+f(i,j-1)+f(i,j+1)-4f(i,j).
$$

**Restoring background after the Laplacian:** the derivative image alone has lost constant/slow background content. Recombine it with the original image:

$$
g=f-\nabla^2f.
$$

The **minus sign** matches the negative-centre Laplacian above. Subtracting its kernel from the identity kernel gives centre $1-(-4)=5$ and neighbouring weights $-1$, exactly the displayed sharpening mask. If a question supplies the opposite Laplacian sign, the recombination sign changes too. Restoring the background in this sharpening operation means adding back original image content; it is not a full inversion of degradation.

## 7. Restoration and noise models

*PYQ: filtering/noise portion of IT25 Q3(b).*

**Restoration** aims to recover an image’s original content and quality from a degraded observation. It assumes the degradation mechanism is known or can be estimated. **Enhancement** instead aims for a result more suitable for a task or observer; an attractive result need not accurately reproduce the original.

If $f$ is the original image, $h$ the degradation function, $\eta$ additive noise and $g$ the observed image, the degradation model is

$$
g(x,y)=h(x,y)*f(x,y)+\eta(x,y),
$$

where $*$ denotes convolution. A restoration operation estimates the original, written $\hat f$. With noise-only degradation, the task reduces to suitable noise-removal filtering. In the frequency-domain approach, transform the degraded image, filter/process that representation, then apply the inverse transformation.

**Diagram to remember:** original $f$ → degradation $h$ → addition of noise $\eta$ → observed $g$ → restoration → estimated original $\hat f$. Put the noise arrow into the addition junction, not into the original image.

### 7.1 Noise distributions and their appearance

A noise model describes possible noise values through a probability density/distribution. Choose a model using knowledge of the acquisition physics and estimate its parameters from the histogram of a small **flat image region**, where actual scene variation is limited. A whole-image histogram also contains object/background differences and need not directly represent noise.

| Model | Shape to recognize/draw | Typical association or use |
| --- | --- | --- |
| **Gaussian** | Symmetric bell about mean $\mu$; $\sigma$ controls spread. | Poor illumination. |
| **Rayleigh** | Starts at a lower limit, rises to a peak, then has a longer right tail. | Range images. |
| **Gamma** | Asymmetric positive-side hump. | Laser imaging. |
| **Exponential** | Largest at its starting point and decays to the right. | Laser imaging. |
| **Uniform** | Constant height over an interval $[a,b]$, zero outside. | Least used of these models. |
| **Impulse / salt-and-pepper** | Two spikes corresponding to dark and bright impulses. | Faulty switching during imaging; isolated extreme pixels. |

Draw noise value on the horizontal axis and probability/density on the vertical axis. Impulse noise produces conspicuous extremes in noisy images and their histograms, while other distributions spread intensities in characteristic ways. The associations in the table are examples, not exclusive causes of each noise type. Use the distribution shape and acquisition context to recognize the model and choose a suitable filter.

Mean and Gaussian filters reduce fluctuations by averaging but may blur edges. A filter should match the observed degradation; smoothing is not automatically the best response to every noise model.

### 7.2 SUPPLEMENTARY G5 — order-statistics filters

*A Q1.1/Q7; IPCV Q3(a).* An order-statistics filter sorts neighbourhood values and selects or combines them by rank. In particular, the **median** selects the central ordered value, so a few extreme intensities influence it less than they influence a mean. This is useful for salt-and-pepper noise and often retains boundaries better than ordinary averaging; it is not a linear weighted-sum filter.

| Filter | Rule for sorted values $z_1\le\cdots\le z_N$ | Typical role |
| --- | --- | --- |
| Median | Middle value for an odd-sized window. | Suppress isolated impulse outliers. |
| Minimum | $z_1$. | Remove isolated bright “salt” values, with possible dark-region expansion. |
| Maximum | $z_N$. | Remove isolated dark “pepper” values, with possible bright-region expansion. |
| Midpoint | $(z_1+z_N)/2$. | Estimate a centre value for bounded fluctuations; sensitive to extreme outliers. |
| Alpha-trimmed mean | Discard $d/2$ lowest and $d/2$ highest values; average the remaining $N-d$. | Compromise between averaging and resistance to outliers. |

For alpha trimming, $d$ is an even integer with $0\le d<N$:

$$
g=\frac{1}{N-d}\sum_{k=d/2+1}^{N-d/2}z_k.
$$

**Worked window example:** sorted values $[0,10,11,12,13,14,15,16,255]$ give a median of 13, minimum of 0, maximum of 255 and midpoint of 127.5. With $d=2$, discard 0 and 255; the remaining sum is 91, so the alpha-trimmed mean is $91/7=13$. The ordinary mean is $346/9\approx38.44$, illustrating how the bright outlier distorts an average. Trimming $d=2$ means two values in total, not two from each end.

## 8. SUPPLEMENTARY G8 — warping and geometric transformations

*PYQ: IT25 Q3(b)/Q4; IPCV Q2(a).*

**Filtering** calculates intensity values from a pixel or neighbourhood. **Warping** changes where image content is placed by transforming coordinates and resampling the image. Smoothing noise is filtering; shifting, rotating or scaling image content is warping. Warping alone does not remove acquisition noise.

### 8.1 Homogeneous coordinates and matrix order

Represent a Cartesian point as a column vector $\mathbf p=[x,y,1]^T$. The extra coordinate lets translation and linear transforms be combined by matrix multiplication:

$$
T=\begin{bmatrix}1&0&t_x\\0&1&t_y\\0&0&1\end{bmatrix},\quad
R=\begin{bmatrix}\cos\theta&-\sin\theta&0\\\sin\theta&\cos\theta&0\\0&0&1\end{bmatrix},\quad
S=\begin{bmatrix}s_x&0&0\\0&s_y&0\\0&0&1\end{bmatrix}.
$$

Here $\theta$ is positive counter-clockwise in the usual Cartesian axes, $t_x,t_y$ are translations, and $s_x,s_y$ are scale factors. With column vectors, the **rightmost transform acts first**. Translation then rotation then scaling therefore gives $\mathbf p'=SRT\mathbf p$. Image row coordinates may point downward; do not silently substitute that convention into this explicitly Cartesian CCW question.

### 8.2 Worked IT25 Q4 — 10 marks

Given $P=(2,3)$, translate by $(3,-2)$, rotate $90^\circ$ CCW about the origin, then scale $x$ by 2 and $y$ by 1:

$$
T=\begin{bmatrix}1&0&3\\0&1&-2\\0&0&1\end{bmatrix},\quad
R=\begin{bmatrix}0&-1&0\\1&0&0\\0&0&1\end{bmatrix},\quad
S=\begin{bmatrix}2&0&0\\0&1&0\\0&0&1\end{bmatrix}.
$$

First multiply the matrices in the required order:

$$
RT=\begin{bmatrix}0&-1&2\\1&0&3\\0&0&1\end{bmatrix},\qquad
SRT=\begin{bmatrix}0&-2&4\\1&0&3\\0&0&1\end{bmatrix}.
$$

Then apply the composite:

$$
\mathbf p'=\begin{bmatrix}0&-2&4\\1&0&3\\0&0&1\end{bmatrix}
\begin{bmatrix}2\\3\\1\end{bmatrix}
=\begin{bmatrix}-2\\5\\1\end{bmatrix}.
$$

Thus $P'=(-2,5)$. Independently check the sequence: $(2,3)\to(5,1)\to(-1,5)\to(-2,5)$. Reversing the matrix order changes the answer because these operations generally do not commute.

### 8.3 Aligning an image to a reference

**Image registration** aligns images of the same scene to a common coordinate system. The procedure is to choose corresponding features/points in the reference and the other image, estimate a suitable transformation from those correspondences, transform the moving image’s coordinates, and resample intensities on the reference grid. Finally, check that corresponding structures overlap. Translation, rotation and scaling may suffice for an appropriate image pair. The reference defines the target coordinate system; intensity enhancement alone cannot correct a geometric misalignment. These steps form the answer to IPCV Q2(a).

## 9. Digital video, processing and analytics

### 9.1 A video as a spatiotemporal signal

A **digital video** is an ordered sequence of image frames representing changes in objects, the background, or both. Each frame is a 2D image. Once time is included, the video is a **3D spatiotemporal signal**, $V(x,y,t)$, where $x,y$ identify a position and $t$ identifies a time/frame. This does not mean the video automatically contains 3D scene geometry.

**Worked sample-count example:** a $1920\times1080$ video with 100 frames has array dimensions $1920\times1080\times100$, or **207360000 spatial–temporal sample locations** for one value per location. These are samples, not bytes; storage additionally depends on channels, bit depth and compression. Draw several 2D frames stacked along a time axis to explain the representation.

### 9.2 Processing stages and operations

The video workflow begins with **acquisition**, then **sampling** into discrete spatial/temporal measurements and **quantization** into intensity levels. **Compression** reduces data for **storage/transmission**, and **display/playback** reconstructs a visible sequence. Each stage has a distinct role: sampling decides where/when measurements exist; quantization decides which intensity values are representable.

**Spatial processing** operates within frames, such as filtering, enhancement and edge detection. **Temporal processing** compares frames over time, such as frame differencing or motion estimation. **Motion analysis** detects or tracks moving objects. Video enhancement improves frame quality; video segmentation separates regions/foreground; video compression reduces redundancy within a frame (**intra-frame**) or between frames (**inter-frame**). Applications include surveillance, traffic monitoring and sports analysis.

### 9.3 Video processing vs video analytics

Video processing mainly modifies the signal; video analytics extracts information about what is happening. Processing may therefore supply cleaner frames to analytics, but a processed image and an interpreted event are different outputs.

| Aspect | Video processing | Video analytics |
| --- | --- | --- |
| Main purpose | Improve, transform or manipulate video. | Extract information and support decisions. |
| Focus | Pixels and frames. | Objects, events and behaviour. |
| Operations | Filtering, enhancement, compression, resizing, stabilization, frame extraction. | Detection, tracking, classification, activity recognition. |
| Output | Modified video or frames. | Counts, events, alerts, statistics or decisions. |
| AI requirement | Not necessarily AI-based. | Frequently uses AI/ML/DL. |
| Example | Remove noise or resize 4K video to HD. | Detect a restricted-area entry or count vehicles. |

### 9.4 Visual and intelligent surveillance

**Object detection** locates objects in a frame; **object tracking** maintains their association over time. A basic visual-surveillance system combines both, so it can use a sequence rather than unrelated individual detections.

```
CCTV sequence → object detection + object tracking → surveillance information

Video capture → preprocessing/features → spatiotemporal modelling
              → anomaly/intrusion detection → alerts → monitoring/response
```

In the intelligent-surveillance pipeline, preprocessing reduces nuisance variation and extracts useful features. Spatiotemporal modelling uses appearance and motion across a sequence to describe behaviour. Detection identifies an intrusion or departure from expected behaviour; alert generation communicates it, and monitoring/response uses the result. For a theory answer, connect these functions instead of merely writing that surveillance “uses AI.”

## 10. Object detection and tracking methods

### 10.1 Object detection by background modelling

Background modelling learns what the scene normally looks like, typically from a static camera, and detects regions that differ from that model. Let $I_t(x,y)$ be the current frame and $B_t(x,y)$ the background estimate. The difference image and foreground mask are

$$
D_t(x,y)=|I_t(x,y)-B_t(x,y)|,\qquad
M_t(x,y)=\begin{cases}1,&D_t(x,y)>T,\\0,&\text{otherwise}.\end{cases}
$$

$T$ is a difference threshold and $M_t$ is a binary foreground mask. Clean the mask to remove noise/fill holes, then localize objects, for example with bounding boxes, area and centroid. Erosion/dilation, opening/closing and hole filling clean the mask; connected-component processing groups foreground pixels into regions.

A simple background update is the **running average**:

$$
B_t=(1-\alpha)B_{t-1}+\alpha I_t,\qquad0<\alpha<1.
$$

$\alpha$ is the learning rate: a small value retains more of the previous background and adapts slowly; a larger value responds faster to recent observations. The model can accommodate gradual changes rather than treating every small change as a new object.

**Worked pixel example:** if one pixel has $B_{t-1}=20$, $I_t=30$ and $\alpha=0.1$, then $B_t=0.9\times20+0.1\times30=21$. Using this updated estimate, $D_t=|30-21|=9$; with $T=8$, $M_t=1$. A question must make clear which background estimate is used. Follow its update/detection order.

**Diagram:** frames → background model → difference image → thresholded mask → cleaned mask → detected object. Possible background models include running average, GMM, kernel-density and codebook methods. GMM is described in §10.4.

### 10.2 Object detection by object modelling

Object modelling represents the **target’s appearance**, then searches for it in a new image. Collect views of the target under different conditions, align/normalize the examples, extract distinguishing features, and build a model/template. Extract features from candidate regions in the new image, compare them with the model, and localize sufficiently good matches.

Possible representations include colour histograms, HOG, SIFT/SURF/ORB and CNN features. Comparison measures include SSD, normalized cross-correlation, cosine similarity and chi-square distance. Together they form the **representation → search → match → localization** pipeline. For example, among similarity scores 0.12,0.18,0.86,0.21,0.15, the candidate scoring 0.86 is the strongest and passes the chosen threshold. **Caution:** larger similarity scores indicate better matches, whereas a raw difference measure such as SSD is better when smaller.

The key distinction is what is modelled: **background modelling asks what differs from the normal scene; object modelling asks what matches the target**. Draw the first with a background image and foreground mask, and the second with a target template and candidate boxes.

### 10.3 Frame differencing

Frame differencing detects change by comparing consecutive frames, without requiring a persistent background model. Compute the absolute difference and threshold it:

$$
D_t(x,y)=|I_t(x,y)-I_{t-1}(x,y)|,\qquad
B_t(x,y)=\begin{cases}1,&D_t(x,y)\ge T,\\0,&\text{otherwise}.\end{cases}
$$

Here $B_t$ denotes a **binary mask**; in background modelling, the same symbol denotes a background estimate. The threshold rules also differ: frame differencing here uses $\ge T$, while §10.1 uses $>T$. Follow the rule specified in the question, especially at equality. The absolute difference measures change in either direction; plain signed subtraction would miss some darkening changes under a positive threshold.

The workflow is frames → absolute difference → threshold → postprocessing → object detection. Opening/closing can remove small noise and improve the detected mask. The method is simple, fast and useful for a static background with moving objects, but camera noise or illumination changes can also create differences. It detects changed regions, which need not be a complete object silhouette.

**Worked frame example**, using $T=5$:

$$
I_{t-1}=\begin{bmatrix}10 & 10 & 20\\10 & 30 & 30\end{bmatrix},\qquad
I_t=\begin{bmatrix}12 & 18 & 15\\10 & 24 & 35\end{bmatrix}
$$

$$
D_t=\begin{bmatrix}2 & 8 & 5\\0 & 6 & 5\end{bmatrix},\qquad
B_t=\begin{bmatrix}0 & 1 & 1\\0 & 1 & 1\end{bmatrix}
$$

For example $|24-30|=6$ is foreground, and a difference exactly 5 is foreground because the condition is $\ge5$.

### 10.4 Multi-frame intruder detection and GMM

**Two/three-frame illustration.** A moving object occupies different positions in successive frames. A difference of two frames can highlight both vacated and newly occupied regions. Combining change masks from successive pairs keeps common change evidence, but the illustrated three-frame result can still contain only part of the object. The five-frame scheme adds further mask combinations to recover more of the target-frame region.

**Five-frame procedure.** Take $I_{t-2}, I_{t-1}, I_t, I_{t+1}, I_{t+2}$. Let $G_{a,b}$ denote the binary change mask from a neighbouring frame pair. Combine the masks as follows: $\cdot$ denotes intersection/AND and $+$ denotes union/OR. These operations combine binary masks, not unbounded grayscale sums.

For $j=t-1,t,t+1$:

$$
\begin{aligned}
GA_j&=G_{j-1,j}\cdot G_{j,j+1},\\
GO_j&=G_{j-1,j}+G_{j,j+1},\\
GD_j&=|GO_j-GA_j|.
\end{aligned}
$$

Then combine the neighbouring masks:

$$
\begin{aligned}
GE_{t-1}&=GD_{t-1}\cdot GD_t,\\
GE_{t+1}&=GD_t\cdot GD_{t+1},\\
GM_{t-1}&=G_{t-1,t}+GE_{t-1},\\
GM_{t+1}&=G_{t,t+1}+GE_{t+1},\\
GS_{t-1}&=|GM_{t-1}-G_{t-1,t}|,\\
GS_{t+1}&=|GM_{t+1}-G_{t,t+1}|,\\
O_t&=GS_{t-1}+GA_t+GS_{t+1}.
\end{aligned}
$$

$O_t$ is the resulting change-detection mask for frame $I_t$. The procedure combines pairwise overlap, difference and recovered side regions into a target-frame mask. **Diagram to remember:** five frames → four pair masks $G$ → $GA/GO$ → $GD$ → $GE$ → $GM/GS$ → $O_t$. **Caution:** this procedure assumes that the initial binary pair masks are available. It does not specify how to initialize their differences or thresholds, so those rules must be given before calculating numerical masks.

**Gaussian Mixture Model (GMM).** A single pixel may legitimately take several background appearances over time, for example because underwater vegetation moves or illumination varies. One fixed background value is then inadequate. Model the pixel with **multiple adaptive Gaussian distributions** so that several recurring appearances can belong to the background. Values that do not fit the background distributions are classified as foreground. Draw several Gaussian humps for a pixel’s possible background values; explain why multiple modes are useful.

### 10.5 Wronskian Change Detection Model

The Wronskian Change Detection Model is a **vector image model** for detecting changes between frames. Instead of representing a pixel using only its own intensity, include its **region of support**: the illustrated $3\times3$ neighbourhood is arranged as a nine-component vector. The model thus includes **spatio-contextual** information, meaning information from the pixel’s surrounding region.

Let $\vec F_t(x,y)_i$ denote component $i$ of the support vector at time $t$ and $n$ its number of components. Defining the component ratio $q_i=\vec F_t(x,y)_i/\vec F_{t-1}(x,y)_i$, the change measure is written as

$$
|W|=\frac1n\sum_{i=1}^{n}(q_i^2-q_i).
$$

If corresponding nonzero components are unchanged, each ratio is 1 and contributes zero. **Caution:** despite the notation $|W|$, the right-hand side can be negative. A ratio is undefined when its denominator is zero, and the expression alone specifies neither a zero-denominator rule nor a decision threshold. Do not assume such rules when applying it. Understand the model through its purpose, support-vector diagram and formula.

**Diagram:** mark a centre pixel → draw its $3\times3$ support → list those nine pixel values as a column vector → compare vectors from two frames. Keep the ordering identical across frames.

### 10.6 Crowd clustering and particle-filter tracking

**Clustering and reassignment.** Grouping blobs with related features into clusters helps distinguish candidate regions before tracking/reassignment. For example, 13 detected blobs can be grouped by a GMM with $k=3$. **Example only (not for memorization):** $C_1=\{F_5,F_7\}$, $C_2=\{F_1,F_3,F_8,F_{10}\}$, and $C_3=\{F_2,F_4,F_6,F_9,F_{11},F_{12},F_{13}\}$. Draw three groups with cluster density increasing down the diagram. Remember the grouping/reassignment concept rather than the exact feature memberships.

**Particle filter framework.** Tracking maintains a description of an object’s motion over time. A motion state can be written as

$$
S_{t-1}=\{x_c,y_c,v_x,v_y,a_x,a_y\},
$$

where $x_c,y_c$ locate the object centre, $v_x,v_y$ describe velocity and $a_x,a_y$ acceleration. A particle representation keeps multiple candidate states/locations to represent uncertainty, rather than assuming one exact position. The diagram shows particles and a window at $t-1$, followed by candidate windows/centres at $t$. Explain how the location and motion description help follow an object into the next frame. Draw the two time instants and candidate windows to show the motion and uncertainty.

## 11. Detection architecture and CNN workflow

### 11.1 Backbone, neck and detection head

A general deep-learning detection architecture separates three jobs. The **backbone** extracts hierarchical feature maps from the image, ranging from local visual patterns to richer object-related representations. The **neck** combines features across levels/scales so that objects of different sizes can be detected while retaining useful spatial information. The **detection head** uses these features to predict an object’s class, bounding box and objectness/confidence.

```
Input image → backbone (feature extraction) → neck (feature aggregation)
            → detection head (class + box + objectness)
            → confidence filtering and NMS → final detections
```

The input is resized before processing. A bounding box is represented as $(x,y,w,h)$: position and size. The final stage uses a confidence threshold and **Non-Maximum Suppression (NMS)** to suppress redundant competing detections and produce the final boxes.

| Neck | Full form and role |
| --- | --- |
| **FPN** | **Feature Pyramid Network**: combines higher-level semantic features with lower-level spatial features for detecting objects at different scales. |
| **PAN** | **Path Aggregation Network**: adds a bottom-up path to improve the flow of localization/low-level information. |
| **BiFPN** | **Bidirectional Feature Pyramid Network**: repeats top-down and bottom-up fusion with learnable weights for efficient multi-scale aggregation. |

**YOLO and Faster R-CNN** perform object detection; **DeepSORT** supports tracking; **3D CNNs/LSTM/Transformers** model temporal information; and **autoencoders/GANs** support anomaly detection.

### 11.2 Working of a CNN model

A **Convolutional Neural Network (CNN)** learns hierarchical features from images and uses them for a task such as classification. A typical classification workflow passes a hand image through convolution, activation, pooling, flattening, fully connected layers and softmax. The steps explain how an image array becomes a class prediction:

1. **Input:** represent the image numerically; the illustration uses $128\times128\times1$, a single-channel input.
2. **Convolution:** slide small filters over local regions and multiply/add values to form feature maps. Different filters capture patterns such as edges, lines and textures.
3. **ReLU activation:** replace negative responses by zero using $\operatorname{ReLU}(z)=\max(0,z)$. This introduces nonlinearity so the network can represent more than a sequence of linear operations.
4. **Max pooling:** keep the largest value from each small region, reducing spatial size and computation while retaining strong responses. For example:
    
    $$
    \begin{bmatrix}4&7\\1&3\end{bmatrix}\to7.
    $$
    
5. **Flatten:** arrange the remaining feature-map entries into a one-dimensional feature vector.
6. **Fully connected layers:** combine the extracted features to produce class-related scores.
7. **Softmax output:** convert scores into class probabilities whose sum is 1; select the largest as the predicted class. For example, the output may assign 0.92 for “fractured” and 0.08 for “non-fractured.”

**Diagram to remember:** image → convolutional feature maps → ReLU → pooled maps → flattened vector → fully connected layers → class probabilities. The learned hierarchy is **edges → textures → parts → objects → classification**. Convolution extracts local information, pooling reduces its spatial size, and the final layers combine it for the decision; these operations should not be described as interchangeable.