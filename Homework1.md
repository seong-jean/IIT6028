## Implement a basic image processing pipeline

Homework Assignment 1  
Environment: MATLAB

[editor on GitHub](https://github.com/seong-jean/IIT6028/edit/gh-pages/Homework1.md)

## Initials

```matlab
tiff_img = imread('C:/Users/ksj/MATLAB/Projects/A1/assign1/data/banana_slug.tiff');
[Height, Width] = size(tiff_img);
X = ['Image Height : ',num2str(Height),', Image Width : ',num2str(Width)];
disp(X);
class(tiff_img);
img = double(tiff_img);
imwrite(tiff_img, 'tiff_img.png');

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

## Linearization
```matlab
img = (img - 2047)/(15000-2047);
img = max(0, img);
img = min(1, img);

imwrite(img, 'img_Linearization.png');
```

## Identifying the Correct Bayer Pattern
```
ba1 = img(1:2:end, 1:2:end);
ba2 = img(1:2:end, 2:2:end);
ba3 = img(2:2:end, 1:2:end);
ba4 = img(2:2:end, 2:2:end);
mean_ba = [mean(ba1(:)), mean(ba2(:)), mean(ba3(:)), mean(ba4(:))];
disp(mean_ba);
img_rggb = cat(3, ba1, ba3, ba4);
img_bggr = cat(3, ba4, ba3, ba1);
imwrite(min(1, img_rggb*5), 'img_BayerPattern_rggb.png');
imwrite(min(1, img_bggr*5), 'img_BayerPattern_bggr.png');
img_rgb = img_rggb;
```

## White Balancing
```
im_r = max(max(img_rgb(:, :, 1)));
im_g = max(max(img_rgb(:, :, 2)));
im_b = max(max(img_rgb(:, :, 3)));
img_wb = cat(3, img_rgb(:,:,1) * im_g / im_r, img_rgb(:,:,2), img_rgb(:,:,3) * im_g / im_b);
imwrite(img_wb, 'img_WhiteBalancing.png');
```

## Demosaicing
```
img_wb_dem_r = interp2(img_wb(:,:,1));
img_wb_dem_g = interp2(img_wb(:,:,2));
img_wb_dem_b = interp2(img_wb(:,:,3));
img_wb_dem = cat(3, img_wb_dem_r, img_wb_dem_g, img_wb_dem_b);
imwrite(img_wb_dem, 'img_Demosaicing.png');
```

## Brightness Adjustment and Gamma Correction
```
img_wb_dem_gray = rgb2gray(img_wb_dem);
img_wb_dem = min(1, img_wb_dem * 2.8);
if img_wb_dem_gray <= 0.0031308
    img_wb_dem_out = 12.92 * img_wb_dem;
else
    img_wb_dem_out = (1.055) * power(img_wb_dem, 1/2.4) - 0.055;
end
imwrite(img_wb_dem_out, 'img_GammaCorrection.png');
```

## Compression
```
quality_value = [90, 70, 50, 35, 20, 10, 5];
quality_size = size(quality_value);
disp(quality_size);
for i = 1:7
    file_name = 'img_wb_dem_gamma_q' + string(quality_value(i)) + '.jpeg';
    imwrite(img_wb_dem_out, file_name, 'quality', quality_value(i));
end
```
Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
