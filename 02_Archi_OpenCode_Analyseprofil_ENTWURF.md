# Archi – OpenCode-Analyseprofil (ENTWURF)

**Stand:** 20. Oktober 2026 · **Version:** 0.1  
**Status:** Separat gespeicherter Vorschlag; nicht aktiviert.  
**Zielprojekt:** C:\Users\USER\Documents\ChatGPT\Archi  
**Modellwahl:** ollama/gpt-oss-20b-32k, wie bereits beschlossen.

## 1. Ziel und getroffene Entwurfsentscheidung

Das Profil soll vorhandene Projektdateien lesen, Verzeichnisse auflisten und öffentliche Primärquellen von OpenCode und Ollama abrufen können. Änderungen, Shellbefehle, Code Mode, Browsersteuerung, Unteragenten und zusätzliche Integrationen werden im Vorschlag gesperrt. Internet ist erlaubt; die erste Quellenfreigabe ist bewusst klein.

**Ausgewählt:** OpenCodes vorhandene V2-Permissions, ergänzt um dessen Policies. Keine eigene Rechteengine und keine neue Infrastruktur.

| Vorschlag | Fähigkeiten | Pro | Contra / Grenze |
|---|---|---|---|
| Eigenes primäres Analyseprofil mit Permissions und zusätzlichen Policies | Projektlesen und Abruf festgelegter Dokumentationsseiten; übrige Werkzeugaktionen grundsätzlich verweigern | Vorhandene Mechanik; Regeln einzeln prüfbar; Policies berücksichtigen auch abweichende Agentenrechte | Versionsabhängig; Zusammenführung mit weiteren Konfigurationen muss geprüft werden; keine Betriebssystem-Sandbox |

Glob und Grep bleiben in dieser ersten Fassung gesperrt: Ihre Berechtigungsressource ist das Suchmuster, bei Grep der reguläre Ausdruck, nicht das Suchverzeichnis. Eine Pfadfreigabe lässt sich daraus nicht ableiten. Für die wenigen Archi-Dateien genügt zunächst die Verzeichnisauflistung durch Read. Das ist eine konservative Entwurfsentscheidung, kein belegter Fehler dieser Suchwerkzeuge. [Permissions](https://opencode.ai/v2/docs/permissions/), [Tools](https://opencode.ai/v2/docs/tools/)

## 2. Nachweisstand und Voraussetzungen

**Aus dem abgeglichenen Vorgängerbericht übernommen, hier nicht erneut als Laufzeittest ausgeführt:**

- Die globale Datei C:\Users\USER\.config\opencode\opencode.json enthielt das Zielmodell, den Ollama-Endpunkt http://127.0.0.1:11434/v1 und websearch:false. Das war ein Dateibeleg.
- Ollamas Metadaten-Endpunkte waren erreichbar; das Zielmodell war damals geladen. Das ist eine zeitgebundene Beobachtung. Eine Inferenz durch uns wurde nicht ausgeführt.
- Der untersuchte OpenCode-Debug-Endpunkt antwortete mit 401. Aktiver Agent, wirksame zusammengeführte Regeln, gespeicherte Freigaben und OpenCode-MCP-Verbindungen blieben UNVERIFIED.
- Desktop-Dateiversion 2.0.26 und Backend-Dateimetadatum 1.4.2 beweisen nicht, dass die aktive Installation das heutige V2-Schema vollständig unterstützt.

**Einordnung des zuletzt eingebrachten Rechteberichts:** Fehlende explizite Regeln in einer Datei beweisen keine Sperre. Die V2-Dokumentation nennt erlaubende Agenten-Grundregeln. websearch:false entfernt das Suchwerkzeug, verbietet aber nicht automatisch Webfetch oder andere Netzwerkwege. Eine leere LM-Studio-MCP-Datei ist kein Beleg für OpenCodes Integrationen. [Permissions](https://opencode.ai/v2/docs/permissions/), [Websearch](https://opencode.ai/v2/docs/websearch/)

**Voraussetzung für jede spätere Anwendung:** Die aktive OpenCode-Location muss genau dem Zielprojekt entsprechen. Ein zusätzlich als intern behandelter Projekt-Worktree darf den Lesebereich nicht über Archi hinaus erweitern. Der vorliegende Ordner ist beim aktuellen Abgleich kein Git-Repository. Eine falsch gewählte Location wird durch diesen JSON-Vorschlag nicht auf den genannten absoluten Pfad zurückgesetzt.

Das am 20.10.2026 gelesene [öffentliche Schema](https://opencode.ai/config.json) beschreibt die verwendeten Felder. Die lokale Unterstützung und Wirksamkeit bleiben bis zur gesonderten Prüfung offen.

## 3. Konkreter JSON-Vorschlag

Dieser Block ist ausschließlich Entwurfsinhalt. Er wurde nicht als automatisch geladene opencode.json oder opencode.jsonc gespeichert. Er ergänzt bei einer später ausdrücklich freigegebenen Anwendung die vorhandene Providerdefinition; er ist kein Ersatz für die globale Konfiguration. Endpunkt, Modellregistrierung und bestehende Provider-Policies werden hier nicht neu definiert.

Die Wiederholung der Regeln ist beabsichtigt: gemeinsame Permissions, explizite Regeln des ausgewählten Agenten und zusätzliche Policy-Prüfung. Breite Sperren stehen vor den Ausnahmen; Umgebungsdatei-Sperren folgen der Lesefreigabe.

~~~json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "ollama/gpt-oss-20b-32k",
  "default_agent": "archi-analyse",
  "websearch": false,
  "lsp": false,
  "formatter": false,
  "permissions": [
    { "action": "*", "resource": "*", "effect": "deny" },
    { "action": "read", "resource": "*", "effect": "allow" },
    { "action": "read", "resource": "..", "effect": "deny" },
    { "action": "read", "resource": "../*", "effect": "deny" },
    { "action": "read", "resource": "*.env", "effect": "deny" },
    { "action": "read", "resource": "*.env.*", "effect": "deny" },
    { "action": "webfetch", "resource": "https://opencode.ai/v2/docs/*", "effect": "allow" },
    { "action": "webfetch", "resource": "https://opencode.ai/config.json", "effect": "allow" },
    { "action": "webfetch", "resource": "https://docs.ollama.com/*", "effect": "allow" },
    { "action": "question", "resource": "*", "effect": "allow" }
  ],
  "agents": {
    "archi-analyse": {
      "description": "Archi-Projekt lesen und freigegebene öffentliche Dokumentation auswerten",
      "mode": "primary",
      "model": "ollama/gpt-oss-20b-32k",
      "permissions": [
        { "action": "*", "resource": "*", "effect": "deny" },
        { "action": "read", "resource": "*", "effect": "allow" },
        { "action": "read", "resource": "..", "effect": "deny" },
        { "action": "read", "resource": "../*", "effect": "deny" },
        { "action": "read", "resource": "*.env", "effect": "deny" },
        { "action": "read", "resource": "*.env.*", "effect": "deny" },
        { "action": "webfetch", "resource": "https://opencode.ai/v2/docs/*", "effect": "allow" },
        { "action": "webfetch", "resource": "https://opencode.ai/config.json", "effect": "allow" },
        { "action": "webfetch", "resource": "https://docs.ollama.com/*", "effect": "allow" },
        { "action": "question", "resource": "*", "effect": "allow" }
      ]
    }
  },
  "policies": [
    { "action": "tool.use", "resource": "*", "effect": "deny" },
    { "action": "tool.use", "resource": "read:*", "effect": "allow" },
    { "action": "tool.use", "resource": "read:..", "effect": "deny" },
    { "action": "tool.use", "resource": "read:../*", "effect": "deny" },
    { "action": "tool.use", "resource": "read:*.env", "effect": "deny" },
    { "action": "tool.use", "resource": "read:*.env.*", "effect": "deny" },
    { "action": "tool.use", "resource": "webfetch:https://opencode.ai/v2/docs/*", "effect": "allow" },
    { "action": "tool.use", "resource": "webfetch:https://opencode.ai/config.json", "effect": "allow" },
    { "action": "tool.use", "resource": "webfetch:https://docs.ollama.com/*", "effect": "allow" },
    { "action": "tool.use", "resource": "question:*", "effect": "allow" },
    { "action": "integration.use", "resource": "*", "effect": "deny" }
  ]
}
~~~

Syntaxgrundlage: [Config](https://opencode.ai/v2/docs/config/), [Agents](https://opencode.ai/v2/docs/agents/), [Schema](https://opencode.ai/config.json). Die Regeln sind eigene Entwurfswerte auf Grundlage dieser Schnittstellen.

## 4. Vorgesehene Wirkung

Die folgende Tabelle beschreibt die Absicht nach dokumentierter V2-Auswertung, keine beobachtete Laufzeitwirkung.

| Aktion / Fähigkeit | Vorgesehenes Ergebnis | Begründung oder Einschränkung |
|---|---|---|
| Read einer normalen Archi-Datei | ALLOW | Relative interne Pfade; vorausgesetzt, Location/Worktree stimmen |
| Read eines Archi-Verzeichnisses | ALLOW | Liefert eine nicht rekursive Auflistung; keine Shell nötig |
| Read von .env, .env.local oder verschachtelten entsprechenden Dateien | DENY | Nachgelagerte Muster; auch .env.example bleibt gesperrt |
| Read außerhalb von Location und Projekt-Worktree | DENY | Zusätzlicher external_directory-Check fällt unter die allgemeine Sperre |
| Read eines normalisierten relativen Elternpfads .. oder ../… | DENY | Zusätzliche Read-Sperre, auch für einen andernfalls intern behandelten Elternpfad |
| Edit, Write, Patch | DENY | Nutzen die Action edit; keine Ausnahme |
| Shell, auch vermeintlich nur lesende Befehle | DENY | Keine Ausnahme für shell; keine unsichere Befehls-Blockliste |
| Execute / Code Mode und Browserkatalog | DENY | Keine Ausnahme für execute oder browser |
| Unteragenten | DENY | Keine Ausnahme für subagent; keine automatische Rechtevererbung vorausgesetzt |
| Glob, Grep | DENY | Bewusst keine Freigabe in dieser ersten Fassung |
| Webfetch der drei URL-Muster im JSON | ALLOW | Öffentliche Dokumentation; keine Anmeldung oder neue Zugangsdaten vorgesehen |
| Andere URLs, einschließlich direkt angefragter localhost-/LAN-Adressen | DENY | Nicht in der URL-Liste; Weiterleitungen sind damit noch nicht geprüft |
| Websearch / Suchmaschinenabfragen | NICHT VERFÜGBAR vorgesehen | websearch:false bleibt erhalten; keine Suchanbieter-Einrichtung |
| MCP, Skills und zusätzliche Plugins | DENY im Profil | Keine Einzelintegration für diesen Baustein freigegeben; integration.use sperrt Laden, tool.use zusätzlich geprüfte Aufrufe |
| Rückfragen an den Nutzer | ALLOW | question:* erlaubt Interaktion, erteilt selbst keine zusätzlichen Werkzeugrechte |

Die automatische Recherche umfasst hier den Abruf bekannter Dokumentations-URLs. Eine freie Websuche ist damit nicht eingerichtet. Ein späterer Suchanbieter oder eine weitere Quelle wäre eine gezielte Erweiterung; daraus folgt keine grundsätzliche Offlinepflicht. [Websearch](https://opencode.ai/v2/docs/websearch/)

LSP und Formatter sind im Vorschlag zusätzlich ausgeschaltet. Das reduziert diese vorgesehenen Hilfsfunktionen; es ist kein Beweis, dass OpenCode keinerlei Hintergrundprogramme oder interne Schreibvorgänge ausführt. Die Oberfläche benötigt weiterhin unter anderem eigene Sitzungsdaten. [Schema](https://opencode.ai/config.json), [Config](https://opencode.ai/v2/docs/config/)

## 5. Warum Permissions allein nicht genügen

Agentenregeln werden nach gemeinsamen Permissions angehängt. Unteragenten verwenden eigene Rechte. Deshalb ist weder ein einzelnes globales Regelarray noch der eingebaute Plan-Agent eine ausreichende Begrenzung für diesen Zweck. Ein bestehender Chat übernimmt durch default_agent auch nicht automatisch einen anderen Agenten; gespeicherte Modellwahl und aktiver Agent müssen später getrennt kontrolliert werden. [Agents](https://opencode.ai/v2/docs/agents/)

Die ergänzende tool.use-Policy prüft die Kombination aus Action und Ressource. Ihr Allow hebt nur die vorherige Policy-Sperre für diese Ressource auf; die Agenten-Permission muss zusätzlich erlauben. Nicht freigegebene geprüfte Aktionen bleiben dadurch auch bei breiteren Agentenrechten gesperrt. [Policies](https://opencode.ai/v2/docs/policies/)

**Grenzen:**

- Höhere Policy-Quellen, etwa globale, System- oder Console-Regeln, werden später ausgewertet und können Projektregeln auch durch Allow verändern. Die vollständig wirksame Reihenfolge ist hier nicht nachgewiesen.
- Policies greifen bei den jeweiligen Prüfstellen. Sie sind keine allgemeine Prozess- oder Netzwerksperre und sandboxen keinen beliebigen Plugin-Code. Code Mode bleibt deshalb gesperrt; dessen API hat auch direkte Netzwerk- und Hilfsfunktionen. [Policies](https://opencode.ai/v2/docs/policies/), [Tools](https://opencode.ai/v2/docs/tools/)
- Die Lesefreigabe hängt von der Location ab. Junctions, symbolische Links und andere Pfadbesonderheiten sind lokal nicht getestet. Windows-Pfade werden laut Dokumentation normalisiert; das beweist keine NTFS-Isolation. [Permissions](https://opencode.ai/v2/docs/permissions/)
- Die URL-Liste ist eine Prüfung angefragter Zeichenfolgen. Redirects, DNS-Auflösung und endgültige Zieladressen sind damit nicht als auf öffentliche Netze beschränkt nachgewiesen. Auch erlaubte URL-Pfade können Informationen enthalten. Kein Nachweis einer technischen Verhinderung von Datenabfluss.
- Verhaltensregel für die spätere Nutzung: Keine lokalen Dateiinhalte, Geheimnisse oder privaten Angaben in URL-Pfade oder Suchparameter übernehmen. Diese Anweisung ersetzt keine technische Datenausgangskontrolle.
- Die .env-Muster sind eine kleine Zusatzsperre, kein vollständiger Geheimnisschutz. Anders benannte vertrauliche Dateien und bereits im Chat vorhandene Inhalte werden dadurch nicht entfernt. Verzeichnisauflistungen können weiterhin Dateinamen zeigen.
- Eine Berechtigung erzeugt kein Werkzeug und beweist keine zuverlässigen Tool Calls von GPT-OSS 20B. Wird ein benötigtes Werkzeug lokal nur über gesperrten Code Mode angeboten, bleibt es unbenutzbar; die Sperre wird dafür nicht automatisch gelockert.

**Einordnung:** Dieser Baustein ist ein Vorschlag zur Begrenzung modellgesteuerter Werkzeuge. Eine wirksame Betriebssystem-Sandbox, Archis Prüfpipeline, der zentrale Korrekturzähler und die Gesamtabnahme bleiben eigenständige offene Arbeiten.

## 6. Prüfprotokoll dieses Entwurfs

| Prüfung | Ergebnis / Nachweisgrenze |
|---|---|
| Eingebettetes JSON | PASS: genau ein JSON-Block; mit PowerShell ConvertFrom-Json vollständig geparst |
| Feldformen gegen aktuelles Online-Schema | PASS: Test-Json mit dem frisch abgerufenen Schema lieferte True; Prüfprozess Exitcode 0 |
| Actions und Regelreihenfolge | PASS als statische Prüfung: allgemeine Sperren zuerst; Eltern-/Umgebungsdatei-Sperren nach Lesefreigabe; alle Einzelregeln haben ein entsprechendes tool.use-Gegenstück |
| Modell- und Agentenbindung | PASS als statische Prüfung: identische 10 Permissions im gemeinsamen und Agenten-Array; beide Modellfelder unverändert auf dem Zielmodell; primärer Agent stimmt mit default_agent überein |
| Unabhängiges Quellenreview | PASS für diesen Dokumententwurf: zweiter Agent prüfte offizielle V2-Quellen und gespeicherte Fassung; zusätzliche Elternpfad-Sperren übernommen; danach keine weiteren konkreten Befunde |
| Erhalt vorhandener Dateien | PASS: SHA256 von globaler OpenCode-Konfiguration, Architekturdatei und beiden vorhandenen Übergabeberichten gegenüber dem Beginn dieses Auftrags unverändert |
| Keine aktive Projektkonfiguration angelegt | PASS: opencode.json/jsonc direkt in Archi und unter .opencode sind weiterhin nicht vorhanden |
| Unterstützung der lokal installierten OpenCode-Version | UNVERIFIED; kein Backend oder CLI gestartet |
| Tatsächliche Freigaben/Sperren, Projektgrenze, Redirects und Tool-Verfügbarkeit | UNVERIFIED; kein Anwendungstest |
| Modellinferenz und wirksame Sandbox | NICHT GETESTET / NICHT NACHGEWIESEN |

Beleg des verwendeten Online-Schemas: SHA256 der als UTF-8 codierten Antwortzeichenfolge war 786B5FFC800629BC22145325BEAA1DC4C7E6BDBBD5C5E6AFD8CD2D59D9FC863C. Das Schema wurde für die Prüfung nur gelesen und nicht als weitere Projektdatei gespeichert. Statische Prüfungen und Quellenreview sind ausdrücklich kein Nachweis lokaler Rechte-Durchsetzung.

Es wurden keine aktiven OpenCode-Dateien geändert, keine dauerhaften Freigaben erteilt oder zurückgesetzt, keine Reloads ausgelöst, keine Pakete installiert und keine Modelle oder Dienste durch diesen Auftrag gestartet oder gestoppt. Das ursprüngliche Architekturdokument und die vorhandenen Übergabeberichte bleiben erhalten.

## 7. Genau ein späterer nächster Schritt

**Einen gesondert freigegebenen, begrenzten Kompatibilitäts- und Rechtefunktionstest dieser Fassung vorbereiten und durchführen:** Zuerst aktive Version, Location und vollständig zusammengeführte Regeln prüfen; anschließend in einem ausdrücklich freigegebenen Testkontext eine harmlose Projektdatei und eine erlaubte Dokumentationsseite abrufen sowie die erwarteten Ablehnungen gezielt nachweisen. Keine bereits gesperrte gefährliche Aktion tatsächlich ausführen. Falls die aktive Version oder Werkzeugbereitstellung nicht passt, mit Befund stoppen und die Sperren erhalten.

Die spätere Aktivierung, nötige Testdateien und etwaige Inferenz sind durch die jetzige Entwurfsfreigabe nicht autorisiert. Dieser Bericht fragt keine erneute Zustimmung für die bereits erledigte Entwurfserstellung ab.

## 8. Quellen und lokale Grundlagen

Öffentliche Primärquellen, am 20.10.2026 gezielt gelesen:

- [OpenCode V2 Permissions](https://opencode.ai/v2/docs/permissions/) – Actions, Ressourcen, Muster und Agenten-Grundregeln.
- [OpenCode V2 Policies](https://opencode.ai/v2/docs/policies/) – zusätzliche Sperren und Rangfolge der Quellen.
- [OpenCode V2 Agents](https://opencode.ai/v2/docs/agents/) – primärer Agent, Zusammenführung und Unteragentenrechte.
- [OpenCode V2 Tools](https://opencode.ai/v2/docs/tools/) – Read, Suchwerkzeuge, Webfetch und Code Mode.
- [OpenCode V2 Websearch](https://opencode.ai/v2/docs/websearch/) – Anbieter und Bedeutung von websearch:false.
- [OpenCode V2 Config](https://opencode.ai/v2/docs/config/) und [Konfigurationsschema](https://opencode.ai/config.json) – verwendete Feldformen; keine lokale Versionsgarantie.

Lokale Grundlagen:

- C:\Users\USER\Documents\ChatGPT\Archi\01_Archi_Gesamtarchitektur.md, Version 1.3.
- C:\Users\USER\Documents\ChatGPT\Archi\CHAT-UEBERGABE-STATUS.json – bestätigt diesen Chat als einzigen Nachfolger und den Umfang der Entwurfsfreigabe.

Die ältere Datei CHAT-UEBERGABE.md ist keine zusätzliche Rechtequelle; ihre im aktuellen Übergabebericht benannten Fehlschlüsse wurden nicht übernommen.
