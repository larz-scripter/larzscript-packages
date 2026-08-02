# lz-luhn

The Luhn checksum algorithm: validates card-number-shaped strings and
generates a valid check digit. Checks the number is **well-formed**, not
that a card is real/active/authorized. Install:

```
larzscript pkg install luhn
```

```
import "luhn" as luhn
print(luhn.is_valid("4111111111111111"))   # true (a well-known test number)
print(luhn.check_digit("411111111111111")) # 1 - appending it makes it valid
```
