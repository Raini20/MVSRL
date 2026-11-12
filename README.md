# MVSR-Lab: Visual Servoing (KL – Klassische Bildverarbeitung)

Gruppe: <Vorname Nachname 1>, <Vorname Nachname 2>
Lehrveranstaltung: Moderne Verfahren der Sensorbasierten Roboterregelung, FH Technikum Wien

## Überblick

Visual Servoing für einen UR5e mit Handkamera (Eye-in-Hand) in Gazebo Harmonic / ROS 2 Jazzy.
Der Endeffektor wird ausschließlich anhand der Kamerabilder in eine vorgegebene Sollpose relativ zum Zielobjekt geregelt.

- Merkmalsextraktion (klassische Bildverarbeitung): <Verfahren, z. B. ArUco-Marker / Keypoints + Feature Matching>
- Regelgesetz: <IBVS / PBVS / hybrid>
- Kurze Begründung der Wahl: <...>

Die Simulationsumgebung (Ordner `vs_sim/`, Docker-Setup) stammt von der FH und ist unverändert übernommen. Ihre Dokumentation steht in [README_FH.md](README_FH.md). Unsere Lösung liegt in `student_ws/src/`.

## Voraussetzungen

- Docker (Docker Engine, Docker Desktop oder Colima) mit `docker compose`
- ca. 10 min und einige GB Speicher für den einmaligen Image-Build
- Linux: X11 (für die GUI). Windows/macOS: ein Browser (GUI über noVNC)
- Alle weiteren Abhängigkeiten (Ubuntu 24.04, ROS 2 Jazzy, Gazebo Harmonic, OpenCV) sind im Docker-Image enthalten, eine lokale ROS-Installation ist nicht nötig.

## Installation

ZIP entpacken, im Terminal in den entpackten Ordner wechseln und das Image bauen (einmalig):

    # Linux
    docker compose build

    # Windows / macOS
    docker compose -f docker-compose.novnc.yml build

<Eventuelle zusätzliche Pakete für unsere Lösung (z. B. pip-Pakete) und wie sie installiert werden: ...>

## Programm starten

**1. Container starten**

    # Linux (X11)
    xhost +local:docker
    docker compose run --rm sim              # NVIDIA-GPU: docker compose run --rm sim-nvidia

    # Windows / macOS (GUI im Browser)
    docker compose -f docker-compose.novnc.yml up -d novnc
    docker compose -f docker-compose.novnc.yml run --rm sim
    # danach im Browser http://localhost:8080/vnc.html öffnen

**2. Simulation starten** (im Container)

    ros2 launch vs_sim vs_sim.launch.py

**3. Zweites Terminal im selben Container** (für Startpose und unseren Regler)

    docker exec -it $(docker ps -qf ancestor=vs-sim:jazzy | head -1) bash

**4. Startkonfiguration anfahren** (im zweiten Terminal)

    ros2 run vs_sim go_to_start --target aruco --start 3

**5. Unsere Lösung bauen und starten** (im zweiten Terminal)

    cd /ws/student && colcon build --symlink-install && source install/setup.bash
    ros2 run <paket> <knoten>

macOS (Apple Silicon): Funktionstest der Simulation ohne GUI mit `./run_headless_test.sh`.

## Projektstruktur

    student_ws/src/         unsere ROS-2-Pakete (Lösung)
    vs_sim/                 Simulationspaket der FH (unverändert)
    Dockerfile, docker-compose*.yml, docker/, run_headless_test.sh
                            Docker-Setup der FH (unverändert)
    data/                   Referenzbilder und Logs (werden zur Laufzeit erzeugt, nicht im Git)
    docs/                   Bilder aus der FH-Dokumentation
    results/                Evaluierung (Plots, Tabellen)
    README_FH.md            Original-README der FH

## Methode

### Merkmalsextraktion
<Verfahren, Parameter und Begründung>

### Regelgesetz
<IBVS / PBVS / hybrid; Interaktionsmatrix, Tiefenschätzung, Verstärkung; Begründung>

### Gewählte Parameter
<Parameter und Begründung>

## Evaluierung und Ergebnisse

Untersucht wird der Einfluss der Reglerparameter (z. B. der Verstärkung) auf Konvergenz und Bewegungsverhalten. Als Fehlermaß dient `/ground_truth/pose_error`; die Ground-Truth-Topics werden **nicht** in der Regelung verwendet, sondern nur zur Auswertung.

<Beschreibung der Versuche, Verweis auf `results/`, Ergebnisse>

## Quellen und Hilfsmittel

<Alle verwendeten Quellen, Bibliotheken, Beispiele und Hilfsmittel (auch KI-Tools) angeben.>

## Autoren

<Namen>

---

## Arbeitsstand (vor der Abgabe entfernen)

Mindestanforderungen laut Angabe, Variante KL:

- [ ] Extraktion der Bildmerkmale mit klassischen Bildverarbeitungsverfahren
- [ ] Klassisches Visual-Servoing-Regelgesetz (IBVS, PBVS oder hybrid)
- [ ] Begründung der gewählten Merkmale und des Regelansatzes
- [ ] Untersuchung des Einflusses der Reglerparameter (z. B. Verstärkung) auf Konvergenz und Bewegungsverhalten
- [ ] Keine Deep-Learning- oder Foundation-Model-Verfahren für die Merkmalsextraktion

Abgabe: ZIP über Moodle, benannt `MVSR-Lab_Visual-Servoing_KL_<Nachname1>_<Nachname2>.zip`, mit Code, README, Folien (PowerPoint) und Demovideo. Die README muss die Inbetriebnahme allein erklären (Test mit einem frischen Klon). Kein auskommentierter oder ungenutzter Code.
