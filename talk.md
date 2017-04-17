---
title: Hallo Alexa!
separator: <!--s-->
verticalSeparator: <!--v-->
theme: my-custom
highlightTheme: solarized-light
scripts: mermaid.min.js,mermaid-config.js
---
# Hallo Alexa!

Stefan Siegl (<stefan.siegl@mayflower.de>)

<!--s-->
# Wer ist Alexa?

* Amazon's *Alexa Voice Service* ist das sprachverarbeitende Backend
* Frontends
 * Amazon Echo
 * Amazon Echo Dot
 * Android App: u.a. Reverb
 * Eigenbau: Raspberry Pi

Note:
Ansprachen:
* "Alexa, wer bist du?"
* "Alexa, wie alt bist du?"

Lustigerweise geht "Alexa, wie alt bist du eigentlich?" nicht ;-)

<!--s-->
# Arten der Interaktion

* Ansprache immer über Audio-Eingabe
* Antwort dann mind. per Sprache
* Skill kann in Alexa-App auch Karten anzeigen

<!--v-->
# Skill Arten

* Alexa-eigene Interaktionen
* Custom Interaction Skill
* Smart Home Skill
* Flash Briefing Skill

<!--v-->
## Alexa-eigene Interaktionen

* wie ist das Wetter?
* wird es morgen Regnen?
* wecke mich um 6 Uhr morgens auf

Note:
* diese Art von Interaktion kann leider nur Amazon bereitstellen
* Features Timer bzw. Wecker stellen können von einem third-party Skill
  auch nicht auf anderem Weg bereitgestellt werden
  (custom skills sind immer passiv)

<!--s-->
# Custom Skills

* frage **Ken Guru** nach einem falschen Zitat
* gib mir ein falsches Zitat von **Ken Guru**
* frage **Widerstand Farbcodes** was sind die Farben für zehn Kiloohm

Note:
* ein Custom Skill muss immer direkt angesprochen werden
* es muss immer der *invocation name* genannt werden

<!--v-->
## Ansprache ohne weitere Absicht
* Alexa, öffne     _invocation name_
* Alexa, starte    _invocation name_
* Alexa, spiele    _invocation name_
* Alexa, sprich zu _invocation name_

<!--v-->
## Ansprache mit Absicht
* Alexa, frag'     _invocation name_ **intent**
* Alexa, sage      _invocation name_ **intent**

* Alexa, **intent** von   _invocation name_
* Alexa, **intent** mit   _invocation name_

<!--v-->
## Dialoge

Beispiel aus Doku:

* Alexa, **get high tide** from _tide pooler_.
* For what city?
* Seattle.
* Wednesday February 17th in Seattle, ...

Note:
* auch hier wieder der *invocation name* in der ersten Ansprache
* wenn der Skill eine Frage stellt, kann er die Session offen lassen
* das Alexa Framework kümmert sich dann ggf. auch um einen "re-prompt"

<!--s-->
# Smart Home Skills

* Alexa enumeriert vorab die Geräte
* Interaktionsmuster sind fest vorgegeben
* Gerät muss nicht alles implementieren, nur soweit sinnvoll

<!--v-->
## Beispiele

* Alexa, suche meine Geräte
* Schalte das **Licht im Schlafzimmer** ein
* Dimme das **Licht im Wohnzimmer** auf 50%
* Stelle die Temperatur im **Schlafzimmer** auf 23 Grad

Note:
* Wenn man ein neues Gerät hat, muss man immer wieder die Geräte neu suchen lassen
* bei der inquiry gibt jedes Gerät einen eigenen "friendly name" an
* die Geräte können in der Alexa App in Gruppen zusammengefasst werden

<!--s-->
# Flash Briefing

* Alexa, was sind die Nachrichten?
* Alexa, was ist meine tägliche Zusammenfassung?
* Alexa, gib mir meine tägliche Zusammenfassung

Note:
Ergebnis: von allen solchen Skills werden Meldungen eingesammelt
und nacheinander vorgelesen.

<!--s-->
# Architektur

<div class="mermaid">
sequenceDiagram
    participant Echo
    participant App
    participant AVS
    participant Skill
    Echo->>+AVS: Spracheingabe
    Note right of AVS: Verarbeitung + Intent-Interpretation
    AVS->>-Skill: JSON
    activate Skill
    Note right of Skill: Verarbeitung + ggf. Seiteneffekte
    Skill->>AVS: SSML + Karte
    deactivate Skill
    AVS->>Echo: Sprachausgabe
    AVS-->>App: Karte
</div>

<!--v-->
## Intent-Interpretation

Custom Skill deklariert (pro Sprache)

  * Intent Schema
  * Sample Utterances

Ein Intent kann Platzhalter enthalten: Slots

Slots müssen wohldefiniert sein!

<!--v-->
## Alexa ist ganz schön doof

* wie Intent-Mapping geschieht, ist ein "Geheimnis" von Amazon
* Slots haben Enumeration, kann aber auch "null" oder ganz was anderes sein

Note:
* AVS macht zunächst Spracherkennung, hat dann Text
* (offenbar) relativ wenig Sprachanalyse, einfache String-Vergleiche
* Beispiel:
  * ein Skill hat zwei Intents, einen relativ kurz, der andere relativ lang
  * auch wenn man jetzt totalen Unsinn redet, aber relativ viel sagt, dann wird
    Alexa den längeren der beiden Intents auslösen
* bei Slots wichtig, dass Wortanzahl passt
* das mit den Slots und der Wortanzahl kann man auch für "Freitext Slots" verwenden,
  die es an sich nicht gibt

<!--v-->
## Intent Schema

```json
{
  "intents": [
    {
      "intent": "GetNewRandomQuoteIntent"
    },
    {
      "intent": "AMAZON.HelpIntent"
    },
    {
      "intent": "AMAZON.StopIntent"
    },
    {
      "intent": "AMAZON.CancelIntent"
    }
  ]
}
```

<!--v-->
## Sample Utterances

```text
GetNewRandomQuoteIntent nach einem Zitat
GetNewRandomQuoteIntent nach einem falschen Zitat
GetNewRandomQuoteIntent nach einem falschen Zitat vom Känguru
GetNewRandomQuoteIntent sag mir ein Zitat
GetNewRandomQuoteIntent sag mir ein falsches Zitat
GetNewRandomQuoteIntent sage mir ein Zitat
GetNewRandomQuoteIntent sage mir ein falsches Zitat
GetNewRandomQuoteIntent gib mir ein Zitat
GetNewRandomQuoteIntent gib mir ein falsches Zitat
GetNewRandomQuoteIntent erzähle mir ein Zitat
```

<!--s-->
# Vergleich

|                       | Custom               | Smart Home          | Flash Briefing     |
|-----------------------|----------------------|---------------------|--------------------|
| Art der Kommunikation | frei                 | vorgegeben          | vorgegeben         |
| Dialoge möglich?      | ja                   | nein                | nein               |
| Ausgabe               | Text, ggf. Karte     | "ok"                | Text + Karte       |
| SSML                  | ja                   | nein                | nein               |
| eigenes Audio         | ja                   | nein                | ja                 |
| OAuth                 | kann                 | muss                | nein               |
| Backend               | HTTPS oder Lambda    | Lambda              | RSS via HTTP/HTTPS |

Note:
* Custom Skills sind komplett frei in ihrer Art der Kommunikation,
  d.h. es sind längere Dialoge möglich, die dann im Backend in einer Session abgebildet werden
  * Ausnahme hier bei Custom Skill die Neuinstallation -> UserId wird neu vergeben
* bei Custom und Smart Home Skills kann man den Nutzer (wieder)erkennen
* mit OAuth ist auch Personenzuordnung möglich
* Custom Skills können Musikwiedergabe steuern

<!--v-->
## User Perspektive

* Custom Skill: mäh, Interaktionsmodell lernen
* Smart Home: leichter Einstieg weil bekanntes Schema
* Flash Briefing: langweilig!?

<!--v-->
## DEV Perspektive

* Custom Skill: cool, geht am meisten
* Smart Home: schon ok, OAuth nervt
* Flash Briefing: mäh

Note:
* OAuth kann man bei Bedarf relativ einfach mit "Login with Amazon" erschlagen

<!--s-->
# Datenaustausch

Beispiel für Custom Skill

<!--v-->
## Request

```json
{
  "session": ...,
  "request": {
    "type": "IntentRequest",
    "requestId": "EdwRequestId.306e0d8d-eb77-4332-ba51-f9c00cc1fb71",
    "locale": "de-DE",
    "timestamp": "2017-04-15T10:54:53Z",
    "intent": {
      "name": "GetNewRandomQuoteIntent",
      "slots": {}
    }
  },
  "version": "1.0"
}
```

Note:
* request.type ist in der Regel IntentRequest; aber auch: StartSession
* locale: kann verwendet werden um Antwort-Sprache zu steuern, wenn Backend für
  mehrere Sprachen genutzt wird

<!--v-->
## Request.session

```json
{
    "sessionId": "SessionId.38b89391-41e6-4619-a50f-5a0be7167e95",
    "application": {
      "applicationId": "amzn1.ask.skill.48da98d2-94f8-41e0-967e-80704f167134"
    },
    "attributes": {},
    "user": {
      "userId": "amzn1.ask.account.AHX3TWXGFLO2BJM56WEEP5KYCCD22EQKHF2UWJOG3CNVSSYBONDGQWC26GFIZ5C7EJPNVPSUCK7P4U2PPVHXVMVTCRXQOB5K2WXOJFJRHKLGXSXGSPYCEAQKMD6HATTOXYUEVI2RKZQ2N7YTTJ7O2CGUUVPIRDWDP75ZVGD5CF7NTM44CXT2ED637ZRBAC3GELHNWQ5BHSKUCFQ"
    },
    "new": true
  }
```

Note:
* sessionId: unique während einer Invocation
* attributes: ist ein Objekt, das der Skill während der Session selbst mit Inhalt füllen kann,
  initial ist dieses natürlich leer
* userId: ein Identifier, der bei Aktivierung des Skills durch den Nutzer vergeben wird
  und auch über mehrere Sessions hinweg konstant bleibt;
  lediglich bei Re-Aktivierung des Skills erfolgt Neuvergabe
* new: flag, ob die Session neu ist

<!--v-->
## Response

```json
{
  "version": "1.0",
  "response": {
    "outputSpeech": {
      "type": "SSML",
      "ssml": "<speak> ... </speak>"
    },
    "card": {
      "content": "...",
      "title": "Känguru's Falsche Zitate",
      "type": "Simple"
    },
    "shouldEndSession": true
  },
  "sessionAttributes": {}
}
```

Note:
* tellWithCard Antwort
* tell = Session wird beendet; keine Fragestellung
* sessionAttributes: nur relevant bei "ask", hier kann ein Zustand mitgeschleppt werden

<!--s-->
# @TODO

* SSML
* evtl. Karten!?
* Smart Home/IoT/MQTT Aufbau
* Pricing
