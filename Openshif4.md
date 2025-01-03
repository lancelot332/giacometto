# OpenShift4
OpenShift 4 è una piattaforma di orchestrazione e gestione di container basata su **Kubernetes**. 
## Risorse:
### Deployment:
Gestisce il ciclo di vita di applicazioni containerizzate, controllando il numero desiderato di Pod in esecuzione e applicando aggiornamenti senza downtime.  
Esempio:
```yaml
apiVersion: apps/v1               # Specifica la versione dell'API Kubernetes utilizzata per il Deployment.
kind: Deployment                  # Definisce il tipo di risorsa come Deployment.
metadata:                         # Sezione che contiene i metadati del Deployment.
  name: my-deployment             # Nome assegnato al Deployment.
  namespace: my-namespace         # Namespace in cui viene creato il Deployment.
spec:                             # Specifica la configurazione desiderata del Deployment.
  replicas: 3                     # Numero di pod che il Deployment deve mantenere in esecuzione.
  selector:                       # Criterio per selezionare i pod gestiti da questo Deployment.
    matchLabels:                  # Etichette che i pod devono avere per essere selezionati.
      app: my-app                 # Etichetta che deve essere presente sui pod per essere selezionati.
  template:                       # Modello che descrive i pod creati dal Deployment.
    metadata:                     # Metadati associati ai pod.
      labels:                     # Etichette assegnate ai pod per identificazione.
        app: my-app               # Etichetta che assegna il valore "my-app" al pod.
    spec:                         # Specifica la configurazione dei pod.
      containers:                 # Definisce i container inclusi nel pod.
      - name: my-container        # Nome assegnato al container.
        image: nginx:latest       # Immagine Docker utilizzata per il container (in questo caso, l'ultima versione di Nginx).
        ports:                    # Porta esposta dal container.
        - containerPort: 80       # Porta interna del container che viene aperta per il traffico (porta HTTP predefinita).
```
### Deploymentconfig
Risorsa specifica di OpenShift che aggiunge controlli avanzati agli aggiornamenti, come hook personalizzati e integrazione nativa con Source-to-Image (S2I).
Differenze tra Deployment e DeploymentConfig:
Il DeploymentConfig è specifica solo per openshift e può utilizzare dei trigger avanzati e questo migliora l' integrazione CI/CD.
Esempio manifest:
```yaml
apiVersion: apps.openshift.io/v1        # Specifica la versione dell'API di OpenShift per il DeploymentConfig.
kind: DeploymentConfig                  # Definisce il tipo di risorsa come DeploymentConfig.
metadata:                               # Contiene i metadati della risorsa.
  name: my-dc                           # Nome assegnato al DeploymentConfig.
  namespace: my-namespace               # Namespace in cui il DeploymentConfig verrà creato.
spec:                                   # Specifica la configurazione del DeploymentConfig.
  replicas: 3                           # Numero di pod che devono essere mantenuti attivi.
  selector:                             # Etichette utilizzate per selezionare i pod gestiti dal DeploymentConfig.
    app: my-app                         # Etichetta che deve essere presente sui pod per essere selezionati.
  strategy:                             # Specifica la strategia di aggiornamento.
    type: Rolling                       # Strategia di aggiornamento che sostituisce gradualmente i vecchi pod con quelli nuovi.
  triggers:                             # Sezione per configurare i trigger del DeploymentConfig.
  - type: ImageChange                   # Trigger basato su modifiche all'immagine.
    imageChangeParams:                  # Parametri per il trigger ImageChange.
      automatic: true                   # Attiva automaticamente un deployment quando l'immagine cambia.
      containerNames:                   # Specifica i container a cui si applica il trigger.
      - my-container                    # Nome del container che verrà aggiornato automaticamente.
      from:                             # Definisce la sorgente dell'immagine monitorata.
        kind: ImageStreamTag            # Tipo di risorsa monitorata (un tag di un ImageStream).
        name: nginx:latest              # Nome dell'immagine e tag (nginx:latest).
        namespace: openshift            # Namespace in cui si trova l'ImageStream.
  - type: ConfigChange                  # Trigger basato su modifiche alla configurazione del DeploymentConfig.
  template:                             # Modello per definire i pod creati dal DeploymentConfig.
    metadata:                           # Metadati associati ai pod.
      labels:                           # Etichette assegnate ai pod per identificazione.
        app: my-app                     # Etichetta che assegna il valore "my-app" ai pod.
    spec:                               # Specifica la configurazione dei pod.
      containers:                       # Elenco dei container inclusi nel pod.
      - name: my-container              # Nome del container.
        image: nginx:latest             # Immagine Docker utilizzata per il container (in questo caso, l'ultima versione di Nginx).
        ports:                          # Elenco delle porte esposte dai container.
        - containerPort: 80             # Porta interna del container aperta per il traffico (porta HTTP predefinita).
```
### StatefulSet:
Gestisce applicazioni stateful con identità stabili e storage persistente.
Viene usata sopratutto per database i quali richiedono una indentità stabile e uno storage persistente.
esempio manifest:
```yaml
apiVersion: apps/v1                      # Specifica la versione dell'API per il StatefulSet.
kind: StatefulSet                        # Definisce il tipo di risorsa come StatefulSet, utile per applicazioni con stato.
metadata:                                # Contiene i metadati per la risorsa.
  name: my-statefulset                   # Nome assegnato al StatefulSet.
spec:                                    # Specifica la configurazione del StatefulSet.
  serviceName: "my-service"              # Nome del servizio headless associato ai pod del StatefulSet.
  replicas: 3                            # Numero di repliche (pod) che devono essere mantenute attive.
  selector:                              # Specifica come selezionare i pod gestiti dal StatefulSet.
    matchLabels:                         # Etichette utilizzate per identificare i pod.
      app: my-app                        # Etichetta che deve essere presente sui pod.
  template:                              # Modello per definire i pod creati dal StatefulSet.
    metadata:                            # Metadati associati ai pod.
      labels:                            # Etichette assegnate ai pod per identificazione.
        app: my-app                      # Etichetta che assegna il valore "my-app" ai pod.
    spec:                                # Specifica la configurazione dei pod.
      containers:                        # Elenco dei container inclusi nel pod.
      - name: my-container               # Nome del container all'interno del pod.
        image: mysql:latest              # Immagine Docker utilizzata per il container (in questo caso, l'ultima versione di MySQL).
        ports:                           # Elenco delle porte esposte dai container.
        - containerPort: 3306            # Porta interna del container aperta per il traffico (porta predefinita di MySQL).
  volumeClaimTemplates:                  # Modelli per PersistentVolumeClaim associati a ciascun pod.
  - metadata:                            # Metadati per il volume persistente.
      name: data                         # Nome assegnato al volume (usato come riferimento nei pod).
    spec:                                # Specifica la configurazione del volume persistente.
      accessModes: ["ReadWriteOnce"]     # Modalità di accesso al volume (lettura/scrittura da un solo nodo).
      resources:                         # Richiesta di risorse per il volume.
        requests:                        # Specifica le risorse richieste.
          storage: 10Gi                  # Dimensione richiesta per il volume (10 GiB).
```
### Routes
Risorsa OpenShift per esporre servizi interni all'esterno del cluster.  
Differenze con ingress:  
Routes come il DeploymentConfig è una risorsa specifica per openshift il controllo del traffico viene gestito da un controller ingress per l' ingress per le routes viene gestito dal router di openshift.  funzionalita di routing: le routes  Routing HTTP/HTTPS con funzionalità avanzate, come wildcard e gestione del TLS.  
Esempio manifest:
```yaml
apiVersion: route.openshift.io/v1   # Definisce la versione API della risorsa. 'route.openshift.io/v1' indica una risorsa Route specifica di OpenShift.
kind: Route                         # Indica che si sta creando una risorsa di tipo 'Route', utilizzata per esporre servizi all'esterno del cluster OpenShift.
metadata:                            # Sezione che definisce i metadati della risorsa.
  name: my-route                     # Il nome della Route. Questo nome viene utilizzato per identificare la Route all'interno di OpenShift.
spec:                                # Sezione che definisce la configurazione specifica della Route.
  host: my-app.example.com           # L'hostname tramite cui il traffico esterno raggiungerà la Route. In questo caso, il traffico HTTP sarà indirizzato a "my-app.example.com".
  to:                                 # Sezione che definisce il servizio a cui il traffico verrà indirizzato.
    kind: Service                     # Il tipo di risorsa a cui il traffico è indirizzato. Qui è un servizio (Service) di Kubernetes.
    name: my-service                  # Il nome del servizio a cui il traffico verrà inviato. Il traffico in ingresso alla Route verrà diretto al servizio "my-service".
  tls:                                # Sezione per configurare la gestione del traffico TLS (Transport Layer Security).
    termination: edge                 # Definisce il tipo di terminazione TLS. "edge" significa che la crittografia TLS è terminata al livello del router OpenShift, prima che il traffico non crittografato venga inoltrato al servizio.
```
### Ingress
Risorsa Kubernetes standard per gestire l'accesso HTTP/S.
Esempio manifest:
```yaml
apiVersion: networking.k8s.io/v1  # Specifica la versione dell'API per la risorsa, in questo caso una risorsa Ingress di Kubernetes. 'networking.k8s.io/v1' è la versione stabile.
kind: Ingress                      # Definisce che il tipo di risorsa è 'Ingress', che gestisce il traffico in ingresso (HTTP/HTTPS) verso i servizi all'interno del cluster.
metadata:                           # Sezione che definisce i metadati della risorsa.
  name: my-ingress                  # Il nome dell'Ingress, utilizzato per identificare la risorsa all'interno del cluster. In questo caso è chiamato 'my-ingress'.
spec:                               # Sezione che definisce la configurazione specifica della risorsa Ingress.
  rules:                             # Sezione che definisce le regole di routing per l'Ingress. Ogni regola indirizza il traffico in ingresso verso un backend.
  - host: my-app.example.com        # Il nome dell'host per il quale la regola di routing è applicata. In questo caso, il traffico per 'my-app.example.com' verrà instradato secondo le regole definite sotto.
    http:                            # Specifica che il traffico gestito dalla regola è di tipo HTTP.
      paths:                          # Sezione che definisce i percorsi (URL) per cui la regola si applica.
      - path: /                       # Il percorso relativo che deve essere corrispondente per la regola. In questo caso, il percorso '/' (root) della URL.
        pathType: Prefix              # Tipo di percorso. 'Prefix' significa che tutti i percorsi che iniziano con '/' (root) saranno abbinati a questa regola.
        backend:                      # Definisce il backend (il servizio a cui indirizzare il traffico).
          service:                    # Tipo di risorsa a cui indirizzare il traffico, in questo caso un servizio (Service) Kubernetes.
            name: my-service          # Il nome del servizio di destinazione. Qui il traffico verrà indirizzato al servizio denominato 'my-service'.
            port:                      # La porta del servizio a cui il traffico verrà inviato.
              number: 80              # La porta del servizio a cui il traffico verrà inviato, in questo caso la porta 80 (tipica per il traffico HTTP).
```
### PresistentVolume (PV)
Rappresenta uno storage fisico preconfigurato o dinamicamente allocato.
```yaml
apiVersion: v1                      # Specifica la versione dell'API per la risorsa. 'v1' è la versione stabile per la risorsa PersistentVolume in Kubernetes.
kind: PersistentVolume               # Definisce che il tipo di risorsa è 'PersistentVolume', una risorsa di storage che può essere utilizzata dai pod nel cluster.
metadata:                            # Sezione che contiene i metadati della risorsa.
  name: my-pv                        # Il nome del PersistentVolume. Questo nome è utilizzato per identificare la risorsa all'interno del cluster.
spec:                                # Sezione che contiene la configurazione specifica del PersistentVolume.
  capacity:                          # Sezione che definisce la capacità di storage del volume.
    storage: 10Gi                    # La quantità di storage disponibile per il PersistentVolume, in questo caso 10 GiB.
  accessModes:                       # Sezione che definisce i modi di accesso consentiti per il volume.
  - ReadWriteOnce                    # Tipo di accesso consentito: 'ReadWriteOnce' significa che il volume può essere montato in lettura e scrittura da un singolo nodo alla volta.
  persistentVolumeReclaimPolicy:     # Sezione che definisce la politica di reclamazione del volume una volta che non è più utilizzato.
    Retain                            # La politica 'Retain' significa che il volume non verrà eliminato quando il PersistentVolumeClaim che lo utilizza viene rimosso. Il volume deve essere gestito manualmente.
  hostPath:                           # Sezione che definisce il tipo di storage backend. In questo caso, si tratta di un volume di tipo hostPath, che utilizza una directory locale sul nodo.
    path: /mnt/data                   # Il percorso della directory sul nodo che verrà utilizzato come PersistentVolume. Qui è '/mnt/data', una directory locale del nodo.
``` 
### PersistentVolumeClaims (PVC)
Richiesta di storage da parte di un'applicazione.
```yaml
apiVersion: v1                      # Specifica la versione dell'API per la risorsa. 'v1' è la versione stabile per la risorsa PersistentVolumeClaim in Kubernetes.
kind: PersistentVolumeClaim          # Indica che il tipo di risorsa è 'PersistentVolumeClaim' (PVC), che rappresenta una richiesta di storage da utilizzare nei pod.
metadata:                            # Sezione che contiene i metadati della risorsa.
  name: my-pvc                       # Il nome del PersistentVolumeClaim, utilizzato per identificare la risorsa nel cluster. Qui è chiamato 'my-pvc'.
spec:                                # Sezione che contiene la configurazione specifica del PersistentVolumeClaim.
  accessModes:                       # Sezione che definisce i modi di accesso richiesti per il volume.
  - ReadWriteOnce                    # Tipo di accesso richiesto per il volume. 'ReadWriteOnce' significa che il volume deve essere montato in modalità lettura e scrittura da un singolo nodo alla volta.
  resources:                         # Sezione che definisce le risorse richieste per il PVC.
    requests:                         # Sezione che definisce la quantità di storage richiesta per il volume.
      storage: 5Gi                    # La quantità di storage richiesta per il PVC. In questo caso, si richiedono 5 GiB di spazio di archiviazione.
```
### Role
Gestisce permessi granulari in un namespace.
```yaml
apiVersion: rbac.authorization.k8s.io/v1  # Specifica la versione dell'API per la risorsa. In questo caso, è 'v1' della API RBAC (Role-Based Access Control) per la gestione dei permessi.
kind: Role                                # Indica che il tipo di risorsa è 'Role', che definisce i permessi per gli utenti all'interno di un namespace specifico.
metadata:                                  # Sezione che contiene i metadati per la risorsa.
  namespace: my-namespace                  # Definisce il namespace in cui il ruolo sarà valido. In questo caso, il ruolo è applicato a 'my-namespace'.
  name: pod-reader                         # Il nome del ruolo. Qui è chiamato 'pod-reader' e rappresenta un ruolo che consente l'accesso in lettura ai pod.
rules:                                    # Sezione che definisce le regole di accesso specifiche per il ruolo.
- apiGroups: [""]                          # Specifica i gruppi API a cui si applicano le regole. Un array vuoto (""), significa che si applicano alle risorse base, come 'pods'.
  resources: ["pods"]                      # Indica le risorse per le quali sono definite le regole. In questo caso, la risorsa è 'pods', cioè i pod nel cluster.
  verbs: ["get", "watch", "list"]          # Definisce le azioni (verbi) consentite sulla risorsa. 'get' consente di visualizzare i dettagli del pod, 'watch' di monitorare i cambiamenti, e 'list' di elencare i pod.
```
### ClusterRole
Gestisce permessi a livello di cluster.
### ServiceAccount
Identità per applicazioni che interagiscono con l'API del cluster.
```yaml
# Definisce un ServiceAccount con il nome 'my-service-account' nel namespace 'my-namespace'.
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account       # Nome del ServiceAccount
  namespace: my-namespace         # Namespace in cui viene creato il ServiceAccount

---
# Definisce un RoleBinding che lega un ruolo a un ServiceAccount all'interno di un namespace specifico.
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding       # Nome del RoleBinding
  namespace: my-namespace        # Namespace in cui viene applicato il RoleBinding (stesso del ServiceAccount)
subjects:
- kind: ServiceAccount           # Tipo di soggetto: un ServiceAccount
  name: my-service-account       # Nome del ServiceAccount a cui viene assegnato il ruolo
  namespace: my-namespace        # Namespace in cui è presente il ServiceAccount
roleRef:
  kind: Role                     # Tipo di ruolo: un Role (limitato a un namespace)
  name: pod-reader               # Nome del ruolo che viene assegnato al ServiceAccount
  apiGroup: rbac.authorization.k8s.io  # Gruppo API a cui appartiene il ruolo (per il controllo degli accessi)

---
# Definisce un ClusterRoleBinding che lega un ruolo a un ServiceAccount a livello di cluster.
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-view-binding     # Nome del ClusterRoleBinding
subjects:
- kind: ServiceAccount           # Tipo di soggetto: un ServiceAccount
  name: my-service-account       # Nome del ServiceAccount a cui viene assegnato il ruolo
  namespace: my-namespace        # Namespace in cui è presente il ServiceAccount
roleRef:
  kind: ClusterRole             # Tipo di ruolo: un ClusterRole (per l'intero cluster)
  name: view                    # Nome del ruolo che viene assegnato al ServiceAccount
  apiGroup: rbac.authorization.k8s.io  # Gruppo API a cui appartiene il ruolo (per il controllo degli accessi a livello di cluster)
```
### Secret
Archivia dati sensibili.
```yaml
# Definisce un Secret con un campo di dati in base64
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  password: cGFzc3dvcmQ=  # Base64("password") - La password codificata in base64
```
### ConfigMap
Archivia configurazioni non sensibili.
```yaml
apiVersion: v1  # Indica la versione dell'API di Kubernetes utilizzata per creare la risorsa. In questo caso è la versione v1 per ConfigMap.
kind: ConfigMap  # Definisce il tipo di risorsa, che in questo caso è un ConfigMap. I ConfigMap vengono usati per memorizzare dati di configurazione.
metadata:  # Contiene i metadati relativi alla risorsa, come il nome e le etichette.
  name: my-config  # Nome del ConfigMap. Questo sarà usato per fare riferimento a questo ConfigMap all'interno di Kubernetes.
data:  # La sezione che contiene i dati effettivi del ConfigMap.
  config.json: |  # La chiave 'config.json' è il nome del file o della configurazione che viene archiviato nel ConfigMap.
    {  # Inizia il contenuto del file JSON, che sarà un dato di configurazione. Il carattere '|' permette di preservare il formato multilinea.
      "key": "value"  # Dato di configurazione in formato JSON. Una chiave 'key' con il valore 'value'.
    }  # Fine del contenuto JSON.

```
### DaemonSet
risorsa che assicura che una copia di un pod sia eseguita su ogni nodo
```yaml
apiVersion: apps/v1  # Versione dell'API di Kubernetes/OpenShift utilizzata per creare il DaemonSet
kind: DaemonSet  # Tipo di risorsa, in questo caso un DaemonSet
metadata:
  name: nginx-daemonset  # Nome del DaemonSet
  namespace: my-namespace  # Namespace in cui creare il DaemonSet
spec:
  selector:
    matchLabels:
      app: nginx-daemon  # Etichetta per il selettore dei pod
  template:
    metadata:
      labels:
        app: nginx-daemon  # Etichetta da applicare ai pod creati dal DaemonSet
    spec:
      containers:
      - name: nginx  # Nome del container
        image: nginx:latest  # Immagine del container (nginx)
        ports:
        - containerPort: 80  # Porta esposta dal container
      securityContext:
        runAsUser: 1000  # Utente con cui il container viene eseguito (importante per rispettare le SCC in OpenShift)
        runAsGroup: 1000  # Gruppo con cui il container viene eseguito (importante per rispettare le SCC in OpenShift)
      tolerations:  # Permette ai pod di essere schedulati su nodi con taints specifici
      - key: "node-role.kubernetes.io/master"  # Chiave del taint per i nodi master
        operator: "Exists"  # Operatore per verificare se esiste il taint
        effect: "NoSchedule"  # Effetto del taint: il pod non sarà schedulato su nodi master a meno che non abbia una toleration
  updateStrategy:
    type: RollingUpdate  # Strategia di aggiornamento dei pod (rolling update)
  terminationGracePeriodSeconds: 30  # Tempo di grace (in secondi) per la terminazione dei pod
```
