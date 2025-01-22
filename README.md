### Инструкция к выполению

1. Практические задачи выполняйте на личной рабочей станции или созданной ВМ (рекомендую debian).
2. Своё решение к задачам оформите в вашем GitHub репозитории в формате **markdown**!!!

### **Цель лабораторной работы**  
1. Настроить взаимодействие между Jenkins, GitLab, SonarQube, PostgreSQL и Nexus.  
2. Выполнить полный DevOps-цикл для Python-приложения: тестирование, анализ кода, сборка Docker-образа и публикация в Nexus.  
3. Автоматизировать интеграцию с использованием Docker Compose.  

---

### **Простое Python-приложение**  
Приложение будет представлять собой утилиту `text_tool.py`, которая выполняет простые операции с текстом:  
- Подсчитывает количество слов.  
- Находит самое длинное слово.  

---

### **Задача 1. Подготовка окружения**  
1. Настройте `docker-compose.yaml` для сервисов:
   - **Jenkins**  
   - **GitLab**  
   - **SonarQube (с PostgreSQL)**  
   - **Nexus**  

2. Поднимите все сервисы через `docker-compose up -d`.  
3. Протестируйте их доступность.  

Приложите файл docker-compose.yaml и скриншоты работающих сервисов.

---

### **Задача 2. Создание Python-приложения**  
1. Создайте файл `text_tool.py` с функционалом:  

```python
def count_words(text):
    """Подсчитывает количество слов в тексте."""
    return len(text.split())

def longest_word(text):
    """Находит самое длинное слово в тексте."""
    words = text.split()
    return max(words, key=len) if words else None
```

2. Напишите модульные тесты в файле `test_text_tool.py`:  

```python
import pytest
from text_tool import count_words, longest_word

def test_count_words():
    assert count_words("Hello world") == 2
    assert count_words("") == 0

def test_longest_word():
    assert longest_word("Hello world") == "Hello"
    assert longest_word("") is None
```

3. Создайте файл `requirements.txt` с зависимостями:  

```
pytest
```

4. Загрузите приложение и тесты в новый репозиторий GitLab.  

---

### **Задача 3. Интеграция инструментов**  
#### **GitLab CI/CD**  
1. Настройте `.gitlab-ci.yml`:  

```yaml
stages:
  - test

test:
  stage: test
  image: python:3.10
  script:
    - pip install -r requirements.txt
    - pytest --junitxml=report.xml
  artifacts:
    paths:
      - report.xml
```

2. Убедитесь, что при каждом пуше запускаются тесты.  

#### **Jenkins**  
1. Настройте Jenkins Pipeline, который:  
   - Клонирует репозиторий.  
   - Выполняет тесты (`pytest`).  
   - Генерирует отчёт о покрытии тестами.  

Пример `Jenkinsfile`:  

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://<ваш_репозиторий>.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'pytest --junitxml=report.xml'
            }
        }
    }
    post {
        always {
            junit 'report.xml'
        }
    }
}
```

---

### **Задача 4. Анализ кода с SonarQube**  

1. Настройте SonarQube для анализа Python-кода:  
   - Установите плагин для Python.  
   - Создайте проект в интерфейсе SonarQube.  
   - Получите ключ для интеграции.  

2. Добавьте шаг анализа в Jenkins:  

```groovy
stage('Code Analysis') {
    steps {
        sh 'sonar-scanner \
            -Dsonar.projectKey=python-lab \
            -Dsonar.sources=. \
            -Dsonar.host.url=http://<sonarqube_url> \
            -Dsonar.login=<token>'
    }
}
```

---

### **Задача 5. Публикация Docker-образа в Nexus**  
1. Создайте `Dockerfile`:  

```dockerfile
FROM python:3.10
COPY text_tool.py /app/
WORKDIR /app
CMD ["python"]
```

2. Настройте шаг публикации в Jenkins:  

```groovy
stage('Build and Push Docker Image') {
    steps {
        sh 'docker build -t nexus:5000/python-lab:${BUILD_NUMBER} .'
        sh 'docker push nexus:5000/python-lab:${BUILD_NUMBER}'
    }
}
```

3. Проверьте, что образ загружен в Nexus.  

---
