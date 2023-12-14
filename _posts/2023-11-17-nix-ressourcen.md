---
layout: post
title:  "Nix-Ressourcen"
tags: nix nixos
---

# Welche Ressourcen gibt es, um sich über Nix zu informieren?
Der Zugang zum Nix-Ökosystem ist nicht unbedingt einfach. Henrik Lissner bringt die vielen Fallstricke und Hürden wunderbar auf den Punkt, wenn er sich vehement gegen die Nutzung von NixOS [ausspricht](https://github.com/hlissner/dotfiles#frequently-asked-questions){:target="_blank"}. Schwer zu lernen, oberflächliche Dokumentation, unintuitive Sprache, zu viel Aufwand wenn es nur um die Verwaltung weniger Systeme geht - alles Erfahrungen, die ich in den letzten Wochen auch gemacht habe.

Diese Situation ist gerade deshalb frustrierend, weil sich Nix in aktiver Entwicklung befindet. Es ist zum Teil nicht immer leicht zu erkennen, ob Ausführungen überholten Prinzipien folgen oder Code-Listings veraltet sind. Das offenkundigste Beispiel dafür ist der Bruch, den die Einführung von Flakes bedeutete. Kanäle (*channels*) haben damit ihre Bedeutung verloren. Einige Kommandozeilen-Werkzeuge wurden im Wesentlichen durch andere ersetzt, die ähnlichen Zwecken dienen. Für einen Überlick über Änderungen, siehe einen [Abschnitt](https://nixos-and-flakes.thiscute.world/nixos-with-flakes/introduction-to-flakes#nix-flakes-and-classic-nix){:target="_blank"} im *NixOS & Flakes Book*.

Man muss also *wirklich* wollen. Ich will, und deshalb habe ich – primär für mich for future reference – in kommentierter Form all die Quellen zusammengetragen, die mir bisher geholfen haben, Nix und NixOS zumindest ein bisschen besser zu verstehen. Vielleicht hilft der Überblick auch anderen Einsteigern.

Ian Henry hat 2021 eine [Artikel-Serie](https://ianthehenry.com/posts/how-to-learn-nix/introduction){:target="_blank"} veröffentlicht, die mir völlig von der Seele spricht. Ich glaube wenn ich ein besserer Autor wäre, würde dieser Beitrag so ziemlich jeden Absatz daraus reproduzieren.

## Die offizielle Webseite und Dokumentation
Die [offizielle Webseite](https://nixos.org/){:target="_blank"} vom NixOS-Projekt gibt einen guten Überblick darüber, [wie Nix funktioniert](https://nixos.org/guides/how-nix-works){:target="_blank"} und was den Paketmanager gegenüber ähnlichen Systemen auszeichnet. Dadurch lernt man zwar, warum es gute Gründe dafür gibt, Nix und NixOS zu lernen und professionell oder privat einzusetzen. Klare Richtungsangaben für den weiteren Lernprozess sucht man aber leider vergebens.

Zugegeben, es gibt da die [offizielle Dokumentation](https://nix.dev/){:target="_blank"}, sogar mit [Tutorial](https://nix.dev/tutorials/){:target="_blank"} und [First Steps](https://nix.dev/tutorials/first-steps/){:target="_blank"}-Anleitung. Doch ich muss gestehen, dass ich es zum Einstieg eher entmutigend fand. Viele Neulinge wollen vielleicht einfach erstmal ihren Daily Driver mit VS Code und Firefox einrichten. Docker, virtuelle Maschinen, Terraform und Raspberry Pis haben zweifelos ihren Platz, aber suchen die meisten nicht zunächst etwas Bodenständigeres?

Deutlich hilfreicher ist das offizielle Bedienungshandbuch (*reference manual*). Die Einträge zum [Nix-Paketmanager](https://nixos.org/manual/nix/stable/){:target="_blank"}, zu [NixOS](https://nixos.org/manual/nixos/stable/){:target="_blank"}, zum [Nixpkgs-Repo](https://nixos.org/manual/nixpkgs/stable/){:target="_blank"} und zur [Nix-Sprache](https://nixos.org/manual/nix/stable/language/index.html){:target="_blank"} sind hervorragend, um erste Grundkenntnisse zu vertiefen. Wie so oft bei Software-Dokumentationen wird ein Wissen über Grundbegriffe und Zwecke aber leider vorausgesetzt, um sich in den umfassenden Dokumenten zurechtzufinden.

## Literatur
Ich weiß nicht, wie es anderen geht, aber mir persönlich fällt es schwer, das notwendige Grundverständnis der zugrundeliegenden Konzepte und Ansätze zu erwerben. Für gewöhnlich mag ich zum Einstieg in ein neues Thema die systematischen Darstellungen, wie Einführungsbücher sie geben. Mich überrascht, dass es trotz zunehmender Popularität (noch immer) keine Literatur dieser Art zu Nix und NixOS zu geben scheint. Nicht nur keine gute; Wissen scheint prinzipiell allein über Blog-Posts, Dokumentationen und Mouth-to-Mouth weitergegeben zu werden.

Die vielleicht beste Grundlage bietet [The Purely Functional Deployment Model](https://dspace.library.uu.nl/handle/1874/7540), einer 2006 von Eelco Dolstra verteidigten Dissertation. Die Arbeit präsentiert die Ergebnisse eines 2003 begonnenen Forschungsprojekts, das in NixOS resultierte. 

Es liegt in der Natur der Sache, dass sich viele Kapitel und Abschnitte an ein spezialisiertes Fachpublikum richten. Dennoch gibt insbesondere das Einleitungskapitel eine nicht allzu technische Erklärung vieler Grundideen des funktionalen Ansatzes. Vereinfachte Beispiele illustrieren, wie Pakete und Build-Vorgänge durch Nix-Ausdrücke beschrieben werden können.

Die Nix-Sprache ist vergleichsweise einfach aufgebaut. Wer sich für Computersprachen interessiert, findet einen erfrischend praktisch orientierte Darstellung ihrer Syntax und Semantik. Sie abstrahiert von Details über ihre konkrete Implementierung und ist deshalb denke ich bereits mit soliden Grundkenntnissen theoretischer Informatik zugänglich.

Auf Tweag gibt Dolstra eine sehr gute [Einführung](https://www.tweag.io/blog/2020-05-25-flakes/){:target="_blank"} in die fast schon revolutionäre Idee von Flakes. Die dreiteilige Artikelserie motiviert zunächst die neue Herangehensweise gegenüber der traditionellen Nutzung von Kanälen und zeigt die damit verbundenen Vorteile auf. Die weiteren Ausführungen gehen dann auf präzise Weise auf viele Details ein, die bei den knappen Darstellungen anderer Seiten keine Erwähnung finden.

## Blogs und andere Community-Beiträge
Das inoffizielle [NixOS-Wiki](https://nixos.wiki/wiki/){:target="_blank"} gehört zu den leserlicheren Ressourcen, um sein Verständnis zu zentralen Themen zu vertiefen. Leider ist es nicht so umfassend und regelmäßig gewartet, wie man vielleicht erwarten oder hoffen würde. Zwischen dem traditionellen Ansatz und einer Flakes-orientierten Herangehensweise wird nicht strikt unterschieden (wie etwa der Artikel über den [Nix-Paketmanager](https://nixos.wiki/wiki/Nix_package_manager){:target="_blank"} demonstriert).

Ich habe oben bereits kurz das [NixOS & Flakes Book](https://nixos-and-flakes.thiscute.world/){:target="_blank"} von Ryan Yin angesprochen. Dabei handelt es sich um einen Einstieg in die Verwendung von NixOS und der Systemkonfiguration mit Flakes. Ich finde das "Buch" gerade hilfreich, um Flake [Flake Inputs](https://nixos-and-flakes.thiscute.world/other-usage-of-flakes/inputs){:target="_blank"} und die verschiedenen Arten von [Flake Outputs](https://nixos-and-flakes.thiscute.world/other-usage-of-flakes/outputs){:target="_blank"} zu verstehen. Wer bereits weiß, was die Nixpkgs-Collection ist, findet [auf der Seite](https://nixos-and-flakes.thiscute.world/nixpkgs/intro){:target="_blank"} auch Erklärungen für eine fortgeschrittene Interaktion mit dem Repo.

Für mich neu war, dass Flakes nicht nur Systemkonfigurationen ausgeben können. Sie können so aufgebaut werden, dass bestimmte Versionen von Anwendungen mit bestimmten Versionen ihrer Dependencies ausgeführt werden können. Besonders faszinierend finde die Erklärung von [Entwicklungsumgebungen](https://nixos-and-flakes.thiscute.world/development/intro){:target="_blank"} und der intendierten Logik dabei. Zur Interaktion mit den erstellten Flakes dient ein [neues Kommandozeileninterface](https://nixos-and-flakes.thiscute.world/other-usage-of-flakes/the-new-cli){:target="_blank"}.

Einen ähnlichen Zweck verfolgt Rohit Goswamis ["A Tutorial Introduction to Nix"](https://rgoswami.me/posts/ccon-tut-nix/){:target="_blank"}. Der Artikel erklärt, wie Pythons virtuelle Umgebungen durch Nix' `mkShell` ersetzt werden können. Der Artikel folgte auf einen Fachkongress ([CarpentryCon@Home 2020](https://2020.carpentrycon.org/){:target="_blank"}), der von einer renommierten Bildungseinrichtung veranstaltet wird. Das spiegelt sich in der Qualität der Veröffentlichung.

Nix-Konfigurationen und Paket-Definitionen werden in der Nix-Programmiersprache geschrieben. Ein gutes Verständnis der Sprache ist deshalb für alle Zwecke unabdingbar. Yin [nennt](https://nixos-and-flakes.thiscute.world/the-nix-language/){:target="_blank"} drei Seiten, mit deren Hilfe man sich in gut zwei Stunden up-to-speed bringen kann. Die Seiten sind [Nix Language Basics (die offizielle NixOS-Dokumentation)](https://nix.dev/tutorials/first-steps/nix-language){:target="_blank"}, [Nix: A One Pager](https://github.com/tazjin/nix-1p){:target="_blank"} und der umfassende [Eintrag](https://nixos.org/manual/nix/stable/language/){:target="_blank"} im NixOS-Bedienungshandbuch. Empfehlenswert finde ich die Auswahl vor allem deshalb, weil sie verschiedene Aspekte der Sprache hervorheben und das überschaubare Material auf verschiedene Weisen strukturieren.

Die sogenannten [Nix Pills](https://nixos.org/guides/nix-pills/pr01){:target="_blank"} bilden eine Serie von Blog-Posts, die Luca Bruno 2014 und 2015 geschrieben hat und die später nochmal in leichter zugänglicher Form wiederveröffentlicht wurden. Sie richtet sich primär an Leute, die selbst auf aktive Weise zu Nix beitragen wollen. Gerade die späteren Artikel sind nicht unbedingt die leichteste Kost, aber wie der Name nahelegt, bekommt man sie in kleinen Happen serviert. Sehr gut, um sich ein Hintergrundwissen zu den Abläufen und zu Ideen wie den wichtigen Derivations anzueignen.

Einen ähnlichen Zweck verfolgt Justin Woo mit seinen [Nix-Shorts](https://github.com/justinwoo/nix-shorts){:target="_blank"}. Die Anzahl von Artikeln in der Serie ist überschaubar und die thematisierten Fragestellungen dürften für Anfänger von keinem unmittelbaren Interesse sein. Doch sie sind kurz, klar und betreffen Dinge, die später wahrscheinlich wichtig werden.

## Pakete und Optionen
Die Namensgebung populärer Anwendungen folgt generell einem intuitiven Schema. Das Paket für Neovim heißt `neovim`, das Paket für Firefox heißt `firefox`, das Paket für Emacs heißt `emacs`, you get the idea. Dennoch wird man sich häufig fragen, unter welchem Namen man ein bekanntes Paket in Nix findet. Dazu gibt es auf der offiziellen Seite eine [Suchmaschine](https://search.nixos.org){:target="_blank"}. Über einen weiteren Tab kann man auch nach Namen und Erklärungen der vielen Optionen suchen, die bei der NixOS-Konfiguration zur Verfügung stehen. Für die Optionen, die im Kontext von Flakes verwendet werden können, gibt es einen eigenen Tab.

## Vorlagen und Beispiele
Cole Mickens präsentiert eine [Minimalkonfiguration](https://github.com/colemickens/nixos-flake-example){:target="_blank"} für NixOS. Im Wesentlichen werden nur `flake.nix` und `configuration.nix` verwendet. In der README werden einige hilfreiche Erklärungen über die Verwendung von Flakes gegeben. Die vorangestellten Warnungen erscheinen mir aber eher ein wenig kryptisch.

Einem ähnlichen Zweck dient die [Nix Starter Config](https://github.com/Misterio77/nix-starter-configs){:target="_blank"}. In der Standard-Version erhöht sie den Komplexitätsgrad dadurch, dass der Home-Manager eingebunden und Overlays verwendet werden. In der README werden ausgewählte Aspekte ausgiebig erklärt und einige Vorschläge gemacht, wie man sie weiter ausbauen könnte.

Ein weiteres [Starter-Pack](https://github.com/LGUG2Z/nixos-wsl-starter){:target="_blank"} hat LGUG2Z speziell für WSL (dem Windows Subsystem for Linux) zusammengestellt. Anders als das Beispiel von Misterio77 werden dabei richtige Anwendungen eingebunden, insbesondere eine durch [LunarVim](https://www.lunarvim.org/){:target="_blank"} vorkonfigurierte Version von Neovim. Das Projekt wird von Jeezy in einem [Video](https://youtu.be/UmRXXYxq8k4?si=uS0GgGMLdWPg7oiJ){:target="_blank"} erläutert. In einem [anderen Video](https://youtu.be/LzRi9rPV2p4?si=Hy0YIi3jIYB3fJ0t){:target="_blank"} wird die Konfiguration ausgeweitet, um zugleich (lokal) für WSL und (remote) für NixOS in einer Hetzner-Cloud genutzt werden zu können. Sehr illustrativ, um einen Eindruck davon zu gewinnen, welche Rolle Abstraktionen bei Nix spielen.

[Digga](https://github.com/divnix/digga){:target="_blank"} nennt sich eine NixOS-Konfiguration, die von vielen weitaus fortgeschritteneren Techniken Gebrauch macht. Alles andere als einsteigerfreundlich, doch ein großer Schritt in Richtung wahres Leben (eines qualifiizierten Nix-Nutzers). Die Anmerkungen sind hilfreich, aber leider wieder nicht sehr umfangreich.

Der Home-Manager wird genutzt, um Dotfiles auf eine ähnliche Weise mit Nix zu verwalten wie das Kernsystem. Es versteht sich, dass sich der Komplexitätsgrad dadurch nochmal deutlich erhöht, gerade in Bezug auf umfassend konfigurierbare Anwendungen wie Neovim, Emacs oder ein Fenster-Manager.

Im *NixOS & Flakes Book* wird [erläutert](https://nixos-and-flakes.thiscute.world/nixos-with-flakes/modularize-the-configuration){:target="_blank"}, wie ein [System mit i3](https://github.com/ryan4yin/nix-config/tree/i3-kickstarter){:target="_blank"} konfiguriert werden kann. Dabei kommen Funktionen aus der `lib`-Standardbibliothek zur Anwendung und der modulare Aufbau spiegelt sich in einer komplexen Verzeichnisstruktur. Die verwendeten Techniken und Operationen werden sicher nicht in allen Details motiviert, aber trotzdem eine sehr  gute Vorlage, wenn man selbst i3 oder eine vergleichbare Anwendung verwenden möchte.

Matúš Beňko erklärt in einem kürzlich veröffentlichten [Blog-Post](https://primamateria.github.io/blog/neovim-nix/){:target="_blank"}, wie ein Flake für Neovim erstellt werden kann. Von Interesse ist die Konfiguration vor allem deshalb, weil sie auch auf die Verwaltung von Plugins eingeht. Nebenbei wird auch gezeigt, wie man ausführbare Flakes erstellt. Flakes dieser Art repräsentieren in jeder Hinsicht bestimmte Versionen eines Software-Pakets und können in dieser Form an andere Nix-Nutzer weitergegeben werden.

Eine [Konfiguration](https://gitlab.com/librephoenix/nixos-config/-/tree/main?ref_type=heads){:target="_blank"} von LibrePhoenix zeigt, wie verschiedene Profile (Persönlich, Arbeit, WSL, Homelab) in *einer* Konfiguration integriert werden können. Er verwendet Emacs und [Stylix](https://github.com/danth/stylix){:target="_blank"}, ein Modul, mit dem ein einheitliches Farbschema für verschiedene Anwendungen und Umgebungen gesetzt werden kann.

Oben habe ich Henrik Lissner für seine negative Haltung gegenüber NixOS zitiert. In Wahrheit hat er aber selbst eine der wohl als Vorlage meistgenutzen [Beispiel-Konfigurationen](https://github.com/hlissner/dotfiles){:target="_blank"} für NixOS veröffentlicht. Das Alleinstellungsmerkmal ist sein eigenes CLI-Werkzeug, `hey`, das die verschiedenen Befehle zur Interaktion mit Flakes vereinigt. Ich weiß nicht, ob er selbst seine Konfiguration einsetzt, sie scheint mir aber für den täglichen Gebrauch völlig geeignet zu sein. Leider gibt es keinerlei Dokumentation zu ihren Komponenten.

Henrik führt eine Reihe von "echten" Konfigurationen an, die ihm bei der Erstellung seiner eigenen geholfen haben:
- <a href="https://github.com/LEXUGE/nixos" target="_blank">https://github.com/LEXUGE/nixos</a>
- <a href="https://github.com/bqv/rc" target="_blank">https://github.com/bqv/rc</a>
- <a href="https://git.sr.ht/~dunklecat/nixos-config/tree" target="_blank">https://git.sr.ht/~dunklecat/nixos-config/tree</a>

Im NixOS-Wiki gibt es eine [Sammlung](https://nixos.wiki/wiki/Configuration_Collection){:target="_blank"} von Konfigurationsdateien.

## YouTube
Einige YouTuber veröffentlichen sehr guten Content zum Thema Nix. Dabei handelt es sich um sehr kleine Kanäle und die Produktionsqualität und Ausrichtung variiert. Sie sind jedoch durchweg sehr sehenswert. Persönlich bevorzuge ich eher kürzere und eher geskriptete Videos, weshalb ich Live-Streams hier nicht aufgenommen habe.

[LibrePhoenix](https://www.youtube.com/channel/UCeZyoDTk0J-UPhd7MUktexw){:target="_blank"} hat vor kurzem zwei tolle Einführungsvideos zur Konfiguration von NixOS mit Flakes und dem Home-Manager veröffentlicht. Sie umschiffen gekonnt all die frustrierenden technischen Details ohne den Eindruck dummer Toy-Beispiele zu erwecken. Er geht auch kurz darauf ein, wie man eine virtuelle Maschine nutzen kann, um seine Konfiguration vorab zu testen. Emmet hat bereits fast 800 Abonnenten, was denke ich zeigt, dass viele Neulinge seine Videos etwas abgewinnen können.

Oben habe ich bereits über die Videos von [Jeezy/LGUG2Z](https://www.youtube.com/@LGUG2Z){:target="_blank"} gesprochen. Seine Videos zeichnet sich vor allem dadurch aus, dass er nicht lange um den heißen Brei herumredet. Das Problem des Tages wird klar umgrenzt und im Anschluss daran ohne Umwege gelöst. Die fertigen Lösungen werden als GitHub-Repo verpackt. Perfekt für diejenigen, die Anwendungen zum Laufen bringen wollen, ohne sich mit dem Klein-Klein herumärgern zu müssen.

Wem es demgegenüber vor allem um die Hintergründe geht, dem wird der [Kanal](https://www.youtube.com/@jonringer117/featured){:target="_blank"} von Jon Ringer gefallen. Man bekommt augenblicklich den Eindruck, dass Jon *wirklich* weiß, wovon er redet. Er widmet sich einzelnen Spezialgebieten in zumeist kurzen Videos. Auf abstrakte Aussagen folgen fast beiläufig Kommentare dazu, was daraus in der Praxis folgt. Es liegen Monate zwischen seinen Videos, aber noch gibt es Hoffnung, dass er nochmal wiederkommt.

<!-- ## Tipps und Best Practices -->
<!-- https://discourse.nixos.org/t/tips-tricks-for-nixos-desktop/28488/2 -->
<!-- https://nixos-and-flakes.thiscute.world/best-practices/intro -->
<!-- https://nix.dev/guides/best-practices -->
