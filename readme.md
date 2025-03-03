# Aplikacje WWW

## 1. Cel i zakres przedmiotu

Celem przedmiotu jest przybliżenie osobie studenckiej zagadnień związanych z projektowaniem aplikacji WWW z wykorzystaniem języka Python oraz frameworka Django. W trakcie zajęć zajmiemy się implementacją backendu (czyli część aplikacji związana z implementacją funkcjonalności po stronie serwera) oraz frontendu, czyli części związanej z wizualną prezentacją aplikacji.

W trakcie zajęć osoba studencka pozna m.in. zagadnienia takie jak:
* konfiguracja bazy danych na potrzeby aplikacji
* podstawowa obsługa narzędzia Git
* tworzenie i zarządzanie modelami we frameworku Django
* migracja bazy i rozwiązywanie problemów z migracjami
* implementacja logiki biznesowej (wymagań klienta) w projekcie
* tworzenie API REST-owego na potrzeby różnych technologii frontendowych
* wykorzystanie HTML oraz CSS do tworzenia prostych szablonów stron
* widoki Django z wykorzystaniem modeli, formularzy oraz szablonów
* obsługa i implementacja autentykacji i uwierzytelniania z użyciem Django
* poznanie narzędzi zarządzania projektem Django oraz panelu administratora i jego możliwości
* wprowadzenie do Javascript i prosty frontend oparty o biblioteke vue.js


## 2. Oprogramowanie

W trakcie zajęć do pracy niezbędne będzie posiadanie:
* zainstalowanego interpretera **Pythona** w wersji 3.11 lub nowszej
* **framework Django** w wersji 4.2.*
* narzędzie IDE, preferowane **PyCharm Professional** (licencja studencka) lub wersja **Community**. Może to być również inne oprogramowanie ze wsparciem dla języka Python, np. Visual Studio Code
* narzędzie Git do zarządzania kodem projektu
* możliwe, że w zależności od konfiguracji projektu niezbędne będzie zainstalowanie i konfiguracja odpowiedniego serwera bazy danych

## 3. Warunki zaliczenia przedmiotu.

- Efektem finalnym pracy na zajęciach będzie **projekt API** stworzony w Django Rest Framework lub po uzgodnieniu w innym frameworku,
- osoby studenckie mogą dobrać się w pary lub pracować samodzielnie (w wyjątkowych przypadkach dozwolona jest praca w 3-osobowej grupie),
- Projekt osoby studenckiej będzie przechowywany w **repozytorium GitHub**, do którego prowadzący będzie miał wgląd przez cały okres trwania zajęć,
- W przypadku pracy grupowej, obie osoby muszą być dodane jako collaborator do repozytorium,
- Oceniane będą:
   - Systematyczność commitów (oraz ich [sens](https://medium.com/@steveamaza/how-to-write-a-proper-git-commit-message-e028865e5791)),
   - Jakość kodu programistycznego,
   - Implementacja zagadnień przedstawionych na zajęciach,
   - Poziom skomplikowania projektu (np. ilość modeli, działanie endpointów, itp.)
- Proponowany temat API (co te API będzie robić) jest dowolny i będzie przedstawiony prowadzącemu na drugich zajęciach.
- Prowadzący rezerwuje możliwość przeprowadzenia kolokwium w przypadku braku pracy na zajęciach przez grupę osób studenckich.

## 4. Wymagania projektu.

### Zakres projektu zaliczeniowego: 

#### Opcja 1

**Projekt Django REST API**

1.	**Modele**:
* 4-5 modeli adekwatnych do tematu projektu

2.	**Uwierzytelnianie**
* Część endpointów powinna być zabezpieczona przed dostępem dla niezalogowanych użytkowników. (patrz przykładowa aplikacja)

3.	**Endpointy:**
* CRUD - (odczyt, dodanie, edycja i usunięcie obiektu)
* Minimum 2 endpointy, które wyjdą poza schemat CRUD, np. zestawienie miesięczne zamówień, lista wypożyczeń dla użytkownika, lista towarów zaczynających się od itp. (patrz przykładowa aplikacja)

#### Opcja 2
**Projekt Django + HTML + CSS**

1.	**Modele**:
* 4-5 modeli adekwatnych do tematu projektu

2. **Widoki**:
* lista obiektów danego modelu (dla wszystkich dodanych modeli w models.py)
* szczegóły dla danej instancji modelu (detail view)
* z możliwością edycji obiektu oraz usunięcia obiektu (patrz przykładowa aplikacja)
* minimum 2 widoki dodatkowe: np. zestawienie zamówień za dany rok/miesiąc, lista produktów dla kategorii, postów danego użytkownika, itp. (patrz przykładowa aplikacja)

3. **Inne wymagania**:
* dodany szablon strony (Django templates, patrz lab 6)
* dodany plik ze stylami (raczej prosty) do tego szablonu (patrz przykładowa aplikacja)
