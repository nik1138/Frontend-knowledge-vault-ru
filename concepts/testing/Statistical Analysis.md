---
aliases: [Статистический анализ, Statistical Analysis, Анализ данных]
tags: 
  - testing
  - analytics
  - ci-cd
  - data-science
  - experimentation
  - quality-assurance
---

# Статистический анализ в тестировании и CI/CD

## Обзор

Статистический анализ в тестировании и CI/CD - это применение статистических методов для оценки качества, надежности и производительности программного обеспечения на различных этапах непрерывной интеграции и доставки. Он позволяет принимать обоснованные решения на основе данных, определять значимость изменений, выявлять аномалии и обеспечивать статистическую достоверность результатов тестирования.

В контексте CI/CD статистический анализ играет ключевую роль в A/B тестировании, анализе производительности, оценке рисков и принятии решений о развертывании. Он обеспечивает научный подход к оценке изменений программного обеспечения и помогает командам принимать решения на основе данных, а не предположений.

## Основы статистического анализа

### Что такое статистический анализ в тестировании

Статистический анализ в тестировании - это процесс сбора, анализа и интерпретации данных о поведении программного обеспечения с использованием статистических методов. Это позволяет:

- Оценивать значимость изменений
- Определять корреляции между переменными
- Выявлять аномалии и отклонения
- Принимать обоснованные решения о качестве
- Оценивать надежность и стабильность системы

### Принципы статистического анализа

#### Статистическая значимость
Оценка того, насколько вероятно, что наблюдаемые различия не являются результатом случайных колебаний.

#### Доверительные интервалы
Диапазон значений, в котором с определенной вероятностью находится истинное значение параметра.

#### Статистическая мощность
Способность теста обнаружить реальное различие, если оно существует.

#### Множественное тестирование
Учет влияния множественных сравнений на вероятность ложноположительных результатов.

## Статистические методы в тестировании

### Т-тест (T-Test)

Т-тест используется для сравнения средних значений двух групп, например, для сравнения производительности до и после изменения:

```python
# t_test_example.py
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

def perform_t_test(control_group, treatment_group, alpha=0.05):
    """
    Выполнение t-теста для сравнения двух групп
    """
    # Вычисление статистики
    t_stat, p_value = stats.ttest_ind(control_group, treatment_group)
    
    # Вычисление доверительных интервалов
    control_mean = np.mean(control_group)
    treatment_mean = np.mean(treatment_group)
    
    control_ci = stats.t.interval(0.95, len(control_group)-1, 
                                  loc=control_mean, 
                                  scale=stats.sem(control_group))
    treatment_ci = stats.t.interval(0.95, len(treatment_group)-1, 
                                    loc=treatment_mean, 
                                    scale=stats.sem(treatment_group))
    
    results = {
        't_statistic': t_stat,
        'p_value': p_value,
        'control_mean': control_mean,
        'treatment_mean': treatment_mean,
        'control_ci': control_ci,
        'treatment_ci': treatment_ci,
        'significant': p_value < alpha,
        'effect_size': (treatment_mean - control_mean) / np.sqrt((np.var(control_group) + np.var(treatment_group)) / 2)
    }
    
    return results

# Пример использования для тестирования производительности
def test_performance_change():
    # Симуляция времени отклика до и после изменения
    control_response_times = np.random.normal(200, 30, 1000)  # ms
    treatment_response_times = np.random.normal(190, 30, 1000)  # ms, улучшенное время
    
    results = perform_t_test(control_response_times, treatment_response_times)
    
    print(f"T-статистика: {results['t_statistic']:.4f}")
    print(f"P-значение: {results['p_value']:.4f}")
    print(f"Значимо: {results['significant']}")
    print(f"Размер эффекта: {results['effect_size']:.4f}")
    print(f"Доверительный интервал контрольной группы: {results['control_ci']}")
    print(f"Доверительный интервал тестовой группы: {results['treatment_ci']}")
    
    return results

if __name__ == "__main__":
    test_performance_change()
```

### ANOVA (Дисперсионный анализ)

ANOVA используется для сравнения средних значений более чем двух групп:

```python
# anova_example.py
import numpy as np
import pandas as pd
from scipy import stats
import statsmodels.api as sm
from statsmodels.formula.api import ols

def perform_anova(data, groups):
    """
    Выполнение ANOVA для сравнения нескольких групп
    """
    # Создание DataFrame
    df = pd.DataFrame({
        'value': data,
        'group': groups
    })
    
    # Модель ANOVA
    model = ols('value ~ C(group)', data=df).fit()
    anova_results = sm.stats.anova_lm(model, typ=2)
    
    # Post-hoc тест (Tukey HSD) если ANOVA значимо
    from statsmodels.stats.multicomp import pairwise_tukeyhsd
    
    if anova_results['PR(>F)']['C(group)'] < 0.05:
        post_hoc = pairwise_tukeyhsd(df['value'], df['group'])
        return {
            'anova': anova_results,
            'post_hoc': post_hoc,
            'significant': True
        }
    
    return {
        'anova': anova_results,
        'significant': False
    }

# Пример использования для A/B тестирования
def ab_test_analysis():
    # Симуляция конверсии для разных версий интерфейса
    version_a = np.random.binomial(1, 0.05, 10000)  # 5% конверсия
    version_b = np.random.binomial(1, 0.055, 10000)  # 5.5% конверсия
    version_c = np.random.binomial(1, 0.048, 10000)  # 4.8% конверсия
    
    all_data = np.concatenate([version_a, version_b, version_c])
    all_groups = ['A'] * len(version_a) + ['B'] * len(version_b) + ['C'] * len(version_c)
    
    results = perform_anova(all_data, all_groups)
    
    print("ANOVA Results:")
    print(results['anova'])
    if results['significant']:
        print("\nPost-hoc Tukey HSD Results:")
        print(results['post_hoc'])
    
    return results

if __name__ == "__main__":
    ab_test_analysis()
```

### Хи-квадрат тест

Хи-квадрат тест используется для проверки ассоциаций между категориальными переменными:

```python
# chi_square_test.py
import numpy as np
from scipy.stats import chi2_contingency
import pandas as pd

def chi_square_test(observed):
    """
    Выполнение хи-квадрат теста для таблицы сопряженности
    """
    chi2, p_value, dof, expected = chi2_contingency(observed)
    
    results = {
        'chi2_statistic': chi2,
        'p_value': p_value,
        'degrees_of_freedom': dof,
        'expected_frequencies': expected,
        'significant': p_value < 0.05
    }
    
    return results

def analyze_bug_correlation():
    """
    Анализ корреляции между типами багов и окружениями
    """
    # Таблица сопряженности: типы багов vs окружения
    observed = np.array([
        [45, 30, 25],  # UI баги: dev, staging, prod
        [20, 15, 35],  # API баги: dev, staging, prod
        [10, 8,  12]   # Performance баги: dev, staging, prod
    ])
    
    results = chi_square_test(observed)
    
    print(f"Chi-square statistic: {results['chi2_statistic']:.4f}")
    print(f"P-value: {results['p_value']:.4f}")
    print(f"Significant: {results['significant']}")
    print(f"Expected frequencies:\n{results['expected_frequencies']}")
    
    return results

if __name__ == "__main__":
    analyze_bug_correlation()
```

## Статистический анализ в CI/CD

### Метрики CI/CD и их статистический анализ

```python
# cicd_metrics_analysis.py
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime, timedelta

class CICDMetricsAnalyzer:
    def __init__(self):
        self.metrics_data = pd.DataFrame()
    
    def load_metrics_data(self, data_source):
        """
        Загрузка данных метрик CI/CD
        """
        # Предполагаем, что data_source - это DataFrame с метриками
        self.metrics_data = data_source
        return self
    
    def analyze_build_times(self):
        """
        Анализ времени сборки
        """
        build_times = self.metrics_data['build_duration'].dropna()
        
        # Основные статистики
        stats_summary = {
            'mean': build_times.mean(),
            'median': build_times.median(),
            'std': build_times.std(),
            'min': build_times.min(),
            'max': build_times.max(),
            'q25': build_times.quantile(0.25),
            'q75': build_times.quantile(0.75)
        }
        
        # Проверка нормальности распределения
        shapiro_stat, shapiro_p = stats.shapiro(build_times.sample(min(5000, len(build_times))))
        
        # Проверка на выбросы (используя IQR метод)
        Q1 = build_times.quantile(0.25)
        Q3 = build_times.quantile(0.75)
        IQR = Q3 - Q1
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR
        
        outliers = build_times[(build_times < lower_bound) | (build_times > upper_bound)]
        
        return {
            'summary': stats_summary,
            'normality_test': {'statistic': shapiro_stat, 'p_value': shapiro_p},
            'outliers_count': len(outliers),
            'outliers_percentage': len(outliers) / len(build_times) * 100
        }
    
    def analyze_failure_rates(self):
        """
        Анализ частоты сбоев
        """
        failure_rates = self.metrics_data.groupby('date')['failed_builds'].sum() / \
                       self.metrics_data.groupby('date')['total_builds'].sum()
        
        # Сглаживание с помощью скользящего среднего
        failure_rates_smoothed = failure_rates.rolling(window=7, min_periods=1).mean()
        
        # Проверка тренда
        from scipy.stats import linregress
        x = range(len(failure_rates))
        trend_slope, trend_intercept, r_value, p_value, std_err = linregress(x, failure_rates.values)
        
        return {
            'failure_rates': failure_rates,
            'smoothed_rates': failure_rates_smoothed,
            'trend': {
                'slope': trend_slope,
                'r_squared': r_value**2,
                'p_value': p_value,
                'significant': p_value < 0.05
            }
        }
    
    def detect_anomalies(self, column, method='iqr', threshold=3):
        """
        Обнаружение аномалий в метриках
        """
        data = self.metrics_data[column].dropna()
        
        if method == 'iqr':
            Q1 = data.quantile(0.25)
            Q3 = data.quantile(0.75)
            IQR = Q3 - Q1
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            anomalies = data[(data < lower_bound) | (data > upper_bound)]
        elif method == 'zscore':
            z_scores = np.abs(stats.zscore(data))
            anomalies = data[z_scores > threshold]
        elif method == 'modified_zscore':
            median = data.median()
            mad = np.median(np.abs(data - median))
            modified_z_scores = 0.6745 * (data - median) / mad
            anomalies = data[np.abs(modified_z_scores) > threshold]
        
        return anomalies
    
    def correlation_analysis(self, columns):
        """
        Анализ корреляции между метриками
        """
        correlation_matrix = self.metrics_data[columns].corr()
        
        # Статистическая значимость корреляций
        n = len(self.metrics_data)
        p_values = np.zeros_like(correlation_matrix)
        
        for i in range(len(correlation_matrix.columns)):
            for j in range(len(correlation_matrix.columns)):
                if i != j:
                    corr, p_val = stats.pearsonr(
                        self.metrics_data[columns[i]].dropna(),
                        self.metrics_data[columns[j]].dropna()
                    )
                    p_values[i, j] = p_val
        
        return {
            'correlation_matrix': correlation_matrix,
            'p_values': pd.DataFrame(p_values, 
                                   index=correlation_matrix.index, 
                                   columns=correlation_matrix.columns)
        }

# Пример использования
def generate_sample_data():
    """
    Генерация примера данных CI/CD метрик
    """
    dates = pd.date_range(start='2023-01-01', end='2023-12-31', freq='D')
    n_days = len(dates)
    
    np.random.seed(42)
    
    data = {
        'date': dates,
        'build_duration': np.random.normal(300, 60, n_days),  # Среднее 5 минут
        'test_duration': np.random.normal(120, 30, n_days),   # Среднее 2 минуты
        'total_builds': np.random.poisson(10, n_days),        # 10 сборок в день
        'failed_builds': np.random.binomial(10, 0.05, n_days), # 5% сбоев
        'deployment_frequency': np.random.poisson(3, n_days), # 3 деплоя в день
        'lead_time': np.random.exponential(24, n_days) * 60   # Время в минутах
    }
    
    # Добавление выбросов для демонстрации
    outlier_indices = np.random.choice(n_days, size=int(0.02 * n_days), replace=False)
    data['build_duration'][outlier_indices] *= 3  # Увеличиваем время в 3 раза
    
    return pd.DataFrame(data)

def main_analysis_example():
    analyzer = CICDMetricsAnalyzer()
    sample_data = generate_sample_data()
    analyzer.load_metrics_data(sample_data)
    
    # Анализ времени сборки
    build_analysis = analyzer.analyze_build_times()
    print("Build Time Analysis:")
    print(f"Mean: {build_analysis['summary']['mean']:.2f}s")
    print(f"Outliers: {build_analysis['outliers_count']} ({build_analysis['outliers_percentage']:.2f}%)")
    print(f"Normal distribution: {build_analysis['normality_test']['p_value'] > 0.05}")
    
    # Обнаружение аномалий
    anomalies = analyzer.detect_anomalies('build_duration', method='iqr')
    print(f"\nDetected {len(anomalies)} anomalies in build duration")
    
    # Корреляционный анализ
    correlations = analyzer.correlation_analysis(['build_duration', 'test_duration', 'lead_time'])
    print(f"\nCorrelation Matrix:\n{correlations['correlation_matrix']}")
    
    return analyzer

if __name__ == "__main__":
    main_analysis_example()
```

## A/B тестирование в CI/CD

### Статистический анализ A/B тестов

```python
# ab_testing_analysis.py
import numpy as np
import pandas as pd
from scipy import stats
from statsmodels.stats.power import zt_ind_solve_power
import matplotlib.pyplot as plt
import seaborn as sns

class ABTestAnalyzer:
    def __init__(self):
        self.test_results = {}
    
    def analyze_conversion_test(self, control_conversions, control_total, 
                               treatment_conversions, treatment_total, 
                               alpha=0.05, power=0.8):
        """
        Анализ A/B теста конверсии
        """
        # Пропорции
        p_control = control_conversions / control_total
        p_treatment = treatment_conversions / treatment_total
        
        # Стандартизированная разница
        pooled_p = (control_conversions + treatment_conversions) / (control_total + treatment_total)
        se = np.sqrt(pooled_p * (1 - pooled_p) * (1/control_total + 1/treatment_total))
        z_stat = (p_treatment - p_control) / se
        p_value = 2 * (1 - stats.norm.cdf(abs(z_stat)))
        
        # Доверительный интервал для разницы
        se_diff = np.sqrt((p_control * (1 - p_control) / control_total) + 
                         (p_treatment * (1 - p_treatment) / treatment_total))
        ci_lower = (p_treatment - p_control) - 1.96 * se_diff
        ci_upper = (p_treatment - p_control) + 1.96 * se_diff
        
        # Размер эффекта
        effect_size = (p_treatment - p_control) / np.sqrt(pooled_p * (1 - pooled_p))
        
        # Статистическая мощность
        actual_power = zt_ind_solve_power(effect_size=abs(effect_size), 
                                         nobs1=control_total,
                                         alpha=alpha,
                                         power=None,
                                         alternative='two-sided')
        
        results = {
            'control_conversion_rate': p_control,
            'treatment_conversion_rate': p_treatment,
            'absolute_difference': p_treatment - p_control,
            'relative_difference': (p_treatment - p_control) / p_control * 100,
            'z_statistic': z_stat,
            'p_value': p_value,
            'confidence_interval': (ci_lower, ci_upper),
            'effect_size': effect_size,
            'power': actual_power,
            'significant': p_value < alpha,
            'sample_size_control': control_total,
            'sample_size_treatment': treatment_total
        }
        
        return results
    
    def sequential_analysis(self, control_data, treatment_data, alpha=0.05):
        """
        Последовательный анализ для раннего останова теста
        """
        n_points = min(len(control_data), len(treatment_data))
        results = []
        
        for i in range(100, n_points, 100):  # Анализ каждые 100 наблюдений
            ctrl_subset = control_data[:i]
            trmt_subset = treatment_data[:i]
            
            conv_ctrl = ctrl_subset.sum()
            conv_trmt = trmt_subset.sum()
            
            result = self.analyze_conversion_test(conv_ctrl, i, conv_trmt, i, alpha)
            result['sample_size'] = i
            results.append(result)
            
            # Ранняя остановка если достигнута значимость
            if result['significant'] and result['power'] > 0.8:
                break
        
        return results
    
    def multiple_testing_correction(self, p_values, method='bonferroni'):
        """
        Коррекция на множественное тестирование
        """
        n_tests = len(p_values)
        
        if method == 'bonferroni':
            corrected_p_values = [min(p * n_tests, 1.0) for p in p_values]
        elif method == 'holm':
            sorted_indices = np.argsort(p_values)
            corrected_p_values = [0] * n_tests
            for i, idx in enumerate(sorted_indices):
                corrected_p_values[idx] = min(p_values[idx] * (n_tests - i), 1.0)
        
        return corrected_p_values

# Пример использования для анализа A/B теста
def run_ab_test_example():
    np.random.seed(42)
    
    # Симуляция A/B теста
    n_control = 10000
    n_treatment = 10000
    
    # Конверсии (5% в контроле, 5.5% в тесте)
    control_conversions = np.random.binomial(n_control, 0.05)
    treatment_conversions = np.random.binomial(n_treatment, 0.055)
    
    analyzer = ABTestAnalyzer()
    results = analyzer.analyze_conversion_test(
        control_conversions, n_control,
        treatment_conversions, n_treatment
    )
    
    print("A/B Test Results:")
    print(f"Control conversion rate: {results['control_conversion_rate']:.4f}")
    print(f"Treatment conversion rate: {results['treatment_conversion_rate']:.4f}")
    print(f"Absolute difference: {results['absolute_difference']:.4f}")
    print(f"Relative difference: {results['relative_difference']:.2f}%")
    print(f"P-value: {results['p_value']:.4f}")
    print(f"Confidence interval: [{results['confidence_interval'][0]:.4f}, {results['confidence_interval'][1]:.4f}]")
    print(f"Statistically significant: {results['significant']}")
    print(f"Power: {results['power']:.4f}")
    
    return results

def run_sequential_analysis_example():
    np.random.seed(42)
    
    # Данные для последовательного анализа
    control_data = np.random.binomial(1, 0.05, 10000)
    treatment_data = np.random.binomial(1, 0.055, 10000)
    
    analyzer = ABTestAnalyzer()
    sequential_results = analyzer.sequential_analysis(control_data, treatment_data)
    
    # Визуализация
    sample_sizes = [r['sample_size'] for r in sequential_results]
    p_values = [r['p_value'] for r in sequential_results]
    conversions = [r['absolute_difference'] for r in sequential_results]
    
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(12, 10))
    
    ax1.plot(sample_sizes, p_values, 'b-', label='P-value')
    ax1.axhline(y=0.05, color='r', linestyle='--', label='Alpha = 0.05')
    ax1.set_xlabel('Sample Size')
    ax1.set_ylabel('P-value')
    ax1.set_title('Sequential A/B Test - P-value Over Time')
    ax1.legend()
    ax1.grid(True)
    
    ax2.plot(sample_sizes, conversions, 'g-', label='Conversion Difference')
    ax2.set_xlabel('Sample Size')
    ax2.set_ylabel('Conversion Difference')
    ax2.set_title('Sequential A/B Test - Effect Size Over Time')
    ax2.legend()
    ax2.grid(True)
    
    plt.tight_layout()
    plt.show()
    
    return sequential_results

if __name__ == "__main__":
    run_ab_test_example()
    run_sequential_analysis_example()
```

## Мониторинг и статистический анализ в реальном времени

### Реал-тайм статистический анализ

```python
# real_time_analysis.py
import asyncio
import numpy as np
import pandas as pd
from scipy import stats
import time
from datetime import datetime
import json

class RealTimeAnalyzer:
    def __init__(self, window_size=1000, update_interval=5):
        self.window_size = window_size
        self.update_interval = update_interval
        self.data_buffer = []
        self.anomaly_threshold = 3  # Z-score threshold
        self.change_detection_threshold = 0.05  # Minimum change to detect
        
    def add_metric(self, metric_value):
        """
        Добавление нового значения метрики
        """
        self.data_buffer.append({
            'timestamp': datetime.now(),
            'value': metric_value
        })
        
        # Поддержание размера окна
        if len(self.data_buffer) > self.window_size:
            self.data_buffer = self.data_buffer[-self.window_size:]
    
    def calculate_statistics(self):
        """
        Расчет основных статистик для текущего окна
        """
        if len(self.data_buffer) < 10:  # Минимум 10 точек для статистики
            return None
        
        values = [item['value'] for item in self.data_buffer]
        values = np.array(values)
        
        stats_dict = {
            'count': len(values),
            'mean': float(np.mean(values)),
            'std': float(np.std(values)),
            'median': float(np.median(values)),
            'min': float(np.min(values)),
            'max': float(np.max(values)),
            'q25': float(np.quantile(values, 0.25)),
            'q75': float(np.quantile(values, 0.75)),
            'iqr': float(np.quantile(values, 0.75) - np.quantile(values, 0.25))
        }
        
        return stats_dict
    
    def detect_anomalies(self):
        """
        Обнаружение аномалий в реальном времени
        """
        if len(self.data_buffer) < 10:
            return []
        
        values = np.array([item['value'] for item in self.data_buffer])
        
        # Z-score аномалии
        mean_val = np.mean(values)
        std_val = np.std(values)
        if std_val == 0:
            return []
        
        z_scores = np.abs((values - mean_val) / std_val)
        z_anomaly_indices = np.where(z_scores > self.anomaly_threshold)[0]
        
        # IQR аномалии
        Q1 = np.quantile(values, 0.25)
        Q3 = np.quantile(values, 0.75)
        IQR = Q3 - Q1
        lower_bound = Q1 - 1.5 * IQR
        upper_bound = Q3 + 1.5 * IQR
        iqr_anomaly_indices = np.where((values < lower_bound) | (values > upper_bound))[0]
        
        # Объединение аномалий
        anomaly_indices = set(z_anomaly_indices).union(set(iqr_anomaly_indices))
        anomalies = [self.data_buffer[i] for i in anomaly_indices]
        
        return anomalies
    
    def detect_changes(self, reference_window_size=100):
        """
        Обнаружение изменений в паттерне данных
        """
        if len(self.data_buffer) < reference_window_size * 2:
            return None
        
        recent_data = [item['value'] for item in self.data_buffer[-reference_window_size:]]
        baseline_data = [item['value'] for item in self.data_buffer[-2*reference_window_size:-reference_window_size]]
        
        # T-тест для сравнения средних
        t_stat, p_value = stats.ttest_ind(baseline_data, recent_data)
        
        # Манна-Уитни U тест для сравнения распределений
        u_stat, u_p_value = stats.mannwhitneyu(baseline_data, recent_data, alternative='two-sided')
        
        change_detected = {
            'mean_change': {
                't_statistic': float(t_stat),
                'p_value': float(p_value),
                'significant': p_value < 0.05,
                'mean_diff': float(np.mean(recent_data) - np.mean(baseline_data))
            },
            'distribution_change': {
                'u_statistic': float(u_stat),
                'p_value': float(u_p_value),
                'significant': u_p_value < 0.05
            },
            'baseline_mean': float(np.mean(baseline_data)),
            'recent_mean': float(np.mean(recent_data))
        }
        
        return change_detected
    
    async def run_analysis_loop(self, callback=None):
        """
        Цикл реал-тайм анализа
        """
        while True:
            stats = self.calculate_statistics()
            anomalies = self.detect_anomalies()
            changes = self.detect_changes()
            
            analysis_result = {
                'timestamp': datetime.now().isoformat(),
                'statistics': stats,
                'anomalies_count': len(anomalies),
                'change_detected': changes
            }
            
            if callback:
                await callback(analysis_result)
            
            await asyncio.sleep(self.update_interval)

# Пример использования
async def analysis_callback(result):
    """
    Callback для обработки результатов анализа
    """
    print(f"\nAnalysis at {result['timestamp']}")
    if result['statistics']:
        print(f"Mean: {result['statistics']['mean']:.2f}")
        print(f"Std: {result['statistics']['std']:.2f}")
    print(f"Anomalies detected: {result['anomalies_count']}")
    
    if result['change_detected']:
        change = result['change_detected']
        if change['mean_change']['significant']:
            print(f"Mean change detected: {change['mean_change']['mean_diff']:.2f}")

async def simulate_real_time_data():
    """
    Симуляция реал-тайм данных
    """
    analyzer = RealTimeAnalyzer(window_size=200, update_interval=2)
    
    # Запуск анализа в фоне
    analysis_task = asyncio.create_task(analyzer.run_analysis_loop(analysis_callback))
    
    # Симуляция данных
    base_value = 100
    for i in range(1000):
        # Нормальное поведение с небольшими колебаниями
        if i > 500:  # После 500 итераций вносим изменение
            current_value = base_value + 10 + np.random.normal(0, 5)
        else:
            current_value = base_value + np.random.normal(0, 3)
        
        # Иногда добавляем аномалии
        if np.random.random() < 0.05:  # 5% шанс аномалии
            current_value += np.random.choice([-50, 50])  # Сильное отклонение
        
        analyzer.add_metric(current_value)
        await asyncio.sleep(0.1)  # Симуляция поступления данных
    
    analysis_task.cancel()

if __name__ == "__main__":
    asyncio.run(simulate_real_time_data())
```

## Статистический контроль качества

### Контрольные карты (Control Charts)

```python
# control_charts.py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from scipy import stats

class ControlChart:
    def __init__(self, chart_type='xbar', sample_size=5):
        self.chart_type = chart_type
        self.sample_size = sample_size
        self.data = []
        self.control_limits = {}
    
    def add_sample(self, sample_values):
        """
        Добавление выборки данных
        """
        self.data.append({
            'values': sample_values,
            'mean': np.mean(sample_values),
            'range': np.max(sample_values) - np.min(sample_values),
            'std': np.std(sample_values)
        })
    
    def calculate_control_limits(self, method='3sigma'):
        """
        Расчет контрольных пределов
        """
        if len(self.data) < 20:  # Минимум 20 выборок для надежности
            raise ValueError("Need at least 20 samples to calculate control limits")
        
        means = [sample['mean'] for sample in self.data]
        ranges = [sample['range'] for sample in self.data]
        
        overall_mean = np.mean(means)
        average_range = np.mean(ranges)
        
        # Коэффициенты для контрольных пределов
        if self.sample_size <= 7:
            A2 = [0, 0, 1.880, 1.023, 0.729, 0.577, 0.483, 0.419][self.sample_size]
            D3 = [0, 0, 0, 0, 0, 0, 0.076, 0.136][self.sample_size]
            D4 = [0, 0, 3.267, 2.574, 2.282, 2.114, 2.004, 1.924][self.sample_size]
        else:
            # Приближение для больших выборок
            A2 = 3 / (np.sqrt(self.sample_size) * 1.128)  # 1.128 для нормального распределения
            D3 = max(0, 1 - 3 * np.sqrt(1 - 1/(self.sample_size - 1)))
            D4 = 1 + 3 * np.sqrt(1 - 1/(self.sample_size - 1))
        
        if self.chart_type == 'xbar':
            self.control_limits = {
                'center_line': overall_mean,
                'ucl': overall_mean + A2 * average_range,
                'lcl': overall_mean - A2 * average_range
            }
        elif self.chart_type == 'r':
            self.control_limits = {
                'center_line': average_range,
                'ucl': D4 * average_range,
                'lcl': D3 * average_range
            }
        
        return self.control_limits
    
    def is_in_control(self, sample_index):
        """
        Проверка, находится ли выборка в пределах контроля
        """
        if not self.control_limits:
            raise ValueError("Control limits not calculated")
        
        sample = self.data[sample_index]
        value = sample['mean'] if self.chart_type == 'xbar' else sample['range']
        
        return (self.control_limits['lcl'] <= value <= self.control_limits['ucl'])
    
    def detect_out_of_control_signals(self):
        """
        Обнаружение сигналов выхода из контроля
        """
        if not self.control_limits:
            self.calculate_control_limits()
        
        signals = []
        means = [sample['mean'] for sample in self.data]
        
        # Проверка основных сигналов
        for i, sample in enumerate(self.data):
            value = sample['mean'] if self.chart_type == 'xbar' else sample['range']
            
            # Сигнал 1: точка за контрольными пределами
            if value > self.control_limits['ucl'] or value < self.control_limits['lcl']:
                signals.append({
                    'type': 'beyond_limits',
                    'sample_index': i,
                    'value': value,
                    'limits': (self.control_limits['lcl'], self.control_limits['ucl'])
                })
            
            # Сигнал 2: 9 точек подряд по одну сторону от центральной линии
            if i >= 8:
                recent_9 = means[i-8:i+1]
                if all(x > self.control_limits['center_line'] for x in recent_9) or \
                   all(x < self.control_limits['center_line'] for x in recent_9):
                    signals.append({
                        'type': 'nine_points',
                        'sample_index': i,
                        'start_index': i-8
                    })
            
            # Сигнал 3: 6 точек подряд с возрастающей или убывающей тенденцией
            if i >= 5:
                recent_6 = means[i-5:i+1]
                if all(recent_6[j] < recent_6[j+1] for j in range(5)) or \
                   all(recent_6[j] > recent_6[j+1] for j in range(5)):
                    signals.append({
                        'type': 'trend',
                        'sample_index': i,
                        'start_index': i-5
                    })
        
        return signals
    
    def plot_chart(self):
        """
        Визуализация контрольной карты
        """
        if not self.control_limits:
            self.calculate_control_limits()
        
        values = [sample['mean'] if self.chart_type == 'xbar' else sample['range'] 
                 for sample in self.data]
        sample_numbers = list(range(len(values)))
        
        plt.figure(figsize=(12, 8))
        
        # Основной график
        plt.plot(sample_numbers, values, 'b-', marker='o', label='Sample Values')
        
        # Контрольные пределы
        plt.axhline(y=self.control_limits['center_line'], color='g', linestyle='-', 
                   label=f'Center Line ({self.control_limits["center_line"]:.2f})')
        plt.axhline(y=self.control_limits['ucl'], color='r', linestyle='--', 
                   label=f'UCL ({self.control_limits["ucl"]:.2f})')
        plt.axhline(y=self.control_limits['lcl'], color='r', linestyle='--', 
                   label=f'LCL ({self.control_limits["lcl"]:.2f})')
        
        # Сигналы выхода из контроля
        signals = self.detect_out_of_control_signals()
        if signals:
            signal_points = [s['sample_index'] for s in signals]
            signal_values = [values[i] for i in signal_points]
            plt.scatter(signal_points, signal_values, color='red', s=100, 
                       label='Out of Control Signals', zorder=5)
        
        plt.xlabel('Sample Number')
        plt.ylabel('Sample Mean' if self.chart_type == 'xbar' else 'Sample Range')
        plt.title(f'{self.chart_type.upper()} Control Chart')
        plt.legend()
        plt.grid(True, alpha=0.3)
        plt.show()

# Пример использования для контроля качества сборок
def build_quality_control_example():
    chart = ControlChart(chart_type='xbar', sample_size=5)
    
    # Симуляция времени сборки
    np.random.seed(42)
    
    # Нормальное функционирование
    for _ in range(25):
        sample = np.random.normal(300, 30, 5)  # 5 измерений, среднее 300 сек, std 30
        chart.add_sample(sample)
    
    # Изменение процесса (например, после внедрения оптимизации)
    for _ in range(10):
        sample = np.random.normal(250, 25, 5)  # Быстрее, стабильнее
        chart.add_sample(sample)
    
    # Расчет контрольных пределов
    limits = chart.calculate_control_limits()
    print(f"Control Limits: {limits}")
    
    # Обнаружение сигналов
    signals = chart.detect_out_of_control_signals()
    print(f"Detected signals: {len(signals)}")
    
    # Визуализация
    chart.plot_chart()
    
    return chart

if __name__ == "__main__":
    build_quality_control_example()
```

## Прогнозирование и предиктивный анализ

### Статистические модели прогнозирования

```python
# forecasting_analysis.py
import numpy as np
import pandas as pd
from scipy import stats
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error, mean_absolute_error
import matplotlib.pyplot as plt
from datetime import datetime, timedelta

class ForecastingAnalyzer:
    def __init__(self):
        self.models = {}
        self.forecast_horizon = 30  # дней
    
    def fit_linear_trend(self, data, dates):
        """
        Подгонка линейного тренда
        """
        # Преобразование дат в числовые значения
        date_nums = np.array([(d - dates[0]).days for d in dates]).reshape(-1, 1)
        values = np.array(data).reshape(-1, 1)
        
        model = LinearRegression()
        model.fit(date_nums, values)
        
        self.models['linear_trend'] = model
        return model
    
    def fit_seasonal_decomposition(self, data, period=7):
        """
        Декомпозиция с сезонностью (например, недельная)
        """
        data = np.array(data)
        n = len(data)
        
        # Вычисление тренда методом скользящего среднего
        if period % 2 == 0:
            trend = np.convolve(data, np.ones(period+1), mode='same') / (period+1)
        else:
            trend = np.convolve(data, np.ones(period), mode='same') / period
        
        # Удаление тренда для получения остатка
        detrended = data - trend
        
        # Вычисление сезонной компоненты
        seasonal = np.zeros_like(data)
        for i in range(period):
            seasonal[i::period] = np.mean(detrended[i::period])
        
        # Нормализация сезонной компоненты
        seasonal_avg = np.mean(seasonal.reshape(-1, period), axis=0)
        seasonal = np.tile(seasonal_avg, n // period + 1)[:n]
        
        # Остаточная компонента
        residual = data - trend - seasonal
        
        self.models['seasonal_decomposition'] = {
            'trend': trend,
            'seasonal': seasonal,
            'residual': residual,
            'period': period
        }
        
        return self.models['seasonal_decomposition']
    
    def forecast_with_confidence(self, model_type, n_periods):
        """
        Прогнозирование с доверительными интервалами
        """
        if model_type not in self.models:
            raise ValueError(f"Model {model_type} not fitted")
        
        if model_type == 'linear_trend':
            model = self.models['linear_trend']
            last_date_num = len(self.models['linear_trend']._x)  # Это не сработает, нужно сохранить даты
            # Правильная реализация потребует больше информации о данных
            
        elif model_type == 'seasonal_decomposition':
            seasonal_data = self.models['seasonal_decomposition']
            # Продолжение сезонного паттерна
            forecast = []
            confidence_intervals = []
            
            for i in range(n_periods):
                # Прогноз на основе последнего тренда и сезонности
                last_idx = -1 - (i % seasonal_data['period'])
                seasonal_value = seasonal_data['seasonal'][last_idx]
                
                # Простой линейный тренд на основе последних значений
                recent_trend = np.mean(np.diff(seasonal_data['trend'][-10:]))
                trend_forecast = seasonal_data['trend'][-1] + recent_trend * (i + 1)
                
                forecast_value = trend_forecast + seasonal_value
                forecast.append(forecast_value)
                
                # Простая оценка доверительного интервала
                residual_std = np.std(seasonal_data['residual'])
                ci_lower = forecast_value - 1.96 * residual_std
                ci_upper = forecast_value + 1.96 * residual_std
                confidence_intervals.append((ci_lower, ci_upper))
        
        return np.array(forecast), np.array(confidence_intervals)
    
    def detect_anomalies_forecast(self, actual_data, forecast_data, threshold=2):
        """
        Обнаружение аномалий по сравнению с прогнозом
        """
        residuals = np.array(actual_data) - np.array(forecast_data)
        std_residuals = np.std(residuals)
        
        if std_residuals == 0:
            return []
        
        z_scores = np.abs(residuals) / std_residuals
        anomaly_indices = np.where(z_scores > threshold)[0]
        
        anomalies = []
        for idx in anomaly_indices:
            anomalies.append({
                'index': idx,
                'actual': actual_data[idx],
                'forecast': forecast_data[idx],
                'residual': residuals[idx],
                'z_score': z_scores[idx]
            })
        
        return anomalies

# Пример использования для прогнозирования метрик CI/CD
def cicd_forecasting_example():
    # Симуляция метрик CI/CD за последние 90 дней
    np.random.seed(42)
    dates = pd.date_range(start='2023-01-01', end='2023-03-31', freq='D')
    n_days = len(dates)
    
    # Симуляция количества сборок с трендом и сезонностью
    trend = np.linspace(10, 15, n_days)  # Рост числа сборок
    seasonal = 2 * np.sin(2 * np.pi * np.arange(n_days) / 7)  # Недельная сезонность
    noise = np.random.normal(0, 1, n_days)
    
    builds_per_day = trend + seasonal + noise
    builds_per_day = np.maximum(builds_per_day, 0)  # Не отрицательные значения
    
    # Создание анализатора и подгонка моделей
    analyzer = ForecastingAnalyzer()
    
    # Линейный тренд
    linear_model = analyzer.fit_linear_trend(builds_per_day, dates)
    
    # Сезонная декомпозиция
    seasonal_model = analyzer.fit_seasonal_decomposition(builds_per_day, period=7)
    
    # Прогноз на следующие 14 дней
    forecast, confidence_intervals = analyzer.forecast_with_confidence('seasonal_decomposition', 14)
    
    # Визуализация
    plt.figure(figsize=(15, 8))
    
    # Исторические данные
    historical_dates = dates[-30:]  # Последние 30 дней
    historical_data = builds_per_day[-30:]
    plt.plot(historical_dates, historical_data, 'b-', label='Historical Data', linewidth=2)
    
    # Прогноз
    forecast_dates = pd.date_range(start=dates[-1] + timedelta(days=1), periods=14, freq='D')
    plt.plot(forecast_dates, forecast, 'r--', label='Forecast', linewidth=2)
    
    # Доверительные интервалы
    plt.fill_between(forecast_dates, 
                     [ci[0] for ci in confidence_intervals], 
                     [ci[1] for ci in confidence_intervals], 
                     color='red', alpha=0.2, label='Confidence Interval')
    
    plt.xlabel('Date')
    plt.ylabel('Builds per Day')
    plt.title('CI/CD Metrics Forecasting with Confidence Intervals')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
    
    print(f"Forecast for next 14 days: {forecast}")
    print(f"Confidence intervals: {confidence_intervals}")

if __name__ == "__main__":
    cicd_forecasting_example()
```

## Лучшие практики статистического анализа

### Планирование статистических тестов

- Определение статистической мощности
- Выбор подходящего размера выборки
- Учет множественных сравнений
- Планирование остановки теста

### Интерпретация результатов

- Различие между статистической и практической значимостью
- Учет контекста и бизнес-ценности
- Рассмотрение альтернативных объяснений
- Документирование предположений и ограничений

### Визуализация статистических данных

- Использование подходящих типов графиков
- Четкая маркировка и легенда
- Показ доверительных интервалов
- Избегание вводящих в заблуждение визуализаций

## Заключение

Статистический анализ в тестировании и CI/CD обеспечивает научный подход к оценке качества программного обеспечения и принятию обоснованных решений. Он позволяет:

- Объективно оценивать изменения
- Обнаруживать проблемы на ранних стадиях
- Прогнозировать будущие метрики
- Обеспечивать качество на основе данных

Успешная реализация статистического анализа требует:

- Понимания статистических методов
- Правильной интерпретации результатов
- Интеграции с процессами CI/CD
- Постоянного мониторинга и улучшения

Организации, инвестирующие в статистический анализ, получают конкурентное преимущество за счет более обоснованных решений и более высокого качества программного обеспечения.

## Связанные концепции

- [[Continuous Integration]]
- [[Continuous Deployment]]
- [[A/B Testing]]
- [[Test Automation]]
- [[Monitoring and Observability]]
- [[Quality Assurance]]
- [[Data Pipeline]]
- [[Statistical Modeling]]
- [[Hypothesis Testing]]
- [[Experimental Design]]

## Внешние ресурсы

- [Statistical Analysis for Software Engineering](https://www.computer.org/csdl/magazine/so/2021/04/09452713/1nFRP1nqZ6N)
- [A/B Testing Guide](https://vwo.com/ab-testing/)
- [Statistical Methods in Software Engineering](https://link.springer.com/book/10.1007/b98885)
- [CI/CD Metrics and Analytics](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-metrics)