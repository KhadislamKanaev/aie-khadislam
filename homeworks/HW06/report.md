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

- Разбиение: train/test = 80/20, `random_state=42`, `stratify=y`.
- Подбор: GridSearchCV на train; подбирались гиперпараметры дерева/лесa/бустинга (см. `homeworks/HW06/artifacts/search_summaries.json`).
- Метрики:
  - accuracy — общая доля правильных, но при дисбалансе может быть завышенной.
  - F1 — полезна при дисбалансе и фиксированном пороге.
  - ROC-AUC — качество ранжирования вероятностей по всем порогам.
  - PR-AUC — основная метрика для этой задачи из-за сильного дисбаланса (ориентируется на качество положительного класса).

## 3. Models

Сравнивались модели и подбирались параметры (GridSearchCV на train, refit по PR-AUC):

- DummyClassifier: `strategy="stratified"`.
- LogisticRegression: Pipeline(StandardScaler + LogisticRegression), подбор `C ∈ {1.0, 10.0}`.
- DecisionTreeClassifier: подбор `max_depth ∈ {3, None}`, `min_samples_leaf ∈ {1, 10}`.
- RandomForestClassifier: подбор `n_estimators ∈ {100, 200}`, `max_depth ∈ {10, None}`, `min_samples_leaf ∈ {1, 10}`.
- GradientBoostingClassifier: подбор `n_estimators ∈ {100, 200}`, `learning_rate=0.1`, `max_depth ∈ {2, 3}`.

## 4. Results

Финальные метрики (сохранены в `metrics_test.json` и `best_model_meta.json`):

| Model              | Accuracy | F1     | ROC-AUC | PR-AUC |
|--------------------|----------|--------|---------|--------|
| Dummy             | 0.9069  | 0.0472 | 0.5032  | 0.0497 |
| LogisticRegression| 0.9626  | 0.4038 | 0.8225  | 0.4966 |
| DecisionTree      | 0.9641  | 0.5517 | 0.7748  | 0.4809 |
| **RandomForest**  | **0.9738** | **0.6391** | **0.9016** | **0.7688** |
| GradientBoosting  | 0.9710  | 0.6103 | 0.8861  | 0.6753 |


- Победитель: RandomForest.
- Критерий выбора: PR-AUC (test) = 0.7688.
- Лучшие параметры победителя: `n_estimators=100`, `max_depth=None`, `min_samples_leaf=1`.

## 5. Analysis

- Ошибки: confusion matrix для лучшей модели сохранён в `cm.jpg`.
- Интерпретация: permutation importance (top-10) сохранён в `top_features.csv`.
- RF overfitting: CV F1=0.539 → Test F1=0.639 (Δ=0.10)
→ Нужно больше регуляризации (max_depth<20, min_samples_leaf>5)


## 6. Conclusion

- В задаче с сильным дисбалансом PR-AUC и F1 более информативны, чем accuracy.
- Ансамбли (RandomForest/GradientBoosting) заметно сильнее одиночного дерева по PR-AUC и F1.
- RandomForest показал лучший итог: PR-AUC=0.7597 и F1=0.7406 на test.
- Подбор гиперпараметров (depth, min_samples_leaf, n_estimators) существенно влияет на качество и переобучение.
- Честный протокол: подбор на train через CV и финальная оценка на test один раз.
