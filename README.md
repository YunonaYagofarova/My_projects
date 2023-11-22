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


Мы видим, что к концу анализируемого периода маржинальность стала положительной, но существенно уменьшилось количество новых подписок, скорее всего это вызвано снижением затрат на маркетинг (возможно закончилась какая-то рекламная акция).  Большинство пользователей смотрят  от 3 до 12 фильмов, пик просмотров приходится на пт, сб и вс в вечернее время - если увеличится срок жизни пользователя на сайте, эти цифры тоже будут расти.

Небольшое количество просмотренных фильмов, скорее всего, связано с тем, что много пользователей оформили подписку не так давно, а смотрят фильмы чаще именно в выходные. 

Чтобы выйти на маржу уровня 25 % нужно снизить среднюю стоимость привлечения пользователя на 10 %. Повысив Retention на 20 % до уровня 96,72 % мы увеличим общее количество оплат за 6 месяцев, при этом, чтобы выйти на планируемую маржинальность, нужно увелиить на 10 % стоимость подписки и снизить объём скидок. 

При планировании маркетинговых мероприятий желательно проанализировать каналы привлечения подписчиков, учитывать часовые пояса и время просмотра.

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


<details>
  <summary>Нажмите, чтобы развернуть/свернуть код</summary>
  
```
-- Узнаем, когда была первая транзакция для каждого студента. Начиная с этой даты, мы будем собирать его баланс уроков. 
with first_payment as (
select user_id
     , date_trunc ('day',transaction_datetime::date) as first_payment_date
from (select *
              , row_number () over (partition by user_id order by transaction_datetime) as rn
      from skyeng_db.payments
      where status_name = 'success') a
where rn = 1
),
-- Соберем таблицу с датами за каждый календарный день 2016 года. 
all_dates as (
    select distinct date_trunc ('day', class_start_datetime) as dt
    from skyeng_db.classes
    where class_start_datetime between '2016.01.01' and '2016.12.31'
),
-- Узнаем, за какие даты имеет смысл собирать баланс для каждого студента. 
all_dates_by_user as (
select user_id
     , dt
from first_payment b
   join all_dates c on b.first_payment_date <= c.dt
order by user_id, dt
),
-- Найдем все изменения балансов, связанные с успешными транзакциями. 
payments_by_dates as (
select user_id
     , date_trunc ('day', transaction_datetime) as payment_date
     , sum (classes) as transaction_balance_change
from skyeng_db.payments
where status_name = 'success'
group by 1, 2 
),
-- Найдем баланс студентов, который сформирован только транзакциями.
payments_by_dates_cumsum as (
select d.user_id
     , d.dt
     , f.transaction_balance_change
     , sum (transaction_balance_change) over (partition by d.user_id order by d.dt rows between unbounded preceding and current row) as transaction_balance_change_cs
from all_dates_by_user d 
    left join payments_by_dates f on d.user_id = f.user_id and d.dt = f.payment_date
),
-- Найдем изменения балансов из-за прохождения уроков. 
classes_by_dates as (
select user_id
     , date_trunc('day', class_start_datetime) as class_date
     , count (id_class)*-1 as classes
from skyeng_db.classes
where class_status in ('success', 'failed_by_student') 
group by 1, 2
 ),
-- По аналогии с уже проделанным шагом для оплат создадим CTE для хранения кумулятивной суммы количества пройденных уроков. 
classes_by_dates_dates_cumsum as (
select g.user_id
     , g.dt
     , h.classes
     , sum (case when h.classes is null then 0 else h.classes end) over (partition by g.user_id order by g.dt rows between unbounded preceding and current row) as classes_cs
from all_dates_by_user g 
     left join classes_by_dates h on g.user_id = h.user_id and g.dt = h.class_date
),
-- Создадим CTE balances с вычисленными балансами каждого студента. 
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
-- Посмотрим, как менялось общее количество уроков на балансах студентов.
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
</details>


Визуализация: 

![Изменение балансов студентов](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%B2%D0%B8%D0%B7%D1%83%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F%20%D0%B1%D0%B0%D0%BB%D0%B0%D0%BD%D1%81%D1%8B%20%D1%81%D1%82%D1%83%D0%B4%D0%B5%D0%BD%D1%82%D0%BE%D0%B2.png) 

Больше всего уроков пользователи проходят начиная с сентября, летом учатся мало. Начисление уроков проходит практически равномерно в течении учебного года, летом так же наблюдается не большой спад. 
Общий баланс растет планомерно, если посмотреть на кумулятивную сумму начисленных уроков и пройденных уроков, то видно, что они также растут, причем начисления (транзакции) растут быстрее, чем проходится уроков, это может говорить о том, что продажи повышаются либо клиенты остаются с нами (нужно детально изучать откуда приходят транзакции: от новых студентов или старых)

Если посмотреть на разбивку списаний уроков и оплаты (начисление уроков), то видно, что летом было затишье, все держалось на одном уровке, в сентябре пошел рост (объяснимо сезонностью). 
В целом можно заметить регулярность платежей и прохождения уроков.

## Проект третий

### Анализ АБ-Теста, проведенного во всех городах присутствия сети с помощью Python

Цель теста - исследование альтернативного метода воздействия на клиентские покупки с помощью пуш-уведомлений (вместо смс-уведомлений)

Таргет-метрики:

- Конверсия из рекламы в покупку
- Средний чек

##### Поставленные задачи: 

- проанализировать и визуализировать результаты,
- провести сегментацию,
- сделать выводы и сформулировать рекомендации для дальнейших запусков АБ Теста,
- построить таблицу, которая будет в удобной форме хранить результаты АБ Теста,
- на основании Excel-файла построить калькулятор.

  ##### В результате:

- Проведён анализ и объединение таблиц, искючены строки с нулловыми значениями, где необходимо проставлены значения и исследованы торговые точки в городах. Результат группировки торговых точек представлен в виде графика: 

![Группировка торговых точек по количеству в каждом городе](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D1%82%D0%BE%D1%80%D0%B3%D0%BE%D0%B2%D1%8B%D0%B5%20%D1%82%D0%BE%D1%87%D0%BA%D0%B8.png) 

- Создана функция test_calc, которая вычисляет значение t-критерия (критерия Стьюдента) и p_value и, для сравнения средних с помощью функции print выводит сообщение о том, существует ли разница между средними 

<details>
  <summary>Развернуть/свернуть код</summary>
  
```
def test_calc(r1, r2, alpha = 0.05):
    
    s,p = ttest_ind(r1,r2)
    
    if p < alpha:
        print("Гипотеза H0 не подтверждается: средние не равны")
    else:
        print("Гипотеза H0 подтверждается: средние равны")
    
    print("Среднее значение 1 ряда", r1.mean())
    print("Среднее значение 2 ряда", r2.mean())
    print("Разница средних = ", r1.mean()-r2.mean())
    print("P_value = ",p)
    return s, p
```
</details>
    
- Создана функция mann_whitney_func, которая будет рассчитывает значение критерия Манна Уитни и p_value и, для сравнения распределений и с помощью функции print выводит сообщение о том, существует ли разница между средними.

<details>
  <summary>Развернуть/свернуть код</summary> 
  
  ```
  def mann_whitney_func(r1, r2, alpha=.05):
    
    s, p = mannwhitneyu(r1, r2)
    
    if p < alpha:
        print('Распределения не равны')
    else:
        print('Распределения равны')
    
    print("P_value = ",p)
    return s, p
  ```
  </details> 

- Для наглядности создана визуализация платежей в контрольной и тестовой группах и выполнено сравнение платежей и конверсии в платеж для формирования результатов АБ теста

 ![](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%BF%D0%BB%D0%B0%D1%82%D0%B5%D0%B6%D0%B8.png) 
 
 ![](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D1%81%D1%82%D0%B0%D1%82.png) 

Из произведенных рассчетов сделан вывод о том, что обе таргет-метрики тестового воздействия (показа пуш уведомления) выигрывают, то есть приносят большую конверсию и средний чек. 
Однако, для полноты сведений и принятия окончательного решения по результату проведенного теста необходимо провести сегментацию по городам. 

Отдельно были рассмотрены и визуализированы Москва и Санкт-Петербург, затем создан список остальных городов и запущен цикл для проверки показателей каждого города: 
<details>
  <summary>Развернуть/свернуть код</summary> 

```
print(f't-test платежи')
test_calc(df_msk[df_msk['nflag_test']==1]['amt_payment'], df_msk[df_msk['nflag_test']==0]['amt_payment'])
print()
print(f't-test конверсия')
test_calc(df_msk[df_msk['nflag_test']==1]['payflag'],df_msk[df_msk['nflag_test']==0]['payflag'])
print()
print(f'тест Манна Уитни')
mann_whitney_func(df_msk[df_msk['nflag_test']==1]['amt_payment'], df_msk[df_msk['nflag_test']==0]['amt_payment'])
```
 </details>

<details>
  <summary>Развернуть/свернуть код</summary> 

```
for i in df_other_cityes['city'].unique():
    print (i)
    print()
    city_df = df_other_cityes[(df_other_cityes['city'] == i)]
    sns.histplot(data=city_df, x='amt_payment', hue='nflag_test')
    plt.show()
    
    print(f't-test платежи')
    test_calc(city_df[city_df['nflag_test']==1]['amt_payment'], 
              city_df[city_df['nflag_test']==0]['amt_payment'])
    print()
    print(f't-test конверсия')
    test_calc(city_df[city_df['nflag_test']==1]['payflag'],
              city_df[city_df['nflag_test']==0]['payflag'])
    print()
    print(f'тест Манна Уитни')
    mann_whitney_func(city_df[city_df['nflag_test']==1]['amt_payment'], 
                      city_df[city_df['nflag_test']==0]['amt_payment'])
    print()

```
 </details> 

 Посчитав статистические метрики по всем городам, сделан вывод,что средние не равны только в Москве, Санкт-Петергбурге, Волгограде, Владимире и Самаре. В остальных городах разницы нет, тестовая группа не отреагировала на пуш-уведомления повышением среднего чека и конверсии. 

 В завершение создана таблица excel и калькулятор 
 
<details>
  <summary>Развернуть/свернуть код</summary> 

 ```
df_result = pd.DataFrame()

for i in df_SkyLenta_clients_final['city'].unique():
    df_loc = df_SkyLenta_clients_final[df_SkyLenta_clients_final['city']==i]
    print(i)
    for j in df_loc['id_trading_point'].unique():
        
        df_loc_tr = df_loc[df_loc['id_trading_point']==j]
   
        count_test = df_loc_tr[df_loc_tr['nflag_test'] == 1]['nflag_test'].count()
        count_control = df_loc_tr[df_loc_tr['nflag_test'] == 0]['nflag_test'].count()
        count_all = df_loc_tr['nflag_test'].count()

        
        avg_payment_test = df_loc_tr[df_loc_tr['nflag_test']==1]['amt_payment'].mean()
        avg_payment_control = df_loc_tr[df_loc_tr['nflag_test']==0]['amt_payment'].mean()
        diff = avg_payment_test - avg_payment_control
        
        sigma_test = df_loc_tr[df_loc_tr['nflag_test']==1]['amt_payment'].std()
        sigma_control = df_loc_tr[df_loc_tr['nflag_test']==0]['amt_payment'].std()
        
        s_amt, p_amt = test_calc(df_loc_tr[df_loc_tr['nflag_test']==1]['amt_payment'],
                                 df_loc_tr[df_loc_tr['nflag_test']==0]['amt_payment'])
        s_conv, p_conv = test_calc(df_loc_tr[df_loc_tr['nflag_test']==1]['payflag'],
                                   df_loc_tr[df_loc_tr['nflag_test']==0]['payflag'])
        
        m_s, m_p = mann_whitney_func(df_loc_tr[df_loc_tr['nflag_test']==1]['amt_payment'], 
                      df_loc_tr[df_loc_tr['nflag_test']==0]['amt_payment'])
        
        df_result = df_result.append ({'city':i, 'id_trading_point':j, 
                                      'count_test':count_test, 'count_control': count_control, 'count_all': count_all,
                                      'avg_payment_test': avg_payment_test, 'avg_payment_control': avg_payment_control,
                                      'diff': diff, 'sigma_test': sigma_test, 'sigma_control': sigma_control, 
                                      'ttest_amt_s': s_amt, 'ttest_amt_p': p_amt,  
                                      'ttest_conv_s': s_conv, 'ttest_conv_p': p_conv, 
                                      'mann_whitney_s': m_s, 'mann_whitney_p': m_p,}, ignore_index = True)

df_result['percent_count'] = (df_result['count_all']/df_result['count_all'].sum())

df_result['nflag_diff'] = np.where((df_result['ttest_amt_p']<.05) & (df_result['diff']>0), 'Positive', 
                                  np.where((df_result['ttest_amt_p']>.05) & (df_result['diff']<0), 'Negative', 'No diff'))
```
 </details> 

Калькулятор считает следующие показатели: 

1) При положительных исходах: внешним параметром принимаем количество потенциальных клиентов-покупателей за период времени N. 
Опираясь на поля diff, а также на долю покупателей в этой торговой точке среди всех, определяем, какая выгода может быть получена от замены механики A на B при условии, что ей воспользуются N клиентов.


![Калькулятор положительных исходов](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%BA%D0%B0%D0%BB%D1%8C%D0%BA%20%D0%BF%D0%BE%D0%B7%D0%B8%D1%82%D0%B8%D0%B2.png)


2) При отрицательных исходах
Конфигурация калькулятора такая же, как на положительных исходах. Рассчитываем, какая сумма может быть потеряна, если мы заменим механику А на механику В.


![Калькулятор отрицательных исходов](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%BA%D0%B0%D0%BB%D1%8C%D0%BA%20%D0%BD%D0%B5%D0%B3%D0%B0%D1%82%D0%B8%D0%B2.png)


3) При нейтральных исходах предполагается, что результаты на эти торговых точках не видны, так как собрано слишком мало наблюдений. Необходимо понять, какое количество наблюдений потребуется в каждой из этих торговых точек, чтобы разглядеть определенную разницу в средних платежах.
Внешним параметром принимаем MDE - сумма в рублях (разница между средними платежами в группах), которая считается минимально значимой с точки зрения бизнеса.
Для каждой торговой точки рассчитывается количество наблюдений, необходимое для обнаружения разницы масштаба MDE.


![Калькулятор нейтральных исходов](https://github.com/YunonaYagofarova/YunonaYagofarova/blob/main/%D0%BA%D0%B0%D0%BB%D1%8C%D0%BA%20%D0%BD%D0%B5%D0%B9%D1%82%D1%80%D0%B0%D0%BB%D1%8C.png) 

