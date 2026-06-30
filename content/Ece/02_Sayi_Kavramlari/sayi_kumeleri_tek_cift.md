---
title: "Sayı Kümeleri, Tek/Çift & Ardışık Sayılar"
---

# 🔢 Sayı Kümeleri, Tek/Çift & Ardışık Sayılar

Sayıların gruplarını tanımak ve temel sayı özelliklerini bilmek KPSS sorularının ilk basamağıdır.

---

## 1. Sayı Kümeleri

*   **Rakamlar:** $\{0, 1, 2, 3, 4, 5, 6, 7, 8, 9\}$ (Sayıları yazmaya yarayan semboller).
*   **Doğal Sayılar ($\mathbb{N}$):** $\{0, 1, 2, 3, \dots\}$ *(Dikkat: 0 bir doğal sayıdır!)*
*   **Sayma Sayıları ($\mathbb{N}^+$):** $\{1, 2, 3, \dots\}$
*   **Tam Sayılar ($\mathbb{Z}$):** $\{\dots, -2, -1, 0, 1, 2, \dots\}$
    *   Negatif Tam Sayılar ($\mathbb{Z}^-$): $\{-1, -2, -3, \dots\}$
    *   Pozitif Tam Sayılar ($\mathbb{Z}^+$): $\{1, 2, 3, \dots\}$
*   **Rasyonel Sayılar ($\mathbb{Q}$):** Kesirli yazılabilen sayılar. (örn: $\frac{2}{3}, -5, 0$)
*   **Reel (Gerçek) Sayılar ($\mathbb{R}$):** Sayı doğrusundaki tüm sayılar.

---

## 2. Tek ve Çift Sayılar

Son basamağı 1, 3, 5, 7, 9 olanlar **Tek ($T$)**, 0, 2, 4, 6, 8 olanlar **Çift ($Ç$)** sayılardır.

### Toplama ve Çıkarma Kuralları
*   $T \pm T = Ç$ *(3 + 5 = 8)*
*   $Ç \pm Ç = Ç$ *(4 + 2 = 6)*
*   $T \pm Ç = T$ *(5 - 2 = 3)*

### Çarpma Kuralları
*   $T \cdot T = T$ *(3 \cdot 5 = 15)*
*   $Ç \cdot T = Ç$ *(4 \cdot 3 = 12)*
*   $Ç \cdot Ç = Ç$ *(2 \cdot 4 = 8)*
*   **Altın Kural:** Bir çarpma işleminin sonucunun **Çift** olması için çarpanlardan *en az birinin çift olması* yeterlidir.
*   **Üs Kuralı:** Pozitif tam sayı olan $n$ için: $T^n = T$ ve $Ç^n = Ç$'dir. (örn: $3^2 = 9$, $2^3 = 8$)

---

## 3. Ardışık Sayılar ve Toplam Formülleri

Belirli bir kurala göre ardarda gelen sayılardır. (örn: 3, 5, 7, 9...)

### 📐 Hayati İki Formül
1.  **Terim Sayısı (Kaç tane sayı var?):**
    $$\text{Terim Sayısı} = \frac{\text{Son Terim} - \text{İlk Terim}}{\text{Artış Miktarı}} + 1$$
2.  **Terimler Toplamı (Sayıların toplamı kaç?):**
    $$\text{Toplam} = \text{Terim Sayısı} \cdot \left(\frac{\text{Son Terim} + \text{İlk Terim}}{2}\right)$$

*   **Gauss Toplamı (1'den n'e kadar olan sayıların toplamı):**
    $$1 + 2 + 3 + \dots + n = \frac{n \cdot (n+1)}{2}$$
    *   *Örnek:* $1 + 2 + \dots + 10 = \frac{10 \cdot 11}{2} = 55$

---

## 4. Faktöriyel ($!$)

1'den başlayarak o sayıya kadar olan tüm ardışık doğal sayıların çarpımıdır.
*   $0! = 1$ *(Önemli İstisna)*
*   $1! = 1$
*   $2! = 2 \cdot 1 = 2$
*   $3! = 3 \cdot 2 \cdot 1 = 6$
*   $4! = 4 \cdot 3 \cdot 2 \cdot 1 = 24$
*   $5! = 5 \cdot 4! = 120$
*   **Pratik Sadeleştirme:** Büyük faktöriyel küçüğe benzetilip durdurulabilir.
    *   $\frac{6!}{4!} = \frac{6 \cdot 5 \cdot 4!}{4!} = 6 \cdot 5 = 30$
