---
title: A few JavaScript Functions for Images and Files
slug: a-few-javascript-methods-for-images-files

publish_timestamp: June 30, 2018
url: https://www.codingforentrepreneurs.com/blog/a-few-javascript-methods-for-images-files/

---

The goal of this page is to provide some long-term useful functions that you might need to use in your app. We're targeting [React]() but it's all still 100% JavaScript with the ES6 syntax. Many of these functions were adapted/upgraded from StackOverflow.com responses.


### Download a Base64-encoded file
```
function downloadBase64File(base64Data, filename) {
  var element = document.createElement('a');
  element.setAttribute('href', base64Data);
  element.setAttribute('download', filename);
  element.style.display = 'none';
  document.body.appendChild(element);
  element.click();
  document.body.removeChild(element);
}
```


### Convert a Base64-encoded string to a File object
```
function base64StringtoFile(base64String, filename) {
    var arr = base64String.split(','), mime = arr[0].match(/:(.*?);/)[1],
        bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
    while(n--){
        u8arr[n] = bstr.charCodeAt(n);
    }
    return new File([u8arr], filename, {type:mime});
}
```

### Extract an Base64 Image's File Extension
```
function extractImageFileExtensionFromBase64(base64Data){
    return base64Data.substring("data:image/".length, base64Data.indexOf(";base64"))
}
```


### Base64 Image to Canvas with a Crop

This will crop your passed Base64 image to fit on to a canvas of the same exact crop.

```
function image64toCanvasRef(canvasRef, image64, pixelCrop){
  const canvas = canvasRef // document.createElement('canvas');
  canvas.width = pixelCrop.width;
  canvas.height = pixelCrop.height;
  const ctx = canvas.getContext('2d');
  const image = new Image()
  image.src = image64
  image.onload = function() {
      ctx.drawImage(
        image,
        pixelCrop.x,
        pixelCrop.y,
        pixelCrop.width,
        pixelCrop.height,
        0,
        0,
        pixelCrop.width,
        pixelCrop.height
      )
    }
}
```

Example usage:

```
const canvasReference = document.createElement('canvas') // or your own canvas
const base64ImageData = "" // fill in your own base64 image data here.
const myCrop = {
      x: 100,
      y: 100,
      width: 350,
     height: 350
}


image64toCanvasRef(canvasReference, base64ImageData, myCrop)
```

Do you have any ideas? Comment below