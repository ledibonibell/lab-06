# lab-06
После того, как вы настроили взаимодействие с системой непрерывной интеграции, обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься о создание пакетов для измениний, которые помечаются тэгами (см. вкладку releases). Пакет должен содержать приложение solver из предыдущего задания

# Task 1

We create `CMakeLists.txt` for `formatter_ex_lib`, `formatter_lib` and `solver_application` respectively

`CMakeLists.txt` contents for `formatter_ex_lib`:
```
cmake_minimum_required(VERSION 3.4)

project(formatter_ex_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib formatter_lib_dir)

add_library(formatter_ex_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex.cpp)

target_include_directories(formatter_ex_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../formatter_lib )

target_link_libraries(formatter_ex_lib formatter_lib)
```

`CMakeLists.txt` contents for `formatter_lib`:
```
cmake_minimum_required(VERSION 3.4)

project(formatter_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter.cpp ${CMAKE_CURRENT_SOURCE_DIR}/formatter.h)
```

`CMakeLists.txt` contents for `solver_application`:
```
cmake_minimum_required(VERSION 3.4)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(/../formatter_ex_lib formatter_ex_lib_dir)

add_library(solver_lib /../solver_lib/solver.cpp /../solver_lib/solver.h)
add_executable(solver /equation.cpp)

target_include_directories(formatter_ex_lib PUBLIC /../formatter_lib /../formatter_ex_lib /../solver_lib)

target_link_libraries(solver formatter_ex_lib formatter_lib solver_lib)
```

Now let's create `CMakeLists.txt` to link all directories

`CMakeLists.txt` contents:
```
cmake_minimum_required(VERSION 3.4)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib formatter_ex_lib_dir)

add_library(solver_lib ${CMAKE_CURRENT_SOURCE_DIR}/solver_lib/solver.cpp)
add_executable(solver ${CMAKE_CURRENT_SOURCE_DIR}/solver_application/equation.cpp)

target_include_directories(formatter_ex_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_lib ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib ${CMAKE_CURRENT_SOURCE_DIR}/solver_lib)

target_link_libraries(solver formatter_ex_lib formatter_lib solver_lib)

install(TARGETS solver
    RUNTIME DESTINATION bin
)

include(CPackConfig.cmake)
```
![Снимок экрана от 2023-04-04 20-06-46](https://user-images.githubusercontent.com/125737299/232226523-f2b1ed95-a87b-44f8-bc74-5ace932617f4.png)


Let's create `CPackConfig.cmake` (packaging tool)

`CPackConfig.cmake` contents:
```
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT inkey.cherry@gmail.com)
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ app for solving quadratic equations")
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_IGNORE_FILES  "\\\\.cmake;/build/;/.git/;/.github/")

set(CPACK_SOURCE_INSTALLED_DIRECTORIES "${CMAKE_SOURCE_DIR}; /")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_NAME "solverapp-dev")
set(CPACK_DEBIAN_FILE_NAME "solver-${PRINT_VERSION}.deb")
set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "ledibonibell")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_GENERATOR "DEB")

include(CPack)
```
![Снимок экрана от 2023-04-04 20-06-24](https://user-images.githubusercontent.com/125737299/232226546-61de0eae-c7f8-4201-9a49-1743f9eccfdf.png)

Create LICENSE and DESCRIPTION for working our programm

We create two scripts `Action.yml` and `Release.yml` to package and run the program on the GitHub virtual machine

`Action.yml` contents:
```
name: CMake

on:
 push:
  branches: [master]
 pull_request:
  branches: [master]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Configure Solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build Solver
    run: cmake --build ${{github.workspace}}/build

$ git add Action.yml
$ git commit -m "Action"
$ git push origin master
**Вводим логин и токен

$ cat >> Release.yml << EOF
>EOF
$ nano Release.yml
```

`Release.yml` contents:
```
name: CMake

on:
 push:
   tags:
     - v**

permissions:
  contents: write

jobs: 

  build_packages_Linux:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Configure Solver
      run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

    - name: Build Solver
      run: cmake --build ${{github.workspace}}/build

    - name: Build package
      run: cmake --build ${{github.workspace}}/build --target package

    - name: Build source package
      run: cmake --build ${{github.workspace}}/build --target package_source

    - name: Make a release
      uses: ncipollo/release-action@v1.10.0
      with:
        artifacts: "build/*.deb,build/*.tar.gz,build/*.zip"
        replacesArtifacts: false
        GITHUB_TOKEN: ${{ secrets.GH_PAT }}
        allowUpdates: true
```
![Снимок экрана от 2023-04-04 20-06-07](https://user-images.githubusercontent.com/125737299/232226557-eadff437-d4bd-42f0-9b8f-78f318138f4c.png)


After we create a release `v1.0.0.0`






