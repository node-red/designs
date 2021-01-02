---
state: draft
---

# DE translation rules and dictionary

## Summary

This describes the german language specific translation rules.

## Authors

 - @heikokue

## Details

Bei der Übersetzung ins Deutsche sollen folgende Regeln (ggf. in absteigender Priorität) gelten:<br/>
For the translation to german there should apply the following rules:

0. Zielgruppe (Niveau der Übersetung) ist der computertechnisch erfahrene Endbenutzer, der aber nicht Informatik studiert hat und der auch kein IT-Entwickler ist. (The target group (level of translation) is the end user experienced in computer technology, but who has not studied computer science and is also not an IT developer.)
   Es kann davon ausgegangen werden, dass er gängige IT-Begriffe und -Vorgänge und deren Bedeutung kennt (z.B. Download, Upload, Speichern, ...), sodass diese nicht weiter erläutert werden müssen. (It can be assumed that he knows common IT terms and processes and their meaning (e.g. download, upload, save, ...) so that they do not have to be explained further.)
   IT-Spezialistenwissen sollte dagegen ausführlicher umschrieben werden (z.B. Node-RED- und Git-Begriffe). (IT specialist knowledge, however, should be described in more detail (e.g. Node-RED and Git terms))
0. Begriffe sollen möglichst einheitlich übersetzt werden, soweit möglich/sinnvoll (Terms should be consistent translated as possible/sensible). Siehe/see [https://github.com/heikokue/node-red/blob/i18n-de/dev/packages/node_modules/%40node-red/editor-client/locales/de/dictionary.csv dictionary.csv].
0. Zweifelsfälle in der deutschen Gemeinschaft besprechen und von ihr Reviews durchführen lassen (Discuss cases of doubt in the german cummunity and let them do reviews): [https://node-red.slack.com/archives/CK09P5RHR Slack-channel i18n-de]
0. Eigennamen werden 1:1 übernommen (Proper names will overtaken 1:1). Beispiele/Examples:
   Node-RED
   Namen von Nodes (Names of Nodes)
0. Stehende Node-RED-Begriffe (Hauptwörter) werden 1:1 übernommen, jedoch ggf. groß geschrieben (Proper Node-RED-Terms (nouns) will overtaken 1:1, but with a capital first letter where appropriate). Beispiele/Examples:
   Node
   Flow & Subflow
0. Gängige (eingedeutsche) IT-Begriffe werden 1:1 übernommen, jedoch ggf. groß geschrieben (Usual (in german assimilated) IT-terms will overtaken 1:1, but with a capital first letter where appropriate). Siehe/see [https://github.com/heikokue/node-red/blob/i18n-de/dev/packages/node_modules/%40node-red/editor-client/locales/de/dictionary.csv dictionary.csv]. Beispiele/Examples:
   Upload & Download
   zippen
   klicken
   Icon
   Review
0. Bei der Übersetzung sollen übliche IT-Begriffe verwendet werden (For the translation there should be used usual german terms). Siehe/see dictionary /editor-client/locales/de/dictionary.csv. Beispiele/Examples:
   Environment variables => Umgebungsvariablen
   GUI => Bedienoberfläche
0. Es sollte nicht "zwanghaft" übersetzt werden, wenn dann Wortungetüme, Zungenbrecher oder seltene Akademikerwörter herauskommen oder wenn sich kein gleichbedeutendes "griffiges" Wort findet. Im Zweifelsfall erstmal das Wort 1:1 übernehmen (ggf. groß geschrieben). (It should not be translated "obsessively" when out-of-the-way words, tongue twisters or rare academic words come out or when no synonymous "catchy" word can be found. If in doubt, use the word 1:1 first, but with a capital first letter where appropriate.) Siehe/see [https://github.com/heikokue/node-red/blob/i18n-de/dev/packages/node_modules/%40node-red/editor-client/locales/de/dictionary.csv dictionary.csv].
0. Denglisch soll vermieden werden, insbesondere "Ververbungen" wie z.B. "abbranchen", "committen", "reverten", "deployen". Da besser ein deutsches Verb benutzen und das englische Ausgangswort in Klamern dahinter setzen. (Denglish should be avoided, especially verb-mixups such as e.g. "abbranchen", "committen", "reverten", "deployen". Better to use a German verb and put the original English word in brackets after it.) Siehe/see [https://github.com/heikokue/node-red/blob/i18n-de/dev/packages/node_modules/%40node-red/editor-client/locales/de/dictionary.csv dictionary.csv]. Beispiele/Examples:
   commit &rarr; übertragen (commit)
   branch &rarr; abzweigen (branch)
0. Zeichensetzung (punctuation)
   Kurze einzelne Sätze haben normalerweise keinen Punkt am Ende (Short single sentence has no dot at the end)
   Bei mehreren Sätzen endet jeder Satz mit einem Punkt (If several sentences, each sentence ends with a dot)
0. Stil (Style)
   Sätze sollen flüssig lesbar sein (Sentences should be easy to read)
   Die im Englischen übliche persönliche Anrede ("Du") des Benutzers sollte im Deutschen vermieden werden (The usual personal address ("Du") of the user in English should be avoided in German)
   Sätze oder Einzelwörter werden groß geschrieben soweit sinnvoll (Sentences and single-words have a capital first letter as far as sensible)
0. Maschinelle Übersetzungen sollen nicht ungeprüft übernommen werden. (Machine translations should not be accepted without checking.) Siehe/see [https://github.com/heikokue/node-red/blob/i18n-de/dev/packages/node_modules/%40node-red/editor-client/locales/de/dictionary.csv dictionary.csv].

# History

This should be a list of major milestones in the life of the proposal. For example:

- 2021-01-02 - Initial proposal submitted
- yyyy-mm-dd - Moved to in-progress
- yyyy-mm-dd - Shipped in Node-RED 0.20 - moved to complete folder
