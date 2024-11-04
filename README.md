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

  
