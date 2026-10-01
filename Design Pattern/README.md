# Design Pattern (Entwurfsmuster)

<!-- TOC -->
## Inhaltsverzeichnis

- [Was ein Design Pattern ist](#was-ein-design-pattern-ist)
- [Was ein Design Pattern nicht ist](#was-ein-design-pattern-nicht-ist)
- [Aufbau einer Musterbeschreibung](#aufbau-einer-musterbeschreibung)
- [Kategorien](#kategorien)
- [Alle 23 GoF-Patterns](#alle-23-gof-patterns)
  - [Erzeugungsmuster](#erzeugungsmuster)
  - [Strukturmuster](#strukturmuster)
  - [Verhaltensmuster](#verhaltensmuster)
- [UML-Klassendiagramme lesen](#uml-klassendiagramme-lesen)
- [Über die GoF hinaus](#über-die-gof-hinaus)
<!-- /TOC -->

## Was ein Design Pattern ist

Ein **Design Pattern** (Entwurfsmuster) beschreibt einen **bewährten, allgemeinen Lösungsansatz für ein wiederkehrendes Entwurfsproblem**. Es gibt einem Problem und seiner Lösung einen **Namen**, beschreibt, **in welchem Kontext** es auftritt, **wie** die Lösung grundsätzlich aufgebaut ist und **welche Vor- und Nachteile** sie mit sich bringt.

Design Patterns stammen aus der Softwareentwicklung, die Idee dahinter wurde jedoch aus der **Architektur** übernommen: Der Architekt **Christopher Alexander** beschrieb 1977 in *A Pattern Language* über 250 Muster für Städte, Gebäude und Räume – etwa „Fenster zum Sitzen“ oder „Lichteinfall von zwei Seiten in jedem Raum“. Jedes Muster nennt ein wiederkehrendes Problem und den Kern einer Lösung, „so dass man diese Lösung millionenfach anwenden kann, ohne sie je zweimal gleich auszuführen“.

Diese Idee übertrugen Erich **G**amma, Richard **H**elm, Ralph **J**ohnson und John **V**lissides 1994 in ihrem Buch *Design Patterns: Elements of Reusable Object-Oriented Software* auf die Software. Die vier Autoren werden seitdem die **„Gang of Four“ (GoF)** genannt, die 23 dort beschriebenen Muster die **GoF-Patterns**.

Ein Pattern ist damit:

* eine **Vorlage**, die flexibel in verschiedenen Kontexten eingesetzt werden kann,
* ein **gemeinsames Vokabular**: „Das lösen wir mit einem Observer“ ersetzt eine halbe Seite Erklärung,
* **gesammelte Erfahrung**: Andere sind bereits in die Fallen getreten, die das Muster vermeidet.

---

## Was ein Design Pattern nicht ist

Ein Design Pattern ist **kein fertiges Programm** und **keine Bibliothek**, die man unverändert übernimmt. Es liefert eine **strukturierte Herangehensweise** an eine bestimmte **Problemklasse** – die konkrete Umsetzung sieht in jedem Projekt anders aus.

* Der **Beispielcode** in diesem Ordner dient der **Veranschaulichung**. Er muss in aller Regel an die eigenen Gegebenheiten angepasst werden – an Klassen- und Methodennamen, an Item-Namen (z. B. in openHAB), an Schwellenwerte, an Geräte und an die eigene Installation.
* Weil ein Pattern keine vollständige Lösung ist, enthalten die Beispiele oft **zusätzlichen Code**, der über das eigentliche Muster hinausgeht (Initialisierung, Ausgaben, Beispieldaten). Entscheidend ist die **Struktur**, nicht jede Zeile.
* Ein Pattern ist **kein Selbstzweck**. Wer Muster einbaut, nur um Muster zu verwenden, erzeugt unnötige Komplexität. Ein Muster wird verwendet, wenn das **Problem** auftaucht, das es löst.
* Ein Pattern ist **nicht an eine Sprache gebunden**. In Python oder JavaScript schrumpfen manche Muster auf wenige Zeilen (Funktionen sind Objekte, Module sind Singletons), in Java oder C++ sind sie ausführlicher.

---

## Aufbau einer Musterbeschreibung

Jedes Pattern in diesem Ordner folgt derselben Gliederung (angelehnt an die GoF):

| Abschnitt | Inhalt |
| --- | --- |
| **Zweck** | Was das Muster in einem Satz leistet |
| **Problem** | Welche Situation das Muster nötig macht |
| **Lösung** | Die Idee hinter dem Muster |
| **Struktur** | UML-Klassendiagramm (Mermaid) |
| **Beispiel** | Lauffähiges Python-Beispiel, meist aus Smart Home oder Robotik |
| **Praxis** | Wo man dem Muster in echten Frameworks begegnet |
| **Vor- und Nachteile** | Konsequenzen des Einsatzes |
| **Verwandte Muster** | Abgrenzung und Kombination |

---

## Kategorien

Die GoF teilen die Muster nach ihrem **Zweck** in drei Gruppen ein:

| Kategorie | Frage | Muster |
| --- | --- | --- |
| **Erzeugungsmuster** (*Creational*) | Wie werden Objekte erzeugt? | 5 |
| **Strukturmuster** (*Structural*) | Wie werden Klassen und Objekte zu größeren Strukturen zusammengesetzt? | 7 |
| **Verhaltensmuster** (*Behavioral*) | Wie arbeiten Objekte zusammen und verteilen Verantwortung? | 11 |

Zusätzlich unterscheiden sie nach dem **Gültigkeitsbereich**: **Klassenmuster** arbeiten mit Vererbung und sind zur Übersetzungszeit festgelegt (Factory Method, Adapter als Klassenadapter, Interpreter, Template Method), **Objektmuster** arbeiten mit Komposition und sind zur Laufzeit veränderbar (alle anderen).

---

## Alle 23 GoF-Patterns

### Erzeugungsmuster

| Muster | Deutsch | Kurzbeschreibung |
| --- | --- | --- |
| [Abstract Factory](Erzeugungsmuster/Abstract%20Factory.md) | Abstrakte Fabrik | Erzeugt Familien zusammengehöriger Objekte |
| [Builder](Erzeugungsmuster/Builder.md) | Erbauer | Baut komplexe Objekte Schritt für Schritt |
| [Factory Method](Erzeugungsmuster/Factory%20Method.md) | Fabrikmethode | Unterklassen entscheiden, welches Objekt erzeugt wird |
| [Prototype](Erzeugungsmuster/Prototype.md) | Prototyp | Neue Objekte durch Klonen einer Vorlage |
| [Singleton](Erzeugungsmuster/Singleton.md) | Einzelstück | Genau eine Instanz mit globalem Zugriffspunkt |

### Strukturmuster

| Muster | Deutsch | Kurzbeschreibung |
| --- | --- | --- |
| [Adapter](Strukturmuster/Adapter.md) | Adapter | Passt eine Schnittstelle an eine erwartete an |
| [Bridge](Strukturmuster/Bridge.md) | Brücke | Trennt Abstraktion und Implementierung |
| [Composite](Strukturmuster/Composite.md) | Kompositum | Baumstrukturen, Teil und Ganzes einheitlich behandeln |
| [Decorator](Strukturmuster/Decorator.md) | Dekorierer | Fügt Objekten dynamisch Funktionen hinzu |
| [Facade](Strukturmuster/Facade.md) | Fassade | Einfache Schnittstelle zu einem komplexen Subsystem |
| [Flyweight](Strukturmuster/Flyweight.md) | Fliegengewicht | Teilt gemeinsame Daten zwischen vielen Objekten |
| [Proxy](Strukturmuster/Proxy.md) | Stellvertreter | Kontrolliert den Zugriff auf ein Objekt |

### Verhaltensmuster

| Muster | Deutsch | Kurzbeschreibung |
| --- | --- | --- |
| [Chain of Responsibility](Verhaltensmuster/Chain%20of%20Responsibility.md) | Zuständigkeitskette | Anfrage wandert durch eine Kette von Bearbeitern |
| [Command](Verhaltensmuster/Command.md) | Befehl | Kapselt eine Anfrage als Objekt |
| [Interpreter](Verhaltensmuster/Interpreter.md) | Interpreter | Grammatik und Auswertung einer kleinen Sprache |
| [Iterator](Verhaltensmuster/Iterator.md) | Iterator | Sequenzieller Zugriff ohne Kenntnis der inneren Struktur |
| [Mediator](Verhaltensmuster/Mediator.md) | Vermittler | Zentrale Koordination statt direkter Kopplung |
| [Memento](Verhaltensmuster/Memento.md) | Memento | Zustand speichern und wiederherstellen |
| [Observer](Verhaltensmuster/Observer.md) | Beobachter | Abhängige werden über Änderungen benachrichtigt |
| [State](Verhaltensmuster/State.md) | Zustand | Verhalten ändert sich mit dem inneren Zustand |
| [Strategy](Verhaltensmuster/Strategy.md) | Strategie | Austauschbare Algorithmen |
| [Template Method](Verhaltensmuster/Template%20Method.md) | Schablonenmethode | Ablauf fest, einzelne Schritte überschreibbar |
| [Visitor](Verhaltensmuster/Visitor.md) | Besucher | Neue Operationen auf einer Objektstruktur, ohne sie zu ändern |

---

## UML-Klassendiagramme lesen

Die Diagramme sind mit [Mermaid](https://mermaid.js.org/) geschrieben und werden von GitHub und GitLab direkt gerendert.

| Notation | Bedeutung |
| --- | --- |
| `A <\|-- B` | **Vererbung**: B erbt von A (B *ist ein* A) |
| `A <\|.. B` | **Realisierung**: B implementiert das Interface A |
| `A *-- B` | **Komposition**: A besteht aus B, B lebt nicht ohne A |
| `A o-- B` | **Aggregation**: A enthält B, B kann auch allein existieren |
| `A --> B` | **Assoziation**: A kennt/verwendet B dauerhaft |
| `A ..> B` | **Abhängigkeit**: A verwendet B vorübergehend (z. B. erzeugt es) |
| `<<interface>>` / `<<abstract>>` | Schnittstelle bzw. abstrakte Klasse |
| `+` / `-` / `#` | public / private / protected |

---

## Über die GoF hinaus

Die 23 GoF-Patterns sind nur der bekannteste Katalog. Weitere Musterfamilien, denen man im Alltag begegnet:

| Familie | Beispiele |
| --- | --- |
| **Architekturmuster** | MVC, MVVM, Schichtenarchitektur, Microservices, Event-Driven Architecture, Hexagonal Architecture |
| **Integrations- und Messaging-Muster** | Publish/Subscribe (MQTT), Message Queue, Request/Reply |
| **Enterprise-Muster** | Repository, Unit of Work, Data Transfer Object, Dependency Injection |
| **Nebenläufigkeit** | Producer/Consumer, Thread Pool, Future/Promise, Reactor |
| **Cloud / Verteilte Systeme** | Circuit Breaker, Retry, Sidecar, Saga |
| **Anti-Patterns** | Bewährte *Fehler*: God Object, Spaghetti Code, Golden Hammer, Copy-Paste-Programming |

Verwandte Begriffe wie Template, Skeleton, Scaffolding und Orchestrierung: [Software-Konzepte](../Software-Konzepte/README.md).
