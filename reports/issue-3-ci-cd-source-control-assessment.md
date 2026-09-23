# Issue #3 — AI Architecture Assessment: CI/CD ve kaynak kod yönetimi kontrolleri

## İnceleme kapsamı

Bu değerlendirme, issue #3 kapsamındaki şu başlıkları yalnızca repository içinden doğrulanabilen kanıtlara dayanarak inceler:

- pipeline dosyaları
- build/test adımları
- quality gate
- security kontrolleri
- dependency kontrolleri
- artifact yönetimi
- ortama göre deployment
- production onayı
- pipeline secret kullanımı
- branch/PR politikası kanıtları

## Repo envanteri özeti

İnceleme anındaki repository envanteri şu şekilde gözlemlenmiştir:

- repo kökünde yalnızca `rules/` klasörü yer almaktadır (`.git/` hariç)
- `rules/` altında şu iki dosya bulunmaktadır:
  - `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt`
  - `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt`

Gözlemlenen repository snapshot’ında `.github/workflows/`, `azure-pipelines.yml`, `Jenkinsfile`, `Dockerfile`, deployment manifesti, test projesi veya build betiği görülmemektedir. Bu nedenle aşağıdaki birçok kontrol için uygulama kanıtı değil, yalnızca dokümante edilmiş beklenti görülebilmektedir.

## Bulgular tablosu

| Kontrol alanı | Durum | Kanıt | Değerlendirme | Uygulanabilir öneri |
| --- | --- | --- | --- | --- |
| Pipeline dosyaları | **Değerlendirilemedi** | Repo envanterinde çalıştırılabilir pipeline tanımı yok; kılavuz yalnızca workflow dosyalarının repoda bulunması gerektiğini söylüyor (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:43-45`, `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-148`). | Repository dışında yönetilen bir pipeline olasılığı dışlanamadığı için, CI/CD uygulamasının varlığı bu repodan kesin olarak doğrulanamıyor. | `.github/workflows/` altında en az build, test, security/dependency scan ve deployment aşamalarını içeren bir workflow ekleyin. |
| Build adımları | **Değerlendirilemedi** | Kılavuz build/test/güvenlik kontrollerinin merge öncesi tamamlanmasını bekliyor (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:62-64`, `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-146`), ancak repoda build tanımı yok. | Derleme adımlarının nasıl çalıştırıldığı kaynak koddan doğrulanamıyor. | Teknoloji yığınına uygun derleme komutunu pipeline içinde açık bir job olarak tanımlayın. |
| Test adımları | **Değerlendirilemedi** | Test kodları ve test yapılandırmalarının push edilmesi gerektiği belirtilmiş (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:45`, `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:298-301`), ancak repoda test veya test workflow’u yok. | Otomatik test doğrulamasının nerede ve nasıl yapıldığı repodan doğrulanamıyor. | Mevcut teknolojiye uygun test projesi/betiği ve bunu çalıştıran CI adımı ekleyin. |
| Quality gate | **Değerlendirilemedi** | Değerlendirme dokümanı quality gate durumunun değerlendirilmesini bekliyor (`rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:46-52`), fakat repository içinde SonarQube, coverage eşiği veya benzeri bir gate tanımı yok. | Kalite eşiğinin uygulanıp uygulanmadığı kaynak koddan doğrulanamıyor. | Coverage, statik analiz ve kalite eşiğini pipeline’da zorunlu hale getirin. |
| Security kontrolleri | **Değerlendirilemedi** | Kılavuz code scanning ve secret scanning çalıştırılmasını şart koşuyor (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:123-133`), ancak bunu başlatan workflow veya konfigürasyon yok. | Güvenlik kontrollerinin PR aşamasında otomatik çalışıp çalışmadığı repodan doğrulanamıyor. | CodeQL/code scanning ve secret scanning tetiklerini PR ve push olaylarında çalışacak şekilde ekleyin. |
| Dependency kontrolleri | **Değerlendirilemedi** | Kılavuz dependency review kontrolünü bekliyor (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:127-133`), fakat dependency taramasını gösteren workflow ya da manifest doğrulaması yok. | Bağımlılık taramasının otomatik çalışıp çalışmadığı kaynak koddan doğrulanamıyor. | Dependency review veya ekosisteme uygun SCA taramasını PR pipeline’ına ekleyin. |
| Artifact yönetimi | **Değerlendirilemedi** | Repo envanterinde artifact upload, publish veya retention tanımı içeren workflow/config dosyası yok. | Build çıktılarının CI içinde artifact olarak saklanıp saklanmadığı kaynak koddan doğrulanamıyor. | Build çıktısını artifact olarak yayınlayan ve retention ayarı yapan bir job ekleyin. |
| Ortama göre deployment | **Değerlendirilemedi** | Kılavuz, hedef süreç olarak ACR/App Service tabanlı deployment beklentisi tanımlıyor (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:147-149`), ancak repoda environment bazlı deploy tanımı veya manifest yok. | Hangi ortamlara, hangi koşullarla dağıtım yapıldığı kaynak koddan doğrulanamıyor. | Ortam bazlı deployment job’ları, environment korumaları ve kullanılan hedefleri repoda tanımlayın. |
| Production onayı | **Değerlendirilemedi** | Repoda production deployment gate veya environment approval’ı açıkça tanımlayan pipeline/environment dosyası yok. `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:57-59` gereği bu konuda varsayım yapılamaz. | Production’a çıkış için manuel onay mekanizmasının varlığı repodan doğrulanamıyor. | GitHub Environments veya eşdeğer onay mekanizmasını yapılandırın ve pipeline dosyasında açıkça referans verin. |
| Pipeline secret kullanımı | **Değerlendirilemedi** | Kılavuz gerçek secret’ların repoda olmamasını ve gizli değerlerin Actions Secrets / App Settings / Key Vault üzerinden verilmesini ister (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:83-96`), ancak secret tüketen bir pipeline tanımı yok. | Secret yönetimi yaklaşımı dokümante edilmiş olsa da fiili kullanım şekli doğrulanamıyor. | Workflow dosyalarında secret referanslarını açıkça tanımlayın; secret adlarını belgeleyin, değerleri repoya koymayın. |
| Branch/PR akışı — dokümante beklenti | **Mevcut (dokümantasyon düzeyinde)** | Zorunlu branching ve PR akışı yazılı olarak tanımlanmış (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:49-66`). | `main`, `development`, `feature/*` ve `fix/*` için hedeflenen süreç belgelenmiş. | Bu akışı repository ayarları ve workflow tetikleriyle teknik olarak zorunlu hale getirin. |
| Branch protection / required checks / PR policy enforcement | **Değerlendirilemedi** | Kaynak kodda branch protection, required status checks, CODEOWNERS, PR template veya review zorunluluğunu kanıtlayan bir dosya yok. `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:57-59` gereği varsayım yapılmamalıdır. | Belgelenmiş süreç var; ancak bunun GitHub ayarlarıyla uygulanıp uygulanmadığı bu repodan doğrulanamıyor. | Koruma kurallarını repository settings tarafında etkinleştirin; mümkün olan kısımları CODEOWNERS, PR template ve workflow required checks ile görünür hale getirin. |

## Security, dependency ve quality kontrollerinin ayrı değerlendirmesi

### Security

- **Durum:** Değerlendirilemedi
- **Kanıt:** `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:123-133`
- **Sonuç:** Code scanning ve secret scanning beklentisi var, fakat bunu uygulayan workflow veya sonuç kanıtı repoda bulunmadığı için kontrolün çalıştığı doğrulanamıyor.

### Dependency

- **Durum:** Değerlendirilemedi
- **Kanıt:** `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:127-133`
- **Sonuç:** Dependency review bekleniyor, ancak repoda taramayı çalıştıran bir tanım bulunmadığı için kontrolün uygulandığı doğrulanamıyor.

### Quality

- **Durum:** Değerlendirilemedi
- **Kanıt:** `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:46-52`
- **Sonuç:** Quality gate, coverage veya statik analiz eşiği repoda tanımlı görünmediğinden kalite kontrolünün nasıl uygulandığı doğrulanamıyor.

## Öncelikli aksiyonlar

1. `.github/workflows/` altında minimum build + test + security/dependency scan + deploy akışını ekleyin.
2. PR üzerinde çalışan zorunlu quality gate ve güvenlik kontrollerini required check olarak tanımlayın.
3. Artifact üretimi, saklama süresi ve deployment hedeflerini pipeline’da görünür hale getirin.
4. Branch/PR politikasını yalnız dokümantasyonda bırakmayın; CODEOWNERS, PR template, environment approval ve branch protection ile uygulanabilir hale getirin.

## Sonuç

Repository, CI/CD ve kaynak kod yönetimi için beklentileri açıklayan dokümanlar içeriyor; ancak bu beklentileri uygulayan pipeline ve enforcement kanıtları repoda bulunmuyor. Bu nedenle pipeline, build, test, quality, security, dependency, artifact ve deployment kontrolleri ile kaynak koddan doğrulanamayan branch protection, PR enforcement ve production approval başlıkları için sonuç **Değerlendirilemedi** olmuştur; yalnızca branch/PR akışına ilişkin dokümante beklenti repoda açıkça görülebilmektedir.
