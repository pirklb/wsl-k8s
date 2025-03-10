# kubectl

## context

kubectl dient zur Kommunikation mit Kubernetes. Es verwendet einen context. Dieser legt den Cluster und den User fest. Ein Context ist dabei die Kombination **cluster** und **user**. Man kann mehrere contexte in einem config File hinterlegen. 

## Beispiel config (für die Definition der Cluster, User und Contexte - im Beispiel sind zwei Cluster und zwei User definiert, jeweils ein User gehört zu einem Cluster, die Contexte heißen `ctx-clu1-usr1` und `ctx-clu2-usr2`):

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0...Qo=
    server: https://cr...com:443
  name: cluster1
- cluster:
    certificate-authority-data: LS0...LQo=
    server: https://pr...com:443
  name: cluster2
contexts:
- context:
    cluster: cluster1
    user: user1
  name: ctx-clu1-usr1
- context:
    cluster: cluster2
    namespace: mynamespace
    user: user2
  name: ctx-clu2-usr2
current-context: ctx-clu1-usr1
kind: Config
preferences: {}
users:
- name: user1
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://...com
      - --oidc-client-id=33...com
      - --oidc-client-secret=<geheim>
      - --oidc-extra-scope=openid profile email
      command: kubectl
- name: user2
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
      - oidc-login
      - get-token
      - --oidc-issuer-url=https://...com
      - --oidc-client-id=33...com
      - --oidc-client-secret=<geheim>
      - --oidc-extra-scope=openid profile email
      command: kubectl
      env: null
```

### Anzeigen der verfügbaren contexte

`kubectl config get-contexts`

### Umschalten zwischen verschiedenen Contexten

`kubectl config use-context <contextname>`

Man kann auch unbenannte Kontexte haben, wie man den dann auswählt, weiß ich nicht. Der Name des contexts wird bei get-contexts angezeigt (er muss explizit im config File eingetragen sein).
