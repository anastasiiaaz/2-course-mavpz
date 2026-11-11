## 1. Загальний список учасників

- **Актори (Люди та Зовнішні сервіси):**
- `Customer` — покупець
- `Admin` — адміністратор системи
- `Payment Gateway` — платіжний шлюз
- `Email Service` — поштовий сервіс

- **Системні компоненти:**
- `System (App)` — програмний модуль системи (інтерфейс + бекенд-логіка).
- `Database` — база даних для фіксації станів сутностей (`Ticket`, `Event`, `Payment`, `Seat`).

---

## 2. Сценарії та кроки виконання

### UC7: Buy Ticket (Купівля квитка)

- **Основний актор:** `Customer`
- **Інші учасники:** `System`, `Database`, `Payment Gateway`, `Email Service`

**Кроки сценарію:**

**Кроки:** Покупець обирає місце, після чого System тимчасово резервує його в Database (`reserveSeat`). За наявності промокоду System перевіряє його в Database та коригує ціну. Після підтвердження оплати System надсилає запит `processPayment` до Payment Gateway. У разі успіху System реєструє в Database квиток зі статусом `ACTIVE` (`issueTicket`), показує екран підтвердження та надсилає лист через Email Service. При відмові шлюзу System викликає `cancelReservation` у Database і виводить помилку.

---

### UC19: Cancel Event

- **Основний актор:** Customer
- **Інші учасники:** System, Database, Payment Gateway, Email Service

**Кроки:** Адміністратор ініціює `cancelEvent`, після чого System оновлює статус події в Database на `CANCELLED` і отримує список квитків. У циклі для кожного квитка System надсилає `processRefund` до Payment Gateway: при успіху змінює статуси квитка та платежу в Database на `REFUNDED` і надсилає сповіщення через Email Service, а при відмові — логує помилку в Database (`logRefundError`). Наприкінці System інформує Admin про завершення.

---

### UC18: Scan Ticket (Контроль та сканування квитка)

- **Основний актор:** `Admin`
- **Інші учасники:** `System`, `Database`

**Кроки:** Адміністратор зчитує QR-код (`scanTicket`), а System перевіряє статус квитка в Database (`getTicketStatus`). Якщо статус `ACTIVE`, System оновлює його на `USED` у Database і видає «ACCESS GRANTED». Якщо статус вже `USED`, видається помилка про повторний прохід, а при статусі `REFUNDED` чи відсутності квитка — повідомлення про недійсність.

---

- **TicketStatus**: `RESERVED`, `ACTIVE`, `USED`, `REFUNDED`, `EXPIRED`
- **EventStatus**: `DRAFT`, `PUBLISHED`, `CANCELLED`, `COMPLETED`
- **PaymentStatus**: `PENDING`, `SUCCESS`, `FAILED`, `REFUNDED`
