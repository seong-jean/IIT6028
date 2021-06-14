## Light Field Rendering, Focal Stacks, and Depth from Defocus

Homework Assignment 5
Environment: MATLAB
Student ID: 2021314109
Student Name: Seongjean Kim

This assignment focuses on exploring light fields, focal stacks, and depth from defocus.
Having access to the full light field of a scene allows creating images that correspond to different viewpoints.
Such different viewpoints will be combined to create a all-focus image as a result.


## Initials

First, the light field image is loaded to Matlab.
Then, we create a 5-dimensional array L(u, v, s, t, c) from the loaded image.
From above, u and v correspond to the coordinates on the aperture,
s and t correspond to the coordinates on the lenslet array.
c corresponds to the color channels of the input, where in this project c will be 3.

```matlab
img_lightfields = imread('chessboard_lightfield.png');

global u v s t c
u = 16;
v = 16;
s = size(img_lightfield, 1) / u;
t = size(img_lightfield, 2) / v;
c = 3;

img_array = zeros(u, v, s, t, c);
img_array = uint8(img_array);
for l = 1:s
    for m = 1:t
        for n = 1:u
            for o = 1:v
                img_array(n, o, l, m, 1) = img_lightfield(u*(l-1)+n, v*(m-1)+o, 1);
                img_array(n, o, l, m, 2) = img_lightfield(u*(l-1)+n, v*(m-1)+o, 2);
                img_array(n, o, l, m, 3) = img_lightfield(u*(l-1)+n, v*(m-1)+o, 3);
            end
        end
    end
end
```

## Sub-aperture views
A sub-aperture is created by rearranging the pixels in the input image.

```matlab

```

The resulting image is shown below in a form of a 16x16 2D mosaic.
The u value is increased vertically, and the v value is increased horizontally.

<p align="center">
    <img src="images/capture.PNG" width="80%" height="50%">
    <p align="center">RGB plotted (Left: Uniform, Center: Tent, Right: Gaussian)</p> 
</p>
