---
title: Extract GPS &amp; Exif Data from Images using Python
slug: extract-gps-exif-images-python

publish_timestamp: Nov. 21, 2017
url: https://www.codingforentrepreneurs.com/blog/extract-gps-exif-images-python/

---

Smartphones & DSLRS attach all kinds of meta data to images. This meta data, known as [EXIF Data](https://en.wikipedia.org/wiki/Exif), has attribuates like make, model, and even GPS data about the image itself. This is how Facebook, Instagram, and Twitter, can identify the image's location without you being near it or even allowing "location services" being turned on. 

This post is about implementing a simple script so you can do the same in your projects. Enjoy!


### Requirements:
```
pip install pillow
```


### Example usage:
```
path_name = 'path/to/your/image.jpg'
meta_data =  ImageMetaData(path_name)
latlng =meta_data.get_lat_lng()
print(latlng)
exif_data = meta_data.get_exif_data()
print(exif_data)

```


### The Script

```
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS


class ImageMetaData(object):
    '''
    Extract the exif data from any image. Data includes GPS coordinates, 
    Focal Length, Manufacture, and more.
    '''
    exif_data = None
    image = None

    def __init__(self, img_path):
        self.image = Image.open(img_path)
        #print(self.image._getexif())
        self.get_exif_data()
        super(ImageMetaData, self).__init__()

    def get_exif_data(self):
        """Returns a dictionary from the exif data of an PIL Image item. Also converts the GPS Tags"""
        exif_data = {}
        info = self.image._getexif()
        if info:
            for tag, value in info.items():
                decoded = TAGS.get(tag, tag)
                if decoded == "GPSInfo":
                    gps_data = {}
                    for t in value:
                        sub_decoded = GPSTAGS.get(t, t)
                        gps_data[sub_decoded] = value[t]

                    exif_data[decoded] = gps_data
                else:
                    exif_data[decoded] = value
        self.exif_data = exif_data
        return exif_data

    def get_if_exist(self, data, key):
        if key in data:
            return data[key]
        return None

    def convert_to_degress(self, value):

        """Helper function to convert the GPS coordinates 
        stored in the EXIF to degress in float format"""
        d0 = value[0][0]
        d1 = value[0][1]
        d = float(d0) / float(d1)

        m0 = value[1][0]
        m1 = value[1][1]
        m = float(m0) / float(m1)

        s0 = value[2][0]
        s1 = value[2][1]
        s = float(s0) / float(s1)

        return d + (m / 60.0) + (s / 3600.0)

    def get_lat_lng(self):
        """Returns the latitude and longitude, if available, from the provided exif_data (obtained through get_exif_data above)"""
        lat = None
        lng = None
        exif_data = self.get_exif_data()
        #print(exif_data)
        if "GPSInfo" in exif_data:      
            gps_info = exif_data["GPSInfo"]
            gps_latitude = self.get_if_exist(gps_info, "GPSLatitude")
            gps_latitude_ref = self.get_if_exist(gps_info, 'GPSLatitudeRef')
            gps_longitude = self.get_if_exist(gps_info, 'GPSLongitude')
            gps_longitude_ref = self.get_if_exist(gps_info, 'GPSLongitudeRef')
            if gps_latitude and gps_latitude_ref and gps_longitude and gps_longitude_ref:
                lat = self.convert_to_degress(gps_latitude)
                if gps_latitude_ref != "N":                     
                    lat = 0 - lat
                lng = self.convert_to_degress(gps_longitude)
                if gps_longitude_ref != "E":
                    lng = 0 - lng
        return lat, lng
```

This code was adapted + updated from [this gist](https://gist.github.com/erans/983821).