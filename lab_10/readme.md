# Projektowanie Aplikacji WWW, semestr 2024Z

## Lab 10 - Uprawnienia w aplikacji Django oraz w Django Rest Framework.

> **Materiały uzupełniające:**
> * Ciekawy artykuł o uprawnieniach dla Django: https://realpython.com/manage-users-in-django-admin/ (UWAGA: linki odnoszą do dokumentacji dla wersi 2.x Django)
> * Django i uprawnienia: https://docs.djangoproject.com/pl/4.2/topics/auth/default/#permissions-and-authorization


## 1. Uprawnienia, a panel administracyjny Django.

Django posiada wbudowany moduł `auth`, dzięki któremu dostarczany jest mechanizm uwierzytelniania i autoryzacji dla użytkowników aplikacji. Jeżeli aplikacje `django.contrib.admin` oraz `django.contrib.auth`  są zainstalowane to w ramach panelu administracyjnego możliwe jest tworzenie użytkowników, grup i przypisywanie im uprawnień.

Ogólna zasada zarządzania uprawnieniami jest taka, iż powinniśmy uprawnienia przypisywać do grup, a nie bezpośrednio do użytkowników. To pozwala na tworzenie różnych poziomów uprawnień (możemy przypisać użytkownika do wielu grup) i zdecydowanie łatwiejszego nimi zarządzania (pojawienie się nowego użytkownika danego działu nie wymaga kopiowania pojedynczych uprawnień tylko przypisania go do odpowiedniej grupy).

Domyślne uprawnienia dla modeli w panelu administracyjnym, będą kontrolowane przez ten panel dla użytkowników w zespole (atrybut użytkownika), którym ten atrybut umożliwia zalogowanie się w panelu. Pozostałe, stworzone przez nas uprawnienia można przypisywać poprzez panel, ale nie zadziałają z automatu i ich reguły muszą być określone na poziomie np. modeli administracyjnych (`ModelAdmin`). Właściwe wykorzystanie tych właściwości i wykorzystanie własnej implementacji modelu użytkownika (przesłonięcie wbudowanego modelu `User`) pozwala zbudować całkiem funkcjonalną aplikację opartą o Django admin.

## 2. Uprawnienia a modele Django.

Django posiada wbudowany mechanizm podstawowych uprawnień dla każdego modelu, który w aplikacji zostanie utworzony. Dla każdego modelu zostaną utworzone cztery prawa:

* `add` - pozwala na stworzenie nowej instancji modelu,
* `delete` - pozwala na usunięcie instancji modelu,
* `change` - pozwala na aktualizację instancji modelu,
* `view` - pozwala na wyświetlenie instancji obiektu.

Aby całość funkcjonowała poprawnie w ekosystemie projektu nazwy uprawnień nadawane są wg. konwencji:
`<nazwa_aplikacji>.<akcja>_<nazwa_modelu>` np. `blog.change_topic`.

> **Każdy superuser posiada wszystkie prawa, chociaż bliższe prawdy jest stwierdzenie, że tak na prawdę te uprawnienia nie są dla niego sprawdzane w momencie użycia metody `has_perm(uprawnienie)`, które dla tekiego uzytkownika zawsze zwraca wartość `True`.**


Samo istnienie uprawnień nie powoduje, że w każdym miejscu aplikacji działa mechanizm ich weryfikacji, jeżeli na danym modelu odbywa się jedna z operacji CRUD. Jedynym miejscem, w którym się to odbywa jest panel administracyjny.

W każdym innym miejscu musimy zaimplementować metody sprawdzające posiadanie przez użytkownika wykonującego daną akcję stosownych uprawnień.

**_Listing 1_**  
```python
from django.core.exceptions import PermissionDenied

def category_view(request, pk):
    if not request.user.has_perm('blog.view_category'):
        raise PermissionDenied()
    try:
        category = Category.objects.get(pk=pk)
        return HttpResponse(f"Nazwa kategorii: {category.name}")
    except Category.DoesNotExist:
        return HttpResponse(f"W bazie nie ma kategorii o id={pk}.")
```

Można wykorzystać również dekoratory na wzór:

**_Listing 2_**

```python
from django.contrib.auth.decorators import permission_required

@permission_required('ankiety.view_person')
def category_view(request, pk):
    try:
        category = Category.objects.get(pk=pk)
        return HttpResponse(f"Nazwa kategorii: {category.name}")
    except Category.DoesNotExist:
        return HttpResponse(f"W bazie nie ma kategorii o id={pk}.")
```

Oprócz wbudowanych uprawnień dla modeli możliwe jest również zdefiniowanie dodatkowych uprawnień w klasie wewnętrznej `Meta` każdego modelu.

**_Listing 3_**

```python
class Post(models.Model):
    ...
    class Meta:
        permissions = [
            ("change_post_creator", "Pozwala przypisać innego usera jako autora postu."),
            ("change_assign_to_topic", "Pozwala przypisać post do innego topiku."),
        ]
```
Aby takie uprawnienia pojawiły się w panelu administracyjnym i było możliwe ich wykorzystanie w projekcie należy wykonać proces migracji, gdyż funkcja tworząca uprawnienia jest powiązana z sygnałem `post_migrate`.
Implementacja logiki tych uprawnień spoczywa w całości na programiście.

Możliwe jest również weryfikowanie uprawnień na poziomie szablonów widoków Django (jeżeli to jest nasza docelowa technologia wizualizacji dla projektu) odwołując się do zmiennej `perms` w szablonie.

**_Listing 4_**
```python
{% if perms.blog.delete_post %}
<button>Delete</button>
{% endif %}
```

## 3.Uprawnienia a Django Rest Framework

Django Rest Framework dostarcza kilka wbudowanych uprawnień, z których część została przedstawiona na poprzednich zajęciach (np. `IsAuthenticated`).

Pozostałe zostały opisane tutaj: https://www.django-rest-framework.org/api-guide/permissions/#api-reference

### 1. **`AllowAny`**
Daje dostęp wszystkim użytkownikom, niezależnie od tego, czy są uwierzytelnieni.

**Przykład:**
```python
from rest_framework.permissions import AllowAny
from rest_framework.views import APIView
from rest_framework.response import Response

class PublicView(APIView):
    permission_classes = [AllowAny]

    def get(self, request):
        return Response({"message": "Accessible by anyone!"})
```

### 2. **`IsAuthenticated`**
Pozwala na dostęp tylko użytkownikom zalogowanym.

**Przykład:**
```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class PrivateView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({"message": f"Hello, {request.user.username}!"})
```

### 3. **`IsAdminUser`**
Pozwala na dostęp tylko użytkownikom, którzy mają flagę `is_staff=True`.

**Przykład:**
```python
from rest_framework.permissions import IsAdminUser
from rest_framework.views import APIView
from rest_framework.response import Response

class AdminOnlyView(APIView):
    permission_classes = [IsAdminUser]

    def get(self, request):
        return Response({"message": "Accessible only by admin users."})
```

### 4. **`IsAuthenticatedOrReadOnly`**
Pozwala na pełny dostęp użytkownikom uwierzytelnionym, ale użytkownicy niezalogowani mają tylko dostęp do odczytu (np. metody `GET`, `HEAD`, `OPTIONS`).

**Przykład:**
```python
from rest_framework.permissions import IsAuthenticatedOrReadOnly
from rest_framework.views import APIView
from rest_framework.response import Response

class AuthenticatedOrReadOnlyView(APIView):
    permission_classes = [IsAuthenticatedOrReadOnly]

    def get(self, request):
        return Response({"message": "Anyone can read this."})

    def post(self, request):
        return Response({"message": f"Hello, {request.user.username}, your data has been saved."})
```

### 5. **`DjangoModelPermissions`**
Dostęp oparty na uprawnieniach przypisanych w modelach Django. Użytkownik musi być uwierzytelniony i mieć przypisane odpowiednie uprawnienia (np. `add`, `change`, `delete`, `view`).

**Przykład:**
```python
from rest_framework.permissions import DjangoModelPermissions
from rest_framework.viewsets import ModelViewSet
from myapp.models import MyModel
from myapp.serializers import MyModelSerializer

class MyModelViewSet(ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer
    permission_classes = [DjangoModelPermissions]
```

### 6. **`DjangoModelPermissionsOrAnonReadOnly`**
Rozszerzenie `DjangoModelPermissions`. Użytkownicy anonimowi mają dostęp tylko do odczytu.

**Przykład:**
```python
from rest_framework.permissions import DjangoModelPermissionsOrAnonReadOnly
from rest_framework.viewsets import ModelViewSet
from myapp.models import MyModel
from myapp.serializers import MyModelSerializer

class MyModelViewSet(ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer
    permission_classes = [DjangoModelPermissionsOrAnonReadOnly]
```

### 7. **`DjangoObjectPermissions`**
Kontroluje dostęp do poszczególnych obiektów za pomocą uprawnień przypisanych do nich.

**Przykład:**
```python
from rest_framework.permissions import DjangoObjectPermissions
from rest_framework.viewsets import ModelViewSet
from myapp.models import MyModel
from myapp.serializers import MyModelSerializer

class MyModelViewSet(ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer
    permission_classes = [DjangoObjectPermissions]
```

> **Uwaga:** Należy włączyć `django-guardian` lub podobny system do obsługi uprawnień obiektowych.

### 8. **Custom Permissions (Własne uprawnienia)**
Możemy definiować własne klasy uprawnień, implementując metodę `has_permission` i/lub `has_object_permission`.

**Przykład:**
```python
from rest_framework.permissions import BasePermission
from rest_framework.views import APIView
from rest_framework.response import Response

class IsOwner(BasePermission):
    def has_object_permission(self, request, view, obj):
        return obj.owner == request.user

class OwnerOnlyView(APIView):
    permission_classes = [IsOwner]

    def get(self, request, obj):
        return Response({"message": "You are the owner of this object."})
```


**Wykorzystanie istniejących uprawnień modeli w DRF.**

Dokumentacja: https://www.django-rest-framework.org/api-guide/permissions/#djangomodelpermissions

<del>Z racji tego, że aktualnie nie ma oficjalnego rozwiązania pozwalającego na wykorzystanie `DjangoModelPermissions` w przypadku wykorzystania widoków funkcyjnych (to te, gdzie używamy dekoratora `@api_view`) należy niezbędną logikę widoków dla DRF przepisać na class based views.</del>

> **UPDATE:** Można jednak obejść problem wykorzystania `DjangoModelPermissions` w widokach funkcyjnych ustawiając ten mechanizm jako główny w pliku `settings.py`, a następnie w widokach przesłaniając go innymi wartościami dekoratora `@permission_classes`

Wpis w `settings.py`:
>**UWAGA! - tu użyta klasa rozszerzająca DjangoModelPermissions - patrz listing 6).**
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': (
        'blog.permissions.CustomDjangoModelPermissions',
    )
}
```
**_Listing 4_**
```python
# i teraz widok funkcyjny może wyglądać tak
@api_view(['GET'])
@permission_classes([IsAuthenticated, CustomDjangoModelPermissions])
@authentication_classes([BearerTokenAuthentication])
def topic_detail(request, pk):

    """
    :param request: obiekt DRF Request
    :param pk: id obiektu Topic
    :return: Response (with status and/or object/s data)
    """

    if request.method == 'GET':
        try:
            topic = Topic.objects.get(pk=pk)
        except Topic.DoesNotExist:
            return Response(status=status.HTTP_404_NOT_FOUND)

        serializer = TopicModelSerializer(topic)
        return Response(serializer.data)

```

Przykład dla modelu Category poniżej.

**_Listing 5**

```python
class CategoryDetail(APIView):
    authentication_classes = [BearerTokenAuthentication]
    permission_classes = [IsAuthenticated, CustomDjangoModelPermissions]

    # dodanie tej metody lub pola klasy o nazwie queryset jest niezbędne
    # aby DjangoModelPermissions działało poprawnie (stosowny błąd w oknie konsoli
    # nam o tym przypomni)
    def get_queryset(self):
        return Category.objects.all()

    def get_object(self, pk):
        try:
            return Category.objects.get(pk=pk)
        except Category.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        category = self.get_object(pk)
        serializer = CategoryModelSerializer(category)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        category = self.get_object(pk)
        serializer = CategoryModelSerializer(category, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        team = self.get_object(pk)
        team.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

W dokumentacji znajdziemy informacje o mapowaniu żądań na uprawnienia:
* żądania `POST` wymagają posiadania prawa `add` na modelu,
* żądania `PUT` i `PATCH` wymagają posiadania prawa `change` na modelu,
* żądania `DELETE` wymagają posiadania prawa `delete` na modelu.

A gdzie żądanie `GET`? Otóż chociaż mogłoby się wydawać logicznym, że powinno być mapowanie na uprawnienie `view` dla modelu, to tak nie jest. Można jednak rozszerzyć bazową klasę `DjangoModelPermissions` i dodać tę funkcjonalność.

**_Listing 6_**

Kod został umieszczony w pliku `permissions.py` w folderze aplikacji (nie projektu).
```python
import copy

from rest_framework import permissions


class CustomDjangoModelPermissions(permissions.DjangoModelPermissions):

    def __init__(self):
        self.perms_map = copy.deepcopy(self.perms_map)
        self.perms_map['GET'] = ['%(app_label)s.view_%(model_name)s']
```


## **Zadania**

**Zadanie 1**  
Dodaj nową grupę w panelu administracyjnym. Dodaj do tej grupy jedno wybrane uprawnienie domyślne dla modelu, który został dodany do zarządzania w panelu administracyjnym. Stwórz nowego użytkownika, który będzie "w zespole", ale nie będzie superużytkownikiem i przypisz go do tej grupy. Zaloguj się na konto stworzonego użytkownika i sprawdź czy kontrola tego uprawnienia działa poprawnie.

**Zadanie 2**  
Korzystając z przykładów z listingów 1 oraz 2 dodaj prosty widok, w logice którego sprawdź czy user posiada uprawnienie `view_osoba` i wyświetlaj odpowiednią treść. **(patrz listing 1)**

**Zadanie 3**  
Do modelu `Osoba` dodaj własne uprawnienie o nazwie `can_view_other_persons`, które dodaj do logiki z zadania 2 i jeżeli jest ono przypisane to pozwalaj wyświetlać obiekty modelu `Osoba`, których zalogowany użytkownik nie jest właścicielem. W przeciwnym wypadku nie daj takiej możliwości.

**Zadanie 4**  
Przetestuj działanie klasy `DjangoModelPermissions` (lub `CustomDjangoModelPermissions`) z DRF z różnymi prawami dostępu (GET, PUT, POST, DELETE). Pamiętaj, że użytkownikowi, który nie jest superuserem należy przypisać stosowne prawa do modelu (poprzez przypisanie ich do grupy).

