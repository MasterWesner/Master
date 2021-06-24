# Aufsetzen eines lokalen Registries

Vor dem Aufetzen:

- Mit `docker info` die aktuelle Installation checken.

  Wichtig ist hierbei der Output: ` Insecure Registries: 127.0.0.0/8`. Wird wie hier nicht die IP des Hosts angegeben muss die Konfiguration des Docker Daemons geändert werden.
  
- Konfigurieren des Docker Daemons.

  Die Konfiguration erfolgt über ein `daemon.json` File welches bei Windows entweder unter `%userprofile%\.docker\daemon.json` oder unter   `C:\ProgramData\Docker\config\daemon.json`zu finden ist. Bei Linux unter `/etc/docker/daemon.json`.
  Mit `{ "insecure-registries":["host:port"] }` bwz.   `["registry_host:registry_port"]` kann dem Daemon die IP des Registries zugeordnet werden. 
  
--> Diese Konfiguration muss neben dem Server Device auch auf dem Ziel Device durchgeführt werden! (Bei beiden gleiche Daten bei `["registry_host:registry_port"]`!)




## Variante 1:
Aufsetzen des Registries mit `docker run -d -p 5001:5000 --restart=always --name registry -v F:\Daten\Studium\Master\Masterarbeit\Entwicklung\Docker\Registry:/var/lib/registry/ registry:2` --> Das Registry ist üben den Port 5001 erreichbar, startet automatisch nach jedem Neustart der Docker Engine, besizt den Namen "registry" und ist mit einem lokalen Verzeichnis verknüpft. Diese Konfiguration ermöglicht es den Inhalt des Registrys im lokalen Verzeichnis zu betrachten und somit eine erhöhte Transparent in die Vorgänge zu bringen. Dies ist hauptächlich in der Entwicklungsphase relevant. In der Produktivumgebung sollte dies mit Log-Files oder vergleichbarem ermöglicht werden.


## Variante 2:
Für eine zusätzliche Flexibilität kann das Registry mit einer `config.yml` Datei konfigurierbar gemacht werden. Dazu wird in einem lokalen Verzeichnis das `config.yml` File erstellt und dieses im `docker run` Prozess mit `-v <HostDirectory>.config.yml:/etc/docker/registry/config.yml` mit dem Configurationsverzeichnis im Container verknüpft.
In meinem Fall lautet der gesammte Befehl:

`docker run -p 1100:5000 --network host --restart=always --name registry_conf -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf:/var/lib/registry/ -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf/Config/config.yml:/etc/docker/registry/config.yml registry:2`

Wichtig ist hierbei, dass der der veröffentlichte Port im run Befehl mit dem des Configuration Files übereinstimmt!

Dieser ermöglicht außerdem alle Funktionen von Variante 1. Um über localhost auf das Registry zugreifen zu können muss jedoch zusätzlich der Netzwerkmodus ausgewählt werden.
Bei einer Änderung des Configuration Files wird das Registry mit `docker restart <RegistryName>`aktuallisiert. 
Eine Änderung von: `addr: localhost:5001` bewirkt beispielsweise eine Änderung des Ports des Registries. 

Eine vollständige übersicht über die möglichen Einstellungen ist hier zu finden: https://docs.docker.com/registry/configuration/
