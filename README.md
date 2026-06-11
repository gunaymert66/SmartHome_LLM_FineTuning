# 🏠 Home LLM — Türkçe Sesli Komut ile Home Assistant Kontrolü

Türkçe doğal dil komutlarını Home Assistant JSON formatına çeviren, **Gemma-2B-IT** tabanlı fine-tuned bir dil modeli.

---

## 📌 Proje Özeti

Bu proje, kullanıcıların Türkçe olarak söyledikleri ev otomasyonu komutlarını ("ışığı aç", "klimayı kapat", "perdeyi indir" vb.) doğrudan Home Assistant'ın anlayacağı JSON servis çağrılarına dönüştüren bir LLM modeli eğitmektedir.

**Örnek:**
```
Girdi : "odadaki ışığı aç"
Çıktı : {"service": "light.turn_on", "entity_id": "light.living_room"}
```

---

## 🗂️ Proje Yapısı

```
AI.ipynb                    # Ana notebook (veri üretimi + fine-tuning)
TR_Commands.xlsx            # (Opsiyonel) Temel Türkçe komut listesi
train_dataset.jsonl         # Eğitim veri seti (otomatik üretilir)
test_dataset.jsonl          # Test veri seti (otomatik üretilir)
home_llm_gemma_lora/        # Eğitilmiş LoRA ağırlıkları (çıktı)
```

---

## 🎯 Desteklenen Intent'ler (Komut Kategorileri)

| Intent | Açıklama |
|---|---|
| `light.turn_on` | Işık aç |
| `light.turn_off` | Işık kapat |
| `light.set_brightness` | Parlaklık ayarla |
| `climate.turn_on` | Klima / ısıtma aç |
| `climate.turn_off` | Klima / ısıtma kapat |
| `climate.set_temperature` | Sıcaklık ayarla |
| `media_player.turn_on` | TV / medya oynatıcı aç |
| `media_player.turn_off` | TV / medya oynatıcı kapat |
| `media_player.media_pause` | Müzik / video duraklat |
| `cover.open_cover` | Panjur / perde aç |
| `cover.close_cover` | Panjur / perde kapat |
| `scene.turn_on` | Sahne / senaryo başlat |

---

## 🔧 Kullanılan Teknolojiler

- **Model:** `google/gemma-2b-it`
- **Fine-tuning:** LoRA (PEFT) — `r=16`, `lora_alpha=32`
- **Quantization:** 4-bit NF4 (BitsAndBytes)
- **Trainer:** HuggingFace `SFTTrainer` (TRL)
- **Platform:** Google Colab (GPU)
- **Deney Takibi:** Weights & Biases (offline mod)

---

## 📊 Veri Seti

Veri seti tamamen **şablon tabanlı sentetik üretim** ile oluşturulmuştur.

- Her intent için anahtar kelimeler × eylemler × cümle şablonları çapraz çarpılarak komutlar üretilir.
- Opsiyonel olarak `TR_Commands.xlsx` dosyasından da temel komutlar okunabilir.
- **Toplam:** ~2387 benzersiz örnek
- **Bölünme:** %80 eğitim (1909) / %20 test (478), stratified

---

## 🚀 Kurulum ve Çalıştırma

### 1. Gereksinimler

```bash
pip install transformers peft trl bitsandbytes datasets accelerate pandas openpyxl
```

### 2. HuggingFace Token

Google Colab'da `Secrets` bölümüne `HF_TOKEN` adıyla HuggingFace erişim tokenınızı ekleyin (Gemma modeli için gereklidir).

### 3. Notebook'u Çalıştırın

`AI.ipynb` dosyasını Google Colab'da sırayla çalıştırın:

| Hücre | İşlem |
|---|---|
| 1 | Veri setini üret, `train_dataset.jsonl` ve `test_dataset.jsonl` oluştur |
| 2 | Dosyaları Google Drive'a kaydet |
| 3 | Gemma-2B-IT modelini yükle, LoRA ile fine-tune et, modeli kaydet |

### 4. Eğitim Parametreleri

```python
num_train_epochs        = 3
learning_rate           = 2e-4
per_device_batch_size   = 4
gradient_accumulation   = 4
max_seq_length          = 256
optimizer               = paged_adamw_32bit
```

---

## 💡 Model Çıktısı Kullanımı

Eğitim bittikten sonra model `home_llm_gemma_lora/` klasörüne kaydedilir. Yükleyip kullanmak için:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained("google/gemma-2b-it", ...)
model = PeftModel.from_pretrained(base_model, "home_llm_gemma_lora")
tokenizer = AutoTokenizer.from_pretrained("home_llm_gemma_lora")

prompt = "Aşağıdaki komutu Home Assistant JSON formatına çevir.\nKomut: ışığı aç\nÇıktı:"
inputs = tokenizer(prompt, return_tensors="pt")
outputs = model.generate(**inputs, max_new_tokens=50)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

---

## 📈 Eğitim Sonuçları

| Epoch | Train Loss | Val Loss |
|---|---|---|
| 1 | 0.2391 | — |
| 2 | — | — |
| 3 | — | — |

> Eğitim yaklaşık **5 dakika 41 saniye** sürmüştür (Google Colab GPU, 363 adım).
> Eğitilebilir parametre oranı: **%0.15** (3.6M / 2.5B)

---

## 📝 Notlar

- `TR_Commands.xlsx` dosyası proje dizininde bulunmuyorsa, sistem otomatik olarak yerleşik yedek komutlarla devam eder.
- Daha fazla intent veya daha zengin şablonlar eklenerek veri seti genişletilebilir.
- Model çıktısı doğrudan Home Assistant REST API veya WebSocket arayüzüne gönderilebilir.

---

## 📄 Lisans

Bu proje eğitim amaçlıdır. Gemma modeli [Google'ın kullanım koşullarına](https://ai.google.dev/gemma/terms) tabidir.
