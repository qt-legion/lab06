[![Build Status](https://travis-ci.com/qt-legion/lab06)](https://travis-ci.org/qt-legion/lab06)
## Laboratory work VI

Данная лабораторная работа посвещена изучению средств пакетирования на примере **CPack**

```sh
$ open https://cmake.org/Wiki/CMake:CPackPackageGenerators
```

## Tasks

- [x] 1. Создать публичный репозиторий с названием **lab06** на сервисе **GitHub**
- [x] 2. Выполнить инструкцию учебного материала
- [x] 3. Ознакомиться со ссылками учебного материала
- [x] 4. Составить отчет и отправить ссылку личным сообщением в **Slack**

## Tutorial

```sh
$ export GITHUB_USERNAME=<имя_пользователя>
$ export GITHUB_EMAIL=<адрес_почтового_ящика>
$ alias edit=<nano|vi|vim|subl>
$ alias gsed=sed # for *-nix system
```

```sh
$ cd ${GITHUB_USERNAME}/workspace
$ pushd .
$ source scripts/activate
```

```sh
$ git clone https://github.com/${GITHUB_USERNAME}/lab05 projects/lab06
$ cd projects/lab06
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
```

```sh
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_STRING "v\${PRINT_VERSION}")
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION\
  \${PRINT_VERSION_MAJOR}.\${PRINT_VERSION_MINOR}.\${PRINT_VERSION_PATCH}.\${PRINT_VERSION_TWEAK})
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_TWEAK 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_PATCH 0)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MINOR 1)
' CMakeLists.txt
$ gsed -i '/project(print)/a\
set(PRINT_VERSION_MAJOR 0)
' CMakeLists.txt
$ git diff
```

```sh
$ touch DESCRIPTION && edit DESCRIPTION
$ touch ChangeLog.md
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```

```sh
$ cat > CPackConfig.cmake <<EOF
include(InstallRequiredSystemLibraries)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF
set(CPACK_PACKAGE_CONTACT ${GITHUB_EMAIL})
set(CPACK_PACKAGE_VERSION_MAJOR \${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR \${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH \${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK \${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION \${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE \${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RESOURCE_FILE_LICENSE \${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README \${CMAKE_CURRENT_SOURCE_DIR}/README.md)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_RPM_PACKAGE_NAME "print-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "print")
set(CPACK_RPM_CHANGELOG_FILE \${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

set(CPACK_DEBIAN_PACKAGE_NAME "libprint-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
EOF
```

```sh
$ cat >> CPackConfig.cmake <<EOF

include(CPack)
EOF
```

```sh
$ cat >> CMakeLists.txt <<EOF

include(CPackConfig.cmake)
EOF
```

```sh
$ gsed -i 's/lab05/lab06/g' README.md
```

```sh
$ git add .
$ git commit -m"added cpack config"
$ git tag v0.1.0.0
$ git push origin master --tags
```

```sh
$ travis login --auto
$ travis enable
```

```sh
$ cmake -H. -B_build
$ cmake --build _build
$ cd _build
$ cpack -G "TGZ"
$ cd ..
```

```sh
$ cmake -H. -B_build -DCPACK_GENERATOR="TGZ"
$ cmake --build _build --target package
```

```sh
$ mkdir artifacts
$ mv _build/*.tar.gz artifacts
$ tree artifacts
```

## Report

```sh
$ popd
$ export LAB_NUMBER=06
$ git clone https://github.com/tp-labs/lab${LAB_NUMBER} tasks/lab${LAB_NUMBER}
$ mkdir reports/lab${LAB_NUMBER}
$ cp tasks/lab${LAB_NUMBER}/README.md reports/lab${LAB_NUMBER}/REPORT.md
$ cd reports/lab${LAB_NUMBER}
$ edit REPORT.md
$ gist REPORT.md
```

## Homework

После того, как вы настроили взаимодействие с системой непрерывной интеграции,</br>
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься</br>
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку [releases](https://github.com/tp-labs/lab06/releases)).</br>
Пакет должен содержать приложение _solver_ из [предыдущего задания](https://github.com/tp-labs/lab03#задание-1)
Таким образом, каждый новый релиз будет состоять из следующих компонентов:
- архивы с файлами исходного кода (`.tar.gz`, `.zip`)
- пакеты с бинарным файлом _solver_ (`.deb`, `.rpm`, `.msi`, `.dmg`)

В качестве подсказки:
```sh
$ cat .travis.yml
os: osx
script:
...
- cpack -G DragNDrop # dmg

$ cat .travis.yml
os: linux
script:
...
- cpack -G DEB # deb

$ cat .travis.yml
os: linux
addons:
  apt:
    packages:
    - rpm
script:
...
- cpack -G RPM # rpm

$ cat appveyor.yml
platform:
- x86
- x64
build_script:
...
- cpack -G WIX # msi
```

##Повторяем все команды из туториала, заменяя конфигурацию и описание на solver
В конечном итоге получился вот такой CMakeLists:

```
cmake_minimum_required(VERSION 3.4)
project(formatter)
set(SOLV_VERSION_MAJOR 0)
set(SOLV_VERSION_MINOR 1)
set(SOLV_VERSION_PATCH 0)
set(SOLV_VERSION_TWEAK 0)
set(SOLV_VERSION
  ${SOLV_VERSION_MAJOR}.${SOLV_VERSION_MINOR}.${SOLV_VERSION_PATCH}.${SOLV_VERSION_TWEAK})
set(SOLV_VERSION_STRING "v${SOLV_VERSION}")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter STATIC formatter_lib/formatter.cpp)
include_directories(formatter_lib)

add_library(formatter_ex STATIC formatter_ex_lib/formatter_ex.cpp)
target_link_libraries(formatter_ex formatter)
include_directories(formatter_ex_lib)

add_executable(hello_world hello_world_application/hello_world.cpp)
target_link_libraries(hello_world formatter formatter_ex)

add_library(solver_lib STATIC solver_lib/solver.cpp)
include_directories(solver_lib)

add_executable(solver solver_application/equation.cpp)
target_link_libraries(solver formatter formatter_ex solver_lib)

include(CPackConfig.cmake)
```
И вот такой CPackConfig.cmake

```
include(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_CONTACT andrey.n190402@gmail.com)
set(CPACK_PACKAGE_VERSION_MAJOR ${SOLV_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${SOLV_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${SOLV_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${SOLV_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${SOLV_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing")

set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)
set(CPACK_RPM_PACKAGE_NAME "solv-devel")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "solv")
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
set(CPACK_DEBIAN_PACKAGE_NAME "solv-dev")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

include(CPack)
```
Выполняем сборку проекта и собираем архив
```
$ cmake -H. -B_build
$ cmake --build _build
$ cd _build
$ cpack -G DEB
$ cd ..
```


После надо заполнить .travis.yml
Для получения кода используем travis encrypt (токен с правами repo)
```
language: cpp

compiler:
- gcc
- clang
os:
 - linux

jobs:
  include:
  - name: "all projects"
    script:
    - cmake -H. -B_build
    - cmake --build _build
  - name: "each CMakeLists.txt"

script:
- source ./script
- cd /home/travis/build/qt-legion/lab06/solver_application/_build
- cpack -G DEB
    

addons:
  apt:
    sources:
      - george-edison55-precise-backports
    packages:
      - cmake
      - cmake-data

deploy:
  provider: releases
  api_key:
    secure: "Sk7fd3LKIszIxzNsL1bvvJkTryYAVeIzxg6te2yLfhsJ7L9yVwa+6DCztL3NsDjGw2KYBmRZGlUhsnD74uk2SbhgwGkMvh/csRnjROPqPy1hoAxWnkmdiLU1VBjSN/Ngk8i37lOGgmUYPdz51i8beVmVTEfRy7GZcUKjQzyz+J5OY+88vWTa5nbllVSg9ns+a/uS3fHJ+7MASG9/17HtsLSeCHD7axBX4FhclIldkKJDwbhpWYrGzgXbgi07sXcWF5pOHYT05pokm2OaXxa70YTSyIGbTeKkM3FYajt7HK+A++OrbF1S472uXqlJ/hUv3b8haCl2wpF1ndDXfXJY8fWmJjGcSgJCGkATrdF74awcpVGxCngKsutQOSVAnbn7Rng19DmegaeWc5Iw7p6wq6JA2gVDTPnqLvMO6q7tu7rA3aH9q31KMryjQ5fDXnp3tJB8PiyY4PE+J1JOi0wbCivZbcl39LSugCBcNsczirkJWc9ZjN8/ptdjTqmIgrMUSEWXg4KEquZ4BJizqybwHZvGuQsg+IQmGVo2irnkWKZgy1P3LEWysigCuJ/wh5pc66iuPjIKSgb6u768ZfEIcvvOwFQ1Sx6ENK+OmBOkBdv0x9SfIqF1Jkf/WCe0LBjcwwng9dfvaaVF9VgjoLwbbB2udUDZf0lJVMqSA0NFdWE="
  file: "/home/travis/build/qt-legion/lab06/solver_application/_build/solver-0.1.0.0-Linux.deb"
  skip_cleanup: true
```

## Links

- [DMG](https://cmake.org/cmake/help/latest/module/CPackDMG.html)
- [DEB](https://cmake.org/cmake/help/latest/module/CPackDeb.html)
- [RPM](https://cmake.org/cmake/help/latest/module/CPackRPM.html)
- [NSIS](https://cmake.org/cmake/help/latest/module/CPackNSIS.html)

```
Copyright (c) 2015-2021 The ISC Authors
```
