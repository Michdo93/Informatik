# 🏭 Build-Systeme

Ein **Build-System** automatisiert alle Schritte vom Quelltext zum fertigen Ergebnis (Programm, Paket, Container-Image). Vor allem baut es **nur das neu, was sich geändert hat**, und macht den Build für alle im Team **reproduzierbar**.

<!-- TOC -->
## Inhaltsverzeichnis

- [Überblick](#überblick)
- [Make](#make)
- [CMake](#cmake)
- [Java: Maven und Gradle](#java-maven-und-gradle)
- [C# / .NET](#c--net)
- [Python](#python)
- [ROS 2: colcon](#ros-2-colcon)
- [Gute Praxis](#gute-praxis)
<!-- /TOC -->

## Überblick

| Ökosystem | Build-System / Werkzeug | Konfigurationsdatei |
| --- | --- | --- |
| C / C++ | **Make**, **CMake** (+ Ninja), Meson | `Makefile`, `CMakeLists.txt`, `meson.build` |
| Java | **Maven**, **Gradle** | `pom.xml`, `build.gradle(.kts)` |
| C# / .NET | **dotnet** CLI / MSBuild | `*.csproj` |
| Python | **pip** + Build-Backend (setuptools, hatchling, poetry), **uv** | `pyproject.toml` |
| JavaScript | **npm**/pnpm/yarn, Vite, webpack | `package.json` |
| ROS 2 | **colcon** (+ ament/CMake) | `package.xml`, `CMakeLists.txt`/`setup.py` |
| Container | **Docker** | `Dockerfile` |
| Rust / Go | **cargo** / **go build** | `Cargo.toml` / `go.mod` |

---

## Make

Der Klassiker (1976, Stuart Feldman). Eine **Regel** beschreibt: Ziel, Abhängigkeiten, Befehl. Make vergleicht **Zeitstempel** und führt den Befehl nur aus, wenn eine Abhängigkeit neuer ist als das Ziel.

```make
# Makefile - recipe lines MUST start with a TAB, not spaces!
CC      := gcc
CFLAGS  := -Wall -Wextra -O2 -g
LDLIBS  := -lm
OBJ     := main.o sensor.o

sensor: $(OBJ)
	$(CC) $(OBJ) $(LDLIBS) -o $@

%.o: %.c sensor.h
	$(CC) $(CFLAGS) -c $< -o $@

.PHONY: clean
clean:
	rm -f $(OBJ) sensor
```

`$@` = Ziel, `$<` = erste Abhängigkeit. Make wird auch gern als einfacher **Aufgaben-Starter** in beliebigen Projekten genutzt (`make test`, `make deploy`).

---

## CMake

CMake ist ein **Meta-Build-System**: Es erzeugt aus einer plattformunabhängigen Beschreibung die eigentlichen Build-Dateien (Makefiles, Ninja, Visual-Studio-Projekte). Standard in C++-Projekten und in **ROS**.

```cmake
cmake_minimum_required(VERSION 3.20)
project(sensor_reader LANGUAGES C)

set(CMAKE_C_STANDARD 11)

find_package(PkgConfig REQUIRED)
pkg_check_modules(PAHO REQUIRED paho-mqtt3c)   # find an installed library

add_executable(sensor_reader src/main.c src/sensor.c)
target_include_directories(sensor_reader PRIVATE include ${PAHO_INCLUDE_DIRS})
target_link_libraries(sensor_reader PRIVATE ${PAHO_LIBRARIES} m)
target_compile_options(sensor_reader PRIVATE -Wall -Wextra)

install(TARGETS sensor_reader DESTINATION bin)
```

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release   # configure (out-of-source)
cmake --build build                                       # build
ctest --test-dir build                                    # run tests
sudo cmake --install build                                # install
```

> **Out-of-Source-Build:** Alle erzeugten Dateien landen in `build/` – der Quellordner bleibt sauber, und `build/` gehört in die `.gitignore`.

---

## Java: Maven und Gradle

```bash
mvn package            # compile, test, package to target/*.jar
gradle build           # or ./gradlew build (wrapper - preferred)
```

Beide laden **Abhängigkeiten automatisch** aus Maven Central. Der **Gradle-Wrapper** (`gradlew`) gehört ins Repo, damit alle dieselbe Gradle-Version nutzen.

---

## C# / .NET

```bash
dotnet new console -n LabTool
dotnet add package MQTTnet
dotnet build
dotnet run
dotnet publish -c Release -r linux-arm64 --self-contained   # for a Raspberry Pi
```

---

## Python

Python braucht für reinen Python-Code keinen Compiler-Build, aber ein **Packaging**:

```toml
# pyproject.toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "benq-control"
version = "0.3.0"
requires-python = ">=3.10"
dependencies = ["paho-mqtt>=2.0"]

[project.optional-dependencies]
dev = ["pytest", "ruff"]

[project.scripts]
benq-control = "benq_control.__main__:main"
```

```bash
python -m venv .venv && . .venv/bin/activate
pip install -e ".[dev]"       # editable install for development
python -m build               # creates dist/*.whl and dist/*.tar.gz
```

---

## ROS 2: colcon

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select my_robot_driver
source install/setup.bash
```

---

## Gute Praxis

* [ ] **Ein Befehl** baut das Projekt – dokumentiert in der README.
* [ ] **Versionen festlegen** (Lockfiles: `package-lock.json`, `poetry.lock`/`uv.lock`, Gradle-Wrapper, Docker-Image-Tags).
* [ ] Build-Ergebnisse (`build/`, `dist/`, `target/`, `*.o`) **nicht** committen.
* [ ] Warnungen einschalten (`-Wall -Wextra`) und ernst nehmen.
* [ ] Build im **CI** (GitHub Actions) automatisch bei jedem Push.
* [ ] Für andere Architekturen: [Cross-Compiling](Cross-Compiling.md).

---
