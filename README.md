# Aufsetzen eines lokalen Registries

Vor dem Aufetzen:

- Mit `docker info` die aktuelle Installation checken.

  Wichtig ist hierbei der Output: ` Insecure Registries: 127.0.0.0/8`. Wird wie hier nicht die IP des Hosts angegeben muss die Konfiguration des Docker Daemons geändert werden.
  
- Konfigurieren des Docker Daemons.

  Die Konfiguration erfolgt über ein `daemon.json` File welches bei Windows entweder unter `%userprofile%\.docker\daemon.json` oder unter   `C:\ProgramData\Docker\config\daemon.json`zu finden ist. Bei Linux unter `/etc/docker/daemon.json`.
  Mit `{ "insecure-registries":["host_IP:port"] }` bwz.   `["registry_host_IP:registry_port"]` kann dem Daemon die IP des Registries zugeordnet werden. --> siehe beispielhaft das `daemon.json` File.
  
  Unter Windows kann die Konfiguration außerdem auch in der Docker Desktop Benutzeroberfläche durchgeführt werden. Unter "Einstellungen" / "Docker Engine" kann die `daemon.json` Datei direkt bearbeitet werden.
  
**--> Diese Konfiguration muss neben dem Server Device auch auf dem Ziel Device durchgeführt werden! (Bei allen Devices die auf das Registry zugreifen sollen müssen die Daten bei `["registry_host:registry_port"]` identisch sein!)**


Fix found here: https://stackoverflow.com/questions/49674004/docker-repository-server-gave-http-response-to-https-client/63227959#63227959


## Aktuelle Variante:
### Registry mit UI

1. Download der `config.yml` und der `compose.yml` Datei
2. Anpassen des Speicherverzeichnis der `config.yml` Datei und des gewünschten Daten-Verzeichnis in der `compose.yml` Datei (bei `volumes`)
4. Auswahl des Verzeichnis im CLI mit `cd`
5. Starten des Registries mit `docker compose up` <br>
        alternativ: `docker-compose up`

Zugang zum Registry erfolgt über `:5000` und zum UI über `localhost:80`
Gegbenenfalls muss die URL des Registries im UI über das Menue hinzugefügt werden (`http://localhost:5000`)



_______________________






## Alte Varianten:

## Variante 1: 
Aufsetzen des Registries mit `docker run -d -p 5000:5000 --restart=always --name registry -v F:\Daten\Studium\Master\Masterarbeit\Entwicklung\Docker\Registry:/var/lib/registry/ registry:2` --> Das Registry ist üben den Port 5001 erreichbar, startet automatisch nach jedem Neustart der Docker Engine, besizt den Namen "registry" und ist mit einem lokalen Verzeichnis verknüpft. Diese Konfiguration ermöglicht es den Inhalt des Registrys im lokalen Verzeichnis zu betrachten und somit eine erhöhte Transparent in die Vorgänge zu bringen. Dies ist hauptächlich in der Entwicklungsphase relevant. In der Produktivumgebung sollte dies mit Log-Files oder vergleichbarem ermöglicht werden.


## Variante 2: -konfigurierbar
Für eine zusätzliche Flexibilität kann das Registry mit einer `config.yml` Datei konfigurierbar gemacht werden. Dazu wird in einem lokalen Verzeichnis das `config.yml` File erstellt und dieses im `docker run` Prozess mit `-v <HostDirectory>.config.yml:/etc/docker/registry/config.yml` mit dem Configurationsverzeichnis im Container verknüpft.
In meinem Fall lautet der gesammte Befehl:

`docker run -p 1100:5000 --network host --restart=always --name registry_conf -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf:/var/lib/registry/ -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf/Config/config.yml:/etc/docker/registry/config.yml registry:2`

_________________________________________________
**Aktueller Stand 24.06.21:**

`docker run -a=STDOUT -a=STDERR -p 5000:5000 --network default --restart=always --name registry_conf -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf:/var/lib/registry/ -v F:/Daten/Studium/Master/Masterarbeit/Entwicklung/Docker/Registry_Conf/Config/config.yml:/etc/docker/registry/config.yml registry:2`

Mit dieser Konfiguration ist der Output von `docker inspect`, bis auf speicherort- und bezeichnugsspezifische Daten bei beiden Varianten identisch. Auf dieser Basis kann in Zukunft eine Optimierung erfolgen!

Das gespeicherte `config.yml` File ist lediglich mit den grundlegend notwendigen Konfigureationen ausgeführt.
_________________________________________________




Wichtig ist hierbei generell, dass der der veröffentlichte Port im run Befehl mit dem des Configuration Files übereinstimmt!

Bei einer Änderung des `config.yml`Files wird das Registry mit `docker restart <RegistryName>`aktualisiert. 
Eine Änderung von: `addr: localhost:5001` bewirkt beispielsweise eine Änderung des zugewiesenen Ports des Registries. 

Eine vollständige übersicht über die möglichen Einstellungen ist hier zu finden: https://docs.docker.com/registry/configuration/


# Push und Pull

## Push
- Vom Registry Host Device:  `docker push localhost:<registry_port>/<Image_name>:<tag>`   --> beachte: localhost! (geht auch mit Host IP, jedoch nicht, wenn direkt aus dem Build Prozesse heraus das Image ins Registry gepushed werden soll!)
- Von einem anderen Device im lokalen Netzwerk: `docker push <registry_host_IP>:<registry_port>/<Image_name>:<tag>`

## Pull
- Vom Registry Host Device: `docker pull localhost:<registry_port>/<Image_name>:<tag>`  --> Funktioniert auch bei Images die von einem anderen Device aus mit `<registry_host_IP>:<registry_port>` gepushed wurden!
- Von einem anderen Device im lokalen Netzwerk: `docker pull <registry_host_IP>:<registry_port>/<Image_name>:<tag>`
