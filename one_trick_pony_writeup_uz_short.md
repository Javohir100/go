# One Trick Pony — Crypto (qisqa writeup, Uzbek)

## Ma’lumot
- Ulanish: `nc HOST PORT`
- Menu:
  - `1) Etch Message Rune` — biz bergan xabar uchun `signature` qaytaradi
  - `2) Request Wrapped Starshard` — to‘g‘ri `snow-otp` bersak flag beradi

> TODO: skrinshotlar joyi (o‘zingiz qo‘shasiz).

---

## Zaiflik nimada?
Server imzoni shunday qiladi:

- `snowmark = SHA512(msg) mod FROST_PRIME`
- `k = RNG dan 500 bit`  (lantern_key)
- `signature = snowmark^k mod FROST_PRIME`

**Muammo:** `FROST_PRIME-1` juda “smooth” (kichik faktorlar ko‘p), shuning uchun
`k` ni `signature` dan **discrete log** orqali tez topish mumkin (Pohlig–Hellman).

Demak, har bir `signature` bizga RNG’dan **500 bit** “oqizib beradi”.

RNG esa:
- `holly_prime` (34-bit prime) server boshida **print** qiladi
- `seed` har safar `+1` bo‘ladi
- bit = `seed` kvadrat qoldiqmi (Legendre testi)

Shuning uchun RNG aslida “tasodifiy” emas: boshlang‘ich `seed` ni topsak, keyingi barcha bitlarni hisoblaymiz.

---

## Yechim g‘oyasi (sodda)
1) `msg` tanlaymiz: `snowmark` guruhda generator bo‘lsin (DLP osonroq bo‘ladi).
2) Ko‘p marta `signature` olamiz.
3) Har `signature` uchun `k = dlog(signature)` topamiz → bu `k` ning 500 biti RNG bitlari.
4) Yetarli bit yig‘ilgach, `holly_prime` bo‘yicha Legendre-bitlar ketma-ketligiga mos keladigan **boshlanish joyini (seed siljishini)** topamiz (shift search / BSGS).
5) Keyin server flag so‘ragan joydagi OTP bitlarini hisoblab, `2)` ga yuboramiz → flag.

---

## Ishga tushirish
```bash
python3 exploit_clean.py
```

- Skript sizdan `HOST` va `PORT` so‘raydi.
- Oxirida `{"starshard": "HTB{...}"}` chiqishi kerak.

---

## Skrinshotlar
- TODO: `nc` ulanish
- TODO: exploit ishlashi va flag chiqishi
