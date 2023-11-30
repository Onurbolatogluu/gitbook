# 🥳 CRD & Operator

CRD ile bizler kubernetes'in default objeleri gibi kendimiz custom objeler yaratabiliyoruz. Yani aynen bir pod,deployment vb obje tipleri gibi kendimiz bir custom obje tipinin tanımını kubernetes'e gönderip, kendi oluşturduğumuz obje tipine göre objeler oluşturabiliyoruz.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```

Yukarıdaki örnekteki yaml dosyasını biz kubernetes'e gönderirsek, crontabs.stable.example.com isminde bir obje tipi oluşturmuş oluyoruz.&#x20;

Operator ise, cluster admin'in veya developer'ın cluster'da yapacağı işlerin otomatize hale getirilmesinde yardımcı olmaktadır.&#x20;

Misal bir database oluşturmak için, gerekli tanımları yaml dosyasına girer ve bunu kubernetes'e göndeririz. Ardından gerekirse database uygulaması için gerekli tanımları vb girmemiz gerekebilir. Yani süreç burada manuel işliyor. Bizler bunun yerine bir operator oluşturup, CRD'leri kullanarak tüm bu manuel süreçleri otomatik hale getiriyoruz.&#x20;

Yani şunu diyebiliyoruz; Bir adet DB adında CRD oluşturuyoruz, bu DB CRD'inden DB objesi oluşturuyoruz, daha sonra yazdığımız operator ile ne zaman bir DB objesi yaratılırsa, kubernetes de git bir mysql deployment yarat şeklinde bir workflow kurabiliyoruz. Yani işin obje kısmını CRD ile halledip, otomatize etme kısmını da operator ile hallediyoruz. Kubernetes operator konusunda bize default bir SDK sağlamıyor. Fakat operator desteği sunan bir çok SDK mevcut. Aşağıdaki linkten bunları inceleyebilirsiniz.

{% embed url="https://kubernetes.io/docs/concepts/extend-kubernetes/operator/" %}

### Service Mesh

Kubernetes üzerinde bir çok servisi ayrı, ayrı podlar olarak çalıştırıyoruz. Bu kubernetes podları üzerinde çalışan servislerin birbirleri olan iletişiminin durumu, servislerin sorunlarının debug edilebilmesi gibi bir çok kolaylığı sağlayan uygulamalar service mesh uygulamalarıdır. Service mesh uygulamalarının servislerin durumu hakkında bilgi edinebilmesi ve proxy olarak görev alan kısım ise servislerimiz ile aynı pod içerisinde çalışan sidecar containerlardır. Proxy görevi gören sidecar containerlar asıl servisimize gelen tüm trafiği üzerinden geçirir.

{% embed url="https://linkerd.io/2.12/getting-started/" %}

{% embed url="https://istio.io/" %}



