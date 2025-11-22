---
aliases: [Управление рисками, Risk Management, Риск-менеджмент]
tags: 
  - risk-management
  - ci-cd
  - security
  - compliance
  - governance
  - quality-assurance
---

# Управление рисками в CI/CD

## Обзор

Управление рисками в CI/CD - это систематический подход к идентификации, оценке, контролю и мониторингу рисков, связанных с процессами непрерывной интеграции и доставки программного обеспечения. Это критически важный аспект обеспечения надежности, безопасности и качества доставки программного обеспечения. Управление рисками в CI/CD включает в себя как технические, так и организационные аспекты, обеспечивая, что изменения в коде могут быть безопасно и надежно доставлены в продакшен среду с минимальным воздействием на бизнес и пользователей.

В современных условиях, когда изменения программного обеспечения происходят часто и быстро, управление рисками становится особенно важным. Оно позволяет командам принимать обоснованные решения, минимизировать потенциальный ущерб и обеспечивать стабильность систем.

## Основы управления рисками

### Что такое риск в CI/CD

Риск в контексте CI/CD - это потенциальная возможность того, что событие или действие в процессе доставки программного обеспечения приведет к нежелательным последствиям, таким как:

- Сбои в работе системы
- Утечки данных
- Нарушение безопасности
- Нарушение соответствия требованиям
- Потеря дохода
- Повреждение репутации

### Принципы управления рисками

#### Идентификация рисков
Систематическое выявление потенциальных угроз и уязвимостей в процессах CI/CD.

#### Оценка рисков
Анализ вероятности и воздействия рисков с определением приоритетов.

#### Контроль рисков
Реализация мер для снижения вероятности и/или воздействия рисков.

#### Мониторинг и отчетность
Постоянное наблюдение за рисками и создание отчетов для заинтересованных сторон.

#### Культура безопасности
Создание среды, в которой управление рисками является коллективной ответственностью.

## Типы рисков в CI/CD

### Технические риски

#### Риски безопасности

```yaml
# security-risk-assessment.yaml
security_risks:
  code_injection:
    probability: 0.3
    impact: 9
    severity: high
    controls:
      - static_code_analysis
      - dependency_scanning
      - runtime_protection
    mitigation_strategies:
      - input_validation
      - output_encoding
      - principle_of_least_privilege

  dependency_vulnerabilities:
    probability: 0.7
    impact: 8
    severity: high
    controls:
      - snyk_integration
      - dependabot_updates
      - vulnerability_scanning
    mitigation_strategies:
      - automated_dependency_updates
      - vulnerability_management_policy
      - supply_chain_verification

  misconfigured_infrastructure:
    probability: 0.4
    impact: 7
    severity: medium
    controls:
      - infrastructure_as_code_validation
      - security_policy_enforcement
      - configuration_scanning
    mitigation_strategies:
      - terraform_security_scanning
      - configuration_management_standards
      - automated_policy_enforcement
```

#### Риски производительности

```yaml
# performance-risk-assessment.yaml
performance_risks:
  resource_exhaustion:
    probability: 0.2
    impact: 8
    severity: high
    controls:
      - resource_limiting
      - auto_scaling
      - performance_monitoring
    metrics:
      - cpu_utilization
      - memory_usage
      - disk_io
      - network_bandwidth

  scalability_issues:
    probability: 0.3
    impact: 6
    severity: medium
    controls:
      - load_testing
      - capacity_planning
      - horizontal_scaling
    metrics:
      - concurrent_users
      - response_time
      - throughput
      - error_rate

  deployment_failures:
    probability: 0.1
    impact: 9
    severity: high
    controls:
      - canary_deployments
      - health_checks
      - rollback_procedures
    metrics:
      - deployment_success_rate
      - rollback_frequency
      - downtime_duration
```

### Операционные риски

#### Риски процесса доставки

```yaml
# delivery-process-risks.yaml
delivery_process_risks:
  insufficient_testing:
    probability: 0.5
    impact: 7
    severity: high
    controls:
      - automated_testing
      - code_coverage_requirements
      - manual_qa_processes
    prevention_measures:
      - mandatory_test_coverage_thresholds
      - peer_code_reviews
      - automated_quality_gates

  deployment_coordination:
    probability: 0.3
    impact: 6
    severity: medium
    controls:
      - deployment_scheduling
      - communication_protocols
      - incident_response_plans
    prevention_measures:
      - deployment_windows
      - stakeholder_notifications
      - coordination_tools

  rollback_complexity:
    probability: 0.4
    impact: 8
    severity: high
    controls:
      - automated_rollback_procedures
      - blue_green_deployments
      - feature_flags
    prevention_measures:
      - reversible_architecture
      - data_migration_strategies
      - backward_compatibility
```

### Бизнес-риски

#### Риски соответствия требованиям

```yaml
# compliance-risk-assessment.yaml
compliance_risks:
  gdpr_non_compliance:
    probability: 0.2
    impact: 10
    severity: critical
    controls:
      - data_classification
      - privacy_controls
      - audit_logging
    compliance_frameworks:
      - gdpr
      - ccpa
      - hipaa

  sox_compliance:
    probability: 0.1
    impact: 9
    severity: critical
    controls:
      - change_management
      - access_controls
      - financial_reporting
    compliance_frameworks:
      - sox
      - soc2

  industry_regulations:
    probability: 0.3
    impact: 8
    severity: high
    controls:
      - regulatory_monitoring
      - compliance_automation
      - audit_preparation
    compliance_frameworks:
      - pci_dss
      - iso_27001
      - nist_csf
```

## Матрица рисков

### Оценка вероятности и воздействия

```python
# risk_matrix.py
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

class RiskMatrix:
    def __init__(self):
        # Матрица вероятности x воздействия
        self.matrix = np.array([
            [1, 2, 3, 4, 5],  # Very Low Impact
            [2, 3, 4, 5, 6],  # Low Impact
            [3, 4, 5, 6, 7],  # Medium Impact
            [4, 5, 6, 7, 8],  # High Impact
            [5, 6, 7, 8, 9]   # Very High Impact
        ])
        
        self.probability_levels = {
            'Very Low': 0.1,
            'Low': 0.25,
            'Medium': 0.5,
            'High': 0.75,
            'Very High': 0.9
        }
        
        self.impact_levels = {
            'Very Low': 1,
            'Low': 2,
            'Medium': 3,
            'High': 4,
            'Very High': 5
        }
        
        self.risk_categories = {
            1: 'Very Low',
            2: 'Low', 
            3: 'Low',
            4: 'Medium',
            5: 'Medium',
            6: 'High',
            7: 'High',
            8: 'Very High',
            9: 'Very High'
        }

    def calculate_risk_score(self, probability: float, impact: int) -> int:
        """Рассчитать риск на основе вероятности и воздействия"""
        prob_idx = self._get_probability_index(probability)
        impact_idx = self._get_impact_index(impact)
        return self.matrix[prob_idx][impact_idx]

    def _get_probability_index(self, probability: float) -> int:
        """Получить индекс вероятности (0-4)"""
        if probability <= 0.2:
            return 0  # Very Low
        elif probability <= 0.4:
            return 1  # Low
        elif probability <= 0.6:
            return 2  # Medium
        elif probability <= 0.8:
            return 3  # High
        else:
            return 4  # Very High

    def _get_impact_index(self, impact: int) -> int:
        """Получить индекс воздействия (0-4)"""
        return min(impact - 1, 4)

    def categorize_risk(self, risk_score: int) -> str:
        """Категоризировать риск по значению"""
        return self.risk_categories.get(risk_score, 'Unknown')

    def visualize_matrix(self):
        """Визуализировать матрицу рисков"""
        plt.figure(figsize=(10, 8))
        
        # Создание heatmap
        sns.heatmap(
            self.matrix,
            annot=True,
            fmt='d',
            cmap='RdYlGn_r',
            center=5,
            xticklabels=['Very Low', 'Low', 'Medium', 'High', 'Very High'],
            yticklabels=['Very Low', 'Low', 'Medium', 'High', 'Very High'],
            cbar_kws={'label': 'Risk Score'}
        )
        
        plt.title('Risk Matrix: Probability vs Impact')
        plt.xlabel('Impact Level')
        plt.ylabel('Probability Level')
        plt.tight_layout()
        plt.show()

    def assess_risk_scenario(self, name: str, probability: float, impact: int, 
                           description: str = "", controls: list = None) -> dict:
        """Оценить конкретный сценарий риска"""
        risk_score = self.calculate_risk_score(probability, impact)
        category = self.categorize_risk(risk_score)
        
        return {
            'name': name,
            'probability': probability,
            'impact': impact,
            'risk_score': risk_score,
            'category': category,
            'description': description,
            'controls': controls or [],
            'mitigation_priority': self._get_mitigation_priority(category)
        }

    def _get_mitigation_priority(self, category: str) -> str:
        """Определить приоритет устранения"""
        priority_map = {
            'Very High': 'Critical',
            'High': 'High',
            'Medium': 'Medium', 
            'Low': 'Low',
            'Very Low': 'Low'
        }
        return priority_map.get(category, 'Unknown')

# Пример использования матрицы рисков
def example_risk_matrix():
    matrix = RiskMatrix()
    
    # Оценка различных рисков CI/CD
    risks = [
        matrix.assess_risk_scenario(
            name="Production Deployment Failure",
            probability=0.3,
            impact=5,
            description="Deployment fails and causes system downtime",
            controls=["Automated rollback", "Health checks", "Canary deployment"]
        ),
        matrix.assess_risk_scenario(
            name="Security Vulnerability Introduction",
            probability=0.2,
            impact=5,
            description="Security vulnerability introduced through code change",
            controls=["Static code analysis", "Dependency scanning", "Security testing"]
        ),
        matrix.assess_risk_scenario(
            name="Data Breach During Deployment",
            probability=0.1,
            impact=5,
            description="Sensitive data exposed during deployment process",
            controls=["Encryption", "Access controls", "Audit logging"]
        ),
        matrix.assess_risk_scenario(
            name="Performance Degradation",
            probability=0.4,
            impact=3,
            description="Deployment causes performance degradation",
            controls=["Performance testing", "Load testing", "Monitoring"]
        )
    ]
    
    # Визуализация матрицы
    matrix.visualize_matrix()
    
    # Вывод результатов
    print("Risk Assessment Results:")
    print("=" * 50)
    for risk in risks:
        print(f"Risk: {risk['name']}")
        print(f"  Probability: {risk['probability']:.2f}")
        print(f"  Impact: {risk['impact']}")
        print(f"  Risk Score: {risk['risk_score']}")
        print(f"  Category: {risk['category']}")
        print(f"  Priority: {risk['mitigation_priority']}")
        print(f"  Controls: {', '.join(risk['controls'])}")
        print()
    
    return risks

if __name__ == "__main__":
    example_risk_matrix()
```

## Идентификация рисков

### Методы идентификации рисков

#### SWOT-анализ

```python
# swot_analysis.py
class SWOTAnalysis:
    def __init__(self, system_name: str):
        self.system_name = system_name
        self.swot = {
            'strengths': [],
            'weaknesses': [],
            'opportunities': [],
            'threats': []
        }
    
    def add_strength(self, strength: str, description: str = ""):
        self.swot['strengths'].append({'item': strength, 'description': description})
    
    def add_weakness(self, weakness: str, description: str = ""):
        self.swot['weaknesses'].append({'item': weakness, 'description': description})
    
    def add_opportunity(self, opportunity: str, description: str = ""):
        self.swot['opportunities'].append({'item': opportunity, 'description': description})
    
    def add_threat(self, threat: str, description: str = ""):
        self.swot['threats'].append({'item': threat, 'description': description})
    
    def analyze_ci_cd_context(self):
        """Анализ SWOT в контексте CI/CD"""
        self.add_strength(
            "Automated Testing",
            "Comprehensive automated test suite reduces human error"
        )
        self.add_strength(
            "Fast Feedback Loop",
            "Quick identification of issues in development cycle"
        )
        self.add_strength(
            "Infrastructure as Code",
            "Consistent and reproducible environments"
        )
        
        self.add_weakness(
            "Complexity",
            "CI/CD pipeline complexity can introduce hidden risks"
        )
        self.add_weakness(
            "Tool Dependency",
            "Over-reliance on CI/CD tools creates single points of failure"
        )
        self.add_weakness(
            "Skill Gap",
            "Team may lack expertise in CI/CD security practices"
        )
        
        self.add_opportunity(
            "Security Integration",
            "Opportunity to integrate security testing into pipeline"
        )
        self.add_opportunity(
            "Compliance Automation",
            "Automate compliance checks and reporting"
        )
        self.add_opportunity(
            "Observability Enhancement",
            "Improve system monitoring and alerting"
        )
        
        self.add_threat(
            "Supply Chain Attacks",
            "Third-party dependencies may contain malicious code"
        )
        self.add_threat(
            "Configuration Drift",
            "Environments may diverge from intended configuration"
        )
        self.add_threat(
            "Insider Threats",
            "Malicious insiders could compromise pipeline integrity"
        )
    
    def generate_risk_from_swot(self) -> list:
        """Генерация рисков на основе SWOT анализа"""
        risks = []
        
        # Риски из слабостей
        for weakness in self.swot['weaknesses']:
            risks.append({
                'type': 'internal',
                'source': 'weakness',
                'description': f"Exploitation of weakness: {weakness['item']}",
                'mitigation': f"Address weakness: {weakness['item']}",
                'category': 'operational'
            })
        
        # Риски из угроз
        for threat in self.swot['threats']:
            risks.append({
                'type': 'external',
                'source': 'threat',
                'description': f"Realization of threat: {threat['item']}",
                'mitigation': f"Mitigate threat: {threat['item']}",
                'category': 'security'
            })
        
        return risks

# Пример использования
def example_swot_analysis():
    swot = SWOTAnalysis("E-commerce Platform CI/CD")
    swot.analyze_ci_cd_context()
    
    print("SWOT Analysis for CI/CD System:")
    print("=" * 40)
    
    for category, items in swot.swot.items():
        print(f"\n{category.upper()}:")
        for item in items:
            print(f"  - {item['item']}: {item['description']}")
    
    # Генерация рисков
    risks = swot.generate_risk_from_swot()
    print(f"\nGenerated Risks ({len(risks)}):")
    for risk in risks:
        print(f"  - {risk['description']} (Category: {risk['category']})")
    
    return swot

if __name__ == "__main__":
    example_swot_analysis()
```

#### Мозговой штурм рисков

```python
# risk_brainstorming.py
class RiskBrainstorming:
    def __init__(self, session_topic: str):
        self.session_topic = session_topic
        self.participants = []
        self.risks = []
        self.categories = [
            'security', 'performance', 'availability', 'compliance', 
            'data', 'process', 'technology', 'human'
        ]
    
    def add_participant(self, name: str, role: str):
        self.participants.append({'name': name, 'role': role})
    
    def conduct_session(self, duration_minutes: int = 60):
        """Проведение сессии мозгового штурма"""
        print(f"Conducting risk brainstorming session: {self.session_topic}")
        print(f"Duration: {duration_minutes} minutes")
        print(f"Participants: {[p['name'] for p in self.participants]}")
        
        # Использование различных техник генерации идей
        techniques = [
            self._free_association_technique,
            self._role_playing_technique,
            self._what_if_technique,
            self._historical_analysis_technique
        ]
        
        for technique in techniques:
            technique()
    
    def _free_association_technique(self):
        """Техника свободной ассоциации"""
        print("\nFree Association Technique:")
        print("Identifying risks through free association...")
        
        # Использование категорий для структурированного подхода
        for category in self.categories:
            risks_in_category = self._generate_risks_by_category(category)
            for risk in risks_in_category:
                self.add_risk(risk['description'], category, risk['probability'], risk['impact'])
    
    def _role_playing_technique(self):
        """Техника ролевой игры"""
        print("\nRole Playing Technique:")
        print("Simulating different stakeholder perspectives...")
        
        roles = [
            'malicious_attacker', 'disgruntled_employee', 
            'overworked_developer', 'compliance_officer'
        ]
        
        for role in roles:
            role_risks = self._simulate_role_risks(role)
            for risk in role_risks:
                self.add_risk(risk['description'], risk['category'], 
                            risk['probability'], risk['impact'])
    
    def _what_if_technique(self):
        """Техника "что если" """
        print("\nWhat-If Technique:")
        print("Exploring hypothetical scenarios...")
        
        what_if_scenarios = [
            "What if the CI server is compromised?",
            "What if a dependency introduces a vulnerability?",
            "What if the deployment pipeline fails during peak hours?",
            "What if sensitive data is accidentally committed?",
            "What if there's a configuration drift in production?"
        ]
        
        for scenario in what_if_scenarios:
            risk = self._analyze_what_if_scenario(scenario)
            if risk:
                self.add_risk(risk['description'], risk['category'], 
                            risk['probability'], risk['impact'])
    
    def _historical_analysis_technique(self):
        """Техника исторического анализа"""
        print("\nHistorical Analysis Technique:")
        print("Analyzing past incidents and near-misses...")
        
        # Использование исторических данных
        historical_incidents = self._get_historical_incidents()
        for incident in historical_incidents:
            risk = self._derive_risk_from_incident(incident)
            if risk:
                self.add_risk(risk['description'], risk['category'], 
                            risk['probability'], risk['impact'])
    
    def _generate_risks_by_category(self, category: str) -> list:
        """Генерация рисков по категории"""
        risk_templates = {
            'security': [
                {'description': f'Security vulnerability in {category} components', 'probability': 0.3, 'impact': 8},
                {'description': f'Malicious code injection through {category}', 'probability': 0.1, 'impact': 9},
                {'description': f'Unauthorized access to {category} resources', 'probability': 0.2, 'impact': 7}
            ],
            'performance': [
                {'description': f'Performance degradation in {category} operations', 'probability': 0.4, 'impact': 6},
                {'description': f'Scaling issues in {category} infrastructure', 'probability': 0.3, 'impact': 7},
                {'description': f'Resource exhaustion in {category} systems', 'probability': 0.2, 'impact': 8}
            ],
            'availability': [
                {'description': f'Service disruption in {category} systems', 'probability': 0.2, 'impact': 9},
                {'description': f'Dependency failure affecting {category}', 'probability': 0.3, 'impact': 8},
                {'description': f'Infrastructure outage impacting {category}', 'probability': 0.1, 'impact': 10}
            ]
        }
        
        return risk_templates.get(category, [
            {'description': f'General risk in {category}', 'probability': 0.2, 'impact': 5}
        ])
    
    def _simulate_role_risks(self, role: str) -> list:
        """Симуляция рисков из перспективы роли"""
        role_risks = {
            'malicious_attacker': [
                {'description': 'Attack through CI/CD pipeline', 'category': 'security', 'probability': 0.1, 'impact': 10},
                {'description': 'Supply chain attack', 'category': 'security', 'probability': 0.05, 'impact': 9}
            ],
            'disgruntled_employee': [
                {'description': 'Sabotage of deployment pipeline', 'category': 'human', 'probability': 0.05, 'impact': 8},
                {'description': 'Data corruption by insider', 'category': 'data', 'probability': 0.03, 'impact': 7}
            ],
            'overworked_developer': [
                {'description': 'Human error in code deployment', 'category': 'human', 'probability': 0.3, 'impact': 6},
                {'description': 'Skipping security checks due to pressure', 'category': 'process', 'probability': 0.2, 'impact': 7}
            ],
            'compliance_officer': [
                {'description': 'Regulatory violation in deployment', 'category': 'compliance', 'probability': 0.1, 'impact': 9},
                {'description': 'Audit trail gaps', 'category': 'compliance', 'probability': 0.2, 'impact': 6}
            ]
        }
        
        return role_risks.get(role, [])
    
    def _analyze_what_if_scenario(self, scenario: str) -> dict:
        """Анализ сценария 'что если' """
        # Простая логика для демонстрации
        if 'compromised' in scenario.lower():
            return {
                'description': f'Security breach: {scenario}',
                'category': 'security',
                'probability': 0.1,
                'impact': 10
            }
        elif 'vulnerability' in scenario.lower():
            return {
                'description': f'Software vulnerability: {scenario}',
                'category': 'security',
                'probability': 0.3,
                'impact': 8
            }
        elif 'fails' in scenario.lower():
            return {
                'description': f'System failure: {scenario}',
                'category': 'availability',
                'probability': 0.2,
                'impact': 9
            }
        elif 'committed' in scenario.lower():
            return {
                'description': f'Data exposure: {scenario}',
                'category': 'data',
                'probability': 0.1,
                'impact': 8
            }
        
        return None
    
    def _get_historical_incidents(self) -> list:
        """Получение исторических инцидентов (симуляция)"""
        return [
            {'type': 'deployment_failure', 'severity': 'high', 'cause': 'configuration_error'},
            {'type': 'security_breach', 'severity': 'critical', 'cause': 'vulnerable_dependency'},
            {'type': 'performance_issue', 'severity': 'medium', 'cause': 'resource_leak'}
        ]
    
    def _derive_risk_from_incident(self, incident: dict) -> dict:
        """Выведение риска из инцидента"""
        impact_map = {'low': 3, 'medium': 5, 'high': 7, 'critical': 9}
        
        return {
            'description': f'Recurrence of {incident["type"]} due to {incident["cause"]}',
            'category': 'process' if 'configuration' in incident['cause'] else 'security',
            'probability': 0.4,  # Higher probability for known issues
            'impact': impact_map.get(incident['severity'], 5)
        }
    
    def add_risk(self, description: str, category: str, probability: float, impact: int):
        """Добавление риска"""
        risk = {
            'id': f"RISK_{len(self.risks) + 1:03d}",
            'description': description,
            'category': category,
            'probability': probability,
            'impact': impact,
            'status': 'identified',
            'created_at': '2023-11-11T10:00:00Z'
        }
        self.risks.append(risk)
    
    def get_risk_summary(self) -> dict:
        """Получение сводки по рискам"""
        categories = {}
        priorities = {'critical': 0, 'high': 0, 'medium': 0, 'low': 0}
        
        for risk in self.risks:
            # Категоризация по приоритету
            if risk['impact'] >= 8:
                priority = 'critical'
            elif risk['impact'] >= 6:
                priority = 'high'
            elif risk['impact'] >= 4:
                priority = 'medium'
            else:
                priority = 'low'
            
            priorities[priority] += 1
            
            # Категоризация по типу
            cat = risk['category']
            if cat not in categories:
                categories[cat] = {'count': 0, 'risks': []}
            categories[cat]['count'] += 1
            categories[cat]['risks'].append(risk)
        
        return {
            'total_risks': len(self.risks),
            'by_priority': priorities,
            'by_category': categories,
            'most_common_category': max(categories.keys(), key=lambda x: categories[x]['count']) if categories else None
        }

# Пример использования
def example_brainstorming():
    brainstorming = RiskBrainstorming("E-commerce Platform CI/CD Risk Identification")
    
    # Добавление участников
    brainstorming.add_participant("Alice", "Security Engineer")
    brainstorming.add_participant("Bob", "DevOps Engineer") 
    brainstorming.add_participant("Carol", "QA Manager")
    brainstorming.add_participant("David", "Compliance Officer")
    
    # Проведение сессии
    brainstorming.conduct_session(duration_minutes=90)
    
    # Получение результатов
    summary = brainstorming.get_risk_summary()
    
    print(f"\nRisk Brainstorming Results:")
    print(f"Total Identified Risks: {summary['total_risks']}")
    print(f"Most Common Category: {summary['most_common_category']}")
    print(f"Priority Distribution: {summary['by_priority']}")
    
    print(f"\nTop 5 Risks:")
    sorted_risks = sorted(brainstorming.risks, 
                         key=lambda x: x['impact'] * x['probability'], 
                         reverse=True)
    for i, risk in enumerate(sorted_risks[:5]):
        print(f"  {i+1}. {risk['description']} (Score: {risk['impact'] * risk['probability']:.2f})")
    
    return brainstorming

if __name__ == "__main__":
    example_brainstorming()
```

## Оценка рисков

### Количественная оценка

```python
# quantitative_risk_assessment.py
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt
from typing import Dict, List, Tuple
import warnings
warnings.filterwarnings('ignore')

class QuantitativeRiskAssessment:
    def __init__(self):
        self.risks = []
        self.monte_carlo_results = {}
    
    def add_risk(self, name: str, probability: float, impact_min: float, 
                 impact_max: float, impact_type: str = 'monetary') -> Dict:
        """
        Добавление риска с количественной оценкой
        
        Args:
            name: Название риска
            probability: Вероятность возникновения (0-1)
            impact_min: Минимальное воздействие
            impact_max: Максимальное воздействие  
            impact_type: Тип воздействия ('monetary', 'time', 'quality')
        """
        risk = {
            'name': name,
            'probability': probability,
            'impact_min': impact_min,
            'impact_max': impact_max,
            'impact_type': impact_type,
            'expected_impact': (impact_min + impact_max) / 2,
            'risk_score': probability * (impact_min + impact_max) / 2
        }
        
        self.risks.append(risk)
        return risk
    
    def monte_carlo_simulation(self, iterations: int = 10000) -> Dict:
        """
        Проведение симуляции Монте-Карло для оценки совокупного риска
        """
        results = {}
        
        for risk in self.risks:
            # Генерация сценариев для каждого риска
            scenarios = []
            
            for _ in range(iterations):
                # Вероятность реализации риска
                if np.random.random() < risk['probability']:
                    # Воздействие (равномерное распределение между min и max)
                    impact = np.random.uniform(risk['impact_min'], risk['impact_max'])
                    scenarios.append(impact)
                else:
                    scenarios.append(0)
            
            results[risk['name']] = np.array(scenarios)
        
        # Совокупный риск
        total_scenarios = np.zeros(iterations)
        for risk_name, scenarios in results.items():
            total_scenarios += scenarios
        
        results['total_portfolio_risk'] = total_scenarios
        
        # Статистика
        stats_results = {}
        for risk_name, scenarios in results.items():
            stats_results[risk_name] = {
                'mean': float(np.mean(scenarios)),
                'std': float(np.std(scenarios)),
                'min': float(np.min(scenarios)),
                'max': float(np.max(scenarios)),
                'var_95': float(np.percentile(scenarios, 5)),  # Value at Risk 95%
                'var_99': float(np.percentile(scenarios, 1)),  # Value at Risk 99%
                'expected_shortfall_95': float(np.mean(scenarios[scenarios <= np.percentile(scenarios, 5)])),
                'probability_exceeding_threshold': float(np.mean(scenarios > np.mean(scenarios) * 2))
            }
        
        self.monte_carlo_results = stats_results
        return stats_results
    
    def calculate_annual_loss_expectancy(self, risk: Dict, frequency_per_year: float) -> Dict:
        """
        Расчет Annual Loss Expectancy (ALE)
        ALE = SLE × ARO (Annual Rate of Occurrence)
        """
        single_loss_expectancy = risk['expected_impact']
        annual_rate_of_occurrence = frequency_per_year
        ale = single_loss_expectancy * annual_rate_of_occurrence
        
        return {
            'single_loss_expectancy': single_loss_expectancy,
            'annual_rate_of_occurrence': annual_rate_of_occurrence,
            'annual_loss_expectancy': ale,
            'risk_priority': ale  # Чем выше ALE, тем выше приоритет
        }
    
    def calculate_return_on_security_investment(self, 
                                              current_annual_loss: float,
                                              projected_annual_loss: float,
                                              security_investment: float) -> Dict:
        """
        Расчет Return on Security Investment (ROSI)
        ROSI = (Current ALE - Projected ALE) / Security Investment
        """
        risk_reduction = current_annual_loss - projected_annual_loss
        rosi = risk_reduction / security_investment if security_investment > 0 else 0
        
        return {
            'current_annual_loss': current_annual_loss,
            'projected_annual_loss': projected_annual_loss,
            'risk_reduction': risk_reduction,
            'security_investment': security_investment,
            'return_on_security_investment': rosi,
            'payback_period_years': security_investment / risk_reduction if risk_reduction > 0 else float('inf')
        }
    
    def prioritize_risks(self) -> List[Dict]:
        """
        Приоритезация рисков на основе различных метрик
        """
        prioritized = []
        
        for risk in self.risks:
            # Расчет различных показателей приоритетности
            priority_metrics = {
                'risk_score': risk['risk_score'],
                'ale_equivalent': risk['probability'] * risk['expected_impact'],
                'impact_weighted_probability': risk['probability'] * risk['expected_impact'] / (risk['impact_max'] or 1)
            }
            
            # Комбинированный приоритет (можно настроить веса)
            combined_priority = (
                priority_metrics['risk_score'] * 0.4 +
                priority_metrics['ale_equivalent'] * 0.4 +
                priority_metrics['impact_weighted_probability'] * 0.2
            )
            
            prioritized.append({
                **risk,
                'priority_metrics': priority_metrics,
                'combined_priority': combined_priority
            })
        
        # Сортировка по комбинированному приоритету
        prioritized.sort(key=lambda x: x['combined_priority'], reverse=True)
        return prioritized
    
    def visualize_risk_distribution(self):
        """
        Визуализация распределения рисков
        """
        if not self.monte_carlo_results:
            print("Run Monte Carlo simulation first to visualize risk distribution")
            return
        
        fig, axes = plt.subplots(2, 2, figsize=(15, 12))
        
        # 1. Гистограмма совокупного риска
        total_risk_scenarios = self.monte_carlo_results['total_portfolio_risk']
        axes[0, 0].hist(total_risk_scenarios, bins=50, alpha=0.7, edgecolor='black')
        axes[0, 0].set_title('Distribution of Total Portfolio Risk')
        axes[0, 0].set_xlabel('Risk Impact ($)')
        axes[0, 0].set_ylabel('Frequency')
        axes[0, 0].grid(True, alpha=0.3)
        
        # 2. Box plot рисков
        risk_names = [name for name in self.monte_carlo_results.keys() if name != 'total_portfolio_risk']
        risk_data = [self.monte_carlo_results[name] for name in risk_names]
        
        axes[0, 1].boxplot(risk_data, labels=risk_names, vert=True)
        axes[0, 1].set_title('Risk Distribution Comparison')
        axes[0, 1].tick_params(axis='x', rotation=45)
        
        # 3. Risk vs Probability scatter
        probabilities = [r['probability'] for r in self.risks]
        impacts = [r['expected_impact'] for r in self.risks]
        names = [r['name'] for r in self.risks]
        
        scatter = axes[1, 0].scatter(probabilities, impacts, 
                                   s=[r['risk_score']*100 for r in self.risks],
                                   alpha=0.6)
        axes[1, 0].set_xlabel('Probability')
        axes[1, 0].set_ylabel('Expected Impact')
        axes[1, 0].set_title('Risk Matrix: Probability vs Impact')
        axes[1, 0].grid(True, alpha=0.3)
        
        # 4. Cumulative risk distribution
        sorted_total = np.sort(total_risk_scenarios)
        cumulative_prob = np.arange(1, len(sorted_total) + 1) / len(sorted_total)
        
        axes[1, 1].plot(sorted_total, cumulative_prob)
        axes[1, 1].set_xlabel('Risk Impact ($)')
        axes[1, 1].set_ylabel('Cumulative Probability')
        axes[1, 1].set_title('Cumulative Risk Distribution')
        axes[1, 1].grid(True, alpha=0.3)
        
        plt.tight_layout()
        plt.show()
    
    def generate_risk_report(self) -> str:
        """
        Генерация текстового отчета по рискам
        """
        report = []
        report.append("# Risk Assessment Report\n")
        report.append(f"Generated on: {pd.Timestamp.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        
        # Сводная статистика
        report.append("## Executive Summary\n")
        report.append(f"- Total Risks Identified: {len(self.risks)}\n")
        report.append(f"- Total Expected Annual Loss: ${sum(r['risk_score'] for r in self.risks):,.2f}\n")
        
        if self.monte_carlo_results:
            total_stats = self.monte_carlo_results['total_portfolio_risk']
            report.append(f"- 95% VaR (Value at Risk): ${np.percentile(total_stats, 5):,.2f}\n")
            report.append(f"- Expected Total Risk: ${np.mean(total_stats):,.2f}\n")
            report.append(f"- Risk Standard Deviation: ${np.std(total_stats):,.2f}\n")
        
        report.append("\n## Individual Risk Assessments\n")
        
        prioritized = self.prioritize_risks()
        for i, risk in enumerate(prioritized, 1):
            report.append(f"### {i}. {risk['name']}\n")
            report.append(f"- **Probability**: {risk['probability']:.2%}\n")
            report.append(f"- **Impact Range**: ${risk['impact_min']:,.2f} - ${risk['impact_max']:,.2f}\n")
            report.append(f"- **Expected Impact**: ${risk['expected_impact']:,.2f}\n")
            report.append(f"- **Risk Score**: {risk['risk_score']:.2f}\n")
            report.append(f"- **Priority**: {risk['combined_priority']:.2f}\n")
            report.append(f"- **Type**: {risk['impact_type']}\n\n")
        
        if self.monte_carlo_results:
            report.append("## Monte Carlo Simulation Results\n")
            for risk_name, stats in self.monte_carlo_results.items():
                if risk_name != 'total_portfolio_risk':
                    report.append(f"### {risk_name}\n")
                    report.append(f"- Mean Impact: ${stats['mean']:,.2f}\n")
                    report.append(f"- Std Dev: ${stats['std']:,.2f}\n")
                    report.append(f"- 95% VaR: ${stats['var_95']:,.2f}\n")
                    report.append(f"- 99% VaR: ${stats['var_99']:,.2f}\n\n")
        
        return "".join(report)

# Пример использования количественной оценки рисков
def example_quantitative_assessment():
    assessor = QuantitativeRiskAssessment()
    
    # Добавление рисков CI/CD
    assessor.add_risk(
        name="Production Deployment Failure",
        probability=0.15,  # 15% chance per year
        impact_min=50000,  # $50K minimum loss
        impact_max=500000, # $500K maximum loss
        impact_type="monetary"
    )
    
    assessor.add_risk(
        name="Security Vulnerability Exploitation",
        probability=0.1,   # 10% chance per year
        impact_min=100000, # $100K minimum loss
        impact_max=1000000, # $1M maximum loss
        impact_type="monetary"
    )
    
    assessor.add_risk(
        name="Data Breach During Deployment",
        probability=0.05,  # 5% chance per year
        impact_min=200000, # $200K minimum loss
        impact_max=2000000, # $2M maximum loss
        impact_type="monetary"
    )
    
    assessor.add_risk(
        name="Performance Degradation",
        probability=0.25,  # 25% chance per year
        impact_min=10000,  # $10K minimum loss
        impact_max=100000, # $100K maximum loss
        impact_type="monetary"
    )
    
    assessor.add_risk(
        name="Compliance Violation",
        probability=0.08,  # 8% chance per year
        impact_min=150000, # $150K minimum loss
        impact_max=1500000, # $1.5M maximum loss
        impact_type="monetary"
    )
    
    # Проведение симуляции Монте-Карло
    monte_carlo_results = assessor.monte_carlo_simulation(iterations=5000)
    
    # Приоритезация рисков
    prioritized_risks = assessor.prioritize_risks()
    
    print("Quantitative Risk Assessment Results:")
    print("=" * 50)
    
    print("\nTop 5 Prioritized Risks:")
    for i, risk in enumerate(prioritized_risks[:5], 1):
        print(f"{i}. {risk['name']}: Priority Score = {risk['combined_priority']:.2f}")
        print(f"   Probability: {risk['probability']:.2%}, Expected Impact: ${risk['expected_impact']:,.2f}")
        print()
    
    # Расчет ALE для топ рисков
    print("Annual Loss Expectancy (ALE) Calculations:")
    print("-" * 45)
    for risk in prioritized_risks[:3]:
        ale_result = assessor.calculate_annual_loss_expectancy(risk, risk['probability'])
        print(f"{risk['name']}: ALE = ${ale_result['annual_loss_expectancy']:,.2f}")
    
    # Визуализация (закомментировано для предотвращения отображения при запуске)
    # assessor.visualize_risk_distribution()
    
    # Генерация отчета
    report = assessor.generate_risk_report()
    print(f"\nGenerated comprehensive risk report ({len(report)} characters)")
    
    return assessor

if __name__ == "__main__":
    example_quantitative_assessment()
```

### Качественная оценка

```python
# qualitative_risk_assessment.py
from enum import Enum
from dataclasses import dataclass
from typing import Dict, List, Optional
import pandas as pd

class ProbabilityLevel(Enum):
    VERY_LOW = (0.05, "Very Low", "VL")
    LOW = (0.15, "Low", "L") 
    MEDIUM = (0.3, "Medium", "M")
    HIGH = (0.6, "High", "H")
    VERY_HIGH = (0.9, "Very High", "VH")

class ImpactLevel(Enum):
    VERY_LOW = (1, "Very Low", "VL")
    LOW = (2, "Low", "L")
    MEDIUM = (3, "Medium", "M") 
    HIGH = (4, "High", "H")
    VERY_HIGH = (5, "Very High", "VH")

class RiskLevel(Enum):
    VERY_LOW = ("Very Low", "VL", "#90EE90")  # Light Green
    LOW = ("Low", "L", "#98FB98")             # Pale Green
    MEDIUM = ("Medium", "M", "#FFFF00")       # Yellow
    HIGH = ("High", "H", "#FFA500")           # Orange
    VERY_HIGH = ("Very High", "VH", "#FF0000") # Red

@dataclass
class QualitativeRisk:
    name: str
    description: str
    probability: ProbabilityLevel
    impact: ImpactLevel
    category: str
    owner: str
    detection_likelihood: ProbabilityLevel = ProbabilityLevel.MEDIUM
    controllability: ImpactLevel = ImpactLevel.MEDIUM
    risk_level: Optional[RiskLevel] = None

class QualitativeRiskAssessment:
    def __init__(self):
        self.risks = []
        self.assessment_matrix = self._create_assessment_matrix()
    
    def _create_assessment_matrix(self) -> Dict:
        """Создание матрицы оценки рисков (вероятность x воздействие)"""
        matrix = {}
        
        for prob in ProbabilityLevel:
            for imp in ImpactLevel:
                risk_score = prob.value[0] * imp.value[0] * 10  # Масштабирование
                
                if risk_score <= 2:
                    level = RiskLevel.VERY_LOW
                elif risk_score <= 4:
                    level = RiskLevel.LOW
                elif risk_score <= 8:
                    level = RiskLevel.MEDIUM
                elif risk_score <= 15:
                    level = RiskLevel.HIGH
                else:
                    level = RiskLevel.VERY_HIGH
                
                matrix[(prob, imp)] = {
                    'score': risk_score,
                    'level': level
                }
        
        return matrix
    
    def add_risk(self, name: str, description: str, probability: ProbabilityLevel, 
                 impact: ImpactLevel, category: str, owner: str,
                 detection_likelihood: ProbabilityLevel = ProbabilityLevel.MEDIUM,
                 controllability: ImpactLevel = ImpactLevel.MEDIUM) -> QualitativeRisk:
        """Добавление качественного риска"""
        risk = QualitativeRisk(
            name=name,
            description=description,
            probability=probability,
            impact=impact,
            category=category,
            owner=owner,
            detection_likelihood=detection_likelihood,
            controllability=controllability
        )
        
        # Определение уровня риска
        matrix_entry = self.assessment_matrix[(probability, impact)]
        risk.risk_level = matrix_entry['level']
        
        self.risks.append(risk)
        return risk
    
    def assess_detection_likelihood(self, risk: QualitativeRisk) -> str:
        """Оценка вероятности обнаружения риска"""
        detection_factors = {
            ProbabilityLevel.VERY_HIGH: "Excellent detection capabilities",
            ProbabilityLevel.HIGH: "Good detection capabilities", 
            ProbabilityLevel.MEDIUM: "Moderate detection capabilities",
            ProbabilityLevel.LOW: "Limited detection capabilities",
            ProbabilityLevel.VERY_LOW: "Poor detection capabilities"
        }
        
        return detection_factors[risk.detection_likelihood]
    
    def assess_controllability(self, risk: QualitativeRisk) -> str:
        """Оценка управляемости риска"""
        control_factors = {
            ImpactLevel.VERY_HIGH: "Highly controllable with existing measures",
            ImpactLevel.HIGH: "Well controllable with some effort",
            ImpactLevel.MEDIUM: "Moderately controllable",
            ImpactLevel.LOW: "Difficult to control",
            ImpactLevel.VERY_LOW: "Nearly uncontrollable"
        }
        
        return control_factors[risk.controllability]
    
    def calculate_risk_exposure(self, risk: QualitativeRisk) -> Dict:
        """Расчет экспозиции риска"""
        return {
            'base_risk': self.assessment_matrix[(risk.probability, risk.impact)]['score'],
            'detection_adjustment': risk.detection_likelihood.value[0] * 2,  # Влияние на обнаружение
            'control_adjustment': (5 - risk.controllability.value[0]) * 1.5,  # Влияние на управляемость
            'adjusted_exposure': (
                self.assessment_matrix[(risk.probability, risk.impact)]['score'] *
                (1 + risk.detection_likelihood.value[0] * 0.5) *
                (1 + (5 - risk.controllability.value[0]) * 0.3)
            )
        }
    
    def generate_risk_heatmap_data(self) -> pd.DataFrame:
        """Генерация данных для тепловой карты рисков"""
        data = []
        
        for risk in self.risks:
            data.append({
                'Risk': risk.name,
                'Category': risk.category,
                'Owner': risk.owner,
                'Probability': risk.probability.value[1],
                'Impact': risk.impact.value[1],
                'Risk_Level': risk.risk_level.value[1],
                'Color_Code': risk.risk_level.value[2],
                'Probability_Score': risk.probability.value[0],
                'Impact_Score': risk.impact.value[0],
                'Risk_Score': self.assessment_matrix[(risk.probability, risk.impact)]['score']
            })
        
        return pd.DataFrame(data)
    
    def categorize_risks(self) -> Dict[str, List[QualitativeRisk]]:
        """Категоризация рисков"""
        categories = {}
        
        for risk in self.risks:
            if risk.category not in categories:
                categories[risk.category] = []
            categories[risk.category].append(risk)
        
        return categories
    
    def prioritize_by_stakeholder_importance(self, stakeholder_weights: Dict[str, float]) -> List[QualitativeRisk]:
        """Приоритезация рисков по важности для заинтересованных сторон"""
        weighted_risks = []
        
        for risk in self.risks:
            stakeholder_importance = stakeholder_weights.get(risk.owner, 1.0)
            risk_score = self.assessment_matrix[(risk.probability, risk.impact)]['score']
            weighted_score = risk_score * stakeholder_importance
            
            weighted_risks.append({
                'risk': risk,
                'weighted_score': weighted_score,
                'stakeholder_weight': stakeholder_importance
            })
        
        # Сортировка по взвешенному баллу
        weighted_risks.sort(key=lambda x: x['weighted_score'], reverse=True)
        
        return [item['risk'] for item in weighted_risks]
    
    def generate_qualitative_report(self) -> str:
        """Генерация качественного отчета по рискам"""
        report = []
        report.append("# Qualitative Risk Assessment Report\n")
        
        # Сводка
        report.append("## Executive Summary\n")
        report.append(f"- Total Risks Assessed: {len(self.risks)}\n")
        
        # Распределение по уровням
        level_counts = {}
        for risk in self.risks:
            level_name = risk.risk_level.value[0]
            level_counts[level_name] = level_counts.get(level_name, 0) + 1
        
        report.append("- Risk Distribution:\n")
        for level, count in level_counts.items():
            report.append(f"  - {level}: {count} risks\n")
        
        # Распределение по категориям
        categories = self.categorize_risks()
        report.append("\n- Risk Categories:\n")
        for category, risks in categories.items():
            report.append(f"  - {category}: {len(risks)} risks\n")
        
        # Детали рисков
        report.append("\n## Detailed Risk Assessment\n")
        
        # Сортировка рисков по уровню
        sorted_risks = sorted(self.risks, 
                            key=lambda x: self.assessment_matrix[(x.probability, x.impact)]['score'], 
                            reverse=True)
        
        for i, risk in enumerate(sorted_risks, 1):
            report.append(f"### {i}. {risk.name}\n")
            report.append(f"**Category**: {risk.category}  \n")
            report.append(f"**Owner**: {risk.owner}  \n")
            report.append(f"**Description**: {risk.description}  \n")
            report.append(f"**Probability**: {risk.probability.value[1]} ({risk.probability.value[0]:.0%})  \n")
            report.append(f"**Impact**: {risk.impact.value[1]} ({risk.impact.value[0]})  \n")
            report.append(f"**Risk Level**: {risk.risk_level.value[0]}  \n")
            report.append(f"**Risk Score**: {self.assessment_matrix[(risk.probability, risk.impact)]['score']:.1f}  \n")
            
            # Дополнительная оценка
            exposure = self.calculate_risk_exposure(risk)
            report.append(f"**Adjusted Exposure**: {exposure['adjusted_exposure']:.2f}  \n")
            
            report.append(f"**Detection Likelihood**: {self.assess_detection_likelihood(risk)}  \n")
            report.append(f"**Controllability**: {self.assess_controllability(risk)}  \n")
            
            report.append("\n")
        
        # Рекомендации
        report.append("## Recommendations\n")
        high_risks = [r for r in self.risks if r.risk_level in [RiskLevel.HIGH, RiskLevel.VERY_HIGH]]
        if high_risks:
            report.append(f"### Immediate Actions Required ({len(high_risks)} high-priority risks)\n")
            for risk in high_risks:
                report.append(f"- **{risk.name}**: Implement immediate mitigation strategies\n")
        
        # Отчет по владельцам
        report.append("\n## Owner Accountability\n")
        owners = {}
        for risk in self.risks:
            if risk.owner not in owners:
                owners[risk.owner] = {'risks': [], 'total_score': 0}
            owners[risk.owner]['risks'].append(risk)
            owners[risk.owner]['total_score'] += self.assessment_matrix[(risk.probability, risk.impact)]['score']
        
        for owner, data in owners.items():
            report.append(f"- **{owner}**: {len(data['risks'])} risks, Total Score: {data['total_score']:.1f}\n")
        
        return "".join(report)

# Пример использования качественной оценки
def example_qualitative_assessment():
    assessor = QualitativeRiskAssessment()
    
    # Добавление рисков CI/CD
    assessor.add_risk(
        name="Production Deployment Failure",
        description="Deployment process fails causing system downtime",
        probability=ProbabilityLevel.MEDIUM,
        impact=ImpactLevel.VERY_HIGH,
        category="Operational",
        owner="DevOps Team",
        detection_likelihood=ProbabilityLevel.HIGH,
        controllability=ImpactLevel.MEDIUM
    )
    
    assessor.add_risk(
        name="Security Vulnerability Introduction",
        description="Security vulnerability introduced through code changes",
        probability=ProbabilityLevel.LOW,
        impact=ImpactLevel.VERY_HIGH,
        category="Security", 
        owner="Security Team",
        detection_likelihood=ProbabilityLevel.MEDIUM,
        controllability=ImpactLevel.HIGH
    )
    
    assessor.add_risk(
        name="Performance Degradation",
        description="Deployment causes system performance issues",
        probability=ProbabilityLevel.HIGH,
        impact=ImpactLevel.MEDIUM,
        category="Performance",
        owner="Platform Team",
        detection_likelihood=ProbabilityLevel.HIGH,
        controllability=ImpactLevel.HIGH
    )
    
    assessor.add_risk(
        name="Data Corruption During Migration",
        description="Database migration process corrupts critical data",
        probability=ProbabilityLevel.LOW,
        impact=ImpactLevel.VERY_HIGH,
        category="Data",
        owner="Database Team",
        detection_likelihood=ProbabilityLevel.LOW,
        controllability=ImpactLevel.LOW
    )
    
    assessor.add_risk(
        name="Compliance Violation",
        description="Deployment violates regulatory compliance requirements",
        probability=ProbabilityLevel.MEDIUM,
        impact=ImpactLevel.HIGH,
        category="Compliance",
        owner="Compliance Team",
        detection_likelihood=ProbabilityLevel.MEDIUM,
        controllability=ImpactLevel.MEDIUM
    )
    
    assessor.add_risk(
        name="Third-Party Dependency Risk",
        description="Critical third-party dependency becomes unavailable",
        probability=ProbabilityLevel.MEDIUM,
        impact=ImpactLevel.HIGH,
        category="Supply Chain",
        owner="Architecture Team",
        detection_likelihood=ProbabilityLevel.LOW,
        controllability=ImpactLevel.LOW
    )
    
    # Генерация отчета
    report = assessor.generate_qualitative_report()
    print("Qualitative Risk Assessment Report Generated")
    print(f"Report length: {len(report)} characters")
    
    # Анализ по категориям
    categories = assessor.categorize_risks()
    print(f"\nRisks by Category:")
    for category, risks in categories.items():
        print(f"  {category}: {len(risks)} risks")
        high_risks = [r for r in risks if r.risk_level in [RiskLevel.HIGH, RiskLevel.VERY_HIGH]]
        if high_risks:
            print(f"    High priority in {category}: {len(high_risks)}")
    
    # Приоритезация по заинтересованным сторонам
    stakeholder_weights = {
        "DevOps Team": 1.2,
        "Security Team": 1.5,
        "Platform Team": 1.0,
        "Database Team": 1.3,
        "Compliance Team": 1.4,
        "Architecture Team": 1.1
    }
    
    prioritized_by_stakeholders = assessor.prioritize_by_stakeholder_importance(stakeholder_weights)
    print(f"\nTop 3 Risks by Stakeholder Priority:")
    for i, risk in enumerate(prioritized_by_stakeholders[:3], 1):
        exposure = assessor.calculate_risk_exposure(risk)
        print(f"{i}. {risk.name} (Weighted Score: {exposure['adjusted_exposure']:.2f})")
    
    return assessor

if __name__ == "__main__":
    example_qualitative_assessment()
```

## Контроль рисков

### Стратегии контроля рисков

```python
# risk_control_strategies.py
from enum import Enum
from dataclasses import dataclass
from typing import List, Dict, Optional
import pandas as pd

class ControlType(Enum):
    PREVENTIVE = "Preventive"
    DETECTIVE = "Detective" 
    CORRECTIVE = "Corrective"
    COMPENSATING = "Compensating"

class ControlEffectiveness(Enum):
    VERY_HIGH = 5
    HIGH = 4
    MEDIUM = 3
    LOW = 2
    VERY_LOW = 1

@dataclass
class RiskControl:
    name: str
    description: str
    type: ControlType
    effectiveness: ControlEffectiveness
    cost: float  # Annual cost in USD
    implementation_effort: str  # Low, Medium, High
    responsible_team: str
    maturity_level: int  # 1-5 scale
    dependencies: List[str] = None

@dataclass
class RiskMitigationPlan:
    risk_name: str
    controls: List[RiskControl]
    residual_risk_level: str
    implementation_timeline: str
    budget_allocated: float
    success_metrics: List[str]

class RiskControlFramework:
    def __init__(self):
        self.controls = []
        self.mitigation_plans = []
        self.control_inventory = {}
    
    def add_control(self, control: RiskControl):
        """Добавить контроль риска"""
        self.controls.append(control)
        self.control_inventory[control.name] = control
    
    def create_mitigation_plan(self, risk_name: str, controls: List[RiskControl],
                             residual_risk_level: str, implementation_timeline: str,
                             budget_allocated: float, success_metrics: List[str] = None) -> RiskMitigationPlan:
        """Создать план смягчения риска"""
        plan = RiskMitigationPlan(
            risk_name=risk_name,
            controls=controls,
            residual_risk_level=residual_risk_level,
            implementation_timeline=implementation_timeline,
            budget_allocated=budget_allocated,
            success_metrics=success_metrics or []
        )
        
        self.mitigation_plans.append(plan)
        return plan
    
    def calculate_control_effectiveness(self, controls: List[RiskControl]) -> Dict:
        """Рассчитать эффективность набора контролов"""
        if not controls:
            return {
                'overall_effectiveness': ControlEffectiveness.VERY_LOW,
                'effectiveness_score': 0,
                'cost_efficiency': 0,
                'coverage_gaps': []
            }
        
        # Средняя эффективность
        avg_effectiveness = sum(c.effectiveness.value for c in controls) / len(controls)
        
        # Общая стоимость
        total_cost = sum(c.cost for c in controls)
        
        # Эффективность на доллар
        cost_efficiency = avg_effectiveness / total_cost if total_cost > 0 else 0
        
        # Покрытие типов контролов
        control_types = [c.type for c in controls]
        required_types = set(ControlType)
        coverage_gaps = [ct for ct in required_types if ct not in control_types]
        
        return {
            'overall_effectiveness': ControlEffectiveness(round(avg_effectiveness)),
            'effectiveness_score': avg_effectiveness,
            'total_cost': total_cost,
            'cost_efficiency': cost_efficiency,
            'coverage_gaps': coverage_gaps,
            'control_diversity': len(set(control_types))
        }
    
    def assess_residual_risk(self, inherent_risk_level: str, 
                           applied_controls: List[RiskControl]) -> str:
        """Оценить остаточный риск после применения контролов"""
        # Симуляция снижения риска
        effectiveness_map = {
            'Very High': 0.1,  # 90% reduction
            'High': 0.25,      # 75% reduction  
            'Medium': 0.5,     # 50% reduction
            'Low': 0.75,       # 25% reduction
            'Very Low': 0.9    # 10% reduction
        }
        
        # Базовое снижение
        reduction_factor = 1.0
        for control in applied_controls:
            reduction = effectiveness_map.get(
                control.effectiveness.name.replace('_', ' ').title(), 0.5
            )
            reduction_factor *= reduction
        
        # Карта уровней риска
        risk_levels = {
            'Very Low': 1,
            'Low': 2, 
            'Medium': 3,
            'High': 4,
            'Very High': 5
        }
        
        # Рассчитать остаточный уровень
        inherent_score = risk_levels.get(inherent_risk_level, 3)
        residual_score = inherent_score * reduction_factor
        
        # Определить уровень остаточного риска
        if residual_score <= 1.5:
            return 'Very Low'
        elif residual_score <= 2.5:
            return 'Low'
        elif residual_score <= 3.5:
            return 'Medium'
        elif residual_score <= 4.5:
            return 'High'
        else:
            return 'Very High'
    
    def optimize_control_portfolio(self, budget: float, 
                                 target_risk_reduction: float = 0.7) -> List[RiskControl]:
        """Оптимизировать портфель контролов в рамках бюджета"""
        # Сортировка контролов по соотношению эффективность/стоимость
        controls_with_ratio = []
        
        for control in self.controls:
            if control.cost > 0:
                efficiency_ratio = control.effectiveness.value / control.cost
                controls_with_ratio.append((control, efficiency_ratio))
        
        # Сортировка по эффективности на доллар
        controls_with_ratio.sort(key=lambda x: x[1], reverse=True)
        
        selected_controls = []
        total_cost = 0
        
        for control, efficiency_ratio in controls_with_ratio:
            if total_cost + control.cost <= budget:
                selected_controls.append(control)
                total_cost += control.cost
        
        return selected_controls
    
    def generate_control_report(self) -> str:
        """Генерация отчета по контролам"""
        report = []
        report.append("# Risk Control Report\n")
        
        # Сводка по контролам
        report.append("## Control Inventory Summary\n")
        report.append(f"- Total Controls: {len(self.controls)}\n")
        
        # Распределение по типам
        type_counts = {}
        for control in self.controls:
            type_name = control.type.value
            type_counts[type_name] = type_counts.get(type_name, 0) + 1
        
        report.append("- Controls by Type:\n")
        for control_type, count in type_counts.items():
            report.append(f"  - {control_type}: {count}\n")
        
        # Распределение по эффективности
        effectiveness_counts = {}
        for control in self.controls:
            eff_name = control.effectiveness.name.replace('_', ' ').title()
            effectiveness_counts[eff_name] = effectiveness_counts.get(eff_name, 0) + 1
        
        report.append("- Controls by Effectiveness:\n")
        for effectiveness, count in effectiveness_counts.items():
            report.append(f"  - {effectiveness}: {count}\n")
        
        # Стоимость
        total_cost = sum(c.cost for c in self.controls)
        avg_cost = total_cost / len(self.controls) if self.controls else 0
        report.append(f"- Total Annual Control Cost: ${total_cost:,.2f}\n")
        report.append(f"- Average Control Cost: ${avg_cost:,.2f}\n")
        
        # Подробности контролов
        report.append("\n## Control Details\n")
        
        for i, control in enumerate(self.controls, 1):
            report.append(f"### {i}. {control.name}\n")
            report.append(f"**Type**: {control.type.value}  \n")
            report.append(f"**Effectiveness**: {control.effectiveness.name.replace('_', ' ').title()}  \n")
            report.append(f"**Cost**: ${control.cost:,.2f}/year  \n")
            report.append(f"**Effort**: {control.implementation_effort}  \n")
            report.append(f"**Responsible**: {control.responsible_team}  \n")
            report.append(f"**Maturity**: {control.maturity_level}/5  \n")
            report.append(f"**Description**: {control.description}  \n")
            
            if control.dependencies:
                report.append(f"**Dependencies**: {', '.join(control.dependencies)}  \n")
            
            report.append("\n")
        
        # Планы смягчения
        report.append("## Mitigation Plans\n")
        for i, plan in enumerate(self.mitigation_plans, 1):
            report.append(f"### {i}. {plan.risk_name}\n")
            report.append(f"**Timeline**: {plan.implementation_timeline}  \n")
            report.append(f"**Budget**: ${plan.budget_allocated:,.2f}  \n")
            report.append(f"**Residual Risk**: {plan.residual_risk_level}  \n")
            
            if plan.success_metrics:
                report.append("**Success Metrics**:  \n")
                for metric in plan.success_metrics:
                    report.append(f"  - {metric}  \n")
            
            report.append("**Controls Applied**:  \n")
            for control in plan.controls:
                report.append(f"  - {control.name} ({control.type.value})  \n")
            
            report.append("\n")
        
        return "".join(report)
    
    def calculate_roi_on_controls(self) -> Dict:
        """Рассчитать ROI на контролы рисков"""
        if not self.controls:
            return {}
        
        total_control_cost = sum(c.cost for c in self.controls)
        
        # Симуляция предотвращенных потерь
        prevented_losses = 0
        for control in self.controls:
            # Оценка предотвращенных потерь на основе эффективности
            potential_loss_prevention = control.effectiveness.value * 10000  # Условная величина
            prevented_losses += potential_loss_prevention
        
        roi = (prevented_losses - total_control_cost) / total_control_cost if total_control_cost > 0 else 0
        
        return {
            'total_control_cost': total_control_cost,
            'estimated_prevented_losses': prevented_losses,
            'return_on_investment': roi,
            'benefit_cost_ratio': prevented_losses / total_control_cost if total_control_cost > 0 else 0
        }

# Пример использования системы контроля рисков
def example_risk_control():
    framework = RiskControlFramework()
    
    # Добавление контролов для рисков CI/CD
    framework.add_control(RiskControl(
        name="Automated Security Scanning",
        description="Scan code and dependencies for vulnerabilities",
        type=ControlType.PREVENTIVE,
        effectiveness=ControlEffectiveness.HIGH,
        cost=50000,
        implementation_effort="Medium",
        responsible_team="Security Team",
        maturity_level=4,
        dependencies=["CI/CD Pipeline", "Security Tools"]
    ))
    
    framework.add_control(RiskControl(
        name="Peer Code Reviews",
        description="Mandatory code reviews before merging",
        type=ControlType.PREVENTIVE,
        effectiveness=ControlEffectiveness.MEDIUM,
        cost=20000,
        implementation_effort="Low", 
        responsible_team="Engineering Teams",
        maturity_level=5
    ))
    
    framework.add_control(RiskControl(
        name="Automated Testing Suite",
        description="Comprehensive automated testing including unit, integration, and e2e tests",
        type=ControlType.PREVENTIVE,
        effectiveness=ControlEffectiveness.HIGH,
        cost=75000,
        implementation_effort="High",
        responsible_team="QA Team",
        maturity_level=4
    ))
    
    framework.add_control(RiskControl(
        name="Real-time Monitoring",
        description="Continuous monitoring of application and infrastructure metrics",
        type=ControlType.DETECTIVE,
        effectiveness=ControlEffectiveness.HIGH,
        cost=30000,
        implementation_effort="Medium",
        responsible_team="Platform Team",
        maturity_level=3
    ))
    
    framework.add_control(RiskControl(
        name="Incident Response Plan",
        description="Documented procedures for responding to security incidents",
        type=ControlType.CORRECTIVE,
        effectiveness=ControlEffectiveness.MEDIUM,
        cost=15000,
        implementation_effort="Low",
        responsible_team="Security Team",
        maturity_level=2
    ))
    
    framework.add_control(RiskControl(
        name="Canary Deployments",
        description="Gradual rollout of changes to limited user base",
        type=ControlType.PREVENTIVE,
        effectiveness=ControlEffectiveness.HIGH,
        cost=40000,
        implementation_effort="High",
        responsible_team="DevOps Team",
        maturity_level=3
    ))
    
    framework.add_control(RiskControl(
        name="Backup and Recovery",
        description="Regular backups with tested recovery procedures",
        type=ControlType.CORRECTIVE,
        effectiveness=ControlEffectiveness.VERY_HIGH,
        cost=25000,
        implementation_effort="Medium",
        responsible_team="Infrastructure Team",
        maturity_level=4
    ))
    
    framework.add_control(RiskControl(
        name="Access Controls",
        description="Role-based access control for CI/CD systems",
        type=ControlType.PREVENTIVE,
        effectiveness=ControlEffectiveness.HIGH,
        cost=35000,
        implementation_effort="Medium",
        responsible_team="Security Team",
        maturity_level=5
    ))
    
    # Создание планов смягчения
    critical_risk_controls = [c for c in framework.controls 
                            if c.name in ["Automated Security Scanning", "Automated Testing Suite", 
                                        "Canary Deployments", "Access Controls"]]
    
    framework.create_mitigation_plan(
        risk_name="Production Deployment Failure",
        controls=critical_risk_controls,
        residual_risk_level="Low",
        implementation_timeline="3 months",
        budget_allocated=200000,
        success_metrics=[
            "Reduce deployment failures by 80%",
            "Achieve 99.9% deployment success rate",
            "Reduce mean time to recovery to < 15 minutes"
        ]
    )
    
    security_risk_controls = [c for c in framework.controls 
                            if c.name in ["Automated Security Scanning", "Peer Code Reviews", 
                                        "Access Controls", "Incident Response Plan"]]
    
    framework.create_mitigation_plan(
        risk_name="Security Vulnerability Introduction",
        controls=security_risk_controls,
        residual_risk_level="Medium",
        implementation_timeline="6 months", 
        budget_allocated=120000,
        success_metrics=[
            "Zero critical vulnerabilities in production",
            "Reduce security incidents by 70%",
            "Achieve 100% dependency scanning coverage"
        ]
    )
    
    # Оценка эффективности контролов
    effectiveness = framework.calculate_control_effectiveness(framework.controls)
    print(f"Overall Control Effectiveness: {effectiveness['overall_effectiveness'].name}")
    print(f"Cost Efficiency: {effectiveness['cost_efficiency']:.4f}")
    print(f"Coverage Gaps: {effectiveness['coverage_gaps']}")
    
    # Оценка остаточного риска
    residual_risk = framework.assess_residual_risk("Very High", critical_risk_controls)
    print(f"Residual Risk after controls: {residual_risk}")
    
    # Оптимизация в рамках бюджета
    optimized_controls = framework.optimize_control_portfolio(budget=150000)
    print(f"Optimized control portfolio ({len(optimized_controls)} controls) within $150K budget")
    
    # Расчет ROI
    roi_analysis = framework.calculate_roi_on_controls()
    print(f"ROI on Risk Controls: {roi_analysis['return_on_investment']:.2%}")
    print(f"Benefit/Cost Ratio: {roi_analysis['benefit_cost_ratio']:.2f}")
    
    # Генерация отчета
    report = framework.generate_control_report()
    print(f"\nGenerated comprehensive control report ({len(report)} characters)")
    
    return framework

if __name__ == "__main__":
    example_risk_control()
```

### Мониторинг и отчетность по рискам

```python
# risk_monitoring_reporting.py
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from typing import Dict, List, Optional
import json
import matplotlib.pyplot as plt
import seaborn as sns

class RiskMonitoringDashboard:
    def __init__(self):
        self.risk_register = []
        self.monitoring_metrics = []
        self.incident_log = []
        self.compliance_tracker = []
    
    def add_risk_to_register(self, risk_data: Dict):
        """Добавить риск в реестр"""
        risk_entry = {
            'id': f"RISK-{len(self.risk_register) + 1:04d}",
            'added_date': datetime.now().isoformat(),
            'last_updated': datetime.now().isoformat(),
            'status': 'Active',
            **risk_data
        }
        self.risk_register.append(risk_entry)
    
    def log_monitoring_metric(self, metric_name: str, value: float, 
                            risk_id: str, timestamp: datetime = None):
        """Зарегистрировать метрику мониторинга рисков"""
        if timestamp is None:
            timestamp = datetime.now()
        
        metric_entry = {
            'timestamp': timestamp.isoformat(),
            'metric_name': metric_name,
            'value': value,
            'risk_id': risk_id,
            'status': 'Normal' if self._is_value_normal(metric_name, value) else 'Alert'
        }
        
        self.monitoring_metrics.append(metric_entry)
    
    def _is_value_normal(self, metric_name: str, value: float) -> bool:
        """Проверить, является ли значение метрики нормальным"""
        # Пороговые значения для различных метрик
        thresholds = {
            'deployment_failure_rate': 0.05,  # 5% порог
            'security_vulnerability_count': 5,  # 5 уязвимостей порог
            'performance_degradation_percent': 10,  # 10% порог
            'compliance_violation_count': 0,  # 0 нарушений порог
            'error_rate': 0.01,  # 1% порог
            'response_time_ms': 2000  # 2000ms порог
        }
        
        threshold = thresholds.get(metric_name, float('inf'))
        return value <= threshold
    
    def log_incident(self, incident_data: Dict):
        """Зарегистрировать инцидент риска"""
        incident_entry = {
            'id': f"INC-{len(self.incident_log) + 1:04d}",
            'occurred_at': datetime.now().isoformat(),
            'resolved_at': None,
            'severity': 'Medium',
            'status': 'Open',
            **incident_data
        }
        self.incident_log.append(incident_entry)
    
    def track_compliance(self, requirement: str, status: str, evidence: str = ""):
        """Отслеживать соответствие требованиям"""
        compliance_entry = {
            'requirement': requirement,
            'status': status,
            'checked_at': datetime.now().isoformat(),
            'evidence': evidence,
            'next_review_date': (datetime.now() + timedelta(days=30)).isoformat()
        }
        self.compliance_tracker.append(compliance_entry)
    
    def calculate_risk_trends(self, days_back: int = 30) -> Dict:
        """Рассчитать тренды рисков за указанный период"""
        start_date = datetime.now() - timedelta(days=days_back)
        
        # Агрегация метрик за период
        period_metrics = [
            m for m in self.monitoring_metrics 
            if datetime.fromisoformat(m['timestamp']) >= start_date
        ]
        
        # Группировка по метрикам
        trends = {}
        for metric in period_metrics:
            metric_name = metric['metric_name']
            if metric_name not in trends:
                trends[metric_name] = {
                    'values': [],
                    'timestamps': [],
                    'alerts': 0,
                    'average': 0
                }
            
            trends[metric_name]['values'].append(metric['value'])
            trends[metric_name]['timestamps'].append(datetime.fromisoformat(metric['timestamp']))
            if metric['status'] == 'Alert':
                trends[metric_name]['alerts'] += 1
        
        # Расчет статистики
        for metric_name, data in trends.items():
            if data['values']:
                data['average'] = sum(data['values']) / len(data['values'])
                
                # Тренд (линейная регрессия)
                if len(data['values']) > 1:
                    x = range(len(data['values']))
                    coefficients = np.polyfit(x, data['values'], 1)
                    data['trend_slope'] = float(coefficients[0])
                    data['trend_direction'] = 'increasing' if coefficients[0] > 0 else 'decreasing'
                else:
                    data['trend_slope'] = 0
                    data['trend_direction'] = 'stable'
        
        return trends
    
    def generate_risk_dashboard_data(self) -> Dict:
        """Сгенерировать данные для дашборда рисков"""
        current_date = datetime.now()
        
        # Сводка по рискам
        risk_summary = {
            'total_active_risks': len([r for r in self.risk_register if r['status'] == 'Active']),
            'high_priority_risks': len([r for r in self.risk_register if r.get('priority') == 'High']),
            'monitored_metrics': len(set(m['metric_name'] for m in self.monitoring_metrics)),
            'open_incidents': len([i for i in self.incident_log if i['status'] == 'Open']),
            'compliance_items': len(self.compliance_tracker),
            'compliance_status': self._calculate_compliance_status()
        }
        
        # Тренды рисков
        risk_trends = self.calculate_risk_trends(days_back=7)
        
        # Метрики производительности
        performance_metrics = self._calculate_performance_metrics()
        
        # Критические риски
        critical_risks = [
            r for r in self.risk_register 
            if r.get('priority') in ['Critical', 'High'] and r['status'] == 'Active'
        ]
        
        # Активные инциденты
        active_incidents = [
            i for i in self.incident_log 
            if i['status'] in ['Open', 'In Progress']
        ]
        
        return {
            'summary': risk_summary,
            'trends': risk_trends,
            'performance': performance_metrics,
            'critical_risks': critical_risks,
            'active_incidents': active_incidents,
            'generated_at': current_date.isoformat()
        }
    
    def _calculate_compliance_status(self) -> Dict:
        """Рассчитать статус соответствия"""
        if not self.compliance_tracker:
            return {'overall_status': 'No Data', 'compliant_items': 0, 'total_items': 0}
        
        compliant_count = len([c for c in self.compliance_tracker if c['status'] == 'Compliant'])
        total_count = len(self.compliance_tracker)
        
        overall_status = 'Compliant' if compliant_count == total_count else 'Partial'
        if compliant_count == 0:
            overall_status = 'Non-Compliant'
        
        return {
            'overall_status': overall_status,
            'compliant_items': compliant_count,
            'total_items': total_count,
            'compliance_percentage': (compliant_count / total_count) * 100 if total_count > 0 else 0
        }
    
    def _calculate_performance_metrics(self) -> Dict:
        """Рассчитать метрики производительности"""
        recent_metrics = [
            m for m in self.monitoring_metrics 
            if datetime.fromisoformat(m['timestamp']) >= datetime.now() - timedelta(days=7)
        ]
        
        if not recent_metrics:
            return {}
        
        # Агрегация по типам метрик
        metrics_summary = {}
        for metric in recent_metrics:
            metric_name = metric['metric_name']
            if metric_name not in metrics_summary:
                metrics_summary[metric_name] = {
                    'count': 0,
                    'sum': 0,
                    'max': float('-inf'),
                    'min': float('inf'),
                    'alerts': 0
                }
            
            metrics_summary[metric_name]['count'] += 1
            metrics_summary[metric_name]['sum'] += metric['value']
            metrics_summary[metric_name]['max'] = max(metrics_summary[metric_name]['max'], metric['value'])
            metrics_summary[metric_name]['min'] = min(metrics_summary[metric_name]['min'], metric['value'])
            
            if metric['status'] == 'Alert':
                metrics_summary[metric_name]['alerts'] += 1
        
        # Расчет средних значений
        for metric_name, data in metrics_summary.items():
            if data['count'] > 0:
                data['average'] = data['sum'] / data['count']
        
        return metrics_summary
    
    def generate_risk_report(self) -> str:
        """Сгенерировать текстовый отчет по рискам"""
        dashboard_data = self.generate_risk_dashboard_data()
        
        report = []
        report.append("# Risk Monitoring and Reporting Dashboard\n")
        report.append(f"Generated on: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        
        # Сводка
        report.append("## Executive Summary\n")
        summary = dashboard_data['summary']
        report.append(f"- Total Active Risks: {summary['total_active_risks']}\n")
        report.append(f"- High Priority Risks: {summary['high_priority_risks']}\n")
        report.append(f"- Open Incidents: {summary['open_incidents']}\n")
        report.append(f"- Compliance Status: {summary['compliance_status']['overall_status']} "
                     f"({summary['compliance_status']['compliance_percentage']:.1f}% compliant)\n\n")
        
        # Критические риски
        if dashboard_data['critical_risks']:
            report.append("## Critical Risks\n")
            for risk in dashboard_data['critical_risks'][:5]:  # Топ 5
                report.append(f"- **{risk['name']}**: {risk.get('description', 'No description')}\n")
            report.append("\n")
        
        # Активные инциденты
        if dashboard_data['active_incidents']:
            report.append("## Active Incidents\n")
            for incident in dashboard_data['active_incidents'][:5]:  # Топ 5
                report.append(f"- **{incident['type']}**: {incident.get('description', 'No description')} "
                            f"(Severity: {incident['severity']})\n")
            report.append("\n")
        
        # Метрики мониторинга
        report.append("## Monitoring Metrics\n")
        for metric_name, data in dashboard_data['performance'].items():
            report.append(f"### {metric_name}\n")
            report.append(f"- Average: {data.get('average', 0):.2f}\n")
            report.append(f"- Maximum: {data.get('max', 0):.2f}\n")
            report.append(f"- Minimum: {data.get('min', 0):.2f}\n")
            report.append(f"- Alerts: {data.get('alerts', 0)}\n")
            report.append(f"- Total Measurements: {data.get('count', 0)}\n\n")
        
        # Тренды
        report.append("## Risk Trends (Last 7 Days)\n")
        for metric_name, trend_data in dashboard_data['trends'].items():
            if 'trend_direction' in trend_data:
                report.append(f"- **{metric_name}**: {trend_data['trend_direction']} "
                            f"(slope: {trend_data['trend_slope']:.4f})\n")
        report.append("\n")
        
        # Рекомендации
        report.append("## Recommendations\n")
        if summary['high_priority_risks'] > 0:
            report.append(f"- **Immediate Action Required**: Address {summary['high_priority_risks']} high-priority risks\n")
        if summary['open_incidents'] > 0:
            report.append(f"- **Incident Resolution**: Resolve {summary['open_incidents']} open incidents\n")
        if summary['compliance_status']['compliance_percentage'] < 100:
            report.append("- **Compliance Improvement**: Address compliance gaps\n")
        
        return "".join(report)
    
    def visualize_risk_dashboard(self):
        """Визуализировать дашборд рисков"""
        dashboard_data = self.generate_risk_dashboard_data()
        
        fig, axes = plt.subplots(2, 2, figsize=(15, 12))
        
        # 1. Распределение рисков по приоритету
        priority_counts = {}
        for risk in self.risk_register:
            priority = risk.get('priority', 'Medium')
            priority_counts[priority] = priority_counts.get(priority, 0) + 1
        
        if priority_counts:
            axes[0, 0].pie(priority_counts.values(), labels=priority_counts.keys(), autopct='%1.1f%%')
            axes[0, 0].set_title('Risk Distribution by Priority')
        
        # 2. Количество инцидентов по типам
        incident_types = {}
        for incident in self.incident_log:
            incident_type = incident.get('type', 'Unknown')
            incident_types[incident_type] = incident_types.get(incident_type, 0) + 1
        
        if incident_types:
            axes[0, 1].bar(incident_types.keys(), incident_types.values())
            axes[0, 1].set_title('Incidents by Type')
            axes[0, 1].tick_params(axis='x', rotation=45)
        
        # 3. Тренды метрик
        if dashboard_data['trends']:
            trends = dashboard_data['trends']
            for i, (metric_name, trend_data) in enumerate(list(trends.items())[:2]):  # Показать первые 2 метрики
                if 'values' in trend_data and len(trend_data['values']) > 1:
                    ax_idx = 1 if i == 0 else 1  # Используем ось [1,0] и [1,1]
                    col = i % 2
                    axes[1, col].plot(trend_data['values'], marker='o', label=metric_name)
                    axes[1, col].set_title(f'{metric_name} Trend')
                    axes[1, col].set_xlabel('Time Points')
                    axes[1, col].set_ylabel('Value')
                    axes[1, col].grid(True, alpha=0.3)
        
        # 4. Статус соответствия
        comp_status = dashboard_data['summary']['compliance_status']
        if comp_status['total_items'] > 0:
            compliant = comp_status['compliant_items']
            non_compliant = comp_status['total_items'] - compliant
            axes[1, 1].bar(['Compliant', 'Non-Compliant'], [compliant, non_compliant])
            axes[1, 1].set_title('Compliance Status')
            axes[1, 1].set_ylabel('Number of Items')
        
        plt.tight_layout()
        plt.show()

# Пример использования системы мониторинга рисков
def example_risk_monitoring():
    dashboard = RiskMonitoringDashboard()
    
    # Добавление рисков в реестр
    dashboard.add_risk_to_register({
        'name': 'Production Deployment Failure',
        'description': 'Risk of deployment causing system downtime',
        'category': 'Operational',
        'priority': 'High',
        'probability': 0.15,
        'impact': 8
    })
    
    dashboard.add_risk_to_register({
        'name': 'Security Vulnerability Introduction',
        'description': 'Risk of introducing security vulnerabilities',
        'category': 'Security',
        'priority': 'Critical',
        'probability': 0.1,
        'impact': 9
    })
    
    dashboard.add_risk_to_register({
        'name': 'Performance Degradation',
        'description': 'Risk of deployment causing performance issues',
        'category': 'Performance',
        'priority': 'Medium',
        'probability': 0.25,
        'impact': 6
    })
    
    # Логирование метрик мониторинга
    for i in range(10):
        timestamp = datetime.now() - timedelta(hours=i*2)
        dashboard.log_monitoring_metric(
            'deployment_failure_rate', 
            np.random.uniform(0.01, 0.08), 
            'RISK-0001', 
            timestamp
        )
        dashboard.log_monitoring_metric(
            'security_vulnerability_count',
            np.random.randint(0, 10),
            'RISK-0002',
            timestamp
        )
        dashboard.log_monitoring_metric(
            'performance_degradation_percent',
            np.random.uniform(2, 15),
            'RISK-0003',
            timestamp
        )
    
    # Логирование инцидентов
    dashboard.log_incident({
        'type': 'Deployment Failure',
        'description': 'Production deployment failed due to configuration error',
        'severity': 'High',
        'risk_id': 'RISK-0001'
    })
    
    dashboard.log_incident({
        'type': 'Security Vulnerability',
        'description': 'Critical vulnerability found in dependency',
        'severity': 'Critical', 
        'risk_id': 'RISK-0002'
    })
    
    # Отслеживание соответствия
    dashboard.track_compliance('PCI DSS Requirement 6.2', 'Compliant', 'Security scan passed')
    dashboard.track_compliance('SOX Section 404', 'Non-Compliant', 'Documentation gap identified')
    dashboard.track_compliance('GDPR Article 32', 'Compliant', 'Technical measures implemented')
    
    # Генерация данных дашборда
    dashboard_data = dashboard.generate_risk_dashboard_data()
    
    print("Risk Monitoring Dashboard Data:")
    print(f"Summary: {dashboard_data['summary']}")
    print(f"Number of critical risks: {len(dashboard_data['critical_risks'])}")
    print(f"Active incidents: {len(dashboard_data['active_incidents'])}")
    
    # Генерация отчета
    report = dashboard.generate_risk_report()
    print(f"\nGenerated risk report ({len(report)} characters)")
    
    # Визуализация (закомментировано для предотвращения отображения при запуске)
    # dashboard.visualize_risk_dashboard()
    
    return dashboard

if __name__ == "__main__":
    example_risk_monitoring()
```

## Культура управления рисками

### Создание культуры безопасности

```python
# risk_culture_framework.py
from enum import Enum
from dataclasses import dataclass
from typing import List, Dict, Optional
import pandas as pd

class RiskCultureDimension(Enum):
    LEADERSHIP_COMMITMENT = "Leadership Commitment"
    RISK_AWARENESS = "Risk Awareness"
    COMMUNICATION = "Communication"
    ACCOUNTABILITY = "Accountability"
    CONTINUOUS_IMPROVEMENT = "Continuous Improvement"
    INCIDENT_LEARNING = "Incident Learning"

class CultureMaturityLevel(Enum):
    INITIAL = (1, "Initial", "Ad-hoc, inconsistent practices")
    MANAGED = (2, "Managed", "Basic processes in place")
    DEFINED = (3, "Defined", "Standardized practices")
    QUANTITATIVELY_MANAGED = (4, "Quantitatively Managed", "Measurable objectives")
    OPTIMIZING = (5, "Optimizing", "Continuously improving")

@dataclass
class CultureAssessmentQuestion:
    dimension: RiskCultureDimension
    question: str
    measurement_scale: str  # "likert", "yes_no", "frequency"
    weight: float = 1.0

@dataclass
class CultureAssessmentResult:
    dimension: RiskCultureDimension
    score: float  # 1-5 scale
    maturity_level: CultureMaturityLevel
    improvement_areas: List[str]
    recommendations: List[str]

class RiskCultureFramework:
    def __init__(self):
        self.questions = self._initialize_assessment_questions()
        self.assessment_results = []
        self.culture_initiatives = []
        self.training_programs = []
    
    def _initialize_assessment_questions(self) -> List[CultureAssessmentQuestion]:
        """Инициализация вопросов для оценки культуры рисков"""
        questions = [
            # Leadership Commitment
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.LEADERSHIP_COMMITMENT,
                question="Leaders actively participate in risk management activities",
                measurement_scale="likert",
                weight=1.2
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.LEADERSHIP_COMMITMENT,
                question="Leaders allocate adequate resources for risk management",
                measurement_scale="likert",
                weight=1.2
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.LEADERSHIP_COMMITMENT,
                question="Leaders communicate the importance of risk management",
                measurement_scale="likert",
                weight=1.1
            ),
            
            # Risk Awareness
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.RISK_AWARENESS,
                question="Team members understand their role in risk management",
                measurement_scale="likert",
                weight=1.0
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.RISK_AWARENESS,
                question="Team members can identify risks in their work area",
                measurement_scale="likert",
                weight=1.0
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.RISK_AWARENESS,
                question="Risk awareness is part of the onboarding process",
                measurement_scale="yes_no",
                weight=0.8
            ),
            
            # Communication
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.COMMUNICATION,
                question="Risk information flows freely across the organization",
                measurement_scale="likert",
                weight=1.0
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.COMMUNICATION,
                question="Team members feel comfortable reporting risk concerns",
                measurement_scale="likert",
                weight=1.1
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.COMMUNICATION,
                question="Risk discussions happen regularly in team meetings",
                measurement_scale="frequency",
                weight=0.9
            ),
            
            # Accountability
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.ACCOUNTABILITY,
                question="Team members are held accountable for risk management",
                measurement_scale="likert",
                weight=1.0
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.ACCOUNTABILITY,
                question="Risk responsibilities are clearly defined",
                measurement_scale="yes_no",
                weight=0.9
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.ACCOUNTABILITY,
                question="Risk performance is included in performance reviews",
                measurement_scale="yes_no",
                weight=0.8
            ),
            
            # Continuous Improvement
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.CONTINUOUS_IMPROVEMENT,
                question="Lessons learned from incidents are documented and shared",
                measurement_scale="likert",
                weight=1.1
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.CONTINUOUS_IMPROVEMENT,
                question="Risk management processes are regularly reviewed and updated",
                measurement_scale="likert",
                weight=1.0
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.CONTINUOUS_IMPROVEMENT,
                question="Improvement suggestions are welcomed and acted upon",
                measurement_scale="likert",
                weight=1.0
            ),
            
            # Incident Learning
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.INCIDENT_LEARNING,
                question="Incidents are viewed as learning opportunities",
                measurement_scale="likert",
                weight=1.1
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.INCIDENT_LEARNING,
                question="Blame-free environment exists for incident reporting",
                measurement_scale="likert",
                weight=1.2
            ),
            CultureAssessmentQuestion(
                dimension=RiskCultureDimension.INCIDENT_LEARNING,
                question="Root cause analysis is performed for significant incidents",
                measurement_scale="yes_no",
                weight=1.0
            )
        ]
        
        return questions
    
    def conduct_culture_assessment(self, responses: Dict[str, any]) -> List[CultureAssessmentResult]:
        """
        Провести оценку культуры рисков на основе ответов
        
        Args:
            responses: Словарь ответов {question_text: response_value}
        """
        results = []
        
        # Группировка ответов по измерениям
        responses_by_dimension = {}
        for question in self.questions:
            if question.question in responses:
                if question.dimension not in responses_by_dimension:
                    responses_by_dimension[question.dimension] = []
                
                responses_by_dimension[question.dimension].append({
                    'question': question,
                    'response': responses[question.question],
                    'weight': question.weight
                })
        
        # Оценка каждого измерения
        for dimension, dimension_responses in responses_by_dimension.items():
            score = self._calculate_dimension_score(dimension_responses)
            maturity_level = self._determine_maturity_level(score)
            improvement_areas = self._identify_improvement_areas(dimension, dimension_responses)
            recommendations = self._generate_recommendations(dimension, improvement_areas)
            
            result = CultureAssessmentResult(
                dimension=dimension,
                score=score,
                maturity_level=maturity_level,
                improvement_areas=improvement_areas,
                recommendations=recommendations
            )
            
            results.append(result)
        
        self.assessment_results = results
        return results
    
    def _calculate_dimension_score(self, responses: List[Dict]) -> float:
        """Рассчитать балл для измерения культуры"""
        total_weighted_score = 0
        total_weight = 0
        
        for item in responses:
            question = item['question']
            response = item['response']
            weight = item['weight']
            
            # Преобразование ответа в числовое значение
            numeric_response = self._convert_response_to_numeric(question.measurement_scale, response)
            
            total_weighted_score += numeric_response * weight
            total_weight += weight
        
        if total_weight == 0:
            return 1.0  # Минимальный балл по умолчанию
        
        return min(5.0, max(1.0, total_weighted_score / total_weight))
    
    def _convert_response_to_numeric(self, scale: str, response) -> float:
        """Преобразовать ответ в числовое значение"""
        if scale == "likert":
            # Предполагаем шкалу 1-5
            if isinstance(response, (int, float)):
                return float(response)
            elif isinstance(response, str):
                mapping = {
                    "strongly_disagree": 1, "disagree": 2, "neutral": 3,
                    "agree": 4, "strongly_agree": 5,
                    "never": 1, "rarely": 2, "sometimes": 3, "often": 4, "always": 5
                }
                return float(mapping.get(response.lower(), 3))  # neutral по умолчанию
        elif scale == "yes_no":
            if isinstance(response, bool):
                return 5.0 if response else 1.0
            elif isinstance(response, str):
                return 5.0 if response.lower() in ['yes', 'true', 'present'] else 1.0
        elif scale == "frequency":
            if isinstance(response, str):
                mapping = {
                    "never": 1, "rarely": 2, "monthly": 3, "weekly": 4, "daily": 5
                }
                return float(mapping.get(response.lower(), 3))
        
        return 3.0  # neutral по умолчанию
    
    def _determine_maturity_level(self, score: float) -> CultureMaturityLevel:
        """Определить уровень зрелости по баллу"""
        if score <= 1.8:
            return CultureMaturityLevel.INITIAL
        elif score <= 2.6:
            return CultureMaturityLevel.MANAGED
        elif score <= 3.4:
            return CultureMaturityLevel.DEFINED
        elif score <= 4.2:
            return CultureMaturityLevel.QUANTITATIVELY_MANAGED
        else:
            return CultureMaturityLevel.OPTIMIZING
    
    def _identify_improvement_areas(self, dimension: RiskCultureDimension, 
                                  responses: List[Dict]) -> List[str]:
        """Определить области для улучшения"""
        improvement_areas = []
        
        low_scoring_questions = [
            item for item in responses 
            if self._convert_response_to_numeric(item['question'].measurement_scale, item['response']) < 3.0
        ]
        
        for item in low_scoring_questions:
            improvement_areas.append(f"Improve '{item['question'].question}' - Current score: {self._convert_response_to_numeric(item['question'].measurement_scale, item['response']):.1f}")
        
        return improvement_areas
    
    def _generate_recommendations(self, dimension: RiskCultureDimension, 
                                improvement_areas: List[str]) -> List[str]:
        """Сгенерировать рекомендации"""
        recommendations = []
        
        if improvement_areas:
            recommendations.append(f"Focus on strengthening {dimension.value} capabilities")
            recommendations.extend([f"- {area}" for area in improvement_areas])
        
        # Общие рекомендации по измерению
        general_recommendations = {
            RiskCultureDimension.LEADERSHIP_COMMITMENT: [
                "Increase leadership visibility in risk activities",
                "Align risk objectives with business goals",
                "Provide visible support for risk initiatives"
            ],
            RiskCultureDimension.RISK_AWARENESS: [
                "Implement regular risk awareness training",
                "Create risk communication materials",
                "Establish risk champions program"
            ],
            RiskCultureDimension.COMMUNICATION: [
                "Establish regular risk forums",
                "Improve risk reporting mechanisms",
                "Encourage open dialogue about risks"
            ],
            RiskCultureDimension.ACCOUNTABILITY: [
                "Clarify risk roles and responsibilities",
                "Integrate risk metrics into KPIs",
                "Establish risk accountability frameworks"
            ],
            RiskCultureDimension.CONTINUOUS_IMPROVEMENT: [
                "Implement lessons learned processes",
                "Regularly review and update procedures",
                "Foster innovation in risk management"
            ],
            RiskCultureDimension.INCIDENT_LEARNING: [
                "Create blame-free reporting environment",
                "Conduct thorough root cause analyses",
                "Share incident learnings across organization"
            ]
        }
        
        recommendations.extend(general_recommendations.get(dimension, []))
        
        return recommendations
    
    def create_training_program(self, name: str, target_audience: str, 
                              duration_weeks: int, objectives: List[str]) -> Dict:
        """Создать программу обучения культуре рисков"""
        program = {
            'name': name,
            'target_audience': target_audience,
            'duration_weeks': duration_weeks,
            'objectives': objectives,
            'modules': self._design_training_modules(name),
            'assessment_method': 'pre_post_survey',
            'success_metrics': [
                'Knowledge gain',
                'Behavior change',
                'Culture assessment improvement'
            ]
        }
        
        self.training_programs.append(program)
        return program
    
    def _design_training_modules(self, program_name: str) -> List[Dict]:
        """Спроектировать модули обучения"""
        base_modules = [
            {
                'name': 'Introduction to Risk Culture',
                'duration_hours': 2,
                'learning_objectives': ['Define risk culture', 'Understand its importance']
            },
            {
                'name': 'Personal Risk Responsibility',
                'duration_hours': 3,
                'learning_objectives': ['Identify personal risk roles', 'Recognize individual impact']
            },
            {
                'name': 'Risk Communication Skills',
                'duration_hours': 2,
                'learning_objectives': ['Effective risk communication', 'Reporting procedures']
            },
            {
                'name': 'Incident Learning and Improvement',
                'duration_hours': 2,
                'learning_objectives': ['Learn from incidents', 'Continuous improvement mindset']
            }
        ]
        
        # Добавить специфические модули в зависимости от программы
        if 'leadership' in program_name.lower():
            base_modules.append({
                'name': 'Leading Risk Culture',
                'duration_hours': 4,
                'learning_objectives': ['Leadership role in culture', 'Setting the tone']
            })
        
        return base_modules
    
    def implement_culture_initiative(self, initiative_name: str, 
                                   target_dimension: RiskCultureDimension,
                                   start_date: str, budget: float,
                                   success_criteria: List[str]) -> Dict:
        """Реализовать инициативу по формированию культуры рисков"""
        initiative = {
            'name': initiative_name,
            'target_dimension': target_dimension.value,
            'start_date': start_date,
            'budget': budget,
            'success_criteria': success_criteria,
            'status': 'Planning',
            'timeline': self._create_initiative_timeline(initiative_name),
            'responsible_team': self._assign_responsible_team(target_dimension),
            'communication_plan': self._create_communication_plan(initiative_name),
            'progress_metrics': [
                'Participation rates',
                'Engagement scores',
                'Behavior change indicators',
                'Culture assessment scores'
            ]
        }
        
        self.culture_initiatives.append(initiative)
        return initiative
    
    def _create_initiative_timeline(self, initiative_name: str) -> List[Dict]:
        """Создать временной план инициативы"""
        phases = [
            {'phase': 'Planning', 'duration_weeks': 2, 'activities': ['Define scope', 'Allocate resources']},
            {'phase': 'Implementation', 'duration_weeks': 8, 'activities': ['Execute activities', 'Monitor progress']},
            {'phase': 'Evaluation', 'duration_weeks': 2, 'activities': ['Measure outcomes', 'Assess impact']},
            {'phase': 'Sustainment', 'duration_weeks': 4, 'activities': ['Embed practices', 'Continuous monitoring']}
        ]
        
        timeline = []
        current_week = 0
        
        for phase in phases:
            timeline.append({
                'phase': phase['phase'],
                'start_week': current_week,
                'end_week': current_week + phase['duration_weeks'],
                'activities': phase['activities']
            })
            current_week += phase['duration_weeks']
        
        return timeline
    
    def _assign_responsible_team(self, dimension: RiskCultureDimension) -> str:
        """Назначить ответственную команду"""
        team_assignments = {
            RiskCultureDimension.LEADERSHIP_COMMITMENT: 'Executive Leadership',
            RiskCultureDimension.RISK_AWARENESS: 'Training Department',
            RiskCultureDimension.COMMUNICATION: 'Communications Team',
            RiskCultureDimension.ACCOUNTABILITY: 'HR Department',
            RiskCultureDimension.CONTINUOUS_IMPROVEMENT: 'Process Excellence',
            RiskCultureDimension.INCIDENT_LEARNING: 'Quality Assurance'
        }
        
        return team_assignments.get(dimension, 'Risk Management Office')
    
    def _create_communication_plan(self, initiative_name: str) -> Dict:
        """Создать план коммуникации"""
        return {
            'channels': ['Email', 'Town halls', 'Intranet', 'Team meetings'],
            'frequency': 'Bi-weekly updates',
            'stakeholders': ['All employees', 'Management', 'Risk committee'],
            'materials': ['Newsletters', 'Posters', 'Videos', 'Presentations']
        }
    
    def generate_culture_report(self) -> str:
        """Сгенерировать отчет по культуре рисков"""
        report = []
        report.append("# Risk Culture Assessment Report\n")
        report.append(f"Generated on: {pd.Timestamp.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        
        # Сводка по оценке культуры
        report.append("## Culture Assessment Summary\n")
        if self.assessment_results:
            avg_score = sum(r.score for r in self.assessment_results) / len(self.assessment_results)
            report.append(f"- Average Culture Score: {avg_score:.2f}/5.0\n")
            report.append(f"- Overall Maturity Level: {self._get_overall_maturity().value[1]}\n\n")
            
            report.append("### Dimension Scores:\n")
            for result in self.assessment_results:
                report.append(f"- {result.dimension.value}: {result.score:.2f} "
                            f"({result.maturity_level.value[1]})\n")
        else:
            report.append("- No culture assessment conducted yet\n\n")
        
        # Инициативы по культуре
        report.append("## Culture Initiatives\n")
        report.append(f"- Total Initiatives: {len(self.culture_initiatives)}\n")
        
        for i, initiative in enumerate(self.culture_initiatives, 1):
            report.append(f"### {i}. {initiative['name']}\n")
            report.append(f"**Target Dimension**: {initiative['target_dimension']}\n")
            report.append(f"**Status**: {initiative['status']}\n")
            report.append(f"**Budget**: ${initiative['budget']:,.2f}\n")
            report.append(f"**Responsible Team**: {initiative['responsible_team']}\n")
            report.append("\n")
        
        # Программы обучения
        report.append("## Training Programs\n")
        report.append(f"- Total Programs: {len(self.training_programs)}\n")
        
        for i, program in enumerate(self.training_programs, 1):
            report.append(f"### {i}. {program['name']}\n")
            report.append(f"**Target Audience**: {program['target_audience']}\n")
            report.append(f"**Duration**: {program['duration_weeks']} weeks\n")
            report.append(f"**Modules**: {len(program['modules'])}\n")
            report.append("\n")
        
        # Рекомендации
        report.append("## Key Recommendations\n")
        all_recommendations = []
        for result in self.assessment_results:
            all_recommendations.extend(result.recommendations)
        
        if all_recommendations:
            for rec in all_recommendations[:10]:  # Показать первые 10 рекомендаций
                report.append(f"- {rec}\n")
        else:
            report.append("- Conduct culture assessment to generate recommendations\n")
        
        return "".join(report)
    
    def _get_overall_maturity(self) -> CultureMaturityLevel:
        """Определить общий уровень зрелости культуры"""
        if not self.assessment_results:
            return CultureMaturityLevel.INITIAL
        
        avg_score = sum(r.score for r in self.assessment_results) / len(self.assessment_results)
        return self._determine_maturity_level(avg_score)

# Пример использования фреймворка культуры рисков
def example_risk_culture():
    framework = RiskCultureFramework()
    
    # Симуляция ответов на оценку культуры
    sample_responses = {
        "Leaders actively participate in risk management activities": "agree",
        "Leaders allocate adequate resources for risk management": "neutral", 
        "Leaders communicate the importance of risk management": "agree",
        "Team members understand their role in risk management": "sometimes",
        "Team members can identify risks in their work area": "often",
        "Risk awareness is part of the onboarding process": False,
        "Risk information flows freely across the organization": "agree",
        "Team members feel comfortable reporting risk concerns": "neutral",
        "Risk discussions happen regularly in team meetings": "weekly",
        "Team members are held accountable for risk management": "disagree",
        "Risk responsibilities are clearly defined": False,
        "Risk performance is included in performance reviews": False,
        "Lessons learned from incidents are documented and shared": "rarely",
        "Risk management processes are regularly reviewed and updated": "neutral",
        "Improvement suggestions are welcomed and acted upon": "sometimes",
        "Incidents are viewed as learning opportunities": "agree",
        "Blame-free environment exists for incident reporting": "disagree",
        "Root cause analysis is performed for significant incidents": True
    }
    
    # Проведение оценки культуры
    assessment_results = framework.conduct_culture_assessment(sample_responses)
    
    print("Risk Culture Assessment Results:")
    print("=" * 40)
    for result in assessment_results:
        print(f"{result.dimension.value}: {result.score:.2f} ({result.maturity_level.value[1]})")
        print(f"  Improvement Areas: {len(result.improvement_areas)}")
        print(f"  Recommendations: {len(result.recommendations)}")
        print()
    
    # Создание программы обучения
    training_program = framework.create_training_program(
        name="Risk Culture for Engineers",
        target_audience="Engineering Teams",
        duration_weeks=4,
        objectives=[
            "Enhance risk awareness among developers",
            "Improve secure coding practices",
            "Strengthen incident reporting culture"
        ]
    )
    
    print(f"Created training program: {training_program['name']}")
    print(f"Modules: {len(training_program['modules'])}")
    
    # Реализация инициативы по культуре
    culture_initiative = framework.implement_culture_initiative(
        initiative_name="Leadership Risk Champions Program",
        target_dimension=RiskCultureDimension.LEADERSHIP_COMMITMENT,
        start_date="2024-01-15",
        budget=50000,
        success_criteria=[
            "Increased leadership participation in risk activities",
            "Improved risk communication from leadership",
            "Higher risk culture assessment scores"
        ]
    )
    
    print(f"Implemented initiative: {culture_initiative['name']}")
    print(f"Timeline phases: {len(culture_initiative['timeline'])}")
    
    # Генерация отчета
    report = framework.generate_culture_report()
    print(f"\nGenerated culture report ({len(report)} characters)")
    
    return framework

if __name__ == "__main__":
    example_risk_culture()
```

## Лучшие практики управления рисками в CI/CD

### Планирование и стратегия

- Ранняя идентификация рисков в процессе разработки
- Интеграция оценки рисков в CI/CD пайплайны
- Создание культуры безопасности и ответственности
- Регулярный пересмотр и обновление оценки рисков

### Мониторинг и отчетность

- Непрерывный мониторинг метрик рисков
- Автоматические алерты при превышении порогов
- Регулярные отчеты для заинтересованных сторон
- Визуализация рисков и трендов

### Инцидент-менеджмент

- Четкие процедуры реагирования на инциденты
- Регулярные учения и тренировки
- Анализ инцидентов и извлечение уроков
- Постоянное улучшение процессов

## Заключение

Управление рисками в CI/CD является критическим компонентом обеспечения надежности, безопасности и качества доставки программного обеспечения. Оно требует системного подхода, включающего идентификацию, оценку, контроль и мониторинг рисков на всех этапах процесса доставки.

Успешное управление рисками в CI/CD обеспечивает:

- Снижение вероятности и воздействия негативных событий
- Повышение надежности и безопасности систем
- Улучшение качества программного обеспечения
- Соблюдение нормативных требований
- Повышение доверия пользователей и заинтересованных сторон

Организации, инвестирующие в эффективное управление рисками в CI/CD, получают значительные преимущества в стабильности, безопасности и надежности своих систем доставки программного обеспечения.

## Связанные концепции

- [[Security Best Practices]]
- [[Compliance Management]]
- [[Quality Assurance]]
- [[Continuous Integration]]
- [[Continuous Deployment]]
- [[DevOps Practices]]
- [[Monitoring and Observability]]
- [[Incident Response]]

## Внешние ресурсы

- [NIST Risk Management Framework](https://csrc.nist.gov/publications/detail/sp/800-39/final)
- [ISO 31000 Risk Management](https://www.iso.org/isoiec-31000-risk-management.html)
- [OWASP Risk Assessment Methodology](https://owasp.org/www-community/Application_Threat_Modeling)
- [FAIR Risk Analysis Model](https://www.fairinstitute.org/)