## Implement a basic image processing pipeline

Homework Assignment 1  
Environment: MATLAB
<html>
    <head>
        <!-- Add the slick-theme.css if you want default styling -->
        <link rel="stylesheet" type="text/css" href="//cdn.jsdelivr.net/npm/slick-carousel@1.8.1/slick/slick.css"/>
        <!-- Add the slick-theme.css if you want default styling -->
        <link rel="stylesheet" type="text/css" href="//cdn.jsdelivr.net/npm/slick-carousel@1.8.1/slick/slick-theme.css"/>
    </head>


## Initials

```matlab
tiff_img = imread('C:/Users/ksj/MATLAB/Projects/A1/assign1/data/banana_slug.tiff');
[Height, Width] = size(tiff_img);
X = ['Image Height : ',num2str(Height),', Image Width : ',num2str(Width)];
disp(X);
disp(class(tiff_img));
img = double(tiff_img);
imwrite(tiff_img, 'tiff_img.png');
```
<p align="center">
    <img src="Images/tiff_img.png" width="50%" height="50%">
</p>

banana_slug.tiff file is loaded using imread function.  
Using size and class, we obtain information of the image format.  
Its size is 2856 * 4290 (Height * Width), and each value is a unit16 format.  
For further process, we change the format to double, and save it as img.  



## Linearization

```matlab
img = (img - 2047)/(15000-2047);
img = max(0, img);
img = min(1, img);
imwrite(img, 'img_Linearization.png');
```
Because only 14 of these pixesl contain useful data, we convert the image into a linear array within the range [0,1].  
Using min and max functions, I removed values above 1 and below 0.  

<p align="center">
    <img src="Images/img_Linearization.png" width="50%" height="50%">
</p>


## Identifying the Correct Bayer Pattern

```matlab
ba1 = img(1:2:end, 1:2:end);
ba2 = img(1:2:end, 2:2:end);
ba3 = img(2:2:end, 1:2:end);
ba4 = img(2:2:end, 2:2:end);
mean_ba = [mean(ba1(:)), mean(ba2(:)), mean(ba3(:)), mean(ba4(:))];
disp(mean_ba);
```
To identify which Bayer pattern is used, I compared the mean value of each space of the 2x2 squares.  
mean_ba shows the mean values, and from here we can acknowledge that the mean of ba2 and ba3 have similar values(=green).  
Since I know the greens, now i need to figure out between RGGB and BGGR.  

```matlab
img_rggb = cat(3, ba1, ba3, ba4);
img_bggr = cat(3, ba4, ba3, ba1);
imwrite(min(1, img_rggb*5), 'img_BayerPattern_rggb.png');
imwrite(min(1, img_bggr*5), 'img_BayerPattern_bggr.png');
img_rgb = img_rggb;
```
<p align="center">
    <img src="Images/img_BayerPattern_rggb.png" width="40%" height="40%">
    <img src="Images/img_BayerPattern_bggr.png" width="40%" height="40%">
</p>

Now I compare the results by concatenating 3 values in each format.  
Knowing that the banana slug should have a yellow color, I set RGGB as the Bayer pattern.  


## White Balancing
### White world automatic white balancing
```matlab
im_r = max(max(img_rgb(:, :, 1)));
im_g = max(max(img_rgb(:, :, 2)));
im_b = max(max(img_rgb(:, :, 3)));
img_wb = cat(3, img_rgb(:,:,1) * im_g / im_r, img_rgb(:,:,2), img_rgb(:,:,3) * im_g / im_b);
imwrite(img_wb, 'img_WhiteBalancing.png');
```

<p align="center">
    <img src="Images/img_WhiteBalancing.png" width="50%" height="50%">
</p>

The image has a high Green value overall, White Balancing is done to adjust Red and Blue values.  
Red and Blue values are incresed by a certain ratio obatined.  
Here I used 'white world automatic white balancing' for this assignment.  

### Gray world automatic white balancing
```matlab
im_r = mean(mean(img_rgb(:, :, 1)));
im_g = mean(mean(img_rgb(:, :, 2)));
im_b = mean(mean(img_rgb(:, :, 3)));
img_wb = cat(3, img_rgb(:,:,1) * im_g / im_r, img_rgb(:,:,2), img_rgb(:,:,3) * im_g / im_b);
imwrite(img_wb, 'img_GrayBalancing.png');
```
<p align="center">
    <img src="Images/img_GrayBalancing.png" width="50%" height="50%">
</p>

This is the script for 'gray world automatic white balancing'. It was not used for this assignment.  

## Demosaicing

```matlab
img_wb_dem_r = interp2(img_wb(:,:,1));
img_wb_dem_g = interp2(img_wb(:,:,2));
img_wb_dem_b = interp2(img_wb(:,:,3));
img_wb_dem = cat(3, img_wb_dem_r, img_wb_dem_g, img_wb_dem_b);
imwrite(img_wb_dem, 'img_Demosaicing.png');
```
To retrieve color, demosaicing is needed.  
Here instead of using the demosaic function, it is improvised using interp2 function.  

<p align="center">
    <img src="Images/img_Demosaicing.png" width="50%" height="50%">
</p>

## Brightness Adjustment and Gamma Correction

```matlab
img_wb_dem_gray = rgb2gray(img_wb_dem);
img_wb_dem = min(1, img_wb_dem * 2.8);
if img_wb_dem_gray <= 0.0031308
    img_wb_dem_out = 12.92 * img_wb_dem;
else
    img_wb_dem_out = (1.055) * power(img_wb_dem, 1/2.4) - 0.055;
end
imwrite(img_wb_dem_out, 'img_GammaCorrection.png');
```
<p align="center">
    <img src="Images/img_GammaCorrection.png" width="50%" height="50%">
</p>

Still, the image is too dark, I adjusted the image brightness by 2.8.
Then, gamma correction(tone reproduction) was applied to the image.


## Compression

```matlab
quality_value = [95, 70, 50, 35, 20, 10, 5];
quality_size = size(quality_value);
disp(quality_size);
for i = 1:7
    file_name = 'img_wb_dem_gamma_q' + string(quality_value(i)) + '.jpeg';
    imwrite(img_wb_dem_out, file_name, 'quality', quality_value(i));
end
```

    <body>
    <div id="main-carousel" class="carousel slide" data-ride="carousel">
        <!-- Indicators -->
        <ol class="carousel-indicators">
          <li data-target="#main-carousel" data-slide-to="0" class="active"></li>
          <li data-target="#main-carousel" data-slide-to="1"></li>
          <li data-target="#main-carousel" data-slide-to="2"></li>
          <li data-target="#main-carousel" data-slide-to="3"></li>
          <li data-target="#main-carousel" data-slide-to="4"></li>
          <li data-target="#main-carousel" data-slide-to="5"></li>
          <li data-target="#main-carousel" data-slide-to="6"></li>
        </ol>

        <!-- Wrapper for slides -->
        <div class="carousel-inner" role="listbox">

          <div class="item active" id="item_1">
            <img class="results_img" src="Images/img_wb_dem_gamma_q95.jpeg" alt="img_1">
            <div class="carousel-caption">
              <h3>Quality 95%</h3>
              <p>original tiff image</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q70.jpeg" alt="img_2">
            <div class="carousel-caption">
              <h3>Quality 70%</h3>
              <p>after linearization</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q50.jpeg" alt="img_3">
            <div class="carousel-caption">
              <h3>Quality 50%</h3>
              <p>left-top: bggr / right-top: gbrg / left-bottom: grbg / right-bottom: rggb(best)</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q35.jpeg" alt="img_4">
            <div class="carousel-caption">
              <h3>Quality 35%</h3>
              <p>results of rggb(best)</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q20.jpeg" alt="img_5">
            <div class="carousel-caption">
              <h3>Quality 20%</h3>
              <p>top: gray world / bottom: white world(best)</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q10.jpeg" alt="img_6">
            <div class="carousel-caption">
              <h3>Quality 10%</h3>
              <p>results of white world automatic white balancing(best)</p>
            </div>
          </div>

          <div class="item">
            <img class="results_img" src="Images/img_wb_dem_gamma_q5.jpeg" alt="img_7">
            <div class="carousel-caption">
              <h3>Quality 5%</h3>
              <p>used results of white world AWB</p>
            </div>
          </div>
        </div>

        <!-- Controls -->
        <a class="left carousel-control" href="#main-carousel" role="button" data-slide="prev">
          <span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>
          <span class="sr-only">Previous</span>
        </a>
        <a class="right carousel-control" href="#main-carousel" role="button" data-slide="next">
          <span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span>
          <span class="sr-only">Next</span>
        </a>
      </div>
    </div>
    <script type="text/javascript" src="//cdn.jsdelivr.net/npm/slick-carousel@1.8.1/slick/slick.min.js"></script>
    </body>
</html>      
<p align="center">
    <img src="Images/img_wb_dem_gamma_q95.jpeg" width="20%" height="20%">
    <img src="Images/img_wb_dem_gamma_q50.jpeg" width="20%" height="20%">
    <img src="Images/img_wb_dem_gamma_q35.jpeg" width="20%" height="20%">
    <img src="Images/img_wb_dem_gamma_q10.jpeg" width="20%" height="20%">
    <img src="Images/img_wb_dem_gamma_q5.jpeg" width="20%" height="20%">
</p>

Finally, i apply compression to the image in several values.  
From quality value 35 and  lower, compression can be easily observed.


