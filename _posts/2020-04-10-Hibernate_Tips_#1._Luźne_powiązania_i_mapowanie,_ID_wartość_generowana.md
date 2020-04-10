---
layout: post
title:  "Hibernate Tips #1 - Generowanie ID, Relacja Mant-to-one z luźnym wiązaniem."
date:   2020-04-10 21:45:00 +0530
categories: Hibernate
---
W tym poście opisałem dwa szybkie patenty na ostatnio spotkane problemy, generowanie ID i najlepszy typ danych dla kolumny ID, jak również jak skonfigurować relację Many-to-One stosując luźne wiązanie pomiędzy  encjami, wykorzystując same identyfikatory. 

##### Generowanie ID oraz typ kolumny w Postgresie

Używając Postrgresa mamy do dyspozycji takie typ danych jak **SERIAL** oraz **BIGSERIAL**, jednak po zgłębieniu tematu okazuje się że stosowanie tego typu danych przy jednoczesnym korzystaniu z JPA nie jest najlepszym wyborem. 

Zarówno **SERIAL** oraz **BIGSERIAL** są typami danych auto-inkrementującymi, jest to zrealizowane poprzez wygenerowanie oddzielnej sekwencji dla danej kolumny. Nazwa sekwencji będzie według wzoru `nazwaTabeli_nazwaKolumny_seq`. 

Jeżeli zarządzamy bazą danych za pomocą skryptów SQL i Flyway czy innego tego typy narzędzia, lepszym rozwiązaniem jest ustawienie kolumny z ID jako zwykły **INT** i stworzenie sekwencji dal tej kolumny, oraz podanie nazwy sekwencji nad polem z identyfikatorem w encji JPA wykorzystując adnotację `@GeneratedValue(generator = "nazwa_sekwencji")`. Warto pamiętać o pominięciu pola `stategy` ponieważ wtedy będziemy musieli dodatkowo podać dodatkowe parametry z wykorzystując adnotację `@SequenceGenerator()`, a to nie zawsze jest konieczne. 

Więcej na ten temat można przeczytać we wpisie Vlada pod adresem:[ PostgreSQL SERIAL column and Hibernate IDENTITY generator](https://vladmihalcea.com/postgresql-serial-column-hibernate-identity/ )

##### Relacja Mant-to-one z luźnym wiązaniem, tylko po identyfikatorach.

Jeżeli stosujemy w encjach same identyfikatory zamiast pełnych obiektów problematyczne może być szybkie i proste ogarnięcie relacji Many-to-One tak aby automatycznie dodawały na się wpisy do tabeli łączącej. Dobrym sposobem na to jest wykorzystanie adnotacji:  

```java
@ElementCollection
	@CollectionTable(name = "post_comments",
                     joinColumns = @JoinColumn(name = "post_id"))
@Column(name = "comment_id")
private Set<Long> comentsId;
```

Powyższy przykład znaczy dokładnie to samo co poniższy przykład przy wykorzystaniu standardowego sposobu używanie Hibernate. 

```java
@ManyToMany
    @JoinTable(name = "post_comments",
            joinColumns = @JoinColumn(name = "post_id"),
            inverseJoinColumns = @JoinColumn(name = "comment_id"))
    private Set<Comments> comments;
```

Informację na ten temat można zgłębić w ty wątku na Stack Overflow - [How to load only ids from Many to Many mapping tables?](https://stackoverflow.com/questions/16999997/how-to-load-only-ids-from-many-to-many-mapping-tables)