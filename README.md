<p align="center"><a href="README.md"><strong>Polski</strong></a> | <a href="README.en.md">English</a></p>

<p align="center">
  <img src="assets/login.png" alt="Int3gro.pl" width="600"/>
</p>

<h1 align="center">Int3gro.pl</h1>

<h3 align="center">Integrator Allegro</h3>

<p align="center">
  <em>Int3gro.pl łączy katalogi hurtowni części samochodowych z Allegro: importuje dane, scala duplikaty, wystawia oferty i przekazuje zamówienia do właściwego dostawcy.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Next.js-16-000000?style=for-the-badge&logo=next.js" alt="Next.js 16"/>
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react" alt="React 19"/>
  <img src="https://img.shields.io/badge/TypeScript-5-3178C6?style=for-the-badge&logo=typescript" alt="TypeScript"/>
  <img src="https://img.shields.io/badge/Supabase-PostgreSQL-3FCF8E?style=for-the-badge&logo=supabase" alt="Supabase"/>
  <img src="https://img.shields.io/badge/Tailwind_CSS-3.4-06B6D4?style=for-the-badge&logo=tailwindcss" alt="Tailwind CSS"/>
  <img src="https://img.shields.io/badge/Netlify-Deploy-00C7B7?style=for-the-badge&logo=netlify" alt="Netlify"/>
</p>

---

## Spis treści

- [O projekcie](#o-projekcie)
- [Zrzuty ekranu](#zrzuty-ekranu)
- [Kod źródłowy](#kod-źródłowy)
- [Stack](#stack)
- [Funkcje](#funkcje)
- [Architektura](#architektura)
- [Statystyki](#statystyki)
- [Moja rola](#moja-rola)
- [Kontakt](#kontakt)

---

## O projekcie

**Int3gro.pl** to platforma integracyjna na produkcji. Synchronizuje katalogi hurtowni części samochodowych z **Allegro**, największym marketplace w Polsce.

Pobiera dane z hurtowni ELIT, Inter Cars, Inter Team, Florimex i Mekonomen, scala produkty i usuwa duplikaty, wystawia oferty na Allegro i nimi zarządza, a zamówienia klientów przekazuje do właściwego dostawcy.

Panel działa na [int3gro.pl](https://int3gro.pl), na Netlify. Pipeline w Pythonie na Linode co noc importuje katalogi, synchronizuje stany magazynowe i automatyzuje cykl życia ofert. Supabase obsługuje bazę danych, Edge Functions, RLS i zaplanowane zadania.

---

## Zrzuty ekranu

| Logowanie | Dashboard |
|:---:|:---:|
| ![Logowanie](assets/login.png) | ![Dashboard](assets/dashboard.png) |

| Katalog produktów | Generator ofert |
|:---:|:---:|
| ![Wyszukiwanie w katalogu](assets/katalog-search.png) | ![Generator ofert](assets/generator.png) |

| Drafty | Nawigacja |
|:---:|:---:|
| ![Drafty](assets/drafty.png) | ![Nawigacja](assets/navigation.png) |

> **Uwaga:** Wszystkie dane na zrzutach są fikcyjne i powstały wyłącznie na potrzeby demonstracji. Nazwy produktów, ceny, stany magazynowe i szczegóły zamówień są wymyślone i nie odzwierciedlają rekordów żadnego dostawcy ani marketplace.

---

## Kod źródłowy

> **Kod źródłowy jest prywatny.**

---

## Stack

### Panel

```
Next.js 16 + React 19      // App Router, logowanie po stronie serwera, dev na Turbopacku
TypeScript 5               // typy w całym projekcie
Tailwind CSS 3.4           // klasy przy elementach i Framer Motion
TanStack Table 8           // tabele produktów i zamówień, dane brane z serwera
```

### Baza i funkcje

```
Supabase PostgreSQL        // wspólny widok produktów, uprawnienia wierszy (RLS), pg_cron
19 Edge Functions          // API Allegro, składanie zamówień, cykl życia ofert
pg_trgm                    // szybkie szukanie pełnym tekstem po katalogach
Netlify                    // panel int3gro.pl na produkcji
```

### Pipeline

```
Python 3 na Linode         // co noc odbiór FTP od hurtowni i synchronizacja z Allegro
23 skrypty                 // deduplikacja, nieaktualne oferty, zakresy cen, stany magazynowe
Allegro REST API           // oferty, zamówienia, kategorie, realizacja
API hurtowni               // ELIT po SOAP, Inter Cars po OAuth, Inter Team po mTLS
```

---

## Funkcje

### Dashboard

- **Karty KPI** - Na pulpicie widać aktywne oferty, szkice, zamówienia i kolejkę wznawiania.
- **Tabele zamówień** - W tabeli są zamówienia z Allegro wraz ze statusem routingu do dostawcy.
- **Nocne skrypty** - Widać status skryptów z nocnych przebiegów.
- **Wznawianie ofert** - Zakończone oferty, które mają stan, wracają na Allegro hurtem.

### Katalog produktów

- **Drzewo kategorii** - Kategorie Allegro są ułożone w drzewo. Przy każdej widać, ile jest w niej produktów.
- **Szukanie** - Szukanie idzie trigramami (pg_trgm) po scalonych katalogach hurtowni.
- **Strony katalogu** - Tabela mieści tysiące produktów, ze stanem i z informacją, czy oferta jest wystawiona. Długi wynik jest pocięty na strony.

### Generator ofert

- **Wystawianie z kategorii** - Wybiera się produkty z kategorii i ustawia marżę osobno dla każdej.
- **Kalkulator ceny** - Liczy minimalną i maksymalną cenę sprzedaży oraz zasady prowizji.
- **Wystawianie hurtem** - Na Allegro idzie naraz wiele ofert. Wystawienie przechodzi przez Edge Functions.

### Szkice

- **Podgląd przed publikacją** - Są tu produkty gotowe do publikacji, z podglądem ceny.
- **Cena z Allegro** - Panel pyta Allegro o cenę, zanim oferta zostanie wystawiona. Odpowiedź przychodzi od razu.

### Nocny pipeline

- **Odbiór katalogów** - Co noc przychodzą katalogi ELIT, Inter Cars i Inter Team. Źródłem jest FTP tych hurtowni.
- **Deduplikacja** - Produkty z trzech źródeł schodzą się po EAN do jednego widoku.
- **Koniec i powrót oferty** - Oferta ze stanem zero kończy się sama. Gdy stan magazynowy wraca, oferta wraca na Allegro.
- **Zamówienie do hurtowni** - Zamówienie z Allegro system składa w ELIT, Inter Cars albo Inter Team. Wołanie idzie przez Edge Functions.

### Bezpieczeństwo

- **Wejście do panelu** - Sesja leży w ciasteczku. Middleware nie otwiera zamkniętych stron bez tej sesji.
- **Tylko administrator** - Do tabel katalogu i ustawień dostęp ma tylko administrator (RLS).
- **Nocne wywołania** - Zaplanowane Edge Functions są uwierzytelniane tajnym nagłówkiem.

---

## Architektura

```mermaid
graph TB
    subgraph suppliers [Hurtownie]
        ELIT[ELIT]
        IC[Inter Cars]
        IT[Inter Team]
    end
    subgraph pipeline [Pipeline w Pythonie na Linode]
        Ingest[Odbiór katalogów]
        Sync[Synchronizacja z Allegro]
        Lifecycle[Cykl życia ofert i deduplikacja]
    end
    subgraph backend [Supabase Cloud]
        PG[(PostgreSQL i RLS)]
        Edge[Edge Functions]
        Cron[Zadania pg_cron]
    end
    subgraph frontend [Next.js na Netlify]
        UI[Dashboard, Katalog, Generator, Drafty]
    end
    Allegro[Allegro API]
    ELIT --> Ingest
    IC --> Ingest
    IT --> Ingest
    Ingest --> PG
    Sync --> Allegro
    PG --> Sync
    Cron --> Edge
    Edge --> Allegro
    UI --> PG
    UI --> Edge
```

---

## Statystyki

### Liczby

| Element | Liczba |
|---|---|
| **Ścieżki panelu** | 5 |
| **Komponenty React** | ~30+ |
| **Edge Functions w Supabase** | 19 |
| **Migracje SQL** | 46 |
| **Skrypty Pythona** | 23 |
| **Integracje z hurtowniami** | 5 |

### Skrót

| Obszar | Co tam jest |
|---|---|
| **Katalog** | Scalenie wielu hurtowni, szukanie trigramami, drzewo kategorii |
| **Oferty** | Wystawianie hurtem, szkice, kalkulator ceny, wznawianie |
| **Zamówienia** | Pobranie z Allegro, przekazanie do dostawcy, statusy |
| **Pipeline** | Import co noc, deduplikacja, stany, sprzątanie nieaktualnych ofert |
| **Bezpieczeństwo** | RLS, logowanie po stronie serwera, tajny nagłówek, dostęp tylko dla administratora |

---

## Moja rola

Zbudowałem Int3gro.pl, platformę do synchronizacji katalogów hurtowni części samochodowych z Allegro.

---

## Kontakt

| Gdzie | Adres |
|---|---|
| **Email** | [wandrysiak.it@gmail.com](mailto:wandrysiak.it@gmail.com) |
| **GitHub** | [wandrysiak](https://github.com/wandrysiak) |

---

**Int3gro.pl** łączy hurtowe katalogi części samochodowych z Allegro w dużej skali.

<p align="center"><em>Zrobili to Wojciech Andrysiak i Kamil Kaczmarek.</em></p>
