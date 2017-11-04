[![Build Status](https://travis-ci.org/n0k8t/lab10.svg?branch=master)](https://travis-ci.org/n0k8t/lab10)
the demo application redirects data from stdin to a file **log.txt** using a package **print**.

## Laboratory work X

Данная лабораторная работа посвещена изучению систем управления пакетами на примере **Hunter**

```ShellSession
$ open https://github.com/ruslo/hunter
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab10** на сервисе **GitHub**
- [x] 2. Сгенирировать токен для доступа к сервису **GitHub** с правами **repo**
- [x] 3. Выполнить инструкцию учебного материала
- [x] 4. Ознакомиться со ссылками учебного материала
- [x] 5. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```ShellSession
$ export GITHUB_USERNAME=n0k8t
$ export GITHUB_TOKEN=xxxxxxxxxxx
```
Install `hub` for github
```ShellSession
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
$ go get github.com/github/hub
```
Create configuration for `hub`
```ShellSession
$ mkdir ~/.config
$ cat > ~/.config/hub <<EOF
github.com:
- user: ${GITHUB_USERNAME}
  oauth_token: ${GITHUB_TOKEN}
  protocol: https
EOF
$ git config --global hub.protocol https
```
Get `SHA1` checksum for `v0.1.0.0.tar.gz package`
```ShellSession
$ wget https://github.com/${GITHUB_USERNAME}/lab09/archive/v0.1.0.0.tar.gz
$ export PRINT_SHA1=`openssl sha1 v0.1.0.0.tar.gz | cut -d'=' -f2 | cut -c2-41`
$ echo $PRINT_SHA1
cd4dd9635fc431795a5c1b96a35e41361a558d27

$ rm -rf v0.1.0.0.tar.gz
```
Fork from `ruslo/hunter`
```ShellSession
$ git clone https://github.com/ruslo/hunter projects/hunter
$ cd projects/hunter && git checkout v0.19.137
$ git remote show
origin

$ hub fork
$ git remote show
origin
pettro98

$ git remote show ${GITHUB_USERNAME}
* внешний репозиторий pettro98
  URL для извлечения: https://github.com/pettro98/hunter.git
  URL для отправки: https://github.com/pettro98/hunter.git
  HEAD ветка: master
  Внешние ветки:
    master              отслеживается
    pr.new.toolchain.id отслеживается
    testing-packages    отслеживается
  Локальная ссылка, настроенная для «git push»:
    master будет отправлена в master (уже актуальна)

```
Configure `hunter` for `print package`
```ShellSession
$ mkdir cmake/projects/print
$ cat > cmake/projects/print/hunter.cmake <<EOF
include(hunter_add_version)
include(hunter_cacheable)
include(hunter_cmake_args)
include(hunter_download)
include(hunter_pick_scheme)

hunter_add_version(  # add general information about package
    PACKAGE_NAME
    print
    VERSION
    "0.1.0.0"
    URL
    "https://github.com/${GITHUB_USERNAME}/lab09/archive/v0.1.0.0.tar.gz"
    SHA1
    ${PRINT_SHA1}
)

hunter_pick_scheme(DEFAULT url_sha1_cmake)  # sceme for cmake build only

hunter_cmake_args(
    print
    CMAKE_ARGS
    BUILD_EXAMPLES=NO
    BUILD_TESTS=NO
)
hunter_cacheable(print) # give permission to cache package
hunter_download(PACKAGE_NAME print) #add package to project
EOF
```
Set version to build
```ShellSession
$ cat >> cmake/configs/default.cmake <<EOF
hunter_config(print VERSION 0.1.0.0)
EOF

```
Commit and tag changes
```ShellSession
$ git add .
$ git commit -m"added print package"
$ git push ${GIHUB_USERNAME} master
$ git tag v0.19.137.1
$ git push ${GIHUB_USERNAME} master --tags
$ cd ..
```
init repository for `lab10`
```ShellSession
$ export HUNTER_ROOT=`pwd`/hunter
$ mkdir lab10 && cd lab10
$ git init
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab10
```
Add `demo.cpp` file
```ShellSession
$ mkdir sources
$ cat > sources/demo.cpp <<EOF
#include <print.hpp>

int main(int argc, char** argv) {
	std::string text;
	while(std::cin >> text) {
		std::ofstream out("log.txt", std::ios_base::app);
		print(text, out);
		out << std::endl;
	}
}
EOF
```
Extract `HunterGate.cmake`
```ShellSession
$ wget https://github.com/hunter-packages/gate/archive/v0.8.1.tar.gz 
$ tar -xzvf v0.8.1.tar.gz gate-0.8.1/cmake/HunterGate.cmake 
gate-0.8.1/cmake/HunterGate.cmake

$ mkdir cmake
$ mv gate-0.8.1/cmake/HunterGate.cmake cmake
$ rm -rf gate*/
$ rm *.tar.gz
```
Set minimal `CMakeLists` file
```ShellSession
$ cat > CMakeLists.txt <<EOF
cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_STANDARD 11)
EOF
```
Download created package and get `SHA1` checksum
```
$ wget https://github.com/${GITHUB_USERNAME}/hunter/archive/v0.19.137.1.tar.gz
$ export HUNTER_SHA1=`openssl sha1 v0.19.137.1.tar.gz | cut -d'=' -f2 | cut -c2-41`
$ echo $HUNTER_SHA1
1978ec9931279bd1b3389ea62af3abec4241ab99

$ rm -rf v0.19.137.1.tar.gz
```
Add `HunterGate module`
```ShellSession
$ cat >> CMakeLists.txt <<EOF

include(cmake/HunterGate.cmake)

HunterGate(
    URL "https://github.com/${GITHUB_USERNAME}/hunter/archive/v0.19.137.1.tar.gz"
    SHA1 "${HUNTER_SHA1}"
)
EOF
```
Set general project settings and add external packages
```ShellSession
$ cat >> CMakeLists.txt <<EOF

project(demo)

hunter_add_package(print)
find_package(print)

add_executable(demo \${CMAKE_CURRENT_SOURCE_DIR}/sources/demo.cpp)
target_link_libraries(demo print)

install(TARGETS demo RUNTIME DESTINATION bin)
EOF
```
Set github to ignore files and directories
```ShellSession
$ cat > .gitignore <<EOF
*build*/
*install*/
*.swp
EOF
```
Add TravisCI buld sign to readme file
```ShellSession
$ cat > README.md <<EOF
[![Build Status](https://travis-ci.org/${GITHUB_USERNAME}/lab10.svg?branch=master)](https://travis-ci.org/${GITHUB_USERNAME}/lab10)
the demo application redirects data from stdin to a file **log.txt** using a package **print**.
EOF
```
Set travis configuration
```ShellSession
$ cat > .travis.yml <<EOF
language: cpp

script:   
- cmake -H. -B_build
- cmake --build _build
EOF
```
Check for errors in travis configuration
```ShellSession
$ travis lint

```
Push changes to remote origin
```ShellSession
$ git add .
$ git commit -m"first commit"
$ git push origin master
```
Enable travis in repository
```ShellSession
$ travis login --auto
$ travis enable
```
Build demo and test it
```ShellSession
$ cmake -H. -B_build -DCMAKE_INSTALL_PREFIX=_install
$ cmake --build _build --target install
$ mkdir artifacts && cd artifacts
$ echo "text1 text2 text3" | ../_install/bin/demo
$ cat log.txt
text1
text2
text3

```

## Report
make a report
```ShellSession
$ popd
$ export LAB_NUMBER=10
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gistup -m "lab${LAB_NUMBER}"
```

## Links

- [hub](https://hub.github.com/)
- [polly](https://github.com/ruslo/polly)
- [conan](https://conan.io)

```
Copyright (c) 2017 Братья Вершинины
```
