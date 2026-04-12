---
name: rule
description: "Yeni bir kodlama/mimari kuralı ekleme. Kullanıcı Türkçe doğal dilde söyler, skill İngilizce yapılandırılmış formatta doğru dosyaya yazar. 3 kapsam: proje (varsayılan), --global, --team."
argument-hint: "[--global|--team] <Türkçe doğal dilde kural>"
---

# /rule Skill

Türkçe ifade edilen bir kuralı analiz edip doğru dosyaya İngilizce yapılandırılmış formatta yazar.

---

## Üç Kapsam

| Flag | Hedef | Ne Zaman |
|------|-------|----------|
| *(hiçbiri)* | Proje `.claude/` dizini | Bu projeye özel kurallar (varsayılan) |
| `--global` | `~/.claude/rules/` | Her projede geçerli kişisel kurallar |
| `--team` | `~/agent-teams/{team}/` dosyaları | Team repo'sundaki agent veya rule dosyaları |

**`--team` aktif team tespiti:**
- `~/.claude/agents/` altındaki symlink'lerin `readlink` sonucundan `~/agent-teams/{team-name}/` çıkarılır
- Tek team varsa otomatik kullanılır
- Birden fazla team varsa AskUserQuestion ile sorulur

---

## Akış

### 1. Kuralı Analiz Et
Kullanıcının Türkçe ifadesinden şu bilgileri çıkar:
- **Konu:** Hangi tür kural? (kodlama, mimari, naming, error handling, vb.)
- **Kapsam:** Hangi uygulama(lar)ı etkiliyor?
- **Motivasyon:** Neden bu kural? (Açıkça söylenmemişse mantıklı bir Why çıkar — emin değilsen sor.)

### 2. Hedef Dosyayı Belirle

**Proje kapsamında (varsayılan):**

Projenin `.claude/docs/coding-standards/` dizinine bak ve mevcut app dosyalarını tespit et.

| Kapsam | Dosya |
|--------|-------|
| Tüm uygulamalar için ortak | `.claude/rules/coding-common.md` |
| Belirli bir uygulama | `.claude/docs/coding-standards/{app}.md` (mevcut dosyadan seç) |

**Global kapsamda (`--global`):**

| Kapsam | Dosya |
|--------|-------|
| Genel kural | `~/.claude/rules/{konu}.md` (mevcut dosya varsa ekle, yoksa oluştur) |

**Team kapsamında (`--team`):**

Kuralın neyi ilgilendirdiğine göre hedef belirlenir:

| İlgili alan | Dosya |
|------------|-------|
| Bir agent'ın bilgi tabanı | `~/agent-teams/{team}/agents/{agent}.md` |
| Team geneli kural | `~/agent-teams/{team}/rules/{konu}.md` |

Birden fazla ama hepsi değil → kullanıcıya sor.

### 3. Mevcut Kuralları Kontrol Et
Hedef dosyayı **mutlaka oku**. Üç durum olabilir:
- **Tamamen yeni kural:** Yeni bir bölüm olarak ekle
- **Mevcut kuralı genişletme/güncelleme:** In-place güncelle, duplikasyon yaratma
- **Çakışma:** İki kural birbirine aykırı → kullanıcıya sor, varsayım yapma

### 4. Yapılandırılmış Formatta Yaz
İngilizce, **detaylı ve net** — kısa değil. Eksik kural hiç olmayan kuraldan daha tehlikeli.

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
- **Asla varsayım yapma.** Eksik bilgi varsa sor.
- **Kısa tutma — açıkla.** Atlanan detay = uygulanmayan kural.
- **Edge case'leri yakala.** `Don't apply when` ekle.
- **Örnek ver.** Hem ✅ hem ❌.
- **Benzersiz id ver.** Önce dosyayı oku, çakışma olmasın.

### 6. Yaz ve Doğrula
- Hedef dosyayı Edit ile güncelle.
- Kullanıcıya kısa özet ver: hangi dosyaya, hangi id ile yazıldı.

### 7. Team Kapsamında Git Push
`--team` ile yazılan kurallarda:
```bash
cd ~/agent-teams/{team-name}
git add -A
git commit -m "rule: {kebab-case-rule-id}"
git push
```

---

## Önemli Kurallar

1. **Dil:** Kullanıcı Türkçe söyler, skill İngilizce yazar.
2. **Eksik bilgi varsa sor.** Asla doldurma.
3. **Duplikasyon yaratma.** Önce mevcut kuralları oku.
4. **Dosya yolu doğrula.** Kapsamı yanlış belirlersen kural yanlış dosyaya gider.
5. **Format sapması yok.** Tüm zorunlu alanlar dolu olmalı: Rule, Why, Apply when, Examples.
6. **Team kapsamında otomatik git push.** Kural yazıldıktan sonra commit + push.
