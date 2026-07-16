# Operatoren und Bindungsstärke

Python wertet einen Ausdruck nicht strikt von links nach rechts aus, sondern nach der
**Bindungsstärke** (Priorität) der beteiligten Operatoren. Operatoren mit höherer
Bindungsstärke werden zuerst ausgewertet. In der folgenden Tabelle stehen die stärker
bindenden Operatoren oben und die schwächer bindenden unten.

## Übersicht der Operatoren

| Operator  | Bedeutung                        |
|-----------|----------------------------------|
| `a ** b`  | Potenz aᵇ                        |
| `a * b`   | a mal b                          |
| `a / b`   | a geteilt durch b                |
| `a // b`  | Integer-Division von a und b     |
| `a % b`   | a modulo b                       |
| `a + b`   | a plus b                         |
| `a - b`   | a minus b                        |
| `a ^ b`   | entweder a oder b                |
| `a < b`   | a kleiner als b                  |
| `a <= b`  | a kleiner als oder gleich b      |
| `a == b`  | a gleich b                       |
| `a != b`  | a ungleich b                     |
| `a >= b`  | a größer als oder gleich b       |
| `a > b`   | a größer als b                   |
| `not a`   | nicht a                          |
| `a and b` | a und b                          |
| `a or b`  | a oder b                         |

*Tabelle 1: Bindungsstärke verschiedener Operatoren (von stark nach schwach bindend).*

## Gruppen gleicher Bindungsstärke

Operatoren innerhalb einer Gruppe binden gleich stark und werden von links nach rechts
ausgewertet. Von der stärksten zur schwächsten Bindung:

1. **Potenz** – `**`
2. **Multiplikative Operatoren** – `*`, `/`, `//`, `%`
3. **Additive Operatoren** – `+`, `-`
4. **Bitweises Exklusiv-Oder** – `^` (entweder … oder)
5. **Vergleichsoperatoren** – `<`, `<=`, `==`, `!=`, `>=`, `>`
6. **Logisches `not`**
7. **Logisches `and`**
8. **Logisches `or`**

:::tip Klammern schlagen Bindungsstärke
Im Zweifel machen runde Klammern `( )` die gewünschte Auswertungsreihenfolge explizit
und die Absicht für Mitlesende klarer – unabhängig von der Bindungsstärke.
:::
