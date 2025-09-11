---
icon: tree-deciduous
---

# Basics of CICD

## CI/CD nedir? (kısaca)

* **CI (Continuous Integration)**: Kodlar küçük parçalar halinde sık sık **main**’e entegre edilir. Her entegrede otomatik **build + test + scan** çalışır. Amaç: hatayı erken yakalamak.
* **CD** iki anlama gelir:
  * **Continuous Delivery**: Prod’a gitmeden **manuel onay** adımı vardır.
  * **Continuous Deployment**: Testler geçerse **otomatik** prod’a gider, onay yok.

## Tipik akış

1. Kod **Git** repo’da (GitHub/GitLab/Bitbucket fark etmez).
2. Geliştirici, yeni iş için **feature branch** açar: `feature/login-page`.
3. Kod yazar → **commit** → **pull request (PR)** açar.
4. PR açılınca **CI pipeline** tetiklenir:
   * **Unit tests** (örn. `npm test`, `mvn test`)
   * **Dependency scan** (örn. OWASP Dependency-Check)
   * **Build/Artifact** (örn. `.jar`, Docker image)
   * **Static/Vulnerability scan** (örn. SonarQube, Trivy)
5. Bir adım fail olursa geliştirici düzeltip **aynı PR’a push** eder; CI yeniden koşar.
6. CI yeşil + **code review** onayı → PR **merge** edilir.
7. **Main**’e merge olunca CI **yeniden** koşar.\
   Neden tekrar? Çünkü A ve B feature’ları **birlikte** ilk kez main’de buluştu; ayrı ayrı geçse de beraberken patlayabilir. Bu ekstra koşu “entegrasyon” sorunlarını yakalar.
8. **CD aşaması**:
   * **Staging**’e otomatik deploy + **smoke test**.
   * **Continuous Deployment** ise staging’den sonra **prod’a otomatik** geçiş.
   * **Continuous Delivery** ise prod’a geçmeden **manuel onay (human gate)** ister.

## Jenkins’e bağlayalım

Jenkins’te bu akış **Jenkinsfile** ile koda dökülür. Önemli parçalar:

* **agent**: Pipeline’ın koşacağı **node/agent**.
* **stages**: Build, Test, Scan, Package, Deploy gibi adımlar.
* **triggers**: PR/push ile otomatik tetikleme (SCM webhook).
* **when**: Hangi branch’te hangi stage koşacak?
* **input**: Continuous Delivery için **manuel onay** adımı.

### Minimal örnek Jenkinsfile

Aşağıdaki örnek Node.js üzerinden (Java/Maven de benzer).\
Feature branch’te **sadece CI**, `main`’de **CI + staging deploy + (opsiyonel) prod**:

```groovy
pipeline {
  agent any
  options { timestamps() }
  triggers { pollSCM('@daily') } // pratikte webhook önerilir

  environment {
    APP_NAME = 'demo-app'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        sh 'node -v || true'
        sh 'npm ci'
        sh 'npm run build'
      }
    }

    stage('Unit Tests') {
      steps { sh 'npm test -- --ci --reporter=junit --reporter-options "mochaFile=reports/junit.xml"' }
      post { always { junit 'reports/junit.xml' } }
    }

    stage('Dependency Scan') {
      steps { sh 'npx audit-ci --low' } // örnek; kurumsalda OWASP Dependency-Check tercih edilebilir
    }

    stage('Package Artifact') {
      steps { sh 'tar -czf build-artifact.tgz dist package.json' }
      post { success { archiveArtifacts artifacts: 'build-artifact.tgz', fingerprint: true } }
    }

    stage('Deploy to Staging') {
      when { branch 'main' }
      steps {
        // örnek deploy: ssh/scp, helm, kubectl, ansible vs.
        sh 'echo "Deploying to STAGING..."'
        sh 'sleep 3'
      }
    }

    stage('Smoke Tests (Staging)') {
      when { branch 'main' }
      steps { sh 'curl -f http://staging.example.internal/healthz' }
    }

    stage('Manual Approval for Prod') {
      when { branch 'main' }
      steps {
        // Continuous Delivery (manuel onay). Continuous Deployment istersen bu stage'i kaldır.
        input message: 'Prod’a deploy etmek için onay verin', ok: 'Deploy'
      }
    }

    stage('Deploy to Prod') {
      when { branch 'main' }
      steps {
        sh 'echo "Deploying to PROD..."'
        sh 'sleep 3'
      }
    }
  }
}

```

Notlar:

* **Feature branch** push/PR: “Deploy” stage’leri koşmaz; **Build/Test/Scan** yeter.
* **Main** merge: tamamı koşar; staging → smoke → **manuel onay** (CD) → prod.
* Otomatik prod (Continuous Deployment) istersen **Manual Approval** stage’ini sil.

## Basit Git akışı (örnek komutlar)

```bash
# yeni özellik
git checkout -b feature/login
# kod yaz…
git add .
git commit -m "login page"
git push -u origin feature/login

# GitHub/GitLab'da PR aç → CI otomatik koşar → review → merge
# main'e merge → Jenkins main pipeline → staging → (onay) → prod
```
