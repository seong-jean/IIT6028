## Implement a basic image processing pipeline

Homework Assignment 1

Environment: MATLAB

[editor on GitHub](https://github.com/seong-jean/IIT6028/edit/gh-pages/Homework1.md)

## Initials

```markdown

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
```markdown
img = (img - 2047)/(15000-2047);
img = max(0, img);
img = min(1, img);

imwrite(img, 'img_Linearization.png');
```

## Identifying the Correct Bayer Pattern
```markdown
ba1 = img(1:2:end, 1:2:end);
ba2 = img(1:2:end, 2:2:end);
ba3 = img(2:2:end, 1:2:end);
ba4 = img(2:2:end, 2:2:end);
mean_ba = [mean(ba1(:)), mean(ba2(:)), mean(ba3(:)), mean(ba4(:))];
disp(mean_ba);
img_rgb = cat(3, ba1, ba3, ba4);

imwrite(min(1, img_rgb*5), 'img_BayerPattern.png');
```

## White Balancing
```markdown
img_wbg = white_balancing_grey(img_rgb);
img_wb = white_balancing_white(img_rgb);

imwrite(img_wb, 'img_WhiteBalancing.png');
```

## Demosaicing
```markdown
img_wb_dem_r = interp2(img_wb(:,:,1));
img_wb_dem_g = interp2(img_wb(:,:,2));
img_wb_dem_b = interp2(img_wb(:,:,3));
img_wb_dem = cat(3, img_wb_dem_r, img_wb_dem_g, img_wb_dem_b);

imwrite(img_wb_dem, 'img_Demosaicing.png');
```

## Brightness Adjustment and Gamma Correction
```markdown
img_wb_dem_gamma = rgb2gray(img_wb_dem);
if img_wb_dem_gamma < 0.0031308
    img_wb_dem_gamma = 12.92 * img_wb_dem;
else
    img_wb_dem_gamma = (1.055) * power(img_wb_dem, 1/2.4) - 0.055;
end
imwrite(img_wb_dem_gamma, 'img_GammaCorrection.png');
```

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
