---
layout: case
title: "Очистка, разведывательный анализ, подготовка к загрузке данных в модель: кейс по ETL"
date: 2024-05-02
categories: [data-cleaning, web-analytics]
tags: [pandas, python, marketing, portfolio, car-sales]
description: >
  Очистка и структурирование веб-данных для анализа поведения пользователей на сайте автомобильного дилера. Подготовка признаков, извлечение модели авто, обработка пропусков и типизация.
---

# Очистка и подготовка веб-данных для модели. Кейс по ETL

<p>
  <strong> Полная версия блокнота с кодом в репозитории GitHub для уточнения технических деталей:</strong><br>  
  <a href="https://github.com/matejkoas/sberauto/blob/master/sberauto_work.ipynb" class="btn btn-primary" target="_blank">
    Смотреть на GitHub
  </a>
</p>

## Контекст

У клиента — крупного автомобильного портала — стояла задача: разобраться в поведении пользователей на сайте и понять, какие модели авто их интересуют. Имеющиеся данные о сессиях и событиях были "сырыми" и требовали глубокой предобработки.

---

## Цель проекта

- Подготовка данных для построения воронки: визит → карточка авто → событие
- Извлечение модели авто из URL
- Очистка и типизация данных для дальнейшей загрузки в BI или ML-систему

---

## Что было сделано

### 1. Очистка данных

- Удалены дубликаты строк
- Колонка `event_value` и `device_model` удалены из-за 100% и 99% пропусков соответственно
- Пропуски в `utm_campaign`, `device_os`, `event_label` и других колонках заполнены **модой** или **медианой** в зависимости от типа

### 2. Преобразование URL в признак модели

- Из поля `hit_page_path` извлечена модель автомобиля
- Все параметры после `?` удалены
- Нелогичные и неинформативные пути заменены на `'other page'`

## Решение задач анализа данных

Для решения аналитических задач произведена дополнительная подготовка данных:

- Объединены датасеты `df_sessions` и `df_hits` по признаку `session_id`, чтобы связать источники трафика с пользовательскими действиями.
- Сгенерирован бинарный признак **"conversion"**:  
  `1`, если пользователь совершил целевое действие, и `0` — если нет.
- Сформирован признак **"канал"** со значениями:
  - `органический` — если источник трафика соответствует SEO или прямым заходам,
  - `платный` — если это реклама (например, utm_source содержит `yandex`, `google`, `cpc`).
- Добавлен признак **"город присутствия"**, где города сегментируются на:
  - `город присутствия` — если город входит в список целевых,
  - `другие` — все остальные.
- Выделены социальные сети в отдельный признак **"соцсети"**:
  - `соцсети` — если источник содержит `vk`, `facebook`, `instagram`, `ok`,
  - `другие` — все остальные источники.

> 📌 Эти признаки позволили провести более точный анализ каналов трафика, выявить сильные и слабые стороны рекламных кампаний и построить корректную воронку конверсии.

### Проверка гипотезы 1: влияет ли тип трафика на конверсию

📌 **Гипотеза:**  
Органический трафик не отличается от платного по коэффициенту конверсии (CR) в целевые события.

Для проверки этой гипотезы были использованы два статистических метода:

- **Z-тест для пропорций** — классический метод сравнения двух групп по доле успешных исходов;
- **Тест хи-квадрат (χ²)** — для оценки значимости различий между двумя категориальными признаками: `channel` и `conversion`.

🔍 **Результат**: различие между Conversion Rate органического и платного трафика оказалось статистически значимым. Это говорит о том, что источники трафика действительно по-разному влияют на вероятность конверсии.

#### Визуализация

Ниже представлено сравнение средних CR для двух типов трафика:

   <div class="case-image">
      <img src="{{ site.baseurl }}/assets/images/sber_compartion.png" alt="кейсы python" class="img-fluid w-50">
    </div>

> 📈 Вывод: органический и платный трафик следует анализировать и оптимизировать раздельно — с учетом их эффективности в достижении целевых действий.

### Гипотеза 2: Влияние типа устройства на Conversion Rate

**Формулировка гипотезы:**  
Трафик с мобильных устройств не отличается от трафика с десктопных устройств с точки зрения CR (Conversion Rate) в целевые события.

**Проверка гипотезы с помощью Z-теста:**  
Результаты теста показывают, что различие между CR мобильных и десктопных устройств статистически значимо.

**Проверка гипотезы методом Хи-квадрат:**  
Также подтверждает наличие статистически значимых различий между двумя типами трафика.

#### Визуализация
   <div class="case-image">
        <img src="{{ site.baseurl }}/assets/images/sber_devices.png" alt="кейсы python" class="img-fluid w-50">
    </div>

### Гипотеза 3: Влияние географии на Conversion Rate

**Формулировка гипотезы:**  
Трафик из городов присутствия (Москва и область, Санкт-Петербург) не отличается от трафика из иных регионов с точки зрения CR (Conversion Rate) в целевые события.

**Проверка гипотезы с помощью Z-теста:**  
Результаты показывают, что различие между CR в городах присутствия и остальных регионах статистически значимо.

**Проверка гипотезы методом Хи-квадрат:**  
Также подтверждает наличие статистически значимого различия между двумя географическими группами.

#### Визуализация
   <div class="case-image">
        <img src="{{ site.baseurl }}/assets/images/sber_regions.png" alt="кейсы python" class="img-fluid w-50">
    </div>

### Выводы по проверке гипотез

Все три гипотезы были проверены двумя разными методами с уровнем значимости 0,05, и получены следующие результаты:

#### 1. **Гипотеза: Органический трафик vs Платный трафик**
   - **Результат**: Различие между CR органического и CR платного трафика статистически значимо.
   - **Вывод**: Конверсия органического трафика выше, чем у платного. Это может говорить о более высоком качестве аудитории, привлеченной через органические каналы, или о большем доверии пользователей к не рекламированным ссылкам.

#### 2. **Гипотеза: Мобильный трафик vs Десктопный трафик**
   - **Результат**: Различие между CR мобильных и десктопных устройств статистически значимо.
   - **Вывод**: Мобильные пользователи демонстрируют более высокий коэффициент конверсии по сравнению с пользователями десктопных устройств. Это может быть связано с увеличением использования мобильных устройств для онлайн-покупок и удобством мобильных интерфейсов.

#### 3. **Гипотеза: Трафик из городов присутствия vs Трафик из других городов**
   - **Результат**: Различие между CR в городах присутствия (Москва и Санкт-Петербург) и других городах статистически значимо.
   - **Вывод**: Пользователи из крупных городов показывают более высокий коэффициент конверсии, чем пользователи из других регионов. Это может быть связано с локальными особенностями рынка, а также с более высокими затратами на маркетинг в крупных городах.

### Заключение

Анализ показал, что:

- Органический трафик имеет более высокий коэффициент конверсии по сравнению с платным.
- Мобильный трафик демонстрирует лучшие результаты в сравнении с десктопным.
- Регионы с высокой концентрацией пользователей (города присутствия) показывают более высокие показатели по конверсии.

Эти результаты могут быть полезными для дальнейшей оптимизации маркетинговых стратегий и более точного планирования рекламных кампаний.



