# 1) Карта экосистем (куда именно писать на Rust)

* **Substrate / Polkadot:**

  * *Runtime (L1)*: пишется на Rust через **FRAME** (пэллеты/модули). Это «низкоуровневые» блокчейн-модули, а не контракты. Документация FRAME/Runtime Dev.
  * *Контракты (L2 на parachain с pallet-contracts)*: **ink!** — Rust → WebAssembly. Старт с `cargo-contract` и пример «Flipper». ([ink!][1], [GitHub][2])
* **Solana:** смарт-программы на Rust (BPF), поверх — фреймворк **Anchor** с декларативными проверками аккаунтов, CPI и миграциями. Официальные доки по SPL/программам и Anchor Book.
* **NEAR:** контракты на Rust → Wasm. Вызовы в JSON, состояние — **Borsh**; SDK и сериализация детально в доках. ([Docs.rs][3], [docs.near.org][4])
* **CosmWasm (Cosmos-SDK):** контракты на Rust → Wasm; стандартные крейты `cosmwasm-std`, `cw-storage-plus`, мульти-тест и паттерны. CosmWasm Book. ([book.cosmwasm.com][5])

---

# 2) Базовые «системные» темы Rust для блокчейна

1. **`no_std`, Wasm и детерминизм**

   * Контракты компилируются в Wasm (ink!, NEAR, CosmWasm). Убираем источники недетерминизма (рандом без seed, плавающие типы в CosmWasm запрещены). CosmWasm отдельно подчеркивает избегание `f64`. ([book.cosmwasm.com][5])
   * Для Solana — BPF таргет, ограничения по стеку/вызовам; Anchor упрощает шаблоны.

2. **Сериализация и ABI**

   * **NEAR**: JSON для входа/выхода, **Borsh** для состояния. ([docs.near.org][4])
   * **CosmWasm**: `serde`/JSON, модель сообщений и Queries. ([book.cosmwasm.com][5])
   * **ink!**: SCALE-кодек (через Substrate), готовые шаблоны.

3. **Модель аккаунтов и владение состоянием**

   * **Solana**: всё — «аккаунты» с данными; проверка владельца, PDA, CPI. Официальные главы про Accounts и CPI.
   * **NEAR**: контракт «привязан» к аккаунту; хранение через коллекции SDK, Borsh-состояние. ([Docs.rs][3])
   * **CosmWasm**: ключ-значение Storage, события, сообщения и IBC. ([book.cosmwasm.com][5])
   * **ink!**: доступ к env, хранилищам и событиям из шаблонов. ([Crates.io][6])

---

# 3) Короткие минимальные примеры (ссылки на «эталоны»)

### ink! (Polkadot / contracts)

```rust
// flipper, минимальный контракт
// cargo install cargo-contract && cargo contract new flipper
// build: cargo contract build
#![cfg_attr(not(feature = "std"), no_std)]
#[ink::contract]
mod flipper {
    #[ink(storage)]
    pub struct Flipper { value: bool }
    impl Flipper {
        #[ink(constructor)]
        pub fn new(init: bool) -> Self { Self { value: init } }
        #[ink(message)]
        pub fn flip(&mut self) { self.value = !self.value; }
        #[ink(message)]
        pub fn get(&self) -> bool { self.value }
    }
}
```

Официальный «Flipper» в доках. ([use-ink.github.io][7])

### Solana (Anchor)

```rust
// Anchor: cargo add anchor-lang
use anchor_lang::prelude::*;

#[program]
pub mod counter {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        ctx.accounts.state.count = 0; Ok(())
    }
    pub fn inc(ctx: Context<Inc>) -> Result<()> {
        ctx.accounts.state.count += 1; Ok(())
    }
}

#[account]
pub struct State { pub count: u64 }

#[derive(Accounts)]
pub struct Initialize<'info> {
    #[account(init, payer = user, space = 8 + 8)]
    pub state: Account<'info, State>,
    #[account(mut)] pub user: Signer<'info>,
    pub system_program: Program<'info, System>,
}

#[derive(Accounts)]
pub struct Inc<'info> {
    #[account(mut, has_one = /* опциональные проверки */ )]
    pub state: Account<'info, State>,
}
```

Смотреть Anchor Book для аккаунтов/PDA/CPI.

### NEAR (near-sdk)

```rust
use near_sdk::{near, AccountId, PanicOnDefault};
use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};

#[near(contract_state)]
#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]
pub struct Counter { value: u64 }

#[near]
impl Counter {
    #[init]
    pub fn new() -> Self { Self { value: 0 } }

    pub fn inc(&mut self) { self.value += 1; }

    pub fn get(&self) -> u64 { self.value }
}
```

NEAR использует JSON для I/O и Borsh для состояния. ([docs.near.org][4])

### CosmWasm

```rust
// Cargo.toml: cosmwasm-std, cw-storage-plus, schemars, serde
use cosmwasm_std::*;
use cw_storage_plus::Item;

#[derive(Serialize, Deserialize, Clone, Debug, PartialEq, Eq, schemars::JsonSchema)]
pub struct State { pub count: u64 }
const STATE: Item<State> = Item::new("state");

#[entry_point]
pub fn instantiate(deps: DepsMut, _env: Env, _info: MessageInfo, _: ()) -> StdResult<Response> {
    STATE.save(deps.storage, &State { count: 0 })?;
    Ok(Response::default())
}

#[entry_point]
pub fn execute(deps: DepsMut, _env: Env, _info: MessageInfo, msg: String) -> StdResult<Response> {
    if msg == "inc" {
        STATE.update(deps.storage, |mut s| { s.count += 1; Ok(s) })?;
    }
    Ok(Response::default())
}

#[entry_point]
pub fn query(deps: Deps, _env: Env, _: ()) -> StdResult<Binary> {
    to_binary(&STATE.load(deps.storage)?)
}
```

Шаблоны/паттерны — в книге CosmWasm. ([book.cosmwasm.com][5])

---

# 4) Хранение и сериализация: подводные камни

* **NEAR + Borsh**: состояние компактно, но требует осторожных миграций (не менять порядок полей без мигрейта). См. заметки по сериализации и near-sdk атрибуты `#[near(serializers=[borsh, json])]`. ([docs.near.org][4], [Docs.rs][8])
* **CosmWasm**: сообщения через JSON/serde, состояние через `cw-storage-plus`, уделите внимание схемам, `multitest`. ([book.cosmwasm.com][5])
* **ink!**: типичный путь — SCALE; стартовые примеры и окружение в доках. ([Crates.io][6], [use-ink.github.io][7])

---

# 5) Безопасность смарт-контрактов (Rust-специфика + платформенные правила)

**Общее:**

* *Детерминизм и отказ от недетерминированных источников:* без `rand`, без сетевого I/O, без `SystemTime::now()` (используйте env/ блок-тайм контрактной платформы).
* *Panic policy:* на Wasm лучше `panic=abort`, избегать `unwrap()`/`expect()` в пользовательских путях.
* *Аудит сериализации:* чётко фиксируйте схемы, особенно с Borsh/ SCALE.

**Платформенно:**

* **Solana/Anchor**:

  * Валидация аккаунтов и прав через `#[account(...)]` constraints, PDA seeds, *не хранить приватные ключи в аккаунтах*. Разделы Accounts/CPI в доках.
* **NEAR**:

  * Понимать, что вход/выход — JSON (дороже по газу), а состояние — Borsh. Следите за «upgradable state» и миграциями. ([docs.near.org][4])
* **CosmWasm**:

  * Минимизируйте хранение, используйте события/атрибуты корректно, читайте раздел «Good practices». ([book.cosmwasm.com][5])
* **ink!**:

  * Проверяйте права доступа и «реэнтрантность» (если внешние вызовы); используйте шаблоны из официальных примеров. ([use-ink.github.io][7])

---

# 6) Тестирование, локальные сети и CI

* **ink!**: `cargo-contract test`, интеграционные примеры из ink examples. ([GitHub][2])
* **Solana/Anchor**: `solana-test-validator`, `anchor test`, мок-аккаунты и CPI-тесты. Anchor Book.
* **NEAR**: юнит-тесты (wasm-сборка), симуляция; документация SDK. ([Docs.rs][3])
* **CosmWasm**: `cw-multi-test` — мощный фреймворк для unit/integration. CosmWasm Book (разделы Query/Multitest). ([book.cosmwasm.com][5])

---

# 7) Производительность и газ

* **Размер Wasm и аллокации:** включайте `lto`, `opt-level = "z"`, `codegen-units = 1`; избегайте ненужных аллокаций (`String` ↔ `&str`, `Vec::with_capacity`), упрощайте схемы.
* **Solana (BPF)**: будьте внимательны к вычислительным бюджетам; Anchor генерирует проверки, но они стоят compute units. Смотрите SPL/доки.
* **NEAR**: JSON дороже Borsh — переносите тяжёлые данные в состояние/в off-chain, используйте `#[payable]` для платных методов. ([Docs.rs][3])
* **CosmWasm**: стройте сообщения экономно, используйте батчи, храните хэши вместо больших payload’ов. ([book.cosmwasm.com][5])

---

# 8) Межконтрактные и межцепочечные вызовы

* **Solana**: CPI (cross-program invocation) — стандартный способ композиции.
* **Substrate/Polkadot**: XCM для межцепочечных сообщений на уровне экосистемы.
* **CosmWasm**: IBC из коробки (контракты как акторы). ([book.cosmwasm.com][5])

---

# 9) Апгрейды и миграции

* **NEAR**: фиксируйте Borsh-схемы; добавление полей — в конец; при несовместимости — миграционная функция/инициализатор. ([docs.near.org][4])
* **CosmWasm**: храните `contract_version`, пишите миграции, проверяйте совместимость Storage. (см. Book — «Good practices»/«State») ([book.cosmwasm.com][5])
* **ink!**: поддерживайте совместимость SCALE при изменении хранилища; используйте проверенные шаблоны. ([use-ink.github.io][7])
* **Solana/Anchor**: миграции аккаунтов через новые инструкционы и проверки `#[account]`; сохраняйте layout.

---

# 10) Криптография и Zero-Knowledge (дополнительно)

Если вам нужны on-chain доказательства/верификаторы или off-chain-преверка с верификацией в контракте:

* **arkworks** — экосистема Rust для zkSNARK’ов (поля, кривые, R1CS, примитивы). ([arkworks][9], [GitHub][10], [Crates.io][11])
* **Halo2** (Zcash) и производные (halo2-base/axiom, ecc): конструирование Plonk-совместимых схем. ([Zcash][12], [Crates.io][13])
* Полезные заметки по производительности/параллелизму в halo2-stack (использование Rayon). ([Crates.io][14])

---

# 11) Что ещё положить в «личный учебный план»

1. **Язык/инструменты Rust, ориентированные на блокчейн:** `no_std`, `core/alloc`, `serde`/`borsh`, `thiserror`, `anyhow` (off-chain), `proptest`/`quickcheck` для генеративных тестов.
2. **Безопасные шаблоны хранения:** *newtype* для токенов/сумм, явные лимиты (`u128` для балансов), инварианты в одном месте.
3. **DevOps:** локальные валидаторы (`solana-test-validator`, `wasmd` для CosmWasm), пайплайны сборок, размеры артефактов и проверка детерминизма.
4. **Чтение кода референс-контрактов:** ink! examples, cw-plus, примеры Anchor/SPL. ([GitHub][2])

---

## Быстрые ссылки на «официальщину»

* **ink! / cargo-contract / примеры:** старт/«Flipper». ([ink!][1], [use-ink.github.io][7])
* **Substrate FRAME (runtime-разработка на Rust):** обзор.
* **Solana:** Program model, Accounts, CPI; **Anchor Book**.
* **NEAR:** near-sdk (атрибуты, init/payable) и заметки по сериализации JSON/Borsh. ([Docs.rs][3], [docs.near.org][4])
* **CosmWasm:** книга, API, multitest, storage. ([book.cosmwasm.com][5])
* **ZK на Rust:** arkworks, halo2. ([arkworks][9], [Zcash][12])

---

## Куда двигаться дальше (практика по уровням)

* **Level 1 (разогрев):**

  * Разверните и протестируйте `Flipper` на ink! и простой счётчик на CosmWasm/NEAR.
  * В Anchor напишите счётчик + PDA счет. ([use-ink.github.io][7], [book.cosmwasm.com][5], [docs.near.org][4])
* **Level 2 (композиция):**

  * Solana CPI между двумя программами; в CosmWasm — межконтрактные вызовы и события. ([book.cosmwasm.com][5])
* **Level 3 (производство):**

  * Апгрейд контракта (миграции); лимиты газа; аудит сериализации (Borsh/JSON/SCALE). ([docs.near.org][4], [book.cosmwasm.com][5])
* **Level 4 (криптография/ZK):**

  * Соберите простой доказуемый вычислительный пример на arkworks/halo2; разберитесь с параметрами/многопоточностью. ([arkworks][9], [Crates.io][14])

Если хотите, могу собрать для вас практический «курс» из 12–15 упражнений с прогрессией по каждой из четырёх экосистем (repo-скелеты + тесты) и чек-лист «в прод».

[1]: https://use.ink/docs/v6/getting-started/creating-an-ink-project/?utm_source=chatgpt.com "Create an ink! Project | Documentation | ink!"
[2]: https://github.com/use-ink/ink-examples?utm_source=chatgpt.com "A set of examples for ink! smart contract language. Happy ..."
[3]: https://docs.rs/near-sdk/latest/near_sdk/?utm_source=chatgpt.com "near_sdk - Rust"
[4]: https://docs.near.org/smart-contracts/anatomy/serialization?utm_source=chatgpt.com "Notes on Serialization | NEAR Documentation"
[5]: https://book.cosmwasm.com/ "Introduction - CosmWasm book"
[6]: https://crates.io/crates/ink_env?utm_source=chatgpt.com "ink_env - crates.io: Rust Package Registry"
[7]: https://use-ink.github.io/ink/ink/attr.contract.html?utm_source=chatgpt.com "contract in ink - Rust"
[8]: https://docs.rs/near-sdk/latest/near_sdk/attr.near.html?utm_source=chatgpt.com "near in near_sdk - Rust"
[9]: https://arkworks.rs/?utm_source=chatgpt.com "arkworks"
[10]: https://github.com/arkworks-rs?utm_source=chatgpt.com "arkworks"
[11]: https://crates.io/crates/ark-r1cs-std?utm_source=chatgpt.com "ark-r1cs-std - crates.io: Rust Package Registry"
[12]: https://zcash.github.io/halo2/?utm_source=chatgpt.com "The halo2 Book - Zcash"
[13]: https://crates.io/crates/halo2-ecc?utm_source=chatgpt.com "halo2-ecc - crates.io: Rust Package Registry"
[14]: https://crates.io/crates/halo2-axiom?utm_source=chatgpt.com "halo2-axiom - crates.io: Rust Package Registry"
