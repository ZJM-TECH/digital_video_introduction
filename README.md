[![license](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)](https://img.shields.io/badge/license-BSD--3--Clause-blue.svg)

# Intro

A gentle introduction to video technology, although it's aimed for software developers / engineering, we want to make it easy **for anyone to learn**. This idea was born during a mini workshop for newcomers to video technology.

The goal is to introduce some digital video subjects with a **simple text, visual element and practical examples**, where is possible, and make this knowledge available everywhere. Please, feel free to correct, suggest and improve it.

There will be **hands-on** sections which requires you to have **docker installed** and this repo cloned.

```bash
git clone https://github.com/leandromoreira/digital_video_introduction.git
cd digital_video_introduction
./setup.sh
```
> **WARNING**: when you see a `./s/ffmpeg` or `./s/mediainfo` command, it means we're running a **containerized version** of that program, which already includes all the needed requirements.

All the **hands-on should be performed from the folder you cloned** this repository, for the **jupyter examples** you must start the server `./s/start_jupyter.sh` and copy the url and use it on your browser.

# Index

- [Intro](#intro)
- [Index](#index)
- [Basic terminology](#basic-terminology)
      - [Other ways to encode a color image](#other-ways-to-encode-a-color-image)
      - [Hands-on: play around with image and color](#hands-on-play-around-with-image-and-color)
      - [DVD is DAR 4:3](#dvd-is-dar-43)
      - [Hands-on: Check video properties](#hands-on-check-video-properties)
- [Redundancy removal](#redundancy-removal)
  * [Colors, Luminance and our eyes](#colors-luminance-and-our-eyes)
    + [Color model](#color-model)
    + [Converting between YCbCr and RGB](#converting-between-ycbcr-and-rgb)
    + [Chroma subsampling](#chroma-subsampling)
    + [Hands-on: Check YCbCr histogram](#hands-on-check-ycbcr-histogram)
  * [Frame types](#frame-types)
    + [I Frame (intra, keyframe)](#i-frame-intra-keyframe)
    + [P Frame (predicted)](#p-frame-predicted)
    + [B Frame (bi-predictive)](#b-frame-bi-predictive)
  * [Temporal redundancy (inter prediction)](#temporal-redundancy-inter-prediction)
      - [Hands-on: See the motion vectors](#hands-on-see-the-motion-vectors)
  * [Spatial redundancy (intra prediction)](#spatial-redundancy-intra-prediction)
      - [Hands-on: Check intra predictions](#hands-on-check-intra-predictions)
- [How does a video codec work?](#how-does-a-video-codec-work)
  * [What? Why? How?](#what-why-how)
  * [History](#history)
      - [The born of AV1](#the-born-of-av1)
  * [A generic codec](#a-generic-codec)
  * [1st step - picture partitioning](#1st-step---picture-partitioning)
    + [Hands-on: Check partitions](#hands-on-check-partitions)
  * [2nd step - predictions](#2nd-step---predictions)
  * [3rd step - transform](#3rd-step---transform)
  * [4th step - quantization](#4th-step---quantization)
  * [5th step - entropy coding](#5th-step---entropy-coding)
    + [Delta coding:](#delta-coding)
    + [VLC coding:](#vlc-coding)
    + [Arithmetic coding:](#arithmetic-coding)
    + [Hands-on: CABAC vs CAVLC](#hands-on-cabac-vs-cavlc)
  * [6th step - bitstream format](#6th-step---bitstream-format)
    + [H264 bitstream](#h264-bitstream)
    + [Hands-on: Inspect the H264 bitstream](#hands-on-inspect-the-h264-bitstream)
  * [Review](#review)
  * [How does H265 can achieve better compression ratio than H264?](#how-does-h265-can-achieve-better-compression-ratio-than-h264)
- [Adaptive streaming](#adaptive-streaming)
  * [What? Why? How?](#what-why-how-1)
  * [HLS and Dash](#hls-and-dash)
  * [Building a bit rate ladder](#building-a-bit-rate-ladder)
- [Audio codec](#audio-codec)
- [How to use jupyter](#how-to-use-jupyter)
- [References](#references)

# Basic terminology

An **image** can be thought as a **2D matrix** and if we think about **colors**, we can extrapolate this idea seeing this image as a **3D matrix** where the **additional dimensions** are used to provide **color data**.

If we chose to represent these colors using the [primary colors (red, green and blue)](https://en.wikipedia.org/wiki/Primary_color), we then can define the tree planes: the first one **red**, the second **green** and the last the **blue** color.

![an image is a 3d matrix RGB](/i/image_3d_matrix_rgb.png "An image is a 3D matrix")

Each point in this matrix, we'll call it **a pixel** (picture element), will hold the **intensity** (usually a numeric value) of that given color. A **total red color** means 0 of green, 0 of blue and max of red, the **pink color** can be formed with (using 0 to 255 as the possible range) with **Red=255, Green=192 and Blue=203**.

> #### Other ways to encode a color image
> There are much more models to represent an image with colors. We could use a indexed palette where we'd spend only a byte for each pixel instead of 3, comparing it to RGB model. In this model instead of a 3D matrix we'd use a 2D matrix, saving memory but having much less color options.
>
> ![NES palette](/i/nes-color-palette.png "NES palette")

For instance, look at the picture down bellow, the first face is full colored, the rest is the red, green and blue (but in gray tones) planes.

![RGB channels intensity](/i/rgb_channels_intensity.png "RGB channels intensity")

We can see that the **red color** will be the one that **contributes more** (the brightest parts in the second face) to the final color while the **blue color** contribution can be mostly **only seen in Mario's eyes** (last face) and part of his clothes, see how **all the planes contributes less** (darkest parts) to the **Mario's mustache**.


And each color intensity requires a certain amount of bits, this quantity is know as **bit depth**. Let's say we spend **8 bits** (accepting values from 0 to 255) per color (plane), therefore we have a **color depth** of **24 (8 * 3) bits** and we can also infer that we could use 2 to the power of 24 different colors.

> **It's great** to learn [how an image is captured from the world to the bits](http://www.cambridgeincolour.com/tutorials/camera-sensors.htm).

Another property of an image is the **resolution**, which is the number of pixels in one dimension. It is often presented as width × height, for example the **4×4** image bellow.

![image resolution](/i/resolution.png "image resolution")

> #### Hands-on: play around with image and color
> You can [play around with image and colors](/image_as_3d_array.ipynb) using [jupyter](#how-to-use-jupyter) (python, numpy, matplotlib and etc).
>
> You can also learn [how image filters (edge detection, sharpen, blur...) work](/filters_are_easy.ipynb).

Another property we can see while working with images or video is the **aspect ratio** which is simple describes the proportional relationship between width and height of an image or pixel.

When people says this movie or picture is **16x9** they usually are referring to the **Display Aspect Ratio (DAR)** and we also can have different shapes of a pixel, we call this **Pixel Aspect Ratio (PAR)**.

![display aspect ratio](/i/DAR.png "display aspect ratio")

![pixel aspect ratio](/i/PAR.png "pixel aspect ratio")

> #### DVD is DAR 4:3
> Although the real resolution of a DVD is 704x480 it still keeps a 4:3 aspect ratio because it has a PAR of 10:11 (704x10/480x11)

Finally we can define a **video** as a **succession of *n* frames** in **time** which can be seen as another dimension, *n* is the frame rate or frames per second (FPS).

![video](/i/video.png "video")

The amount of bits per second needed to show a video is its **bit rate**. For example, a video with 30 frames per second, 24 bits per pixel, resolution of 480x240 will need **82,944,000 bits per second** or 82.944 Mbps (30x480x240x24) if we don't employ any kind of compression.

When the **bit rate** is nearly constant it's called constant bit rate (**CBR**) but it also can vary then called variable bit rate (**VBR**).

> This graph shows a constrained VBR which doesn't spend too many bits while the frame is black.
>
> ![constrained vbr](/i/vbr.png "constrained vbr")

In the early days engineering come up with a technique for doubling the perceived frame rate of a video display **without consuming extra bandwidth**, this technique is known as **interlaced video**. It basically sends half of the screen in 1 "frame" and the next "frame" they send the other half.

Today screens render mostly using **progressive scan technique**, progressive is a way of displaying, storing, or transmitting moving images in which all the lines of each frame are drawn in sequence.

![interlaced vs progressive](/i/interlaced_vs_progressive.png "interlaced vs progressive")

Now we have an idea about what is an **image**, how its **colors** are arranged, how many **bits per second** do we spend to show a video, if it's constant (CBR)  or variable (VBR), with a given **resolution** using a given **frame rate** and many other terms such as interlaced, PAR and others.

> #### Hands-on: Check video properties
> You can [check most of the  explained properties with ffmpeg or mediainfo.](https://github.com/leandromoreira/introduction_video_technology/blob/master/enconding_pratical_examples.md#inspect-stream)

# Redundancy removal

We learned that is not feasible to use video without any compression, **a single one hour video** at 720p resolution with 30fps would **require 2.38Tb<sup>*</sup>**. We need to find a way to compress the video, **using solely lossless data compression algorithms**, like DEFLATE (used in PKZIP, Gzip, and PNG), **won't help** as much as we need.

> <sup>*</sup> We found this number by multiplying 1280 x 720 x 24 x 30 x 3600 (width, height, bits per pixel, fps and time in seconds)

We can **exploit how our vision works**, we're better to distinguish brightness than colors, the **repetitions in time**, a video contains a lot of images with few changes, and the **repetitions within image**, each frame also contains many areas using the same or similar color.

## Colors, Luminance and our eyes

Our eyes are [more sensible to brightness than colors](http://vanseodesign.com/web-design/color-luminance/), you can test it for yourself, look at this picture.

![luminance vs color](/i/luminance_vs_color.png "luminance vs color")

If you are unable to see that the colors of the **squares A and B are identical** in the right side, that's fine, it's our brain playing tricks on us to **pay more attention to light and dark than color**. There is a connector, with the same color, in the left side so we (our brain) can easily spot that in fact they're the same color.

Once we know that we're more sensible to **luma** (the brightness in an image) we can try to exploit it.

### Color model

We first learned [how to color images](/digital_video_introduction#basic-terminology) work using **RGB model** but there are others models. In fact, there is a model that separates luma (brightness) from  chrominance (colors) and it is known as **YCbCr**<sup>*</sup>.

> <sup>*</sup> there are more models which does the same separation.

This color model uses **Y** to represent the brightness and plus two color channels **Cb** (chroma blue) and **Cr** (chrome red). The [YCbCr](https://en.wikipedia.org/wiki/YCbCr) can be derived from RGB and it also can be converted back to RGB. Using this model we can produced full colored images as we can see down bellow.

![ycbcr example](/i/ycbcr.png "ycbcr example")

### Converting between YCbCr and RGB

Some may argue, how can we produce all the **colors without using the green**?

To answer this question we'll walk through a conversion from RGB to YCbCr. We'll use the coefficients from the **[standard BT.601](https://en.wikipedia.org/wiki/Rec._601)** that was recommended by the **[group ITU-R<sup>*</sup>](https://en.wikipedia.org/wiki/ITU-R)** . The first step is to **calculate the luma**, we'll use the constants suggested by ITU and replace the RGB values.

```
Y = 0.299R + 0.587G + 0.114B
```

Once we had the luma, we can **split the colors** (chroma blue and red):

```
Cb = 0.564(B - Y)
Cr = 0.713(R - Y)
```

And we can also **convert it back** and even getting the **green by using YCbCr**.

```
R = Y + 1.402Cr
B = Y + 1.772Cb
G = Y - 0.344Cb - 0.714Cr
```

> <sup>*</sup> groups and standards are common in digital video, they usually defines what are the standards, for instance [what is 4K? what frame rate should we use? resolution? color model?](https://en.wikipedia.org/wiki/Rec._2020)

Generally, the **displays** (monitors, TVs, screens and etc) shows **only the RGB model**, see some of them in a zoomed level, they organize the RGB channels in different manners:

![pixel geometry](/i/pixel_geometry.jpg "pixel geometry")

### Chroma subsampling

Once we were able to separate luma from chroma, we can take advantage of the human visual system that is more capable to see luma than chroma. **Chroma subsampling** is the technique of encoding images using **less resolution for chroma than for luma**.



![ycbcr subsampling resolutions](/i/ycbcr_subsampling_resolution.png "ycbcr subsampling resolutions")


How much should we reduce from the chroma resolution?! it turns out that there is already some schemes that describes how to handle resolution and the merge (`final color = Y + Cb + Cr`).

These schemas are known as subsampling systems (or ratios), they are identified by: **4:4:4, 4:2:3, 4:2:1, 4:1:1, 4:2:0, 4:1:0 and 3:1:1**. And each one of them defines how much should we discard in the chroma resolution as well as how we should merge the three planes (Y, Cb, Cr).

You can see the same image encoded by the main chroma subsampling types, the first row of images are the final YCbCr while the last row of images shows the chroma resolution.

![chroma subsampling examples](/i/chroma_subsampling_examples.jpg "chroma subsampling examples")

Previously we calculated that we needed [2.3Tb of storage to keep a video file with one hour at 720p resolution and 30fps](/digital_video_introduction#redundancy-removal), if we use **YCbCr 4:2:0** we can cut **this size in half (1.19Tb)**<sup>*</sup> but it is still far from the ideal.

> <sup>*</sup> we found this value by multiplying width, height, bits per pixel and fps, before we needed 24 bits now we only need 12.

<br/>

> ### Hands-on: Check YCbCr histogram
> You can [check the YCbCr histogram with ffmpeg.](/enconding_pratical_examples.md#generates-yuv-histogram) This scene has more blue contribution which is showed by the [histogram](https://en.wikipedia.org/wiki/Histogram).
>
> ![ycbcr color histogram](/i/yuv_histogram.png "ycbcr color histogram")

## Frame types

### I Frame (intra, keyframe)

### P Frame (predicted)

### B Frame (bi-predictive)

## Temporal redundancy (inter prediction)

> #### Hands-on: See the motion vectors
> We can [generate a video with the inter prediction (motion vectors)  with ffmpeg.](/enconding_pratical_examples.md#generate-debug-video)
>
> ![inter prediction (motion vectors) with ffmpeg](/i/motion_vectors_ffmpeg.png "inter prediction (motion vectors) with ffmpeg")
>
> Or we can use the [Intel Video Pro Analyzer](https://software.intel.com/en-us/intel-video-pro-analyzer) (which is paid but there is a free trial version which limits you to only the first 10 frames).
>
> ![inter prediction intel video pro analyzer](/i/inter_prediction_intel_video_pro_analyzer.png "inter prediction intel video pro analyzer")

## Spatial redundancy (intra prediction)

> #### Hands-on: Check intra predictions
> You can [generate a video with macro blocks and their predictions with ffmpeg.](/enconding_pratical_examples.md#generate-debug-video) Please check the ffmpeg documentation to understand the [meaning of each block color](https://trac.ffmpeg.org/wiki/Debug/MacroblocksAndMotionVectors).
>
> ![intra prediction (macro blocks) with ffmpeg](/i/macro_blocks_ffmpeg.png "inter prediction (motion vectors) with ffmpeg")
>
> Or we can use the [Intel Video Pro Analyzer](https://software.intel.com/en-us/intel-video-pro-analyzer) (which is paid but there is a free trial version which limits you to only the first 10 frames).
>
> ![intra prediction intel video pro analyzer](/i/intra_prediction_intel_video_pro_analyzer.png "intra prediction intel video pro analyzer")

# How does a video codec work?

## What? Why? How?

**What?** It's a software / hardware that compresses or decompresses digital video. **Why?** Market and society demands higher quality videos with limited bandwidth or storage, remember when we [calculated the needed bandwidth](#basic-videoimage-terminology) for a 30 frames per second, 24 bits per pixel, resolution of 480x240 video? It was **82.944 Mbps** with none compression applied. It's the only way to delivery HD/FullHD/4K in TVs and Internet. **How?** We'll take brief look a the major techniques here.

> **CODEC vs Container**
>
> One common mistake that beginners often do is to confuse digital video CODEC and [digital video container](https://en.wikipedia.org/wiki/Digital_container_format). We can think of **containers** as a wrapper format which contains metadata of the video and possible audio too, and the **compressed video is the codec** can be seen as its payload.
>
> Usually the extension of a video file defines its video container. For instance, the file `video.mp4` is probably a **[MPEG-4 Part 14](https://en.wikipedia.org/wiki/MPEG-4_Part_14)** container and a file named `video.mkv` it's probably a **[matroska](https://en.wikipedia.org/wiki/Matroska)**. To be completly sure about the codec and container format we can use [ffmpeg or mediainfo](/enconding_pratical_examples.md#inspect-stream).

## History

Before we jump in the inner works of a generic codec, let's look back to understand a little better about some old video codecs.

The video codec [H261](https://en.wikipedia.org/wiki/H.261)  was born in 1990 (technically 1988), it was designed to work with **data rates of 64 kbit/s**. It already uses ideas such as chroma subsampling, macro block and etc. In the year of 1995 the **H263** video codec standard was published but it continued to be extended until 2001.


In 2003 the first version of **H.264/AVC** was completed, in the same year, a company called **TrueMotion** released their video codec as a **royalty free** lossy video compression called **VP3**. In 2008, **Google bought** this company, in the same year they released the **VP8**. In December of 2012, Google released the **VP9** and it's  **supported by roughly ¾ of the browser market** (mobile included).

 **[AV1](https://en.wikipedia.org/wiki/AOMedia_Video_1)** is a new video codec, **royalty-free**, open source being designed by the [Alliance for Open Media (AOMedia)](http://aomedia.org/) which is composed by the **companies: Google, Mozilla, Microsoft, Amazon, Netflix, AMD, ARM, NVidia, Intel, Cisco** among others. The **first version** 0.1.0 of the reference codec was **published on April 7, 2016**.

![codec history timeline](/i/codec_history_timeline.png "codec history timeline")

> #### The born of AV1
>
> Early 2015, Google was working on [VP10](https://en.wikipedia.org/wiki/VP9#Successor:_from_VP10_to_AV1), Xiph (Mozilla) was working on [Daala](https://xiph.org/daala/) and Cisco open-sourced its royalty-free video codec called [Thor](https://tools.ietf.org/html/draft-fuldseth-netvc-thor-03).
>
> Then MPEG LA first announces annual caps for HEVC (H265) 8 times higher than H264 but soon after it releases new rules:
> * **no annual cap**,
> * **content fee** (0.5% of revenue) and
> * **per-unit fees about 10 times higher than h264**.
>
> Then the [alliance for open media](http://aomedia.org/about-us/) was created by companies from hardware manufacturer (Intel, AMD, ARM , Nvidia, Cisco), content delivery (Google, Netflix, Amazon), browser maintainers (Google, Mozilla) and many more interested companies.
>
> The companies have a common goal, a royalty-free video codec and then AV1 was born with a much [simpler patent license](http://aomedia.org/license/patent/). **Timothy B. Terriberry** did an awesome presentation, which is the source of this section, about the [AV1 conception, license model and its current state](https://www.youtube.com/watch?v=lzPaldsmJbk).
>
> You'll be surprised to know that you can **analyze the AV1 codec through your browser**, go to: http://aomanalyzer.org/
>
> ![av1 browser analyzer](/i/av1_browser_analyzer.png "av1 browser analyzer")
>
> PS: If you want to learn more about the history of the codecs you must learn the basics behind [video compression patents](https://www.vcodex.com/video-compression-patents/).

## A generic codec

We're going to introduce the **main mechanics behind a generic video codec** but most of these concepts are useful and used in modern codecs such as VP9, AV1 and HEVC. Be sure to understand that we're going to simplify things a LOT. Sometimes we'll use a real example (mostly H264) to demonstrate a technique.

## 1st step - picture partitioning

The first step is to **divide the frame** into several **partitions, sub-partitions** and beyond.

![picture partitioning](/i/picture_partitioning.png "picture partitioning")

**But why?** There are many reasons, for instance, when we split the picture we can work the predictions more precisely, using small partitions for the small moving parts while use bigger partitions to static background.

Usually, the CODECs **organize these partitions** into slices (or tiles), macro (or coding tree units) and many sub partitions. The max size of these partitions varies, HEVC sets 64x64 while AVC uses 16x16 but the sub-partitions can reach sizes of 4x4.

Remember that we learned how **frames are typed**?! Well, you can **apply those ideas to blocks** too, therefore we can have I-Slice, B-Slice, I-Macroblock and etc.

> ### Hands-on: Check partitions
> We can also use the [Intel Video Pro Analyzer](https://software.intel.com/en-us/intel-video-pro-analyzer) (which is paid but there is a free trial version which limits you to only the first 10 frames). Here's a [VP9 partitions](/enconding_pratical_examples.md#transcoding) analyzed.
>
> ![VP9 partitions view intel video pro analyzer ](/i/paritions_view_intel_video_pro_analyzer.png "VP9 partitions view intel video pro analyzer")

## 2nd step - predictions

## 3rd step - transform

## 4th step - quantization

## 5th step - entropy coding

After we quantized the data (image blocks/slices/frames) we still can compress it in a lossless way. There are many ways (algorithms) to compress data. We're going to briefly experience some of them, for a deeper understanding you can read the amazing book [Understanding Compression: Data Compression for Modern Developers](https://www.amazon.com/Understanding-Compression-Data-Modern-Developers/dp/1491961538/).

### Delta coding:

I love the simplicity of this method (it's amazing), let's say we need to compress the following numbers `[0,1,2,3,4,5,6,7]` and if we just decrease the current number to its previous and we'll get the `[0,1,1,1,1,1,1,1]` array which is highly compressible.

Both encoder and decoder **must know** the rule of delta formation.

### VLC coding:

Let's suppose we have a stream with the symbols: **a**, **e**, **r** and **t** and their probability (from 0 to 1) is represented by this table.

|             | a   | e   | r    | t   |
|-------------|-----|-----|------|-----|
| probability | 0.3 | 0.3 | 0.2 |  0.2 |

We can assign unique binary codes (preferable small) to the most probable and bigger codes to the least probable ones.

|             | a   | e   | r    | t   |
|-------------|-----|-----|------|-----|
| probability | 0.3 | 0.3 | 0.2 | 0.2 |
| binary code | 0 | 10 | 110 | 1110 |

Let's compress the stream **eat**, assuming we would spend 8 bits for each symbol, we would spend **24 bits** without any compression. But in case we replace each symbol for its code we can save space.

The first step is to encode the symbol **e** which is `10` and the second symbol is **a** which is added (not in the mathematical way) `[10][0]` and finally the third symbol **t** which makes our final compressed bitstream to be `[10][0][1110]` or `1001110` which only requires **7 bits** (3.4 times less space than the original).

Notice that each code must be a unique prefixed code [Huffman can help you to find these numbers](https://en.wikipedia.org/wiki/Huffman_coding). Though it has some issues there are [video codecs that still offers](https://en.wikipedia.org/wiki/Context-adaptive_variable-length_coding) this method and it's the  algorithm for many application which requires compression.

Both encoder and decoder **must know** the symbol table with its code therefore you need to send the table too.

### Arithmetic coding:

Let's suppose we have a stream with the symbols: **a**, **e**, **r**, **s** and **t** and their probability is represented by this table.

|             | a   | e   | r    | s    | t   |
|-------------|-----|-----|------|------|-----|
| probability | 0.3 | 0.3 | 0.15 | 0.05 | 0.2 |

With this table in mind we can build ranges containing all the possible symbols sorted by the most frequents.

![initial arithmetic range](/i/range.png "initial arithmetic range")

Now let's encode the stream **eat**, we pick the first symbol **e** which is located within the subrange **0.3 to 0.6** (but not included) and we take this subrange and split it again using the same proportions used before but within this new range.

![second sub range](/i/second_subrange.png "second sub range")

Let's continue to encode our stream **eat**, now we take the second symbol **a** which is within the new subrange **0.3 to 0.39** and then we take our last symbol **t** and we do the same process again and we get the last subrange **0.354 to 0.372**.

![final arithmetic range](/i/arithimetic_range.png "final arithmetic range")

We just need to pick a number within the last subrange **0.354 to 0.372**, let's chose **0.36** but we could chose any number within this subrange. With **only** this number we'll be able to recovery our original stream **eat**. If you think about it, it's like if we were drawing a line within ranges of ranges to encode our stream.

![final range traverse](/i/range_show.png "final range traverse")

The **reverse process** (A.K.A. decoding) is equally easy, with our number **0.36** and our original range we can run the same process but now using this number to reveal the stream encoded behind this number.

With the first range we notice that our number fits at the **e** slice therefore it's our first symbol, now we split this subrange again, doing the same process as before, and we'll notice that **0.36** fits the symbol **a** and after we repeat the process we came to the last symbol **t** (forming our original encoded stream *eat*).

Both encoder and decoder **must know** the symbol probability table, therefore you need to send the table.

Pretty neat isn't? People are damm smart to come up with such solution, some [video codec uses](https://en.wikipedia.org/wiki/Context-adaptive_binary_arithmetic_coding) (or at least offers as an option) this technique.

The idea is to lossless compress the quantized bitstream, for sure this article is missing tons of details, reasons, trade-offs and etc. But [you should learn more](https://www.amazon.com/Understanding-Compression-Data-Modern-Developers/dp/1491961538/) as a developer. Newer codecs are trying to use different [entropy coding algorithms like ANS.](https://en.wikipedia.org/wiki/Asymmetric_Numeral_Systems)

> ### Hands-on: CABAC vs CAVLC
> You can [generate two streams, one with CABAC and other with CAVLC](https://github.com/leandromoreira/introduction_video_technology/blob/master/enconding_pratical_examples.md#cabac-vs-cavlc) and **compare the time** it took to generate each of them as well as **the final size**.

## 6th step - bitstream format

After we did all these steps we need to **pack the compressed frames and context to these steps**. We need to explicitly inform to the decoder about **the decisions taken by the encoder**, things like: bit depth, color space, resolution, predictions info (motion vectors, direction of prediction), profile, level, frame rate, frame type, frame number and many more.

We're going to study, superficially, the H264 bitstream. Our first step is to [generate a minimal  H264 <sup>*</sup> bitstream](/enconding_pratical_examples.md#generate-a-single-frame-h264-bitstream), we can do that using our own repository and [ffmpeg](http://ffmpeg.org/).

```
./s/ffmpeg -i /files/i/minimal.png -pix_fmt yuv420p /files/v/minimal_yuv420.h264
```

> <sup>*</sup> ffmpeg adds, by default, all the encoding parameter as a **SEI NAL**, soon we'll define what is a NAL.

This command will generate a raw h264 bitstream with a **single frame**, 64x64, with color space yuv420 and using the following image as the frame.

> ![used frame to generate minimal h264 bitstream](/i/minimal.png "used frame to generate minimal h264 bitstream")

### H264 bitstream

The AVC (H264) standard defines that the information will be send in **macro frames** (in the network sense), called **[NAL](https://en.wikipedia.org/wiki/Network_Abstraction_Layer)** (Network Abstraction Layer). The main goal of the NAL is the provision of a "network-friendly" video representation, this standard must work on TVs (stream based), Internet (packet based) among others.

![NAL units H264](/i/nal_units.png "NAL units H264")

There is a **[synchronization marker](https://en.wikipedia.org/wiki/Frame_synchronization)** to define the boundaries among the NAL's units. Each synchronization marker holds a value of `0x00 0x00 0x01` except to the very first one which is `0x00 0x00 0x00 0x01`. If we run the **hexdump** on the generated h264 bitstream, we can identify at least three NALs in the beginning of the file.

![synchronization marker on NAL units](/i/minimal_yuv420_hex.png "synchronization marker on NAL units")

As we said before, the decoder needs to know not only the picture data but also the details of the video, frame, colors, used parameters and others. The **first byte** of each NAL defines its category and **type**.

| NAL type id  | Description  |
|---  |---|
| 0  |  Undefined |
| 1  |  Coded slice of a non-IDR picture |
| 2  |  Coded slice data partition A |
| 3  |  Coded slice data partition B |
| 4  |  Coded slice data partition C |
| 5  |  **IDR** Coded slice of an IDR picture |
| 6  |  **SEI** Supplemental enhancement information |
| 7  |  **SPS** Sequence parameter set |
| 8  |  **PPS** Picture parameter set |
| 9  |  Access unit delimiter |
| 10 |  End of sequence |
| 12 |  End of stream |
| ... |  ... |

Usually the first NAL of a bitstream is a **SPS**, this type of NAL is responsible to inform the general encoding variables like **profile**, **level**, **resolution** and others.

If we skip the first synchronization marker we can decode the **first byte** to know what **type of NAL** is the first one.

For instance the first byte after the synchronization marker is `01100111`, where the first bit (`0`) is to the field **forbidden_zero_bit**, the next 2 bits (`11`) tell us the field **nal_ref_idc** which indicates whether this NAL is a reference field or not and the rest 5 bits (`00111`) inform us the field **nal_unit_type**, in this case it's a **SPS** (7) NAL unit.

The second byte (`binary=01100100, hex=0x64, dec=100`) of a SPS NAL is the field **profile_idc** which shows the profile that the encoder has used, in this case we used  the **[constrained high profile](https://en.wikipedia.org/wiki/H.264/MPEG-4_AVC#Profiles)**, it's a high profile without support of B (bi-predictive) slices.

![SPS binary view](/i/minimal_yuv420_bin.png "SPS binary view")

When we read the H264 bitstream spec for a SPS NAL we'll find many values for **parameter name**, **category** and a **description**, for instance let's look at `pic_width_in_mbs_minus_1` and `pic_height_in_map_units_minus_1` fields.

| Parameter name  | Category  |  Description  |
|---  |---|---|
| pic_width_in_mbs_minus_1 |  0 | ue(v) |
| pic_height_in_map_units_minus_1 |  0 | ue(v) |

> **ue(v)**: unsigned integer [Exp-Golomb-coded](https://pythonhosted.org/bitstring/exp-golomb.html)

If we do some math with the value of these fields we will end up with the **resolution**. We can represent a `1920 x 1080` using a `pic_width_in_mbs_minus_1` with the value of `119 ( (119 + 1) * macroblock_size = 120 * 16 = 1920) `, again saving space, instead of encode `1920` we did it with `119`.

If we continue to examine our created video with a binary view (ex: `xxd -b -c 11 v/minimal_yuv420.h264`), we can skip to the last NAL which is the frame itself.

![h264 idr slice header](/i/slice_nal_idr_bin.png "h264 idr slice header")

We can see its first 6 bytes values: `01100101 10001000 10000100 00000000 00100001 11111111`. As we already know the first byte tell us about what type of NAL it is, in this case (`00101`) it's an **IDR Slice (5)** and we can further inspect it:

![h264 slice header spec](/i/slice_header.png "h264 slice header spec")

Using the spec info we can decode what type of slice (**slice_type**), frame number (**frame_num**) among others important fields.

In order to get the values of some fields (`ue(v), me(v), se(v) or te(v)`) we need to decode it using a special decoder called [Exponential-Golomb](https://pythonhosted.org/bitstring/exp-golomb.html), this method is **very efficient to encode variable values**, mostly when there are many default values.

> The values of **slice_type** and **frame_num** of this video are: 7 (I slice) and 0 (the first frame).

We can see the **bitstream as a protocol** and if you want or need to learn more about this bitstream please refer to the [ITU H264 spec.]( http://www.itu.int/rec/T-REC-H.264-201610-I) Here's a macro diagram which shows where the picture data (compressed YUV) resides.

![h264 bitstream macro diagram](/i/h264_bitstream_macro_diagram.png "h264 bitstream macro diagram")

We can explore others bitstreams like the [VP9 bitstream](https://storage.googleapis.com/downloads.webmproject.org/docs/vp9/vp9-bitstream-specification-v0.6-20160331-draft.pdf), [H265 (HEVC)](handle.itu.int/11.1002/1000/11885-en?locatt=format:pdf) or even our **new best friend** [**AV1** bitstream](https://medium.com/@mbebenita/av1-bitstream-analyzer-d25f1c27072b#.d5a89oxz8
), [do they all look similar? No](http://www.gpac-licensing.com/2016/07/12/vp9-av1-bitstream-format/), but once you learned one you can easily get the others.

> ### Hands-on: Inspect the H264 bitstream
> We can [generate a single frame video](https://github.com/leandromoreira/introduction_video_technology/blob/master/enconding_pratical_examples.md#generate-a-single-frame-video) and use  [mediainfo](https://en.wikipedia.org/wiki/MediaInfo) to inspect its H264 bitstream. In fact, you can even see the [source code that parses h264 (AVC)](https://github.com/MediaArea/MediaInfoLib/blob/master/Source/MediaInfo/Video/File_Avc.cpp) bitstream.
>
> ![mediainfo details h264 bitstream](/i/mediainfo_details_1.png "mediainfo details h264 bitstream")
>
> We can also use the [Intel Video Pro Analyzer](https://software.intel.com/en-us/intel-video-pro-analyzer) which is paid but there is a free trial version which limits you to only the first 10 frames but that's okay for learning purposes.
>
> ![intel video pro analyzer details h264 bitstream](/i/intel-video-pro-analyzer.png "intel video pro analyzer details h264 bitstream")

## How H265 can achieve better compression ratio than H264

[WIP]

# Adaptive streaming

## What? Why? How?
Creating multiple playlist thinking about mobile network
## HLS and Dash

## Building a bit rate ladder

We could create our bit rate options based on many

# Encoding parameters the whys

[WIP]

# Audio codec

[WIP]

# How to use jupyter

Make sure you have **docker installed** and just run `./s/start_jupyter.sh` and follow the instructions on the terminal.

# References

The richest content is here, where all the info we saw in this text was extracted, based or inspired by. You can deepen your knowledge with these amazing links, books, videos and etc.

* https://www.coursera.org/learn/digital/
* https://storage.googleapis.com/downloads.webmproject.org/docs/vp9/vp9-bitstream-specification-v0.6-20160331-draft.pdf
* http://iphome.hhi.de/wiegand/assets/pdfs/2012_12_IEEE-HEVC-Overview.pdf
* https://medium.com/@mbebenita/av1-bitstream-analyzer-d25f1c27072b#.d5a89oxz8
* https://arxiv.org/pdf/1702.00817v1.pdf
* https://people.xiph.org/~xiphmont/demo/daala/demo1.shtml
* https://people.xiph.org/~jm/daala/revisiting/
* https://people.xiph.org/~tterribe/pubs/lca2012/auckland/intro_to_video1.pdf
* https://xiph.org/video/vid1.shtml
* https://xiph.org/video/vid2.shtml
* https://www.youtube.com/watch?v=lzPaldsmJbk
* https://fosdem.org/2017/schedule/event/om_av1/
* http://ffmpeg.org/documentation.html
* https://trac.ffmpeg.org/wiki/Debug/MacroblocksAndMotionVectors
* http://www.itu.int/rec/T-REC-H.264-201610-I
* https://www.amazon.com/Understanding-Compression-Data-Modern-Developers/dp/1491961538/ref=sr_1_1?s=books&ie=UTF8&qid=1486395327&sr=1-1
* https://www.amazon.com/H-264-Advanced-Video-Compression-Standard/dp/0470516925
* https://www.amazon.com/Practical-Guide-Video-Audio-Compression/dp/0240806301/ref=sr_1_3?s=books&ie=UTF8&qid=1486396914&sr=1-3&keywords=A+PRACTICAL+GUIDE+TO+VIDEO+AUDIO
* https://www.amazon.com/Video-Encoding-Numbers-Eliminate-Guesswork/dp/0998453005/ref=sr_1_1?s=books&ie=UTF8&qid=1486396940&sr=1-1&keywords=jan+ozer
* http://www.cambridgeincolour.com/tutorials/camera-sensors.htm
* http://www.streamingmediaglobal.com/Articles/ReadArticle.aspx?ArticleID=116505&PageNum=1
* https://en.wikipedia.org/wiki/Intra-frame_coding
* https://en.wikipedia.org/wiki/Inter_frame
* http://stackoverflow.com/questions/38094302/how-to-understand-header-of-h264
* http://gentlelogic.blogspot.com.br/2011/11/exploring-h264-part-2-h264-bitstream.html
* https://trac.ffmpeg.org/wiki/Encode/H.264
* http://www.itu.int/ITU-T/recommendations/rec.aspx?rec=12904&lang=en
* http://stackoverflow.com/a/24890903
* https://cardinalpeak.com/blog/worlds-smallest-h-264-encoder/
* https://cardinalpeak.com/blog/the-h-264-sequence-parameter-set/
* https://codesequoia.wordpress.com/category/video/
* https://ffmpeg.org/ffprobe.html
* https://en.wikipedia.org/wiki/Presentation_timestamp
* https://www.linkedin.com/pulse/video-streaming-methodology-reema-majumdar
* https://leandromoreira.com.br/2016/10/09/how-to-measure-video-quality-perception/
* https://www.linkedin.com/pulse/brief-history-video-codecs-yoav-nativ
* http://www.slideshare.net/vcodex/a-short-history-of-video-coding
* https://en.wikibooks.org/wiki/MeGUI/x264_Settings
* http://www.slideshare.net/vcodex/introduction-to-video-compression-13394338
* https://www.lifewire.com/cmos-image-sensor-493271
* https://www.youtube.com/watch?v=9vgtJJ2wwMA
* https://developer.android.com/guide/topics/media/media-formats.html
* https://www.encoding.com/android/
* https://developer.apple.com/library/content/technotes/tn2224/_index.html
* http://www.lighterra.com/papers/videoencodingh264/
* http://www.slideshare.net/ManoharKuse/hevc-intra-coding
* http://www.slideshare.net/mwalendo/h264vs-hevc
* http://www.slideshare.net/rvarun7777/final-seminar-46117193
* http://x265.org/hevc-h265/
* http://www.compression.ru/video/codec_comparison/h264_2012/mpeg4_avc_h264_video_codecs_comparison.pdf
* http://vanseodesign.com/web-design/color-luminance/
* https://en.wikipedia.org/wiki/AOMedia_Video_1
* https://www.vcodex.com/h264avc-intra-precition/
* http://bbb3d.renderfarming.net/download.html
* http://www.slideshare.net/vcodex/a-short-history-of-video-coding
* https://sites.google.com/site/linuxencoding/x264-ffmpeg-mapping
* https://www.encoding.com/http-live-streaming-hls/
* https://en.wikipedia.org/wiki/Adaptive_bitrate_streaming
* http://www.adobe.com/devnet/adobe-media-server/articles/h264_encoding.html
* https://www.youtube.com/watch?v=Lto-ajuqW3w&list=PLzH6n4zXuckpKAj1_88VS-8Z6yn9zX_P6
* https://www.youtube.com/watch?v=LFXN9PiOGtY
* https://github.com/leandromoreira/encoding101
* https://cardinalpeak.com/blog/the-h-264-sequence-parameter-set/
* https://cardinalpeak.com/blog/worlds-smallest-h-264-encoder/
* https://tools.ietf.org/html/draft-fuldseth-netvc-thor-03
* https://prezi.com/8m7thtvl4ywr/mp3-and-aac-explained/
* http://www.slideshare.net/MadhawaKasun/audio-compression-23398426
* http://techblog.netflix.com/2016/08/a-large-scale-comparison-of-x264-x265.html
* http://yumichan.net/video-processing/video-compression/introduction-to-h264-nal-unit/
* http://www.springer.com/cda/content/document/cda_downloaddocument/9783642147029-c1.pdf
* http://www.red.com/learn/red-101/video-chroma-subsampling
* http://www.streamingmedia.com/Articles/Editorial/Featured-Articles/A-Progress-Report-The-Alliance-for-Open-Media-and-the-AV1-Codec-110383.aspx
* http://www.explainthatstuff.com/digitalcameras.html
* https://www.youtube.com/watch?v=LWxu4rkZBLw
* http://www.csc.villanova.edu/~rschumey/csc4800/dct.html
* https://en.wikipedia.org/wiki/File:H.264_block_diagram_with_quality_score.jpg
* https://softwaredevelopmentperestroika.wordpress.com/2014/02/11/image-processing-with-python-numpy-scipy-image-convolution/
* http://stackoverflow.com/a/24890903
* http://www.hkvstar.com
* http://www.hometheatersound.com/
* http://web.ece.ucdavis.edu/cerl/ReliableJPEG/Cung/jpeg.html
* https://en.wikipedia.org/wiki/Pixel_aspect_ratio
