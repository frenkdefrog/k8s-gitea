## Gitea
A telepítéshez az alábbi "összetevők" szükségesek:
* Gitea helmchart: https://gitea.com/gitea/helm-chart
* gitea-values.yml fájl
* gitea-virtual-service.yml fájl
* gitea-admin-password-sealed.yaml fájl

### A telepítés menete:
**1. Namespaces létrehozása**
A Gitea-t célszerű külön namespace-be telepíteni és engedélyezni az istio sidecar injectiont.
```
kubectl create ns gitea
```
```
kubectl label namespace gitea istio-injection=enabled
```

**2. A megfelelő secretek telepítése**
Annak érdekében, hogy az admin jelszót ne kelljen plain-textben tárolni, szükséges, hogy a sealed secretet előzetesen telepítsük a klaszterbe.
(Sealed secrets: https://github.com/bitnami-labs/sealed-secrets)

```
kubectl apply -f admin-password-sealed.yaml -n gitea
kubectl apply -f gitea-config-mailer-password-secret-sealed.yaml -n gitea
```

**3. Gateway, mtls és Virtual service telepítése**
Annak érdekében, hogy a telepített Gitea-t klaszteren kívülről is el lehessen érni, az szükséges, hogy az ISTIO systemnek megadjuk a gateway-t, illetve egy virtualservice típusú erőforrást hozzunk létre. Ezt megtehetjük az alábbi fájl "telepítével":
```
kubectl apply -f istio-peer-authentication.yaml
```
```
kubectl apply -f istio-virtual-service.yml
```

Telepítsük a self-signed tanusítványt is:
```
kubectl apply -f istio-self-signed-cert.yaml
```

**4. Helm chart telepítése**
A gitea helmchart oldalán szükséges repozitory hozzáadása, majd telepítése a megfelelő values fájl segítségével. Fontos, hogy a `gitea-values.yml` fájlt előtte a szükséges paraméterekkel ki kelle egészíteni.

```
helm repo add gitea-charts https://dl.gitea.io/charts/
helm repo update
helm upgrade --install gitea gitea-charts/gitea --values chart-values.yml --namespace gitea
```

**5. Teszt**
Ha minden jól ment, akkor a values fájlban megadott domain név meglátogatásával a Gitea UI felületét kellene látnunk!