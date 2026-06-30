---
title: "Bölme, Bölünebilme & EBOB-EKOK"
---

# ➗ Bölme, Bölünebilme & EBOB-EKOK

Sayıları bölerken kalan bulma kuralları ve ortak kat/bölen hesapları.

---

## 1. Bölme Formülü

Bir $A$ sayısını $B$ sayısına böldüğümüzde bölüm $C$ ve kalan $K$ ise:
1.  **Ana Formül:** $A = B \cdot C + K$
2.  **Kalan Kuralı:** Kalan her zaman bölenden küçük ve sıfıra eşit veya büyük olmalıdır: $0 \le K < B$
    *   *Kalan 0 ise sayı tam bölünüyor demektir.*

---

## 2. Bölünebilme Kuralları

*   **2 ile Bölünebilme:** Son basamak çift (0, 2, 4, 6, 8) olmalı.
*   **3 ile Bölünebilme:** Rakamlar toplamı 3 veya 3'ün katı olmalı.
*   **4 ile Bölünebilme:** Son iki basamak 4'ün katı (veya 00) olmalı.
*   **5 ile Bölünebilme:** Son basamak 0 veya 5 olmalı.
*   **8 ile Bölünebilme:** Son üç basamak 8'in katı olmalı.
*   **9 ile Bölünebilme:** Rakamlar toplamı 9 veya 9'un katı olmalı.
*   **10 ile Bölünebilme:** Son basamak 0 olmalı.

> [!TIP]
> **Birleşik Bölme Kuralları:**
> Bir sayı aralarında asal iki sayıya bölünüyorsa, bunların çarpımına da bölünür.
> *   **6 ile bölünebilme:** Hem 2 hem 3'e bölünmeli.
> *   **12 ile bölünebilme:** Hem 3 hem 4'e bölünmeli.
> *   **15 ile bölünebilme:** Hem 3 hem 5'e bölünmeli.
> *   **36 ile bölünebilme:** Hem 4 hem 9'a bölünmeli.

---

## 3. Asal Sayılar

Yalnızca 1'e ve kendisine tam bölünebilen, 1'den büyük sayılardır.
*   Asal Sayılar: $\{2, 3, 5, 7, 11, 13, 17, 19, 23, \dots\}$
*   **Dikkat:** 2 en küçük ve **tek çift** asal sayıdır. Negatif asal sayı yoktur.

---

## 4. EBOB ve EKOK

*   **EBOB (En Büyük Ortak Bölen):** İki veya daha fazla sayıyı aynı anda bölen en büyük sayıdır.
*   **EKOK (En Küçük Ortak Kat):** İki veya daha fazla sayının en küçük ortak katı olan sayıdır.

### 📐 Altın Kural
İki sayı olan $a$ ve $b$ için:
$$a \cdot b = \text{EBOB}(a, b) \cdot \text{EKOK}(a, b)$$
*(Yani iki sayının çarpımı, EBOB ve EKOK'larının çarpımına eşittir.)*
