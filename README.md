# Criação de uma API usando Django REST Framework

Este arquivo descreve o processo de criação de uma API REST a partir do zero.
Neste caso, uma API para gerenciamento de uma escola.

É praticamento um passo-a-passo do que foi demonstrado pelo Guilherme Lima
[neste vídeo](https://www.youtube.com/watch?v=BKChTO8GADk) do YouTube.

## Preparando o ambiente

### Criando o diretório do projeto

 ```sh
mkdir django-api
cd django-api
```

### Criando o ambiente desenvolvimento virtual (VDE)

No vídeo original é mostrado como usar o python venv para gerenciar o ambiente e
pip para gerenciar a instalação dos pacotes.
Aqui descrevo como usar o conda, que gerencia tanto o ambiente virtual quanto os
pacotes.
```sh
conda create --prefix ./venv python django djangorestframework  # siga as instruções
conda activate ./venv
conda list -e > environment.yml  # salva as especificações do ambiente
```

> Daqui pra frente, assume-se que o VDE esteja sempre ativo!

### Inicializando o projeto Django

```sh
# Cria `manage.py` e a pasta `config` com a estrutura básica do projeto
django-admin startproject config .
```

```sh
# Sobe o servidor
python manage.py runserver
# Podemos acessar o projeto pelo web browser; url: http://127.0.0.1:8000/
```

### Criando a aplicação 'escola'

```sh
# Cria a estrutura da app na pasta 'escola'
python manage.py startapp escola
```

Edita-se o arquivo `config/settings.py` para registrar `escola` na lista `INSTALLED_APPS`

```python
# config/settings.py
# (...)
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'escola',   # adicionado à lista pré-existente
]
```

### Atualizando a aplicação

```sh
python manage.py makemigrations
python manage.py migrate
```

Neste ponto, o servidor só mostra a página inicial da Django.
Para ver a aplicação, precisamos criar um superusuário.

### Criando um superusuário

```sh
python manage.py createsuperuser
```

Agora podemos acessar a página de administração do projeto em <http://127.0.0.1:8000/admin/>

## Desenvolvendo a API

A API será desenvolvida com Django REST.
Portanto, precisamos incluir `rest_framework` em `config/settings.py`, na lista de apps instaladas:

```python
# config/settings.py
# (...)
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'escola',
    'rest_framework',  # Django REST framework
]
# (...)
```

### Criando os o modelo 'Aluno' da app 'escola'

Edita-se o arquivo `escola/models.py` para criar a classe `Aluno`

```python
# escola/models.py
from django.db import models

class Aluno(models.Model):
    nome = models.CharField(max_length=30)
    rg = models.CharField(max_length=9)

    def __str__(self):
        return self.nome
```

### Criando o serializer do modelo 'Aluno'

O serializer é responsável por converter objetos em JSON e vice versa.
Os arquivos JSON são usados nas requisições HTTP.

Neste passo, cria-se o arquivo `escola/serializer.py`

```python
# escola/serializer.py
from rest_framework import serializers
from escola.models import Aluno

class AlunoSerializer(serializers.ModelSerializer):
    class Meta:     # é necessário criar a classe Meta
        model = Aluno
        fields = ['id', 'nome', 'rg']  # campos que serão incluídos no JSON
        # N.B. Não é obrigatório serializar todos os attributos definidos em escola.models.Aluno!
        # P.ex., poderíamos excluir 'rg' para restringir acesso a informações sensíveis.
        # Note também que 'id' é herdado de 'models.Model' pela classe Aluno.
```

### Criando a view para 'Aluno'

Em Django, as páginas são construídas a partir de views.
Neste caso, edita-se o arquivo `escola/views.py`

```python
# escola/views.py
#from django.shortcuts import render    # não será usado!
from rest_framework import viewsets

# NOTE views.py é criado assumindo-se que uma página será renderizada.
#   Como estamos criando uma API, não precisamos do 'django.shortcuts.render',
#   mas de 'rest_framework.viewsets'

from escola.models import Aluno
from escola.serializer import AlunoSerializer

class AlunosViewSet(viewsets.ModelViewSet):
    queryset = Aluno.objects.all()
    serializer_class = AlunoSerializer
```

### Registrando Alunos

O próximo passo é editar `escola/admin.py`

```python
from django.contrib import admin
from escola.models import Aluno

class Alunos(admin.ModelAdmin):
    list_display = ('id', 'nome', 'rg')
    list_display_links = ('id', 'nome')
    search_fields = ('nome',)

admin.site.register(Aluno, Alunos)
```

### Mostrando a API

Por fim, edita-se `config/urls.py` para que a API seja exibida em <http://127.0.0.1:8000/>

```python
# config/urls.py
from django.contrib import admin
from django.urls import path, include

from rest_framework import routers

from escola.views import AlunosViewSet

router = routers.DefaultRouter()
router.register(r'alunos', AlunosViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include(router.urls)),
]
```

## Conclusão

Neste ponto, deveria ser possível acessar a API via <http://127.0.0.1:8000/>

Deve ser possível fazer o CRUD dos Alunos a partir da API.

---

