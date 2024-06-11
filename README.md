# Laba6
CPack - это инструмент упаковки скомпилированного проекта в платформо-зависимые пакеты и установщики.
После того, как вы настроили взаимодействие с системой непрерывной интеграции, обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься о создание пакетов для измениний, которые помечаются тэгами (см. вкладку releases). Пакет должен содержать приложение solver из предыдущего задания Таким образом, каждый новый релиз будет состоять из следующих компонентов:

архивы с файлами исходного кода (.tar.gz, .zip) пакеты с бинарным файлом solver (.deb, .rpm, .msi, .dmg)

Для этого нужно добавить ветвление в конфигурационные файлы для CI со следующей логикой: если commit помечен тэгом, то необходимо собрать пакеты (DEB, RPM, WIX, DragNDrop, ...) и разместить их на сервисе GitHub. (см. пример для Travi CI)

Клонируем репозиторий с 4 лабораторной
git clone https://github.com/${GITHUB_USERNAME}/laba4 laba6
cd laba6
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab06dzd
Cloning into 'lab06'...
remote: Enumerating objects: 327, done.
remote: Counting objects: 100% (327/327), done.
remote: Compressing objects: 100% (153/153), done.
remote: Total 327 (delta 145), reused 286 (delta 128), pack-reused 0
Receiving objects: 100% (327/327), 1.12 MiB | 5.27 MiB/s, done.
Resolving deltas: 100% (145/145), done.
Cmake в laba6
	cat > CMakeLists.txt <<EOF
	cmake_minimum_required(VERSION 3.4)
	project(lab06)
	set(CMAKE_CXX_STANDARD 11)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)

	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/solver_application)

	include(CPack.cmake)
	EOF
Cpack
	cat > CPack.cmake <<EOF
	include(InstallRequiredSystemLibraries)
	
	set(CPACK_PACKAGE_CONTACT eisnersandy2@gmail.com)
	set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
	set(CPACK_PACKAGE_NAME "reshala")
	set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ library")
	set(CPACK_PACKAGE_PACK_NAME "reshala-${PRINT_VERSION}")
	
	set(CPACK_SOURCE_INSTALLED_DIRECTORIES 
	  "${CMAKE_SOURCE_DIR}/solver_application; solver_application"
	  "${CMAKE_SOURCE_DIR}/solver_lib; solver_lib"
	  "${CMAKE_SOURCE_DIR}/formatter_ex_lib; formatter_ex_lib"
	  "${CMAKE_SOURCE_DIR}/formatter_lib; formatter_lib")
	
	set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)
	
	set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
	
	set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
	set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
	
	set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
	set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
	set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "reshaet")
	
	set(CPACK_GENERATOR "DEB;RPM")
	
	set(CPACK_RPM_PACKAGE_SUMMARY "solves equations")
	
	include(CPack)
	EOF
	
	git push origin master
Username for 'https://github.com': Ezh-Vkh
Password for 'https://Ezh-Vkh@github.com':
Enumerating objects: 327, done.
Counting objects: 100% (327/327), done.
Delta compression using up to 8 threads
Compressing objects: 100% (136/136), done.
Writing objects: 100% (327/327), 1.12 MiB | 1.55 MiB/s, done.
Total 327 (delta 145), reused 327 (delta 145), pack-reused 0
remote: Resolving deltas: 100% (145/145), done.
To https://github.com/Ezh-Vkh/lab06dzd
 * [new branch]      master -> master
Workflow
Файл - для выполнения build.

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
Файл - build and release

name: CMake Build and Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  build_and_release:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Configure Solver
        run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

      - name: Build Solver
        run: cmake --build ${{github.workspace}}/build

      - name: Build Package
        run: cmake --build ${{github.workspace}}/build --target package

      - name: Build Source Package
        run: cmake --build ${{github.workspace}}/build --target package_source

      - name: Make a Release
        uses: ncipollo/release-action@v1.14.0
        with:
          artifacts: |
            DEB
            RPM
            tar.gz
            zip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
	git tag -a v1.1.2 -m "v1.1.2"
	git add README.md
	git push origin v1.1.2
	Username for 'https://github.com': Ezh-Vkh
	Password for 'https://Ezh-Vkh@github.com':
	Enumerating objects: 328, done.
	Counting objects: 100% (328/328), done.
	Delta compression using up to 8 threads
	Compressing objects: 100% (137/137), done.
	Writing objects: 100% (328/328), 1.12 MiB | 2.12 MiB/s, done.
	Total 328 (delta 145), reused 327 (delta 145), pack-reused 0
	remote: Resolving deltas: 100% (145/145), done.
	To https://github.com/Ezh-Vkh/laba6dzd
	 * [new tag]         v1.1.2 -> v1.1.2
	git add preview.png
	git tag -a v1.1.4 -m "v1.1.2"
	git push origin v1.1.4
	Username for 'https://github.com': Ezh-Vkh
	Password for 'https://Ezh-Vkh@github.com':
	Enumerating objects: 1, done.
	Counting objects: 100% (1/1), done.
	Writing objects: 100% (1/1), 157 bytes | 157.00 KiB/s, done.
	Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
	To https://github.com/Ezh-Vkh/laba6dzd
	 * [new tag]         v1.1.4 -> v1.1.4
	git add README.md
	git tag -a v1.1.6 -m "New README"
	git push origin v1.1.6
	Username for 'https://github.com': Ezh-Vkh
	Password for 'https://Ezh-Vkh@github.com':
	Enumerating objects: 1, done.
	Counting objects: 100% (1/1), done.
	Writing objects: 100% (1/1), 168 bytes | 84.00 KiB/s, done.
	Total 1 (delta 0), reused 0 (delta 0), pack-reused 0
	To https://github.com/Ezh-Vkh/laba6dzd
	 * [new tag]         v1.1.6 -> v1.1.6
