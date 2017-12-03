# img2plot
Converting images to a text graphic

```python
%matplotlib inline
import itertools
import matplotlib.pyplot as plt
from matplotlib.pyplot import imshow
import matplotlib.image as mpimg
import numpy as np
import pandas as pd
from PIL import Image
```

### Step 1: Download an image

Let's start by using some Ipython magic to download an image!


```python
!wget -O - 'https://avatars2.githubusercontent.com/u/15658638?v=3&s=200' > ./img/input/tf_logo.png
```

    --2017-12-02 19:20:57--  https://avatars2.githubusercontent.com/u/15658638?v=3&s=200
    Resolving avatars2.githubusercontent.com... 151.101.208.133
    Connecting to avatars2.githubusercontent.com|151.101.208.133|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 7015 (6.9K) [image/png]
    Saving to: 'STDOUT'
    
    -                   100%[===================>]   6.85K  --.-KB/s    in 0.001s  
    
    2017-12-02 19:20:58 (9.57 MB/s) - written to stdout [7015/7015]
    


### Step 2: Read and display the image


```python
img = Image.open("./img/input/tf_logo.png")
```


```python
# first display the image as is
imshow(img)
```



![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_6_1.png)


As a sidenote, here is a some neat little iPython magic that can be used to increase the resolution of images rendered in the notebook.


```python
%config InlineBackend.figure_format = 'retina'
imshow(img)
```


![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_8_1.png)


### Step 3: Transforming the image to gray scale


```python
# the Python imaging Library allows to read the image in greyscale, but lets ignore that for now
img_la = Image.open("./img/input/tf_logo.png").convert('LA')
imshow(img_la)
```

![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_10_1.png)


The `get_gray_scale` function define below transforms an image to grayscale by applying a grayscale transformation to each pixel in the your image, and writing those to a new image object


```python
def get_gray_scale(img):
    '''
    transform an image by applying a grayscale transformation to each pixel,
    and writing those to a new image object

    Parameters
    ----------
    img: an image object

    Returns
    -------
    df: a new image object in grayscale
    '''
    # Get size
    width, height = img.size

    # define new Image object
    img_gray = Image.new("RGB", (width, height), "white")
    pixels = img_gray.load()    

    for i in range(0, width):
        for j in range(0, height):
            pixel = img.getpixel((i, j))

            # Get R, G, B values (This are int from 0 to 255)
            red, green, blue =   pixel[0], pixel[1], pixel[2]

            # Transform to grayscale
            gray = (red * 0.299) + (green * 0.587) + (blue * 0.114)

            # Set Pixel in new image
            pixels[i, j] = (int(gray), int(gray), int(gray))
            
    return img_gray
```


```python
img_gray = get_gray_scale(img)
```


```python
imshow(img_gray)
```

![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_14_1.png)


### Step 4: Thresholding your image to Black and White

Let's improve our function to allow for pixel intensity thresholding. In this case, if the `rgb_threshold` is set to `None` then the function is identical to `get_gray_scale`, otherwise it will set the pixel value to:
- black if the pixel value is below `rgb_threshold`
- white depending if the pixel value is above `rgb_threshold`


```python
def get_bw_img(img, rgb_threshold=None):
    '''
    transform an image by applying a grayscale transformation to each pixel,
    and writing those to a new image object

    Parameters
    ----------
    img: an image object

    Returns
    -------
    df: a new image object in grayscale
    '''
    # Get size
    width, height = img.size
    
    # define new Image object
    img_threshold = Image.new("RGB", (width, height), "white")
    pixels = img_threshold.load()
    
    for i in range(0, width):
        for j in range(0, height):

            pixel = img.getpixel((i, j))

            # Get R, G, B values (This are int from 0 to 255)
            red, green, blue =   pixel[0], pixel[1], pixel[2]
            
            # Transform to grayscale
            gray = int((red * 0.299) + (green * 0.587) + (blue * 0.114))
            
            if rgb_threshold is None:
                # if no threshold is given, then apply grayscale
                pixels[i, j] = (int(gray), int(gray), int(gray))
            else:   
                if gray <= rgb_threshold:
                    # Set Pixel to black
                    pixels[i, j] = (0, 0, 0)
                else:
                    # Set Pixel to white
                    pixels[i, j] = (255, 255, 255)
            
    return img_threshold
```


```python
# plot 4 images as gray scale
plt.subplot(221)
img_threshold = get_bw_img(img, 50)
plt.imshow(img_threshold)

plt.subplot(222)
img_threshold = get_bw_img(img, 100)
plt.imshow(img_threshold)

plt.subplot(223)
img_threshold = get_bw_img(img, 150)
plt.imshow(img_threshold)

plt.subplot(224)
img_threshold = get_bw_img(img, 200)
plt.imshow(img_threshold)
# show the plot
#plt.show()
```

![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_18_1.png)


### Step 5: Regenerating the image with custom characters

With an image composed of only black and white pixels, we can now iterate through each pixel and depending on its values, plot any character of our choice at the corresponding pixel position.


```python
# get a black and white version of our image
img_threshold = get_bw_img(img, 200)
```


```python
# get size of the image
width, height = img_threshold.size
print(width, height)
```

    200 200



```python
# initialize pot
%time
fig, ax = plt.subplots(figsize=(20, 20))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

for i in range(0, width):
    for j in range(0, height):
        pixel = img_threshold.getpixel((i, j))
        if np.sum(pixel) == 0:
            ax.annotate('!', xy=(i, height-j), fontsize=8)
```

![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_23_1.png)


If you've just ran this analysis, you will notice that it took a while to run. This is because of image is of dimensions 200x200, which means we have to iterate and plot through 40,000 pixel values! Furthermore, the large number of pixels means that the space between neighboring pixels overlap and reduces visibility. Fortunately, we can resolve this issue by simply resizing our image, which can be done in one line of code.


```python
# define new image size
height = 50
width = 50
size = height, width

# resize black and white images to desired dimensions
img_resized = img_gray.resize(size, Image.ANTIALIAS)
img_resized = get_bw_img(img_resized, 200)
```

Now let's replot our image using characters instead. You can see how the plot is generate a lot faster!


```python
%time
fig, ax = plt.subplots(figsize=(10, 10))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

counter = 0
for i in range(0, height):
    counter += 1
    for j in range(0, width):
        pixel = img_resized.getpixel((j, i))
        if np.sum(pixel) == 0:
            ax.annotate(counter, xy=(j, height-i), fontsize=8)
            #ax.annotate(counter, xy=(i, height-j), fontsize=8)
            #ax.annotate(counter, xy=(j, i), fontsize=8)
```

![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_27_1.png)


### Step 5: Personalizing your img2plot

We were previously using numbers, because it is helpful to see the order in which pixels are rendered on the graph. However, we can now go beyond that and personalize our plots a little bit more. Let's start by using the `@` symbol instead!


```python
height, width = img_resized.size

fig, ax = plt.subplots(figsize=(10, 10))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

for i in range(0, height):
    counter += 1
    for j in range(0, width):
        pixel = img_resized.getpixel((j, i))
        
        if np.sum(pixel) == 0:
            ax.annotate('@', xy=(j, height-i), fontsize=8)
        else:
            ax.annotate('.', xy=(j, height-i), fontsize=8)
```


![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_30_0.png)


Great! It works, so let's try with some text data now.


```python
#text = [x.lower() for x in list(text[1].values[0][1]) if x !=' ']
text = [x.lower() for x in list('hello world!')]
```


```python
width, height = img_resized.size

fig, ax = plt.subplots(figsize=(10, 10))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

counter = -1
for i in range(0, width):
    for j in range(0, height):
        pixel = img_resized.getpixel((j, i))
        
        if np.sum(pixel) == 0:
            counter += 1
            if counter == len(text):
                counter = 0
            ax.annotate(text[counter], xy=(j, height-i), fontsize=10)
```


![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_33_0.png)


That looks a little bit confusing, but maybe it will be a little clearer if we use longer strings instead. Let's try again, but this time using the content of Albert Einstein's Wikipedia page.


```python
rawtext = pd.read_html('https://en.wikipedia.org/wiki/Albert_Einstein')
```


```python
text = ' '.join(list(itertools.chain.from_iterable(rawtext[1].values)))
```


```python
width, height = 50, 50

fig, ax = plt.subplots(figsize=(10, 10))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

counter = -1
for i in range(0, width):
    for j in range(0, height):
        pixel = img_resized.getpixel((j, i))
        
        if np.sum(pixel) == 0:
            counter += 1
            if counter == len(text):
                counter = 0
            ax.annotate(text[counter], xy=(j, height-i), fontsize=11)
```


![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_37_0.png)


Let's switch the default matplolib style to make things a bit prettier.


```python
plt.style.use('ggplot')
```


```python
width, height = 50, 50

fig, ax = plt.subplots(figsize=(10, 10))
ax.set_xlim(0,width)
ax.set_ylim(0, height)

counter = -1
for i in range(0, width):
    for j in range(0, height):
        pixel = img_resized.getpixel((j, i))
        
        if np.sum(pixel) == 0:
            counter += 1
            if counter == len(text):
                counter = 0
            ax.annotate(text[counter], xy=(j, height-i), fontsize=11)
```


![png](https://github.com/tlfvincent/img2plot/blob/master/img/readme/output_40_0.png)

