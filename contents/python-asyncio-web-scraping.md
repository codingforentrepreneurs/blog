---
title: Python Asyncio Web Scraping
slug: python-asyncio-web-scraping

publish_timestamp: March 28, 2018
url: https://www.codingforentrepreneurs.com/blog/python-asyncio-web-scraping/

---


A lot of Python programs are Synchronous, meaning all functions happen sequentially, in order, and in a way that seems to be intuitive, especially for a new programmer. 

Synchronous is like "hey program do A and then B and then C". The program does "A", once "A" is done, it does "B", once B is done, it does "C". 

Chances are good if you've been working in Python you have written Synchronous programs.

Enter Asynchronus.

This is a different ball game all together. Programs still execute in order but can execute despite what's going on in the main thread.

What that means is when A is running, B can also be running, and so can C. If B or C relies on A then B or C can wait on A to finish before running.

This allows for potentially much faster programs and, of course, the potential to overwork your computer.

Here's look at a more practical example:

Let's say you have to compress a folder of videos for uploading to a webserver. 

Your synchronous program will compress video 1, then video 2, and finally video 3. Then it will upload video 1, video 2, and finally video 3. The order is important here as it implies this program is running processes one after another.

Your asynchronous program will compress video 1 AND video 2 AND video 3 at the same time. Once *any* of the videos are done compressing they immediately start to upload to the web server. Doing multiple things at once in a program can have serious impact on how well it works.

In other words, using async the time savings could be huge. 

Let's assume in both Synchronous and Asynchronous programs that
- Video 1 takes 10 minutes to process; 2 minutes to upload
- Video 2 takes 25 minutes to process; 3 minutes to upload
- Video 3 takes 12 minutes to process: 2 minutes to upload


Synchronous timeline
- 10 minutes; video 1 processed
- 35 minutes: video 2 processed
- 47 minutes: video 3 processed
- 49 minutes: video 1 uploaded
- 52 minutes: video 2 uploaded
- 54 minutes: video 3 uploaded
- Program completes in 54 minutes.

Asynchronous timeline
- 10 minutes; video 1 processed
- 12 minutes: video 1 uploaded, video 3 processed
- 14 minutes: video 3 uploaded
- 25 minutes: video 2 processed
- 28 minutes: video 2 uploaded
- Program completes in 28 minutes


Now, the math is not going to be this nice because processing 3 videos at once vs just one video at a time, will likely slow down your computer. But, if you have a powerful enough computer, the speed difference might not be noticeable.

Computers, as well as servers, have had major speed increases which is why I'd argue it's one of the reasons doing Async with Python has become so popular. Further, open source packages like `asyncio` have made doing async with python simple.


### Web Scraping, A Practical Example.
Below we'll go through this process with web scraping to extract text from a blog post. For more context, consider watching the related video.

#### Requirements
- Python 3.6 +




##### Install requirements
The below syntax is for `jupyter`, just remove `!` if you install via Terminal/PowerShell/Command Prompt.
```python
!pip install aiohttp aiodns requests BeautifulSoup4
```

### First, the Synchronous Way
```python
import re
import requests
from bs4 import BeautifulSoup  # https://www.crummy.com/software/BeautifulSoup/bs4/doc/
from urllib.parse import urlparse
url = "https://tim.blog/" # the blog we'll scrape
```


```python
import requests
from bs4 import BeautifulSoup
from collections import Counter
import re
from urllib.parse import urlparse
import time

content_area_class = 'entry-content' # the name of the div class for any given blog post

def fetch_url(url):
    r = requests.get(url)
    return r.text

def soup_a(html, display_result=False):
    soup = BeautifulSoup(html, 'html.parser')
    if display_result:
        print(soup.prettify())
    return soup

def extracting_text(html):
    soup = soup_a(html)
    content_area = soup.find("div", {"class": content_area_class})
    text = content_area.text
    return text


blogpost_pattern = r'^/(?P<year>\d+){4}/(?P<month>\d+){2}/(?P<day>\d+){2}/(?P<slug>[\w-]+)/$'

def matching_path(path):
    pattern = re.compile(blogpost_pattern)
    matching = pattern.match(path)
    if  matching is None:
        return False, None
    return True, matching


def processing_string(string):
    string = string.replace("\n", " ")
    string = string.replace("—", "")
    string = string.replace("|", "")
    string = string.replace("\n", " ")
    string = string.replace("\"", "")
    string = string.replace("\xa0", " ")
    string = string.replace("“", "")
    string = string.replace("”", "")
    string = string.replace("–", "")
    string = string.replace(".", "")
    string = string.replace("?", "")
    string = string.replace(",", "")
    string = string.replace("'ve", " have")
    #string = string.replace(".", " || PERIOD ||")
    #string = string.replace("?", " || QUESTION_MARK ||")
    return string.strip()


def extracting_local_links(html, root_domain, local_only=True):
    soup = soup_a(html)
    href_tags = soup.find_all("a", href=True)
    links = [a['href'] for a in href_tags]
    if local_only:
        paths = []
        for x in links:
            if urlparse(x).netloc == root_domain:
                link = urlparse(x).path
                is_matching = matching_path(link)
                if is_matching[0]:
                    paths.append(link)
    return list(set(paths))



def main_sync(url):
    start_time = time.time()
    parsed_url = urlparse(url)
    root_domain = parsed_url.netloc
    domain = parsed_url.geturl() # includes trailing /
    #print(domain)
    html = fetch_url(url)
    paths = extracting_local_links(html, root_domain)
    #print(paths)
    words = []
    for path in list(set(paths)):
        start_ = time.time()
        post_url  = domain + path
        new_html = fetch_url(post_url)
        text = extracting_text(new_html)
        string = processing_string(text)
        sub_words = string.split()
        words += sub_words
        print(time.time() - start_, "seconds")
    word_freq = Counter(words)
    print("Entire request took", time.time()-start_time, "seconds")
    print(word_freq.most_common(10))



main_sync('http://tim.blog')
```

### Now, the Asynchronous Way
```python
import aiohttp
import asyncio
import async_timeout
from bs4 import BeautifulSoup
from collections import Counter
import re
from urllib.parse import urlparse
import time

content_area_class = 'entry-content'

async def fetch(session, url):
    async with async_timeout.timeout(10):
        async with session.get(url) as response:
            return await response.text()

async def soup_d(html, display_result=False):
    soup = BeautifulSoup(html, 'html.parser')
    if display_result:
        print(soup.prettify())
    return soup

async def extract_text(html):
    soup = await soup_d(html)
    content_area = soup.find("div", {"class": content_area_class})
    text = content_area.text
    return text


blogpost_pattern = r'^/(?P<year>\d+){4}/(?P<month>\d+){2}/(?P<day>\d+){2}/(?P<slug>[\w-]+)/$'

async def match_path(path):
    pattern = re.compile(blogpost_pattern)
    matching = pattern.match(path)
    if  matching is None:
        return False, None
    return True, matching


async def process_string(string):
    string = string.replace("\n", " ")
    string = string.replace("—", "")
    string = string.replace("|", "")
    string = string.replace("\n", " ")
    string = string.replace("\"", "")
    string = string.replace("\xa0", " ")
    string = string.replace("“", "")
    string = string.replace("”", "")
    string = string.replace("–", "")
    string = string.replace(".", "")
    string = string.replace("?", "")
    string = string.replace(",", "")
    string = string.replace("'ve", " have")
    #string = string.replace(".", " || PERIOD ||")
    #string = string.replace("?", " || QUESTION_MARK ||")
    return string.strip()


async def extract_local_links(html, root_domain, local_only=True):
    soup = await soup_d(html)
    href_tags = soup.find_all("a", href=True)
    links = [a['href'] for a in href_tags]
    if local_only:
        paths = []
        for x in links:
            if urlparse(x).netloc == root_domain:
                link = urlparse(x).path
                is_matching = await match_path(link)
                if is_matching[0]:
                    paths.append(link)
    return list(set(paths))



async def main(url):
    async with aiohttp.ClientSession() as session:
        start_time = time.time()
        parsed_url = urlparse(url)
        root_domain = parsed_url.netloc
        domain = parsed_url.geturl() # includes trailing /
        #print(domain)
        html = await fetch(session, url)
        paths = await extract_local_links(html, root_domain)
        #print(paths)
        words = []
        for path in list(set(paths)):
            start_ = time.time()
            post_url  = domain + path
            new_html = await fetch(session, post_url)
            text = await extract_text(new_html)
            string = await process_string(text)
            sub_words = string.split()
            words += sub_words
            print(time.time() - start_, "seconds")
        word_freq = Counter(words)
        print("Entire request took", time.time()-start_time, "seconds")
        print(word_freq.most_common(10))



loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main('http://tim.blog'))
except:
    pass
```

There is no doubt all of this can be improved even further but the task shows clear results. Using async and Asyncio allowed us to scape a few web pages much faster than just synchronous loading. 

Do you have questions? Comments? Suggestions? Let us know below.
