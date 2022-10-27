---
title: Download the MovieLens Dataset with Python
slug: download-the-movielens-dataset-with-python

publish_timestamp: Sept. 4, 2022
url: https://www.codingforentrepreneurs.com/blog/download-the-movielens-dataset-with-python/

---


In our [Django & Machine Learning: Recommender Course](https://www.codingforentrepreneurs.com/courses/recommender/) we use the MovieLens dataset as a basis for learning how to do Collaborative Filtering. This dataset is used often for this exact use case which is the reason we use it.

I wanted to make an easily repeatable way to download this dataset so I wrote this blog post as a reference for you to do so. 

The only dependencies are:

- Python 3
- Python [Requests](https://requests.readthedocs.io/en/latest/)


### Install Requirements
As always, I recommend you do this using a Python Virtual Environment (such as [venv](https://docs.python.org/3/tutorial/venv.html)):

```bash
python -m pip install requests --upgrade
```


### The Python Module: `movielens_dl.py`
```python
import argparse
import pathlib
import tempfile
from zipfile import ZipFile

import requests


MOVIELENS_URLS = {
    'latest': "http://files.grouplens.org/datasets/movielens/ml-latest.zip",
    'latest-small': "http://files.grouplens.org/datasets/movielens/ml-latest-small.zip"
}

def download_movielens(
        dest='movielens',
        package='latest-small',
        mkdir=True,
        verbose=False,
    ):
    url = MOVIELENS_URLS.get(package)
    if not url:
        raise Exception(f"Movie lens package: {package} was not found.")
    if verbose is True:
        print(f"Downloading from {url}")
    output_dir = pathlib.Path(dest).resolve()
    if not output_dir.exists():
        if mkdir:
            output_dir.mkdir(exist_ok=True)
        else:
            raise Exception(f"{output_dir} does not exist. Pass `mkdir=True`")
    with requests.get(url, stream=True) as r:
        r.raise_for_status()
        total_size_in_bytes= int(r.headers.get('content-length', 0))
        with tempfile.NamedTemporaryFile(mode='rb+') as temp_f:
            downloaded = 0
            dl_iteration = 0
            chunk_size = 8192
            total_chunks = total_size_in_bytes / chunk_size if total_size_in_bytes else 100
            for chunk in r.iter_content(chunk_size=chunk_size):
                if verbose is True:
                    downloaded += chunk_size
                    dl_iteration += 1
                    percent = (100 * dl_iteration * 1.0/total_chunks)
                    if dl_iteration % 10 == 0 and percent < 100:
                        print(f'Completed {percent:2f}%')
                    elif percent >= 99.9:
                        print(f'Download completed. Now unzipping...')
                temp_f.write(chunk)
            with ZipFile(temp_f, 'r') as zipf:
                zipf.extractall(output_dir)
                if verbose is True:
                    print(f"\n\nUnzipped.\n\nFiles downloaded and unziped to:\n\n{dest.resolve()}")


def setup_args():
    parser = argparse.ArgumentParser(description='Download movielens')
    parser.add_argument('path', default='movielens', type=pathlib.Path, nargs='?', help='Write the download path')
    parser.add_argument('--verbose', default=False, action='store_true')
    parser.add_argument('--package', default='latest-small', type=str)
    parser.add_argument('--mkdir', default=False, action='store_true')
    return parser.parse_args()

if __name__ == "__main__":
    args = setup_args()
    path = args.path
    verbose = args.verbose
    package = args.package
    mkdir = args.mkdir
    download_movielens(path, mkdir=mkdir, package=package, verbose=verbose)
```

### Usage
```bash
python movielens_dl.py datasets  --package latest-small --verbose --mkdir
```
