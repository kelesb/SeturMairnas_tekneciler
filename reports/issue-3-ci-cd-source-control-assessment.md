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

İnceleme anındaki repository envanteri, repository root görünümü ve görülebilen tracked dosyaların tamamı üzerinden şu şekilde gözlemlenmiştir:

- repo kökünde yalnızca `rules/` klasörü yer almaktadır (`.git/` hariç)
- `rules/` altında şu iki dosya bulunmaktadır:
  - `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt`
  - `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt`

Bu yöntemle gözlemlenen tracked repository içeriğinde `.github/workflows/`, `azure-pipelines.yml`, `Jenkinsfile`, deployment manifesti, test projesi veya build betiği görülmemektedir. Bu nedenle aşağıdaki birçok kontrol için uygulama kanıtı değil, yalnızca dokümante edilmiş beklenti görülebilmektedir.

## Bulgular tablosu

<table>
  <thead>
    <tr>
      <th>Kontrol alanı</th>
      <th>Durum</th>
      <th>Kanıt</th>
      <th>Değerlendirme</th>
      <th>Uygulanabilir öneri</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Pipeline dosyaları</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Repo envanterinde çalıştırılabilir pipeline tanımı yok; kılavuz yalnızca workflow dosyalarının repoda bulunması gerektiğini söylüyor (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:43-45</code>, <code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-148</code>).</td>
      <td>Repository dışında yönetilen bir pipeline olasılığı dışlanamadığı için, CI/CD uygulamasının varlığı bu repodan kesin olarak doğrulanamıyor.</td>
      <td><code>.github/workflows/</code> altında en az build, test, security/dependency scan ve deployment aşamalarını içeren bir workflow ekleyin.</td>
    </tr>
    <tr>
      <td>Build adımları</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kılavuz build/test/güvenlik kontrollerinin merge öncesi tamamlanmasını bekliyor (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:62-64</code>, <code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-146</code>), ancak repoda build tanımı yok.</td>
      <td>Derleme adımlarının nasıl çalıştırıldığı kaynak koddan doğrulanamıyor.</td>
      <td>Teknoloji yığınına uygun derleme komutunu pipeline içinde açık bir job olarak tanımlayın.</td>
    </tr>
    <tr>
      <td>Test adımları</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Test kodları ve test yapılandırmalarının push edilmesi gerektiği belirtilmiş (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:45</code>, <code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:298-301</code>), ancak repoda test veya test workflow’u yok.</td>
      <td>Otomatik test doğrulamasının nerede ve nasıl yapıldığı repodan doğrulanamıyor.</td>
      <td>Mevcut teknolojiye uygun test projesi/betiği ve bunu çalıştıran CI adımı ekleyin.</td>
    </tr>
    <tr>
      <td>Quality gate</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Değerlendirme dokümanı quality gate durumunun değerlendirilmesini bekliyor (<code>rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:46-52</code>), fakat repository içinde SonarQube, coverage eşiği veya benzeri bir gate tanımı yok.</td>
      <td>Kalite eşiğinin uygulanıp uygulanmadığı kaynak koddan doğrulanamıyor.</td>
      <td>Coverage, statik analiz ve kalite eşiğini pipeline’da zorunlu hale getirin.</td>
    </tr>
    <tr>
      <td>Security kontrolleri</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kılavuz code scanning ve secret scanning çalıştırılmasını şart koşuyor (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:123-133</code>), ancak bunu başlatan workflow veya konfigürasyon yok.</td>
      <td>Güvenlik kontrollerinin PR aşamasında otomatik çalışıp çalışmadığı repodan doğrulanamıyor.</td>
      <td>CodeQL/code scanning ve secret scanning tetiklerini PR ve push olaylarında çalışacak şekilde ekleyin.</td>
    </tr>
    <tr>
      <td>Dependency kontrolleri</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kılavuz dependency review kontrolünü bekliyor (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:127-133</code>), fakat dependency taramasını gösteren workflow ya da manifest doğrulaması yok.</td>
      <td>Bağımlılık taramasının otomatik çalışıp çalışmadığı kaynak koddan doğrulanamıyor.</td>
      <td>Dependency review veya ekosisteme uygun SCA taramasını PR pipeline’ına ekleyin.</td>
    </tr>
    <tr>
      <td>Artifact yönetimi</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Repo envanterinde artifact upload, publish veya retention tanımı içeren workflow/config dosyası yok.</td>
      <td>Build çıktılarının CI içinde artifact olarak saklanıp saklanmadığı kaynak koddan doğrulanamıyor.</td>
      <td>Build çıktısını artifact olarak yayınlayan ve retention ayarı yapan bir job ekleyin.</td>
    </tr>
    <tr>
      <td>Ortama göre deployment</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kılavuz, ortam bazlı deployment beklentisi tanımlıyor (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-149</code>), ancak repoda environment bazlı deploy tanımı veya manifest yok.</td>
      <td>Hangi ortamlara, hangi koşullarla dağıtım yapıldığı kaynak koddan doğrulanamıyor.</td>
      <td>Ortam bazlı deployment job’ları, environment korumaları ve kullanılan hedefleri repoda tanımlayın.</td>
    </tr>
    <tr>
      <td>Production onayı</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Repoda production deployment gate veya environment approval’ı açıkça tanımlayan pipeline/environment dosyası yok. <code>rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:57-59</code> gereği bu konuda varsayım yapılamaz.</td>
      <td>Production’a çıkış için manuel onay mekanizmasının varlığı repodan doğrulanamıyor.</td>
      <td>GitHub Environments veya eşdeğer onay mekanizmasını yapılandırın ve pipeline dosyasında açıkça referans verin.</td>
    </tr>
    <tr>
      <td>Pipeline secret kullanımı</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kılavuz gerçek secret’ların repoda olmamasını ve gizli değerlerin Actions Secrets / App Settings / Key Vault üzerinden verilmesini ister (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:83-96</code>), ancak secret tüketen bir pipeline tanımı yok.</td>
      <td>Secret yönetimi yaklaşımı dokümante edilmiş olsa da fiili kullanım şekli doğrulanamıyor.</td>
      <td>Workflow dosyalarında secret referanslarını açıkça tanımlayın; secret adlarını belgeleyin, değerleri repoya koymayın.</td>
    </tr>
    <tr>
      <td>Branch/PR akışı — dokümante beklenti</td>
      <td><strong>Mevcut (dokümantasyon düzeyinde)</strong></td>
      <td>Zorunlu branching ve PR akışı yazılı olarak tanımlanmış (<code>rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:49-66</code>).</td>
      <td><code>main</code>, <code>development</code>, <code>feature/*</code> ve <code>fix/*</code> için hedeflenen süreç belgelenmiş.</td>
      <td>Bu akışı repository ayarları ve workflow tetikleriyle teknik olarak zorunlu hale getirin.</td>
    </tr>
    <tr>
      <td>Branch protection / required checks / PR policy enforcement</td>
      <td><strong>Değerlendirilemedi</strong></td>
      <td>Kaynak kodda branch protection, required status checks, CODEOWNERS, PR template veya review zorunluluğunu kanıtlayan bir dosya yok. <code>rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:57-59</code> gereği varsayım yapılmamalıdır.</td>
      <td>Belgelenmiş süreç var; ancak bunun GitHub ayarlarıyla uygulanıp uygulanmadığı bu repodan doğrulanamıyor.</td>
      <td>Koruma kurallarını repository settings tarafında etkinleştirin; mümkün olan kısımları CODEOWNERS, PR template ve workflow required checks ile görünür hale getirin.</td>
    </tr>
  </tbody>
</table>

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
