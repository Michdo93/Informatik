# Dateistruktur & Projektorganisation.md

Auch für **Dateistruktur und Projektorganisation** gibt es etablierte **Standards, Konventionen und Best Practices**, die sich oft an Sprache, Framework oder Build-Tool orientieren. Eine gute Struktur hilft bei:

* **Wartbarkeit**
* **Lesbarkeit**
* **Teamzusammenarbeit**
* **Deployment / CI/CD**

---

<!-- TOC -->
## Inhaltsverzeichnis

- [🗂️ Allgemeine Prinzipien guter Dateiorganisation](#️-allgemeine-prinzipien-guter-dateiorganisation)
- [📁 Beispiele für typische Projektstrukturen](#-beispiele-für-typische-projektstrukturen)
  - [🔹 Python (PEP 8 + Paketstruktur)](#-python-pep-8--paketstruktur)
  - [🔹 Java (Maven- oder Gradle-Struktur)](#-java-maven--oder-gradle-struktur)
  - [🔹 Node.js / JavaScript / TypeScript (Common)](#-nodejs--javascript--typescript-common)
  - [🔹 PHP (PSR-konforme Struktur, z. B. Laravel)](#-php-psr-konforme-struktur-zb-laravel)
  - [🔹 Go (Standardstruktur nach golang-standards/project-layout)](#-go-standardstruktur-nach-golang-standardsproject-layout)
- [🧱 Weitere nützliche Ordnerkonventionen (sprachübergreifend)](#-weitere-nützliche-ordnerkonventionen-sprachübergreifend)
- [📌 Standardisierungsquellen](#-standardisierungsquellen)
<!-- /TOC -->

## 🗂️ Allgemeine Prinzipien guter Dateiorganisation

| Prinzip                                     | Bedeutung                                                    |
| ------------------------------------------- | ------------------------------------------------------------ |
| **Trennung nach Funktion**                  | z. B. `models/`, `controllers/`, `views/`                    |
| **Trennung nach Schicht**                   | z. B. `core/`, `infrastructure/`, `presentation/`, `domain/` |
| **Trennung nach Feature (Modularisierung)** | z. B. `user/`, `product/`, `auth/`                           |
| **Konsistente Benennung**                   | Ordner- und Dateinamen folgen Namenskonventionen             |
| **Index- oder Hauptdateien nutzen**         | z. B. `index.js`, `main.py`, `App.kt`, `init.lua`            |
| **Kapselung / Minimale Sichtbarkeit**       | Dateien enthalten nur, was nötig ist                         |

---

## 📁 Beispiele für typische Projektstrukturen

---

### 🔹 **Python (PEP 8 + Paketstruktur)**

```plaintext
my_project/
│
├── my_project/           # Hauptpaket
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   └── utils.py
│
├── tests/
│   └── test_models.py
│
├── requirements.txt
├── setup.py
└── README.md
```

---

### 🔹 **Java (Maven- oder Gradle-Struktur)**

```plaintext
my-app/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/app/
│   │   │       ├── App.java
│   │   │       └── service/UserService.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       └── java/
│           └── com/example/app/
│               └── AppTest.java
├── pom.xml (oder build.gradle)
```

✅ Standard bei Maven/Gradle-Projekten

---

### 🔹 **Node.js / JavaScript / TypeScript (Common)**

```plaintext
my-app/
├── src/
│   ├── components/
│   │   └── Button.tsx
│   ├── pages/
│   │   └── Home.tsx
│   ├── utils/
│   │   └── formatDate.ts
│   └── index.tsx
├── public/
├── tests/
├── package.json
└── tsconfig.json
```

✅ Häufig in React/Next.js/TypeScript-Projekten

---

### 🔹 **PHP (PSR-konforme Struktur, z. B. Laravel)**

```plaintext
my-project/
├── app/
│   ├── Models/
│   ├── Http/
│   │   ├── Controllers/
│   │   └── Middleware/
│   ├── Providers/
├── resources/
│   └── views/
├── routes/
│   └── web.php
├── tests/
├── public/
│   └── index.php
├── composer.json
└── .env
```

✅ Laravel (ein PHP-Framework) folgt PSR + Framework-Struktur

---

### 🔹 **Go (Standardstruktur nach `golang-standards/project-layout`)**

```plaintext
myapp/
├── cmd/
│   └── myapp/
│       └── main.go
├── internal/
│   └── user/
├── pkg/
│   └── logger/
├── api/
├── configs/
├── scripts/
├── go.mod
└── README.md
```

✅ Offizielle Empfehlung: [https://github.com/golang-standards/project-layout](https://github.com/golang-standards/project-layout)

---

## 🧱 Weitere nützliche Ordnerkonventionen (sprachübergreifend)

| Ordnername         | Zweck                                      |
| ------------------ | ------------------------------------------ |
| `src/`             | Quellcode                                  |
| `lib/`             | Wiederverwendbare Module / Libraries       |
| `test/` / `tests/` | Unit- und Integrationstests                |
| `bin/`             | Ausführbare Dateien / CLI                  |
| `public/`          | Öffentlich zugängliche Dateien (z. B. Web) |
| `config/`          | Konfigurationsdateien                      |
| `docs/`            | Dokumentation                              |
| `scripts/`         | Build-/Install-/CI-Skripte                 |
| `dist/` / `build/` | Kompilierte / gebündelte Artefakte         |
| `.env`             | Umgebungsvariablen                         |

---

## 📌 Standardisierungsquellen

| Sprache/Ökosystem | Quelle / Standard                                                    |
| ----------------- | -------------------------------------------------------------------- |
| **Python**        | [PEP 8](https://peps.python.org/pep-0008/), `setup.py`               |
| **Java**          | Maven, Gradle                                                        |
| **JS/TS**         | Node.js, React/Next, Angular CLI                                     |
| **PHP**           | [PSR-4 Autoloading](https://www.php-fig.org/psr/psr-4/)              |
| **Go**            | [project-layout](https://github.com/golang-standards/project-layout) |
| **Rust**          | `src/main.rs`, `Cargo.toml`                                          |
| **C# (.NET)**     | Standard Visual Studio Projektstruktur                               |

---
