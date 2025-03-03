# Instrukcja ewentualnego uruchomienia projektu

Kod źródłowy projektów jest do wglądu i wykorzystania jako przykładowe rozwiązanie, więc do tego nie potrzeba uruchamiać projektu.
Gdyby jednak ktoś chciał, to poniżej informacje oraz instrukcja.

Z projektów zostały usnięte pliki bazy danych oraz migracje.

Instrukcja uruchmienia projektu:

1. Rozpakowujemy archiwum.
2. Otwieramy folder główny w programie, z którego korzystamy na zajęciach (Visual Studio Code).
3. Tworzymy wirtualne środowisko Pythona na potrzeby projektu (lub wskazujemy to samo co dla projektu z zajęć - jeżeli wiemy jak.).
4. Instalujemy django==4.2.19, django-debug-toolbar oraz djangorestframework narzędziem pip. (`pip install <nazwa>`)
5. Przygotowujemy migracje: `python manage.py makemigrations`, a później `python manage.py migrate`
6. Tworzymy super użytkownika, aby można było zalogować się do panelu: `python manage.py createsuperuser` i podajemy niezbędne dane.
7. Uruchamiamy serwer poleceniem `python manage.py runserver` i testujemy aplikację.