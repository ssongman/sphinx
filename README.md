# sphinx
sphinx 를 활용하여 markdown 문서를 web-doc 으로 생성, nginx container 로 deploy





# 1. sphinx



sphinx 는 소스파일을 읽어서 output(html 등) 을 만들어주는 tool 이다.

그러므로 이렇게 만들어진 html 을 읽을 수 있는 별도의 WAS (Nginx 등) 가 필요하다.

아래와 같이 sphinx Container 를 실행시켜 놓은 후 Build 환경을 만들어 보자.



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

### (1) quickstart

```bash
$ cd ~/song/sphinx/web-docs

# container내로 진입
$ docker exec -it sphinx bash


# quickstart
root@e59e5fa52b44:/docs$ sphinx-quickstart

Welcome to the Sphinx 6.2.1 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: y

The project name will occur in several places in the built documentation.
> Project name: DevPilot Manual
> Author name(s): ktds
> Project release []:

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> Project language [en]: ko

Creating file /docs/source/conf.py.
Creating file /docs/source/index.rst.
Creating file /docs/Makefile.
Creating file /docs/make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file /docs/source/index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.




$ ls -ltr /docs
drwxr-xr-x 2 root root 4096 Aug  7 04:42 build
drwxr-xr-x 4 root root 4096 Aug  7 04:42 source
-rw-r--r-- 1 root root  804 Aug  7 04:42 make.bat
-rw-r--r-- 1 root root  638 Aug  7 04:42 Makefile

$ cd source/
$ root@e59e5fa52b44:/docs/source# ll
total 16
drwxr-xr-x 2 root root 4096 Aug  7 04:42 _templates
drwxr-xr-x 2 root root 4096 Aug  7 04:42 _static
-rw-r--r-- 1 root root  916 Aug  7 04:42 conf.py
-rw-r--r-- 1 root root  461 Aug  7 04:42 index.rst

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


# markdown 설치 (md 파일 지원)
> pip install sphinx-markdown-tables --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org

> pip install recommonmark --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org

> pip install --upgrade myst-parser --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org





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



### (4) requirements



docs/requirements.txt

```sh
myst-parser
sphinx-copybutton
sphinx-design
sphinx-inline-tabs
sphinx-tabs

```









## 4) 테마 선택 및 적용



alabaster가 기본 테마 이름인데 이를 다른 이름으로 바꿀 수 있다. 별 다른 조치 없이 테마 이름만 변경해서 적용할 수 있는 것들을 builtin theme이라고 한다.




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



#### 추가 설정

참고 : https://pradyunsg.me/furo/customisation/

```python


html_theme_options = {
    "light_css_variables": {
        "color-brand-primary": "red",
        "color-brand-content": "#CC3333",
        "color-admonition-background": "orange",
    },
}

# sidebar
# document 의 Sidebar 에서 프로젝트의 이름을 보여줄지 제어한다.
# logo 를 보여주기를 원할때 유용하다. 기본값은 false 이다.
html_theme_options = {
    "sidebar_hide_name": True,
}



# light_css_variables
# 사용자가 원하는 색상으로 변경 가능
html_theme_options = {
    "light_css_variables": {
        "color-brand-primary": "#7C4DFF",
        "color-brand-content": "#7C4DFF",
    },
}

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





## 5) Build



### md 파일 적용

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





### Build



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





### Tips: Dockerfile

If you would like to install dependencies, use `sphinxdoc/sphinx` as a base image::

```dockerfile
# in your Dockerfile
FROM sphinxdoc/sphinx

WORKDIR /docs
ADD requirements.txt /docs
RUN pip3 install -r requirements.tx
```





## 6) index.rst

RST(ReStructuredText) 파일은 주로 python 프로그래밍 커뮤니티에서 사용되는 기술문서파일 형식이다.



### (1) sample

#### sample1

```rst
.. DevPilot Online Manual documentation master file, created by
   sphinx-quickstart on Thu Aug  3 02:05:52 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

DevPilot Online Manual
==================================================

.. note::
   updating...

.. toctree::
   :maxdepth: 6
   :caption: 목차
   :name: mastertoc
   :numbered:
   :glob:

   guide/*

```



#### sample2

```rst
제목
====

이 문서는 Sphinx Furo 테마를 이용하여 생성되었습니다.

코드 블록 예제
------------

아래는 Python 코드 블록의 예제입니다.

.. code-block:: python

    def greet(name):
        """인사하는 함수"""
        return f"안녕하세요, {name}!"

    print(greet("홍길동"))

리스트
------

- 항목 1
- 항목 2
- 항목 3

링크
----

`Furo 테마 공식 홈페이지 <https://pradyunsg.me/furo/>`_
```





#### sample3

```rst
Sample reStructuredText Document
===============================

This is a sample reStructuredText (rst) document demonstrating various
elements you can use in rst.

Section Title
-------------

This is a subsection under the main section. You can use different levels
of section titles using equal signs or dashes.

Lists
-----

- Bullet point item 1
- Bullet point item 2
- Bullet point item 3

1. Numbered item 1
2. Numbered item 2
3. Numbered item 3

Code Block
----------

You can display code blocks like this:

.. code-block:: python

   def greet(name):
       return f"Hello, {name}!"

   print(greet("John"))

Links
-----

You can create links to other websites or documents:

`Sphinx Official Website <https://www.sphinx-doc.org/>`_

Inline Markup
-------------

You can add emphasis, strong text, literals, and more:

*This text is in italics.*
**This text is in bold.**
``Monospace text.``
``Literal text.``
​````python
def add(x, y):
    return x + y
```





### (2) 수정



```sh

# container내로 진입
$ docker exec -it sphinx bash

 
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
    -v /root/song/sphinx/web-docs/docs/build/html:/code \
    nginx:1.25


# 삭제
$ docker rm -f song-docs

```







# 3. DevPilot



## 1) Source File 



```sh
$ cd ~/song/ssphinx/web-docs

# container내로 진입
$ docker exec -it sphinx bash

```



### git clone

```sh


# git clone
$ mkdir -p /song

$ cd /song

# git clone -b <branchname> <remote-repo-url>
$ git clone http://gitlab.dev.icis.kt.co.kr/sa/arsenal_docs.git

# root / n******

# 확인
$ ll ./arsenal_docs
-rw-r--r-- 1 root root  764 Aug  7 04:13  make.bat
-rw-r--r-- 1 root root  638 Aug  7 04:13  Makefile
-rw-r--r-- 1 root root 3110 Aug  7 04:13 '스핑크스 설치.md'
drwxr-xr-x 3 root root 4096 Aug  7 04:13  source


# pull
$ cd /song/arsenal_docs
$ git pull

```



### copy source files

```sh

# copy source files

$ cd /docs
# $ rm -r /docs/source/
$ rm -r /docs/source/guide

$ cp -r /song/arsenal_docs/source/guide /docs/source

```



## 2) index.md





### 참고 index.rst



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

리스트
------

- 항목 1
- 항목 2
- 항목 3

링크
----

`Furo 테마 공식 홈페이지 <https://pradyunsg.me/furo/>`_


```







### index.md

````

---
hide-toc: true
---

# DevPilot Manual

이 문서는 DevPilot Online Manual 입니다.

- 문서설명 시작
- 문서설명 종료



```{toctree}
:hidden:

guide/01.Arsenal 소개
guide/02.Container Transformation
guide/03.docker
guide/04.docker

```

```{toctree}
:caption: Development
:hidden:

kitchen-sink/index
```



====================================


```{toctree}
:hidden:

quickstart
customisation/index
reference/index
recommendations
```

```{toctree}
:caption: Development
:hidden:

contributing/index
kitchen-sink/index
stability
changelog
license
```


````







### _static file 설정







./source/_static/pied-piper-admonition.css

```css
body {
  --icon--pied-piper: url('data:image/svg+xml;charset=utf-8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 576 512"><path d="M244 246c-3.2-2-6.3-2.9-10.1-2.9-6.6 0-12.6 3.2-19.3 3.7l1.7 4.9zm135.9 197.9c-19 0-64.1 9.5-79.9 19.8l6.9 45.1c35.7 6.1 70.1 3.6 106-9.8-4.8-10-23.5-55.1-33-55.1zM340.8 177c6.6 2.8 11.5 9.2 22.7 22.1 2-1.4 7.5-5.2 7.5-8.6 0-4.9-11.8-13.2-13.2-23 11.2-5.7 25.2-6 37.6-8.9 68.1-16.4 116.3-52.9 146.8-116.7C548.3 29.3 554 16.1 554.6 2l-2 2.6c-28.4 50-33 63.2-81.3 100-31.9 24.4-69.2 40.2-106.6 54.6l-6.3-.3v-21.8c-19.6 1.6-19.7-14.6-31.6-23-18.7 20.6-31.6 40.8-58.9 51.1-12.7 4.8-19.6 10-25.9 21.8 34.9-16.4 91.2-13.5 98.8-10zM555.5 0l-.6 1.1-.3.9.6-.6zm-59.2 382.1c-33.9-56.9-75.3-118.4-150-115.5l-.3-6c-1.1-13.5 32.8 3.2 35.1-31l-14.4 7.2c-19.8-45.7-8.6-54.3-65.5-54.3-14.7 0-26.7 1.7-41.4 4.6 2.9 18.6 2.2 36.7-10.9 50.3l19.5 5.5c-1.7 3.2-2.9 6.3-2.9 9.8 0 21 42.8 2.9 42.8 33.6 0 18.4-36.8 60.1-54.9 60.1-8 0-53.7-50-53.4-60.1l.3-4.6 52.3-11.5c13-2.6 12.3-22.7-2.9-22.7-3.7 0-43.1 9.2-49.4 10.6-2-5.2-7.5-14.1-13.8-14.1-3.2 0-6.3 3.2-9.5 4-9.2 2.6-31 2.9-21.5 20.1L15.9 298.5c-5.5 1.1-8.9 6.3-8.9 11.8 0 6 5.5 10.9 11.5 10.9 8 0 131.3-28.4 147.4-32.2 2.6 3.2 4.6 6.3 7.8 8.6 20.1 14.4 59.8 85.9 76.4 85.9 24.1 0 58-22.4 71.3-41.9 3.2-4.3 6.9-7.5 12.4-6.9.6 13.8-31.6 34.2-33 43.7-1.4 10.2-1 35.2-.3 41.1 26.7 8.1 52-3.6 77.9-2.9 4.3-21 10.6-41.9 9.8-63.5l-.3-9.5c-1.4-34.2-10.9-38.5-34.8-58.6-1.1-1.1-2.6-2.6-3.7-4 2.2-1.4 1.1-1 4.6-1.7 88.5 0 56.3 183.6 111.5 229.9 33.1-15 72.5-27.9 103.5-47.2-29-25.6-52.6-45.7-72.7-79.9zm-196.2 46.1v27.2l11.8-3.4-2.9-23.8zm-68.7-150.4l24.1 61.2 21-13.8-31.3-50.9zm84.4 154.9l2 12.4c9-1.5 58.4-6.6 58.4-14.1 0-1.4-.6-3.2-.9-4.6-26.8 0-36.9 3.8-59.5 6.3z"/></svg>');
}
.admonition.pied-piper {
  border-color: rgb(43, 155, 70);
}
.admonition.pied-piper > .admonition-title {
  background-color: rgba(43, 155, 70, 0.1);
  border-color: rgb(43, 155, 70);
}
.admonition.pied-piper > .admonition-title::before {
  background-color: rgb(43, 155, 70);
  -webkit-mask-image: var(--icon--pied-piper);
  mask-image: var(--icon--pied-piper);
}
```





./source/_static/readthedocs-dummy.js

```js
var READTHEDOCS_DATA = {
  project: "furo",
  version: "latest",
  language: "en",
  proxied_api_host: "https://readthedocs.org",
  features: {
    docsearch_disabled: true,
  },
};
```















## 3) build



재빌드 후 nginx 재기동

```sh

# Build
$ cd /docs/
  rm -rf build
  make html


# 삭제 nginx 실행
$ docker rm -f song-docs

  docker run -d --name song-docs -p 8082:80 \
    -v /root/song/sphinx/web-docs/nginx/conf:/etc/nginx/conf.d \
    -v /root/song/sphinx/web-docs/docs/build/html:/code \
    nginx:1.25


```









# 4. Arsenal 사례

## 1) sphinx jenkins pipeline

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







## 2) index.rst

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









# 5. PDF 문서 생

pdf 변환을 위해서는 반드시 Latex 설치가 필요하다.

latex 설치



## 1) 대상 확인

```


latexmk
texlive-latex-recommended
texlive-latex-extra
texlive-xetex
fonts-freefont-otf
texlive-fonts-recommended
texlive-lang-greek
tex-gyre

```



## 2) apt install

```sh

$ apt update

# install packages:
$ apt install fonts-freefont-otf latexmk python3-venv \
texlive-fonts-recommended texlive-latex-recommended \
texlive-latex-extra texlive-lang-greek \
tex-gyre texlive-xetex

```





## 3) conf.py 수정



PDF 문서를 생성하기 위해서는 각 프로젝트의 source/conf.py 파일을 수정해야 한다.

해당 파일을 열어 다음과 같은 항목을 추가 한다.



conf.py 수정

```python

...
html_theme = 'press'
...
html_static_path = ['_static']
...


# -- Options for LaTeX output ------------------------------------------------
latex_engine = 'xelatex'
latex_elements = {
    # The paper size ('letterpaper' or 'a4paper').
    #
    'papersize': 'a4paper',
 
    # The font size ('10pt', '11pt' or '12pt').
    #
    'pointsize': '10pt',
 
    # Additional stuff for the LaTeX preamble.
    #
    'preamble': '',
 
    'babel': ' ',
 
    # Latex figure (float) alignment
    #
    'figure_align': 'htbp',
 
 
    # kotex config
    'figure_align': 'htbp',
 
    'fontpkg': r'''
\usepackage{kotex}
 
% 영문 폰트 설정
\setmainfont[Mapping=tex-text]{나눔고딕}
\setsansfont[Mapping=tex-text]{나눔명조}
\setmonofont{D2Coding}
 
% 한글 폰트 설정
\setmainhangulfont[Mapping=tex-text]{나눔고딕}
\setsanshangulfont[Mapping=tex-text]{나눔명조}
\setmonohangulfont{D2Coding}
 
''',
}

```





## 4) build



```sh


$ make latexpdf



# 
$ make latexpdf LATEXMKOPTS="-silent"


$ ll /docs/_build/latex/songdocs.pdf
-rw-r--r-- 1 root root 8365 Aug  7 15:39 songdocs.pdf





```







```sh

# docker container 밖에서

$ ll /docs/_build/latex/songdocs.pdf


/home/song/song/sphinx/pdf-output

docker cp song-docs:/docs/_build/latex/songdocs.pdf /home/song/song/sphinx/pdf-output





```





## 9) trouble shooting



### kotex.sty file 없음

현상

```sh

$ make latexpdf
...
! LaTeX Error: File `kotex.sty' not found.

```

원인 및 조치

```sh


LINUX환경에서 kotex를 못찾는 문제입니다. 
> sudo yum install texlive-* 로 설치했구요 (texlive-full이 안되더라구요) 
overleaf에서는 컴파일 잘 되는데 리눅스 환경에서 kotex설치가 안되는 상황입니다.




# TeXLive 2013 설치
$ apt install texlive-* 


# 조치후 build

$ make latexpdf

```

