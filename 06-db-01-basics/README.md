1# Домашнее задание к занятию "6.1. Типы и структура СУБД"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Архитектор ПО решил проконсультироваться у вас, какой тип БД 
лучше выбрать для хранения определенных данных.

Он вам предоставил следующие типы сущностей, которые нужно будет хранить в БД:

- Электронные чеки в json виде
```
  Ответ: Документо-ориентированная база данных. NO SQL. 
  Хотя все больше поставщиков ориентированы на создание универсальных базам. 
```
- Склады и автомобильные дороги для логистической компании
```
  Ответ. Формально бы я ответил, что реаляционные 
  (посколько все равно нужно хранить транспортные операции, 
  и свое время доминировал классический Oracle). 
  Но сейчас даже SAP HANA использует Hadoop, что является No SQL, база колончатая.
  И возможно под эту задачу подходят сетевые и графовые базы данных (типа AWS Neptun)
  , но с таковыми, работая в транспортной компании не пересекался, скорее экзотика. 
  ```
- Генеалогические деревья
```
  Ответ. Иерархические база, но опять таки ноды можно спокойно хранить в реляционной базе.
```
- Кэш идентификаторов клиентов с ограниченным временем жизни для движка аутенфикации
```  
  Ответ. Ну тут - ключ-значение. Типа Redis.
```
- Отношения клиент-покупка для интернет-магазина
```
  Ответ. Реаляционная, хотя при больших объемах продаж, скорее уже колончатая (биг-дата).
```  
Выберите подходящие типы СУБД для каждой сущности и объясните свой выбор.
```
PS. Я лишь к тому, что если организация небольшая, не много данных, 
во многих случаях вполне достаточно реаляционной. 
Но последнее время значительное место в работе занимает универсальная база данных Postgre.
```
## Задача 2

Вы создали распределенное высоконагруженное приложение и хотите классифицировать его согласно 
CAP-теореме. Какой классификации по CAP-теореме соответствует ваша система, если 
(каждый пункт - это отдельная реализация вашей системы и для каждого пункта надо привести классификацию):

- Данные записываются на все узлы с задержкой до часа (асинхронная запись)
```
   CA
```
- При сетевых сбоях, система может разделиться на 2 раздельных кластера
```
   AP
```
- Система может не прислать корректный ответ или сбросить соединение
```
   CP
```

А согласно PACELC-теореме, как бы вы классифицировали данные реализации?
```
1. PC/EL 2. PA/EL 3.PA/EC 
https://habr.com/ru/company/gaz-is/blog/551986/
```
## Задача 3

Могут ли в одной системе сочетаться принципы BASE и ACID? Почему?
```
Не могут, в BASE могут быть неверные данные. 
Возможна деградация сессий. Возможно неустойчивость состояния.
```
## Задача 4

Вам дали задачу написать системное решение, основой которого бы послужили:

- фиксация некоторых значений с временем жизни
- реакция на истечение таймаута

Вы слышали о key-value хранилище, которое имеет механизм [Pub/Sub](https://habr.com/ru/post/278237/). 
Что это за система? Какие минусы выбора данной системы?
```
Переодически с Redis работаю. Классика key-value.
По сути ключи в памяти хранятся, в случае отказа базы, данные будут утеряны. 
Хотя есть опция сохранения на жеский диск. Но тогда, база работает медленнее.

Для использования нужно рассчитывать ресурсы памяти на хранение данных. 
В основном, описываемое решение похоже на хранения тасков к исполнению и очередей.
В случае Питона, это Celery-подобные решения.
---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
