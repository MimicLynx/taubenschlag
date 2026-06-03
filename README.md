# Taubenschlag

Taubenschlag ist ein portables Werkzeug aus einer einzelnen HTML-Datei. Es erstellt LARP-Ingame-Korrespondenz und speichert sie über den Druckdialog des Browsers als DIN-A4-PDF.

Die aktuelle Version enthält Vorlagen für Briefe, Zeitungen und Steckbriefe. Gemeinsame Felder werden je nach Vorlage passend beschriftet, vorlagenspezifische Steuerungen bleiben verborgen, bis sie relevant sind, Layoutvarianten wechseln mit der ausgewählten Vorlage, jede Vorlage hat passende Beispielinhalte, und die Standard-Schriftkombinationen passen zum Dokumenttyp.

## Nutzung

Diese Datei im Browser öffnen:

```text
index.html
```

Dann:

1. Oberflächensprache wählen. Deutsch ist voreingestellt; Englisch und Spanisch sind ebenfalls verfügbar.
2. Vorlage wählen:
   - Brief
   - Zeitung
   - Steckbrief
3. Formular ausfüllen. Die Feldbeschriftungen passen sich an die gewählte Vorlage an.
   - Wenn der aktuelle Inhalt noch dem eingebauten Beispiel entspricht, ersetzt ein Vorlagenwechsel ihn durch den passenden Beispieltext der neuen Vorlage.
   - Wenn Felder bereits bearbeitet wurden, bleibt der eigene Text beim Vorlagenwechsel erhalten.
4. Texturmuster, Texturdeckkraft, optionale Farbüberlagerung, Titelschrift, Textschrift, Ornamentstil, Layoutvariante, Zeilenabstand und Seitenränder wählen.
   - Briefe verwenden standardmäßig handschriftliche Schriften.
   - Zeitungen und Steckbriefe verwenden standardmäßig Druck- bzw. Display-Schriften, keine Handschriften.
   - Wenn eine Schrift manuell geändert wurde, bleibt diese Auswahl beim Vorlagenwechsel erhalten.
5. Optional Dateien auswählen:
   - randloses Seitenhintergrundbild
   - transparente PNG-Unterschrift
   - transparentes PNG-Siegel oder Stempel
   - Porträt oder Wappen für Steckbriefe
   - Artikelbild für Zeitungen, platzierbar am Anfang, in der Mitte oder am Ende des Artikelflusses
6. Optional einen Entwurf als JSON exportieren oder importieren.
7. `Drucken / PDF speichern` anklicken.
8. Im Druckdialog des Browsers:
   - A4 wählen
   - als PDF speichern
   - Hintergrundgrafiken aktivieren, falls der Browser das verlangt
   - nach Möglichkeit keine Seitenränder verwenden

## Portabilität

Das Werkzeug für die Nutzung ist `index.html`.

Es enthält:

- eingebettetes CSS
- eingebettetes JavaScript
- keine externen Skripte
- keine externen Stilvorlagen
- keine Netzwerkanforderung
- keinen Serverbedarf

Ausgewählte Bilder werden nur lokal in der aktuellen Browsersitzung gelesen. Sie werden nirgendwohin hochgeladen. Beim Neuladen der Seite verschwinden sie. Druckbare Bilder im Dokument werden schwarzweiß dargestellt; ausgewählte Hintergründe bleiben Seitenhintergründe.

## Sicherheitsmodell

Taubenschlag ist eine statische Browser-Anwendung. Es gibt keinen Hochlade-Endpunkt, keine serverseitige Speicherung und kein ausführbares Webverzeichnis. Eine im Browser ausgewählte `.php`-, `.jsp`- oder ähnliche Datei kann in dieser Architektur keine Webshell werden, weil serverseitig nichts existiert, das sie ausführen könnte. Das ändert sich sofort, wenn später ein Serverteil ergänzt wird. Dann müssen ausgewählte Dateien serverseitig erneut validiert und außerhalb ausführbarer Pfade gespeichert werden. Bedrohungsmodelle ändern sich gern genau dann, wenn jemand „nur kurz“ eine Funktion anschraubt.

Aktuelle clientseitige Schutzmaßnahmen:

- Textfeldinhalte werden mit Text-APIs (`textContent` / Text-Nodes) gerendert, nicht über HTML-Injection-Sinks.
- Textfelder und Textbereiche haben `maxlength`-Grenzen, um versehentlichen browserseitigen Ressourcenmissbrauch zu reduzieren.
- Der Entwurfsimport akzeptiert nur ein kleines JSON-Schema: feste Anwendungs-/Versions-Hülle, Whitelist für Schlüssel, Enum-Prüfungen, Zahlenbereichsprüfungen und Textlängenlimits. Importierte Bilder werden ignoriert.
- Das Dokument enthält eine restriktive CSP für statische Anwendungen und eine `no-referrer`-Policy: keine Netzwerkverbindungen, Formulare, Objekte, Frames, Worker, Medien oder Base-URI.
- Bildauswahl ist auf PNG, JPEG und WebP begrenzt.
- SVG, HTML, XML, Skripte, unbekannte Typen und getarnte Dateien werden abgelehnt.
- Ausgewählte Dateien sind auf 8 MB begrenzt, bevor der Browser sie als Data-URL liest.
- Dateisignaturen werden geprüft, bevor eine Datei als Bild verwendet wird.

Wenn die Datei öffentlich bereitgestellt wird, sollte dieselbe CSP zusätzlich als HTTP-Antwort-Header gesetzt werden, nicht nur als Meta-Tag. Falls später ein Serverteil mit Hochlade- oder Speicherfunktion dazukommt, reicht clientseitige Prüfung nicht: serverseitig erneut validieren, sichere Dateinamen erzeugen, Dateien außerhalb ausführbarer Pfade speichern, mit sicheren Inhalts-Headern ausliefern und ausgewählte Bytes niemals in denselben Pfad wie ausführbaren Code legen. Die Blasttür funktioniert nur, solange niemand daneben eine kleinere Tür einbaut.

Empfohlene Header für statisches Hosting:

```text
Content-Security-Policy: default-src 'none'; script-src 'unsafe-inline'; style-src 'unsafe-inline'; img-src data: blob:; font-src data:; connect-src 'none'; form-action 'none'; object-src 'none'; base-uri 'none'; frame-src 'none'; media-src 'none'; worker-src 'none'; manifest-src 'none'
Referrer-Policy: no-referrer
X-Content-Type-Options: nosniff
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=(), interest-cohort=()
```

Die CSP erlaubt weiterhin eingebettete Skripte und eingebettete Stile, weil das Ergebnis absichtlich eine einzelne, vollständig eigenständige HTML-Datei ist. Falls das Projekt später nicht mehr aus einer einzelnen Datei bestehen soll, sollten Skript und CSS in eigene Dateien ausgelagert und `'unsafe-inline'` durch Hashes oder Nonces ersetzt werden. Eleganz ist optional. Die Sicherheitsgrenze nicht aufzuweichen ist weniger optional.

## Umgesetzte Funktionen

- DIN-A4-Hochformatvorschau
- Drucken-/PDF-Speichern-Schaltfläche mit `window.print()`
- Sprachwahl für die Oberfläche:
  - Deutsch
  - Englisch
  - Spanisch
- Vorlagenauswahl:
  - Brief
  - Zeitung
  - Steckbrief
- Gemeinsame Inhaltsfelder und optionale Datei-/Steuergruppen werden je nach Vorlage umbeschriftet oder ausgeblendet, damit im ausgewählten Modus keine irrelevanten Steuerungen erscheinen.
- Vorlagenspezifische Beispielinhalte für Briefe, Zeitungen und Steckbriefe. Beispieltext wird beim Vorlagenwechsel aktualisiert, solange die eingebauten Beispiele unverändert sind; bearbeitete Nutzerinhalte bleiben erhalten.
- Eingebettete schwarzweiße Texturmuster mit Deckkraftregler:
  - Schwarzes Papier (`Black Paper`)
  - Beiges Papier (`Beige Paper`)
  - Pappe (`Cardboard`)
  - Cremefarbenes Papier (`Cream Paper`)
  - Exklusives Papier (`Exclusive Paper`)
  - Helle Papierfasern (`Light Paper Fibers`)
  - Naturpapier (`Natural Paper`)
  - Papier 2 (`Paper 2`)
- Farbüberlagerung ist optional und standardmäßig deaktiviert; ohne Aktivierung bleibt die Seite weiß mit schwarz-grauer Textur.
- Auswahl für randlose Hintergrundbilder
- Entwurfsexport/-import für Text und Einstellungen als begrenzte JSON-Datei; ausgewählte Bilder bleiben sitzungsgebunden und werden nicht eingeschlossen
- Auswahl für Unterschriftenbild
- Auswahl für Siegel-/Stempelbild
- Steuerung für Position, Größe und Deckkraft von Siegel/Stempel
- Nur handschriftliche oder mittelalterlich wirkende Druckschriften, mit getrennten Auswahlfeldern für Titel- und Textschrift statt fest verdrahteter Vorlagen-Typografie. Eingebettete Optionen: Fleur De Leah, Hurricane, Ruthie, Island Moments, Jacquard 24, Uncial Antiqua, UnifrakturMaguntia, New Rocker, UnifrakturCook, Manufacturing Consent, Almendra Display, Splash, Estonia, Monsieur La Doulaise, Pirata One und Astloch.
- Vorlagenspezifische Standardschriften:
  - Brief: Hurricane für Titel, Fleur De Leah für Text
  - Zeitung: UnifrakturMaguntia für Titel, Uncial Antiqua für Text
  - Steckbrief: Manufacturing Consent für Titel, New Rocker für Text
- Beim Vorlagenwechsel werden Standardschriften nur aktualisiert, wenn die aktuelle Schrift noch dem alten Vorlagenstandard entspricht. Manuell gewählte Schriften bleiben erhalten.
- Vorlagenspezifische Layoutvarianten:
  - Brief: Förmlicher Brief, kompakter Erlass, amtlicher Befehl, persönliche Notiz
  - Zeitung: Breites Zeitungsblatt, Gazette-Blatt, öffentlicher Aushang
    - Das breite Zeitungsblatt behält das klassische zweispaltige Zeitungslayout.
    - Gazette-Blatt ist enger und kompakter, mit drei Spalten und kleinerem Zeitungskopf.
    - Öffentlicher Aushang wird zu einer größeren einspaltigen Bekanntmachung mit zentriertem Text.
  - Steckbrief: Amtlicher Haftbefehl, rauer Aushang, Belohnungsplakat
    - Amtlicher Haftbefehl behält den sauberen gerahmten Amtsstil.
    - Rauer Aushang nutzt einen gestrichelten, linksbündigen, leicht unregelmäßigen Anschlagstil.
    - Belohnungsplakat betont Titel und Belohnungsblock durch stärkere Rahmung.
- Zeitungsbild-Auswahl mit Platzierung im Artikeltextfluss:
  - Anfang
  - Mitte
  - Ende
- Druckbare Bilder im Dokument werden standardmäßig schwarzweiß gerendert.
- Steuerungen für Zeilenabstand und Seitenrand
- Kuratierte Ornamentstile für Trenner und Schmuckelemente:
  - Fleurons
  - Kreuze
  - Ranken
  - Sterne
- Historische Ornamenttrenner für Betreff-/Kopftrennung statt schlichter moderner Linien
- Überlaufwarnung, wenn der Inhalt nicht mehr auf eine DIN-A4-Seite passt
- Vorschautext vermeidet fest eingebaute Formulierungen wie ein nicht editierbares Betreff-Präfix. Wenn ein Label wie `Betreff:` gewünscht ist, gehört es in das Betrefffeld.

## Vorlagenverhalten

Das Formular verwendet ein gemeinsames Datenmodell und bildet es in die jeweilige Druckvorlage ab:

- Brief:
  - `Von`, `An`, `Ort`, `Datum`, `Betreff`, `Text`, `Grußformel`, `Name/Titel der Unterschrift`, `Nachschrift / Notizen`
- Zeitung:
  - `Zeitungstitel`, `Ort`, `Datum`, `Schlagzeile`, `Artikeltext`, `Fußzeile / Notiz`
  - optionales Artikelbild innerhalb der Artikelspalten
- Steckbrief:
  - `Ort`, `Datum`, `Gesuchter Name / Titel`, `Beschreibung / Anschuldigung`, `Behördenzeile`, `Verantwortlicher Name/Titel`, `Belohnung / Notiz`

## Geplante Vorlagen

Aktuell keine. Neue Vorlagen erst hinzufügen, wenn sie rendern, drucken und die Validierung bestehen. Deaktivierte Fantasien bleiben Fantasien, auch wenn sie in einem Dropdown stehen.

Die aktuellen Prüfungen stellen sicher, dass `index.html` portabel bleibt und das erwartete Verhalten für A4, Sprachen, Vorlagen, Druck, historische Schriften/Hintergründe, editierbare Vorschau und Dateisicherheit enthält.

## Dateien

```text
index.html                 Portables Endnutzer-Werkzeug
README.md                  Nutzungs- und Projektnotizen
```
