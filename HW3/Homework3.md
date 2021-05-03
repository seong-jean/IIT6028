## Implement Poisson Blending

Homework Assignment 3  
Environment: MATLAB
Student ID: 2021314109
Student Name: Seongjean Kim

This assignment focuses on gradient-domain processing, and in particular Poisson blending.
Functions are implemented to blend an object or textrue from a source into a target image.


## Toy Problem

```matlab
function im_out = toy_reconstruct(toyim)

[imh, imw, nn] = size(toyim);
toyim = im2double(toyim);
im_size = imh*imw;
im2var = zeros(imh, imw);
im2var(1:im_size) = 1:im_size;

A = sparse(imh*(imw-1)+(imh-1)*imw+1, im_size);
b = zeros(im_size, nn);
e = 0;

for h = 1:imh
    for w = 1:imw-1
        e = e+1;
        A(e,im2var(h,w+1)) = 1;
        A(e,im2var(h,w)) = -1;
        b(e) = toyim(h,w+1) - toyim(h,w);
    end
end

for w = 1:imw
    for h = 1:imh-1
        e = e+1;
        A(e, im2var(h+1,w)) = 1;
        A(e, im2var(h,w)) = -1;
        b(e) = toyim(h+1,w) - toyim(h,w);
    end
end

e = e+1;
A(e, im2var(1,1)) = 1;
b(e) = toyim(1,1);

v = A \ b;
im_out = reshape(v, [imh, imw]);
end
```
<p align="center">
    <img src="images/1-2.png" width="40%" height="40%">
    <img src="images/toy_problem.png" width="40%" height="40%">
    <p align="center">Input(Left) and Result(Right) of toy_reconstruct.m</p> 
</p>

toy_reconstruction function is a practice on implemntation for gradient domain processing.
The process considers each x and y axis seperately and stores the gradient values using sparse function.
The first for loop is processed on each row, and the next loop is processed on each column.
After each process is completed, the \ function of matlab is used to compute division in the opposite direction.
The resulting image is the returned as above.

The overall difference between the original and result image was as follow.

Error: 0+2.5417e-06i

Which, we can conclude that the two images are almost identical.
Now that we understand how to implement gradient domain processing, we move on to implementing Poisson blending.



## Poisson Blending

```matlab
function im_blend = poissonBlend(im_s, mask_s, im_background)

[imh, imw, nn] = size(im_s);

im2var = zeros(imh, imw);
im_size = imh*imw;
im2var(1:im_size) = 1: im_size;

A = sparse(im_size, im_size);
b = zeros(im_size, nn);
e = 0;
disp("loop start");
for h = 1:imh
    for w = 1:imw
        e = e+1;
        if mask_s(h,w) == 1
            A(e, im2var(h,w)) = 4;
            A(e, im2var(h,w-1)) = -1;
            A(e, im2var(h,w+1)) = -1;
            A(e, im2var(h-1,w)) = -1;
            A(e, im2var(h+1,w)) = -1;
            b(e,:) = 4*im_s(h,w,:) - im_s(h,w+1,:) - im_s(h,w-1,:) - im_s(h-1,w,:) - im_s(h+1,w,:);
        else
            A(e, im2var(h,w)) = 1;
            b(e,:) = im_background(h,w,:);
        end
    end
end
v = A \ b;
im_blend = reshape(v, [imh, imw, nn]);
end
```
<p align="center">
    <img src="images/2-3.png" width="50%" height="50%">
    <p align="center">Poisson blending</p> 
</p>

Using the toy_reconstruct function as the base, we now add blending onto the process.
The background image for this process will be hiking.jpg, and images blended to this will be penguin-chick.jpeg and penguin.jpg



```matlab
function output = upsample(gaussian_in, residual_in)
    gaussian_temp = imresize(gaussian_in, 2);
    output = gaussian_temp + residual_in;
end
```
<p align="center">
    <img src="images/yiq_upsample.png" width="50%" height="50%">
    <p align="center">Upsampling</p> 
</p>

Using the function implemented above, such Laplacian pyramid for each frame can be upsampled to reform the original image of that frame.


## Temporal Filtering

```matlab
function Hd = butterworthBandpassFilter(Fs, N, Fc1, Fc2)
    h  = fdesign.bandpass('N,F3dB1,F3dB2', N, Fc1, Fc2, Fs);
    Hd = design(h, 'butter');
end
```

<p align="center">
    <img src="images/Temporal_filtering_face.png" width="50%" height="50%">
    <p align="center">Temporal filtering for face</p> 
</p>

Temporal filtering is based on butterworthBandpassFilter. 
Using this function, the frequency band of interest is extracted.
The values used for baby2.mp4 is Fs = 30, N = 256, Fc1 = 0.83, Fc2 = 1.16
The values used for face.mp4 is Fs = 30, N = 256, Fc1 = 0.8, Fc2 = 1.0


## Extracting the Frequency Band of Interest

```matlab
function output = extracting(Hd, input)
    [height, width, ch, frame_index] = size(input);
    output = zeros(height, width, ch, frame_index);
    out_pixel = zeros(frame_index, ch);
    Hd_fft = freqz(Hd, frame_index);
    for h = 1: height
        for w = 1: width
            for c = 1:ch
                out_pixel(:,ch) = input(h,w,ch,:);
                out_pixel_fft = fft(out_pixel);
                out_filtered = abs(ifft(out_pixel_fft .* Hd_fft));
                output(h,w,ch,:) = out_filtered(:, ch);
            end
        end
    end
end
```
<p align="center">
    <img src="images/residual_by5.png" width="50%" height="50%">
    <p align="center">Residual image multiplied by 5</p> 
</p>

In this process, freqz function is used to extract Hd_fft info.
What is returned is the frequency components of the filter for fast computation.



## Image Reconstruction
```matlab
frames_recon = zeros(height, width, ch, frame_index);

R_Ex_Re_0 = zeros(size(R_Ex_0,1), size(R_Ex_0,2), ch);
R_Ex_Re_1 = zeros(size(R_Ex_1,1), size(R_Ex_1,2), ch);
R_Ex_Re_2 = zeros(size(R_Ex_2,1), size(R_Ex_2,2), ch);
R_Ex_Re_3 = zeros(size(R_Ex_3,1), size(R_Ex_3,2), ch);
G_Ex_Re_4 = zeros(size(G_Ex_4,1), size(G_Ex_4,2), ch);
disp(size(R_Ex_3,1));
disp(size(G_Ex_4,1));
for i = 1:3
    for t = 1: frame_index
        R_Ex_Re_0(:,:,i) = R_Ex_0(:,:,i,t);
        R_Ex_Re_1(:,:,i) = R_Ex_1(:,:,i,t);
        R_Ex_Re_2(:,:,i) = R_Ex_2(:,:,i,t);
        R_Ex_Re_3(:,:,i) = R_Ex_3(:,:,i,t);
        G_Ex_Re_4(:,:,i) = G_Ex_4(:,:,i,t);
        frame_recon = frames(:,:,i,t) + upsample(upsample(upsample(upsample(G_Ex_Re_4, 120*R_Ex_Re_3), R_Ex_Re_2), R_Ex_Re_1), R_Ex_Re_0);
        frames_recon(:,:,i,t) = frame_recon(:,:,i);
    end
end
```

<p align="center">
    <img src="images/face.gif" width="50%" height="50%">
    <p align="center">face.mp4 with Eulerian video magnification</p>
</p>


<p align="center">
    <img src="images/baby2.gif" width="50%" height="50%">
    <p align="center">baby2.mp4 with Eulerian video magnification</p>
</p>

This process upsamples the Laplacian pyramid into a single image per frame.
By setting the a's as different values, we can obtain results with different frequency values amplified.
The frames are exported and saved as a video.

## Capture and Motion-Magnify your own Video

<p align="center">
    <img src="images/own.gif" width="50%" height="50%">
    <p align="center">Own Video Motion-Magnify</p>
</p>
 
This is the result of motion magnifying on a short clip, where the sleeve of the jacket was shifted slightly due to wind blow.
