Хорошо, давай я соберу **курс из 12–15 практических упражнений** по Rust для блокчейна, где шаг за шагом пройдём Substrate/ink!, Solana/Anchor, NEAR и CosmWasm, включая апгрейды, межконтрактное взаимодействие и ZK.

Сделаем так:

* Упражнения будут идти по уровням сложности.
* Дам ссылки на код и документацию, чтобы сразу можно было повторить.
* В конце — **чек-лист для продакшена**.

---

## 📚 Практический курс «Rust в блокчейне»

### **Level 1 – Разогрев (простые контракты)**

1. **ink! — Flipper**

   * Цель: развернуть и протестировать минимальный контракт (`flip()` / `get()`).
   * Код: [ink! Flipper example](https://use.ink/examples/flipper)
   * Шаги: `cargo contract new flipper` → `cargo contract build` → тесты.

2. **NEAR — Counter**

   * Цель: контракт с инкрементом счётчика, вызовы через near-cli.
   * Код: [near-sdk-rs examples](https://github.com/near/near-sdk-rs/tree/master/examples)
   * Использовать JSON-вызовы и Borsh для состояния.

3. **CosmWasm — Counter**

   * Цель: базовый execute/query, хранение в `cw-storage-plus`.
   * Код: [cw-template](https://github.com/CosmWasm/cw-template)
   * Деплой в локальный wasmd, тесты через `cargo wasm` + `wasmd tx/query`.

4. **Solana/Anchor — Counter**

   * Цель: PDA-хранилище для счётчика, команды `inc()` и `get()`.
   * Код: [Anchor Book – example counter](https://book.anchor-lang.com/chapter_3/intro.html)
   * Запуск в `solana-test-validator`.

---

### **Level 2 – Композиция и события**

5. **ink! — Контракт с событиями**

   * Цель: генерировать события при изменении состояния.
   * Ссылка: [ink! events](https://use.ink/basics/events)

6. **NEAR — Коллекция пользователей**

   * Цель: хранить в `UnorderedMap<AccountId, u64>` и уметь читать.
   * Документация: [NEAR Collections](https://docs.rs/near-sdk/latest/near_sdk/collections/index.html)

7. **CosmWasm — Межконтрактный вызов**

   * Цель: вызвать другой контракт и получить ответ.
   * Ссылка: [CosmWasm inter-contract](https://docs.cosmwasm.com/docs/smart-contracts/message)

8. **Solana/Anchor — CPI вызов**

   * Цель: вызвать метод из другой программы Anchor.
   * Документация: [Anchor CPI](https://book.anchor-lang.com/chapter_6/intro.html)

---

### **Level 3 – Продвинутые паттерны**

9. **ink! — Контракт с доступом по ролям**

   * Цель: роль `admin` может менять состояние, остальные — нет.
   * Ссылка: [ink! access control pattern](https://github.com/paritytech/ink-examples)

10. **NEAR — Миграция состояния**

    * Цель: изменить структуру, добавить поле и сохранить данные.
    * Документация: [State migration NEAR](https://docs.near.org/develop/contracts/migration)

11. **CosmWasm — IBC контракт**

    * Цель: отправить и принять сообщение через IBC.
    * Ссылка: [IBC CosmWasm](https://docs.cosmwasm.com/docs/smart-contracts/ibc)

12. **Solana — Token Mint**

    * Цель: создать токен и выпускать его в аккаунты пользователей.
    * Документация: [SPL Token Program](https://spl.solana.com/token)

---

### **Level 4 – Оптимизация и ZK**

13. **Оптимизация Wasm (ink!, NEAR, CosmWasm)**

    * Включить LTO, `opt-level=z`, минимизировать аллокации.
    * Ссылка: [Rust Wasm size optimizations](https://rustwasm.github.io/book/game-of-life/code-size.html)

14. **Solana — Вычислительный бюджет**

    * Измерить compute units и оптимизировать код.
    * Документация: [Compute Budget Solana](https://docs.solana.com/developing/programming-model/runtime#compute-budget)

15. **ZK-доказательство на Rust**

    * Использовать `arkworks` для генерации и проверки простого доказательства.
    * Ссылка: [arkworks examples](https://github.com/arkworks-rs)

---

## ✅ Чек-лист для продакшена

* [ ] Сборка с `panic=abort` и оптимизация размера.
* [ ] Лимиты газа/compute units протестированы.
* [ ] Права доступа проверяются в каждом entrypoint.
* [ ] Миграции состояния документированы.
* [ ] Все сериализаторы/десериализаторы зафиксированы.
* [ ] Юнит-тесты и интеграционные тесты проходят.
* [ ] Нет `unwrap()`/`expect()` в пользовательских путях.
* [ ] Проведён аудит зависимостей (cargo-deny).

---

Если хочешь, я могу сделать **готовый .md файл** с этим курсом,
где к каждому упражнению будут:

* минимальный шаблон кода,
* команды сборки/деплоя,
* и ссылки на официальные примеры.

Тогда получится прям пошаговый учебник с кодом.

Сделать тебе такой файл?
