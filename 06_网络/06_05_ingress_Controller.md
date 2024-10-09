
# 1 Nginx Ingress Controller


## 1.1 Configure Ingress TLS/SSL Certificates


https://maxanuj.medium.com/how-to-configure-ingress-tls-ssl-certificates-in-kubernetes-cedafb29dd48

--verschiedene TLS Certs konfigurieren  für denselben Hostnamen und Port verschiedene 

[https://kubernetes.github.io/ingress-nginx/how-it-works/#building-the-nginx-model](https://kubernetes.github.io/ingress-nginx/how-it-works/#building-the-nginx-model "https://kubernetes.github.io/ingress-nginx/how-it-works/#building-the-nginx-model")
Wenn man mehrere Ingresses mit demselben Host-Namen (aber unterschiedlichen Pfaden) hat, gilt bei NGinx: 
- If more than one Ingress contains a TLS section for the same host, the oldest rule wins.

结论: 
man kann gar keine zwei verschiedenen TLS-Server-Certs für dieselbe https://Host:Port Kombination haben und das Cert kann auch nicht HTTP-pfadabhängig geliefert werden. 
Das Cert wird geliefert, ausgewertet und benutz,  bevor überhaupt das erste  vom HTTP Request (verschlüsselt) übermittelt werden kann.

问题1
Im Cert steht ja weder der Pfad, noch der Port drin, warum sollte man dann 2 unterschiedliche Certs haben für den gleichen Host
Und warum sollte dann für den einen Pfad das eine gelten und für den anderen das andere?
答案: Das "Problem" tritt bei uns wahrscheinlich sogar implizit auf wenn man selfSigned verwendet, weil das Helm Chart  für jeden Ingress ein neues Cert generiert. Nur ist es dann auch egal, weil diese self signed Certs eh niemand anschaut. 
Und wir haben inzwischen 3-4 Ingresses, wovon bei 2-3 derselbe Hostname vorgesehen ist, bei denen ich aber TLS jeweils beliebig konfigurieren kann. 





