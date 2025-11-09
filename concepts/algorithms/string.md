---
aliases: ["Алгоритмы строк", "Строковые алгоритмы"]
tags: 
  - #algorithms
  - #strings
  - #data-structures
  - #complexity
---

# Строковые алгоритмы

## Введение

Строковые алгоритмы представляют собой важную область компьютерных наук, посвященную эффективной обработке текстовых данных. Они находят широкое применение в поисковых системах, текстовых редакторах, биоинформатике, компиляторах и других приложениях.

## Алгоритм Кнута-Морриса-Пратта (KMP)

Алгоритм Кнута-Морриса-Пратта используется для поиска подстроки в строке за линейное время, избегая повторного сравнения уже обработанных символов.

### Временная и пространственная сложность
- Время: O(n + m), где n - длина текста, m - длина образца
- Память: O(m) для массива префиксов

### Реализация

```python
def kmp_search(text, pattern):
    if not pattern:
        return 0
    
    # Вычисление префикс-функции
    def compute_lps(pattern):
        lps = [0] * len(pattern)
        length = 0
        i = 1
        
        while i < len(pattern):
            if pattern[i] == pattern[length]:
                length += 1
                lps[i] = length
                i += 1
            else:
                if length != 0:
                    length = lps[length - 1]
                else:
                    lps[i] = 0
                    i += 1
        return lps
    
    lps = compute_lps(pattern)
    i = 0  # индекс для text
    j = 0  # индекс для pattern
    
    while i < len(text):
        if pattern[j] == text[i]:
            i += 1
            j += 1
        
        if j == len(pattern):
            return i - j  # Найдено совпадение
        elif i < len(text) and pattern[j] != text[i]:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
    
    return -1  # Не найдено
```

### Когда использовать
- Когда текст и образец известны заранее
- Для точного поиска подстроки
- Когда важна предсказуемая производительность

### Преимущества и недостатки
- **Преимущества**: Гарантированное линейное время выполнения
- **Недостатки**: Дополнительная память для префикс-массива

## Алгоритм Рабина-Карпа

Алгоритм Рабина-Карпа использует хеширование для поиска подстроки в строке, эффективно обрабатывая коллизии.

### Временная и пространственная сложность
- Время: O(n + m) в среднем, O(nm) в худшем случае
- Память: O(1)

### Реализация

```python
def rabin_karp_search(text, pattern, base=256, prime=101):
    n = len(text)
    m = len(pattern)
    
    if m > n:
        return -1
    
    # Вычисляем хеш для образца и первой подстроки текста
    pattern_hash = 0
    text_hash = 0
    h = 1  # h = pow(base, m-1) % prime
    
    for i in range(m - 1):
        h = (h * base) % prime
    
    # Вычисляем хеш для образца и первой подстроки текста
    for i in range(m):
        pattern_hash = (base * pattern_hash + ord(pattern[i])) % prime
        text_hash = (base * text_hash + ord(text[i])) % prime
    
    # Проверяем совпадения хешей
    for i in range(n - m + 1):
        if pattern_hash == text_hash:
            # Проверяем символы один за другим
            if text[i:i+m] == pattern:
                return i
        
        # Вычисляем хеш для следующей подстроки
        if i < n - m:
            text_hash = (base * (text_hash - ord(text[i]) * h) + ord(text[i + m])) % prime
            
            # Обрабатываем отрицательный хеш
            if text_hash < 0:
                text_hash += prime
    
    return -1
```

### Когда использовать
- Для поиска нескольких образцов одновременно
- При поиске подстрок с одинаковой длиной
- В приложениях, где важна простота реализации

### Преимущества и недостатки
- **Преимущества**: Простота реализации, возможность поиска нескольких образцов
- **Недостатки**: Потенциальные коллизии хешей, худшее время O(nm)

## Z-алгоритм

Z-алгоритм находит все вхождения строки в себя (Z-функция), что полезно для решения задач на строки.

### Временная и пространственная сложность
- Время: O(n)
- Память: O(n)

### Реализация

```python
def compute_z(s):
    n = len(s)
    z = [0] * n
    l, r = 0, 0
    
    for i in range(1, n):
        if i <= r:
            z[i] = min(r - i + 1, z[i - l])
        
        while i + z[i] < n and s[z[i]] == s[i + z[i]]:
            z[i] += 1
        
        if i + z[i] - 1 > r:
            l, r = i, i + z[i] - 1
    
    return z

def z_algorithm_search(text, pattern):
    concat = pattern + "$" + text
    z = compute_z(concat)
    occurrences = []
    
    for i in range(len(pattern) + 1, len(concat)):
        if z[i] == len(pattern):
            occurrences.append(i - len(pattern) - 1)
    
    return occurrences
```

### Когда использовать
- Для решения задач на строки, связанных с суффиксами
- При необходимости находить все вхождения подстроки
- В задачах на палиндромы и симметрию строк

### Преимущества и недостатки
- **Преимущества**: Линейное время, универсальность
- **Недостатки**: Требует дополнительной памяти для конкатенации

## Алгоритм Бойера-Мура

Алгоритм Бойера-Мура использует два эвристических правила для пропуска символов при поиске, что делает его быстрым на практике.

### Временная и пространственная сложность
- Время: O(nm) в худшем случае, O(n/m) в среднем
- Память: O(m + alphabet_size)

### Реализация

```python
def bad_char_table(pattern):
    table = {}
    for i in range(len(pattern)):
        table[pattern[i]] = i
    return table

def good_suffix_table(pattern):
    m = len(pattern)
    suffix = [0] * m
    border = [0] * m
    
    # Вычисление суффиксной таблицы
    f = g = m - 1
    for i in range(m - 1, -1, -1):
        if i == m - 1 or i == g and f == g:
            g -= 1
            while g >= 0 and pattern[g] == pattern[m - 1 - (i - g)]:
                g -= 1
            f = g + 1
        border[i] = f + m - 1 - i
    
    # Вычисление таблицы смещений
    shift = [0] * m
    for i in range(m):
        shift[i] = m
    j = 0
    
    for i in range(m - 1, -1, -1):
        if suffix[i] == i + 1:
            while j < m - 1 - i:
                if shift[j] == m:
                    shift[j] = m - 1 - i
                j += 1
    
    for i in range(m - 1):
        shift[m - 1 - suffix[i]] = m - 1 - i
    
    return shift

def boyer_moore_search(text, pattern):
    if not pattern:
        return 0
    
    n, m = len(text), len(pattern)
    bad_char = bad_char_table(pattern)
    good_suffix = good_suffix_table(pattern)
    
    s = 0  # смещение шаблона относительно текста
    while s <= n - m:
        j = m - 1
        
        while j >= 0 and pattern[j] == text[s + j]:
            j -= 1
        
        if j < 0:
            return s  # Совпадение найдено
        else:
            bad_char_shift = j - bad_char.get(text[s + j], -1)
            good_suffix_shift = good_suffix[j]
            s += max(bad_char_shift, good_suffix_shift)
    
    return -1
```

### Когда использовать
- Для длинных образцов с небольшим алфавитом
- Когда текст содержит мало совпадений
- В приложениях, где важна средняя производительность

### Преимущества и недостатки
- **Преимущества**: Отличная средняя производительность
- **Недостатки**: Сложная реализация, худшее время O(nm)

## Строковое хеширование

Строковое хеширование позволяет сравнивать подстроки за O(1) после предварительной обработки за O(n).

### Временная и пространственная сложность
- Время: O(n) для предварительной обработки, O(1) для запросов
- Память: O(n)

### Реализация

```python
class StringHasher:
    def __init__(self, s, base=256, mod=10**9 + 7):
        self.s = s
        self.base = base
        self.mod = mod
        self.n = len(s)
        
        # Вычисляем хеши префиксов
        self.hashes = [0] * (self.n + 1)
        self.powers = [1] * (self.n + 1)
        
        for i in range(self.n):
            self.hashes[i + 1] = (self.hashes[i] * self.base + ord(s[i])) % self.mod
            self.powers[i + 1] = (self.powers[i] * self.base) % self.mod
    
    def get_hash(self, left, right):
        """Возвращает хеш подстроки s[left:right]"""
        hash_val = (self.hashes[right] - self.hashes[left] * self.powers[right - left]) % self.mod
        return hash_val
    
    def compare_substrings(self, left1, right1, left2, right2):
        """Сравнивает две подстроки"""
        if right1 - left1 != right2 - left2:
            return False
        return self.get_hash(left1, right1) == self.get_hash(left2, right2)
```

### Когда использовать
- Для сравнения подстрок
- В задачах на палиндромы, суффиксы, префиксы
- При необходимости быстрого сравнения строк

## Структура данных Тrie (Префиксное дерево)

Trie - это древовидная структура данных, эффективно хранящая строки и позволяющая быстро искать префиксы.

### Временная и пространственная сложность
- Время: O(m) для вставки и поиска (m - длина строки)
- Память: O(ALPHABET_SIZE * N * M), где N - количество строк, M - средняя длина

### Реализация

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()
    
    def insert(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_end_of_word = True
    
    def search(self, word):
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_end_of_word
    
    def starts_with(self, prefix):
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
    
    def autocomplete(self, prefix):
        node = self.root
        # Сначала спускаемся до конца префикса
        for char in prefix:
            if char not in node.children:
                return []
            node = node.children[char]
        
        # Затем собираем все слова с этим префиксом
        results = []
        self._dfs(node, prefix, results)
        return results
    
    def _dfs(self, node, prefix, results):
        if node.is_end_of_word:
            results.append(prefix)
        
        for char, child_node in node.children.items():
            self._dfs(child_node, prefix + char, results)
```

### Когда использовать
- Для автозаполнения и поиска по префиксу
- В поисковых системах
- Для хранения словарей и проверки орфографии

### Преимущества и недостатки
- **Преимущества**: Быстрый поиск по префиксу, эффективное хранение общих префиксов
- **Недостатки**: Высокое потребление памяти для разнообразных строк

## Практические применения

Строковые алгоритмы находят широкое применение в:

- [[text-processing]] - обработка и анализ текстов
- [[bioinformatics]] - анализ ДНК и белковых последовательностей
- [[search-engines]] - индексация и поиск документов
- [[compilers]] - лексический анализ и синтаксический разбор
- [[data-compression]] - алгоритмы сжатия, основанные на строковых паттернах

## Сравнение алгоритмов

| Алгоритм | Время | Память | Когда использовать |
|----------|-------|--------|-------------------|
| KMP | O(n + m) | O(m) | Гарантированное линейное время |
| Рабин-Карп | O(n + m) avg | O(1) | Поиск нескольких образцов |
| Z-алгоритм | O(n) | O(n) | Задачи на суффиксы и префиксы |
| Бойер-Мур | O(n/m) avg | O(m) | Длинные образцы, маленький алфавит |
| Хеширование | O(1) запрос | O(n) | Сравнение подстрок |
| Trie | O(m) | O(N*M) | Поиск по префиксу |

## Заключение

Строковые алгоритмы - мощный инструмент для эффективной обработки текстовых данных. Выбор конкретного алгоритма зависит от требований задачи, размера данных и характера входных данных. Понимание этих алгоритмов важно для разработчиков, работающих с текстовыми данными, системами поиска и обработки естественного языка.

См. также: [[algorithms]], [[data-structures]], [[complexity-analysis]]