---
aliases: [Экспериментальный дизайн, Design of Experiments, Эксперименты]
tags: 
  - experimentation
  - testing
  - analytics
  - ci-cd
  - data-science
  - statistical-analysis
---

# Экспериментальный дизайн в CI/CD

## Обзор

Экспериментальный дизайн в CI/CD - это систематический подход к планированию, выполнению и анализу экспериментов в контексте непрерывной интеграции и доставки. Он обеспечивает научный метод для оценки изменений программного обеспечения, позволяя командам принимать обоснованные решения на основе данных, а не предположений. Экспериментальный дизайн включает в себя планирование A/B тестов, многовариантных тестов, анализ причинно-следственных связей и оптимизацию процессов доставки программного обеспечения.

В современных условиях, когда изменения программного обеспечения происходят часто и быстро, экспериментальный дизайн становится критически важным для обеспечения качества, производительности и удовлетворенности пользователей. Он позволяет безопасно тестировать гипотезы в продакшене с минимальным риском для пользователей.

## Основы экспериментального дизайна

### Что такое экспериментальный дизайн

Экспериментальный дизайн - это методология планирования и проведения экспериментов для получения надежных и достоверных результатов. В контексте CI/CD он включает:

- Определение целей эксперимента
- Планирование факторов и уровней
- Определение метрик успеха
- Управление случайностью и контролем
- Планирование размера выборки
- Анализ причинно-следственных связей

### Принципы экспериментального дизайна

#### Рандомизация
Случайное распределение участников эксперимента между контрольной и тестовой группами для устранения систематических смещений.

#### Репликация
Повторение эксперимента или использование достаточного размера выборки для обеспечения надежности результатов.

#### Локальный контроль
Управление факторами, которые могут повлиять на результаты эксперимента, путем контроля условий или статистического учета.

#### Балансировка
Распределение факторов таким образом, чтобы они были сбалансированы между группами, позволяя изолировать эффект отдельных факторов.

## Типы экспериментов в CI/CD

### A/B тестирование

A/B тестирование - это сравнение двух версий (A и B) одного и того же элемента для определения, какая из них работает лучше:

```python
# ab_test_design.py
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
from typing import Dict, List, Tuple
import random

class ABTestDesigner:
    def __init__(self):
        self.experiment_config = {}
        self.results = {}
    
    def design_ab_test(self, 
                      metric_name: str,
                      baseline_value: float,
                      expected_improvement: float,
                      power: float = 0.8,
                      significance_level: float = 0.05,
                      two_tailed: bool = True) -> Dict:
        """
        Дизайн A/B теста с расчетом необходимого размера выборки
        """
        # Расчет минимально детектируемого эффекта
        mde = expected_improvement / baseline_value  # Относительный эффект
        
        # Приблизительный расчет размера выборки для пропорций
        if metric_name in ['conversion_rate', 'click_through_rate']:
            sample_size = self._calculate_sample_size_proportion(
                baseline_value, mde, power, significance_level, two_tailed
            )
        else:
            sample_size = self._calculate_sample_size_mean(
                baseline_value, expected_improvement, power, significance_level, two_tailed
            )
        
        # Рекомендации по продолжительности
        estimated_duration = self._estimate_duration(sample_size)
        
        config = {
            'metric': metric_name,
            'baseline_value': baseline_value,
            'expected_improvement': expected_improvement,
            'minimum_detectable_effect': mde,
            'required_sample_size_per_group': int(sample_size),
            'total_required_sample_size': int(sample_size * 2),
            'power': power,
            'significance_level': significance_level,
            'estimated_duration_days': estimated_duration,
            'randomization_unit': 'user',
            'blocking_factors': ['region', 'device_type', 'traffic_source']
        }
        
        self.experiment_config = config
        return config
    
    def _calculate_sample_size_proportion(self, p1: float, mde: float, power: float, alpha: float, two_tailed: bool):
        """
        Расчет размера выборки для сравнения пропорций
        """
        p2 = p1 * (1 + mde)  # Ожидаемая пропорция в тестовой группе
        pooled_p = (p1 + p2) / 2
        
        # Z-значения для уровня значимости и мощности
        z_alpha = stats.norm.ppf(1 - alpha/2) if two_tailed else stats.norm.ppf(1 - alpha)
        z_beta = stats.norm.ppf(power)
        
        # Расчет размера выборки
        effect_size = (p1 - p2) / np.sqrt(pooled_p * (1 - pooled_p))
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return n
    
    def _calculate_sample_size_mean(self, mean1: float, expected_diff: float, power: float, alpha: float, two_tailed: bool):
        """
        Расчет размера выборки для сравнения средних
        """
        # Предполагаем стандартное отклонение как 10% от среднего
        std_dev = mean1 * 0.1
        
        # Z-значения
        z_alpha = stats.norm.ppf(1 - alpha/2) if two_tailed else stats.norm.ppf(1 - alpha)
        z_beta = stats.norm.ppf(power)
        
        # Расчет размера выборки
        effect_size = expected_diff / std_dev
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return n
    
    def _estimate_duration(self, sample_size: float) -> int:
        """
        Оценка продолжительности эксперимента в днях
        """
        # Предполагаем 1000 уникальных пользователей в день
        daily_users = 1000
        return int(np.ceil(sample_size / daily_users))
    
    def simulate_ab_test(self, n_control: int, n_treatment: int, 
                        control_conversion: float, treatment_conversion: float) -> Dict:
        """
        Симуляция A/B теста
        """
        # Генерация данных
        control_conversions = np.random.binomial(1, control_conversion, n_control)
        treatment_conversions = np.random.binomial(1, treatment_conversion, n_treatment)
        
        # Статистический анализ
        control_rate = np.mean(control_conversions)
        treatment_rate = np.mean(treatment_conversions)
        
        # T-тест
        t_stat, p_value = stats.ttest_ind(
            control_conversions, treatment_conversions, equal_var=False
        )
        
        # Доверительный интервал для разницы
        se_diff = np.sqrt(
            (control_rate * (1 - control_rate) / n_control) + 
            (treatment_rate * (1 - treatment_rate) / n_treatment)
        )
        ci_lower = (treatment_rate - control_rate) - 1.96 * se_diff
        ci_upper = (treatment_rate - control_rate) + 1.96 * se_diff
        
        results = {
            'control_group': {
                'size': n_control,
                'conversions': int(np.sum(control_conversions)),
                'conversion_rate': float(control_rate)
            },
            'treatment_group': {
                'size': n_treatment,
                'conversions': int(np.sum(treatment_conversions)),
                'conversion_rate': float(treatment_rate)
            },
            'statistical_test': {
                't_statistic': float(t_stat),
                'p_value': float(p_value),
                'significant': p_value < 0.05
            },
            'effect_size': {
                'absolute_difference': float(treatment_rate - control_rate),
                'relative_difference': float((treatment_rate - control_rate) / control_rate * 100),
                'confidence_interval': (float(ci_lower), float(ci_upper))
            }
        }
        
        self.results = results
        return results

# Пример использования
def example_ab_test():
    designer = ABTestDesigner()
    
    # Дизайн A/B теста для улучшения конверсии
    config = designer.design_ab_test(
        metric_name='conversion_rate',
        baseline_value=0.05,  # 5% конверсия
        expected_improvement=0.005,  # Ожидаемое улучшение 0.5%
        power=0.8,
        significance_level=0.05
    )
    
    print("A/B Test Configuration:")
    for key, value in config.items():
        print(f"{key}: {value}")
    
    # Симуляция теста
    results = designer.simulate_ab_test(
        n_control=5000,
        n_treatment=5000,
        control_conversion=0.05,
        treatment_conversion=0.055
    )
    
    print("\nA/B Test Results:")
    print(f"Control Conversion Rate: {results['control_group']['conversion_rate']:.4f}")
    print(f"Treatment Conversion Rate: {results['treatment_group']['conversion_rate']:.4f}")
    print(f"Absolute Difference: {results['effect_size']['absolute_difference']:.4f}")
    print(f"Relative Difference: {results['effect_size']['relative_difference']:.2f}%")
    print(f"Statistically Significant: {results['statistical_test']['significant']}")
    print(f"P-value: {results['statistical_test']['p_value']:.4f}")
    
    return designer

if __name__ == "__main__":
    example_ab_test()
```

### Многовариантное тестирование (A/B/n)

Тестирование нескольких вариантов одновременно:

```python
# multivariate_test.py
import numpy as np
import pandas as pd
from scipy.stats import chi2
from itertools import combinations
import matplotlib.pyplot as plt

class MultivariateTestDesigner:
    def __init__(self):
        self.experiment_factors = []
        self.factor_levels = {}
        self.results = {}
    
    def define_experiment_factors(self, factors: Dict[str, List]) -> 'MultivariateTestDesigner':
        """
        Определение факторов эксперимента и их уровней
        """
        self.factor_levels = factors
        self.experiment_factors = list(factors.keys())
        
        # Расчет общего количества комбинаций
        total_combinations = 1
        for levels in factors.values():
            total_combinations *= len(levels)
        
        print(f"Total experimental combinations: {total_combinations}")
        
        return self
    
    def calculate_sample_size_factorial(self, 
                                      baseline_metric: float,
                                      mde_per_factor: Dict[str, float],
                                      power: float = 0.8,
                                      alpha: float = 0.05) -> Dict:
        """
        Расчет размера выборки для факторного эксперимента
        """
        # Для каждого фактора рассчитываем необходимый размер выборки
        sample_sizes = {}
        for factor, levels in self.factor_levels.items():
            if factor in mde_per_factor:
                # Используем расчет для A/B теста для каждого фактора
                base_size = self._get_ab_sample_size(baseline_metric, mde_per_factor[factor])
                # Учитываем количество уровней фактора
                sample_sizes[factor] = base_size * len(levels)
        
        # Общий размер - максимальный из всех факторов
        total_sample_size = max(sample_sizes.values()) if sample_sizes else 1000
        
        return {
            'factor_sample_sizes': sample_sizes,
            'total_sample_size': total_sample_size,
            'combinations_per_factor': {f: len(levels) for f, levels in self.factor_levels.items()}
        }
    
    def _get_ab_sample_size(self, p1: float, mde: float) -> int:
        """
        Внутренний метод для расчета A/B размера выборки
        """
        p2 = p1 * (1 + mde)
        pooled_p = (p1 + p2) / 2
        
        z_alpha = 1.96  # 95% confidence
        z_beta = 0.84   # 80% power
        
        effect_size = (p1 - p2) / np.sqrt(pooled_p * (1 - pooled_p))
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return int(np.ceil(n))
    
    def generate_factorial_design(self, total_sample_size: int) -> pd.DataFrame:
        """
        Генерация факторного дизайна эксперимента
        """
        # Создание всех возможных комбинаций факторов
        from itertools import product
        
        factor_combinations = list(product(*self.factor_levels.values()))
        
        # Распределение участников по комбинациям
        participants_per_combination = total_sample_size // len(factor_combinations)
        remainder = total_sample_size % len(factor_combinations)
        
        design_data = []
        for i, combination in enumerate(factor_combinations):
            # Определяем размер выборки для этой комбинации
            size = participants_per_combination + (1 if i < remainder else 0)
            
            # Создаем записи для этой комбинации
            for j in range(size):
                record = {'participant_id': f'P_{i}_{j}'}
                for k, factor in enumerate(self.experiment_factors):
                    record[factor] = combination[k]
                design_data.append(record)
        
        return pd.DataFrame(design_data)
    
    def analyze_factorial_results(self, results_df: pd.DataFrame) -> Dict:
        """
        Анализ результатов факторного эксперимента
        """
        # Предполагаем, что результаты содержат столбцы факторов и метрику
        metric_col = [col for col in results_df.columns if col not in self.experiment_factors and col != 'participant_id'][0]
        
        # ANOVA анализ
        from statsmodels.stats.anova import anova_lm
        from statsmodels.formula.api import ols
        
        # Формирование формулы для ANOVA
        formula_parts = [metric_col]
        for factor in self.experiment_factors:
            formula_parts.append(f'C({factor})')
        
        formula = ' ~ '.join(formula_parts)
        
        # Выполнение ANOVA
        model = ols(formula, data=results_df).fit()
        anova_results = anova_lm(model, typ=2)
        
        # Анализ взаимодействий (для пар факторов)
        interaction_effects = {}
        for factor1, factor2 in combinations(self.experiment_factors, 2):
            interaction_formula = f'{metric_col} ~ C({factor1}) + C({factor2}) + C({factor1}):C({factor2})'
            interaction_model = ols(interaction_formula, data=results_df).fit()
            interaction_anova = anova_lm(interaction_model, typ=2)
            
            interaction_effects[f'{factor1}_x_{factor2}'] = {
                'f_statistic': float(interaction_anova.loc[f'C({factor1}):C({factor2})', 'F']),
                'p_value': float(interaction_anova.loc[f'C({factor1}):C({factor2})', 'PR(>F)']),
                'significant': interaction_anova.loc[f'C({factor1}):C({factor2})', 'PR(>F)'] < 0.05
            }
        
        return {
            'main_effects': anova_results,
            'interaction_effects': interaction_effects,
            'model_summary': model.summary()
        }

# Пример использования для тестирования элементов интерфейса
def example_multivariate_test():
    designer = MultivariateTestDesigner()
    
    # Определение факторов эксперимента
    factors = {
        'button_color': ['red', 'blue', 'green'],
        'button_size': ['small', 'medium', 'large'],
        'button_text': ['Buy', 'Purchase', 'Get']
    }
    
    designer.define_experiment_factors(factors)
    
    # Расчет размера выборки
    sample_size_config = designer.calculate_sample_size_factorial(
        baseline_metric=0.05,
        mde_per_factor={'button_color': 0.02, 'button_size': 0.01, 'button_text': 0.015}
    )
    
    print("Sample Size Configuration:")
    for key, value in sample_size_config.items():
        print(f"{key}: {value}")
    
    # Генерация дизайна эксперимента
    design_df = designer.generate_factorial_design(sample_size_config['total_sample_size'])
    print(f"\nGenerated design with {len(design_df)} participants")
    print(f"First 10 rows:\n{design_df.head(10)}")
    
    return designer

if __name__ == "__main__":
    example_multivariate_test()
```

### Факторные эксперименты

Систематическое тестирование комбинаций факторов:

```python
# factorial_experiment.py
import numpy as np
import pandas as pd
from scipy.stats import f
import itertools

class FactorialExperiment:
    def __init__(self, factors: Dict[str, List]):
        self.factors = factors
        self.factor_names = list(factors.keys())
        self.factor_levels = list(factors.values())
        self.design_matrix = None
        self.results = None
    
    def generate_full_factorial_design(self) -> pd.DataFrame:
        """
        Генерация полного факторного дизайна
        """
        # Создание всех комбинаций факторов
        factor_combinations = list(itertools.product(*self.factor_levels))
        
        # Создание DataFrame
        design_data = {}
        for i, factor_name in enumerate(self.factor_names):
            design_data[factor_name] = [combo[i] for combo in factor_combinations]
        
        # Добавляем номер эксперимента
        design_data['experiment_id'] = range(len(factor_combinations))
        
        self.design_matrix = pd.DataFrame(design_data)
        return self.design_matrix
    
    def generate_fractional_factorial_design(self, resolution: int = 4) -> pd.DataFrame:
        """
        Генерация дробного факторного дизайна
        """
        # Для простоты реализуем 2^k-1 дробный факторный дизайн
        # где k - количество факторов с 2 уровнями
        
        # Преобразуем факторы к 2-уровневому формату если возможно
        binary_factors = {}
        for name, levels in self.factors.items():
            if len(levels) >= 2:
                binary_factors[name] = levels[:2]  # Берем первые 2 уровня
        
        if len(binary_factors) < 2:
            raise ValueError("Need at least 2 factors with 2+ levels for fractional factorial")
        
        # Создаем 2^(k-1) дизайн
        k = len(binary_factors)
        n_runs = 2**(k-1)
        
        # Генерация основного блока
        main_effects = []
        for i in range(n_runs):
            run = []
            for j in range(k-1):
                # Определяем уровень для каждого фактора
                level = (i // (2**j)) % 2
                run.append(binary_factors[list(binary_factors.keys())[j]][level])
            main_effects.append(run)
        
        # Добавляем последний фактор как взаимодействие остальных
        last_factor = []
        for run in main_effects:
            # Для простоты - последний фактор как XOR первых (2-level)
            last_level = 0
            for factor_val in run:
                # Преобразуем в бинарное значение
                last_level ^= (0 if factor_val == binary_factors[list(binary_factors.keys())[0]][0] else 1)
            last_factor.append(binary_factors[list(binary_factors.keys())[-1]][last_level])
        
        # Создание DataFrame
        design_data = {}
        for i, (name, levels) in enumerate(binary_factors.items()):
            if i < k-1:
                design_data[name] = [run[i] for run in main_effects]
            else:
                design_data[name] = last_factor
        
        design_data['experiment_id'] = range(n_runs)
        
        self.design_matrix = pd.DataFrame(design_data)
        return self.design_matrix
    
    def analyze_main_effects(self, response_data: List[float]) -> Dict:
        """
        Анализ основных эффектов факторов
        """
        if self.design_matrix is None:
            raise ValueError("Design matrix not generated")
        
        if len(response_data) != len(self.design_matrix):
            raise ValueError("Response data length must match design matrix length")
        
        # Добавляем response данные к дизайну
        analysis_df = self.design_matrix.copy()
        analysis_df['response'] = response_data
        
        # Вычисление основных эффектов
        effects = {}
        for factor_name in self.factor_names:
            factor_effects = {}
            
            for level in self.factors[factor_name]:
                # Среднее значение response для этого уровня фактора
                level_response = analysis_df[analysis_df[factor_name] == level]['response'].mean()
                factor_effects[level] = float(level_response)
            
            # Вычисление эффекта как разницы между уровнями
            levels = list(factor_effects.keys())
            if len(levels) >= 2:
                effect_size = factor_effects[levels[0]] - factor_effects[levels[1]]
                effects[factor_name] = {
                    'level_means': factor_effects,
                    'effect_size': effect_size,
                    'max_response_level': max(factor_effects, key=factor_effects.get)
                }
        
        return effects
    
    def calculate_statistical_significance(self, response_data: List[float], alpha: float = 0.05) -> Dict:
        """
        Расчет статистической значимости эффектов
        """
        # Используем ANOVA для оценки значимости факторов
        from scipy.stats import f_oneway
        
        analysis_df = self.design_matrix.copy()
        analysis_df['response'] = response_data
        
        significance_results = {}
        
        for factor_name in self.factor_names:
            # Группировка данных по фактору
            groups = []
            for level in self.factors[factor_name]:
                group_data = analysis_df[analysis_df[factor_name] == level]['response'].values
                groups.append(group_data)
            
            # One-way ANOVA
            f_stat, p_value = f_oneway(*groups)
            
            significance_results[factor_name] = {
                'f_statistic': float(f_stat),
                'p_value': float(p_value),
                'significant': p_value < alpha,
                'eta_squared': self._calculate_eta_squared(analysis_df, factor_name, 'response')
            }
        
        return significance_results
    
    def _calculate_eta_squared(self, df: pd.DataFrame, factor_col: str, response_col: str) -> float:
        """
        Расчет eta-квадрат (мера эффекта для ANOVA)
        """
        # Общая сумма квадратов
        total_mean = df[response_col].mean()
        ss_total = ((df[response_col] - total_mean) ** 2).sum()
        
        # Сумма квадратов фактора
        factor_means = df.groupby(factor_col)[response_col].mean()
        n_per_group = df.groupby(factor_col).size()
        
        ss_factor = sum(n_per_group * (factor_means - total_mean) ** 2)
        
        return float(ss_factor / ss_total) if ss_total != 0 else 0

# Пример использования для оптимизации производительности
def example_factorial_experiment():
    # Факторы, влияющие на производительность API
    factors = {
        'cache_strategy': ['none', 'redis', 'memory'],
        'database_indexing': ['basic', 'optimized', 'none'],
        'connection_pooling': ['disabled', 'enabled_10', 'enabled_20'],
        'compression': ['none', 'gzip', 'brotli']
    }
    
    experiment = FactorialExperiment(factors)
    
    # Генерация полного факторного дизайна
    design = experiment.generate_full_factorial_design()
    print(f"Full factorial design: {len(design)} runs")
    print(f"First 10 runs:\n{design.head(10)}")
    
    # Симуляция результатов (время отклика в мс)
    np.random.seed(42)
    # Генерируем реалистичные данные с эффектами факторов
    response_data = []
    for _, row in design.iterrows():
        base_time = 200  # Базовое время
        time = base_time
        
        # Добавляем эффекты факторов
        if row['cache_strategy'] == 'redis':
            time *= 0.7  # Ускорение на 30%
        elif row['cache_strategy'] == 'memory':
            time *= 0.6  # Ускорение на 40%
        
        if row['database_indexing'] == 'optimized':
            time *= 0.8  # Ускорение на 20%
        elif row['database_indexing'] == 'none':
            time *= 1.5  # Замедление на 50%
        
        if row['connection_pooling'] == 'enabled_10':
            time *= 0.9  # Небольшое ускорение
        elif row['connection_pooling'] == 'enabled_20':
            time *= 0.85  # Больше ускорение
        
        if row['compression'] == 'gzip':
            time *= 0.95  # Небольшое ускорение за счет меньшего трафика
        elif row['compression'] == 'brotli':
            time *= 0.92  # Лучшая компрессия
        
        # Добавляем шум
        time += np.random.normal(0, 10)
        time = max(50, time)  # Минимальное время 50мс
        
        response_data.append(time)
    
    # Анализ основных эффектов
    effects = experiment.analyze_main_effects(response_data)
    print("\nMain Effects Analysis:")
    for factor, effect_data in effects.items():
        print(f"{factor}:")
        print(f"  Level means: {effect_data['level_means']}")
        print(f"  Effect size: {effect_data['effect_size']:.2f}")
        print(f"  Best level: {effect_data['max_response_level']}")
    
    # Статистический анализ
    significance = experiment.calculate_statistical_significance(response_data)
    print("\nStatistical Significance:")
    for factor, stats_data in significance.items():
        print(f"{factor}:")
        print(f"  F-statistic: {stats_data['f_statistic']:.4f}")
        print(f"  P-value: {stats_data['p_value']:.4f}")
        print(f"  Significant: {stats_data['significant']}")
        print(f"  Effect size (eta²): {stats_data['eta_squared']:.4f}")

if __name__ == "__main__":
    example_factorial_experiment()
```

## Планирование экспериментов

### Определение целей эксперимента

```python
# experiment_planning.py
from enum import Enum
from dataclasses import dataclass
from typing import List, Dict, Any, Optional
import json

class ExperimentType(Enum):
    A_B_TEST = "a_b_test"
    MULTI_VARIATE = "multivariate"
    FACTORIAL = "factorial"
    PILOT = "pilot"
    CANARY = "canary"

class MetricType(Enum):
    PRIMARY = "primary"
    SECONDARY = "secondary"
    DIAGNOSTIC = "diagnostic"
    RISK = "risk"

@dataclass
class ExperimentMetric:
    name: str
    description: str
    type: MetricType
    acceptable_risk: float = 0.05
    minimum_detectable_effect: float = 0.0
    baseline_value: Optional[float] = None

@dataclass
class ExperimentFactor:
    name: str
    description: str
    levels: List[Any]
    is_essential: bool = True

class ExperimentPlanner:
    def __init__(self):
        self.experiment_config = {}
        self.metrics = []
        self.factors = []
        self.constraints = []
    
    def define_experiment(self, 
                         name: str,
                         experiment_type: ExperimentType,
                         description: str,
                         hypothesis: str) -> 'ExperimentPlanner':
        """
        Определение основных параметров эксперимента
        """
        self.experiment_config = {
            'name': name,
            'type': experiment_type.value,
            'description': description,
            'hypothesis': hypothesis,
            'created_at': '2023-11-11T00:00:00Z',
            'status': 'planning'
        }
        return self
    
    def add_metric(self, metric: ExperimentMetric) -> 'ExperimentPlanner':
        """
        Добавление метрики эксперимента
        """
        self.metrics.append(metric)
        return self
    
    def add_factor(self, factor: ExperimentFactor) -> 'ExperimentPlanner':
        """
        Добавление фактора эксперимента
        """
        self.factors.append(factor)
        return self
    
    def add_constraint(self, constraint: Dict[str, Any]) -> 'ExperimentPlanner':
        """
        Добавление ограничения эксперимента
        """
        self.constraints.append(constraint)
        return self
    
    def calculate_sample_size(self, power: float = 0.8, alpha: float = 0.05) -> Dict[str, Any]:
        """
        Расчет размера выборки для эксперимента
        """
        primary_metrics = [m for m in self.metrics if m.type == MetricType.PRIMARY]
        
        if not primary_metrics:
            raise ValueError("At least one primary metric is required")
        
        sample_sizes = {}
        for metric in primary_metrics:
            if metric.baseline_value is not None and metric.minimum_detectable_effect != 0:
                # Расчет для пропорций
                if metric.name in ['conversion_rate', 'click_through_rate']:
                    size = self._calculate_proportion_sample_size(
                        metric.baseline_value, 
                        metric.minimum_detectable_effect, 
                        power, alpha
                    )
                else:
                    # Расчет для средних
                    size = self._calculate_mean_sample_size(
                        metric.baseline_value,
                        metric.minimum_detectable_effect,
                        power, alpha
                    )
                
                sample_sizes[metric.name] = int(size)
        
        # Учитываем количество вариантов в эксперименте
        n_variants = 1
        for factor in self.factors:
            n_variants *= len(factor.levels)
        
        total_sample_size = max(sample_sizes.values()) * n_variants if sample_sizes else 1000
        
        return {
            'per_variant': sample_sizes,
            'total': total_sample_size,
            'n_variants': n_variants,
            'daily_traffic_requirement': int(total_sample_size / 14)  # 2 недели
        }
    
    def _calculate_proportion_sample_size(self, p1: float, mde: float, power: float, alpha: float) -> float:
        """
        Расчет размера выборки для пропорций
        """
        p2 = p1 * (1 + mde)
        pooled_p = (p1 + p2) / 2
        
        z_alpha = 1.96  # 95% confidence
        z_beta = 0.84   # 80% power
        
        effect_size = (p1 - p2) / np.sqrt(pooled_p * (1 - pooled_p))
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return n
    
    def _calculate_mean_sample_size(self, mean1: float, expected_diff: float, power: float, alpha: float) -> float:
        """
        Расчет размера выборки для средних
        """
        # Предполагаем стандартное отклонение как 20% от среднего
        std_dev = mean1 * 0.2
        
        z_alpha = 1.96  # 95% confidence
        z_beta = 0.84   # 80% power
        
        effect_size = expected_diff / std_dev
        n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
        
        return n
    
    def generate_experiment_plan(self) -> Dict[str, Any]:
        """
        Генерация полного плана эксперимента
        """
        sample_size_analysis = self.calculate_sample_size()
        
        plan = {
            'experiment': self.experiment_config,
            'metrics': [
                {
                    'name': m.name,
                    'description': m.description,
                    'type': m.type.value,
                    'baseline_value': m.baseline_value,
                    'minimum_detectable_effect': m.minimum_detectable_effect,
                    'acceptable_risk': m.acceptable_risk
                } for m in self.metrics
            ],
            'factors': [
                {
                    'name': f.name,
                    'description': f.description,
                    'levels': f.levels,
                    'is_essential': f.is_essential
                } for f in self.factors
            ],
            'sample_size': sample_size_analysis,
            'constraints': self.constraints,
            'randomization_scheme': self._determine_randomization_scheme(),
            'blocking_strategy': self._determine_blocking_strategy(),
            'duration_estimate': self._estimate_duration(sample_size_analysis['total']),
            'risk_mitigation': self._determine_risk_mitigation()
        }
        
        return plan
    
    def _determine_randomization_scheme(self) -> str:
        """
        Определение схемы рандомизации
        """
        # Определяем на основе факторов и целей
        if len(self.factors) == 1 and len(self.factors[0].levels) == 2:
            return "simple_randomization"
        else:
            return "stratified_randomization"
    
    def _determine_blocking_strategy(self) -> List[str]:
        """
        Определение стратегии блокировки
        """
        blocking_factors = []
        
        # Определяем потенциальные факторы блокировки
        for factor in self.factors:
            if len(factor.levels) > 2:
                blocking_factors.append(factor.name)
        
        # Добавляем стандартные факторы блокировки
        standard_blocking = ['region', 'device_type', 'traffic_source']
        blocking_factors.extend(standard_blocking)
        
        return list(set(blocking_factors))
    
    def _estimate_duration(self, total_sample_size: int) -> Dict[str, Any]:
        """
        Оценка продолжительности эксперимента
        """
        # Предполагаем средний трафик
        avg_daily_users = 10000
        days_required = int(np.ceil(total_sample_size / avg_daily_users))
        
        # Учитываем статистические требования
        min_duration = 7  # Минимум неделя
        max_duration = 30  # Максимум месяц
        
        estimated_days = max(min_duration, min(days_required, max_duration))
        
        return {
            'estimated_days': estimated_days,
            'min_days': min_duration,
            'max_days': max_duration,
            'daily_users_required': int(np.ceil(total_sample_size / estimated_days))
        }
    
    def _determine_risk_mitigation(self) -> List[Dict[str, str]]:
        """
        Определение мер снижения рисков
        """
        mitigation_strategies = [
            {
                'strategy': 'gradual_rollout',
                'description': 'Поэтапное увеличение трафика в эксперимент'
            },
            {
                'strategy': 'automatic_stop',
                'description': 'Автоматическая остановка при достижении порога риска'
            },
            {
                'strategy': 'control_group',
                'description': 'Сохранение контрольной группы для сравнения'
            }
        ]
        
        # Добавляем специфические меры в зависимости от типа эксперимента
        if self.experiment_config.get('type') == ExperimentType.A_B_TEST.value:
            mitigation_strategies.append({
                'strategy': 'early_stopping',
                'description': 'Возможность ранней остановки при достижении статистической значимости'
            })
        
        return mitigation_strategies

# Пример планирования эксперимента
def example_experiment_planning():
    planner = ExperimentPlanner()
    
    # Определение эксперимента
    planner.define_experiment(
        name="new_checkout_flow",
        experiment_type=ExperimentType.A_B_TEST,
        description="Testing new checkout flow to improve conversion rate",
        hypothesis="New checkout flow will increase conversion rate by 5%"
    )
    
    # Добавление метрик
    planner.add_metric(ExperimentMetric(
        name="conversion_rate",
        description="Percentage of users who complete purchase",
        type=MetricType.PRIMARY,
        baseline_value=0.03,  # 3% текущая конверсия
        minimum_detectable_effect=0.05,  # 5% улучшение
        acceptable_risk=0.05
    )).add_metric(ExperimentMetric(
        name="average_order_value",
        description="Average value of completed orders",
        type=MetricType.SECONDARY,
        baseline_value=75.0,
        minimum_detectable_effect=0.02,
        acceptable_risk=0.1
    )).add_metric(ExperimentMetric(
        name="checkout_abandonment_rate",
        description="Percentage of users who abandon checkout",
        type=MetricType.RISK,
        baseline_value=0.65,
        minimum_detectable_effect=-0.05,  # Отрицательное значение - улучшение
        acceptable_risk=0.05
    ))
    
    # Добавление факторов
    planner.add_factor(ExperimentFactor(
        name="checkout_flow",
        description="Type of checkout flow",
        levels=["old", "new"]
    ))
    
    # Добавление ограничений
    planner.add_constraint({
        "type": "traffic_allocation",
        "value": 0.5,
        "description": "Maximum 50% of traffic to test group"
    }).add_constraint({
        "type": "duration",
        "value": 14,
        "description": "Maximum 14 days duration"
    })
    
    # Генерация плана
    plan = planner.generate_experiment_plan()
    
    print("Experiment Plan:")
    print(json.dumps(plan, indent=2, default=str))
    
    return planner

if __name__ == "__main__":
    example_experiment_planning()
```

## Рандомизация и контроль

### Методы рандомизации

```python
# randomization_methods.py
import numpy as np
import pandas as pd
from typing import List, Dict, Any, Optional
import hashlib
import random

class RandomizationManager:
    def __init__(self, seed: Optional[int] = None):
        if seed:
            np.random.seed(seed)
            random.seed(seed)
        self.assignment_history = {}
    
    def simple_randomization(self, n_participants: int, n_groups: int = 2) -> np.ndarray:
        """
        Простая рандомизация - равномерное распределение по группам
        """
        groups = np.random.choice(n_groups, size=n_participants)
        return groups
    
    def stratified_randomization(self, 
                               strata: np.ndarray, 
                               n_groups: int = 2,
                               proportions: Optional[List[float]] = None) -> np.ndarray:
        """
        Стратифицированная рандомизация - сбалансированное распределение внутри страт
        """
        if proportions is None:
            proportions = [1/n_groups] * n_groups
        
        unique_strata = np.unique(strata)
        assignments = np.zeros(len(strata), dtype=int)
        
        for stratum in unique_strata:
            stratum_mask = strata == stratum
            stratum_size = np.sum(stratum_mask)
            
            # Распределение внутри страты
            stratum_assignments = np.random.choice(
                n_groups, 
                size=stratum_size, 
                p=proportions
            )
            
            assignments[stratum_mask] = stratum_assignments
        
        return assignments
    
    def block_randomization(self, 
                          n_participants: int, 
                          block_size: int = 4,
                          n_groups: int = 2) -> np.ndarray:
        """
        Блочная рандомизация - гарантирует равное распределение в каждом блоке
        """
        assignments = np.zeros(n_participants, dtype=int)
        
        n_blocks = n_participants // block_size
        remaining = n_participants % block_size
        
        for i in range(n_blocks):
            block_start = i * block_size
            block_end = block_start + block_size
            
            # Создаем сбалансированный блок
            block_assignments = np.repeat(range(n_groups), block_size // n_groups)
            # Добавляем оставшиеся элементы
            remaining_in_block = block_size % n_groups
            if remaining_in_block > 0:
                block_assignments = np.append(block_assignments, 
                                            np.random.choice(n_groups, 
                                                           size=remaining_in_block, 
                                                           replace=False))
            
            # Перемешиваем блок
            np.random.shuffle(block_assignments)
            assignments[block_start:block_end] = block_assignments
        
        # Обработка оставшихся участников
        if remaining > 0:
            start_remaining = n_blocks * block_size
            remaining_assignments = np.random.choice(n_groups, size=remaining)
            assignments[start_remaining:] = remaining_assignments
        
        return assignments
    
    def minimization_randomization(self,
                                 factors: List[np.ndarray],
                                 n_groups: int = 2) -> np.ndarray:
        """
        Минимизационная рандомизация - минимизация дисбаланса по факторам
        """
        n_participants = len(factors[0])
        assignments = np.zeros(n_participants, dtype=int)
        
        for i in range(n_participants):
            # Подсчет текущего распределения для каждого фактора
            factor_imbalances = []
            
            for factor in factors:
                unique_values = np.unique(factor[:i])
                imbalances = []
                
                for group in range(n_groups):
                    group_mask = assignments[:i] == group
                    factor_imbalances_for_group = []
                    
                    for value in unique_values:
                        factor_mask = (factor[:i] == value) & group_mask
                        count = np.sum(factor_mask)
                        factor_imbalances_for_group.append(count)
                    
                    imbalances.append(factor_imbalances_for_group)
                
                factor_imbalances.append(imbalances)
            
            # Выбор группы с минимальным дисбалансом
            group_scores = np.zeros(n_groups)
            
            for factor_idx, factor_imbalances_factor in enumerate(factor_imbalances):
                for group_idx, group_imbalances in enumerate(factor_imbalances_factor):
                    # Простая метрика дисбаланса - максимальное отклонение
                    if len(group_imbalances) > 0:
                        group_scores[group_idx] += max(group_imbalances) - min(group_imbalances)
            
            # Случайный выбор среди групп с минимальным счетом
            min_score = np.min(group_scores)
            best_groups = np.where(group_scores == min_score)[0]
            assignments[i] = np.random.choice(best_groups)
        
        return assignments
    
    def hash_based_assignment(self, 
                            identifiers: List[str], 
                            n_groups: int = 2,
                            salt: str = "") -> np.ndarray:
        """
        Рандомизация на основе хеширования - детерминированная рандомизация
        """
        assignments = []
        
        for identifier in identifiers:
            # Создаем хеш от идентификатора и соли
            hash_input = f"{identifier}{salt}".encode('utf-8')
            hash_value = hashlib.sha256(hash_input).hexdigest()
            
            # Преобразуем хеш в число и определяем группу
            hash_int = int(hash_value[:8], 16)  # Берем первые 8 символов
            group = hash_int % n_groups
            assignments.append(group)
        
        return np.array(assignments, dtype=int)
    
    def adaptive_randomization(self,
                             n_participants: int,
                             initial_allocation: float = 0.5,
                             adaptation_rate: float = 0.1) -> np.ndarray:
        """
        Адаптивная рандомизация - изменение вероятностей на основе промежуточных результатов
        """
        assignments = np.zeros(n_participants, dtype=int)
        group_counts = [0, 0]  # [control, treatment]
        
        for i in range(n_participants):
            # Адаптируем вероятности на основе текущих результатов
            # (в реальном сценарии здесь будут реальные результаты)
            prob_treatment = initial_allocation + (
                np.random.normal(0, adaptation_rate)  # Симуляция адаптации
            )
            prob_treatment = max(0.1, min(0.9, prob_treatment))  # Ограничения
            
            assignment = np.random.choice([0, 1], p=[1-prob_treatment, prob_treatment])
            assignments[i] = assignment
            group_counts[assignment] += 1
        
        return assignments

# Пример использования методов рандомизации
def example_randomization():
    manager = RandomizationManager(seed=42)
    
    n_participants = 1000
    
    # Простая рандомизация
    simple_assignments = manager.simple_randomization(n_participants, n_groups=2)
    print(f"Simple randomization - Group 0: {np.sum(simple_assignments == 0)}, Group 1: {np.sum(simple_assignments == 1)}")
    
    # Стратифицированная рандомизация
    strata = np.random.choice(['A', 'B', 'C'], size=n_participants)
    stratified_assignments = manager.stratified_randomization(strata, n_groups=2)
    
    # Проверка баланса
    for s in ['A', 'B', 'C']:
        stratum_mask = strata == s
        group_0_in_stratum = np.sum((stratified_assignments == 0) & stratum_mask)
        group_1_in_stratum = np.sum((stratified_assignments == 1) & stratum_mask)
        print(f"Stratum {s} - Group 0: {group_0_in_stratum}, Group 1: {group_1_in_stratum}")
    
    # Блочная рандомизация
    block_assignments = manager.block_randomization(n_participants, block_size=6, n_groups=2)
    print(f"Block randomization - Group 0: {np.sum(block_assignments == 0)}, Group 1: {np.sum(block_assignments == 1)}")
    
    # Хеш-базированная рандомизация
    user_ids = [f"user_{i}" for i in range(n_participants)]
    hash_assignments = manager.hash_based_assignment(user_ids, n_groups=2)
    print(f"Hash-based assignment - Group 0: {np.sum(hash_assignments == 0)}, Group 1: {np.sum(hash_assignments == 1)}")
    
    return manager

if __name__ == "__main__":
    example_randomization()
```

## Анализ результатов эксперимента

### Статистический анализ

```python
# experiment_analysis.py
import numpy as np
import pandas as pd
from scipy import stats
from statsmodels.stats.power import ttest_power
from statsmodels.stats.weightstats import ttest_ind
from statsmodels.stats.multitest import multipletests
import matplotlib.pyplot as plt
import seaborn as sns
from typing import Dict, List, Tuple, Optional
import warnings
warnings.filterwarnings('ignore')

class ExperimentAnalyzer:
    def __init__(self):
        self.results = {}
        self.effect_sizes = {}
    
    def analyze_ab_test(self, 
                       control_data: np.ndarray, 
                       treatment_data: np.ndarray,
                       alpha: float = 0.05,
                       alternative: str = 'two-sided') -> Dict:
        """
        Анализ A/B теста с полной статистической информацией
        """
        # Основные статистики
        control_mean = np.mean(control_data)
        treatment_mean = np.mean(treatment_data)
        control_std = np.std(control_data)
        treatment_std = np.std(treatment_data)
        
        # T-тест
        t_stat, p_value = stats.ttest_ind(control_data, treatment_data, equal_var=False)
        
        # Доверительные интервалы
        control_se = control_std / np.sqrt(len(control_data))
        treatment_se = treatment_std / np.sqrt(len(treatment_data))
        
        # Доверительный интервал для разницы
        diff = treatment_mean - control_mean
        pooled_se = np.sqrt((control_std**2/len(control_data)) + (treatment_std**2/len(treatment_data)))
        ci_lower = diff - 1.96 * pooled_se
        ci_upper = diff + 1.96 * pooled_se
        
        # Размер эффекта (Cohen's d)
        pooled_std = np.sqrt(((len(control_data)-1)*control_std**2 + (len(treatment_data)-1)*treatment_std**2) / 
                            (len(control_data) + len(treatment_data) - 2))
        cohens_d = diff / pooled_std if pooled_std != 0 else 0
        
        # Статистическая мощность
        power = ttest_power(
            effect_size=abs(cohens_d),
            nobs1=len(control_data),
            nobs2=len(treatment_data),
            alpha=alpha,
            alternative=alternative
        )
        
        # Относительный эффект
        relative_effect = (treatment_mean - control_mean) / control_mean * 100 if control_mean != 0 else 0
        
        results = {
            'descriptive_stats': {
                'control': {
                    'mean': float(control_mean),
                    'std': float(control_std),
                    'n': int(len(control_data)),
                    'median': float(np.median(control_data)),
                    'q25': float(np.quantile(control_data, 0.25)),
                    'q75': float(np.quantile(control_data, 0.75))
                },
                'treatment': {
                    'mean': float(treatment_mean),
                    'std': float(treatment_std),
                    'n': int(len(treatment_data)),
                    'median': float(np.median(treatment_data)),
                    'q25': float(np.quantile(treatment_data, 0.25)),
                    'q75': float(np.quantile(treatment_data, 0.75))
                }
            },
            'statistical_test': {
                't_statistic': float(t_stat),
                'p_value': float(p_value),
                'degrees_of_freedom': float(len(control_data) + len(treatment_data) - 2),
                'significant': p_value < alpha,
                'alternative': alternative
            },
            'effect_size': {
                'absolute_difference': float(diff),
                'relative_difference': float(relative_effect),
                'cohens_d': float(cohens_d),
                'confidence_interval': (float(ci_lower), float(ci_upper)),
                'confidence_level': 0.95
            },
            'power_analysis': {
                'power': float(power),
                'target_power': 0.8,
                'actual_power': float(power)
            },
            'interpretation': self._interpret_results(p_value, cohens_d, power)
        }
        
        self.results = results
        return results
    
    def _interpret_results(self, p_value: float, cohens_d: float, power: float) -> Dict[str, str]:
        """
        Интерпретация результатов статистического теста
        """
        interpretation = {}
        
        # Значимость
        if p_value < 0.001:
            interpretation['significance'] = "Highly significant (p < 0.001)"
        elif p_value < 0.01:
            interpretation['significance'] = "Very significant (p < 0.01)"
        elif p_value < 0.05:
            interpretation['significance'] = "Significant (p < 0.05)"
        else:
            interpretation['significance'] = "Not significant (p >= 0.05)"
        
        # Размер эффекта (Cohen's conventions)
        if abs(cohens_d) < 0.2:
            interpretation['effect_size'] = "Very small effect"
        elif abs(cohens_d) < 0.5:
            interpretation['effect_size'] = "Small effect"
        elif abs(cohens_d) < 0.8:
            interpretation['effect_size'] = "Medium effect"
        else:
            interpretation['effect_size'] = "Large effect"
        
        # Мощность
        if power < 0.5:
            interpretation['power'] = "Low power (< 0.5)"
        elif power < 0.8:
            interpretation['power'] = "Adequate power (0.5-0.8)"
        else:
            interpretation['power'] = "High power (>= 0.8)"
        
        return interpretation
    
    def analyze_multiple_metrics(self, 
                                metric_data: Dict[str, Tuple[np.ndarray, np.ndarray]],
                                correction_method: str = 'bonferroni') -> Dict:
        """
        Анализ множества метрик с коррекцией на множественное тестирование
        """
        results = {}
        
        # Извлечение p-значений
        raw_p_values = []
        metric_names = []
        
        for metric_name, (control_data, treatment_data) in metric_data.items():
            _, p_val = stats.ttest_ind(control_data, treatment_data, equal_var=False)
            raw_p_values.append(p_val)
            metric_names.append(metric_name)
        
        # Коррекция p-значений
        corrected_p_values, rejected, alpha_sidak, alpha_bonf = multipletests(
            raw_p_values, alpha=0.05, method=correction_method
        )
        
        # Анализ каждой метрики
        for i, metric_name in enumerate(metric_names):
            control_data, treatment_data = metric_data[metric_name]
            
            # Базовый анализ для этой метрики
            metric_analysis = self.analyze_ab_test(control_data, treatment_data)
            
            # Добавление информации о коррекции
            metric_analysis['multiple_testing'] = {
                'raw_p_value': float(raw_p_values[i]),
                'corrected_p_value': float(corrected_p_values[i]),
                'significant_after_correction': bool(rejected[i]),
                'correction_method': correction_method
            }
            
            results[metric_name] = metric_analysis
        
        return results
    
    def calculate_sample_size_post_hoc(self, 
                                     control_data: np.ndarray, 
                                     treatment_data: np.ndarray,
                                     desired_power: float = 0.8) -> Dict:
        """
        Пост-хок расчет размера выборки для достижения желаемой мощности
        """
        control_mean = np.mean(control_data)
        treatment_mean = np.mean(treatment_data)
        control_std = np.std(control_data)
        treatment_std = np.std(treatment_data)
        
        # Объединенное стандартное отклонение
        pooled_std = np.sqrt(((len(control_data)-1)*control_std**2 + (len(treatment_data)-1)*treatment_std**2) / 
                            (len(control_data) + len(treatment_data) - 2))
        
        # Размер эффекта
        effect_size = abs(treatment_mean - control_mean) / pooled_std
        
        # Расчет необходимого размера выборки для заданной мощности
        from statsmodels.stats.power import ttest_power
        import scipy.optimize as opt
        
        def power_equation(n, effect_size, power, alpha=0.05):
            return ttest_power(effect_size=effect_size, nobs1=n, nobs2=n, alpha=alpha) - power
        
        # Численное решение
        try:
            required_n = opt.fsolve(power_equation, len(control_data), args=(effect_size, desired_power))[0]
            required_n = max(2, int(np.ceil(required_n)))  # Минимум 2 наблюдения
        except:
            # Приближенная формула
            z_alpha = stats.norm.ppf(1 - 0.05/2)
            z_beta = stats.norm.ppf(desired_power)
            required_n = 2 * ((z_alpha + z_beta) / effect_size) ** 2
            required_n = int(np.ceil(required_n))
        
        return {
            'observed_effect_size': float(effect_size),
            'observed_power': float(ttest_power(
                effect_size=effect_size,
                nobs1=len(control_data),
                nobs2=len(treatment_data),
                alpha=0.05
            )),
            'required_sample_size_per_group': required_n,
            'required_total_sample_size': required_n * 2,
            'desired_power': desired_power,
            'current_sample_size_per_group': len(control_data)
        }
    
    def visualize_results(self, control_data: np.ndarray, treatment_data: np.ndarray, title: str = "Experiment Results"):
        """
        Визуализация результатов эксперимента
        """
        fig, axes = plt.subplots(2, 2, figsize=(15, 12))
        
        # Гистограммы
        axes[0, 0].hist(control_data, alpha=0.7, label='Control', bins=30)
        axes[0, 0].hist(treatment_data, alpha=0.7, label='Treatment', bins=30)
        axes[0, 0].set_title('Distribution Comparison')
        axes[0, 0].legend()
        axes[0, 0].grid(True, alpha=0.3)
        
        # Box plots
        axes[0, 1].boxplot([control_data, treatment_data], labels=['Control', 'Treatment'])
        axes[0, 1].set_title('Box Plot Comparison')
        axes[0, 1].grid(True, alpha=0.3)
        
        # Q-Q plot для проверки нормальности
        from scipy.stats import probplot
        stats.probplot(control_data, dist="norm", plot=axes[1, 0])
        axes[1, 0].set_title('Q-Q Plot - Control Group')
        axes[1, 0].grid(True, alpha=0.3)
        
        stats.probplot(treatment_data, dist="norm", plot=axes[1, 1])
        axes[1, 1].set_title('Q-Q Plot - Treatment Group')
        axes[1, 1].grid(True, alpha=0.3)
        
        plt.suptitle(title)
        plt.tight_layout()
        plt.show()
    
    def bayesian_analysis(self, 
                         control_successes: int, 
                         control_trials: int,
                         treatment_successes: int, 
                         treatment_trials: int) -> Dict:
        """
        Байесовский анализ для пропорций
        """
        # Используем Beta-распределение как априорное
        # Beta(1, 1) = Uniform (непредвзятое априорное)
        
        # Апостериорные распределения
        control_posterior = stats.beta(control_successes + 1, control_trials - control_successes + 1)
        treatment_posterior = stats.beta(treatment_successes + 1, treatment_trials - treatment_successes + 1)
        
        # Сэмплирование для оценки разницы
        n_samples = 100000
        control_samples = control_posterior.rvs(n_samples)
        treatment_samples = treatment_posterior.rvs(n_samples)
        difference_samples = treatment_samples - control_samples
        
        # Основные байесовские метрики
        probability_treatment_better = np.mean(difference_samples > 0)
        credible_interval = np.percentile(difference_samples, [2.5, 97.5])
        
        results = {
            'bayesian_metrics': {
                'probability_treatment_better': float(probability_treatment_better),
                'credible_interval': (float(credible_interval[0]), float(credible_interval[1])),
                'mean_difference': float(np.mean(difference_samples)),
                'median_difference': float(np.median(difference_samples))
            },
            'posterior_distributions': {
                'control': {
                    'mean': float(control_posterior.mean()),
                    'std': float(control_posterior.std()),
                    '95_ci': (float(control_posterior.ppf(0.025)), float(control_posterior.ppf(0.975)))
                },
                'treatment': {
                    'mean': float(treatment_posterior.mean()),
                    'std': float(treatment_posterior.std()),
                    '95_ci': (float(treatment_posterior.ppf(0.025)), float(treatment_posterior.ppf(0.975)))
                }
            }
        }
        
        return results

# Пример анализа эксперимента
def example_experiment_analysis():
    analyzer = ExperimentAnalyzer()
    
    # Симуляция данных A/B теста
    np.random.seed(42)
    
    # Контрольная группа - текущий уровень конверсии 3%
    control_conversions = np.random.binomial(1, 0.03, 5000)
    
    # Тестовая группа - ожидаемый уровень 3.5%
    treatment_conversions = np.random.binomial(1, 0.035, 5000)
    
    # Анализ A/B теста
    results = analyzer.analyze_ab_test(control_conversions, treatment_conversions)
    
    print("A/B Test Analysis Results:")
    print(f"Control Conversion Rate: {results['descriptive_stats']['control']['mean']:.4f}")
    print(f"Treatment Conversion Rate: {results['descriptive_stats']['treatment']['mean']:.4f}")
    print(f"Absolute Difference: {results['effect_size']['absolute_difference']:.4f}")
    print(f"Relative Difference: {results['effect_size']['relative_difference']:.2f}%")
    print(f"P-value: {results['statistical_test']['p_value']:.4f}")
    print(f"Statistically Significant: {results['statistical_test']['significant']}")
    print(f"Effect Size (Cohen's d): {results['effect_size']['cohens_d']:.4f}")
    print(f"Power: {results['power_analysis']['power']:.4f}")
    
    print("\nInterpretation:")
    for key, value in results['interpretation'].items():
        print(f"  {key}: {value}")
    
    # Байесовский анализ
    bayesian_results = analyzer.bayesian_analysis(
        control_successes=int(np.sum(control_conversions)),
        control_trials=len(control_conversions),
        treatment_successes=int(np.sum(treatment_conversions)),
        treatment_trials=len(treatment_conversions)
    )
    
    print(f"\nBayesian Analysis:")
    print(f"Probability treatment is better: {bayesian_results['bayesian_metrics']['probability_treatment_better']:.4f}")
    print(f"95% Credible Interval: {bayesian_results['bayesian_metrics']['credible_interval']}")
    
    # Пост-хок анализ мощности
    power_analysis = analyzer.calculate_sample_size_post_hoc(control_conversions, treatment_conversions)
    print(f"\nPost-hoc Power Analysis:")
    print(f"Required sample size per group: {power_analysis['required_sample_size_per_group']}")
    print(f"Current power: {power_analysis['observed_power']:.4f}")
    
    return analyzer

if __name__ == "__main__":
    example_experiment_analysis()
```

## Множественное тестирование

### Коррекция на множественное тестирование

```python
# multiple_testing.py
import numpy as np
import pandas as pd
from scipy.stats import uniform
from statsmodels.stats.multitest import multipletests
import matplotlib.pyplot as plt
import seaborn as sns

class MultipleTestingCorrector:
    def __init__(self):
        self.correction_results = {}
    
    def bonferroni_correction(self, p_values: List[float]) -> List[float]:
        """
        Коррекция Бонферрони - наиболее консервативная
        """
        n_tests = len(p_values)
        corrected_p_values = [min(p * n_tests, 1.0) for p in p_values]
        return corrected_p_values
    
    def holm_correction(self, p_values: List[float]) -> List[float]:
        """
        Коррекция Холма - менее консервативная, чем Бонферрони
        """
        n_tests = len(p_values)
        sorted_indices = np.argsort(p_values)
        sorted_p_values = np.array(p_values)[sorted_indices]
        
        corrected_p_values = np.zeros(n_tests)
        for i in range(n_tests):
            corrected_p_values[i] = min(sorted_p_values[i] * (n_tests - i), 1.0)
        
        # Восстанавливаем исходный порядок
        original_order = np.empty(n_tests, dtype=int)
        original_order[sorted_indices] = np.arange(n_tests)
        return corrected_p_values[original_order].tolist()
    
    def benjamini_hochberg_correction(self, p_values: List[float]) -> List[float]:
        """
        Процедура Бенджамини-Хохберга для контроля FDR
        """
        n_tests = len(p_values)
        sorted_indices = np.argsort(p_values)
        sorted_p_values = np.array(p_values)[sorted_indices]
        
        corrected_p_values = np.zeros(n_tests)
        for i in range(n_tests - 1, -1, -1):
            if i == n_tests - 1:
                corrected_p_values[i] = sorted_p_values[i]
            else:
                corrected_p_values[i] = min(
                    sorted_p_values[i] * n_tests / (i + 1),
                    corrected_p_values[i + 1]
                )
        
        # Восстанавливаем исходный порядок
        original_order = np.empty(n_tests, dtype=int)
        original_order[sorted_indices] = np.arange(n_tests)
        return corrected_p_values[original_order].tolist()
    
    def apply_correction(self, 
                        p_values: List[float], 
                        method: str = 'fdr_bh',
                        alpha: float = 0.05) -> Dict:
        """
        Применение коррекции множественного тестирования
        """
        if method == 'bonferroni':
            corrected_p_values = self.bonferroni_correction(p_values)
        elif method == 'holm':
            corrected_p_values = self.holm_correction(p_values)
        elif method == 'fdr_bh':
            corrected_p_values = self.benjamini_hochberg_correction(p_values)
        elif method == 'statsmodels':
            # Использование statsmodels для различных методов
            reject, pvals_corrected, alphacSidak, alphacBonf = multipletests(
                p_values, alpha=alpha, method='fdr_bh'
            )
            corrected_p_values = pvals_corrected.tolist()
        else:
            raise ValueError(f"Unknown correction method: {method}")
        
        # Определение значимых результатов
        significant = [p <= alpha for p in corrected_p_values]
        
        results = {
            'original_p_values': p_values,
            'corrected_p_values': corrected_p_values,
            'significant': significant,
            'n_significant': sum(significant),
            'method': method,
            'alpha': alpha
        }
        
        self.correction_results = results
        return results
    
    def compare_correction_methods(self, 
                                 p_values: List[float], 
                                 methods: List[str] = None) -> pd.DataFrame:
        """
        Сравнение различных методов коррекции
        """
        if methods is None:
            methods = ['bonferroni', 'holm', 'fdr_bh']
        
        comparison_data = []
        
        for method in methods:
            try:
                results = self.apply_correction(p_values, method=method)
                comparison_data.append({
                    'method': method,
                    'significant_count': results['n_significant'],
                    'alpha': results['alpha']
                })
            except Exception as e:
                print(f"Error with method {method}: {e}")
        
        return pd.DataFrame(comparison_data)
    
    def simulate_multiple_testing(self, 
                                n_tests: int, 
                                n_true_null: int,
                                effect_size: float = 0.2) -> Dict:
        """
        Симуляция множественного тестирования
        """
        # Генерация p-значений: часть из них действительно нулевая гипотеза
        true_null_mask = [i < n_true_null for i in range(n_tests)]
        
        p_values = []
        for is_true_null in true_null_mask:
            if is_true_null:
                # Для истинно нулевых гипотез p-значения распределены равномерно
                p_val = np.random.uniform(0, 1)
            else:
                # Для альтернативных гипотез p-значения смещены в сторону 0
                # Используем бета-распределение
                p_val = np.random.beta(1, 1 + effect_size*10)
            
            p_values.append(min(p_val, 0.999))  # Избегаем точных 0 и 1
        
        return {
            'p_values': p_values,
            'true_null_mask': true_null_mask,
            'n_tests': n_tests,
            'n_true_null': n_true_null,
            'n_false_null': n_tests - n_true_null
        }

# Пример использования коррекции множественного тестирования
def example_multiple_testing():
    corrector = MultipleTestingCorrector()
    
    # Симуляция p-значений из экспериментов
    np.random.seed(42)
    
    # Предположим, мы тестируем 20 метрик, из которых 15 не должны показывать эффект
    simulated_data = corrector.simulate_multiple_testing(
        n_tests=20,
        n_true_null=15,  # 15 истинно нулевых гипотез
        effect_size=0.3
    )
    
    print(f"Simulated {simulated_data['n_tests']} tests")
    print(f"True null hypotheses: {simulated_data['n_true_null']}")
    print(f"False null hypotheses: {simulated_data['n_false_null']}")
    print(f"Original p-values: {simulated_data['p_values'][:10]}...")  # Показываем первые 10
    
    # Применение различных методов коррекции
    methods = ['bonferroni', 'holm', 'fdr_bh']
    
    print("\nComparison of correction methods:")
    print("-" * 50)
    
    for method in methods:
        results = corrector.apply_correction(simulated_data['p_values'], method=method)
        print(f"{method.upper()}: {results['n_significant']} significant tests")
    
    # Сравнение методов
    comparison_df = corrector.compare_correction_methods(simulated_data['p_values'])
    print(f"\nMethod comparison:\n{comparison_df}")
    
    return corrector

if __name__ == "__main__":
    example_multiple_testing()
```

## Риск-менеджмент в экспериментах

### Оценка и управление рисками

```python
# risk_management.py
import numpy as np
import pandas as pd
from typing import Dict, List, Tuple
import matplotlib.pyplot as plt
import seaborn as sns

class ExperimentRiskManager:
    def __init__(self):
        self.risk_assessment = {}
        self.mitigation_strategies = []
    
    def assess_experiment_risks(self, 
                               business_impact: float,
                               technical_complexity: float,
                               user_experience_impact: float,
                               reversibility: float,
                               novelty: float) -> Dict:
        """
        Оценка рисков эксперимента по нескольким维度
        """
        # Нормализация значений (0-1, где 1 = максимальный риск)
        risks = {
            'business_risk': business_impact,
            'technical_risk': technical_complexity,
            'ux_risk': user_experience_impact,
            'reversibility_risk': 1 - reversibility,  # Чем меньше reversibility, тем выше риск
            'novelty_risk': novelty
        }
        
        # Взвешенная оценка общего риска
        weights = {
            'business_risk': 0.25,
            'technical_risk': 0.2,
            'ux_risk': 0.25,
            'reversibility_risk': 0.15,
            'novelty_risk': 0.15
        }
        
        overall_risk = sum(risks[key] * weights[key] for key in risks.keys())
        
        risk_level = self._categorize_risk(overall_risk)
        
        assessment = {
            'individual_risks': risks,
            'weights': weights,
            'overall_risk_score': overall_risk,
            'risk_level': risk_level,
            'recommendations': self._get_recommendations(risk_level, risks)
        }
        
        self.risk_assessment = assessment
        return assessment
    
    def _categorize_risk(self, score: float) -> str:
        """
        Категоризация уровня риска
        """
        if score < 0.3:
            return 'Low'
        elif score < 0.6:
            return 'Medium'
        elif score < 0.8:
            return 'High'
        else:
            return 'Critical'
    
    def _get_recommendations(self, risk_level: str, risks: Dict) -> List[str]:
        """
        Получение рекомендаций на основе уровня риска
        """
        recommendations = []
        
        if risk_level in ['High', 'Critical']:
            recommendations.extend([
                "Implement gradual rollout strategy",
                "Establish clear rollback procedures",
                "Increase monitoring frequency",
                "Require additional stakeholder approvals"
            ])
        
        if risks['business_risk'] > 0.7:
            recommendations.append("Conduct thorough business impact analysis")
        
        if risks['technical_risk'] > 0.7:
            recommendations.append("Perform extensive technical testing")
        
        if risks['ux_risk'] > 0.7:
            recommendations.append("Conduct user experience testing")
        
        if risks['reversibility_risk'] > 0.7:
            recommendations.append("Implement feature flags for easy toggling")
        
        if risks['novelty_risk'] > 0.7:
            recommendations.append("Start with small percentage of users")
        
        return recommendations
    
    def calculate_sample_size_with_risk_adjustment(self, 
                                                 base_sample_size: int,
                                                 risk_level: str,
                                                 confidence_level: float = 0.95) -> Dict:
        """
        Расчет размера выборки с учетом рисков
        """
        risk_multipliers = {
            'Low': 1.0,
            'Medium': 1.2,
            'High': 1.5,
            'Critical': 2.0
        }
        
        risk_multiplier = risk_multipliers.get(risk_level, 1.0)
        adjusted_sample_size = int(base_sample_size * risk_multiplier)
        
        # Также увеличиваем размер выборки для более высокой достоверности при высоких рисках
        if risk_level in ['High', 'Critical']:
            adjusted_sample_size = int(adjusted_sample_size * 1.2)  # Дополнительный запас
        
        return {
            'base_sample_size': base_sample_size,
            'risk_multiplier': risk_multiplier,
            'adjusted_sample_size': adjusted_sample_size,
            'risk_level': risk_level,
            'confidence_level': confidence_level
        }
    
    def design_safety_mechanisms(self, risk_level: str) -> Dict:
        """
        Проектирование механизмов безопасности в зависимости от уровня риска
        """
        safety_mechanisms = {
            'monitoring': {
                'Low': ['basic_metrics'],
                'Medium': ['basic_metrics', 'error_rates', 'performance'],
                'High': ['basic_metrics', 'error_rates', 'performance', 'user_feedback'],
                'Critical': ['real_time_monitoring', 'automatic_alarms', 'emergency_procedures']
            },
            'rollout_strategy': {
                'Low': 'full_launch',
                'Medium': 'gradual_10_percent_increments',
                'High': 'gradual_5_percent_increments',
                'Critical': 'gradual_1_percent_increments'
            },
            'rollback_procedures': {
                'Low': 'manual_rollback',
                'Medium': 'semi_automated_rollback',
                'High': 'automated_rollback_with_alerts',
                'Critical': 'instant_rollback_with_automated_recovery'
            },
            'approval_process': {
                'Low': 'single_team_approval',
                'Medium': 'cross_team_review',
                'High': 'management_approval_required',
                'Critical': 'executive_approval_required'
            }
        }
        
        mechanisms = {}
        for category, level_mapping in safety_mechanisms.items():
            mechanisms[category] = level_mapping[risk_level]
        
        return mechanisms
    
    def simulate_risk_scenarios(self, n_simulations: int = 1000) -> pd.DataFrame:
        """
        Симуляция сценариев рисков для анализа
        """
        np.random.seed(42)
        
        scenarios = []
        for i in range(n_simulations):
            scenario = {
                'business_impact': np.random.beta(2, 5),  # Смещение к низким значениям
                'technical_complexity': np.random.beta(3, 3),  # Средние значения
                'user_experience_impact': np.random.beta(2, 4),  # Смещение к низким
                'reversibility': np.random.beta(4, 2),  # Смещение к высокой reversibility
                'novelty': np.random.beta(2, 3)  # Смещение к средним значениям
            }
            
            # Вычисляем общий риск для сценария
            weights = {'business_impact': 0.25, 'technical_complexity': 0.2, 
                      'user_experience_impact': 0.25, 'reversibility': -0.15, 'novelty': 0.15}
            
            overall_risk = (scenario['business_impact'] * weights['business_impact'] +
                           scenario['technical_complexity'] * weights['technical_complexity'] +
                           scenario['user_experience_impact'] * weights['user_experience_impact'] +
                           (1 - scenario['reversibility']) * abs(weights['reversibility']) +
                           scenario['novelty'] * weights['novelty'])
            
            scenario['overall_risk'] = overall_risk
            scenario['risk_category'] = self._categorize_risk(overall_risk)
            scenario['scenario_id'] = i
            
            scenarios.append(scenario)
        
        return pd.DataFrame(scenarios)
    
    def visualize_risk_distribution(self, scenarios_df: pd.DataFrame):
        """
        Визуализация распределения рисков
        """
        fig, axes = plt.subplots(2, 2, figsize=(15, 12))
        
        # Распределение общего риска
        axes[0, 0].hist(scenarios_df['overall_risk'], bins=30, alpha=0.7, edgecolor='black')
        axes[0, 0].set_title('Distribution of Overall Risk Scores')
        axes[0, 0].set_xlabel('Risk Score')
        axes[0, 0].set_ylabel('Frequency')
        axes[0, 0].grid(True, alpha=0.3)
        
        # Категории рисков
        risk_counts = scenarios_df['risk_category'].value_counts()
        axes[0, 1].pie(risk_counts.values, labels=risk_counts.index, autopct='%1.1f%%')
        axes[0, 1].set_title('Risk Category Distribution')
        
        # Взаимосвязь между факторами риска
        risk_factors = ['business_impact', 'technical_complexity', 'user_experience_impact']
        risk_data = scenarios_df[risk_factors]
        
        corr_matrix = risk_data.corr()
        sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', center=0, 
                   ax=axes[1, 0], square=True)
        axes[1, 0].set_title('Correlation Between Risk Factors')
        
        # Risk vs Reversibility
        scatter = axes[1, 1].scatter(scenarios_df['reversibility'], scenarios_df['overall_risk'], 
                                    c=scenarios_df['overall_risk'], cmap='viridis', alpha=0.6)
        axes[1, 1].set_xlabel('Reversibility')
        axes[1, 1].set_ylabel('Overall Risk')
        axes[1, 1].set_title('Risk vs Reversibility')
        plt.colorbar(scatter, ax=axes[1, 1])
        
        plt.tight_layout()
        plt.show()

# Пример управления рисками
def example_risk_management():
    manager = ExperimentRiskManager()
    
    # Оценка рисков для эксперимента
    risk_assessment = manager.assess_experiment_risks(
        business_impact=0.6,      # Высокое влияние на бизнес
        technical_complexity=0.4, # Средняя техническая сложность
        user_experience_impact=0.7, # Высокое влияние на UX
        reversibility=0.8,        # Хорошая обратимость
        novelty=0.5               # Средняя новизна
    )
    
    print("Risk Assessment:")
    print(f"Overall Risk Score: {risk_assessment['overall_risk_score']:.3f}")
    print(f"Risk Level: {risk_assessment['risk_level']}")
    print(f"Individual Risks: {risk_assessment['individual_risks']}")
    print(f"Recommendations: {risk_assessment['recommendations']}")
    
    # Расчет размера выборки с учетом рисков
    sample_size_calc = manager.calculate_sample_size_with_risk_adjustment(
        base_sample_size=10000,
        risk_level=risk_assessment['risk_level']
    )
    
    print(f"\nSample Size Calculation:")
    print(f"Base: {sample_size_calc['base_sample_size']}")
    print(f"Adjusted: {sample_size_calc['adjusted_sample_size']}")
    print(f"Multiplier: {sample_size_calc['risk_multiplier']}")
    
    # Дизайн механизмов безопасности
    safety_mechanisms = manager.design_safety_mechanisms(risk_assessment['risk_level'])
    print(f"\nSafety Mechanisms for {risk_assessment['risk_level']} risk:")
    for category, mechanism in safety_mechanisms.items():
        print(f"  {category}: {mechanism}")
    
    # Симуляция сценариев рисков
    scenarios = manager.simulate_risk_scenarios(n_simulations=500)
    print(f"\nSimulated {len(scenarios)} risk scenarios")
    print(f"Risk distribution: {scenarios['risk_category'].value_counts().to_dict()}")
    
    # Визуализация (закомментировано для предотвращения отображения при запуске)
    # manager.visualize_risk_distribution(scenarios)
    
    return manager

if __name__ == "__main__":
    example_risk_management()
```

## Лучшие практики экспериментального дизайна

### Планирование и реализация

- Определение четких гипотез
- Выбор подходящих метрик
- Расчет размера выборки
- Планирование случайности
- Учет множественного тестирования

### Анализ и интерпретация

- Использование надлежащих статистических тестов
- Коррекция на множественное тестирование
- Рассмотрение практической значимости
- Учет контекста и ограничений

### Мониторинг и масштабирование

- Непрерывный мониторинг метрик
- Ранняя остановка при необходимости
- Постепенное масштабирование
- Документирование результатов

## Заключение

Экспериментальный дизайн в CI/CD обеспечивает научный подход к улучшению программного обеспечения. Он позволяет:

- Принимать обоснованные решения на основе данных
- Минимизировать риски изменений
- Оптимизировать пользовательский опыт
- Улучшать бизнес-метрики

Успешная реализация экспериментального дизайна требует:

- Понимания статистических принципов
- Тщательного планирования
- Надлежащего контроля и рандомизации
- Эффективного анализа результатов

Организации, инвестирующие в экспериментальный дизайн, получают конкурентное преимущество за счет более быстрой и безопасной доставки улучшений.

## Связанные концепции

- [[A/B Testing]]
- [[Statistical Analysis]]
- [[Continuous Integration]]
- [[Continuous Deployment]]
- [[Monitoring and Observability]]
- [[Quality Assurance]]
- [[Data Pipeline]]
- [[Feature Flags]]
- [[Canary Deployments]]

## Внешние ресурсы

- [Experiment Design Guidelines](https://www.optimizely.com/optimization-glossary/experiment-design/)
- [A/B Testing Statistical Guide](https://vwo.com/ab-testing/)
- [Statistical Methods for A/B Testing](https://www.evanmiller.org/)
- [CI/CD Experimentation Best Practices](https://martinfowler.com/articles/feature-toggles.html)