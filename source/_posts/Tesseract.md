---
title: Tesseract
date: 2018-06-26 21:02:15
tags:
---
## Install

#### 1. Required package
```bash
apt install autoconf-archive automake g++ libtool make pkg-config # for main recognization
apt install asciidoc # for document
apt-get install libicu-dev libpango1.0-dev libcairo2-dev # for training tool
```

##### 1.1 Build libleptonica-dev from source
```bash
sudo -i # build as root
wget http://www.leptonica.org/source/leptonica-1.76.0.tar.gz # check http://www.leptonica.org/download.html for latest version
tar -xzf leptonica-1.76.0.tar.gz
cd leptonica-1.76.0/
./configure
make
make install
```



#### 2. Clone tesseract from github
```bash
git clone https://github.com/tesseract-ocr/tesseract.git tesseract
```

#### 3. Build as non-root user
```bash
./autogen.sh
./configure --prefix=$HOME/tesseract/bin/
make
make install
```

#### 4. Build training tool
```bash
make training
make training-install
```

#### 5. Get trained data
Download required language from https://github.com/tesseract-ocr/tesseract/wiki/Data-Files