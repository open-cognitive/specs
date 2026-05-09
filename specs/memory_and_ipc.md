# Bellek Yönetimi ve Çekirdekler Arası İletişim (IPC)

## Sorun Tanımı
`neural-engine` (Sistem 1) gigabaytlarca tensor verisi üretir. Bu verinin `logic-gate-core` (Sistem 2) tarafına ağ üzerinden (REST, gRPC) veya bellekte kopyalanarak gönderilmesi kabul edilemez bir performans kaybıdır (Darboğaz/Bottleneck).

## Çözüm: Zero-Copy Shared Memory (Sıfır Kopya Paylaşımlı Bellek)
Open-Cognitive OS, bileşenler arası veri aktarımı için `repr(C)` standartlarında, bellek eşlemeli (memory-mapped) sistemler kullanır.

### Protokol Kuralları
1. **Veri Yapıları:** Her iki repo da `protocol` reposunda tanımlanan aynı Rust `struct` yapılarını referans alır.
2. **Memory Map (mmap):** Ağırlıklar (Weights) ve Token Logit'leri diske veya RAM'e bir kez yazılır. `neural-engine` yazıcı (writer), `logic-gate-core` ise okuyucu (reader) pointer'lara sahiptir. Veri asla kopyalanmaz, sadece bellek adresi (pointer offset) transfer edilir.
3. **FlatBuffers Uyumluluğu:** Çok dilli destek (örneğin WASM tool'larının veri okuması) gerektiğinde, serileştirme yükü olmayan (zero-parsing) FlatBuffers mantığı kullanılacaktır.

### Temel Veri Tipleri (Taslak)
- `TensorContext`: İşlenmekte olan tensorün bellek başlangıç adresi ve boyutlarını tutar.
- `CognitiveSignal`: Mantık kapısından Nöral motora giden "ileri yayılım yap" (forward pass) sinyali.
