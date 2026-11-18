
# MVSR-Lab: Visual Servoing (KL – Klassische Bildverarbeitung)

Gruppe: Rainhard Wipp, Merle <Nachname></nachname>
Lehrveranstaltung: Moderne Verfahren der Sensorbasierten Roboterregelung, FH Technikum Wien

## Überblick

Visual Servoing für einen UR5e mit Handkamera (Eye-in-Hand) in Gazebo Harmonic / ROS 2 Jazzy. Der Endeffektor wird ausschließlich anhand der Kamerabilder in eine vorgegebene Sollpose relativ zum Zielobjekt geregelt.

Wir setzen zwei klassische Regelgesetze um und vergleichen sie:

- **IBVS** (bildbasiert, Rainhard Wipp): Der Regelfehler wird direkt im Bild definiert, die Interaktionsmatrix verwendet die Tiefe der Merkmale.
- **PBVS** (positionsbasiert, Merle <Nachname></nachname>): Aus den Merkmalen wird die Kamerapose relativ zum Objekt geschätzt, der Regelfehler ist der Posenfehler.
  Beide nutzen dieselbe Merkmalserkennung und Pose-Schätzung (SIFT-Merkmale, Matching mit Ratio-Test, Pose per PnP) auf dem Ziel `feature`. Basis dafür sind unsere eigenen Hausübungen 2 und 3. Der Vergleich erfolgt auf denselben Startkonfigurationen mit denselben Metriken.

Die Simulationsumgebung (Ordner `vs_sim/`, Docker-Setup) stammt von der FH und ist unverändert übernommen. Ihre Dokumentation steht in [README_FH.md](README_FH.md). Unsere Lösung liegt in `student_ws/src/`.

## Voraussetzungen

- Docker (Docker Engine, Docker Desktop oder Colima) mit `docker compose`
- ca. 10 min und einige GB Speicher für den einmaligen Image-Build
- Linux: X11 (für die GUI). Windows/macOS: ein Browser (GUI über noVNC); unter Windows alternativ WSL2 mit Ubuntu 24.04
- Alle weiteren Abhängigkeiten (Ubuntu 24.04, ROS 2 Jazzy, Gazebo Harmonic, OpenCV) sind im Docker-Image enthalten, eine lokale ROS-Installation ist nicht nötig.

## Installation

ZIP entpacken, im Terminal in den entpackten Ordner wechseln und das Image bauen (einmalig):

    # Linux
    docker compose build

    # Windows / macOS
    docker compose -f docker-compose.novnc.yml build

<Zusätzliche Python-Pakete für unsere Lösung (siehe requirements.txt der Pakete) und wie sie installiert werden: ...>

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

**3. Zweites Terminal im selben Container** (für Referenz, Startpose und Regler)

    docker exec -it $(docker ps -qf ancestor=vs-sim:jazzy | head -1) bash

**4. Referenzbild aufnehmen und Startkonfiguration anfahren** (im zweiten Terminal)

    ros2 run vs_sim capture_reference --target feature
    ros2 run vs_sim go_to_start --target feature --start 3

**5. Unsere Lösung bauen und starten** (im zweiten Terminal)

    cd /ws/student && colcon build --symlink-install && source install/setup.bash

    # IBVS
    ros2 run vs_ibvs <knoten></knoten>

    # PBVS
    ros2 run vs_pbvs <knoten></knoten>

    # Evaluierung beider Regler auf allen Startkonfigurationen
    ros2 run vs_common <evaluierung></evaluierung>

**6. Kamerabild live ansehen**

`vs_sim.launch.py` öffnet `rqt_image_view` mit `/camera/image_raw`. Das Debug-Bild der Regler (erkannte Merkmale, Sollpunkte, Fehler) lässt sich im selben Fenster über die Topic-Auswahl umschalten (`<Topic>`). Ein weiteres Fenster öffnet man im zweiten Terminal mit `ros2 run rqt_image_view rqt_image_view`. Unter Linux und WSL erscheint es als normales Fenster, unter Windows ohne WSLg und unter macOS im Browser (noVNC, siehe Schritt 1).

macOS (Apple Silicon): Funktionstest der Simulation ohne GUI mit `./run_headless_test.sh`.

## Projektstruktur

    student_ws/src/
      vs_common/            gemeinsam: Merkmale, Matching, Pose (PnP), Evaluierung
      vs_ibvs/              IBVS-Regler (Rainhard Wipp)
      vs_pbvs/              PBVS-Regler (Merle <Nachname></nachname>)
    vs_sim/                 Simulationspaket der FH (unverändert)
    Dockerfile, docker-compose*.yml, docker/, run_headless_test.sh
                            Docker-Setup der FH (unverändert)
    data/                   Referenzbilder und Logs (werden zur Laufzeit erzeugt, nicht im Git)
    docs/                   Bilder aus der FH-Dokumentation
    results/                Evaluierung (Rohdaten, Plots, Tabellen)
    README_FH.md            Original-README der FH

## Methode

### Merkmalsextraktion und Pose (gemeinsam)

SIFT-Merkmale im Referenzbild (Kamerabild der Sollpose), Matching mit Ratio-Test, Pose per `solvePnPRansac` und Verfeinerung. Die 3D-Punkte der Referenzmerkmale folgen aus der bekannten Sollpose und der Zielebene. Ohne verlässliche Pose wird kein Befehl berechnet, die Kamera bleibt stehen. <Parameter und Begründung>

### IBVS

Regelfehler `e = s − s*` in normierten Bildkoordinaten, Interaktionsmatrix `L` mit der Tiefe der Punkte aus der Pose, Regelgesetz `v = −λ · L⁺ · e`. Die Kamerageschwindigkeit wird über die Jacobi-Matrix des UR5e in Gelenkgeschwindigkeiten umgerechnet. <Begründung der Wahl, Verstärkung>

### PBVS

<Regelfehler aus der Kamerapose, Regelgesetz, Umrechnung in Gelenkgeschwindigkeiten, Begründung der Wahl, Verstärkung>

### Gewählte Parameter

<Parameter und Begründung>

## Evaluierung und Ergebnisse

Beide Regler laufen auf denselben Startkonfigurationen S1–S8 mit denselben Abbruchkriterien. Verglichen werden Erfolg, Konvergenzzeit, Endfehler und der Verlauf des Fehlers. Als Fehlermaß dient `/ground_truth/pose_error`; die Ground-Truth-Topics werden **nicht** in der Regelung verwendet, sondern nur zur Auswertung. Pro Regler wird zusätzlich der Einfluss der Verstärkung auf Konvergenz und Bewegungsverhalten untersucht.

<Beschreibung der Versuche, Verweis auf `results/`, Ergebnisse, Vergleich IBVS gegen PBVS, besondere Fälle wie S7 (170° Drehung um die optische Achse)>

## Quellen und Hilfsmittel

- Eigene Hausübungen HÜ 2 (Pose Estimation) und HÜ 3 (IBVS) aus diesem Lehrveranstaltungsblock, öffentliches Repo "MVSR". Der Code wurde für die Simulation angepasst (Einheiten, 3D-Punkte aus der Sollpose, Regelschleife in ROS 2).
- Simulationsumgebung der FH (`vs_sim`, Docker-Setup).
- F. Chaumette, S. Hutchinson: Visual Servo Control, Part I und II (IEEE Robotics & Automation Magazine, 2006/2007).
- OpenCV, NumPy, ROS 2 Jazzy.
- <Weitere Quellen und Hilfsmittel, auch KI-Tools, angeben>

## Autoren

Rainhard Wipp, Merle <Nachname></nachname>

---

## Arbeitsstand (vor der Abgabe entfernen)

Mindestanforderungen laut Angabe, Variante KL:

- [ ] Extraktion der Bildmerkmale mit klassischen Bildverarbeitungsverfahren (`vs_common`)
- [ ] Klassisches Visual-Servoing-Regelgesetz: IBVS (`vs_ibvs`, Rainhard) und PBVS (`vs_pbvs`, Merle)
- [ ] Begründung der gewählten Merkmale und des Regelansatzes
- [ ] Untersuchung des Einflusses der Reglerparameter (z. B. Verstärkung) auf Konvergenz und Bewegungsverhalten, je Regler
- [ ] Keine Deep-Learning- oder Foundation-Model-Verfahren für die Merkmalsextraktion
- [ ] Vergleich IBVS/PBVS auf S1–S8 (`vs_common`, Evaluierung)
  Abgabe: ZIP über Moodle, benannt `MVSR-Lab_Visual-Servoing_KL_<Nachname1>_<Nachname2>.zip`, mit Code, README, Folien (PowerPoint) und Demovideo. Die README muss die Inbetriebnahme allein erklären (Test mit einem frischen Klon). Kein auskommentierter oder ungenutzter Code.
