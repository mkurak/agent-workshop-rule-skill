---
name: rule-wizard
description: "Kural eklemeden önce seçenekli soru-cevap turlarıyla netleştirme sihirbazı. Bağlamdan eksik detayları yakalar, alternatifleri seçenek olarak gösterir, dinamik çoklu kural algılar, sonunda netleşen kural(lar)ı /rule ile ekler. 3 kapsam: proje, --global, --team."
argument-hint: "[--global|--team] <Türkçe bağlam — kuralın genel konusu>"
---

# /rule-wizard Skill

Bir kuralı `/rule` ile direkt yazmadan önce, tek başına yazarken kolayca atlanabilecek detayları (edge case, istisna, alternatif formülasyon, kapsam, motivasyon, örnek varyantları) **seçenekli soru-cevap turlarıyla** ortaya çıkaran sihirbaz. Tartışma tamamlanınca netleşen kural(lar)ı `/rule` skill'ine aktararak dosyaya yazdırır.

> İlgili skill: [/rule](../rule/skill.md) — Bu skill'in sonunda çağırdığı kural yazıcı.
> Sistem dokümantasyonu: [.claude/docs/coding-rules-system.md](../../docs/coding-rules-system.md)

---

## Parametre Zorunluluğu

Skill her zaman bir **bağlam** argümanı ile çağrılır. Bağlam; kuralın genel konusunu, ilk taslak fikri veya karşılaşılan sorunu tanımlayan kısa bir Türkçe metindir.

- ✅ `/rule-wizard API'de logging kullanımı`
- ✅ `/rule-wizard Worker doğrudan DB'ye bağlanmasın`
- ✅ `/rule-wizard Controller'larda try-catch yazılmasın, global handler üstlensin`
- ❌ `/rule-wizard` (bağlamsız çağrı)

**Bağlam verilmediyse:** Akışa devam etme. Kullanıcıya kısa ve net şekilde bağlam iste:

> "Bu skill bir başlangıç bağlamı ister. Lütfen kuralın genel konusunu veya ilk fikrini kısaca yaz: `/rule-wizard <kuralın konusu>`. Örnek: `/rule-wizard Worker Jobs'ların hata yönetimi`."

Kullanıcı yanıt verene kadar hiçbir dosya okuma veya soru sorma yapma.

---

## Faz 1 — Anlama ve Hazırlık

### 1.1 Mevcut Kuralları Oku (Zorunlu)

Sorgulamaya başlamadan **önce** şu dosyaları mutlaka oku:

- `.claude/rules/coding-common.md`
- `.claude/docs/coding-standards/` dizinindeki **tüm .md dosyaları** (her projede farklı olabilir — dinamik olarak listele ve oku)

Amaç:
- **Duplikasyon önleme:** Aynı veya çok benzer kural zaten var mı?
- **Çakışma tespiti:** Yeni kural mevcut bir kuralla çelişiyor mu?
- **Genişletme fırsatı:** Yeni kural, mevcut bir kuralın içine bir madde olarak eklenebilir mi?
- **Cross-reference:** Final adımda `Related` alanı için referans verilebilecek kurallar.

### 1.2 Bağlamı Analiz Et

Kullanıcının verdiği bağlamdan şunları çıkar (sessizce, yanıtta göstermeden):

- **Muhtemel kapsam (scope):** Hangi uygulama(lar)? Common mu, tek uygulama mı?
- **Muhtemel niyet:** Zorlayıcı (must) / yasaklayıcı (must not) / yönlendirici (should)?
- **Etkilenen katmanlar:** Controller, Service, Repository, Consumer, Hub, Job?
- **İlk hipotezler:** Apply when, Why, Examples için ham fikirler.
- **Benzer mevcut kurallar:** 1.1'de bulduğun varsa id'lerini not et.

### 1.3 Analiz Özetini Sun

Kullanıcıya **kısa bir paragraf** halinde ne anladığını özetle. Örnek:

> "Anladığım kadarıyla API tarafında controller'ların içinde try-catch yazılmamasını, hata yönetiminin üstteki bir global handler üzerinden yapılmasını istiyorsun. Bu, `coding-standards/api.md` kapsamına giriyor ve mevcut `no-logic-in-bridges` kuralını tamamlar nitelikte. Şimdi birkaç soruyla detayları netleştireceğim."

Sonra Faz 2'ye geç.

---

## Faz 2 — Seçenekli Sorgulama

### Temel İlkeler (Hepsi Bağlayıcı)

1. **Her soru `AskUserQuestion` aracı ile sorulur.** Düz metin, açık uçlu soru yok. "Şunu açıklar mısın?" gibi sorular yasak — her zaman seçenek üret.
2. **Her soruda 2-4 seçenek.** Platform limiti 4. Daha fazla makul seçenek varsa soruyu böl, asla "en iyi 4"ü seçmekle kısıtlanma.
3. **"Diğer" seçeneği otomatik eklenir.** Kullanıcı serbest metin girebilir, bunu açıkça seçeneğe yazma — araç kendisi ekler.
4. **Önerilen seçenek varsa ilk sıraya koy** ve etiketinde `(Recommended)` yaz. Hangi seçeneği neden önerdiğini seçeneğin `description` alanında kısaca belirt.
5. **Türkçe sor, Türkçe seçenek sun.** `/rule` son adımda İngilizce'ye çevirecek — bu skill kullanıcıyla hep Türkçe konuşur.
6. **Tek turda en fazla 4 soru.** 4'ten fazla soru gerekiyorsa tura böl — birinci turun cevapları ikinci turu besleyebilir.
7. **Bağlamdan net çıkan alana tekrar soru sorma.** Bunun yerine "Şunu şöyle anladım — doğru mu?" onay sorusu sor (ikili seçenek: Doğru / Hayır değiştireceğim).
8. **Seçeneklerin birbirinden farklı ve net olması gerekir.** İki seçenek neredeyse aynıysa bir tanesini at. Opsiyonlar birbirini tamamen kapsamalı — "hepsi işe yarar" durumu varsa bunu kullanıcıya `multiSelect: true` ile sun.

### Kapsanması Gereken Alanlar

Her kural için aşağıdaki alanları soru-cevap ile netleştir. Bağlamdan net çıkanları onay sorusuna dönüştür, çıkmayanları tam sorguya dönüştür.

#### A) Kapsam (Scope) — Nereye yazılacak?

**Eğer `--global` veya `--team` flag'i verilmişse** bu soru atlanır, kapsam bellidir. Flag verilmemişse sor:

**Soru örneği:** "Bu kural nereye yazılsın?"

**Seçenek kalıpları:**
- Bu projeye özel (`.claude/`) — varsayılan
- Tüm projelerimde geçerli (`--global` → `~/.claude/rules/`)
- Team bilgi tabanına (`--team` → agent veya team rule dosyası)

**Proje kapsamı seçilirse takip sorusu:** "Hangi uygulamayı kapsıyor?"
- `.claude/docs/coding-standards/` dizinindeki mevcut dosyaları dinamik listele
- Tüm uygulamalar (common) — `coding-common.md`
- Her mevcut `coding-standards/{app}.md` dosyası için bir seçenek

**Team kapsamı seçilirse takip sorusu:** "Hangi agent'ın bilgi tabanına eklensin?"
- Kurulu team'deki agent dosyalarını listele
- Veya team geneli rule olarak ekle

#### B) Kuralın Tek Cümlelik İfadesi (Rule)

**Soru örneği:** "Kuralın özünü en iyi ifade eden hangisi?"

**Seçenek kalıpları:** Bağlamdan 3 alternatif formülasyon türet. Her biri farklı ton/kısıtlılık sunar:
- **Katı yasak** versiyonu ("X asla yapılmaz")
- **Yönlendirici** versiyonu ("X için Y kullanılır")
- **Koşullu** versiyonu ("Ancak Y durumunda X yapılabilir")

Kullanıcı "Diğer" ile kendi cümlesini yazabilir.

#### C) Motivasyon (Why)

**Soru örneği:** "Bu kuralın birincil motivasyonu nedir?"

**Seçenek kalıpları (bağlama göre seç):**
- Geçmiş hatadan ders (hangisiyse belirt)
- Mimari tutarlılık (tek gerçek kaynak, tek entry point vb.)
- Test edilebilirlik
- Performans
- Güvenlik
- Okunabilirlik / maintainability
- Regülasyon / uyum

Birden fazla motivasyon olabiliyorsa `multiSelect: true` kullan. Ana motivasyonu ilk sıraya koy.

#### D) Apply When (Tetikleme Koşulları)

**Soru örneği:** "Bu kural hangi dosya/kod desenlerinde tetiklensin?"

**Seçenek kalıpları:** Bağlamdan 2-4 spesifik tetikleyici türet. Her seçenek **somut dosya yolu veya kod kalıbı** içermeli:

- `api/Controllers/*.cs` — controller aksiyonlarında
- `api/Services/*.cs` — servis metotlarında
- `api/Consumers/*.cs` — consumer handler'larında
- Belirli attribute/pattern (örn. `[HttpPost]`, `BackgroundService` türevleri)

Birden fazla tetikleyici birlikte seçilebilirse `multiSelect: true`.

#### E) Don't Apply When (İstisnalar) — Opsiyonel

**Soru örneği:** "Bu kuralın geçerli olmadığı durum var mı?"

**Seçenek kalıpları:**
- Hayır, istisna yok
- Evet: test kodu istisna
- Evet: legacy/generated kod istisna
- Diğer: kullanıcı kendi belirtir

Kullanıcı "Hayır" derse `Don't apply when` alanı final metne yazılmaz.

#### F) Örnekler (Examples)

**Amaç:** En az bir ✅ doğru ve bir ❌ yanlış somut örnek.

**Soru örneği:** "Hangi ✅ doğru örnek kuralı en iyi temsil eder?"

**Seçenek kalıpları:** 2-3 kısa kod snippet veya durum türet. Her biri farklı bir açıdan kuralı gösterir. Kullanıcı birini seçer veya "Diğer" ile kendisi yazar. Aynı mantık ❌ örneği için de ayrı bir soru olarak sorulur.

#### G) İlgili Kurallar (Related) — Opsiyonel

**Soru örneği:** "Bu kural mevcut bir kurala referans vermeli mi?"

**Seçenek kalıpları:** Faz 1.1'de okuduğun kurallardan benzer olanların id'lerini listele + "Yok" seçeneği. Örnek:

- `no-logic-in-bridges` (ilgili — birlikte anlam kazanıyor)
- `repository-owns-db-access` (uzaktan ilgili)
- Yok, bağımsız kural

### Tur Planlama Önerisi

- **1. Tur (temel):** Kapsam + Rule ifadesi + Motivasyon — 3 soru
- **2. Tur (davranış):** Apply when + İstisna + Örnek (✅) — 3 soru
- **3. Tur (cila):** Örnek (❌) + Related + (gerekirse) edge case — 2-3 soru

Bağlamdan net çıkan alanlarda tur daralır; belirsizlik varsa ek soru eklenir. Kullanıcı bir tur içinde aynı cevabı iki kez vermeye zorlanmamalı.

---

## Faz 3 — Dinamik Çoklu Kural Algılama

Skill **tek kural varsayımıyla** başlar. Ancak sorgulama sırasında aşağıdaki sinyallerden herhangi biri görülürse, **o anda** kullanıcıya bir ayrım sorusu sor ve ona göre akışı değiştir.

### Algılama Sinyalleri

1. **Scope sorusunda iki farklı uygulama seçiliyor ve doğaları farklı** (örn. hem API hem Worker ama kural API controller'larında ve Worker BackgroundService'lerinde farklı anlam kazanıyor).
2. **Apply when sorusunda seçilen tetikleyiciler birbiriyle ilişkisiz** iki ayrı kod katmanına işaret ediyor (örn. `api/Controllers/` + `api/Repositories/`).
3. **Rule ifadesi iki bağımsız yasağı bir araya getiriyor** ("X yapılmaz ve Y da yapılmaz" şeklinde, iki bağımsız madde).
4. **Örnek sorusunda çıkarılan senaryolar tek bir kuralla açıklanamıyor** — her senaryo farklı bir ilkeyi örnekliyor.
5. **Motivasyon multiSelect'te birbirinden bağımsız iki gerekçe** seçiliyor (örn. performans + güvenlik, ama ikisi ayrı kural olmayı hak ediyor).

### Ayrım Sorusu

Sinyallerden biri tetiklendiğinde, AskUserQuestion ile şu soruyu sor:

**Soru:** "Bu bağlam aslında iki farklı kural gibi gözüküyor. Nasıl ilerleyelim?"

**Seçenekler:**
- **(Recommended)** Ayrı iki kural olarak ekleyelim — her birini ayrı netleştireceğiz
- Tek kural olarak kalsın — Rule ifadesini iki maddeyi kapsayacak şekilde genişletelim
- Şimdilik sadece birine odaklanalım, diğerini sonra ele alalım
- Yanlış algıladım, bu aslında tek bir kural

### Karar Sonrası Akış

- **Ayrı iki kural:** Her kural için Faz 2'yi bağımsız tekrar et. Önce birinci kuralı sonlandır, sonra ikinciye geç. Soru turlarını karıştırma — her kuralın kendi cevap seti olsun.
- **Tek kural olarak kalsın:** Rule ifadesi sorusunu tekrar sor ve iki maddeyi birleştiren formülasyonlar sun.
- **Sadece birine odaklan:** Diğerini unutma; Faz 4'ün sonunda kullanıcıya "diğer kuralı da şimdi ele alalım mı?" diye ikinci turu başlatma fırsatı sun.
- **Yanlış algılama:** Normal akışa geri dön, sinyali görmezden gel.

---

## Faz 4 — Konsolidasyon ve Final Onay

### 4.1 Türkçe Kural Metni Üret

Sorgulama tamamlanınca, toplanan cevaplardan **/rule skill'inin girdisi** olacak şekilde **Türkçe doğal dilde** bir kural metni oluştur.

Bu metin `/rule` skill'inin ayrıştıracağı bütün bilgileri içermeli:
- **Scope** (hangi dosyaya gideceği belli olmalı)
- **Rule** (tek cümlelik net ifade)
- **Why** (motivasyon)
- **Apply when** (spesifik koşullar)
- **Don't apply when** (varsa)
- **Examples** (✅ ve ❌)
- **Related** (varsa)

**Örnek final metin:**

> "API projesindeki controller aksiyonlarında try-catch bloğu yazılmasın — hata yönetimi üst katmandaki global exception handler'a bırakılsın. Bu kural mimari tutarlılığı korumak ve controller'ları gerçekten ince köprüler olarak tutmak için var; try-catch servis veya global handler'ın sorumluluğudur. Uygulanacağı yerler: `api/Controllers/` altındaki tüm `.cs` dosyalarındaki controller aksiyonları. Test kodu istisna sayılır. Doğru örnek: `[HttpPost] public async Task<IActionResult> Create(CreateProductRequest req) { var result = await _productService.CreateAsync(req); return Ok(result); }` — hiçbir try-catch yok. Yanlış örnek: controller içinde `try { ... } catch (Exception ex) { return BadRequest(ex.Message); }` yazmak. İlgili kural: `no-logic-in-bridges`."

Bu metin tek bir paragraf olabilir veya gerekirse iki-üç cümleye bölünebilir — ama sıkıştırılmış `Rule:` formatına dönüştürülmez, `/rule` skill'i kendi ayrıştırmasını yapacak.

### 4.2 Kullanıcıya Göster ve Onay Al

Üretilen Türkçe metni kullanıcıya göster ve şu onay sorusunu AskUserQuestion ile sor:

**Soru:** "Bu metin kuralın son hali mi? `/rule` ile şimdi ekleyebilir miyim?"

**Seçenekler:**
- **(Recommended)** Evet, `/rule` ile ekle
- Metinde bir kısmı düzeltmem gerek — hangi kısım olduğunu söyleyeyim
- Eksik kaldığını düşündüğüm bir alan var — ek soru turu yapalım
- İptal et, şimdilik ekleme

### 4.3 /rule Skill'ini Çağırma

Kullanıcı "Evet, ekle" seçtiğinde:

- **Tek kural varsa:** `/rule <Türkçe final metin>` çağır.
- **Birden fazla kural varsa:** Her birini **sırayla** çağır. Aralarda kullanıcıya kısa bir ilerleme bildirimi ver:
  - "Birinci kural yazıldı (`{id}` — `{dosya}`). Şimdi ikinci kurala geçiyorum."
- **Her `/rule` çağrısından sonra** sonucu kullanıcıya özet olarak aktar.

**Düzeltme seçilirse:** Kullanıcı hangi kısmı düzeltmek istediğini söyler. O kısma karşılık gelen soruyu (veya yakın ilgili soruyu) AskUserQuestion ile tekrar sor, cevabı al, final metni güncelle ve 4.2'yi tekrarla.

**Ek tur seçilirse:** Kullanıcının işaret ettiği eksik alan için yeni bir soru turu yap, sonra 4.1'e dön.

**İptal seçilirse:** Skill'i temiz şekilde sonlandır. Hiçbir dosyaya yazma. Kullanıcıya "Kural yazılmadı, istediğinde `/rule-wizard` ile tekrar başlayabilirsin" de.

### 4.4 Final Özet

Yazma aşaması tamamlandığında kullanıcıya tek bir özet mesaj ver:

- Kaç kural yazıldı
- Her kuralın **id**'si, hangi **dosyaya** yazıldığı
- Varsa **related** olarak işaretlenen mevcut kurallar
- Faz 3'te "sonra ele alalım" denen ertelenmiş kural varsa hatırlat

---

## Kritik Prensipler (Özet)

1. **Bağlam zorunlu.** Skill argümansız çalışmaz — kullanıcıya bağlam ister.
2. **Her soru seçenekli.** AskUserQuestion kullanılır, asla düz metin soru sorulmaz.
3. **4 seçenek yetmezse böl.** 5+ makul seçenek varsa soruyu iki tura böl. Asla "en iyi 4"e kısıtlanma.
4. **Mevcut kuralları önce oku.** Duplikasyon ve çakışmayı erken yakalamak için zorunlu ön koşul.
5. **Hiç varsayım yapma.** Bağlamdan net çıkmayan her alan için soru sorulur. Onay sorusu bile olsa sorulur.
6. **Dinamik çoklu kural algıla.** Tek kural varsayımıyla başla ama ayrışma sinyali görünce kullanıcıya sor.
7. **Final metin Türkçe.** `/rule` skill'i İngilizce'ye çevirecek — bu skill hep Türkçe konuşur ve Türkçe üretir.
8. **Onay olmadan `/rule` çağrılmaz.** Yazma aşamasına geçmeden önce netleşmiş metin kullanıcıya gösterilir ve onay alınır.
9. **Eksik alan, hiç olmayan alandan kötüdür.** Kural formatının zorunlu alanları (Rule, Why, Apply when, Examples) final metinde eksiksiz temsil edilmelidir.
10. **Skill birden fazla kural için tekrar tekrar çalıştırılabilir.** Dinamik ayrım modu Faz 3'te seçildiyse, her kural tek tek Faz 2-4 döngüsünden geçer.
