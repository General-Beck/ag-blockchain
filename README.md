# Активный гражданин. Блокчейн для проверки результатов голосования #

Репозиторий содержит конфигурацию для запуска блокчейн-ноды на любой системе для просмотра результатов опросов на сайте [Активный гражданин](https://ag.mos.ru/)

## Инструкция по развертыванию ##

**Для установки блокчейн-ноды (используемая реализация блокчейн Parity) необходимо предварительно выполнить шаги:**

0. Установить и запустить [Docker](https://www.docker.com) (все необходимые требования и инструкции по установке есть на указанном сайте) 
1. Установить git ([Git for Windows](https://git-for-windows.github.io/), [Git for Linux](https://git-scm.com/book/ru/v1/%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5-%D0%A3%D1%81%D1%82%D0%B0%D0%BD%D0%BE%D0%B2%D0%BA%D0%B0-Git)) для того, чтобы скачать необходимую для блокчейн конфигурацию из репозитория.
2. Открыть порт 30303 для исходящих соединений, в случае если он закрыт (см. настройки firewall системы)

**Выполняем следующие команды в bash (для Linux и MacOS) или powershell (для Windows, от имени администратора):**

0. Скачиваем конфигурацию из репозитория ```git clone https://github.com/moscow-technologies/ag-blockchain.git```
1. Переходим в каталог с конфигурацией ```cd ag-blockchain```
2. Запускаем parity командой ```docker-compose up -d```
3. Получаем и копируем токен (примерно такой *Q7J9-ofgq-EEJU-9nVt*) для авторизации в Parity UI `docker-compose logs | grep token=`

**Нода запущена, должна пойти ее синхронизация с блокчейном. Далее можно зайти в [ее веб-интерфейс](http://localhost:8180) и выолнить его настройку для просмотра результатов опросов**

0. Система запросит токен, вводим полученный предыдущей командой, вводим его
1. Принимаем условия лицензионного соглашения, а также создаем аккаунт в системе (логин и пароль - произвольные) 
2. Переходим в раздел [Settings](http://localhost:8180/#/settings/views) и включаем галочку напротив пункта `Contracts`
3. Переходим в раздел [Status](http://localhost:8180/#/status). Должна идти синхронизация блоков (число напротив надписи `Best block` растет быстрее чем на единицу в секунду). Второе число в `Peers` должно быть больше 0 (если синхронизация работает корректно)!
4. Дожидаемся окончания синхронизации надпись `Chain synchronized (yes)`

## Инструкция по просмотру результатов опросов ##
**Используя Parity UI, можно узнать общую статистику по конкретному опросу, а также как голосовал конкретный пользователь (по его личному UID). Для этого выполняем следующие действия:**

0. Переходим в раздел [Contracts](http://localhost:8180/#/contracts)
1. Для просмотра адресов смарт-контрактов опросов в блокчейн необходимо добавить в просмотр корневой контракт. 
2. Нажимаем `Watch`, появляется мастер, в нем на первом шаге жмем `Next`
3. На втором шаге в поле `network address` вводдим значение ```0xFDb76DaAF371bf5C7122f6f1104458440454FBB1```, в поле `contract name` пишем `Каталог опросов`, в поле `contract abi` - содержимое файла `conracts/Root.abi` данного репозитория и нажимаем `Add contract`
4. Заходим в разделе `Contracts` в `Каталог опросов` и в поле под надписью getAddress вставляем идентификатор опроса на [АГ](https://ag.mos.ru/poll/index), жмем `Query`. Получаем адрес смарт-контракта с соответствующим опросом, копируем его в буфер обмена
5. Возвращаемся в раздел `Contracts`, добавляем в просмотр смарт-контракт опроса: нажимаем `Watch`, `Next`, дальше в поле`network address` - адрес из буфера обмена, `contract name` - будущее название контракта в списке (например `Опрос 3196`), `contract abi` - сожержимое файла `contracts/Poll.abi` из репозитория, нажимаем `AddContract`
6. Заходим в раздел `Contracts` в просмотр смарт-контракта `Опрос 3196`. Здесь хранится соответствие идентификатора вопроса на АГ и адреса смарт-контрактов вопросов (разделы `QuestionIds` и `QuestionsAddress`), копируем адрес. Аналогично, необходимо копировать адреса смарт-контрактов и добавлять в разделе `Contracts` - `Watch` смарт-контракты, хранящие описание, ответы и результаты голосования по конкретному вопросу опроса. В первое поле вставляем адрес, во второе - название (например `Опрос 3195 Вопрос1`), в поле `contract abi` - содержимое файла `contracts/PollQuestion.abi` из репозитория. Нажимаем `AddContract`
7. Переходим в просмотр вопроса и видим содержимое блокчейн

- Идентификатор `QuestionId` 
- Заголовок вопроса `CurrentVersionTitle` 
- Количество проголосовавших `VoterCount`
- Идентифиторы версий опроса `AllExistingVersions` (каждое изменение опроса ведет к созданию версии, голоса по разным версия считаются отдельно)
- Идентифитор текущей версии `CurrentVersion`
- Количество голосов за каждый из ответов в списке `CurrentVersionResults` (в порядке из идентификаторов, совпадает с порядком на сайте АГ)
- Запрос `_versions` c указанием идентификатора версии вохвращает название вопроса и список ответов с идентификаторами (формат JSON) в указанной версии 
- Запрос `AnswerIdsByVersion` c указанием идентификатора версии возвращает список идентификаторов ответов с портала АГ в указанной версии
- Запрос `AnswersByVersion` c указанием идентификатора версии возвращает список ответов с идентификаторами (формат JSON) в указанной версии 
- Запрос `ResultsByVersion` c указанием идентификатора версии возвращает количество голосов за каждый из ответов в указанной версии
- Запрос `VoteOfUser` с указанием UID пользователя на сайте АГ возвразает номер ответа за который голосовал пользователь `result` и хэш ответа, если пользователь вводил текст ответа `value1`, `value2`
