# One Trick Pony (Crypto) — Writeup (UZ)

## Challenge info
- Ulanish: `nc HOST PORT`
- Fayl: `server.py`

## Maqsad
`2)` menyuda server “snow-otp” so‘raydi. To‘g‘ri yuborsak `flag` beradi.

## Kuzatuv
Server `1)` da imzo qaytaradi:

- `snowmark = SHA512(msg) mod FROST_PRIME`
- `k = RNG dan 500 bit`
- `signature = snowmark^k mod FROST_PRIME`

RNG esa `holly_prime` bo‘yicha ishlaydi va `seed` har safar `+1` bo‘ladi, bit esa **Legendre testi** (kvadrat qoldiq / qoldiq emas).

## Zaiflik
1) `FROST_PRIME-1` juda “smooth”, shuning uchun `signature = g^k` dan **k ni discrete log** bilan tez topish mumkin (Pohlig–Hellman).  
2) Shunday qilib har `signature` bizga RNG’dan **500 bit** berib yuboradi.  
3) Yetarli bit yig‘ib, `holly_prime` bo‘yicha Legendre-bit ketma-ketligida qaysi joydan boshlanganini (shift/seed) topamiz.  
4) Seed topilgach, keyingi OTP bitlarini hisoblab `2)` ga yuboramiz.

## Yechim bosqichlari
1. Ulanamiz va banner’dan `holly_prime` ni olamiz.  
2. `msg` tanlaymiz: `g = SHA512(msg) mod FROST_PRIME` generator bo‘lsin.  
3. Shu `g` uchun Pohlig–Hellman tayyorlaymiz.  
4. `Q` marta `1)` orqali `signature` olamiz, har safar `k = dlog_g(signature)` ni topib, 500 bitni `observed` ga qo‘shamiz.  
5. `observed` bitlardan `holly_prime` bo‘yicha shift (sigma) ni topamiz.  
6. `Q*500` bit “sarflangan” bo‘ladi, shuni offset qilib, flag uzunligiga mos `672` OTP bitni hisoblaymiz.  
7. `2)` ga OTP ni yuboramiz va flag chiqadi.

## Run
```bash
python3 exploit_clean.py
```
Skript sizdan `HOST` va `PORT` so‘raydi.

## Proof
- TODO: ulanish skrinshot
- TODO: flag chiqqan skrinshot

## Flag
- `HTB{...}`
