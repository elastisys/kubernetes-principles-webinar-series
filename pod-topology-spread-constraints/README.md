# Pod Topology Spread Constraints

En av anledningarna till att vi kan vilja skala upp antalet Pods är feltolerans: vi vill ha flera instanser av samma programvara körande, så att systemet som helhet kan hantera om det är fel på en eller flera av instanserna.
Fel i det här sammanhanget kan exempelvis vara att själva virtuella maskinen, arbetsnoden i Kubernetes, har fått ett fel.
Eller att den fysiska datorn där virtuella maskinen körs har fått ett.
Eller, i värsta fall, ett helt datacenter!

[Pod Topology Spread Constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/) används för att definiera gränser för hur "skev" en fördelning av Pods får vara.
Meningen är att kunna informera Kubernetes om att det inte räcker att köra 3 instanser av mjukvaran (3 Pods i en Deployment), utan att dessa **också** måste vara *utspridda* på olika virtuella maskiner (exempelvis).

I kodexemplet visas just en sådan Pod Topology Spread Constraint. 
Se först koden som saknar den typen av information ([pod-without-constraint.yaml](pod-without-constraint.yaml)) och sedan den med informationen ([pod-with-constraints.yaml](pod-with-constraints.yaml)).
Den viktiga biten belyses här nedan:

```diff
+       topologySpreadConstraints:
+       - topologyKey: kubernetes.io/hostname # På nivån av worker nodes...
+         maxSkew: 1                          # ...tillåt maximal skevhet 1...
+         whenUnsatisfiable: DoNotSchedule    # ...och förbjud schemaläggning som skulle bryta mot regeln.
+         labelSelector:                      # Vanlig label selector som pekar ut Pods ur just denna
+           matchLabels:                      # Deployment, alltså, med "app" satt till "test-constraints",
+             app: test-constraints           # vilket ju sätts på rader ovan.
```


Vår Deployment begär 5 Pods.
Men om vi testar att lista alla Pods nu, så ser vi att endast 4 av de 5 vi begärde faktiskt startats i den miljö där vi kör:

```sh
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS   AGE
test-constraints-cbb468f88-8mrgb   1/1     Running   0          10s
test-constraints-cbb468f88-9c29j   0/1     Pending   0          11s
test-constraints-cbb468f88-9c4gx   1/1     Running   0          11s
test-constraints-cbb468f88-f9pqz   1/1     Running   0          11s
test-constraints-cbb468f88-khmg6   1/1     Running   0          11s
```

För att förstå anledningen, så kan vi räkna antalet arbetsnoder i Kubernetes-klustret:

```sh
$ kubectl get nodes | grep worker | wc -l
4
```

Med våra regler som säger att vi maximalt tillåter skevhet 1 och förbjuder schemaläggning av Pods ifall regeln skulle brytas, förstår vi att 4 arbetsnoder självklart bara kan köra 4 Pods.
När vi kör utan Pod Topology Spread Constraints, kan vi också se att alla 5 startar, men att de då kan hamna med en helt annan skevhet.