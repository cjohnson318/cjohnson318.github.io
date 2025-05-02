---
layout: post
title: "draft"
date: 2025-0-0 00:00:00 -0700
tags: python
---

{% highlight python %}
import rasterio
with rasterio.open("image.tif", "r") as tif:
    data = tif.read(1) # read the first band
{% endhighlight %}

Key points:

  - TIFF images can hold multiple images in IFD Image File Descriptor
  - each image can have one or more bands
  - images can be either FLOAT or UNIT8
  - rasterio is the preferred python library
  - a lot can be done with gdal and gdal_calc.py