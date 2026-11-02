# Exoplanet Detector

Данный репозиторий представляет из себя проект машинного обучения по анализу данных NASA Kepler и исследованию обнаружения экзопланет по данным о звёздах и их кривым блеска.

> **Текущий этап:** vetting - классификация уже обнаруженных транзитоподобных событий (KOI) как Planetary Candidate (PC/CANDIDATE) или False Positive (FP).
> **Дальнейшая цель:** detection - поиск транзитных сигналов в кривых блеска, включая звёзды без заранее выделенного события.

## Содержание

| Раздел                                               | Описание                                              |
| ---------------------------------------------------- | ----------------------------------------------------- |
| [Research conclusions](docs/research_conclusions.md) | Итоги предыдущих экспериментов, ограничения и решения |
| [Dataset and audit](docs/dataset.md)          | Источники, построение датасета и аудит признаков      |
| [LightGBM](docs/lightgbm.md)                  | Табличная модель                                      |
| [CNN](docs/cnn.md)                            | Анализ кривых блеска с помощью 1D-CNN                 |
| [Ensemble](ensemble.md)                  | Объединение LightGBM и CNN                            |

## 1. Цель проекта

Изначальная идея - система, оценивающая вероятность того, что транзитоподобное событие связано с планетным кандидатом:

1. **LightGBM** анализирует табличные физические характеристики звезды и системы.
2. **1D-CNN** анализирует кривую блеска - изменение яркости звезды во времени.
3. **Ensemble / stacking** объединяет предсказания обеих моделей.

Желанный конечный результат - полностью работающий честный гетерогенный ансамбль моделей, способный обрабатывать разные по природе данные (физ. показатели и блеск) и делающий честный вывод на их основе.

### Этап A - Vetting (текущий)

Классификация уже обнаруженных транзитоподобных событий KOI:

- `CANDIDATE` / `PC` - положительный класс;
- `FALSE POSITIVE` / `FP` - отрицательный класс.

Вопрос этапа: **насколько уже обнаруженный транзитоподобный сигнал похож на планетный кандидат, а не на false positive?**

KOI уже прошли предварительный поиск транзитоподобных событий. Это не обнаружение транзита у произвольной звезды.

### Этап B - Detection (будущее расширение)

Поиск транзитных сигналов в кривых блеска, включая звёзды, для которых сигнал заранее не выделен.

Этап B потребует другой постановки задачи, отрицательных примеров и оценки полноты обнаружения.

## 2. Архитектура

```text
            NASA Kepler data
                    |
       +-------------------------+
       |                         |
Tabular / stellar features    Stellar light curves
       |                         |
    LightGBM                    1D-CNN
       |                         |
       +-----------+-------------+
                   |
             Ensemble / stacking
                   |
           Final candidate score
```

На этапе A обе ветви должны использовать согласованные KOI, labels и разбиения. В LightGBM нельзя включать признаки, напрямую раскрывающие target или результат Robovetter.

## 3. Источники данных

| Источник                                                                                                           | Назначение                                 |
| ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------ |
| [NASA Exoplanet Archive - KOI documentation](https://exoplanetarchive.ipac.caltech.edu/docs/Kepler_KOI_docs.html)  | Описание KOI и релизов                     |
| [Purpose of KOI Tables](https://exoplanetarchive.ipac.caltech.edu/docs/PurposeOfKOITable.html)                     | Различия Cumulative, DR25 и Supplemental   |
| [Kepler Data Products Overview](https://exoplanetarchive.ipac.caltech.edu/docs/Kepler_Data_Products_Overview.html) | Обзор продуктов Kepler                     |
| [Kepler Mission products](https://exoplanetarchive.ipac.caltech.edu/docs/KeplerMission.html)                       | Продукты миссии                            |
| [NASA Exoplanet Archive TAP](https://exoplanetarchive.ipac.caltech.edu/docs/TAP/usingTAP.html)                     | Получение данных через TAP/API             |
| [DR25 DOI](https://doi.org/10.26133/NEA5)                                                                          | Идентификатор релиза DR25                  |
| [MAST Kepler Archive](https://archive.stsci.edu/kepler/)                                                           | Архив кривых блеска и продуктов наблюдений |
| [NASA Exoplanet Archive FAQ](https://exoplanetarchive.ipac.caltech.edu/docs/faq.html)                              | Термины TCE и KOI                          |

### Почему не Cumulative KOI

NASA указывает, что Cumulative KOI объединяет данные разных KOI activity tables. Disposition и параметры транзита для одного объекта могут происходить из разных релизов. Таблица неоднородна и не предназначена для статистических исследований, которым нужна uniform population.

Для нового baseline планируется использовать конкретную Q1–Q17 DR25 KOI table. DR25 создавался с унифицированным Robovetter и ориентирован на статистическую однородность. При этом каталожная disposition не является безусловной физической истиной для каждого объекта.

## 4. Результаты

Итоги предыдущих, сквозных, неактуальных и остальных не итоговых экспериментов и исходящие из них ограничения описаны в [Research conclusions](docs/research_conclusions.md).

## 5. Структура репозитория

```text
exoplanet-detector/
├── README.md
├── docs/    # документация каждой из частей проекта
│   ├── research_conclusions.md
│   ├── dataset.md
│   ├── lightgbm.md
│   ├── cnn.md
│   └── ensemble.md
├── notebooks/
|   └── research/    # сквозные выводы (не для запуска)
├── legacy/
│   ├── notebooks_old/      # не для запуска: только просмотр
│   └── data_old/      # старые/неактуальные результаты/датасеты
└── data/
    ├── raw/    # начальные данные, не изменяются
    └── processed/    # сквозные данные
```

## 6. Терминология

- **KOI** - Kepler Object of Interest, объект с транзитоподобным событием, выделенный для дальнейшего vetting.
- **TCE** - Threshold Crossing Event, событие, найденное pipeline Kepler.
- **PC / CANDIDATE** - планетный кандидат в каталоге.
- **FP** - False Positive.
- **Vetting** - проверка уже обнаруженного события.
- **Detection** - поиск события в кривой блеска.
- **Light curve** - временной ряд измерений яркости.
- **Robovetter** - автоматизированная процедура vetting Kepler.
