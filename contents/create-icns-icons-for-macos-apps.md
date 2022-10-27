---
title: Create icns Icons for macOS Apps.
slug: create-icns-icons-for-macos-apps

publish_timestamp: Aug. 27, 2020
url: https://www.codingforentrepreneurs.com/blog/create-icns-icons-for-macos-apps/

---


I've been working a lot with Electron and PyWebView lately and thus needed to create some icons for my projects.

macOS apps require the file type 'icns' for app icons. 

I found [this post](https://stackoverflow.com/a/20703594) to be incredibly helpful. I modified it below so it's a bit more reusable. 

It's simple. You just need a single `.png` file that's at least 1024 x 1024 pixels. I'd assume that as screen resolutions go up, we'll see this number increase as well.

Once you have that, let's do the following:

#### 1. Navigate to your storage location
```
cd /path/to/destination/
```

#### 2. Set your inputs and output variables

```
input_filepath="cfe_icon_1024_x_1024.png" 
output_iconset_name="CFE.iconset"
mkdir $output_iconset_name
```
> In bash, make sure you have no spaces between setting variables. Such as `my_var="this"` and not `my_var = "this"`

```
sips -z 16 16     $input_filepath --out "${output_iconset_name}/icon_16x16.png"
sips -z 32 32     $input_filepath --out "${output_iconset_name}/icon_16x16@2x.png"
sips -z 32 32     $input_filepath --out "${output_iconset_name}/icon_32x32.png"
sips -z 64 64     $input_filepath --out "${output_iconset_name}/icon_32x32@2x.png"
sips -z 128 128   $input_filepath --out "${output_iconset_name}/icon_128x128.png"
sips -z 256 256   $input_filepath --out "${output_iconset_name}/icon_128x128@2x.png"
sips -z 256 256   $input_filepath --out "${output_iconset_name}/icon_256x256.png"
sips -z 512 512   $input_filepath --out "${output_iconset_name}/icon_256x256@2x.png"
sips -z 512 512   $input_filepath --out "${output_iconset_name}/icon_512x512.png"
```
> Take note that the output file names `icon_256x256@2x.png`,  `icon_512x512.png` and so on should remain the exact same for you as well.

Sips stands for scriptable image processing system and  "This tool is used to query or modify raster image files and ColorSync ICC profiles."

As you can see, `sips` will take our input image, resize it, and output a new image with that new size in our newly created `iconset` folder. You can re-use this iconset in web apps as well.

#### 3. Create the `icns`
Now we can create our `icns`:

```
iconutil -c icns $output_iconset_name
```

> optionally, remove this directory unless you need it in the future with `rm -R $output_iconset_name`


#### 4. All together

```
input_filepath="cfe_icon_1024_x_1024.png"
output_iconset_name="CFE.iconset"
mkdir $output_iconset_name

sips -z 16 16     $input_filepath --out "${output_iconset_name}/icon_16x16.png"
sips -z 32 32     $input_filepath --out "${output_iconset_name}/icon_16x16@2x.png"
sips -z 32 32     $input_filepath --out "${output_iconset_name}/icon_32x32.png"
sips -z 64 64     $input_filepath --out "${output_iconset_name}/icon_32x32@2x.png"
sips -z 128 128   $input_filepath --out "${output_iconset_name}/icon_128x128.png"
sips -z 256 256   $input_filepath --out "${output_iconset_name}/icon_128x128@2x.png"
sips -z 256 256   $input_filepath --out "${output_iconset_name}/icon_256x256.png"
sips -z 512 512   $input_filepath --out "${output_iconset_name}/icon_256x256@2x.png"
sips -z 512 512   $input_filepath --out "${output_iconset_name}/icon_512x512.png"

iconutil -c icns $output_iconset_name

rm -R $output_iconset_name
```

That's it. Pretty cool huh?
