## 1. Процес і джерела

- **Пов'язані EARS-вимоги:** `REQ-05`, `REQ-06`, `REQ-12`, `REQ-13`, `REQ-15`, `REQ-16`, `NFR-3`.
- **Пов'язані артефакти:** User Story 1; use case UC7 (`UC9` `<<extend>>`, `UC20` і `UC22` `<<include>>`); Sequence-діаграма `buy_ticket` (`scenario1.puml`); сутності `Ticket`, `Payment`, `Seat`, `Customer`.
- **Вибрана форма:** PlantUML Activity Diagram

## 2. Обрана форма

Обрано діаграму діяльності, бо процес відповідає на питання «що треба зробити і хто»: дії, розгалуження, fork/join, доріжки, повідомлення. Машина станів описує життєвий цикл одного об'єкта (Ticket) і не показує ролей та паралельності; BPMN надмірний, коли процесом керує одна система, а повідомлення вже описано в sequence.

## 3. Ролі

| Роль            | Хто це                     |
| --------------- | -------------------------- |
| Customer        | автентифікований клієнт    |
| System          | система продажу квитків    |
| Payment Gateway | зовнішній платіжний сервіс |
| Email Service   | зовнішній сервіс листів    |

## 4. Кроки

| №   | Роль            | Крок                                                                        |
| --- | --------------- | --------------------------------------------------------------------------- |
| 1   | Customer        | Select seat                                                                 |
| 2   | System          | Check seat availability; Show message "Seat is already taken" (REQ-16)      |
| 3   | System          | Reserve seat for 10 minutes (REQ-13)                                        |
| 4   | Customer        | Enter promocode (опціонально, REQ-12)                                       |
| 5   | System          | Validate promocode; Recalculate price або Show promocode error              |
| 6   | Customer        | Enter payment details                                                       |
| 7   | System          | Send payment request (повідомлення)                                         |
| 8   | Payment Gateway | Process payment; Send payment result (повідомлення)                         |
| 9   | System          | Cancel seat reservation; Set payment status "failed"; Show message (REQ-15) |
| 10  | System          | Cancel seat reservation; Show "payment not completed" message               |
| 11  | System          | Create ticket (REQ-05, NFR-3)                                               |
| 12  | Customer        | Show success screen                                                         |
| 13  | System          | Send email with ticket (повідомлення, REQ-06, NFR-3)                        |
| 14  | Email Service   | Deliver email; Send delivery status (повідомлення)                          |
| 15  | System          | Mark notification as delivered або Log email failure і Schedule email retry |

## 5. Розгалуження

| Розгалуження        | Гілки                            | Вичерпність                                                                      |
| ------------------- | -------------------------------- | -------------------------------------------------------------------------------- |
| Seat available?     | available / taken                | вичерпно: available йде далі без дій, taken завершує процес                      |
| Use promocode?      | yes / no                         | вичерпно: yes виконує кроки промокоду, no пропускає їх (без дій) і йде до злиття |
| Promocode valid?    | valid / invalid or expired       | дві гілки                                                                        |
| Payment result?     | failed / not completed / success | три гілки; `success` — `else`, тож перелік вичерпний                             |
| Delivery confirmed? | confirmed / error or no response | дві гілки                                                                        |

## 6. Паралельна гілка

Після «Create ticket» виконується `fork` на дві гілки: (а) екран успіху для клієнта, (б) надсилання листа з квитком, отримання статусу доставки й реакція на нього. `join` чекає завершення обох гілок. Усередині `fork` немає дострокових виходів: гілка листа закінчується розгалуженням, і обидві його альтернативи доходять до `join`.

## 7. Повідомлення

| №   | Повідомлення           | Хто надсилає    | Хто приймає     | Що робить процес без відповіді                                                                                            |
| --- | ---------------------- | --------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------- |
| M1  | Send payment request   | System          | Payment Gateway | чекає результату; якщо результату немає, розгалуження «Payment not completed?» → скасування резерву, повідомлення клієнту |
| M2  | Send payment result    | Payment Gateway | System          | `failed` → скасування резерву, статус `failed`; `success` → видача квитка                                                 |
| M3  | Send email with ticket | System          | Email Service   | процес не блокується: клієнт одразу бачить екран успіху (окрема гілка fork)                                               |
| M4  | Send delivery status   | Email Service   | System          | немає статусу або помилка → лог збою і повторна відправка; квиток лишається чинним                                        |
