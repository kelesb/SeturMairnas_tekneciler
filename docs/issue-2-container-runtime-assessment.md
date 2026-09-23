# Issue #2 — AI Architecture Assessment: Container ve çalışma ortamı standartları

## Kapsam ve yöntem

Bu değerlendirme yalnızca repository içinde doğrulanabilen artifact'lere dayanır. `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:60-66` container ve çalışma ortamı incelemesinin Dockerfile, deployment manifestleri, secret yönetimi, health check, kaynak limitleri ve stateless çalışma başlıklarını birlikte değerlendirmesini; güvenlik ve operasyon risklerini ayırmasını; kanıt olmayan altyapı davranışları için varsayım yapılmamasını ister.

## Depoda doğrulanan mevcut kanıt

23 Eylül 2026 tarihli repository envanterinde yalnızca aşağıdaki dosyalar bulunmuştur:

- `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt`
- `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt`

`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:72-80` repo içinde `docs/`, workflow dosyaları, `Dockerfile`, container yapılandırmaları, `.env.example` ve deployment/IaC dosyalarının bulunması gerektiğini tanımlar. İncelenen committe bu artifact'ler mevcut değildir.

## Bulgular

| Başlık | Durum | Kanıt | Güvenlik riski | Operasyonel risk | Uygulanabilir teknik öneri |
| --- | --- | --- | --- | --- | --- |
| Dockerfile ve base image seçimi | Değerlendirilemedi | Repo kökünde `Dockerfile` veya eşdeğeri yok. Değerlendirme kapsamı `rules/AI_Architecture_Assessment_Agent_Değerlendirme_Dokümanı.txt:61-66`, beklenen artifact'ler `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:77-80`. | Onaylı/minimal base image, paket yüzeyi ve CVE azaltımı doğrulanamıyor. | Image build süreci ve taşınabilir runtime standardı kanıtlanamıyor. | Uygulama dili netleşince onaylı ve küçük bir base image kullanan `Dockerfile` ekleyin; final stage için yalnızca runtime bağımlılıklarını bırakın. |
| Multi-stage build ve image boyutu | Değerlendirilemedi | Build tarifi bulunmadığı için stage ayrımı veya image küçültme stratejisi gözlemlenemiyor. | Gereksiz build araçlarının production image'a taşınıp taşınmadığı doğrulanamıyor. | Büyük image nedeniyle daha yavaş build/deploy ve daha yüksek başlangıç süresi riski ölçülemiyor. | Ayrı builder/final stage kullanın; bağımlılık cache'ini builder'da tutup final image'a yalnızca yayın çıktısını kopyalayın. |
| Non-root çalışma | Değerlendirilemedi | `USER` tanımı içeren Dockerfile veya runtime manifesti yok. | Root konteyner çalışması riski dışlanamıyor. | Platform politika uyumsuzluğu ve izin kaynaklı runtime sorunları önceden tespit edilemiyor. | Final image içinde sabit UID/GID'li non-root kullanıcı oluşturup container'ı bu kullanıcı ile başlatın. |
| Port ve health check | Değerlendirilemedi | `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:193-197` `/health` endpoint'i bekliyor; ancak uygulama kodu, manifest veya container health check tanımı yok. | Sağlık endpoint'inin hassas veri sızdırıp sızdırmadığı doğrulanamıyor. | Liveness/readiness veya App Service health probe olmadan hatalı instance'ların trafik almaya devam etme riski var. | Uygulamada `/health` endpoint'i ekleyin; secret döndürmeyen kısa JSON yanıt üretin; deployment manifestinde veya platform ayarında health probe tanımlayın. |
| Environment variable ve secret yönetimi | Kısmen değerlendirilebildi | `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:82-96` gerçek secret'ların repoya girmemesi ve gizli değerlerin GitHub Actions Secrets, Azure App Settings veya Key Vault ile verilmesi gerektiğini söylüyor. Ancak `.env.example`, config şablonu veya deployment secret wiring kanıtı yok. | Secret enjeksiyon zinciri uygulanmadığı için güvenli konfigürasyon standardı doğrulanamıyor. | Ortamlar arası konfigürasyon tekrarlanabilirliği ve bootstrap süreci belirsiz. | Gerçek değer içermeyen `.env.example` ve config şablonları ekleyin; production secret'larını App Settings/Key Vault referanslarıyla besleyin; repository'de yalnızca placeholder tutun. |
| CPU/memory limitleri | Değerlendirilemedi | Kubernetes/App Service/Terraform/Bicep/Helm manifesti yok; bu nedenle request/limit veya SKU kanıtı yok. | Doğrudan bir açıklık kanıtı yok; ancak kaynak sınırlaması yoksa hizmet tüketimi saldırılarına dayanıklılık değerlendirilemiyor. | Yoğun yükte gürültülü komşu etkisi, OOMKill veya kontrolsüz maliyet riski değerlendirilemiyor. | Deployment manifestlerinde açık CPU/memory request-limit alanları veya App Service plan boyutu kanıtı ekleyin. |
| Stateless çalışma | Değerlendirilemedi | Uygulama kodu ve deployment manifesti bulunmadığı için container içinde local disk/session state kullanımı doğrulanamıyor. | Stateful geçici dosyaların hassas veri tutup tutmadığı bilinmiyor. | Yatay ölçeklenebilirlik ve yeniden başlatma dayanıklılığı doğrulanamıyor. | Runtime state'i dış servislere taşıyın; container dosya sistemi ve process memory'sini geçici kabul eden tasarımı belgeleyin. |
| Deployment manifestleri ve CI/CD bağı | Değerlendirilemedi | `.github/workflows` yok; deployment/IaC dosyaları yok. `rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:145-148` build, test, güvenlik kontrolü, image üretimi ve deploy adımlarını zorunlu kılıyor. | Image tarama, secret scanning ve deploy öncesi güvenlik kontrolleri repo içinden kanıtlanamıyor. | Tekrarlanabilir deploy süreci ve değerlendirilen commit ile runtime ilişkisi (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:197`) gösterilemiyor. | CI/CD workflow'ları ekleyin; build/test/security scan/image publish/deploy zincirini tek committen izlenebilir hâle getirin. |

## Güvenlik riskleri

1. **Base image ve non-root kanıtı yok:** Bu committe container güvenlik sertleştirmesi doğrulanamıyor; bu nedenle root çalışma, geniş paket yüzeyi ve patch seviyesi bilinmez kalıyor.
2. **Secret yönetimi uygulama kanıtı yok:** Politika dokümante edilmiş olsa da (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:82-96`), gerçek uygulama artifact'i olmadığından secret'ların nasıl enjekte edildiği doğrulanamıyor.
3. **Build/deploy güvenlik kontrolleri yok:** Workflow ve deployment tanımı olmadığından image scanning, secret scanning ve güvenli yayın zinciri repo içinden ispatlanamıyor.

## Operasyonel riskler

1. **Health check ve port sözleşmesi doğrulanamıyor:** `/health` beklentisi tanımlı (`rules/katilimci_github_ve_AI_kullanim_klavuzu.txt:193-194`), ancak uygulama ve manifest yok.
2. **Kaynak limitleri kanıtlanamıyor:** CPU/memory sınırları bilinmediğinden kapasite planlama ve maliyet kontrolü değerlendirilemiyor.
3. **Stateless çalışma ve yeniden üretilebilir deploy kanıtı yok:** Manifest ve uygulama kodu olmadığı için ölçeklenebilirlik ve commit-to-runtime izlenebilirliği gösterilemiyor.

## Sonuç

Bu commit için container ve çalışma ortamı standardı açısından ana sonuç **uygunsuz yapılandırma tespiti değil, değerlendirme için gerekli artifact eksikliği**dir. Mevcut repo içeriği issue #2'nin istediği Dockerfile, deployment manifesti, workflow ve runtime konfigürasyon kanıtlarını sağlamadığından, güvenli veya güvensiz bir çalışma ortamı hakkında kesin hüküm verilemez.

## Öncelikli sonraki adımlar

1. Uygulamayı ve `Dockerfile`'ı repo içine ekleyin.
2. Multi-stage, non-root ve küçük final image yaklaşımını Dockerfile içinde kanıtlayın.
3. Secret'sız `.env.example` ve ortam bazlı konfigürasyon şablonları ekleyin.
4. Deployment/IaC manifestlerinde health check, port, CPU/memory ve stateless varsayımlarını açıkça gösterin.
5. Build/test/security scan/image publish/deploy adımlarını içeren GitHub Actions workflow'larını ekleyin.
