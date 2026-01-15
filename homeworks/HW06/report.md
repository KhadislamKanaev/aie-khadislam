# HW06 – Report

> Файл: `homeworks/HW06/report.md`  
> Важно: не меняйте названия разделов (заголовков). Заполняйте текстом и/или вставляйте результаты.

## 1. Dataset

- Какой датасет выбран: `S06-hw-dataset-04.csv`.
- Размер: (25000, 62).
- Целевая переменная: `target`.
  - Доли классов: class 0 — 0.9508, class 1 — 0.0492 (сильный дисбаланс).
- Признаки: табличные признаки + столбец `id` (удалялся перед обучением); точные типы (число/категории) не фиксировал.

## 2. Protocol

- Разбиение: train/test (доли и `random_state` не зафиксировал; разбиение делалось один раз перед подбором).
- Подбор: GridSearchCV на train; подбирались гиперпараметры дерева/лесa/бустинга (см. `homeworks/HW06/artifacts/search_summaries.json`).
- Метрики:
  - accuracy — общая доля правильных, но при дисбалансе может быть завышенной.
  - F1 — полезна при дисбалансе и фиксированном пороге.
  - ROC-AUC — качество ранжирования вероятностей по всем порогам.
  - PR-AUC — основная метрика для этой задачи из-за сильного дисбаланса (ориентируется на качество положительного класса).

## 3. Models

Сравнивались модели и подбирались параметры:

- DummyClassifier (baseline): запускался по заданию (финальные метрики в артефактах не сохранены).
- LogisticRegression (baseline из S05): запускалась по заданию (финальные метрики в артефактах не сохранены).
- DecisionTreeClassifier:
  - Подбор: `ccp_alpha=0.001`, `max_depth=5`, `min_samples_leaf=20`.
- RandomForestClassifier:
  - Подбор: `n_estimators=200`, `max_depth=10`, `min_samples_leaf=10`.
- GradientBoostingClassifier (gb):
  - Подбор: `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`.

## 4. Results

Финальные метрики на test (из `homeworks/HW06/artifacts/metrics_test.json`):

| Model | Accuracy (test) | F1 (test) | ROC-AUC (test) | PR-AUC (test) |
|---|---:|---:|---:|---:|
| DecisionTree (ccp_alpha=0.001, max_depth=5, min_samples_leaf=20) | 0.8816 | 0.3739 | 0.8340 | 0.3310 |
| RandomForest (n_estimators=200, max_depth=10, min_samples_leaf=10) | 0.9768 | 0.7406 | 0.9003 | 0.7597 |
| GradientBoosting (n_estimators=100, learning_rate=0.1, max_depth=3) | 0.9736 | 0.6525 | 0.8966 | 0.7322 |

- Победитель: RandomForest.
- Критерий: PR-AUC (0.7597) и также лучший F1 (0.7406) среди сохранённых моделей.
- Лучшие параметры победителя: `max_depth=10`, `min_samples_leaf=10`, `n_estimators=200` (см. `homeworks/HW06/artifacts/best_model_meta.json`).

## 5. Analysis

- Устойчивость: отдельные 5 прогонов с разными `random_state` не зафиксировал (в реальной задаче это важно для оценки стабильности).
- Ошибки: confusion matrix для лучшей модели не сохранил в артефакты.
- Интерпретация: permutation importance (top-10/15) не сохранил в артефакты.

## 6. Conclusion

- В задаче с сильным дисбалансом PR-AUC и F1 более информативны, чем accuracy.
- Ансамбли (RandomForest/GradientBoosting) заметно сильнее одиночного дерева по PR-AUC и F1.
- RandomForest показал лучший итог: PR-AUC=0.7597 и F1=0.7406 на test.
- Подбор гиперпараметров (depth, min_samples_leaf, n_estimators) существенно влияет на качество и переобучение.
- Честный протокол: подбор на train через CV и финальная оценка на test один раз.
