---
title: Convert SVG Icons To CSS Webfonts and Deploy to CDN
slug: svg-icons-to-css-webfonts-to-cdn

publish_timestamp: March 20, 2019
url: https://www.codingforentrepreneurs.com/blog/svg-icons-to-css-webfonts-to-cdn/

---

If you're like me, you've undoubtably used [Font Awesome](http://fontawesome.io/). It's a simple way to have great looking icons that are both easy to remember and easy to implement. I mean `fa fa-github` is rather easy to remember that's the github icon. 


But what if our team decided we want our own awesome icons? 

Enter this guide. Convert SVG icons into your very own css-based fonts (including `eot`, `ttf`, `woff`, and `woff2`).



**Here's what we'll do**
- Download Sample SVG Icons from [Simple Icons](https://github.com/simple-icons/simple-icons)
- Convert SVG Icons into a "webfont icon kit" aka css fonts with [icon-font-generator](https://github.com/Workshape/icon-font-generator)
- Create a CDN via AWS S3 and AWS Cloudfront with a custom domain


**Requirements**
- Download and install [node.js](https://nodejs.org/en/download/)
- Download and install [Git](https://git-scm.com/) (recommended but optional)

### Watch

[[ youtube id=VDy9xktOI6M ]]


### 1. Download Sample Icons
For this, we use [Simple Icons](https://github.com/simple-icons/simple-icons) because it gives us a breadth of branded icons that we might want to include in our own set. 

> Are you making custom SVG icons? If that's the case, you'll have to test your icon a few times. I've had the best luck with single-color (black) SVG icons that are square (ie 30px x 30px).


Add your working directory
```
cd /path/to/your/dev/folder/
mkdir endless_fonts
cd endless_fonts
```

Clone [Simple Fonts]
```
git clone https://github.com/simple-icons/simple-icons .
```

We just want the `icons/` dir. 

Remove all other directories
```
rm -rf .git .github _data scripts images tests
```
Remove all other files
```
rm CNAME CONTRIBUTING.md README.md LICENSE.md package-lock.json package.json index.html stylesheet.css site_script.js
```

Verify only the icons directory is left.
```
$ ls
icons
```

Cool. Now we have the SVG Icons we want.


### 2. SVG Icons to CSS Fonts (A Webfont Kit)

1. Create `package.json`
```
touch package.json
```

Add the following (change as you'd like):
```
{
  "name": "endlessfonts",
  "version": "1.0.0",
  "description": "An example of EndlessFonts.com",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Justin Mitchel",
  "license": "MIT"
}
```

2. Install [icon-font-generator](https://github.com/Workshape/icon-font-generator)
```
npm i -g icon-font-generator
```

3. Make final destination directory
```
mkdir dist
```

4. Build Icon set
```
icon-font-generator icons/*.svg -o dist -n endless -p endless --normalize  --round 0 --mono --height 250
```
`-n` is name of your files (`css`, `eot`, `ttf`, `woff`, `woff2`, etc)
`-p` is the name of your html class prefix like `endless-` or `yours-`, so an icon would be `<i class='endless-apple'></i>`
`--normalize` normalize icons sizes
`--round 0` SVG rounding at 0
`--mono` makes font monospace
`--height 250` Making the height 250 will ensure our webfonts don't distort (try 30, you'll see what I mean)

5. Ignore Broken Icons
In my case, two icons failed: viber and conda-forge. These icons I decided to just not display. I added the following in the `dist/endless.html` inline styles.


```css
.endless-viber {
    display: none;
}
.endless-conda-forge {
    display: none
}

```

6. Verify generated icons

```
cd dist
python3 -m http.server -8888
```

```
open http://localhost:8888/endless.html
```
Change `endless.html` to the name you set above (in `-n`).


### 3. Icon Font Kit as a CDN on AWS

##### 3.1 Setup S3
1. Open S3 [console.aws.amazon.com/s3/](https://console.aws.amazon.com/s3/)
2. Create Bucket
3. Select us-east-1 (this is for the CDN later)
4. Add the name as the cdn domain name such as `www.endlessfonts.com` (ensure you own this domain as well)
5. Set all defaults
6. Select create Create
7. Click **Permissions**
8. Under **Public access settings for this bucket** click **edit** and ***uncheck*** the following:
- Block new public ACLs and uploading public objects (Recommended) 
- Remove public access granted through public ACLs (Recommended)

9. Upload your recently created dist folder as the root.
    - Grant `public read` for all items (recommended so people can access these without a signed key) 
    - Rename `endless.html` as `index.html` Include all files.
    - Use all other defaults

10. Go to **Properties** for the bucket
11. Select **Static website hosting** 
    - Select `Use this bucket to host a website`
    - Enter `index.html` as your index document
    - Hit save

12. Update bucket cors policy in `Permissions` -> `CORS Configuration`
```
<CORSConfiguration>
 <CORSRule>
   <AllowedOrigin>*</AllowedOrigin>
   <AllowedMethod>GET</AllowedMethod>
   <MaxAgeSeconds>10000</MaxAgeSeconds>
 </CORSRule>
</CORSConfiguration>
```


##### 3.2 Setup Route53
1. Open route53 [console.aws.amazon.com/route53/](https://console.aws.amazon.com/route53/)
2. Create Hosted Zone
3. Add your domain in here without such as `www.endlessfonts.com`. Don't have one? Consider [name.com](https://www.name.com/referral/5a470) (Affiliate link.)

> You will need to update your domain registrar's name servers to match the `NS` records in route53. My records were:
```
ns-1770.awsdns-29.co.uk.
ns-203.awsdns-25.com.
ns-1111.awsdns-10.org.
ns-1011.awsdns-62.net.
```

4. Click **Create Record Set**
5. Under `Type` choose `A --  IPv4 address`
6. Under `Alias` click `yes`
7. Under `Alias Target` navigate to your s3 bucket, it should be something like `s3-website.us-east-1.amazonaws.com` after you select your S3 bucket the the domain name.


##### 3.3 Create HTTPs Certificate in ACM
1. Open ACM [console.aws.amazon.com/acm/](https://console.aws.amazon.com/acm/)
2. Ensure in the top right it says 'N. Virginia' for the `us-east-1` region. (`us-east-1` will also be in the url)
3. Click 'Getting started' or 'Request a certificate'
4. Click 'Request a public certificate'
5. Add your domain name and finish setup steps



##### 3.4 Setup Cloudfront
1. Open Cloudfront [console.aws.amazon.com/cloudfront/](https://console.aws.amazon.com/cloudfront/)
2. Click "Create Distribution"
3. Under "web", click "Get Started"
4. Under `Origin Domain Name` look for your s3 bucket, mine was `www.endlessfonts.com.s3.amazonaws.com`
5. For `Origin ID`, I used `www.endlessfonts.com`
6. `Viewer Protocol Policy`: `Redirect HTTP to HTTPS`
7. `Alternate Domain Names(CNAMEs)`: Use your domain `www.endlessfonts.com`
8. `SSL Certificate`:
    - Select `Custom SSL Certificate`
    - Find your certificate you just created in ACM; something like `www.endlessfonts.com (fc7c41bc-49c7-4730-a8c5-dd8df409a1c9)`

9. `Default Root Object`: `index.html` (like we renamed a few steps ago)

10. Click `Create distribution`
11. Back in [route53](https://console.aws.amazon.com/route53/), update your `Alias Target` we set in `Step 3.2` to your cloudfront distrubtion. My value was `d32rfx3a5ksp8e.cloudfront.net`
12. Wait some time for the distribution to be created. 


### 4. Use it.

Our css is located at `https://www.endlessfonts.com/endless.css`. You can usee it with:

```html
<link rel="stylesheet" type="text/css" href="https://www.endlessfonts.com/endless.css" />
```

Create and open a new `index.html` file and try it out. 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Endless Fonts // CDN Example</title>

    <link rel="stylesheet" type="text/css" href="https://www.endlessfonts.com/endless.css" />
</head>
<body>

<div>
    <i class='endless-git'></i>
</div>
</body>
</html>
```


### 5 - Next Step

Take a deeper dive into AWS with...

[![Dive into AWS Icon](https://static.codingforentrepreneurs.com/media/courses/aws/images/DriveIntoAWS-course.jpg)](https://www.codingforentrepreneurs.com/courses/aws)