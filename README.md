# sphinx
sphinx 를 활용하여 markdown 문서를 web-doc 으로 생성, nginx container 로 deploy





# 1. sphinx





## 1) sphinx Container 실행



```sh
$ cd ~/song/sphinx/web-docs


$ docker pull sphinxdoc/sphinx

# quickstart
$ docker run -d --rm --name sphinx \
    -v /root/song/sphinx/web-docs/docs:/docs \
    sphinxdoc/sphinx sleep 365d
    

# container내로 진입
$ docker exec -it sphinx bash


# 삭제시
$ docker rm -f sphinx
```



## 2) Sphinx project 생성

quickstart

```bash
$ cd ~/song/sphinx/web-docs

# container내로 진입
$ docker exec -it sphinx bash


# quickstart
root@e59e5fa52b44:/docs$ sphinx-quickstart


Welcome to the Sphinx 5.3.0 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: n

The project name will occur in several places in the built documentation.
> Project name: DevPilot Online Manual
> Author name(s): ktds
> Project release []: v0.5

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]: ko

Creating file /docs/conf.py.
Creating file /docs/index.rst.
Creating file /docs/Makefile.
Creating file /docs/make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file /docs/index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.



$ ls -ltr /docs
total 28
drwxr-xr-x 2 root root 4096 Aug  3 02:05 _templates
drwxr-xr-x 2 root root 4096 Aug  3 02:05 _static
drwxr-xr-x 2 root root 4096 Aug  3 02:05 _build
-rw-r--r-- 1 root root  800 Aug  3 02:05 make.bat
-rw-r--r-- 1 root root  482 Aug  3 02:05 index.rst
-rw-r--r-- 1 root root  974 Aug  3 02:05 conf.py
-rw-r--r-- 1 root root  634 Aug  3 02:05 Makefile

```





## 3) recommonmark 설치

markdown 사용하기 위해 recommonmark python 패키지 설치한다.

python-pip 패키지가 설치가 안되어 있을경우 설치를 먼저 진행한다.

```sh
# pip 가 없을때는 
$ sudo apt install python-pip

```



###  (1) python library install

사내망에서 python nexus repo 참조하도록 설정

```
$ mkdir -p ~/.config/pip
$ cat > ~/.config/pip/pip.conf
[global]
trusted-host=10.217.59.89
index=http://10.217.59.89/nexus3/repository/pypi-group/pypi
index-url=http://10.217.59.89/nexus3/repository/pypi-group/simple
```



### (2) recommonmark install

```sh

$ pip install recommonmark 

Successfully installed commonmark-0.9.1 recommonmark-0.7.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip available: 22.3 -> 23.2.1
[notice] To update, run: pip install --upgrade pip


```





### (3) conf.py 파일을 수정

* vi 가 없으면

```sh
$ apt-get update

$ apt-get install vim
```



* conf.py 파일 수정

```sh
$ vi conf.py

...
# for Sphinx-1.4 or newer
extensions = ['recommonmark']
...

```







## 4) 테마 선택 및 적용



참고링크 : https://sphinx-themes.org/

### (1) alabaster

default 값이며 가장 좋지 않은듯 하다.

title 부분이 같이 스크롤된다.





### (2) rtd(read the doc) - ( appdu )

```sh
$ pip install sphinx_rtd_theme

Installing collected packages: soupsieve, sphinx, beautifulsoup4, sphinx-basic-ng, furo
  Attempting uninstall: sphinx
    Found existing installation: Sphinx 5.3.0
    Uninstalling Sphinx-5.3.0:
      Successfully uninstalled Sphinx-5.3.0
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
sphinx-rtd-theme 1.2.2 requires sphinx<7,>=1.6, but you have sphinx 7.1.2 which is incompatible.
Successfully installed beautifulsoup4-4.12.2 furo-2023.7.26 soupsieve-2.4.1 sphinx-7.1.2 sphinx-basic-ng-1.0.0b2



```



테마 적용

```python
$ vi conf.py

import sphinx_rtd_theme
#html_theme = 'alabaster'
html_theme = 'sphinx_rtd_theme'
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
 
```



make html 실행

```
Running Sphinx v5.3.0
loading translations [ko]... done
loading pickled environment... done
building [mo]: targets for 0 po files that are out of date
building [html]: targets for 1 source files that are out of date
updating environment: 0 added, 0 changed, 0 removed
looking for now-outdated files... none found
preparing documents... done
writing output... [100%] index
generating indices... genindex done
writing additional pages... search done
copying static files... done
copying extra files... done
dumping search index in English (code: en)... done
dumping object inventory... done
build succeeded.

The HTML pages are in _build/html.

```



### (3) furo

이게 가장 좋아 보이네...

```sh
# 1) 설치
$ pip install furo


# 2) 테마 적용 conf.py
...
html_theme = 'furo'
...


# 3) make html 실행
$ make html
```





### (4) sphinx-book-theme ( arsenal )

좋아 보임

```sh
# 1) 설치
$ pip install sphinx-book-theme

Installing collected packages: typing-extensions, accessible-pygments, sphinx, pydata-sphinx-theme, sphinx-book-theme
  Attempting uninstall: sphinx
    Found existing installation: Sphinx 7.1.2
    Uninstalling Sphinx-7.1.2:
      Successfully uninstalled Sphinx-7.1.2
Successfully installed accessible-pygments-0.0.4 pydata-sphinx-theme-0.13.3 sphinx-6.2.1 sphinx-book-theme-1.0.1 typing-extensions-4.7.1



# 2) 테마 적용 conf.py
...
html_theme = 'sphinx_book_theme'
...


# 3) make html 실행
$ make html
```



### (5) piccolo_theme

왓구는 좋은데 너무 정적임

```sh
# 1) 설치
$ pip install piccolo-theme
Installing collected packages: piccolo-theme
Successfully installed piccolo-theme-0.16.1



# 2) 테마 적용 conf.py
...
html_theme = 'piccolo_theme'
...


# 3) make html 실행
$ make html
```







## 5) Arsenal 사례

### (1) sphinx build 사례

```groovy

            stage('Sphinx Build') {
                container('sphinx') {
                    print "start sphinx build"
                    sh "git clone http://git.cz-dev.container.kt.co.kr/arsenal-suite/arsenal-docs/sphinxdoc.git"
                    sh "sphinx-build -b html ./sphinxdoc/source ./html"
                    sh "rm -rf sphinxdoc"
                }
            }

```







### index.rst

```
.. arsenal_guide documentation master file, created by
   sphinx-quickstart on Fri Apr 30 16:46:48 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

DOverse 가이드
====================

.. note::
   업데이트 중

.. toctree::
   :maxdepth: 6
   :caption: 목차
   :name: mastertoc
   :numbered:
   :glob:

   guide/*
   
```





## 5) md 파일 생성



```
<!-- path/to/file_1.md -->

# Title

## My Subtitle
```





```
<!-- file_2.md -->

[My Subtitle][]

[My Subtitle]: <path/to/file_1:My Subtitle>

```

















Build HTML document::

```bash
$ docker run --rm -v /path/to/document:/docs sphinxdoc/sphinx make html
```

Build EPUB document::

```bash
$ docker run --rm -v /path/to/document:/docs sphinxdoc/sphinx make epub
```

Build PDF document::

```bash
$ docker run --rm -v /path/to/document:/docs sphinxdoc/sphinx-latexpdf make latexpdf
```

## 6) Tips

If you would like to install dependencies, use `sphinxdoc/sphinx` as a base image::

```dockerfile
# in your Dockerfile
FROM sphinxdoc/sphinx

WORKDIR /docs
ADD requirements.txt /docs
RUN pip3 install -r requirements.tx
```





# 2. nginx 



## 1) conf file 생성

```sh
# conf file 생성
$ cd ~/song/sphinx/web-docs
$ mkdir -p nginx/conf/

$ cat > nginx/conf/default.conf
server {
    listen       80 default_server;
    server_name  localhost _;
    index        index.html index.htm;
    root         /code;
    location / {
        autoindex on;
    }
}
---


```





## 2) nginx Container 실행

```sh

$ docker pull nginx:1.25



# nginx 실행
$ docker run -d --name song-docs -p 8082:80 \
    -v /root/song/sphinx/web-docs/nginx/conf:/etc/nginx/conf.d \
    -v /root/song/sphinx/web-docs/docs/_build/html:/code \
    nginx:1.25


# 삭제
$ docker rm -f song-docs

```



