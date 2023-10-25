# ConfigMap vs. Secrets

Både ConfigMaps och Secrets låter en jobba med nyckel-värde-par, som en väldigt enkel databas.
Att de är två olika sorters resurser är primärt för att det därmed går att sätta olika rättigheter i det rollbaserade accesskontrollsystemet (RBAC) i Kubernetes.
Data som finns i dem kan refereras till både som miljövariabler till en container i en Pod eller som en del av filsystemet som en Pod startar med.

Exempelkod finns för såväl [configmap.yaml](configmap.yaml) som för [secret.yaml](secret.yaml). 
Där syns att värdena i en Secret kommer vara base64-kodade.
**Hade det varit en äkta hemlighet hade det verkligen inte varit lämpligt att lägga in den i Git så som vi gjort här!**
Använd istället [SOPS](https://github.com/getsops/sops) eller [Sealed Secrets](https://elastisys.io/compliantkubernetes/user-guide/self-managed-services/sealedsecrets/) för att spara krypterade hemligheter i Git och senare använda i Elastisys Compliant Kubernetes.

Hur dessa värden sedan används står i [pod.yaml](pod.yaml), där de intressantaste bitarna är här:

```yaml
        env:                               # Miljövariabler för programmet kan komma från 
        - name: MAX_THREADS                # ConfigMaps eller från Secrets (eller anges direkt).
          valueFrom:                       # Här kommer rätt värde från en ConfigMap,
            configMapKeyRef:               # som refereras till med
              name: configuration-values   # namnet på ConfigMap:en som sådan och
              key: threads                 # själva nyckeln i den ConfigMap:en
        
        volumeMounts:                      # Vi kan också läsa in ConfigMaps och Secrets helt som
        - name: credentials                # filsystem -- här beskriver vi vilket namnet (internt för denna spec, måste matcha det på rad 52)
          mountPath: "/etc/credentials"    # är och var den skall monteras in, som under /etc/credentials
          readOnly: true                   # Det är bra praxis att montera med read-only, för skrivningar kommer inte propageras

# ...

      volumes:                             # Här definierar vi en lista av "volumes", som vi kan referera till inuti denna spec.
      - name: credentials                  # Vi kallar en för "credentials" och fyller den med innehållet från
        secret:                            # den Secret som heter sensitive-values
          secretName: sensitive-values
```

Se dock hela filen för det fullständiga exemplet.