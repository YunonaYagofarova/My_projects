# My_projects
Здесь представлены мои проекты

## Проект первый 

### Построение калькулятора юнит-экономики для онлайн-кинотеатра  

##### Поставленные задачи: 
<ul>
  <li> На основе полученных данных рассчитать необходимые метрики, провести их анализ и сделать вывод о динамике пользовательской активности на платформе </li>
  <li> Построить калькулятор, котрый будет автоматически рассчитывать следующие метрики: 
    <ul>  
      <li> Количество повторных оплат в каждом месяце </li>
      <li> Retention для каждого месяца </li> 
      <li> Среднее геометрическое Retention </li>
      <li> Лайфтайм </li>
      <li> LTR </li>
      <li> CAC </li>
      <li> Маржинальность </li> 
    </ul>
  <li> Подогнать параметры таким образом, чтобы маржинальность была +25% </li>
  <li> Построить визуализацию </li>
</ul>

##### В результате получили следующее:

калькулятор выглядит так: 

![Калькулятор](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%BA%D0%B0%D0%BB%D1%8C%D0%BA%D1%83%D0%BB%D1%8F%D1%82%D0%BE%D1%80.png)

В графе "Изменение" подставляем свои данные и смотрим на получившуюся маржинальность. Визуально все изменения отражаются на графике: 

![График калькулятора](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%AE%D0%BD%D0%B8%D1%82%20%D0%AD%D0%BA%D0%BE%D0%BD%D0%BE%D0%BC%D0%B8%D0%BA%D0%B0.png)

Дополнительно были рассчитаны и визуализированы необходимые метрики. 

![Активность пользователей](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%B0%D0%BA%D1%82%D0%B8%D0%B2%D0%BD%D0%BE%D1%81%D1%82%D1%8C.png)

![Распределение просмотров по суточным часам](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5%20%D0%BF%D0%BE%20%D0%BF%D0%BE%D1%8F%D1%81%D0%B0%D0%BC.png)

![Распределение просмотров часовым поясам](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D1%87%D0%B0%D1%81%D0%BE%D0%B2%D1%8B%D0%B5.png)


Мы видим, что к концу анализируемого периода маржинальность стала положительной, но существенно уменьшилось количество новых подписок, скорее всего это вызвано снижением затрат на маркетинг (возможно закончилась какая-то рекламная акция).  Большинство пользователей смотрят  от 3 до 12 фильмов, пик просмотров приходится на пт, сб и вс в вечернее время - если увеличится срок жизни пользователя на сайте, эти цифры тоже будут расти. Небольшое количество просмотренных фильмов, скорее всего, связано с тем, что много пользователей оформили подписку не так давно, а смотрят фильмы чаще именно в выходные. Чтобы выйти на маржу уровня 25 % нужно снизить среднюю стоимость привлечения пользователя на 10 % . Повысив Retention на 20 % до уровня 96,72 % мы увеличим общее количество оплат за 6 месяцев, при этом, чтобы выйти на планируемую маржинальность, нужно увелиить на 10 % стоимость подписки и снизить объём скидок. При планировании маркетинговых мероприятий желательно проанализировать каналы привлечения подписчиков, учитывать часовые пояса и время просмотра.

## Проект второй 

### Моделирование изменения балансов студентов

##### Поставленные задачи: 

<ul>
  <li> Проверить корректность данных в таблицах </li>
  <li> Посмотреть изменения балансов студентов за каждый день </li>
  <li> Проверить как это количество менялось под влиянием транзакций и прохождения уроков </li>
  <li> Создать визуализацию итогового результата и сделать выводы  </li>
</ul>  

##### В результате:

Написан код SQL 

```
with first_payment as (
select user_id
     , date_trunc ('day',transaction_datetime::date) as first_payment_date
from (select *
              , row_number () over (partition by user_id order by transaction_datetime) as rn
      from skyeng_db.payments
      where status_name = 'success') a
where rn = 1
),

all_dates as (
    select distinct date_trunc ('day', class_start_datetime) as dt
    from skyeng_db.classes
    where class_start_datetime between '2016.01.01' and '2016.12.31'
),

all_dates_by_user as (
select user_id
     , dt
from first_payment b
   join all_dates c on b.first_payment_date <= c.dt
order by user_id, dt
),

payments_by_dates as (
select user_id
     , date_trunc ('day', transaction_datetime) as payment_date
     , sum (classes) as transaction_balance_change
from skyeng_db.payments
where status_name = 'success'
group by 1, 2 
),

payments_by_dates_cumsum as (
select d.user_id
     , d.dt
     , f.transaction_balance_change
     , sum (transaction_balance_change) over (partition by d.user_id order by d.dt rows between unbounded preceding and current row) as transaction_balance_change_cs
from all_dates_by_user d 
    left join payments_by_dates f on d.user_id = f.user_id and d.dt = f.payment_date
),

classes_by_dates as (
select user_id
     , date_trunc('day', class_start_datetime) as class_date
     , count (id_class)*-1 as classes
from skyeng_db.classes
where class_status in ('success', 'failed_by_student') 
group by 1, 2
 ),

classes_by_dates_dates_cumsum as (
select g.user_id
     , g.dt
     , h.classes
     , sum (case when h.classes is null then 0 else h.classes end) over (partition by g.user_id order by g.dt rows between unbounded preceding and current row) as classes_cs
from all_dates_by_user g 
     left join classes_by_dates h on g.user_id = h.user_id and g.dt = h.class_date
),

balances as (
select k.user_id
     , k.dt
     , k.transaction_balance_change
     , k.transaction_balance_change_cs
     , l.classes
     , l.classes_cs
     , (k.transaction_balance_change_cs + l.classes_cs) as balance
from payments_by_dates_cumsum  k 
     left join classes_by_dates_dates_cumsum l on k.user_id = l.user_id and k.dt = l.dt

select dt
     , sum (transaction_balance_change) as ch_tr_balance_change
     , sum (transaction_balance_change_cs) as ch_tr_balance_change_cs
     , sum (classes) as ch_classes
     , sum (classes_cs) as ch_classes_cs
     , sum (balance) as ch_balance
from balances
group by dt
order by dt
```

