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





## 6) Build



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





## 7) index.rst

RST(ReStructuredText) 파일은 주로 python 프로그래밍 커뮤니티에서 사용되는 기술문서파일 형식이다.



### sample

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





### 수정



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



