---
title: "Kümeler ve Fonksiyonlar"
---

# 🧬 Kümeler ve Fonksiyonlar

Matematiğin gruplandırma (küme) ve eşleme (fonksiyon) mantığı.

---

## 1. Kümeler

Kümeler, iyi tanımlanmış nesneler topluluğudur. Kümedeki eleman sayısı $s(A)$ ile gösterilir.

### Kümelerde İşlemler
*   **Kesişim ($\cap$):** Her iki kümede de **ortak** olan elemanların kümesidir.
*   **Birleşim ($\cup$):** Her iki kümedeki **tüm** elemanların bir araya gelmesiyle oluşan kümedir.
    *   **Birleşim Formülü (Hayati):**
        $$s(A \cup B) = s(A) + s(B) - s(A \cap B)$$
*   **Fark ($\setminus$ veya $-$):** Bir kümede olup diğerinde olmayan elemanlar.
    *   $A \setminus B$: A'da olan ama B'de olmayanlar.
*   **Tümleyen ($A'$):** Evrensel kümede olup A kümesinde olmayan elemanlar.

---

## 2. Fonksiyonlar

Bir tanım kümesindeki her elemanı, değer kümesindeki yalnız bir elemana eşleyen kuraldır. $f(x)$ şeklinde gösterilir.

### Değer Bulma Mantığı
Fonksiyonda parantez içindeki ifadeye göre bilinmeyen yerine sayı yazılır.

*   *Örnek:* $f(x) = 3x - 2$ ise $f(4)$ kaçtır?
    *   $x$ yerine $4$ yazalım:
    *   $f(4) = 3 \cdot 4 - 2 = 12 - 2 = 10$

*   *Örnek:* $f(x + 1) = 2x + 5$ ise $f(3)$ kaçtır?
    *   İçerisinin $3$ olmasını istiyoruz: $x + 1 = 3 \Rightarrow x = 2$ yazmalıyız.
    *   $x$ yerine $2$ yazalım:
    *   $f(3) = 2 \cdot 2 + 5 = 9$

### Özel Fonksiyonlar
*   **Birim (Etkisiz) Fonksiyon:** İçi dışı bir fonksiyondur. Ne girerse o çıkar.
    *   $f(x) = x$ veya $f(2x + 3) = 2x + 3$
*   **Sabit Fonksiyon:** Girdi ne olursa olsun sonuç hep aynı sayıdır, içinde $x$ bulunmaz.
    *   $f(x) = c$ (örn: $f(x) = 5 \Rightarrow f(100) = 5$)
