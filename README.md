# Paylaşılan Veri Yolu Protokolü

Bu çalışma, birden fazla cihazın adil ve zamanlamaya uygun bir şekilde veri yoluna erişimini sağlayan bir **Paylaşılan Veri Yolu Protokolü** tasarlamayı ve doğrulamayı amaçlamaktadır. Protokol, **UPPAAL** modelleme ve formel doğrulama araçları kullanılarak geliştirilmiştir.

---

## Protokol Gereksinimleri

- **Cihazlar**: Birden fazla cihaz (ör. Cihaz A, Cihaz B, Cihaz C) veri yoluna talepte bulunabilir.
- **Güvenlik**: Karşılıklı dışlama (**mutual exclusion**) sağlanmalıdır; aynı anda sadece bir cihaz veri yolunu kullanabilir.
- **Adalet**: Her cihaz belirli bir süre içinde veri yoluna erişim sağlamalıdır, düşük öncelikli cihazlar sonsuza kadar beklememelidir.
- **Zamanlama**: Hiçbir cihazın veri yoluna erişim süresi sonsuza kadar uzamamalıdır.
- **Dinamik Öncelik**: Cihazların öncelikleri, bekleme sürelerine bağlı olarak dinamik şekilde artırılır.

---

## Protokol Tasarımı

Protokol, bir **Arbitraj Birimi (Arbiter)** yardımıyla cihazların veri yoluna erişim taleplerini yönetir. Cihazların her biri:
- **Bekleme (idle)**, 
- **Talep (requesting)**, 
- **Kullanım (using_bus)** durumlarından birinde bulunabilir.

Arbitraj birimi, cihazların statik öncelik değerlerini ve bekleme sürelerini birleştirerek **etkin öncelik** hesaplamalarını yapar. **En yüksek etkin önceliğe sahip cihaz** veri yoluna erişim hakkı kazanır.

**Etkin Öncelik Hesaplama**:

---

## Algoritma

1. Cihazların başlangıç değerlerini belirle.
2. Cihaz talepte bulunursa, `requestBus` fonksiyonunu çağır ve `requesting=true` olarak işaretle.
3. Talepte bulunan cihazların bekleme sürelerini güncelle.
4. **Etkin öncelik** değerlerini yukarıdaki formüle göre hesapla.
5. Arbitraj birimiyle en yüksek etkin önceliğe sahip cihazı seç.
6. Seçilen cihazı veri yoluna erişen cihaz (`selectedDevice`) olarak ata ve bekleme süresini sıfırla.
7. Cihazın işi bittiğinde `releaseBus` fonksiyonunu çağır.
8. Döngü boyunca tüm cihazların durumlarını ve önceliklerini güncelle.

### Sözde Kod
```python
ALPHA = 0.1  # Bekleme süresi ağırlık katsayısı

struct Device:
    staticPriority   # Başlangıç önceliği
    waitTime         # Bekleme süresi
    requesting       # Talep durumu
    usingBus         # Kullanım durumu
    effectivePriority  # Hesaplanan etkin öncelik

function arbitrate(devices[]):
    while true:
        foreach device in devices:
            if device.requesting AND NOT device.usingBus:
                device.waitTime += 1
                device.effectivePriority = device.staticPriority + (device.waitTime * ALPHA)

        selectedDevice = null
        maxPriority = -1
        foreach device in devices:
            if device.requesting AND NOT device.usingBus AND device.effectivePriority > maxPriority:
                selectedDevice = device
                maxPriority = device.effectivePriority

        if selectedDevice != null:
            selectedDevice.usingBus = true
            selectedDevice.waitTime = 0

function requestBus(device):
    device.requesting = true
    device.effectivePriority = device.staticPriority

function releaseBus(device):
    device.usingBus = false
    device.requesting = false
    device.waitTime = 0
# UPPAAL Modeling: Shared Bus Protocol
```

---

## UPPAAL Outputs
<img width="1680" alt="Ekran Resmi 2025-01-08 19 21 30" src="https://github.com/user-attachments/assets/a3c6fb64-6481-4e07-9d73-d1d5194fac4b" />
<img width="1680" alt="Ekran Resmi 2025-01-08 19 21 45" src="https://github.com/user-attachments/assets/21db1e43-ee4e-42f3-85f0-7169c94a4fa0" />
<img width="1680" alt="Ekran Resmi 2025-01-08 19 21 56" src="https://github.com/user-attachments/assets/c63e9d95-77a5-4ae8-99d4-e4d3388507a4" />
<img width="1680" alt="Ekran Resmi 2025-01-08 19 22 14" src="https://github.com/user-attachments/assets/6241f5a8-51be-4235-a00f-8386328f781a" />

### **Verification Queries**


The following queries were used to verify fair access to the bus and safe usage of resources:

- `A[] (d2.USING_BUS imply d2.x ≤ MAX_USAGE)`
- `A[] (d2.REQUESTING imply d2.x ≤ MAX_WAIT)`
- `A[] (d1.REQUESTING imply d1.x ≤ MAX_WAIT)`
- `A[] (d0.REQUESTING imply d0.x ≤ MAX_WAIT)`
- `d2.REQUESTING → d2.USING_BUS`
- `d1.REQUESTING → d1.USING_BUS`
- `d0.REQUESTING → d0.USING_BUS`
- `A[] not ((d0.USING_BUS && d1.USING_BUS) or (d1.USING_BUS && d2.USING_BUS) or (d0.USING_BUS && d2.USING_BUS))`
- `A[] not deadlock`

### **Simulation Results**

The system design confirms:
- Deadlock will never occur.
- All devices can access the shared b

<img width="1680" alt="Ekran Resmi 2025-01-08 19 23 14" src="https://github.com/user-attachments/assets/f64e0f0f-f392-4ec0-bd7c-6caf77ffc7ba" />
<img width="1680" alt="Ekran Resmi 2025-01-08 19 23 24" src="https://github.com/user-attachments/assets/024dd1f6-d7f4-4237-abda-9bd205dc2668" />
<img width="1680" alt="Ekran Resmi 2025-01-08 19 23 43" src="https://github.com/user-attachments/assets/8458aabb-fcfa-445c-96df-3f3eee71b3ec" />


