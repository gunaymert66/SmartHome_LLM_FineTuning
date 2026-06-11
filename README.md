# 🏠 Home LLM — Türkçe Sesli Komut ile Home Assistant Kontrolü

Türkçe doğal dil komutlarını Home Assistant JSON formatına çeviren, **Gemma-2B-IT** ve **LLaMA 3.2-3B-Instruct** tabanlı fine-tuned dil modelleri.

---

## 📌 Proje Özeti

Kullanıcıların Türkçe olarak söyledikleri ev otomasyonu komutlarını ("ışığı aç", "klimayı kapat", "perdeyi indir" vb.) doğrudan Home Assistant'ın anlayacağı JSON servis çağrılarına dönüştüren bir LLM modeli eğitilmiştir.

**Örnek:**
```
Girdi : "odadaki ışığı aç"
Çıktı : {"service": "light.turn_on", "entity_id": "light.living_room"}
```

---

## 🗂️ Proje Yapısı

```
AI.ipynb                    # Ana notebook (veri üretimi + fine-tuning + değerlendirme)
TR_Commands.xlsx            # (Opsiyonel) Temel Türkçe komut listesi
train_dataset.jsonl         # Eğitim veri seti (otomatik üretilir)
test_dataset.jsonl          # Test veri seti (otomatik üretilir)
home_llm_gemma_lora/        # Eğitilmiş Gemma LoRA ağırlıkları (çıktı)
home_llm_llama_lora/        # Eğitilmiş LLaMA LoRA ağırlıkları (çıktı)
gemma_results.csv           # Gemma değerlendirme sonuçları
llama_results.csv           # LLaMA fine-tuned değerlendirme sonuçları
llm_results.csv             # Groq baseline sonuçları
final_results.csv           # Tüm modellerin birleştirilmiş sonuçları
results_chart.png           # Accuracy & F1 karşılaştırma grafiği
latency_chart.png           # Latency karşılaştırma grafiği
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

- **Fine-tuned Modeller:** `google/gemma-2b-it` ve `meta-llama/Llama-3.2-3B-Instruct`
- **Baseline LLM:** `llama-3.1-8b-instant` (Groq API, fine-tune yapılmamış)
- **Fine-tuning:** LoRA (PEFT) — `r=16`, `lora_alpha=32`
- **Quantization:** 4-bit NF4 (BitsAndBytes)
- **Trainer:** HuggingFace `SFTTrainer` (TRL)
- **Platform:** Google Colab (GPU, L4)
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
pip install transformers==4.46.1 trl==0.11.4 peft bitsandbytes accelerate datasets pandas openpyxl
```

### 2. HuggingFace & Groq Token

Google Colab'da `Secrets` bölümüne şunları ekleyin:
- `HF_TOKEN` — Gemma ve LLaMA modelleri için HuggingFace erişim tokenı
- `GROQ_API_KEY` — Groq API üzerinden baseline LLM testi için

### 3. Notebook'u Çalıştırın

`AI.ipynb` dosyasını Google Colab'da sırayla çalıştırın:

| Hücre | İşlem |
|---|---|
| 1 | Veri setini üret → `train_dataset.jsonl` ve `test_dataset.jsonl` oluştur |
| 2 | Dosyaları Google Drive'a kaydet |
| 3 | Gemma-2B-IT modelini yükle, LoRA fine-tune et, kaydet |
| 4 | LLaMA 3.2-3B-Instruct modelini yükle, LoRA fine-tune et, kaydet |
| 5 | LLaMA fine-tuned modelini değerlendir |
| 6 | Gemma fine-tuned modelini değerlendir |
| 7 | Groq API ile LLaMA-3.1-8B baseline'ı değerlendir (fine-tune yok) |
| 8 | Tüm sonuçları birleştir, `final_results.csv` oluştur, grafikler çiz |

---

## 📈 Eğitim Sonuçları

### Gemma-2B-IT (Eğitim süresi: ~5 dk 41 sn | 363 adım)

| Epoch | Train Loss | Val Loss |
|---|---|---|
| 0 | 0.2391 | 0.2246 |
| 1 | 0.1848 | 0.1900 |
| 2 | 0.1717 | 0.1777 |

E�itilebilir parametre: **3,686,400 / 2,509,858,816** (%0.15)

### LLaMA 3.2-3B-Instruct (Eğitim süresi: ~16 dk 35 sn | 363 adım)

| Epoch | Train Loss | Val Loss |
|---|---|---|
| 0 | 0.4545 | 0.4008 |
| 1 | 0.2083 | 0.2209 |
| 2 | 0.2000 | 0.2081 |

E�itilebilir parametre: **9,175,040 / 3,221,924,864** (%0.28)

---

## 🧪 Değerlendirme Sonuçları

Test seti: **478 örnek** | Metrikler: Accuracy, Precision, Recall, F1, Valid JSON Rate, Avg Latency

### LLaMA 3.2 (Fine-tuned)

| Prompt Türü | Accuracy | Precision | Recall | F1 | JSON Rate | Latency |
|---|---|---|---|---|---|---|
| Zero-shot | **0.9686** | 0.9958 | 0.9686 | **0.9798** | 0.9770 | 1.070s |
| Few-shot | 0.9289 | 0.9502 | 0.9289 | 0.9216 | **1.0000** | 1.072s |
| Role-play | **0.9728** | 0.9912 | 0.9728 | **0.9813** | 0.9854 | 1.064s |

### Gemma-2B (Fine-tuned)

| Prompt Türü | Accuracy | Precision | Recall | F1 | JSON Rate | Latency |
|---|---|---|---|---|---|---|
| Zero-shot | 0.7531 | 0.9910 | 0.7531 | 0.8464 | 0.7594 | 2.205s |
| Few-shot | 0.8891 | 0.9565 | 0.8891 | 0.9086 | **1.0000** | 0.957s |
| Role-play | 0.8954 | 0.9880 | 0.8954 | 0.9382 | 0.9059 | 2.098s |

### Groq — LLaMA-3.1-8B (Fine-tune YOK, Baseline)

> Test seti: **150 örnek** (kısaltılmış), Groq API üzerinden

| Prompt Türü | Accuracy | Precision | Recall | F1 | JSON Rate | Latency | Avg Tokens |
|---|---|---|---|---|---|---|---|
| Zero-shot | 0.4267 | 0.6075 | 0.4267 | 0.4372 | 0.9867 | 1.780s | 117.8 |
| Few-shot | **0.6333** | 0.7492 | 0.6333 | **0.6472** | 0.9933 | 2.214s | 194.8 |
| Role-play | 0.4133 | 0.5725 | 0.4133 | 0.4051 | 0.9800 | 2.166s | 138.7 |

### 🏆 Genel Model Karşılaştırması

| Model | En İyi Prompt | Accuracy | F1 |
|---|---|---|---|
| **LLaMA 3.2 (Fine-tuned)** | Role-play | **%97.28** | **0.9813** |
| **LLaMA 3.2 (Fine-tuned)** | Zero-shot | %96.86 | 0.9798 |
| Gemma-2B (Fine-tuned) | Role-play | %89.54 | 0.9382 |
| Gemma-2B (Fine-tuned) | Few-shot | %88.91 | 0.9086 |
| LLaMA-3.1-8B (Groq, baseline) | Few-shot | %63.33 | 0.6472 |

Fine-tuning'in etkisi açık şekilde görülmektedir: Fine-tune edilmemiş LLaMA-3.1-8B (8B parametre, bulut API) en iyi senaryoda %63 accuracy'ye ulaşırken, fine-tune edilmiş LLaMA-3.2-3B (3B parametre, yerel) %97'ye ulaşmıştır.

---

## 💡 Model Çıktısı Kullanımı

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
from peft import PeftModel
import torch

base_model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.2-3B-Instruct",
    torch_dtype=torch.bfloat16,
    device_map="auto"
)
model = PeftModel.from_pretrained(base_model, "home_llm_llama_lora")
tokenizer = AutoTokenizer.from_pretrained("home_llm_llama_lora")

prompt = "Komut: ışığı aç"
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")
outputs = model.generate(**inputs, max_new_tokens=60)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
# {"service": "light.turn_on", "entity_id": "light.living_room"}
```

---

## 📝 Notlar

- `TR_Commands.xlsx` bulunamazsa sistem otomatik olarak yedek komutlarla devam eder.
- Daha fazla intent veya şablon eklenerek veri seti genişletilebilir.
- Model çıktısı doğrudan Home Assistant REST API veya WebSocket arayüzüne gönderilebilir.
- Gemma modeli few-shot ve role-play prompt'larında zero-shot'a göre belirgin şekilde iyileşmektedir.

---

## 📄 Lisans

Bu proje eğitim amaçlıdır. Gemma modeli [Google Gemma Terms](https://ai.google.dev/gemma/terms), LLaMA modeli ise [Meta LLaMA License](https://llama.meta.com/llama-downloads/) koşullarına tabidir.
