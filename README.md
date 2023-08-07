# sphinx
sphinx 를 활용하여 markdown 문서를 web-doc 으로 생성, nginx container 로 deploy___





## 1) sphinx





### (1) sphinx container 생성



```sh
$ cd ~/song/sphinx/web-docs

# quickstart
$ docker run -d --rm --name sphinx \
    -v /home/song/song/sphinx/web-docs/docs:/docs \
    sphinxdoc/sphinx sleep 365d
    

# container내로 진입
$ docker exec -it sphinx bash


```



### (2) Sphinx project 생성

quickstart

```bash
$ cd ~/song/sphinx/web-docs

# quickstart
$ docker run -it --rm \
    -v /home/song/song/sphinx/web-docs/docs:/docs \
    sphinxdoc/sphinx sphinx-quickstart


Welcome to the Sphinx 5.3.0 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: n

The project name will occur in several places in the built documentation.
> Project name: song docs
> Author name(s): ssongman
> Project release []: v1.0

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





# 확인

song@songgram:~/song/sphinx/web-docs/docs$ ll
total 36
drwxr-xr-x 5 song song 4096 Aug  2 23:09 ./
drwxr-xr-x 4 song song 4096 Aug  2 23:02 ../
-rw-r--r-- 1 root root  634 Aug  2 23:09 Makefile
drwxr-xr-x 2 root root 4096 Aug  2 23:09 _build/
drwxr-xr-x 2 root root 4096 Aug  2 23:09 _static/
drwxr-xr-x 2 root root 4096 Aug  2 23:09 _templates/
-rw-r--r-- 1 root root  969 Aug  2 23:09 conf.py
-rw-r--r-- 1 root root  443 Aug  2 23:09 index.rst
-rw-r--r-- 1 root root  800 Aug  2 23:09 make.bat





```





### (3) recommonmark 설치

markdown 사용하기 위해 recommonmark python 패키지 설치한다.

python-pip 패키지가 설치가 안되어 있을경우 설치를 먼저 진행합니다. 

```sh
$ sudo apt install python-pip

$ pip install recommonmark 

Successfully installed commonmark-0.9.1 recommonmark-0.7.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip available: 22.3 -> 23.2.1
[notice] To update, run: pip install --upgrade pip


```



#### conf.py 파일을 수정

```python
# for Sphinx-1.4 or newer
extensions = ['recommonmark']

```





### (4) 테마



#### rtd(read the doc)

```sh
$ pip install sphinx_rtd_theme


Successfully uninstalled docutils-0.19
Successfully installed docutils-0.18.1 sphinx_rtd_theme-1.2.2 sphinxcontrib-jquery-4.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip available: 22.3 -> 23.2.1
[notice] To update, run: pip install --upgrade pip


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



#### furo

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





#### sphinx-book-theme

좋아 보이는데 적용 되지 않음

```sh
# 1) 설치
$ pip install sphinx-book-theme


# 2) 테마 적용 conf.py
...
html_theme = 'sphinx_book_theme'
...


# 3) make html 실행
$ make html
```



#### piccolo_theme

왓구는 좋은데 너무 정적임

```sh
# 1) 설치
$ pip install piccolo-theme


# 2) 테마 적용 conf.py
...
html_theme = 'piccolo_theme'
...


# 3) make html 실행
$ make html
```





### (5) md 파일 생성



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

### (6) Tips

If you would like to install dependencies, use `sphinxdoc/sphinx` as a base image::

```dockerfile
# in your Dockerfile
FROM sphinxdoc/sphinx

WORKDIR /docs
ADD requirements.txt /docs
RUN pip3 install -r requirements.tx
```





## 2) nginx 



### (1) conf file 생성

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





### (2) docs directory 생성

```sh
# docs directory 생성
$ cd ~/song/sphinx/docs
$ mkdir docs

```



### (3) nginx 실행

```sh
# nginx 실행
$ docker run -d --name song-docs -p 8081:80 \
    -v /home/song/song/sphinx/web-docs/nginx/conf:/etc/nginx/conf.d \
    -v /home/song/song/sphinx/web-docs/docs/_build/html:/code \
    nginx:1.25


$ docker rm -f song-docs

```



