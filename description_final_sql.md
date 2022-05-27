

# **Итоговая работа**
### **Авиаперевозки**

![Авиаперевозки](https://github.com/Korchunov/Final_SQL/blob/master/DB/%D0%92%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5.PNG)

## Метаданные

* Тип подключения - **локальный**
* Версия - 1.0.0
* Краткое описание: 
База данных, содержащая информацию о пассажирских авиаперевозках

## Описание Базы Данных

В этом документе описана схема данных, состоящая из восьми таблиц и трёх представлений, одно из которых материализованное. В качестве предметной области выбраны авиаперевозки по России.

Сущности схемы данных:

* Table
  * bookings 
  * aircrafts 
  * airports 
  * boarding_passes 
  * flights 
  * seats 
  * ticket_flights 
  * tickets 
* View
  * bookings.flights_v 
  * view1 
* Materialized view
  * bookings.routes

Основной сущностью является бронирование (bookings). В одно бронирование можно включить несколько пассажиров, каждому из которых выписывается отдельный билет (tickets). Билет имеет уникальный номер и содержит информацию о пассажире. Как таковой пассажир не является отдельной сущностью. Как имя, так и номер документа пассажира могут меняться с течением времени, так что невозможно однозначно найти все билеты одного человека; для простоты можно считать, что все пассажиры уникальны. 

Билет включает один или несколько перелетов (ticket_flights). Несколько перелетов могут включаться в билет в случаях, когда нет нет прямого рейса, соединяющего пункты отправления и назначения (полет с пересадками), либо когда билет взят «туда и обратно». В схеме данных нет жесткого ограничения, но предполагается, что все билеты в одном бронировании имеют одинаковый набор перелетов. 

Каждый рейс (flights) следует из одного аэропорта (airports) в другой. Рейсы с одним номером имеют одинаковые пункты вылета и назначения, но будут отличаться датой отправления. 

При регистрации на рейс пассажиру выдается посадочный талон (boarding_passes), в котором указано место в самолете. Пассажир может зарегистрироваться только на тот рейс, который есть у него в билете. Комбинация рейса и места в самолете должна быть уникальной, чтобы не допустить выдачу двух посадочных талонов на одно место. 

Количество мест (seats) в самолете и их распределение по классам обслуживания зависит от модели самолета (aircrafts), выполняющего рейс. Предполагается, что каждая модель самолета имеет только одну компоновку салона. Схема данных не контролирует, что места в посадочных талонах соответствуют имеющимся в самолете 

Все зависимости и отношения данных сущностей друг к другу можно проследить на **диаграмме схемы данных**

![ER-диаграмма](https://github.com/Korchunov/Final_SQL/blob/master/DB/ER-%D0%B4%D0%B8%D0%B0%D0%B3%D1%80%D0%B0%D0%BC%D0%BC%D0%B0.PNG)

### Домен "**bookings**"

Пассажир заранее (book_date, максимум за месяц до рейса) бронирует билет себе и, возможно, нескольким другим пассажирам. Бронирование идентифицируется номером (book_ref, шестизначная комбинация букв и цифр). 

Поле total_amount хранит общую стоимость включенных в бронирование перелетов всех пассажиров.

**Объекты схемы**

 Столбец      | Тип           | Модификаторы | Описание  
 ----------   | --------      | ------------ | --------             
 book_ref     | char(6)       | NOT NULL     | Номер бронирования 
 book_date    | timestamptz   | NOT NULL     | Дата бронирования 
 total_amount | numeric(10,2) | NOT NULL     | Полная сумма бронирования 
 
> **Индексы:**  

`PRIMARY KEY, btree (book_ref)` 
 
> **Ссылки извне:**  

`TABLE "tickets" FOREIGN KEY (book_ref)
REFERENCES bookings(book_ref)`

---

### Домен "**aircrafts**"

Каждая модель воздушного судна идентифицируется своим трехзначным кодом (aircraft_code). Указывается также название модели (model) и максимальная дальность полета в километрах (range).

**Объекты схемы**

 Столбец       |   Тип   | Модификаторы |    Описание              
 -----------   | ------- | ------------ | --------- 
 aircraft_code | char(3) | NOT NULL     | Код самолета, IATA 
 model         | text    | NOT NULL     | Модель самолета 
 range         | integer | NOT NULL     | Максимальная дальность полета, км 

> **Индексы:**   

`PRIMARY KEY, btree (aircraft_code)` 

> **Ограничения-проверки:**    

`CHECK (range > 0)` 

> **Ссылки извне:**    

`TABLE "flights" FOREIGN KEY (aircraft_code)        
 REFERENCES aircrafts(aircraft_code)`    
`TABLE "seats" FOREIGN KEY (aircraft_code)        
REFERENCES aircrafts(aircraft_code) ON DELETE CASCADE`

  ---

### Домен "**airports**"

Аэропорт идентифицируется трехбуквенным кодом (airport_code) и имеет свое имя (airport_name). 

Для города не предусмотрено отдельной сущности, но название (city) указывается и может служить для того, чтобы определить аэропорты одного города. Также указывается широта (longitude), долгота (latitude) и часовой пояс (timezone).

**Объекты схемы**

 Столбец      |   Тип     | Модификаторы     |           Описание 
 ----------   | -------   | ------------     | ---------------           
 airport_code | char(3)   | NOT NULL         | Код аэропорта 
 airport_name | text      | NOT NULL         | Название аэропорта 
 city         | text      | NOT NULL         | Город 
 longitude    | float     | NOT NULL         | Координаты аэропорта: долгота 
 latitude     | float     | NOT NULL         | Координаты аэропорта: широта 
 timezone     | text      | NOT NULL         | Временная зона аэропорта 
 
 > **Индексы:**  

 `PRIMARY KEY, btree (airport_code)` 
 
 > **Ссылки извне:** 

 `TABLE "flights" FOREIGN KEY (arrival_airport)         
 REFERENCES airports(airport_code)`    
 `TABLE "flights" FOREIGN KEY (departure_airport)         
 REFERENCES airports(airport_code)`

 ---

### Домен "**boarding_passes**"

При регистрации на рейс, которая возможна за сутки до плановой даты отправления, пассажиру выдается посадочный талон. Он идентифицируется также, как и перелет — номером билета и номером рейса. 

Посадочным талонам присваиваются последовательные номера (boarding_no) в порядке регистрации пассажиров на рейс (этот номер будет уникальным только в пределах данного рейса). В посадочном талоне указывается номер места (seat_no).

**Объекты схемы**

 Столбец      |    Тип     | Модификаторы |         Описание     
 ---------    | ---------- | ------------ | ----------------    
  ticket_no   | char(13)   | NOT NULL     | Номер билета 
  flight_id   | integer    | NOT NULL     | Идентификатор рейса 
  boarding_no | integer    | NOT NULL     | Номер посадочного талона 
  seat_no     | varchar(4) | NOT NULL     | Номер места 
  
  > **Индексы:**  

  `PRIMARY KEY, btree (ticket_no, flight_id)`   
  `UNIQUE CONSTRAINT, btree (flight_id, boarding_no) `   
  `UNIQUE CONSTRAINT, btree (flight_id, seat_no) `
  
  > **Ограничения внешнего ключа:**

  `FOREIGN KEY (ticket_no, flight_id)         
  REFERENCES ticket_flights(ticket_no, flight_id)`

  ---

### Домен "**flights**"

Естественный ключ таблицы рейсов состоит из двух полей — номера рейса (flight_no) и даты отправления (scheduled_departure). Чтобы сделать внешние ключи на эту таблицу компактнее, в качестве первичного используется суррогатный ключ (flight_id). 

Рейс всегда соединяет две точки — аэропорты вылета (departure_airport) и прибытия (arrival_airport). Такое понятие, как «рейс с пересадками» отсутствует: если из одного аэропорта до другого нет прямого рейса, в билет просто включаются несколько необходимых рейсов. 

У каждого рейса есть запланированные дата и время вылета (scheduled_departure) и прибытия (scheduled_arrival). Реальные время вылета (actual_departure) и прибытия (actual_arrival) могут отличаться: обычно не сильно, но иногда и на несколько часов, если рейс задержан. 

Статус рейса (status) может принимать одно из следующих значений: 

* Scheduled Рейс доступен для бронирования. Это происходит за месяц до плановой даты вылета; до этого запись о рейсе не существует в базе данных. 

* On Time Рейс доступен для регистрации (за сутки до плановой даты вылета) и не задержан. 

* Delayed Рейс доступен для регистрации (за сутки до плановой даты вылета), но задержан. 

* Departed Самолет уже вылетел и находится в воздухе. 

* Arrived Самолет прибыл в пункт назначения. 

* Cancelled Рейс отменен.

**Объекты схемы**

 Столбец             |     Тип     | Модификаторы |          Описание   
 -------------       | ----------- | ------------ | -----------------        
 flight_id           | serial      | NOT NULL     | Идентификатор рейса 
 flight_no           | char(6)     | NOT NULL     | Номер рейса 
 scheduled_departure | timestamptz | NOT NULL     | Время вылета по расписанию 
 scheduled_arrival   | timestamptz | NOT NULL     | Время прилёта по расписанию 
 departure_airport   | char(3)     | NOT NULL     | Аэропорт отправления 
 arrival_airport     | char(3)     | NOT NULL     | Аэропорт прибытия 
 status              | varchar(20) | NOT NULL     | Статус рейса 
 aircraft_code       | char(3)     | NOT NULL     | Код самолета, IATA 
 actual_departure    | timestamptz |              | Фактическое время вылета 
 actual_arrival      | timestamptz |              | Фактическое время прилёта 
 
> **Индексы:** 

 `PRIMARY KEY, btree (flight_id)`    
 
 `UNIQUE CONSTRAINT, btree (flight_no,scheduled_departure)` 
  
> **Ограничения-проверки:**

`CHECK (scheduled_arrival > scheduled_departure)`    
`CHECK ((actual_arrival IS NULL)       OR  ((actual_departure IS NOT NULL AND actual_arrival IS NOT NULL)            AND (actual_arrival > actual_departure)))`   
`CHECK (status IN ('On Time', 'Delayed', 'Departed',                       'Arrived', 'Scheduled', 'Cancelled'))` 
  
>  **Ограничения внешнего ключа:** 

`FOREIGN KEY (aircraft_code)         
REFERENCES aircrafts(aircraft_code)`    
`FOREIGN KEY (arrival_airport)         
REFERENCES airports(airport_code)`    
`FOREIGN KEY (departure_airport)        
 REFERENCES airports(airport_code)` 

> **Ссылки извне:** 

`TABLE "ticket_flights" 
FOREIGN KEY (flight_id)         
REFERENCES flights(flight_id)`

---

### Домен "**seats**"

Места определяют схему салона каждой модели. Каждое место определяется своим номером (seat_no) и имеет закрепленный за ним класс обслуживания (fare_conditions) — Economy, Comfort или Business.

**Объекты схемы**

 Столбец         |     Тип     | Модификаторы |      Описание 
 -----------     | ----------- | ------------ | ----------     
 aircraft_code   | char(3)     | NOT NULL     | Код самолета, IATA 
 seat_no         | varchar(4)  | NOT NULL     | Номер места 
 fare_conditions | varchar(10) | NOT NULL     | Класс обслуживания 
 
> **Индексы:** 

`PRIMARY KEY, btree (aircraft_code, seat_no)` 

> **Ограничения-проверки:**  

`CHECK (fare_conditions IN ('Economy', 'Comfort', 'Business'))` 

> **Ограничения внешнего ключа:**    

`FOREIGN KEY (aircraft_code)        
 REFERENCES aircrafts(aircraft_code) ON DELETE CASCADE`

 ---

### Домен "**ticket_flights**"

Перелет соединяет билет с рейсом и идентифицируется их номерами. Для каждого перелета указываются его стоимость (amount) и класс обслуживания (fare_conditions).

**Объекты схемы**

 Столбец         |      Тип      | Модификаторы |      Описание       
 -----------     | ------------- | ------------ | -----------
 ticket_no       | char(13)      | NOT NULL     | Номер билета 
 flight_id       | integer       | NOT NULL     | Идентификатор рейса 
 fare_conditions | varchar(10)   | NOT NULL     | Класс обслуживания 
 amount          | numeric(10,2) | NOT NULL     | Стоимость перелета

> **Индексы:** 

`PRIMARY KEY, btree (ticket_no, flight_id)` 

> **Ограничения-проверки:**   

`CHECK (amount >= 0)`    
`CHECK (fare_conditions IN ('Economy', 'Comfort', 'Business'))` 

> **Ограничения внешнего ключа:**    

`FOREIGN KEY (flight_id) 
REFERENCES flights(flight_id)` 

`FOREIGN KEY (ticket_no) 
REFERENCES tickets(ticket_no)`

> **Ссылки извне:** 

`TABLE "boarding_passes" FOREIGN KEY (ticket_no, flight_id)         
REFERENCES ticket_flights(ticket_no, flight_id)`

---

### Домен "**tickets**"

Билет имеет уникальный номер (ticket_no), состоящий из 13 цифр. Билет содержит идентификатор пассажира (passenger_id) — номер документа, удостоверяющего личность, — его фамилию и имя (passenger_name) и контактную информацию (contact_date). Ни идентификатор пассажира, ни имя не являются постоянными (можно поменять паспорт, можно сменить фамилию), поэтому однозначно найти все билеты одного и того же пассажира невозможно.

**Объекты схемы**

 Столбец        |     Тип     | Модификаторы |     Описание 
 -----------    | ----------- | ------------ | -------------          
 ticket_no      | char(13)    | NOT NULL     | Номер билета 
 book_ref       | char(6)     | NOT NULL     | Номер бронирования 
 passenger_id   | varchar(20) | NOT NULL     | Идентификатор пассажира 
 passenger_name | text        | NOT NULL     | Имя пассажира 
 contact_data   | jsonb       |              | Контактные данные пассажира 
 
 > **Индексы:**   

   `PRIMARY KEY, btree (ticket_no)`
  
  > **Ограничения внешнего ключа:**    
  
  `FOREIGN KEY (book_ref) 
  REFERENCES bookings(book_ref)` 
  
  > **Ссылки извне:**   

  `TABLE "ticket_flights" 
  FOREIGN KEY (ticket_no) 
  REFERENCES tickets(ticket_no)`

  ---

### Представление "**bookings.flights_v**"

Над таблицей flights создано представление flights_v, содержащее дополнительную информацию: 

* расшифровку данных об аэропорте вылета (departure_airport, departure_airport_name, departure_city)
* расшифровку данных об аэропорте прибытия (arrival_airport, arrival_airport_name, arrival_city)
* местное время вылета (scheduled_departure_local, actual_departure_local)
* местное время прибытия (scheduled_arrival_local, actual_arrival_local)
* продолжительность полета (scheduled_duration, actual_duration)         

 Столбец                   |     Тип     |       Описание    
 ----------------          | ----------- | ---------------                     
 flight_id                 | integer     | Идентификатор рейса 
 flight_no                 | char(6)     | Номер рейса 
 scheduled_departure       | timestamptz | Время вылета по расписанию 
 scheduled_departure_local | timestamp   | Время вылета по расписанию, местное время в пункте отправления 
 scheduled_arrival         | timestamptz | Время прилёта по расписанию 
 scheduled_arrival_local   | timestamp   | Время прилёта по расписанию, местное время в пункте прибытия 
 scheduled_duration        | interval    | Планируемая продолжительность полета 
 departure_airport         | char(3)     | Код аэропорта отправления 
 departure_airport_name    | text        | Название аэропорта отправления 
 departure_city            | text        | Город отправления 
 arrival_airport           | char(3)     | Код аэропорта прибытия 
 arrival_airport_name      | text        | Название аэропорта прибытия 
 arrival_city              | text        | Город прибытия 
 status                    | varchar(20) | Статус рейса 
 aircraft_code             | char(3)     | Код самолета, IATA 
 actual_departure          | timestamptz | Фактическое время вылета 
 actual_departure_local    | timestamp   | Фактическое время вылета, местное время в пункте отправления 
 actual_arrival            | timestamptz | Фактическое время прилёта 
 actual_arrival_local      | timestamp   | Фактическое время прилёта, местное время в пункте прибытия 
 actual_duration           | interval    | Фактическая продолжительность полета

---

### Представление "**view1**"

Данное представление связывает домены "flights" и "airports". Созданное отношение позволяет нам получить информацию о городе отправления и городе прибытия по каждому рейсу исходя из того, в каких городах находятся данные аэропорты.

Столбец                |    Тип    |              Описание   
---------------        | --------- | ------------------------- 
city_departure         |    text   | Город отправления
city_arrival           |    text   | Город прибытия

> **Синтаксис**

`create view view1 as (
select a.city as city_departure, a2.city as city_arrival
from flights f 
join airports a on f.departure_airport = a.airport_code
join airports a2 on f.arrival_airport = a2.airport_code);`

---

### Материализованное представление "**bookings.routes**"

Таблица рейсов содержит избыточность: из нее можно было бы выделить информацию о маршруте (номер рейса, аэропорты отправления и назначения), которая не зависит от конкретных дат рейсов. Именно такая информация и составляет материализованное представление routes.       

Столбец                |    Тип    |              Описание   
---------------        | --------- | -------------------------           
flight_no              | char(6)   | Номер рейса 
departure_airport      | char(3)   | Код аэропорта отправления 
departure_airport_name | text      | Название аэропорта отправления
departure_city         | text      | Город отправления 
arrival_airport        | char(3)   | Код аэропорта прибытия 
arrival_airport_name   | text      | Название аэропорта прибытия 
arrival_city           | text      | Город прибытия 
aircraft_code          | char(3)   | Код самолета, IATA 
duration               | interval  | Продолжительность полета 
days_of_week           | integer[] | Дни недели, когда выполняются рейсы

