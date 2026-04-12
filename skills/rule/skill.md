---
name: rule
description: Yeni bir kodlama/mimari kuralı ekleme. Kullanıcı Türkçe doğal dilde söyler, skill İngilizce yapılandırılmış formatta doğru dosyaya yazar.
argument-hint: "<Türkçe doğal dilde kural>"
---

# /rule Skill

Türkçe ifade edilen bir kuralı analiz edip doğru `coding-standards` veya `coding-common` dosyasına İngilizce yapılandırılmış formatta yazar.

> Sistem dökümantasyonu: [.claude/docs/coding-rules-system.md](../../docs/coding-rules-system.md)

---

## Akış

### 1. Kuralı Analiz Et
Kullanıcının Türkçe ifadesinden şu bilgileri çıkar:
- **Konu:** Hangi tür kural? (kodlama, mimari, naming, error handling, vb.)
- **Kapsam:** Hangi uygulama(lar)ı etkiliyor?
- **Motivasyon:** Neden bu kural? (Açıkça söylenmemişse mantıklı bir Why çıkar — emin değilsen sor.)

### 2. Hedef Dosyayı Belirle

Önce projenin `.claude/docs/coding-standards/` dizinine bak ve hangi app-specific dosyaların mevcut olduğunu tespit et. Ardından kapsamı belirle:

| Kapsam | Dosya |
|--------|-------|
| Tüm uygulamalar için ortak | `.claude/rules/coding-common.md` |
| Belirli bir uygulama | `.claude/docs/coding-standards/{app}.md` (mevcut dosyadan seç) |

Her projede farklı app dosyaları olabilir (örn: api.md, flutter.md, app.md, socket.md, worker.md). Dosya listesini dinamik olarak oku.

Birden fazla ama hepsi değil → kullanıcıya sor: "common'a mı yazayım yoksa ilgili dosyaların her birine ayrı ayrı mı?"

### 3. Mevcut Kuralları Kontrol Et
Hedef dosyayı (ve gerekiyorsa diğer ilgili dosyaları) **mutlaka oku**. Üç durum olabilir:
- **Tamamen yeni kural:** Yeni bir bölüm olarak ekle
- **Mevcut kuralı genişletme/güncelleme:** Eski kuralı in-place güncelle, duplikasyon yaratma
- **Çakışma:** İki kural birbirine aykırı → kullanıcıya sor, varsayım yapma

### 4. Yapılandırılmış Formatta Yaz
İngilizce, **detaylı ve net** — kısa değil. v2 dersi: eksik kural hiç olmayan kuraldan daha tehlikeli.

```markdown
### {kebab-case-rule-id}
**Rule:** {Tek cümlede kuralın net ifadesi}

**Why:** {Motivasyon. Hangi sorunu önlüyor? Hangi prensibi destekliyor?
Varsa geçmiş hatadan ders. Bu alan boş veya muğlak bırakılmaz.}

**Apply when:** {Hangi durumlarda geçerli — dosya yolları, kod desenleri,
ne tür değişiklikler? Spesifik ol.}

**Don't apply when:** {(Opsiyonel) İstisnalar varsa açıkça yaz.}

**Examples:**
- ✅ Correct: {kod örneği veya somut senaryo}
- ❌ Wrong: {kod örneği veya somut senaryo}

**Related:** {(Opsiyonel) İlgili kural id'leri}
```

### 5. Yazma Kuralları (KRİTİK)
- **Asla varsayım yapma.** Why, Apply when, Examples bölümlerinden herhangi biri kullanıcının ifadesinden net çıkmıyorsa **sor**. "Şu anlama mı geliyor / örnek olarak X mi olur?" gibi spesifik sor.
- **Kısa tutma — açıkla.** Şüphe varsa daha çok yaz. Atlanan detay = uygulanmayan kural.
- **Edge case'leri yakala.** Kuralın geçerli olmadığı durumlar varsa `Don't apply when` ekle.
- **Örnek vermekten çekinme.** Soyut kural → somut örnek. Hem ✅ hem ❌ ver.
- **Benzersiz id ver.** `kebab-case`, mevcut id'lerle çakışmamalı (önce dosyayı oku).

### 6. Yaz ve Doğrula
- Hedef dosyayı Edit ile güncelle (yeni kural ekleme veya mevcut kuralı güncelleme).
- Kullanıcıya **kısa özet** ver: hangi dosyaya, hangi id ile yazıldı, kısa Türkçe açıklama.

---

## Önemli Kurallar

1. **Dil:** Kullanıcı Türkçe söyler, skill İngilizce yazar. Teknik netlik + AI tutarlılığı için.
2. **Eksik bilgi varsa sor.** Asla doldurma. Her zorunlu alan kullanıcı niyetini yansıtmalı.
3. **Duplikasyon yaratma.** Önce mevcut kuralları oku.
4. **Dosya yolu doğrula.** Kapsamı yanlış belirlersen kural yanlış dosyaya gider ve hook tetiklenmez.
5. **Format sapması yok.** Şablondaki tüm zorunlu alanlar dolu olmalı: Rule, Why, Apply when, Examples.
