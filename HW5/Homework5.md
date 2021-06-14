## Explore High Dynamic Range imaging and tonemapping

Homework Assignment 4
Environment: MATLAB
Student ID: 2021314109
Student Name: Seongjean Kim

This assignment focuses on exploring high dynamic range imaging and tonemapping.
HDR is used to create floating-point precision images, and tonemapping is used to create floating-point precision images.


## HDR Imaging
Two image sets are given. One is in a .jpg format, and the other is in a .nef format.
jpg format is easy to use, but the .nef format is not readable in matlab, therefore we need preprocessing to be done.
For such format, we use the dcraw program to change the format to .tiff
dcraw program provides several flags to choose amongst, and for our project I picked on the flags as such.

```
dcraw -w -W -o 1 -q 3 -4 -T
```
The -w flag transfers the image using the camera's white balance.
The -W flag prevents automatic brightening in the image.
The -o flag decides the output color space. 
The order is (raw, sRGB, Adobe, Wide, ProPhoto, XYZ, ACES) and for our project, i choose sRGB(=1).
The -q flag sets the interpolation quality, and for high quality i chose 3
The -4 flag represents linear 16-bit image as an output.
The -T flag writes the output in a .tiff format.

Now we are ready to process the images further on.


## Linearize Rendered Images

```matlab
function [g, lE] = gsolve(Z,B,l,w)
    n = 256;
    A = sparse(size(Z,1)*size(Z,2)+n+1,n+size(Z,1));
    b = zeros(size(A,1),1);
    k = 1;
    for i=1:size(Z,1)
        for j=1:size(Z,2)
            wij = w(Z(i,j)+1);
            A(k,Z(i,j)+1) = wij; 
            A(k,n+i) = -wij; 
            b(k,1) = wij * B(j);
            k=k+1;
        end
    end
    A(k,129) = 1;
    k=k+1;
    for i=1:n-2
        A(k,i)=l*w(i+1); 
        A(k,i+1)=-2*l*w(i+1); 
        A(k,i+2)=l*w(i+1);
        k=k+1;
    end
    x = A\b;
    g = x(1:n);
    lE = x(n+1:size(x, 1));
end
```

This gsolve function is used to linearize the rendered image.
It recieves a Z array which is the pixel values of an image reshaped to a single strip.
B is the shutter speed, l is lambda value of 1000, and w is the weight function.

A total of 3 different weight functions are used in this project.
Uniform, tent, and gaussian weight functions are each seperately used for this project, and the results of each session are compared.

The function starts with setting A and b arrays.
Then we include the data fitting equations
Next, we fix the curve by setting its middle value as 0.
Finally, we include a smoothing equation and solve the system using SVD.
The results are returend to g and lE as a single channel.
This function will be used seperately on R G B channels to retrieve the g and lE for each channel of each image.

To see how things are going on, I plotted each rgb graph to see the smoothing.
The results are for each weight function uniform, tent and gaussian.

<p align="center">
    <img src="images/capture.PNG" width="80%" height="50%">
    <p align="center">RGB plotted (Left: Uniform, Center: Tent, Right: Gaussian)</p> 
</p>

Comparint the smootheness of each, The Gaussian shows the best result, while the uniform shows the worst. 
There seems to be not much difference between the Gaussian and the Tent weight functions.

These are then proceeded in further process.
