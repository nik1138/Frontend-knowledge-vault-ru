---
aliases: [Продуктовая аналитика, Product Analytics, Аналитика продукта]
tags: 
  - analytics
  - ci-cd
  - experimentation
  - business-intelligence
  - data-science
  - user-behavior
---

# Продуктовая аналитика в CI/CD

## Обзор

Продуктовая аналитика в CI/CD - это систематический подход к сбору, анализу и интерпретации данных о поведении пользователей и характеристиках продукта в контексте непрерывной интеграции и доставки. Она обеспечивает объективную основу для принятия решений о развитии продукта, позволяя командам понимать влияние изменений на пользовательский опыт и бизнес-метрики. В рамках CI/CD продуктовая аналитика интегрируется в процесс доставки программного обеспечения, обеспечивая обратную связь о качестве изменений и их воздействии на пользователей.

Продуктовая аналитика в CI/CD сочетает в себе методы анализа данных, статистики и машинного обучения для измерения эффективности продуктовых изменений, выявления паттернов поведения пользователей и оптимизации пользовательского опыта. Это критически важно для быстрой и безопасной доставки ценностей пользователям.

## Основы продуктовой аналитики

### Что такое продуктовая аналитика

Продуктовая аналитика - это область аналитики, сосредоточенная на понимании того, как пользователи взаимодействуют с продуктом, какие функции наиболее ценны, и как изменения влияют на пользовательский опыт и бизнес-результаты. Она включает:

- Отслеживание пользовательских событий
- Анализ воронок конверсии
- Измерение ключевых метрик продукта
- Сегментацию пользователей
- А/В тестирование
- Анализ удержания и вовлеченности

### Принципы продуктовой аналитики

#### Ориентация на пользователя
Анализ должен фокусироваться на реальном поведении пользователей и их потребностях.

#### Основанный на данных подход
Решения должны приниматься на основе объективных данных, а не предположений.

#### Целостность картины
Необходимо учитывать как количественные, так и качественные метрики.

#### Быстрая обратная связь
Системы аналитики должны обеспечивать своевременную обратную связь о внесенных изменениях.

## Ключевые метрики продуктовой аналитики

### DAU и MAU (Daily/Monthly Active Users)

```python
# user_metrics_calculator.py
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from typing import Dict, List, Tuple
import matplotlib.pyplot as plt
import seaborn as sns

class UserMetricsCalculator:
    def __init__(self, user_events_df: pd.DataFrame):
        """
        Инициализация калькулятора пользовательских метрик
        user_events_df должен содержать колонки: user_id, event_timestamp, event_type
        """
        self.events = user_events_df.copy()
        self.events['event_date'] = pd.to_datetime(self.events['event_timestamp']).dt.date
        self.events['event_datetime'] = pd.to_datetime(self.events['event_timestamp'])
    
    def calculate_dau(self, date: datetime.date = None) -> int:
        """
        Расчет DAU (Daily Active Users) для конкретной даты
        """
        if date is None:
            date = datetime.now().date()
        
        dau = self.events[self.events['event_date'] == date]['user_id'].nunique()
        return int(dau)
    
    def calculate_dau_trend(self, days_back: int = 30) -> pd.DataFrame:
        """
        Расчет тренда DAU за последние N дней
        """
        end_date = datetime.now().date()
        start_date = end_date - timedelta(days=days_back)
        
        dau_trend = self.events[
            (self.events['event_date'] >= start_date) & 
            (self.events['event_date'] <= end_date)
        ].groupby('event_date')['user_id'].nunique().reset_index()
        
        dau_trend.columns = ['date', 'dau']
        return dau_trend
    
    def calculate_mau(self, year: int = None, month: int = None) -> int:
        """
        Расчет MAU (Monthly Active Users) для конкретного месяца
        """
        if year is None or month is None:
            year = datetime.now().year
            month = datetime.now().month
        
        # Получаем все даты в месяце
        import calendar
        last_day = calendar.monthrange(year, month)[1]
        start_date = datetime(year, month, 1).date()
        end_date = datetime(year, month, last_day).date()
        
        mau = self.events[
            (self.events['event_date'] >= start_date) & 
            (self.events['event_date'] <= end_date)
        ]['user_id'].nunique()
        
        return int(mau)
    
    def calculate_wau(self, year: int = None, week: int = None) -> int:
        """
        Расчет WAU (Weekly Active Users)
        """
        if year is None:
            year = datetime.now().year
        if week is None:
            week = datetime.now().isocalendar()[1]  # Неделя года
        
        # Находим даты начала и конца недели
        start_of_week = datetime.strptime(f'{year}-{week}-1', "%Y-%W-%w").date()
        end_of_week = start_of_week + timedelta(days=6)
        
        wau = self.events[
            (self.events['event_date'] >= start_of_week) & 
            (self.events['event_date'] <= end_of_week)
        ]['user_id'].nunique()
        
        return int(wau)
    
    def calculate_user_retention(self, cohort_period: str = 'daily') -> pd.DataFrame:
        """
        Расчет удержания пользователей по когортам
        """
        # Создаем когорты
        self.events['cohort_date'] = self.events.groupby('user_id')['event_date'].transform('min')
        self.events['period_number'] = (
            self.events['event_date'] - self.events['cohort_date']
        ).dt.days
        
        if cohort_period == 'weekly':
            self.events['cohort_date'] = self.events['cohort_date'] - pd.to_timedelta(
                self.events['cohort_date'].dt.dayofweek, unit='d'
            )
            self.events['period_number'] = (
                (self.events['event_date'] - self.events['cohort_date']).dt.days // 7
            )
        
        # Считаем размеры когорт
        cohort_sizes = self.events.groupby('cohort_date')['user_id'].nunique().reset_index()
        cohort_sizes.columns = ['cohort_date', 'total_users']
        
        # Считаем активных пользователей в каждом периоде для каждой когорты
        retention = self.events.groupby(['cohort_date', 'period_number'])['user_id'].nunique().reset_index()
        retention.columns = ['cohort_date', 'period_number', 'active_users']
        
        # Объединяем с размерами когорт
        retention = retention.merge(cohort_sizes, on='cohort_date', how='left')
        retention['retention_rate'] = retention['active_users'] / retention['total_users']
        
        return retention
    
    def calculate_engagement_metrics(self) -> Dict[str, float]:
        """
        Расчет метрик вовлеченности
        """
        # Среднее количество сессий на пользователя в день
        daily_sessions = self.events.groupby(['user_id', 'event_date']).size().reset_index(name='sessions')
        avg_sessions_per_user_per_day = daily_sessions['sessions'].mean()
        
        # Средняя продолжительность сессии (если есть время начала и окончания)
        # Для примера предположим, что у нас есть время сессии
        session_data = self.events.groupby(['user_id', 'event_date']).agg({
            'event_datetime': ['min', 'max']
        }).reset_index()
        session_data.columns = ['user_id', 'event_date', 'session_start', 'session_end']
        session_data['session_duration'] = (
            session_data['session_end'] - session_data['session_start']
        ).dt.total_seconds() / 60  # в минутах
        
        avg_session_duration = session_data['session_duration'].mean()
        
        # Частота использования
        user_activity = self.events.groupby('user_id')['event_date'].nunique()
        avg_frequency = user_activity.mean()
        
        return {
            'avg_sessions_per_user_per_day': float(avg_sessions_per_user_per_day),
            'avg_session_duration_minutes': float(avg_session_duration),
            'avg_days_active_per_user': float(avg_frequency),
            'total_unique_users': int(self.events['user_id'].nunique())
        }

# Пример использования
def example_user_metrics():
    # Симуляция данных пользовательских событий
    np.random.seed(42)
    
    # Создаем датасет с событиями пользователей
    n_users = 1000
    n_events = 10000
    date_range = pd.date_range(start='2023-01-01', end='2023-03-31', freq='D')
    
    user_ids = np.random.choice(range(1, n_users + 1), size=n_events)
    event_dates = np.random.choice(date_range, size=n_events)
    event_timestamps = [date + pd.Timedelta(minutes=np.random.randint(0, 1440)) for date in event_dates]
    event_types = np.random.choice(['page_view', 'click', 'purchase', 'signup'], size=n_events)
    
    events_df = pd.DataFrame({
        'user_id': user_ids,
        'event_timestamp': event_timestamps,
        'event_type': event_types
    })
    
    calculator = UserMetricsCalculator(events_df)
    
    # Расчет метрик
    current_dau = calculator.calculate_dau()
    print(f"Current DAU: {current_dau}")
    
    dau_trend = calculator.calculate_dau_trend(days_back=30)
    print(f"DAU trend (last 30 days):\n{dau_trend.tail()}")
    
    current_mau = calculator.calculate_mau()
    print(f"Current MAU: {current_mau}")
    
    engagement_metrics = calculator.calculate_engagement_metrics()
    print(f"Engagement metrics: {engagement_metrics}")
    
    # Расчет удержания
    retention = calculator.calculate_user_retention()
    print(f"Retention data shape: {retention.shape}")
    print(f"First few retention records:\n{retention.head()}")
    
    return calculator

if __name__ == "__main__":
    example_user_metrics()
```

### Воронка конверсии

```python
# conversion_funnel.py
import pandas as pd
import numpy as np
from typing import Dict, List, Optional
import matplotlib.pyplot as plt
import plotly.graph_objects as go
import plotly.express as px

class ConversionFunnelAnalyzer:
    def __init__(self, events_df: pd.DataFrame, user_id_col: str = 'user_id', 
                 event_type_col: str = 'event_type', timestamp_col: str = 'timestamp'):
        self.events = events_df
        self.user_id_col = user_id_col
        self.event_type_col = event_type_col
        self.timestamp_col = timestamp_col
    
    def calculate_funnel(self, steps: List[str], start_date: Optional[str] = None, 
                        end_date: Optional[str] = None) -> pd.DataFrame:
        """
        Расчет воронки конверсии по указанным шагам
        """
        # Фильтрация по дате, если указаны
        events = self.events.copy()
        if start_date:
            events = events[events[self.timestamp_col] >= start_date]
        if end_date:
            events = events[events[self.timestamp_col] <= end_date]
        
        # Сортировка событий по времени для каждого пользователя
        events = events.sort_values([self.user_id_col, self.timestamp_col])
        
        # Для каждого шага находим уникальных пользователей
        funnel_data = []
        
        for i, step in enumerate(steps):
            if i == 0:
                # Первый шаг - все пользователи, совершившие это событие
                step_users = events[events[self.event_type_col] == step][self.user_id_col].unique()
            else:
                # Последующие шаги - пользователи, которые совершили предыдущий шаг и текущий
                prev_step_users = funnel_data[i-1]['users']
                current_step_events = events[events[self.event_type_col] == step]
                step_users = np.intersect1d(prev_step_users, 
                                          current_step_events[self.user_id_col].unique())
            
            step_count = len(step_users)
            
            funnel_data.append({
                'step': step,
                'users': step_users,
                'count': step_count,
                'percentage': step_count / funnel_data[0]['count'] * 100 if i > 0 else 100.0
            })
        
        funnel_df = pd.DataFrame(funnel_data)
        return funnel_df
    
    def calculate_step_to_step_conversion(self, steps: List[str]) -> pd.DataFrame:
        """
        Расчет конверсии между шагами
        """
        funnel_df = self.calculate_funnel(steps)
        
        conversion_rates = []
        for i in range(1, len(funnel_df)):
            prev_count = funnel_df.iloc[i-1]['count']
            current_count = funnel_df.iloc[i]['count']
            conversion_rate = (current_count / prev_count * 100) if prev_count > 0 else 0
            
            conversion_rates.append({
                'from_step': funnel_df.iloc[i-1]['step'],
                'to_step': funnel_df.iloc[i]['step'],
                'conversion_rate': conversion_rate,
                'drop_off': prev_count - current_count,
                'drop_off_percentage': (prev_count - current_count) / prev_count * 100 if prev_count > 0 else 0
            })
        
        return pd.DataFrame(conversion_rates)
    
    def analyze_cohort_funnel(self, steps: List[str], cohort_column: str = 'cohort_date') -> pd.DataFrame:
        """
        Анализ воронки по когортам
        """
        # Создаем когорты по дате первого события
        first_events = self.events.groupby(self.user_id_col).agg({
            self.timestamp_col: 'min',
            self.event_type_col: lambda x: x.iloc[0]
        }).reset_index()
        first_events['cohort_date'] = pd.to_datetime(first_events[self.timestamp_col]).dt.date
        
        # Добавляем когорту к основным событиям
        events_with_cohort = self.events.merge(
            first_events[[self.user_id_col, 'cohort_date']], 
            on=self.user_id_col
        )
        
        # Для каждой когорты рассчитываем воронку
        cohorts = events_with_cohort['cohort_date'].unique()
        cohort_funnels = []
        
        for cohort in cohorts:
            cohort_events = events_with_cohort[events_with_cohort['cohort_date'] == cohort]
            
            if len(cohort_events) > 0:
                # Рассчитываем воронку для этой когорты
                cohort_funnel = self.calculate_funnel(steps)
                cohort_funnel['cohort_date'] = cohort
                cohort_funnels.append(cohort_funnel)
        
        return pd.concat(cohort_funnels, ignore_index=True) if cohort_funnels else pd.DataFrame()
    
    def visualize_funnel(self, funnel_df: pd.DataFrame) -> go.Figure:
        """
        Визуализация воронки
        """
        fig = go.Figure(go.Funnel(
            y=funnel_df['step'],
            x=funnel_df['count'],
            textinfo="value+percent initial"
        ))
        
        fig.update_layout(
            title="Conversion Funnel",
            xaxis_title="Number of Users",
            yaxis_title="Funnel Steps"
        )
        
        return fig
    
    def identify_bottlenecks(self, steps: List[str], threshold: float = 20.0) -> List[Dict]:
        """
        Идентификация узких мест в воронке
        """
        conversion_rates = self.calculate_step_to_step_conversion(steps)
        
        bottlenecks = []
        for _, row in conversion_rates.iterrows():
            if row['conversion_rate'] < threshold:
                bottlenecks.append({
                    'from_step': row['from_step'],
                    'to_step': row['to_step'],
                    'conversion_rate': row['conversion_rate'],
                    'drop_off': row['drop_off'],
                    'issue': f"Low conversion from {row['from_step']} to {row['to_step']}: {row['conversion_rate']:.2f}%"
                })
        
        return bottlenecks

# Пример использования
def example_conversion_funnel():
    # Симуляция данных событий
    np.random.seed(42)
    
    # Создаем симулированные данные для воронки регистрации: visit -> signup -> activate -> purchase
    n_users = 1000
    base_events = []
    
    for user_id in range(1, n_users + 1):
        # Каждый пользователь посещает сайт
        base_events.append({
            'user_id': user_id,
            'event_type': 'visit',
            'timestamp': f'2023-01-{np.random.randint(1, 28):02d} 10:00:00'
        })
        
        # 80% переходит к регистрации
        if np.random.random() < 0.8:
            base_events.append({
                'user_id': user_id,
                'event_type': 'signup',
                'timestamp': f'2023-01-{np.random.randint(1, 28):02d} 10:05:00'
            })
            
            # 60% активирует аккаунт
            if np.random.random() < 0.6:
                base_events.append({
                    'user_id': user_id,
                    'event_type': 'activate',
                    'timestamp': f'2023-01-{np.random.randint(1, 28):02d} 10:10:00'
                })
                
                # 30% совершает покупку
                if np.random.random() < 0.3:
                    base_events.append({
                        'user_id': user_id,
                        'event_type': 'purchase',
                        'timestamp': f'2023-01-{np.random.randint(1, 28):02d} 10:15:00'
                    })
    
    events_df = pd.DataFrame(base_events)
    analyzer = ConversionFunnelAnalyzer(events_df)
    
    # Определяем шаги воронки
    funnel_steps = ['visit', 'signup', 'activate', 'purchase']
    
    # Рассчитываем воронку
    funnel = analyzer.calculate_funnel(funnel_steps)
    print("Conversion Funnel:")
    print(funnel)
    
    # Рассчитываем конверсию между шагами
    conversion_rates = analyzer.calculate_step_to_step_conversion(funnel_steps)
    print(f"\nStep-to-step conversion:")
    print(conversion_rates)
    
    # Идентификация узких мест
    bottlenecks = analyzer.identify_bottlenecks(funnel_steps, threshold=50.0)
    print(f"\nBottlenecks identified: {len(bottlenecks)}")
    for bottleneck in bottlenecks:
        print(f"  {bottleneck['issue']}")
    
    return analyzer

if __name__ == "__main__":
    example_conversion_funnel()
```

### Удержание пользователей

```python
# retention_analyzer.py
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import matplotlib.pyplot as plt
import seaborn as sns
from typing import Dict, List, Optional

class UserRetentionAnalyzer:
    def __init__(self, events_df: pd.DataFrame, user_id_col: str = 'user_id', 
                 date_col: str = 'event_date'):
        self.events = events_df.copy()
        self.user_id_col = user_id_col
        self.date_col = date_col
        
        # Преобразуем дату в datetime, если нужно
        if not pd.api.types.is_datetime64_any_dtype(self.events[self.date_col]):
            self.events[self.date_col] = pd.to_datetime(self.events[self.date_col]).dt.date
    
    def calculate_retention_cohort(self, period: str = 'daily') -> pd.DataFrame:
        """
        Расчет когортного анализа удержания
        """
        # Определяем когорту (первый день активности) для каждого пользователя
        user_cohorts = self.events.groupby(self.user_id_col)[self.date_col].min().reset_index()
        user_cohorts.columns = [self.user_id_col, 'cohort_date']
        
        # Объединяем с событиями
        events_with_cohort = self.events.merge(user_cohorts, on=self.user_id_col)
        
        # Вычисляем номер периода (день/неделя/месяц) с начала когорты
        events_with_cohort['cohort_date'] = pd.to_datetime(events_with_cohort['cohort_date'])
        events_with_cohort[self.date_col] = pd.to_datetime(events_with_cohort[self.date_col])
        
        if period == 'daily':
            events_with_cohort['period_number'] = (
                events_with_cohort[self.date_col] - events_with_cohort['cohort_date']
            ).dt.days
        elif period == 'weekly':
            events_with_cohort['period_number'] = (
                events_with_cohort[self.date_col] - events_with_cohort['cohort_date']
            ).dt.days // 7
        elif period == 'monthly':
            # Более сложный расчет для месяцев
            events_with_cohort['period_number'] = (
                (events_with_cohort[self.date_col].dt.to_period('M') - 
                 events_with_cohort['cohort_date'].dt.to_period('M')).apply(lambda x: x.n)
            )
        
        # Считаем уникальных пользователей в каждом периоде для каждой когорты
        retention_data = events_with_cohort.groupby(
            ['cohort_date', 'period_number']
        )[self.user_id_col].nunique().reset_index()
        retention_data.columns = ['cohort_date', 'period_number', 'active_users']
        
        # Считаем размеры когорт (пользователи в периоде 0)
        cohort_sizes = events_with_cohort[
            events_with_cohort['period_number'] == 0
        ].groupby('cohort_date')[self.user_id_col].nunique().reset_index()
        cohort_sizes.columns = ['cohort_date', 'cohort_size']
        
        # Объединяем данные
        retention_matrix = retention_data.merge(cohort_sizes, on='cohort_date', how='left')
        retention_matrix['retention_rate'] = retention_matrix['active_users'] / retention_matrix['cohort_size']
        
        # Создаем матрицу удержания
        retention_pivot = pd.pivot_table(
            retention_matrix,
            values='retention_rate',
            index='cohort_date',
            columns='period_number',
            fill_value=0
        )
        
        return retention_pivot
    
    def calculate_n_day_retention(self, n_days: int = 1, period: str = 'daily') -> pd.DataFrame:
        """
        Расчет N-day удержания
        """
        # Определяем когорты
        user_cohorts = self.events.groupby(self.user_id_col)[self.date_col].min().reset_index()
        user_cohorts.columns = [self.user_id_col, 'cohort_date']
        
        # Находим следующую активность через N дней
        events_with_cohort = self.events.merge(user_cohorts, on=self.user_id_col)
        events_with_cohort['cohort_date'] = pd.to_datetime(events_with_cohort['cohort_date'])
        events_with_cohort[self.date_col] = pd.to_datetime(events_with_cohort[self.date_col])
        
        # Находим пользователей, которые были активны через N дней
        events_with_cohort['target_date'] = events_with_cohort['cohort_date'] + timedelta(days=n_days)
        
        # Проверяем, были ли пользователи активны в целевой день или позже
        retained_users = events_with_cohort[
            events_with_cohort[self.date_col] >= events_with_cohort['target_date']
        ][[self.user_id_col, 'cohort_date']].drop_duplicates()
        
        # Считаем размеры когорт и удержание
        cohort_sizes = user_cohorts.groupby('cohort_date')[self.user_id_col].nunique()
        retained_counts = retained_users.groupby('cohort_date')[self.user_id_col].nunique()
        
        retention_df = pd.DataFrame({
            'cohort_size': cohort_sizes,
            'retained_users': retained_counts,
        }).fillna(0)
        
        retention_df['retention_rate'] = retention_df['retained_users'] / retention_df['cohort_size']
        retention_df = retention_df.reset_index()
        
        return retention_df
    
    def calculate_cohort_analysis(self) -> Dict:
        """
        Полный когортный анализ
        """
        daily_retention = self.calculate_retention_cohort('daily')
        weekly_retention = self.calculate_retention_cohort('weekly')
        
        # Статистики
        avg_1day_retention = daily_retention.get(1, pd.Series()).mean() if 1 in daily_retention.columns else 0
        avg_7day_retention = weekly_retention.get(1, pd.Series()).mean() if 1 in weekly_retention.columns else 0
        
        return {
            'daily_retention_matrix': daily_retention,
            'weekly_retention_matrix': weekly_retention,
            'average_1day_retention': float(avg_1day_retention),
            'average_7day_retention': float(avg_7day_retention),
            'cohort_sizes': daily_retention.iloc[:, 0] if not daily_retention.empty else pd.Series()
        }
    
    def identify_retention_patterns(self) -> Dict:
        """
        Идентификация паттернов удержания
        """
        retention_data = self.calculate_cohort_analysis()
        weekly_matrix = retention_data['weekly_retention_matrix']
        
        patterns = {}
        
        # Тренды удержания
        if not weekly_matrix.empty:
            # Рассчитываем среднее удержание по периодам
            period_retention = weekly_matrix.mean(axis=0).dropna()
            patterns['period_retention_trend'] = period_retention.to_dict()
            
            # Определяем стабильный уровень удержания
            if len(period_retention) > 4:
                stable_retention = period_retention.iloc[3:].mean()  # Среднее после 4 периодов
                patterns['stable_retention_rate'] = float(stable_retention)
        
        # Когорты с лучшим/худшим удержанием
        if not weekly_matrix.empty and len(weekly_matrix) > 0:
            best_cohort = weekly_matrix.iloc[:, 0].idxmax() if 0 in weekly_matrix.columns else None
            worst_cohort = weekly_matrix.iloc[:, 0].idxmin() if 0 in weekly_matrix.columns else None
            
            patterns['best_retention_cohort'] = str(best_cohort) if best_cohort is not None else None
            patterns['worst_retention_cohort'] = str(worst_cohort) if worst_cohort is not None else None
        
        return patterns
    
    def visualize_retention_heatmap(self, retention_matrix: pd.DataFrame, title: str = "Retention Heatmap"):
        """
        Визуализация матрицы удержания в виде тепловой карты
        """
        plt.figure(figsize=(12, 8))
        sns.heatmap(
            retention_matrix.T, 
            annot=True, 
            fmt='.2%', 
            cmap='RdYlGn', 
            center=0.5,
            cbar_kws={'label': 'Retention Rate'}
        )
        plt.title(title)
        plt.xlabel('Cohort Date')
        plt.ylabel('Period Number')
        plt.xticks(rotation=45)
        plt.tight_layout()
        plt.show()

# Пример использования
def example_retention_analysis():
    # Симуляция данных пользовательской активности
    np.random.seed(42)
    
    # Создаем данные для нескольких когорт
    data = []
    base_date = pd.to_datetime('2023-01-01')
    
    for cohort_week in range(4):  # 4 недели когорт
        cohort_start = base_date + pd.Timedelta(weeks=cohort_week)
        
        # 100 пользователей в каждой когорте
        for user_id in range(100 * cohort_week, 100 * (cohort_week + 1)):
            # Дата первой активности (когорта)
            first_active = cohort_start
            
            # Добавляем первого события
            data.append({
                'user_id': user_id,
                'event_date': first_active.date()
            })
            
            # Симулируем последующую активность с убывающей вероятностью
            for day_offset in range(1, 15):  # 14 дней наблюдения
                # Вероятность активности убывает со временем
                retention_prob = 0.8 * (0.9 ** day_offset)  # Экспоненциальное убывание
                
                if np.random.random() < retention_prob:
                    event_date = first_active + pd.Timedelta(days=day_offset)
                    data.append({
                        'user_id': user_id,
                        'event_date': event_date.date()
                    })
    
    events_df = pd.DataFrame(data)
    analyzer = UserRetentionAnalyzer(events_df)
    
    # Когортный анализ
    cohort_analysis = analyzer.calculate_cohort_analysis()
    print("Cohort Analysis:")
    print(f"Average 1-day retention: {cohort_analysis['average_1day_retention']:.2%}")
    print(f"Average 7-day retention: {cohort_analysis['average_7day_retention']:.2%}")
    
    # Паттерны удержания
    patterns = analyzer.identify_retention_patterns()
    print(f"\nRetention Patterns: {patterns}")
    
    # Матрица удержания
    retention_matrix = cohort_analysis['daily_retention_matrix']
    print(f"\nRetention Matrix shape: {retention_matrix.shape}")
    print(f"First few rows:\n{retention_matrix.head()}")
    
    # Визуализация (закомментировано для предотвращения отображения при запуске)
    # analyzer.visualize_retention_heatmap(retention_matrix.head(5).iloc[:, :7], "Sample Retention Heatmap")
    
    return analyzer

if __name__ == "__main__":
    example_retention_analysis()
```

## Интеграция с CI/CD

### Метрики продуктовой аналитики в CI/CD

```python
# cicd_product_analytics.py
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import requests
import json
from typing import Dict, List, Optional
import time
from dataclasses import dataclass

@dataclass
class DeploymentEvent:
    deployment_id: str
    timestamp: datetime
    environment: str
    version: str
    features: List[str]

@dataclass
class ProductMetric:
    name: str
    value: float
    timestamp: datetime
    deployment_id: Optional[str] = None

class CICDProductAnalytics:
    def __init__(self, analytics_api_url: str, ci_cd_api_url: str):
        self.analytics_api_url = analytics_api_url
        self.ci_cd_api_url = ci_cd_api_url
        self.metrics_history = []
        self.deployments = []
    
    def track_deployment(self, deployment_event: DeploymentEvent):
        """
        Отслеживание события деплоя
        """
        self.deployments.append(deployment_event)
        
        # Отправка события в аналитическую систему
        deployment_data = {
            'deployment_id': deployment_event.deployment_id,
            'timestamp': deployment_event.timestamp.isoformat(),
            'environment': deployment_event.environment,
            'version': deployment_event.version,
            'features': deployment_event.features,
            'event_type': 'deployment'
        }
        
        try:
            response = requests.post(
                f"{self.analytics_api_url}/events",
                json=deployment_data
            )
            response.raise_for_status()
        except requests.RequestException as e:
            print(f"Error sending deployment event: {e}")
    
    def collect_product_metrics(self, deployment_id: str = None) -> List[ProductMetric]:
        """
        Сбор продуктовых метрик, связанных с деплоем
        """
        # Симуляция сбора метрик
        metrics = []
        
        # DAU
        dau = np.random.poisson(10000)  # Случайное значение для примера
        metrics.append(ProductMetric(
            name='dau',
            value=float(dau),
            timestamp=datetime.now(),
            deployment_id=deployment_id
        ))
        
        # Conversion rate
        conversion_rate = np.random.beta(2, 8)  # От 0 до 1, смещение к низким значениям
        metrics.append(ProductMetric(
            name='conversion_rate',
            value=float(conversion_rate),
            timestamp=datetime.now(),
            deployment_id=deployment_id
        ))
        
        # Session duration
        avg_session_duration = np.random.gamma(2, 2)  # В минутах
        metrics.append(ProductMetric(
            name='avg_session_duration_minutes',
            value=float(avg_session_duration),
            timestamp=datetime.now(),
            deployment_id=deployment_id
        ))
        
        # Error rate
        error_rate = np.random.beta(1, 50)  # Очень низкий
        metrics.append(ProductMetric(
            name='error_rate',
            value=float(error_rate),
            timestamp=datetime.now(),
            deployment_id=deployment_id
        ))
        
        self.metrics_history.extend(metrics)
        return metrics
    
    def analyze_deployment_impact(self, deployment_id: str) -> Dict:
        """
        Анализ влияния деплоя на продуктовые метрики
        """
        # Находим метрики до и после деплоя
        deployment = next((d for d in self.deployments if d.deployment_id == deployment_id), None)
        if not deployment:
            raise ValueError(f"Deployment {deployment_id} not found")
        
        # Симуляция анализа влияния
        before_metrics = self.get_metrics_around_time(deployment.timestamp - timedelta(hours=1))
        after_metrics = self.get_metrics_around_time(deployment.timestamp + timedelta(minutes=30))
        
        impact_analysis = {
            'deployment_id': deployment_id,
            'deployment_time': deployment.timestamp.isoformat(),
            'metric_changes': {},
            'significant_changes': [],
            'risk_level': 'low'  # будет рассчитано ниже
        }
        
        # Сравнение метрик
        for metric_name in ['dau', 'conversion_rate', 'error_rate']:
            before_value = self.get_metric_value(before_metrics, metric_name)
            after_value = self.get_metric_value(after_metrics, metric_name)
            
            if before_value and after_value:
                change = ((after_value - before_value) / before_value) * 100 if before_value != 0 else 0
                
                impact_analysis['metric_changes'][metric_name] = {
                    'before': before_value,
                    'after': after_value,
                    'change_percent': change,
                    'significant': abs(change) > 5  # значительное изменение > 5%
                }
                
                if abs(change) > 5:
                    impact_analysis['significant_changes'].append({
                        'metric': metric_name,
                        'change': change,
                        'direction': 'increase' if change > 0 else 'decrease'
                    })
        
        # Оценка риска
        negative_changes = [c for c in impact_analysis['significant_changes'] 
                           if c['metric'] in ['error_rate'] or 
                              (c['metric'] in ['conversion_rate', 'dau'] and c['direction'] == 'decrease')]
        
        if len(negative_changes) > 2:
            impact_analysis['risk_level'] = 'high'
        elif len(negative_changes) > 0:
            impact_analysis['risk_level'] = 'medium'
        else:
            impact_analysis['risk_level'] = 'low'
        
        return impact_analysis
    
    def get_metrics_around_time(self, target_time: datetime, window_minutes: int = 30) -> List[ProductMetric]:
        """
        Получение метрик в окрестности определенного времени
        """
        time_window_start = target_time - timedelta(minutes=window_minutes//2)
        time_window_end = target_time + timedelta(minutes=window_minutes//2)
        
        return [m for m in self.metrics_history 
                if time_window_start <= m.timestamp <= time_window_end]
    
    def get_metric_value(self, metrics: List[ProductMetric], metric_name: str) -> Optional[float]:
        """
        Получение значения метрики по имени
        """
        metric_values = [m.value for m in metrics if m.name == metric_name]
        return np.mean(metric_values) if metric_values else None
    
    def generate_deployment_report(self, deployment_id: str) -> Dict:
        """
        Генерация отчета о деплое
        """
        impact = self.analyze_deployment_impact(deployment_id)
        
        report = {
            'deployment_summary': {
                'id': deployment_id,
                'risk_level': impact['risk_level'],
                'significant_changes_count': len(impact['significant_changes']),
                'timestamp': impact['deployment_time']
            },
            'metric_impact': impact['metric_changes'],
            'recommendations': self._generate_recommendations(impact),
            'confidence_score': self._calculate_confidence_score(impact)
        }
        
        return report
    
    def _generate_recommendations(self, impact_analysis: Dict) -> List[str]:
        """
        Генерация рекомендаций на основе анализа влияния
        """
        recommendations = []
        
        if impact_analysis['risk_level'] == 'high':
            recommendations.append("Consider immediate rollback due to significant negative impact")
        
        for change in impact_analysis['significant_changes']:
            if change['direction'] == 'decrease' and change['metric'] in ['dau', 'conversion_rate']:
                recommendations.append(f"Investigate decrease in {change['metric']}")
            elif change['direction'] == 'increase' and change['metric'] == 'error_rate':
                recommendations.append("High error rate detected, investigate immediately")
            elif change['direction'] == 'increase' and change['metric'] in ['dau', 'conversion_rate']:
                recommendations.append(f"Positive {change['metric']} impact confirmed, consider expanding")
        
        if not recommendations:
            recommendations.append("No significant changes detected, deployment appears stable")
        
        return recommendations
    
    def _calculate_confidence_score(self, impact_analysis: Dict) -> float:
        """
        Расчет оценки уверенности в анализе
        """
        # Чем больше метрик, тем выше уверенность
        # Чем больше значительных изменений, тем ниже уверенность (нужно больше анализа)
        num_metrics = len(impact_analysis['metric_changes'])
        num_significant_changes = len(impact_analysis['significant_changes'])
        
        base_score = min(1.0, num_metrics * 0.2)  # Максимум 1.0 для 5+ metrics
        change_penalty = min(0.5, num_significant_changes * 0.1)  # Пенальти за значительные изменения
        
        confidence = max(0.0, base_score - change_penalty)
        return min(1.0, max(0.0, confidence))

# Пример использования
def example_cicd_analytics():
    # Инициализация системы аналитики
    analytics = CICDProductAnalytics(
        analytics_api_url="http://analytics-api.local",
        ci_cd_api_url="http://cicd-api.local"
    )
    
    # Симуляция деплоя
    deployment = DeploymentEvent(
        deployment_id="deploy_20231111_001",
        timestamp=datetime.now() - timedelta(minutes=30),
        environment="production",
        version="v1.2.3",
        features=["new_checkout_flow", "improved_search"]
    )
    
    analytics.track_deployment(deployment)
    
    # Сбор метрик до и после деплоя
    before_metrics = analytics.collect_product_metrics()
    time.sleep(1)  # Имитация времени между деплоем и сбором метрик
    
    # Симуляция изменения метрик после деплоя
    after_metrics = analytics.collect_product_metrics(deployment.deployment_id)
    
    # Анализ влияния
    impact_analysis = analytics.analyze_deployment_impact(deployment.deployment_id)
    print("Impact Analysis:")
    print(f"Risk Level: {impact_analysis['risk_level']}")
    print(f"Significant Changes: {impact_analysis['significant_changes']}")
    print(f"Metric Changes: {impact_analysis['metric_changes']}")
    
    # Генерация отчета
    report = analytics.generate_deployment_report(deployment.deployment_id)
    print(f"\nDeployment Report: {report}")
    
    return analytics

if __name__ == "__main__":
    example_cicd_analytics()
```

### Автоматические триггеры на основе метрик

```python
# metric_triggers.py
import numpy as np
import pandas as pd
from datetime import datetime, timedelta
from typing import Dict, List, Callable, Any
import time
import json

class MetricTriggerSystem:
    def __init__(self):
        self.triggers = []
        self.trigger_history = []
        self.current_metrics = {}
    
    def add_trigger(self, name: str, metric_name: str, condition: str, 
                   threshold: float, action: Callable, 
                   lookback_window: int = 5, cooldown_period: int = 300):
        """
        Добавление триггера на основе метрики
        
        condition: 'above', 'below', 'equal', 'change_percent'
        """
        trigger = {
            'name': name,
            'metric_name': metric_name,
            'condition': condition,
            'threshold': threshold,
            'action': action,
            'lookback_window': lookback_window,  # в измерениях метрики
            'cooldown_period': cooldown_period,  # в секундах
            'last_triggered': None,
            'enabled': True
        }
        
        self.triggers.append(trigger)
        return trigger
    
    def add_metric_value(self, metric_name: str, value: float, timestamp: datetime = None):
        """
        Добавление нового значения метрики
        """
        if timestamp is None:
            timestamp = datetime.now()
        
        if metric_name not in self.current_metrics:
            self.current_metrics[metric_name] = []
        
        self.current_metrics[metric_name].append({
            'value': value,
            'timestamp': timestamp
        })
        
        # Ограничение истории метрики
        if len(self.current_metrics[metric_name]) > 100:  # Хранить максимум 100 значений
            self.current_metrics[metric_name] = self.current_metrics[metric_name][-50:]
    
    def check_triggers(self) -> List[Dict]:
        """
        Проверка всех активных триггеров
        """
        triggered_events = []
        
        for trigger in self.triggers:
            if not trigger['enabled']:
                continue
            
            # Проверка периода охлаждения
            if (trigger['last_triggered'] and 
                (datetime.now() - trigger['last_triggered']).total_seconds() < trigger['cooldown_period']):
                continue
            
            # Получение истории метрики
            metric_history = self.current_metrics.get(trigger['metric_name'], [])
            if len(metric_history) < trigger['lookback_window']:
                continue
            
            # Выбор значений для проверки
            recent_values = [m['value'] for m in metric_history[-trigger['lookback_window']:]]
            current_value = recent_values[-1]
            
            # Проверка условия
            triggered = False
            if trigger['condition'] == 'above':
                triggered = current_value > trigger['threshold']
            elif trigger['condition'] == 'below':
                triggered = current_value < trigger['threshold']
            elif trigger['condition'] == 'equal':
                triggered = abs(current_value - trigger['threshold']) < 0.001
            elif trigger['condition'] == 'change_percent':
                if len(recent_values) >= 2:
                    previous_value = recent_values[-2]
                    if previous_value != 0:
                        change_percent = abs((current_value - previous_value) / previous_value) * 100
                        triggered = change_percent > trigger['threshold']
            
            if triggered:
                event = {
                    'trigger_name': trigger['name'],
                    'metric_name': trigger['metric_name'],
                    'current_value': current_value,
                    'threshold': trigger['threshold'],
                    'timestamp': datetime.now(),
                    'action_result': None
                }
                
                try:
                    # Выполнение действия
                    result = trigger['action'](event)
                    event['action_result'] = result
                    event['success'] = True
                except Exception as e:
                    event['error'] = str(e)
                    event['success'] = False
                
                triggered_events.append(event)
                trigger['last_triggered'] = datetime.now()
                self.trigger_history.append(event)
        
        return triggered_events
    
    def create_rollback_trigger(self, metric_name: str, threshold: float, 
                               deployment_manager) -> Dict:
        """
        Создание триггера для автоматического отката
        """
        def rollback_action(event: Dict):
            print(f"Triggering rollback due to {event['metric_name']} degradation")
            return deployment_manager.rollback_last_deployment()
        
        return self.add_trigger(
            name=f"auto_rollback_{metric_name}",
            metric_name=metric_name,
            condition='below',
            threshold=threshold,
            action=rollback_action,
            lookback_window=3,
            cooldown_period=600  # 10 минут охлаждения после отката
        )

# Пример системы деплоя для демонстрации
class MockDeploymentManager:
    def __init__(self):
        self.deployments = []
        self.current_version = "v1.0.0"
    
    def rollback_last_deployment(self):
        if self.deployments:
            last_deployment = self.deployments.pop()
            print(f"Rolling back deployment: {last_deployment}")
            # Здесь была бы реальная логика отката
            return {"status": "rolled_back", "version": self.current_version}
        else:
            return {"status": "no_deployments_to_rollback"}

# Пример использования системы триггеров
def example_metric_triggers():
    trigger_system = MetricTriggerSystem()
    deployment_manager = MockDeploymentManager()
    
    # Добавление триггера для высокого уровня ошибок
    def alert_on_high_error_rate(event: Dict):
        print(f"ALERT: High error rate detected: {event['current_value']:.4f}")
        return {"alert_sent": True}
    
    trigger_system.add_trigger(
        name="high_error_rate_alert",
        metric_name="error_rate",
        condition="above",
        threshold=0.05,  # 5% error rate
        action=alert_on_high_error_rate,
        lookback_window=2,
        cooldown_period=300
    )
    
    # Добавление триггера для падения конверсии
    def conversion_drop_action(event: Dict):
        print(f"Conversion rate dropped significantly: {event['current_value']:.4f}")
        return {"investigation_started": True}
    
    trigger_system.add_trigger(
        name="conversion_drop",
        metric_name="conversion_rate",
        condition="change_percent",
        threshold=20.0,  # 20% change
        action=conversion_drop_action,
        lookback_window=2,
        cooldown_period=600
    )
    
    # Симуляция данных метрик
    print("Simulating metric values and checking triggers...")
    
    for i in range(20):
        # Симуляция нормальных значений
        error_rate = np.random.beta(1, 50)  # Очень низкий error rate
        conversion_rate = np.random.beta(2, 8)  # Относительно стабильный conversion rate
        
        # В какой-то момент искусственно увеличиваем error rate
        if i == 15:
            error_rate = 0.08  # Выше порога 0.05
        
        trigger_system.add_metric_value("error_rate", error_rate)
        trigger_system.add_metric_value("conversion_rate", conversion_rate)
        
        # Проверка триггеров
        triggered = trigger_system.check_triggers()
        if triggered:
            print(f"Triggers fired at iteration {i}: {triggered}")
        
        time.sleep(0.1)  # Имитация времени между измерениями
    
    print(f"Total trigger events: {len(trigger_system.trigger_history)}")
    
    return trigger_system

if __name__ == "__main__":
    example_metric_triggers()
```

## А/В тестирование и продуктовая аналитика

### Интеграция A/B тестов с продуктовой аналитикой

```python
# ab_test_analytics.py
import pandas as pd
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns
from typing import Dict, List, Tuple
from datetime import datetime, timedelta

class ABTestAnalytics:
    def __init__(self, events_df: pd.DataFrame):
        self.events = events_df
        self.experiments = {}
    
    def setup_ab_test(self, experiment_id: str, 
                     variant_column: str = 'variant',
                     user_column: str = 'user_id',
                     timestamp_column: str = 'timestamp'):
        """
        Настройка A/B теста для анализа
        """
        self.experiments[experiment_id] = {
            'variant_column': variant_column,
            'user_column': user_column,
            'timestamp_column': timestamp_column,
            'start_time': self.events[timestamp_column].min(),
            'end_time': self.events[timestamp_column].max()
        }
    
    def calculate_ab_metrics(self, experiment_id: str, 
                           success_metric: str,
                           variant_column: str = 'variant',
                           user_column: str = 'user_id') -> Dict:
        """
        Расчет метрик для A/B теста
        """
        experiment = self.experiments.get(experiment_id)
        if not experiment:
            raise ValueError(f"Experiment {experiment_id} not found")
        
        # Фильтрация событий для данного эксперимента
        experiment_events = self.events.copy()
        
        # Группировка по вариантам и пользователю для подсчета успехов
        variant_successes = experiment_events[
            experiment_events['event_type'] == success_metric
        ].groupby([variant_column, user_column]).size().reset_index(name='success_count')
        
        # Подсчет уникальных пользователей в каждом варианте
        variant_users = experiment_events.groupby(variant_column)[user_column].nunique().reset_index()
        variant_users.columns = [variant_column, 'total_users']
        
        # Подсчет успехов в каждом варианте
        variant_success_stats = variant_successes.groupby(variant_column)['success_count'].agg([
            'count',  # количество пользователей с успехами
            'sum',    # общее количество успехов
            'mean'    # среднее количество успехов на пользователя
        ]).reset_index()
        variant_success_stats.columns = [
            variant_column, 'users_with_success', 'total_successes', 'avg_successes_per_user'
        ]
        
        # Объединение данных
        results = variant_users.merge(variant_success_stats, on=variant_column, how='left')
        results['success_rate'] = results['users_with_success'] / results['total_users']
        
        # Заполнение NaN для вариантов без успехов
        results = results.fillna(0)
        
        # Статистический тест (t-test для пропорций)
        variants = results[variant_column].unique()
        if len(variants) >= 2:
            variant1 = results[results[variant_column] == variants[0]]
            variant2 = results[results[variant_column] == variants[1]]
            
            # T-тест для сравнения конверсий
            success1 = variant1['users_with_success'].iloc[0] if len(variant1) > 0 else 0
            total1 = variant1['total_users'].iloc[0] if len(variant1) > 0 else 1
            success2 = variant2['users_with_success'].iloc[0] if len(variant2) > 0 else 0
            total2 = variant2['total_users'].iloc[0] if len(variant2) > 0 else 1
            
            # Пропорции
            p1 = success1 / total1
            p2 = success2 / total2
            
            # Стандартизированная разница
            pooled_p = (success1 + success2) / (total1 + total2)
            se = np.sqrt(pooled_p * (1 - pooled_p) * (1/total1 + 1/total2))
            
            if se != 0:
                z_stat = (p2 - p1) / se
                p_value = 2 * (1 - stats.norm.cdf(abs(z_stat)))
            else:
                z_stat, p_value = 0, 1
            
            statistical_significance = {
                'z_statistic': float(z_stat),
                'p_value': float(p_value),
                'significant': p_value < 0.05,
                'confidence_level': 0.95
            }
        else:
            statistical_significance = {
                'z_statistic': 0,
                'p_value': 1,
                'significant': False,
                'confidence_level': 0.95
            }
        
        return {
            'results': results.to_dict('records'),
            'statistical_significance': statistical_significance,
            'effect_size': float(p2 - p1) if 'p1' in locals() and 'p2' in locals() else 0
        }
    
    def calculate_segmented_ab_metrics(self, experiment_id: str,
                                     success_metric: str,
                                     segment_column: str,
                                     variant_column: str = 'variant') -> Dict:
        """
        Расчет A/B метрик с сегментацией
        """
        experiment_events = self.events.copy()
        
        # Добавляем информацию о сегментах
        segmented_events = experiment_events.merge(
            experiment_events[[segment_column, variant_column]].drop_duplicates(),
            on=[segment_column, variant_column]
        )
        
        # Подсчет успехов по сегментам и вариантам
        success_events = segmented_events[
            segmented_events['event_type'] == success_metric
        ]
        
        segmented_successes = success_events.groupby(
            [segment_column, variant_column]
        ).size().reset_index(name='success_count')
        
        # Подсчет общего количества пользователей по сегментам и вариантам
        segmented_users = experiment_events.groupby(
            [segment_column, variant_column]
        )[self.experiments[experiment_id]['user_column']].nunique().reset_index()
        segmented_users.columns = [segment_column, variant_column, 'total_users']
        
        # Объединение данных
        segmented_results = segmented_users.merge(segmented_successes, 
                                                on=[segment_column, variant_column], 
                                                how='left')
        segmented_results['success_rate'] = (
            segmented_results['success_count'] / segmented_results['total_users']
        )
        segmented_results['success_rate'] = segmented_results['success_rate'].fillna(0)
        
        return segmented_results.to_dict('records')
    
    def analyze_ab_test_time_series(self, experiment_id: str,
                                  success_metric: str,
                                  variant_column: str = 'variant',
                                  time_window: str = 'D') -> pd.DataFrame:
        """
        Анализ A/B теста в разрезе времени
        """
        experiment_events = self.events.copy()
        experiment_events['event_date'] = pd.to_datetime(
            experiment_events[self.experiments[experiment_id]['timestamp_column']]
        ).dt.to_period(time_window)
        
        # Фильтрация событий успеха
        success_events = experiment_events[
            experiment_events['event_type'] == success_metric
        ]
        
        # Подсчет успехов по датам и вариантам
        daily_successes = success_events.groupby(
            [variant_column, 'event_date']
        ).size().reset_index(name='successes')
        
        # Подсчет уникальных пользователей по датам и вариантам
        daily_users = experiment_events.groupby(
            [variant_column, 'event_date']
        )[self.experiments[experiment_id]['user_column']].nunique().reset_index()
        daily_users.columns = [variant_column, 'event_date', 'users']
        
        # Объединение данных
        time_series = daily_users.merge(daily_successes, 
                                       on=[variant_column, 'event_date'], 
                                       how='left')
        time_series['success_rate'] = time_series['successes'] / time_series['users']
        time_series['success_rate'] = time_series['success_rate'].fillna(0)
        
        return time_series
    
    def visualize_ab_results(self, experiment_id: str, success_metric: str):
        """
        Визуализация результатов A/B теста
        """
        results = self.calculate_ab_metrics(experiment_id, success_metric)
        
        fig, axes = plt.subplots(2, 2, figsize=(15, 12))
        
        results_df = pd.DataFrame(results['results'])
        
        # Гистограмма конверсии по вариантам
        axes[0, 0].bar(results_df[results_df.columns[0]], results_df['success_rate'])
        axes[0, 0].set_title('Success Rate by Variant')
        axes[0, 0].set_ylabel('Success Rate')
        axes[0, 0].tick_params(axis='x', rotation=45)
        
        # Количество пользователей по вариантам
        axes[0, 1].bar(results_df[results_df.columns[0]], results_df['total_users'])
        axes[0, 1].set_title('Total Users by Variant')
        axes[0, 1].set_ylabel('User Count')
        axes[0, 1].tick_params(axis='x', rotation=45)
        
        # Количество успехов по вариантам
        axes[1, 0].bar(results_df[results_df.columns[0]], results_df['users_with_success'])
        axes[1, 0].set_title('Users with Success by Variant')
        axes[1, 0].set_ylabel('Success Count')
        axes[1, 0].tick_params(axis='x', rotation=45)
        
        # Статистическая значимость
        stats_data = [results['statistical_significance']['p_value'], 
                     0.05]  # p-value и уровень значимости
        axes[1, 1].bar(['P-value', 'Alpha'], stats_data)
        axes[1, 1].axhline(y=0.05, color='r', linestyle='--', label='Significance Level')
        axes[1, 1].set_title('Statistical Significance')
        axes[1, 1].set_ylabel('Value')
        
        plt.tight_layout()
        plt.show()

# Пример использования
def example_ab_test_analytics():
    # Симуляция данных A/B теста
    np.random.seed(42)
    
    n_users = 10000
    variants = ['control', 'treatment']
    
    # Создание данных для A/B теста
    data = []
    
    for variant in variants:
        # Количество пользователей в варианте
        if variant == 'control':
            variant_users = 5000
            success_rate = 0.12  # 12% конверсия
        else:
            variant_users = 5000
            success_rate = 0.14  # 14% конверсия (2% улучшение)
        
        for user_id in range(variant_users):
            user_id_global = len(data)
            
            # Активность пользователя
            n_sessions = np.random.poisson(3)  # 3 сессии в среднем
            for session in range(n_sessions):
                # Дата сессии
                session_date = pd.Timestamp('2023-01-01') + pd.Timedelta(days=np.random.randint(0, 30))
                
                # Добавляем событие просмотра
                data.append({
                    'user_id': user_id_global,
                    'variant': variant,
                    'event_type': 'page_view',
                    'timestamp': session_date + pd.Timedelta(minutes=np.random.randint(0, 1440))
                })
                
                # С вероятностью success_rate пользователь конвертится
                if np.random.random() < success_rate:
                    data.append({
                        'user_id': user_id_global,
                        'variant': variant,
                        'event_type': 'conversion',
                        'timestamp': session_date + pd.Timedelta(minutes=np.random.randint(1, 1440))
                    })
    
    events_df = pd.DataFrame(data)
    analytics = ABTestAnalytics(events_df)
    
    # Настройка A/B теста
    analytics.setup_ab_test('checkout_test')
    
    # Расчет метрик
    ab_results = analytics.calculate_ab_metrics('checkout_test', 'conversion')
    print("A/B Test Results:")
    for result in ab_results['results']:
        print(f"Variant {result['variant']}: Success Rate = {result['success_rate']:.4f}")
    
    print(f"Statistical Significance: {ab_results['statistical_significance']}")
    print(f"Effect Size: {ab_results['effect_size']:.4f}")
    
    # Визуализация (закомментировано для предотвращения отображения при запуске)
    # analytics.visualize_ab_results('checkout_test', 'conversion')
    
    return analytics

if __name__ == "__main__":
    example_ab_test_analytics()
```

## Мониторинг продуктовых метрик

### Реал-тайм мониторинг

```python
# real_time_monitoring.py
import asyncio
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import time
from typing import Dict, List, Callable
import json

class RealTimeProductMetricsMonitor:
    def __init__(self):
        self.metrics_buffer = []
        self.alerts_history = []
        self.metrics_config = {}
        self.callbacks = []
    
    def add_metric_config(self, metric_name: str, 
                         warning_threshold: float,
                         critical_threshold: float,
                         metric_type: str = 'gauge'):
        """
        Добавление конфигурации метрики
        """
        self.metrics_config[metric_name] = {
            'warning_threshold': warning_threshold,
            'critical_threshold': critical_threshold,
            'metric_type': metric_type,
            'last_value': None,
            'last_alert_time': None
        }
    
    def add_event(self, event_data: Dict):
        """
        Добавление события в буфер
        """
        event_data['timestamp'] = datetime.now()
        self.metrics_buffer.append(event_data)
        
        # Ограничение размера буфера
        if len(self.metrics_buffer) > 10000:
            self.metrics_buffer = self.metrics_buffer[-5000:]
    
    async def process_events_stream(self, events_stream: List[Dict]):
        """
        Обработка потока событий в реальном времени
        """
        for event in events_stream:
            self.add_event(event)
            await self.check_alerts(event)
    
    async def check_alerts(self, event: Dict):
        """
        Проверка условий срабатывания алертов
        """
        # Пример проверки метрик на основе событий
        if event.get('event_type') == 'error':
            error_rate = self.calculate_error_rate(window_minutes=5)
            if 'error_rate' in self.metrics_config:
                config = self.metrics_config['error_rate']
                if error_rate > config['critical_threshold']:
                    await self.trigger_alert('error_rate_critical', error_rate, 'CRITICAL')
                elif error_rate > config['warning_threshold']:
                    await self.trigger_alert('error_rate_warning', error_rate, 'WARNING')
    
    async def trigger_alert(self, alert_type: str, value: float, severity: str):
        """
        Срабатывание алерта
        """
        alert = {
            'alert_type': alert_type,
            'value': value,
            'severity': severity,
            'timestamp': datetime.now(),
            'status': 'triggered'
        }
        
        self.alerts_history.append(alert)
        
        # Вызов зарегистрированных callback'ов
        for callback in self.callbacks:
            try:
                await callback(alert)
            except Exception as e:
                print(f"Error in alert callback: {e}")
    
    def calculate_error_rate(self, window_minutes: int = 5) -> float:
        """
        Расчет уровня ошибок за указанный период
        """
        time_threshold = datetime.now() - timedelta(minutes=window_minutes)
        recent_events = [e for e in self.metrics_buffer 
                        if e['timestamp'] >= time_threshold]
        
        if not recent_events:
            return 0.0
        
        total_events = len(recent_events)
        error_events = len([e for e in recent_events if e.get('event_type') == 'error'])
        
        return error_events / total_events if total_events > 0 else 0.0
    
    def calculate_conversion_rate(self, window_minutes: int = 5) -> float:
        """
        Расчет конверсии за указанный период
        """
        time_threshold = datetime.now() - timedelta(minutes=window_minutes)
        recent_events = [e for e in self.metrics_buffer 
                        if e['timestamp'] >= time_threshold]
        
        if not recent_events:
            return 0.0
        
        page_views = len([e for e in recent_events if e.get('event_type') == 'page_view'])
        conversions = len([e for e in recent_events if e.get('event_type') == 'conversion'])
        
        return conversions / page_views if page_views > 0 else 0.0
    
    def register_alert_callback(self, callback: Callable):
        """
        Регистрация callback для обработки алертов
        """
        self.callbacks.append(callback)
    
    async def run_monitoring_loop(self):
        """
        Цикл мониторинга в реальном времени
        """
        while True:
            # Симуляция получения новых событий
            new_events = self.generate_mock_events(10)
            await self.process_events_stream(new_events)
            
            # Проверка метрик
            await self.check_periodic_metrics()
            
            await asyncio.sleep(1)  # Обновление каждую секунду
    
    def generate_mock_events(self, count: int) -> List[Dict]:
        """
        Генерация симуляционных событий
        """
        events = []
        for _ in range(count):
            event_type = np.random.choice(
                ['page_view', 'click', 'conversion', 'error'],
                p=[0.7, 0.2, 0.08, 0.02]  # 2% ошибок
            )
            
            events.append({
                'event_type': event_type,
                'user_id': np.random.randint(1, 1000),
                'session_id': f"session_{np.random.randint(1, 100)}",
                'value': np.random.random() if event_type == 'value_event' else None
            })
        
        return events
    
    async def check_periodic_metrics(self):
        """
        Проверка метрик по расписанию
        """
        # Проверка DAU
        dau = self.calculate_dau_last_hour()
        if 'dau' in self.metrics_config:
            config = self.metrics_config['dau']
            if dau < config['warning_threshold']:
                await self.trigger_alert('dau_low', dau, 'WARNING')
        
        # Проверка конверсии
        conversion_rate = self.calculate_conversion_rate(window_minutes=10)
        if 'conversion_rate' in self.metrics_config:
            config = self.metrics_config['conversion_rate']
            if conversion_rate < config['warning_threshold']:
                await self.trigger_alert('conversion_rate_low', conversion_rate, 'WARNING')
            elif conversion_rate > config['critical_threshold']:
                await self.trigger_alert('conversion_rate_high', conversion_rate, 'INFO')
    
    def calculate_dau_last_hour(self) -> int:
        """
        Расчет DAU за последний час
        """
        time_threshold = datetime.now() - timedelta(hours=1)
        recent_events = [e for e in self.metrics_buffer 
                        if e['timestamp'] >= time_threshold]
        
        unique_users = set()
        for event in recent_events:
            if 'user_id' in event:
                unique_users.add(event['user_id'])
        
        return len(unique_users)

# Пример использования реал-тайм мониторинга
async def example_real_time_monitoring():
    monitor = RealTimeProductMetricsMonitor()
    
    # Настройка метрик
    monitor.add_metric_config('error_rate', 0.05, 0.1)  # 5% warning, 10% critical
    monitor.add_metric_config('conversion_rate', 0.02, 0.05)  # 2% warning, 5% critical
    monitor.add_metric_config('dau', 1000, 500)  # 1000 warning, 500 critical
    
    # Регистрация callback для уведомлений
    async def alert_notification_callback(alert):
        print(f"ALERT: {alert['alert_type']} - Value: {alert['value']:.4f}, Severity: {alert['severity']}")
    
    monitor.register_alert_callback(alert_notification_callback)
    
    print("Starting real-time monitoring...")
    print("Simulating events and checking metrics...")
    
    # Симуляция работы мониторинга
    for i in range(10):  # 10 итераций для демонстрации
        new_events = monitor.generate_mock_events(20)
        await monitor.process_events_stream(new_events)
        await monitor.check_periodic_metrics()
        
        # Печать текущих метрик
        print(f"Iteration {i+1}:")
        print(f"  Error Rate: {monitor.calculate_error_rate():.4f}")
        print(f"  Conversion Rate: {monitor.calculate_conversion_rate():.4f}")
        print(f"  DAU (last hour): {monitor.calculate_dau_last_hour()}")
        print(f"  Total alerts triggered: {len(monitor.alerts_history)}")
        
        await asyncio.sleep(0.5)  # Небольшая задержка
    
    print(f"\nTotal alerts during monitoring: {len(monitor.alerts_history)}")
    
    return monitor

if __name__ == "__main__":
    asyncio.run(example_real_time_monitoring())
```

## Лучшие практики продуктовой аналитики

### Планирование и реализация

- Определение ключевых бизнес-метрик
- Установка базовых показателей
- Планирование событий слежения
- Обеспечение качества данных

### Мониторинг и анализ

- Непрерывный мониторинг метрик
- Быстрое реагирование на аномалии
- Регулярный анализ трендов
- Документирование инсайтов

### Интеграция с процессами

- Интеграция с CI/CD пайплайнами
- Автоматические триггеры на основе метрик
- Отчетность и визуализация
- Обратная связь с командами разработки

## Заключение

Продуктовая аналитика в CI/CD обеспечивает объективную основу для принятия решений о развитии продукта. Она позволяет командам:

- Понимать влияние изменений на пользователей
- Быстро реагировать на проблемы
- Оптимизировать пользовательский опыт
- Улучшать бизнес-результаты

Успешная реализация продуктовой аналитики требует:

- Четкого понимания бизнес-целей
- Надежной инфраструктуры сбора данных
- Эффективных процессов анализа
- Интеграции с CI/CD процессами

Организации, инвестирующие в продуктовую аналитику, получают конкурентное преимущество за счет более обоснованных решений и быстрого реагирования на изменения.

## Связанные концепции

- [[A/B Testing]]
- [[Statistical Analysis]]
- [[Continuous Integration]]
- [[Continuous Deployment]]
- [[Experiment Design]]
- [[Monitoring and Observability]]
- [[Data Pipeline]]
- [[User Experience]]

## Внешние ресурсы

- [Product Analytics Guide](https://www.mixpanel.com/product-analytics/)
- [A/B Testing Best Practices](https://vwo.com/ab-testing/)
- [Google Analytics for Firebase](https://firebase.google.com/docs/analytics)
- [Amplitude Product Analytics](https://www.amplitude.com/product-analytics)