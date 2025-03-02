# Aplikacje WWW

## Lab 7 - Django Rest Framework, serializacja danych oraz API REST w Django.
---

:warning: :warning: :warning:
> GIT: Rozpoczęcie pracy w ramach kolejnego laba powinno odbywać w nowym branchu po zatwierdzeniu zmian w poprzednim i scaleniu zmian do brancha main, jeżeli wszystko działa poprawnie.

### **1. Wprowadzenie do REST API**

REST API (Representational State Transfer Application Programming Interface) to styl architektoniczny używany do projektowania interfejsów API, który opiera się na protokole HTTP. REST opiera się na kilku zasadach, które zapewniają prostotę, skalowalność i łatwość użycia. Poniżej znajduje się opis podstawowych zasad oraz najczęściej używanych metod żądań w REST API.

**Zasady REST API**

1. **Stateless**: Każde żądanie do serwera musi zawierać wszystkie informacje potrzebne do jego zrozumienia. Serwer nie przechowuje stanu klienta między różnymi żądaniami. To oznacza, że każde żądanie jest niezależne od innych.

2. **Użycie HTTP**: REST API wykorzystuje standardowe metody HTTP, co umożliwia korzystanie z różnych protokołów i narzędzi do komunikacji.

3. **Zasoby**: W REST API zasoby są identyfikowane przez unikalne identyfikatory URI (Uniform Resource Identifier). Zasoby mogą być reprezentowane w różnych formatach, najczęściej jako JSON lub XML.

4. **Interfejs uniformny**: REST API posiada jednolity interfejs, co oznacza, że komunikacja odbywa się w określony sposób, co ułatwia zrozumienie i implementację.

5. **Klient-serwer**: Architektura REST oddziela klienta od serwera, co umożliwia niezależny rozwój obu komponentów.

6. **Cache**: Odpowiedzi z serwera mogą być buforowane, co przyspiesza działanie aplikacji i zmniejsza obciążenie serwera.

**Metody żądań w REST API**

REST API korzysta z różnych metod HTTP do wykonywania operacji na zasobach. Oto najpopularniejsze metody:

1. **GET**:
   - **Opis**: Służy do pobierania zasobów z serwera.
   - **Przykład użycia**: `GET /api/books/` - pobiera listę wszystkich książek.

2. **POST**:
   - **Opis**: Używana do tworzenia nowych zasobów na serwerze.
   - **Przykład użycia**: `POST /api/books/` z danymi książki w ciele żądania, co tworzy nową książkę.

3. **PUT**:
   - **Opis**: Używana do aktualizacji istniejących zasobów. Zazwyczaj wysyła wszystkie dane zasobu, nawet te, które się nie zmieniają.
   - **Przykład użycia**: `PUT /api/books/1/` z danymi aktualizowanej książki w ciele żądania.

4. **PATCH**:
   - **Opis**: Używana do częściowej aktualizacji zasobu. Wysyła tylko te dane, które mają zostać zmienione.
   - **Przykład użycia**: `PATCH /api/books/1/` z danymi, które mają być zmienione.

5. **DELETE**:
   - **Opis**: Służy do usuwania zasobów z serwera.
   - **Przykład użycia**: `DELETE /api/books/1/` - usuwa książkę o identyfikatorze 1.

**Przykłady żądań REST API**

Oto kilka przykładów żądań, które można wysłać do REST API:

- **Pobranie wszystkich książek**:
  ```http
  GET /api/books/
  ```

- **Pobranie konkretnej książki**:
  ```http
  GET /api/books/1/
  ```

- **Utworzenie nowej książki**:
  ```http
  POST /api/books/
  Content-Type: application/json

  {
      "title": "Django for Beginners",
      "author": "William S. Vincent",
      "published_date": "2021-01-01"
  }
  ```

- **Aktualizacja książki**:
  ```http
  PUT /api/books/1/
  Content-Type: application/json

  {
      "title": "Django for Beginners",
      "author": "William S. Vincent",
      "published_date": "2021-01-01"
  }
  ```

- **Częściowa aktualizacja książki**:
  ```http
  PATCH /api/books/1/
  Content-Type: application/json

  {
      "author": "W. S. Vincent"
  }
  ```

- **Usunięcie książki**:
  ```http
  DELETE /api/books/1/
  ```

### **2. Django Rest Framework**

#### **2.1 Wstęp**

Django Rest Framework (DRF) to jeden z dostępnych frameworków (ogólna nazwa określająca spójny zbiór rozwiązań pozwalający na stworzenie własnego rozwiązania opartego o te "ramy"), który dostarcza rozwiązań pozwalających na stworzenie API REST w ramach aplikacji opartej o Django.

> Oficjalna strona: https://www.django-rest-framework.org/

#### **2.2 Instalacja**

> Dokumentacja: https://www.django-rest-framework.org/#installation

Proces instalacji w najprostrzej formie jest dość prosty i składa się z następujących kroków.

**Krok 1**

```console
pip install djangorestframework
# dodatkowo dla wsparcia również dodatkowej funkcjonalności możemy zainstalować
pip install markdown
pip install django-filter
```

**Krok 2**

Modyfikacja pliku `blog\settings.py`
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

I to na razie wystarczy. W późniejszych atapach dodamy jeszcze konfigurację związaną z autentykacją i autoryzacją.


#### **2.3 Serializacja danych w DRF**

Serializacja to proces konwersji złożonych obiektów, takich jak instancje klas, na format, który można łatwo przechowywać lub przesyłać. W kontekście `Django Rest Framework (DRF)`, serializacja polega na przekształceniu danych z modeli Django na formaty takie jak `JSON` lub `XML`, które mogą być przesyłane przez sieć. Serializatory w DRF również umożliwiają deserializację, czyli konwersję danych z formatu `JSON` lub `XML` z powrotem na obiekty Pythona, po uprzednim sprawdzeniu poprawności danych.

Serializatory w DRF działają podobnie do formularzy w Django (`Form` i `ModelForm`). Istnieją dwa główne typy serializatorów:

* `Serializer`: Ogólny sposób kontrolowania requestów.
* `ModelSerializer`: Ułatwia serializację danych modeli Django, automatycznie generując pola na podstawie modelu.

Serializacja jest kluczowa w tworzeniu API, ponieważ umożliwia przesyłanie danych między serwerem a klientem w formacie, który jest łatwy do zrozumienia i przetworzenia przez obie strony.

Po dokładne informacje o serializacji odsyłamy do informacji z wykładu oraz [dokumentacji DRF](https://www.django-rest-framework.org/api-guide/serializers/).

**Przykład klasy do serializacji danych (dziedziczącej po klasie Serializer)\***

**__Listing 1__**
```python
from rest_framework import serializers
from .models import Topic, Category, Post


class TopicSerializer(serializers.Serializer):

    # pole tylko do odczytu, tutaj dla id działa też autoincrement
    id = serializers.IntegerField(read_only=True)

    # pola wymagane
    name = serializers.CharField(required=True)
    category = serializers.PrimaryKeyRelatedField(queryset=Category.objects.all())

    # i pozostałe pola
    created = serializers.DateTimeField()

    # przesłonięcie metody create() z klasy serializers.Serializer
    def create(self, validated_data):
        return Topic.objects.create(**validated_data)

    # przesłonięcie metody update() z klasy serializers.Serializer
    def update(self, instance, validated_data):
        instance.name = validated_data.get('name', instance.name)
        instance.category = validated_data.get('category', instance.category)
        instance.save()
        return instance
```

* Nadpisujemy odpowiednie funkcje `create`, `update` itp. w momencie gdy musimy zapisać obiekty w inny niż domyślny sposób (np. chcemy dodać aktualnie zalogowanego użytkownika jako właściciela stworzonego modelu, chcemy zamienić wielkość znaków, usunąć jakieś znaki z tekstu itp.).

Przetestowanie działania serializera możemy również przeprowadzić z poziomu shella Django. Przykład kolejnych operacji poniżej.

_**Listing 2**_
```python

# uruchamiamy shell poleceniem: python manage.py shell

from posts.models import Topic
from posts.serializers import TopicSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser

# 1. stworzenie nowej instancji klasy Topic (opcjonalne, mamy panel admin do tego również)
topic = Topic(name="DRF",category_id=1)       
topic.save()
topic.id
# 9

# 2. inicjalizacja serializera
serializer = TopicSerializer(topic)
serializer.data
# output - natywny typ danych Pythona (dictionary)
{'id': 9, 'name': 'DRF', 'category': 1, 'created': '2025-03-02T09:42:38.284465Z'}


# 3. serializacja danych do formatu JSON
content = JSONRenderer().render(serializer.data)
content

# output
b'{"id":9,"name":"DRF","category":1,"created":"2025-03-02T09:42:38.284465Z"}'

# w takiej formie możemy przesłać obiekt (lub cały graf obiektów) przez sieć i po "drugiej stronie" dokonać deserializacji odtwarzając graf i stan obiektów

import io

stream = io.BytesIO(content)
data = JSONParser().parse(stream)

# tworzymy obiekt dedykowanego serializera i przekazujemy sparsowane dane
deserializer = TopicSerializer(data=data)
# sprawdzamy, czy dane przechodzą walidację (aktualnie tylko domyślna walidacja, dedykowana zostanie przedstawiona na kolejnych zajęciach)
deserializer.is_valid()
# output
# True

# aby pokazać jak wygląda sytuacja, w której jednak pojawiają się błędy ręcznie
# zmodyfikujemy wartość category w data na None, chociaż w modelu jest wymagane
data['category'] = None
# ponownie musimy zainicjalizować serializer
deserializer = TopicSerializer(data=data)
deserializer.is_valid()
# output
# False

deserializer.errors
# output
# {'category': [ErrorDetail(string='This field may not be null.', code='null')]}

# w przypadku serializerów pisanych "ręcznie" bez automatycznego mapowania pól
# z klasy modelu określamy wymagalność pola (i pozostałe jego cechy) raz jeszcze
# Aby w serializerze określić brak wymagalności dla pola ustawiamy atrybut
# allow_null=True


# aby upewnić się w jaki sposób wyglądają pola wczytanego serializera/deserializera, możemy wywołać zmienną deserializer.fields, aby wyświetlić te dane
deserializer.fields
# output
# {'id': IntegerField(read_only=True), 'name': CharField(required=True), 'category': PrimaryKeyRelatedField(queryset=<QuerySet [<Category: Category object (1)>, <Category: Category object (2)>]>), 'created': DateTimeField()}

# lub
repr(deserializer)

# naprawiamy błąd z kategorią
data['category'] = 1
deserializer = TopicSerializer(data=data)
deserializer.is_valid()
# output
# True


# możemy sprawdzić jak wyglądają dane obiektów po deserializacji i walidacji
deserializer.validated_data
# output
# {'name': 'DRF', 'category': <Category: Category object (1)>, 'created': datetime.datetime(2025, 3, 2, 9, 42, 38, 284465, tzinfo=zoneinfo.ZoneInfo(key='UTC'))}

# oraz utrwalamy dane
deserializer.save()
# sprawdzamy m.in. przyznane id
deserializer.data
# output
# {'id': 10, 'name': 'DRF', 'category': 1, 'created': '2025-03-02T10:02:09.563355Z'}
```


W przykładzie powyżej widać sporo nadmiarowej pracy w stosunku do zdefiniowanych wcześniej modeli (możemy oczywiście chcieć serializować również inne obiekty niż modele z naszego projektu) i na pewno pojawiła się refleksja  - "Czy można wykorzystać jakąś część kodu z klas modeli?". Otóż można, wykorzystując klasę `ModelSerializer` z `Django Rest Framework`.


**Przykład klasy do serializacji danych (dziedziczącej po klasie ModelSerializer)**

Dokumentacja: https://www.django-rest-framework.org/api-guide/serializers/#modelserializer

_**Listing 3**_
```python
class TopicModelSerializer(serializers.ModelSerializer):
    class Meta:
        # musimy wskazać klasę modelu
        model = Topic
        # definiując poniższe pole możemy określić listę właściwości modelu,
        # które chcemy serializować
        fields = ['id', 'name', 'created', 'category']
        # definicja pola modelu tylko do odczytu
        read_only_fields = ['id']
```

Powyższa klasa serializera wykorzystuje wszystkie własności pól z klasy modelu, co znacznie zmniejsza ilość powielanego kodu i redukuje czas i ilość pracy niezbędny do dokonania zmian walidacji pól modelu. Te cechy zostaną pobrane do serializera z definicji modelu, więc nie musimy ich przechowywać w dwóch miejscach.

Testowanie kodu odbywa się adekwatnie do przykładu z listingu nr 1.
