# wsl-k8s
Kubernetes in WSL

Hier fasse ich meine Erfahrungen mit Kubernetes auf WSL zusammen.

## Installation

### Eine WSL Distribution für Kubernetes einrichten

Ich bin diesem Artikel gefolgt: [https://endjin.com/blog/2021/11/setting-up-multiple-wsl-distribution-instances](https://endjin.com/blog/2021/11/setting-up-multiple-wsl-distribution-instances)

1. Exportieren des Ubuntu-24.04 WSL-Images: ``wsl --export Ununtu-24.04 c:\temp\2024q4\wsl-ubuntu2404.tar``
2. Importieren des davor exportieren Images als K8s-Ubuntu-24-04: ``wsl --import K8s-Ubuntu-24.04 .\K8s-Ubuntu-24.04 c:\temp\2024q4\wsl-ubuntu2404.tar`` (ich weiß nicht, wozu der .\K8s-Ubuntu-24.04 Parameter gut ist)
3. In die neue Linux Distribution einsteigen (``wsl -d K8s-Ubuntu-24.04``) und den default user setzen: ``sudo vi /etc/wsl.conf``
4. Dort dann folgendes ergänzen (statt <username> den gewnünschten Usernamen schreiben)
```
[user]
default=<username>
```

### Docker und Minikube installieren

Hierzu bin ich diesem Artikel gefolge: [https://cuteprogramming.blog/2023/05/21/using-docker-and-kubernetes-without-docker-desktop-on-windows-11/](https://cuteprogramming.blog/2023/05/21/using-docker-and-kubernetes-without-docker-desktop-on-windows-11/) - der funktioniert auch für Windows 10.

1. In der Distribution den Package Katalog von Ubuntu aktualisieren ``sudo apt update``
2. Docker installieren: ``sudo apt install docker.io -y``
3. Den eigenen User in die Docker Usergruppe hinzufügen (damit man mit diesem User das docker Kommando ausführen kann): ``sudo usermod -aG docker $USER``
4. kubectl herunterladen: ``curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl``
5. kubectl ausführbar machen: ``chmod +x ./kubectl``
6. kubectl nach /usr/local/bin verschieben:  ``sudo mv ./kubectl /usr/local/bin/kubectl``
7. Kontrolle, ob kubectl funktioniert: ``kubectl --version`` (Es sollten Versionsinformationen über kubectl und den Server angezeigt werden) 
8. Minikube (ein kleiner lokaler K8s-Cluster) herunterladen: ``curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64``
9. Minikube installieren: ``sudo install minikube-linux-amd64 /usr/local/bin/minikube``

### Nacharbeiten

Hier werden sinnvolle Nacharbeiten nach der Installation beschrieben

#### docker und kubectl auch aus der lokalen Powershell ausführen

Man kann aus der lokalen Powershell in Windows auch Befehle in den jeweiligen WSL Distributionen absetzen. Dazu muss man (für die Default Distribution) nur ``wsl`` davor schreiben (z.b. ``wsl node /home/pirklb/index.js```). Will man einen Befehl in einer anderen WSL Distribution absetzen (in diesem Kontext der K8s-Ubuntu-24.04 Distribution) muss man ``wsl -d K8s-Ubuntu-24.04`` davor schreiben (z.b. ``wsl -d K8s-Ubuntu-24.04 docker--version``).

Damit man das nicht immer davor schreiben muss, kann man sich entsprechende Aliase in seinem Powershell-Profil einrichten. Wo das liegt kann man mit echo $PROFILE herausfinden (ACHTUNG: Es kann sein, dass man noch gar kein Powershell-Profile hat, dann hat die Variable zwar einen Wert, aber das referenzierte Skript existiert nicht).

Ich verwende dieses Powershell-Profileskript:

```powershell
$WslK8sDistro='K8s-Ubuntu-24.04'
function Start-WslDocker {
  wsl -d $WslK8sDistro docker $args
}

function Start-WslDockerCompose {
  wsl -d $WslK8sDistro docker-compose $args
}

function Start-WslKubectl {
  wsl -d $WslK8sDistro kubectl $args
}

Set-Alias -Name d -Value Start-WslDocker
Set-Alias -Name dc -Value Start-WslDockerCompose
Set-Alias -Name k -Value Start-WskKubectl
```

Wenn das Skript remote liegt (z. b. am eigenen Homeshare) und die Powershell Execution Policy zumindest auf RemoteSigned steht, muss man dieses Skript zertifizieren. Dazu muss man über das Zertifikatsmanagement am eigenen Client bei den eigenen Zertifikaten unter Aufgaben ein neues Zertifikat anfordern. Dann mit Weiter, Weiter bis zur Auswahl der möglichen Zertifikatstypen gehen und das **WALTER GROUP Code Signing** auswählen und registrieren.

In der Powershell kann man dann mit ``get-childitem cert:\currentuser\my -codesigningcert`` seine Zertifikate prüfen (und sollte eben zumindest das eine Codesignatur-Zertifikat sehen).

Und mit diesem Skript kann man das Skript mit seinem eigenen Codesigning Zertifikat signieren:

```powershell
$script=$PROFILE
if ($script -and (Test-Path -Path $script)) {
  $codingCert=(get-childitem cert:\currentuser\my -codesigningcert)[0]
  if ($codingCert) {
    Set-AuthenticodeSignature -FilePath $script -Certificate $codingCert
  }

} else { 
  write-host "Skript ${script} nicht gefunden" 
}
```

Im Moment habe ich noch das Problem, dass beim Laden des Skripts dann trotzdem die Fehlermeldung: **.: AuthorizationManager check failed.** Das sieht so aus, als ob gar nicht das Skript selbst das Problem hat, sondern der "Kontext" .
