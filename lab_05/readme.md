# Aplikacje WWW

## Lab 5
---
### **1. Praca z obiektami QuerySet.**

> Dokumentacja Django QuerySet: https://docs.djangoproject.com/en/4.2/ref/models/querysets/

API dostarczane przez klasę QuerySet służy do komunikowania się z poziomu aplikacji z bazą danych, która jest skonfigurowana w projekcie Django. Ten mechanizm jest warstwą mechanizmu ORM (Object Relational Mapping) czyli mapowania obiektowo-relacyjnego, które umożliwia konwersję obiektów modeli na tabele w bazie danych. API QuerySet umożliwia pobieranie, tworzenie, aktualizację oraz usuwanie obiektów z bazy danych. Operacje te wykonaliśmy już niejawnie wcześniej zarządzając obiektami z poziomu przeglądarki w formularzach dostarczonych przez panel administracyjny Django, który z API QuerySet korzysta.

Obiekty QuerySet, które stworzymy będą w ostatniej fazie tłumaczyły nasz kod na odpowiedni język zapytań SQL w zależności od silnika baz danych skonfigurowanego w naszym projekcie. Jest to więc wygodna warstwa pośrednia, która nie wymusza na nas znajomości danego dialektu SQL, ale co bardziej istotne, pozwala na zmianę silnika bazy danych w projekcie bez konieczności przepisywania zapytań! Wystarczy, że przygotujemy zrzut danych z aktualnej bazy do ogólnego formatu (np. JSON), zmienimy konfigurację w pliku settings.py na nowy silnik, wykonany migracje aby odtworzyć strukturę w nowej bazie i załadujemy wcześniej zrzucone dane.

#### **1.1 Wykorzystanie Django shell do pracy z API QuerySet.**

W obecnym stanie projektu najwygodniej będzie nam korzystać z QuerySet poprzez Django shell.
Uruchamiamy je poprzez wpisanie w terminalu poniższego polecenia (pamiętaj o ustawieniu wcześniej odpowiedniej ścieżki, tak aby plik manage.py znajdował się w folderze, z którego poniższe polecenie jest wywoływane)

_Listing 1_
```python
python manage.py shell
```

Powinniśmy zobaczyć w konsoli znak zachęty charakterystyczny dla interaktywnego Pythona czyli **>>>**.

W zależności od klas modeli, które posiadamy oraz chcielibyśmy wykorzystać w poniższych zapytaniach powinniśmy je najpierw zaimportować, tak aby interpreter Pythona "widział" je w aktualnej sesji i aby QuerySet "wiedział" jakie cechy (pola) tych obiektów ma do dyspozycji.

Przykładowy import klas modeli `Post`, `Topic` oraz `Category`.

_Listing 2_
```python
# zakładając, że nasza aplikacji, z której chcemy zaimportować modele nazywa się posts
from posts.models import Post, Topic, Category
```

#### **1.2 Pobieranie danych.**

Pobieranie danych może mieć wiele postaci zgodnie z tym co możemy wykonać tworząc zapytania wybierające `SELECT` zawarte w języku SQL, który to zostanie użyty aby finalnie nasze obiekty pobrać z bazy. Można się więc domyślać, że aby zapewnić całe spektrum dostępnych zapytań SQL poprzez obiekt QuerySet, musi on dostarczać wielu metod. W związku z tym w tej dokumentacji zostaną przedstawione tylko wybrane z nich, a do pełnej listy jego możliwości odsyłam do dokumentacji podanej na początku tego dokumentu.

Zaczniemy od najprostrzego zapytania, które pobierze listę wszystkich obiektów danego typu (modelu) z naszej bazy. Warto tutaj zaznaczyć, że nie będziemy tworzyć obiektów klasy QuerySet w sposób bezpośredni czyli tworząc jawnie obiekt tego typu, a poprzez wywołanie metod naszych modeli, które zostały dostarczone przez mechanizm dziedziczenia z klasy `models.Model` API Django (zobacz plik models.py w swojej aplikacji).

_Listing 3_
```python
>>> Post.objects.all()
# w moim przypadku wyświetlone zostało
<QuerySet [<Post: Post object (1)>]>
```

To co widzimy to obiekt QuerySet, który zwrócił dwa obiekty typu `Post` jako wynik tej operacji. To co widzimy tutaj jest efektem wywołania metody `__str__` obiektu `Post`, która w naszym przypadku nie została jawnie zadeklarowana, więc jest odziedziczona po klasie `Model` z ekosystemu Django, stąd widzimy tekst `Post object (1)`.

Możemy przesłonić metodę `__str__` w klasie `Post` i osiągniemy inny efekt. Przykład poniżej:

_Listing 4_
``` python
def __str__(self):
    return self.content[:20]
```

Oznacza to, że za każdym razem gdy będziemy chcieli uzyskać obiekt klasy `Post` w postaci łańcucha znaków metoda `__str__()` zostanie wywołana i zwróci nam 20 pierwszych znaków pola `content` tego konkretnego obiektu.

**Filtrowanie obiektów**

Filtrowanie listy obiektów polega na określeniu wartości pól, które nas interesują spośród wszystkich obiektów w bazie. Poniżej kilka przykładów dla obiektów typu Post. Poniższy przykład wykorzystania metody `get()` nie zwraca obiektu typu QuerySet, tylko obiekt konkretnego typu (lub rzuca (ang. raise) wyjątek, jeżeli nie ma obiektów spełniających kryteria).

_Listing 5_
```python
# pobranie obiektu Post, który posiada atrybut id równy 1
>>> Post.objects.get(id=1)  
<QuerySet [<Post: Abracadabra>]>

# jeżeli obiektów spełniających kryteria nie ma, możemy zobaczyć poniższy wynik
>>> Post.objects.get(id=10) 
Traceback (most recent call last):
  File "<console>", line 1, in <module>
  File "c:\Users\Krzysztof\__projects\__is_app_www_proj\.venv\Lib\site-packages\django\db\models\manager.py", line 87, in manager_method
    return getattr(self.get_queryset(), name)(*args, **kwargs)
           ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^
  File "c:\Users\Krzysztof\__projects\__is_app_www_proj\.venv\Lib\site-packages\django\db\models\query.py", line 637, in get
    raise self.model.DoesNotExist(
        "%s matching query does not exist." % self.model._meta.object_name
    )
posts.models.Post.DoesNotExist: Post matching query does not exist.

# zgłoszony został wyjątek DoesNotExist, czyli takiego obiektu typu Post nie ma w naszej bazie
```

Metoda, dzięki której możemy uzyskać listę obiektów (tu obiekt typu QuerySet), które spełniają zadane kryteria jest metoda `filter()`. Poniżej kilka przykładów.

_Listing 6_
```python
# szukamy obiektów Post dla danego użytkownika (po id)
>>> Post.objects.filter(created_by=1)
<QuerySet [<Post: Abracadabra>]>

# szukamy obiektów, dla których id użytkownika jest > 1
>>> Post.objects.filter(created_by__gt=1)     
<QuerySet []>

# i tutaj widzimy dość specyficzny sposób definiowania niektórych warunków filtrowania
# gdzie do pola o nazwie created_by dodaliśmy część __gt i ponownie użyliśmy znaku = do określenia wartości
# do porównania
```

Tutaj na chwilę się zatrzymamy. A co gdybysmy chcieli wyświetlić trochę więcej szczegółów na temat obiektu/obiektów, które te zapytania nam zwracają?
Możemy zrobić z tymi obiektami dokładnie to samo co z każdym innym obiektem w Pythonie. Jeżeli posiada pola, możemy się do nich odwołać, jeżeli posiadają metody możemy je wywołać. Wykorzystajmy najpierw wcześniejszy przykład z listą wszystkich obiektów, czyli obiekt typu QuerySet z listą oiektów typu Post. Możemy dostać się do poszczególnych obiektów tej listy poprzez indeksowanie lub cięcie (ang. slicing).

_Listing 7_
```python
>>> lista = Post.objects.all()
>>> lista 
<QuerySet [<Post: Abracadabra>]>
>>> lista[0]
<Post: Abracadabra>
>>> lista[0].content
'Abracadabra'
>>> lista[0].created_by
<User: kropiak>
>>> lista[0].topic  
<Topic: Topic object (1)>
>>> lista[0].topic.name
'Powitanie'

# lub aby wyświetlić wartość wszystkich pól, możemy użyć metody values()
>>> lista.values()  
<QuerySet [{'id': 1, 'topic_id': 1, 'content': 'Abracadabra', 'created_at': datetime.datetime(2025, 2, 
23, 10, 2, 5, 90839, tzinfo=datetime.timezone.utc), 'modified_at': datetime.datetime(2025, 2, 23, 10, 2, 5, 90859, tzinfo=datetime.timezone.utc), 'created_by_id': 1}]>
```

A teraz zobaczmy jakie zapytanie jest faktycznie wysyłane do silnika bazy danych, aby zrealizować nasze wywołanie QyerySet API.

_Listing 8_
```python
>>> print(lista.query)   
SELECT "posts_post"."id", "posts_post"."topic_id", "posts_post"."content", "posts_post"."created_at", "posts_post"."modified_at", "posts_post"."created_by_id" FROM "posts_post"
```

Możemy wypisać SQL dla każdego zapytania z tego API, również po to, aby sprawdzić lub lepiej zrozumieć co dana funkcja faktycznie spowoduje w kontekście zapytania do bazy.

Inne przykłady wykorzystania metody `filter()`.

_Listing 9_
```python
# pole topic.name rozpoczyna się od litery P
>>> Topic.objects.filter(name__startswith='P')  
<QuerySet [<Topic: Topic object (1)>]>

# istnieje również metoda, dla której nie będzie miało znaczenia czy to będzie P czy p
# tzw. case insensitive - niewrażliwa na wielkość znaków
>>> Topic.objects.filter(name__istartswith='P')  
<QuerySet [<Topic: Topic object (1)>]>

# obiekty, których pole name zawiera literę 'o'
>>> Topic.objects.filter(name__contains='o') 
<QuerySet [<Topic: Topic object (1)>]>

# lub tylko wybrane pola
>>> lista.values('created_by', 'created_at')  
<QuerySet [{'created_by': 1, 'created_at': datetime.datetime(2025, 2, 23, 10, 2, 5, 90839, tzinfo=datetime.timezone.utc)}]>
```

Czasem będzie nas interesowała lista obiektów, dla której łatwiej jest zdefiniować warunek, którego nie powinny spełniać niż ten, który spełniony być powinien. Możemy wtedy wykorzystać metodę `exclude()` jak w przykładzie poniżej.

_Listing 10_
```python
# wszystkie obiekty Post gdzie id topic jest różne od 2
>>> Post.objects.exclude(topic_id=2) 
<QuerySet [<Post: Abracadabra>]>

# wszystkie obiekty Topic gdzie name nie rozpoczyna się od S
>>> Topic.objects.exclude(name__startswith='S') 
<QuerySet [<Topic: Topic object (1)>, <Topic: Topic object (2)>]>

# ta metoda zezwala na dodanie kolejnych parametrów w jednym wywołaniu, ale
# efekt działania może być na początku nieoczywisty jeżeli nie wczytamy się
# w dokumentację, przykład
>>> print(Topic.objects.exclude(name__startswith='S', category_id=1).query) 
SELECT "posts_topic"."id", "posts_topic"."name", "posts_topic"."category_id", "posts_topic"."created" FROM "posts_topic" WHERE NOT ("posts_topic"."category_id" = 1 AND "posts_topic"."name" LIKE S% ESCAPE '\')

# widać, że pierwszy atrybut exclude zostanie wyłączony z wyników, ale już kolejny będzie warunkiem
# którego będziemy w bazie szukać, aby dodać kolejne wykluczenie należy
# wsposób wywołania łańcuchowego exclude dodawać kolejne warunki
>>> Topic.objects.exclude(name__startswith='S').exclude(category_id=1) 
<QuerySet [<Topic: Topic object (2)>]>
```

Istnieją jeszcze inne metody i parametry filtrowania wartości, ale odsyłam po szczegóły do oficjalnej dokumentacji.

#### **1.3 Tworzenie, edycja i usuwanie obiektów poprzez API QuerySet**

Oprócz pobierania daych z bazy poprzez wywołania API QuerySet możemy rówież takie obiekty zapisywać, modyfikować oraz usuwać.
Zacznijmy od przykładu stworzenia nowego obiektu typu Person, który możemy zainicjalizowac na kilka sposóbów, ale jego utrwalenie, czyli zapisanie w bazie odbywa się poprzez wywołanie na tym obiekcie metody `save()`.

_Listing 11_
```python
new_topic = Topic()
>>> new_topic.name = 'Różne'        
>>> new_topic.save()
# ...
# dość długi komunikat o błędzie
django.db.utils.IntegrityError: NOT NULL constraint failed: posts_topic.category_id
>>> new_topic.category_id = 1
>>> new_topic.save()

# pobranie utworzonej automatycznie wartości klucza głównego (stąd zmienna pk -> primary key)
# której faktyczna nazwa (jeżeli w pliku models.py nie zdefiniowaliśmy inaczej) to id
>>> Topic.objects.get(name='Różne').pk           
3
>>> Topic.objects.get(name='Różne').id
3
```

Oprócz tworzenie obiektów za pośrednictwem `QuerySet` możemy również je aktualizować i wykonać operację `UPDATE` na naszej bazie. Zobaczmy przykład poniżej.

_Listing 12_
```python
>>> topic_rozne = Topic.objects.get(pk=3)
>>> topic_rozne
<Topic: Topic object (3)>
>>> topic_rozne.name = 'Inne'
>>> topic_rozne.name
'Inne'
>>> topic_rozne.save()
>>> topic_rozne.id
3
>>> Topic.objects.all()
<QuerySet [<Topic: Topic object (1)>, <Topic: Topic object (2)>, <Topic: Topic object (3)>]>
```
Aktualizacja wartości pola nie powoduje dodania nowego obiektu typu do bazy co widać po tym, że po wywołaniu metody save() wartość pola id nie uległa zmianie, ale pola name już tak.

Operację update możemy wykonać również w inny sposób. Co jeżeli chcemy zaktualizować większą liczbę obiektów? Czy musimy pobierać każdy z nich i zmieniać wartość? Co jeżeli dla wielu obiektów chcemy zwienić wartość na taką samą? To byłaby bardzo żmudna i powtarzalna praca, której zapewne nie chcielibyśmy wykonywać.
Z pomocą przychodzi nam metoda `update()` klasy `QuerySet`, którą możemy wywałać na pobranym wcześniej zbiorze obiektów. Przykład poniżej.

_Listing 13_
```python
# sprawdzamy jakie kategorie są aktualnie przypisane do topiców
>>> Topic.objects.all().values('category_id')  
<QuerySet [{'category_id': 1}, {'category_id': 1}, {'category_id': 2}]>
# aktualizujemy wszystkie instancje topic, ustawiając wszystkim obiektom wartość id kategorii na 1
>>> Topic.objects.all().update(category_id=1)
# ta liczba poniżej to liczba zmodyfikowanych rekordów w bazie
3
# i na koniec sprawdzamy czy aktualizacja się powiodła
>>> Topic.objects.all().values('category_id') 
<QuerySet [{'category_id': 1}, {'category_id': 1}, {'category_id': 1}]>
```

Usuwanie obiektów działa w podobny sposób, tu używamy metody `delete()`.

_Listing 14_
```python
# wywołanie metody delete bezpośrednio na obiekcie typu Post pobranym z bazy
# pamiętajmym, że metoda get() nie wzraca obiektu typu QuerySet
>>> Topic.objects.get(pk=3).delete()
(1, {'posts.Topic': 1})
>>> Topic.objects.all()                   
<QuerySet [<Topic: Topic object (1)>, <Topic: Topic object (2)>]>

# możemy usunąć więcej obiektów wywołując delete() na obiekcie QuerySet
>>> t1 = Topic()                          
>>> t1.name = 'Programowanie'
>>> t1.category_id = 1
>>> t1.save()
>>> t2 = Topic()
>>> t2.name = 'programowanie' 
>>> t2.category_id = 1     
>>> t2.save()
>>> Topic.objects.all()
<QuerySet [<Topic: Topic object (1)>, <Topic: Topic object (2)>, <Topic: Topic object (4)>, <Topic: Topic object (5)>]>
>>> Topic.objects.filter(name__istartswith='p')
<QuerySet [<Topic: Topic object (1)>, <Topic: Topic object (4)>, <Topic: Topic object (5)>]>
>>> Topic.objects.filter(name__istartswith='p').delete()
(4, {'posts.Post': 1, 'posts.Topic': 3})

# powyższe podsumowanie świadczy o tym, że machanizm kaskadowego usuwania działa poprawnie
# usuwamy topic, do którego należy post, więc post jest również usuwany
```

#### **1.4 Inne wybrane metody API QuerySet**

Istnieją jeszcze inne metody tego API, które mogą być pomocne. Kilka przykładów zostało zaprezentowanych poniżej.

_Listing 15_
```python
# pobranie pierwszego rekordu z tabeli Topic
>>> Topic.objects.first() 
<Topic: Topic object (2)>

# pobranie ostatniego rekordu
>>> Topic.objects.last()  
<Topic: Topic object (2)>

# pobranie unikalnego zbioru wartości
>>> Category.objects.distinct()          
<QuerySet [<Category: Category object (1)>, <Category: Category object (2)>]>

# nasza baza (sqlite) nie obsługuje możliwości pobrania wartości unikalnych dla danej kolumny
>>> Topic.objects.distinct('category_id')
# komunikat
# django.db.utils.NotSupportedError: DISTINCT ON fields 
# is not supported by this database backend

# zliczenie ilości obiektów w QuerySet
>>> Category.objects.count()
2
>>> Category.objects.filter(name__startswith='O').count() 
1

# sortowanie wyników, tutaj po kolumnie name
>>> Category.objects.all().order_by('name') 
<QuerySet [<Category: Category object (2)>, <Category: Category object (1)>]>

# w naszym przypadku nie widać nazw, ale możemy je wyświetlić
>> Category.objects.all().order_by('name').values('name')
<QuerySet [{'name': 'O ludziach'}, {'name': 'Test'}]>


# a kierunek sortowania domyślny to rosnąco, możemy to zmienić dodając do nazwy
# pole, po którym sortujemy znak -
>>> Category.objects.all().order_by('-name').values('name')
<QuerySet [{'name': 'Test'}, {'name': 'O ludziach'}]>
```

#### **Zadania**

> Przed rozpoczęciem pracy zatwierdź wszystkie zmiany na bieżącym branchu, a następnie utwórz branch o nazwie `lab_5` i przełącz się na niego. Po zakończeniu pracy zatwierdź zmiany.

W swoim folderze z projektem (zapewne główny folder projektu\blog) stwórz folder o nazwie `zadania` i tam umieść plik `zadania_lab_5.md`, w którym umieść polecenia będące rozwiązaniem poniższych zadań. Przykład struktury pliku poniżej.

```markdown
# Rozwiązania zadań lab 5

## zadanie 1
```python
tu kod ...
# znak \ poniżej usuwamy, pojawił się, aby było widać źródło pliku
\``` 
## zadanie 2
...
```

**Zadanie 1**  
Uruchom Django shell tak jak w przykładach zaprezentowanych w tym labie i dodaj po 3 nowe obiekty typu `Category`, `Topic` oraz `Post`. Dla minimum jednego posta nie zdefiniuj użytkownika (powinniśmy już mieć w definicji modelu `Post` atrybut `null=True`).

**Zadanie 2**  
Wykonaj zapytanie filtrujące obiekty typu `Topic`, których nazwa rozpoczyna się od wybranej przez Ciebie litery, tak aby zwrócone były niepuste wyniki.

**Zadanie 3**  
Dla danych zwróconych w zadaniu 2 wyświetl listę wszystkich wartości tych obiektów (funkcja values()).

**Zadanie 4**  
Wykonaj pobranie wszystkich obiektów typu `Post` i zapisz je do zmiennej `posts`.

**Zadanie 5**  
Z listy stworzonej w zadaniu 4 za pomocą cięcia (slicing) wyświetl:
* pierwszy element
* ostatni element
* wszystkie elementy w odwróconej kolejności (od ostatniego do pierwszego)
* co drugi element

**Zadanie 6**  
Wyświetl liczbę obiektów typu `Post`, które nie mają przypisanego użytkownika.

**Zadanie 7**  
Wyświetl wszystkie obiekty `Topic` sortując po nazwie w porządku alfabetycznym od Z do A.

**Zadanie 8**  
Wyświetl wszystkie obiekty `Category`, których id jest mniejsze niż 2 używając metody `exclude()`.

**Zadanie 9**  
Usuń jeden wybrany topic, który ma przypisany obiekt `Post` i sprawdź czy ten post również zostal usunięty.