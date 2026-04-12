# Agent Yapılandırma Kuralları

## Children Pattern (Zorunlu)

Her agent şu yapıda organize edilir:

```
~/.claude/agents/{agent-name}/
├── agent.md              ← Kimlik, sorumluluk alanı, temel prensipler (kısa, gömülü)
└── children/             ← Detaylı bilgi, pattern'ler, stratejiler (her konu ayrı dosya)
    ├── konu-1.md
    ├── konu-2.md
    └── ...
```

### Kurallar

1. **agent.md kısa kalır.** Sadece: kimlik, sorumluluk alanı (pozitif liste), temel prensipler (değişmeyen, kısa maddeler), "children/ oku" talimatı.
2. **Detaylı her şey children/ altında.** Stratejiler, pattern'ler, workflow'lar, convention'lar — her biri ayrı .md dosyası.
3. **Yeni konu = yeni dosya.** agent.md'ye dokunmadan children/ altına .md at. Agent "children/ altındaki tüm .md dosyalarını oku" talimatıyla otomatik keşfeder.
4. **Güncelleme = tek dosya.** Bir konuyu güncellemek için sadece ilgili children dosyasına dokunulur.
5. **Monolitik agent dosyası yasak.** Tüm bilgiyi tek .md'ye yığmak yasak — yönetilemez hale gelir.
6. **Bu pattern tüm agent'lar için geçerlidir.** API, Socket, Worker, Flutter, React, Mail, Log, Infra — hepsi aynı yapıda.
