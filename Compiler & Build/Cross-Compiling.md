# 🔀 Cross-Compiling

**Cross-Compiling** bedeutet, ein Programm auf einem Rechner (**Host**) für einen **anderen** Rechnertyp (**Target**) zu bauen – z. B. auf dem x86-64-Laptop für einen ARM-basierten Raspberry Pi oder einen Mikrocontroller.

<!-- TOC -->
## Inhaltsverzeichnis

- [Warum?](#warum)
- [Begriffe](#begriffe)
- [C/C++ mit GCC-Cross-Toolchain](#cc-mit-gcc-cross-toolchain)
- [CMake-Toolchain-Datei](#cmake-toolchain-datei)
- [Container: Docker buildx und QEMU](#container-docker-buildx-und-qemu)
- [Go, Rust, .NET, Python](#go-rust-net-python)
- [Mikrocontroller](#mikrocontroller)
- [Checkliste](#checkliste)
<!-- /TOC -->

## Warum?

| Grund | Beispiel |
| --- | --- |
| **Zielsystem ist zu langsam** | Ein großes C++/ROS-Projekt braucht auf dem Pi Stunden, auf dem PC Minuten |
| **Zielsystem hat zu wenig Speicher** | Compiler stürzt mit „out of memory“ ab |
| **Zielsystem kann gar nicht selbst bauen** | Mikrocontroller (ESP32, STM32, Arduino) haben kein Betriebssystem |
| **Ein Build für viele Plattformen** | Release für Linux x86-64, Linux ARM64, Windows aus einer CI-Pipeline |

---

## Begriffe

| Begriff | Bedeutung |
| --- | --- |
| **Build / Host / Target** | In GNU-Terminologie: *Build* = wo gebaut wird, *Host* = wo das Ergebnis läuft, *Target* = für welche Plattform ein (Compiler-)Ergebnis selbst Code erzeugt. Im Alltag sagt man meist einfach **Host** (mein PC) und **Target** (der Pi). |
| **Architektur** | Befehlssatz: `x86_64`/`amd64`, `aarch64`/`arm64`, `armv7`/`armhf`, `riscv64` |
| **Target-Triplet** | Beschreibt das Ziel: `aarch64-linux-gnu` = ARM64, Linux, glibc; `arm-none-eabi` = ARM ohne Betriebssystem |
| **Toolchain** | Compiler, Assembler, Linker, Bibliotheken **für das Target** |
| **sysroot** | Verzeichnis mit Headern und Bibliotheken **des Zielsystems** (eine Art Abbild von dessen `/usr`) |
| **ABI** | Binärschnittstelle: wie Funktionen aufgerufen werden, wie Daten im Speicher liegen (`gnueabihf` = Hard-Float) |

**Welche Architektur hat mein Gerät?**

```bash
uname -m                 # aarch64 = 64-bit ARM (Pi 3/4/5 with 64-bit OS), armv7l = 32-bit
dpkg --print-architecture
file ./sensor            # what was a binary built for?
#   sensor: ELF 64-bit LSB pie executable, ARM aarch64, ...
```

---

## C/C++ mit GCC-Cross-Toolchain

**1. Toolchain installieren** (Debian/Ubuntu auf dem PC):

```bash
sudo apt install gcc-aarch64-linux-gnu g++-aarch64-linux-gnu    # Pi with 64-bit OS
sudo apt install gcc-arm-linux-gnueabihf                          # 32-bit Raspberry Pi OS
```

**2. Bauen:**

```bash
aarch64-linux-gnu-gcc -O2 -Wall -o sensor main.c sensor.c -lm
file sensor                    # ARM aarch64
scp -P 2222 sensor pi@192.168.10.21:/opt/lab/bin/
```

**3. Mit Abhängigkeiten (sysroot):** Sobald das Programm Bibliotheken nutzt (z. B. `libpaho-mqtt`), braucht der Cross-Compiler deren **ARM-Versionen**. Am einfachsten: das Dateisystem des Pi per `rsync` spiegeln.

```bash
mkdir -p ~/sysroots/pi
# -R keeps the full path (lib/, usr/include/, usr/lib/) below the sysroot
rsync -avzR --rsync-path="sudo rsync" -e "ssh -p 2222" \
      pi@192.168.10.21:/lib pi@192.168.10.21:/usr/include pi@192.168.10.21:/usr/lib \
      ~/sysroots/pi/
aarch64-linux-gnu-gcc --sysroot=$HOME/sysroots/pi -o sensor main.c -lpaho-mqtt3c
```

> Bei manchen Bibliotheken enthalten `.so`-Symlinks im sysroot **absolute** Pfade, die dann auf den Host zeigen. Werkzeuge wie `symlinks -rc` wandeln sie in relative um.

---

## CMake-Toolchain-Datei

In CMake beschreibt man das Ziel in einer **Toolchain-Datei** – der Rest des Projekts bleibt unverändert:

```cmake
# toolchain-aarch64.cmake
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)

set(CMAKE_C_COMPILER   aarch64-linux-gnu-gcc)
set(CMAKE_CXX_COMPILER aarch64-linux-gnu-g++)

set(CMAKE_SYSROOT $ENV{HOME}/sysroots/pi)

# search programs on the host, libraries/headers only in the sysroot
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
```

```bash
cmake -S . -B build-pi -DCMAKE_TOOLCHAIN_FILE=toolchain-aarch64.cmake
cmake --build build-pi
```

---

## Container: Docker buildx und QEMU

Mit **QEMU-Emulation** kann Docker Images für fremde Architekturen bauen – ohne eigene Toolchain. Langsamer als echtes Cross-Compiling, aber sehr bequem.

```bash
# one-time: register QEMU emulators
docker run --privileged --rm tonistiigi/binfmt --install all

# build a multi-arch image and push it to a registry
docker buildx create --use --name multiarch
docker buildx build --platform linux/amd64,linux/arm64 \
       -t registry.lab.local/sensor-service:1.0 --push .
```

Auf dem Pi zieht `docker pull` dann automatisch die ARM64-Variante. Tipp: In `Dockerfile` mit `FROM --platform=$BUILDPLATFORM` und `ARG TARGETARCH` lässt sich **im** Container nativ cross-kompilieren statt emulieren – deutlich schneller.

**Einzelne ARM-Programme auf dem PC testen:**

```bash
sudo apt install qemu-user-static
qemu-aarch64-static -L ~/sysroots/pi ./sensor
```

---

## Go, Rust, .NET, Python

| Sprache | Cross-Compiling |
| --- | --- |
| **Go** | Eingebaut: `GOOS=linux GOARCH=arm64 go build -o sensor` (ohne cgo trivial) |
| **Rust** | `rustup target add aarch64-unknown-linux-gnu`, Linker konfigurieren – oder einfach **`cross build --target …`** (nutzt Docker) |
| **.NET** | `dotnet publish -r linux-arm64 --self-contained` |
| **Java** | Bytecode ist plattformunabhängig – kein Cross-Compiling nötig (außer bei nativen Bibliotheken / GraalVM Native Image) |
| **Python** | Reiner Python-Code läuft überall. Pakete mit C-Erweiterungen brauchen **Wheels** für die Zielarchitektur (`pip download --platform manylinux2014_aarch64 --only-binary=:all: …`), oder sie werden auf dem Pi gebaut. Für Raspberry Pi gibt es vorgebaute Wheels auf **piwheels.org**. |

---

## Mikrocontroller

Bei Mikrocontrollern ist Cross-Compiling der **Normalfall**:

* **Arduino-IDE / PlatformIO / ESP-IDF** bringen die Toolchain mit (`avr-gcc`, `xtensa-esp32-elf-gcc`, `arm-none-eabi-gcc`).
* `none` im Triplet heißt: **kein Betriebssystem**, keine glibc – statt `printf` auf ein Terminal gibt es eine serielle Schnittstelle.
* Debuggen über JTAG/SWD mit OpenOCD + gdb (→ [Remote-Debugging](../Debugging/Remote-Debugging.md)).

---

## Checkliste

* [ ] Zielarchitektur und Betriebssystemversion des Targets bekannt (`uname -m`, `/etc/os-release`).
* [ ] **glibc-Version** des sysroot ≤ der auf dem Ziel (sonst: `version 'GLIBC_2.38' not found`).
* [ ] Build-Anleitung für Host **und** Target in der README.
* [ ] Toolchain-Datei / Dockerfile im Repo versioniert.
* [ ] Ergebnis mit `file` prüfen und auf dem echten Gerät testen.

---
