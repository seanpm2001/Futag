# Оглавление

- [Оглавление](#оглавление)
  - [1. Описание](#1-описание)
  - [2. Установка](#2-установка)
  - [3. Примеры использования](#3-примеры-использования)
  - [4. Авторы](#4-авторы)
  - [5. Статьи и материалы](#5-статьи-и-материалы)
  - [6. Найденные ошибки](#6-найденные-ошибки)

# 1. Описание

FUTAG — это автоматизированный инструмент генерации фаззинг-целей для программных библиотек.
В отличие от обычных программ, программная библиотека может не содержать точки входа и не принимать входные данные, поэтому создание вручную фаззинг-цели для анализа программных библиотек остается проблемой и требует ресурсов. Одним из решением данной проблемы является автоматизация процесса создания фаззинг-целей, что уменьшает количество затрачиваемых ресурсов.
FUTAG использует инструменты Clang и Clang LLVM в качестве внешнего интерфейса для анализа библиотек и генерации фаззинг-целей.
FUTAG запускает статический анализ во время сборки библиотеки для поиска:
- Зависимостей сущностей (типы данных, функции, структуры и т.д.) в исходном коде целевой библиотеки.
- Контекста использования библиотеки.
Далее информация, полученная по результатам статического анализа, используется для генерации фаззинг-целей.

Данный проект основан на LLVM со статическим анализом Clang, а также LLVM lto и распространяется под лицензией ["GPL v3 license"](https://llvm.org/docs/DeveloperPolicy.html#new-llvm-project-license-framework)

В настоящее время Futag поддерживает:
- автоматическую сборку библиотеки с Makefile, cmake и configure;
- генерацию фаззинг-оберток для функций библиотек языка Си и Си++;


# 2. Установка
## 2.1. Использование докер-контейнера
Вы можете попробовать собрать Futag с готовыми [Докер-файлами](https://github.com/ispras/Futag/tree/main/product-tests/build-test) для ОС Ubuntu.

## 2.2. Использование предварительно упакованного пакета
- Загрузите последнюю версию [futag-llvm.1.3.tar.xz](https://github.com/ispras/Futag/releases/tag/1.3) и разархивируйте.

## 2.3. Сборка и установка из исходного кода

### 2.3.1. Зависимости
Данная инструкция позволяет собрать копию проекта и запустить её в Unix-подобной системе. 

Инструмент FUTAG основан на [LLVM-project](https://llvm.org/). Для компиляции проекта необходимо, чтобы следующие пакеты были установлены в вашей системе:

- [CMake](https://cmake.org/) >=3.13.4 [cmake-3.19.3-Linux-x86_64.sh](https://github.com/Kitware/CMake/releases/download/v3.19.3/cmake-3.19.3-Linux-x86_64.sh) - Makefile/workspace generator
- [GCC](https://gcc.gnu.org/)>=7.1.0 C/C++ compiler
- [Python](https://www.python.org/) >=3.8 Automated test suite.
- [pip](https://pypi.org/project/pip/) >=22.0.4
- [zlib](http://zlib.net/) >=1.2.3.4 Compression library
- [GNU Make](http://savannah.gnu.org/projects/make) 3.79, 3.79.1 Makefile/build processor

Для получения более детальной информации о зависимостях, необходимых для сборки LLVM, вы можете ознакомиться с документацией по указанной [ссылке](https://llvm.org/docs/GettingStarted.html#requirements)

Так же необходимо создать симбольную ссылку "python" на "python3" если такой ссылки не существует в вашей системе. В системе Ubuntu это можно сделать, установив пакет python-is-python3.

### 2.3.2. Сборка и установка

- Клонируйте проект:

```bash
  ~$ git clone https://github.com/ispras/Futag
```
- Подготовьте директорию "custom-llvm", запустив скрипт:
```bash
  ~/Futag/custom-llvm$ ./prepare.sh
```
Этот скрипт создает директорию Futag/build и копирует  в неё скрипт Futag/custom-llvm/build.sh

- Запустите скопированный скрипт в "Futag/build":

```bash
  ~/Futag/build$ ./build.sh
```

- В результате инструмент будет установлен в директорию Futag/futag-llvm

## 3. Примеры использования
- Предварительно должен быть установлен пакет futag-<версия>.tar.gz по пути futag-llvm/python-package/:
```bash
  ~$ pip install -r futag-llvm/python-package/requirements.txt
  ~$ pip install futag-llvm/python-package/futag-1.3.tar.gz
```

- Запуск сборки, проверки и анализа

```python
from futag.preprocessor import *

FUTAG_PATH = "/home/futag/Futag-tests/futag-llvm/"
lib_path = "path/to/library/source/code"
build_test = Builder(
    FUTAG_PATH,
    lib_path,
    clean=True, # удалить все папки сгенерированные Futag-ом перед сборкой
    # processes=4, # количество задач при сборке
    # build_ex_params="--with-openssl --with-mhash" # дополнительные параметры при сборке библиотеки
)
build_test.auto_build()
build_test.analyze()
```

- Генерация и компиляция драйверов

```python
from futag.generator import *

FUTAG_PATH = "/home/futag/Futag-tests/futag-llvm/"
lib_path = "path/to/library/source/code"

generator = Generator(
    FUTAG_PATH, # путь к директории "futag-llvm"
    lib_path, # путь к директории содержащей исходные кода исследуемого ПО
    # target_type=AFLPLUSPLUS, 
)

# Генерация фаззинг-оберток 
generator.gen_targets(
  anonymous=False # опция для генерации фаззинг-обертки для функций, которые не имеют публичный доступ
)
# Compile fuzz drivers
generator.compile_targets(
  4, # количество задач при сборке
  # keep_failed=True, # сохранить не скомпилированные цели
  # extra_include="-DHAVE_CONFIG_H", # дополнительные параметры при сборке библиотеки,
  # extra_dynamiclink="-lz", # системные библиотеки для линковки
  # flags="-ferror-limit=1", # значение по умолчанию: ""
)
```
По-умолчанию, успешно скомпилированные фаззинг-обертки для целевых функций находятся в каталоге futag-fuzz-drivers, где для каждой целевой функции создаётся своя поддиректория название которой совпадает с именем целевой функции. 
Если для функции сгенерировалось несколько фаззинг-оберток, в подкаталоге целевой функции создаются соответствующие директории, где к имени целевой функции добавляется порядковый номер.
Документация Python-пакета находится [по ссылке](https://github.com/ispras/Futag/tree/main/src/python/futag-package)

Подобную информацию о работе Futag можно прочитать [по ссылке](https://github.com/ispras/Futag/blob/main/How-to-work-with-Futag.md)

Шаблон скриптов запуска можно посмотреть [здесь](https://github.com/ispras/Futag/blob/main/src/python/template-script.py)

Был создан [репозиторий](https://github.com/thientc/Futag-tests) для тестирования Futag над библиотеками (json-c, php, FreeImage, и т.д.), можете протестировать с [Докер-контейнером](https://github.com/ispras/Futag/tree/main/product-tests/libraries-test).

## 4. Авторы

- [Чан Ти Тхиен](https://github.com/thientc/) (thientc@ispras.ru)
- Курмангалеев Шамиль (kursh@ispras.ru)
- Theodor Arsenij Larionov-Trichkin (tlarionov@ispras.ru)

## 5. Статьи и материалы

- C. T. Tran and S. Kurmangaleev, ["Futag: Automated fuzz target generator for testing software libraries"](https://ieeexplore.ieee.org/document/9693749) 2021 Ivannikov Memorial Workshop (IVMEM), 2021, pp. 80-85, doi: 10.1109/IVMEM53963.2021.00021.

- Исследования по автоматической генерации фаззинг-оберток для функций программных библиотек, Открытая конференция ИСП РАН им. В.П. Иванникова 2022

[![Видео](https://img.youtube.com/vi/qw_tzzgX04E/hqdefault.jpg)](https://www.youtube.com/watch?v=qw_tzzgX04E&t=28122s) 

## 6. Найденные ошибки

- Крэш в функции [png_convert_from_time_t](https://github.com/glennrp/libpng/issues/362) библиотеки [libpng версии 1.6.37](https://github.com/glennrp/libpng) (подвержен)

- Global-buffer-overflow в функции [ErrorIDToName](https://github.com/leethomason/tinyxml2/issues/923) библиотеки tinyxml2 версии 9.0.0