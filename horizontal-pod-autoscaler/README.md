# Horizontal Pod Autoscaler

Molnet låter oss skala horisontellt istället för bara vertikalt: det är mer skalbart att öka antalet instanser av en programvara än att försöka skaffa en större (och större, och större...) server för att öka mängden resurser.
Inom ett Kubernetes-kluster kan vi göra detta med hjälp av Horizontal Pod Autoscaler (HPA).

Vi definierar en helt vanlig Deployment eller ett StatefulSet och sedan refererar vi till det när vi skapar en HPA.
Autoskalarens jobb kommer då vara att ändra antalet `replicas` efter de mätvärden vi bestämmer.

Det finns två versioner av HPA i Kubernetes och den gamla, `v1`, är fortfarande vanlig.
Men den nya, `v2`, erbjuder möjligheten att skala på mer än ett mätvärde **och** möjligheten att skala på fler mätvärden än bara klassiska CPU-lasten.

Exemplet skalar en Deployment ([deployment.yaml](deployment.yaml)) som känns igen från [../pod-topology-spread-constraints](../pod-topology-spread-constraints), med hjälp av en HPA definierad i [horizontal-pod-autoscaler.yaml](horizontal-pod-autoscaler.yaml), som är så kort att den kan återges i sin helhet här:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: test-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test-configuration
  minReplicas: 2
  maxReplicas: 5
  metrics:                       # Vi kan numera skala på en lista av mätvärden!
  - type: Resource               # Här är en "resource" för containers i en Pod
    resource:                    #
      name: cpu                  # Resursen i fråga är CPU-
      target:                    #
        type: Utilization        # -nyttjande, där vi vill skala så att
        averageUtilization: 67   # genomsnittet hamnar på 67%
```

Notera att om vi först skapar vår Deployment genom `kubectl apply -f deployment.yaml` och väntar ett tag, så ser vi att den skapar en ensam instans, precis som antalet `replicas` säger i filen.
Så fort vi skapar vår HPA via `kubectl apply -f horizontal-pod-autoscaler.yaml`, så kommer den att skala upp, för `minReplicas` är ju satt till 2.
