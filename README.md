# JunoSmartContractDeployment
 Инструкция по созданию смарт контракта erc20 в Juno Test Network Lucina

# Установка 
Rust и Junod [installation page.](https://docs.junochain.com/smart-contracts/installation)

Для начала , установим [rustup](https://rustup.rs/).
```
$ rustup default stable
$ cargo version
$ rustup update stable

$ rustup target list --installed
$ rustup target add wasm32-unknown-unknown
```
# Собираем клиент Juno для использования в тестнете
Используя go 1.16.3 для компилирования исполнимого бинарника junod


```
$ clone juno repo 
$ git clone https://github.com/CosmosContracts/Juno.git && cd Juno

> build juno executable

$ make install
$ which junod
```
# Теперь требуется установить CosmWasm
```
$ git clone https://github.com/CosmWasm/cosmwasm-examples
$ cd cosmwasm-examples
$ git fetch
$ git checkout v0.10.0 
$ cd contracts/erc20
```
# Следующий шаг - Компиляция

```
$ rustup default stable
$ cargo wasm
```
Нам нужно ограничить использование газа. Для этого мы пишем:
```
$ sudo docker run --rm -v "$(pwd)":/code \
    --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
    --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
    cosmwasm/rust-optimizer:0.11.4
```
Это создаст cw_erc20.wasm в каталоге 'artifacts'.
# Выгружаем
Стоит заметить, что вам нужно будет узнать адресс активной rpc ноды. На момент написания гайда была использована https://rpc.juno.giansalex.dev:443/
```
$ cd artifacts
$ junod tx wasm store cw_erc20.wasm  --from <your-key> --chain-id=<chain-id> --gas auto --node  https://rpc.juno.giansalex.dev:443/
```
В сроке возврата вам нужно будет найти ID контракта {"key":"code_id","value":"6"} 
# Инициализация контракта.
После того как мы загрузили контракт на нужно его инициализировать. Как пример будем использовать "Aqua coin"

# Генерация JSON с аргументами
Для генерации файла JSON мы будем использовать node REPL но так же можно использовать jq.
Что бы установить node на вашу систему введите `$ sudo apt install nodejs`
Введите `$ node` и нажмите Enter

```js
> const initHash = {
  name: "Aqua Coin",
  symbol: "aqua",
  decimals: 83,
  initial_balances: [
    { address: " juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl", amount: "12345678000"},
  ]
};
< undefined
> JSON.stringify(initHash);
< '{"name":"Aqua Coin","symbol":"aqua","decimals":83,"initial_balances":[{"address":" juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl","amount":"12345678000"}]}'
```
# Теперь нужно создать экземпляр контракта
Обратите внимание, что --amount используется для инициализации новой учетной записи, связанной с контрактом. В приведенном ниже примере 82 - это значение $ CODE_ID.
```js
junod tx wasm instantiate 83 \
    '{"name":"Aqua Coin","symbol":"AQUA","decimals":83,"initial_balances":[{"address":" juno1ucdafm0s4uka6lnmaa7dl7tt489l2wfv3s3wfl ","amount":"12345678000"}]}' \
    --amount 50000ujuno  --label "Aquacoin erc20" --from Maxon --chain-id lucina --gas auto -y --node https://rpc.juno.giansalex.dev:443/
```

Посмотрите вывод и найдите адрес контракта, например, juno1a2b ....

Для удобства использования можем создать переменную нашего смарт-контракта :
` CONTRACT_ADDR=[аддресс вашего контракта из предыдущей команды]`  и проверить `echo $CONTRACT_ADDR`

Теперь делаем запрос нашего контракта 
`junod query wasm contract $CONTRACT_ADDR`

# Использование команд, поддерживаемых функцией execute
`junod tx wasm execute [contract_addr_bech32] [json_encoded_send_args] --amount [coins,optional] [flags]`

Для перевода можно использовать команду:
```
junod tx wasm execute <contract-addr> '{"transfer":{"amount":"200","owner":"<validator-self-delegate-address>","recipient":"<recipient-address>"}}' --from <your-key> --chain-id <chain-id>
```

# Схема в папке контрактов / erc20 внутри cosmwasm-examples
```
tree schema

schema
├── allowance_response.json
├── balance_response.json
├── constants.json
├── execute_msg.json
├── instantiate_msg.json
└── query_msg.json
```

Надеемся процесс создания мемкоина прошел увлекательно и плавно. У вас так же будет возможность сделать это в основной сети.
