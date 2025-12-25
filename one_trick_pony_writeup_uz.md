# One Trick Pony — Crypto Writeup (Uzbek)

> **Eslatma:** skrinshotlarni (rasmlar) o‘zingiz keyin qo‘shasiz — men joyini `TODO` qilib qoldirdim.

## Challenge ma’lumotlari

- **Nomi:** One Trick Pony  
- **Kategoriya:** Crypto  
- **Ulanish:** `nc 154.57.164.65 32158`  
- **Berilgan:** `server.py`

## Vazifa

Server “Snowglobe Cipher Booth” ichida:

1) Biz xohlagan `msg` uchun “imzo” (`signature`) beradi  
2) Keyin “snow-otp” ni to‘g‘ri topa olsak — haqiqiy `flag` (“Starshard”) beradi

Muammo: OTP server tomonda *PRNG* orqali yaratiladi, lekin “randomness” aslida sinadi.

---

## 1. Kod tahlili

### 1.1. RNG qanday ishlaydi?

`TinselRNG(bits)`:

- `holly_prime = getPrime(34)` (34-bit prime)  
- `sleigh_seed` 1..p-1 oralig‘ida tasodifiy
- Har bit uchun:
  ```py
  shimmer = pow(seed, (p-1)//2, p)
  bit = int(shimmer == 1)
  seed = (seed + 1) % p   # 0 bo‘lsa 1 ga o‘tkazadi
  ```
Bu **Legendre belgisi** (quadratic residue testi):  
`a^((p-1)/2) mod p == 1` bo‘lsa `a` kvadrat qoldiq, aks holda `-1` (mod p).

Demak, generatorning chiqishi:
\[
b_i = \left(\frac{seed_i}{p}\right) \in \{0,1\}, \quad seed_{i+1}=seed_i+1 \ (\bmod\ p)
\]
ya’ni oddiy “ketma-ket sonlar” uchun Legendre-bitlar ketma-ketligi.

### 1.2. Imzo (signature) nimadan iborat?

```py
snowmark = SHA512(msg) % FROST_PRIME
lantern_key = frostrng.gather_sparkles(500)   # 500 ta RNG biti => eksponent
signature = snowmark^lantern_key mod FROST_PRIME
```

Bu yerda eng muhim nuqtalar:

- `lantern_key` — RNG chiqishidan to‘g‘ridan-to‘g‘ri **500 bit**  
- `signature` — modul `FROST_PRIME` bo‘yicha eksponentatsiya
- `FROST_PRIME` — prime, shuning uchun guruh: \((\mathbb{Z}/p\mathbb{Z})^\*\) va tartib \(p-1\)

### 1.3. Flag nega chiqadi?

OTP tekshiruv:

```py
snow_otp = frostrng.gather_sparkles(len(STARSCROLL)*8)
user_otp = int(input(...), 2)
if user_otp == snow_otp: real flag
```

Demak, biz RNG’ning **keyingi** bitlarini oldindan ayta olsak — flag bizniki.

`assert len(STARSCROLL.strip("HTB{}")) == 79` bo‘lgani uchun flag formati odatda:
- `"HTB{" + 79 ta belgi + "}"` → **84 bayt**
- OTP uzunligi: `84*8 = 672 bit`

---

## 2. Asosiy zaifliklar (vulnerability)

### A) Discrete log juda oson (Pohlig–Hellman)

`signature = snowmark^k mod FROST_PRIME`

Agar biz `k = dlog_{snowmark}(signature)` ni topa olsak — bu `lantern_key` ning o‘zi.

Bu challenge’da `FROST_PRIME-1` juda “smooth” (kichik tub sonlar ko‘paytmasi).  
Shuning uchun **Pohlig–Hellman** bilan DLP juda tez yechiladi.

> **Trik:** `snowmark` generator bo‘lishi uchun `msg` ni bir necha marta o‘zgartirib, guruh tartibi `FROST_PRIME-1` ga teng bo‘ladigan bazani tanlaymiz.

Natija: har bir so‘rov (Option 1) bizga **500 ta RNG biti** beradi.

### B) RNG holati aslida “shift” (siljish)

`holly_prime` server bannerida print qilinadi:

```py
print(f'{frostrng.holly_prime = }')
```

Demak, `p = holly_prime` bizga ma’lum.

RNG esa `seed, seed+1, seed+2, ...` bo‘yicha Legendre-bit beradi.  
Shuning uchun butun holat — bu \([1..p-1]\) aylanishida qaysi joydan boshlagani, ya’ni **sigma**.

---

## 3. Seed (sigma) ni topish g‘oyasi

Bizda:

- Kuzatilgan bitlar: `observed[0..]`
- Bilamiz: `observed[i] = legendre_bit((sigma+i) mod (p-1) + 1, p)`

`p` ~ 34-bit, ya’ni \(p-1 \approx 2^{34}\). To‘liq brute force bo‘lmaydi.

Shuning uchun **BSGS uslubida shift qidirish**:

- \(N = p-1\)  
- \(m = \lceil \sqrt{N} \rceil\)
- `t = 40` bitli bloklar bilan:
  - *baby step*: `observed[i:i+t]` bloklarini jadvalga yozamiz (`i` 0..m-1)
  - *giant step*: haqiqiy Legendre-ketma-ketlikdan `q*m` joydan boshlab `t` bit hisoblaymiz
  - mos kelsa: `sigma = q*m - i (mod N)`
  - so‘ng 2000 bit bilan verifikatsiya

Bu bilan seed topiladi.

---

## 4. Exploit bosqichlari

1) Serverga ulanib `holly_prime` ni olamiz  
2) `msg` tanlaymiz: `SHA512(msg)%FROST_PRIME` generator bo‘lsin  
3) Shu baza uchun **Pohlig–Hellman** precompute qilamiz  
4) `Q` ta signature yig‘amiz (`Q ≈ (sqrt(p-1)+t)/500`)  
5) Har signature → DLP → `lantern_key` → 500 bitni `observed` ga qo‘shamiz  
6) BSGS shift recovery → `sigma`  
7) OTP qachon boshlanishini hisoblaymiz:
   - Har signature 500 bit yutadi → `offset = Q*500 (mod p-1)`
8) `672` bit OTP ni hisoblab, Option 2 ga yuboramiz  
9) Flag qaytadi ✅

---

## 5. POC / Solve script

> TODO: bu yerga o‘zingiz `exploit.py` ni qo‘shing yoki qisqartirilgan variantini.

```bash
python3 exploit.py
```

**Kutilgan natija:** server `{"starshard": "HTB{...}"}` qaytaradi.

---

## 6. Skrinshotlar

- TODO: Ulanish (`nc ...`)
- TODO: Signature olish jarayoni
- TODO: Exploit ishlashi va flag chiqishi

---

## Xulosa

Bu challenge’da “one-time key” aslida:
- Legendre bitlar ketma-ketligi (seed har safar +1)
- `holly_prime` oshkor qilingan
- `FROST_PRIME-1` smooth bo‘lgani uchun DLP oson

Shu sababli imzolar orqali RNG bitlarini oqizib (leak), seed (sigma) ni topib, keyingi OTP bitlarini aniq hisoblab flag olinadi.
