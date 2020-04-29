---
layout: post
title:  "Hibernate i twój własny Stackoverflow"
date:   2020-04-29 20:40:00 +0530
categories: Hibernate
---

Jakiś czas temu kolega zasypał mnie pytaniami odnośnie błędów, na jakie trafia podczas stawiania pierwszych kroków z Hibernatem. Po sprawdzeniu kilku błędów, na które natrafił mój kolega zauważyłem, iż są to w sumie rzeczy związane z podstawową wiedzą. Zaglądając na YouTuba czy Udemy widzimy, że wszyscy pokazują nam jak tworzyć relację między encjami, jak fajnie działa JpaRepository. Mało kto jednak przestrzega przed błędami, które łatwo popełnić, przez co dużo osób nawet nie jest świadoma, że robi coś źle. Czasem wychodzi to na wierzch w postaci błędów, a czasem tylko pozostaje w logach których nikt nie czyta, chyba że aplikacja przestaje działać. Dzisiaj pokażę Wam, jak łatwo można doprowadzić do tego, że będziemy mieli swój własny Stackowerflow w projekcie i nie, nie jest to powód do dumy ani radości. Chociaż Stackowerflow kojarzy się ciepło i miło każdemu programiście.

## TransientObjectException

Na początek coś prostego, co często może się przydarzyć na początku nauki i co może doprowadzić nas do szewskiej pasji albo obrażenia się na Hibernate na jakiś czas. Wyobraź sobie, że piszemy CMS'a i mamy dwie encje `Post` i `Comment`. Wyglądają następująco, od razu zaznaczam, że w zamieszczonych przykładach pomijam nieistotne fragmenty kodu typu gettery, settery. Cały kod, z podziałem na gałęzie mozesz znaleźć w [tym repozytorium](https://github.com/MroczekBartlomiej/ExampleCMS)

```java
@Entity
@Table(name = "posts")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String content;

    @OneToMany()
    @JoinTable(name = "post_comment",
            joinColumns = {@JoinColumn(name = "comment_id")},
            inverseJoinColumns = {@JoinColumn(name = "post_id")}
    )
    private List<Comment> comments;
}
```

```java
@Entity
@Table(name = "comments")
public class Comment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String content;
    @ManyToOne()
    private Post post;
}
```

Proste dwie encje z relacją też niezbyt skomplikowaną. Nasz post może posiadać wiele komentarzy, natomiast komentarz posiada informację o tym, do jakiego posta należy. Okazuje się, że biznes wymaga od nas możliwości dodawania posta z komentarzem, tak ma być i koniec. No więc mamy nasz serwis, który buduje z Requesta(DTO) encję i przekazuje ją do zapisu.

```java 
public Long createPost(PostRequest postRequest) {

        Post post = new Post();
        Comment comment = new Comment();

        post.setTitle(postRequest.titile);
        post.setContent(postRequest.conntent);
        post.setComments(Collections.singletonList(comment));

        comment.setPost(post);
        comment.setContent(postRequest.commentRequset.content);
        
        return postRepository.save(post).getId();
    }
```

Na pierwszy rzut oka powinno działać, zadowoleni dopisujemy jeszcze jakąś logikę, kilka innych relacji między innymi encjami, kodu przybyło pora na test. Uruchamiamy testy, ale od razu testujemy dodawania przez REST Api i klapa, nie działa. W konsoli mamy wielki czerwony błąd: 

```
 ...nested exception is java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing:
```

Szukamy pomocy w Internecie, znajdujemy różne rozwiązania, kopiujemy do nas i dalej nie działa, nie wiemy czemu i o co chodzi. Mamy trochę nowego kodu więc nawet nie wiemy dokładnie który fragment kodu powoduje błąd. Powyższy problem można naprawić dosłownie jedną linijką. 

W klasie `Post` dodajmy do relacji coś takiego, `cascade = CascadeType.ALL`, tak powinna wyglądać nasza relacja: 

```java
@OneToMany(cascade = CascadeType.ALL)
@JoinTable(name = "post_comment",
        joinColumns = {@JoinColumn(name = "comment_id")},
        inverseJoinColumns = {@JoinColumn(name = "post_id")}
)
private List<Comment> comments;
```

Jeżeli chodzi o wyjaśnienie tego co się stało, to można to ująć następująco. W JPA, jeśli jakakolwiek operacja zostanie zastosowana na encji, wówczas wykona się ona tylko na tej konkretnej encji, u nas był to `Post`. Operacja ta nie będzie wykonana na innych powiązanych z nią encjach. U nas tą powiązaną encją był Komentarz. Nie został om zapisany, a my próbowaliśmy wrzucić już do bazy post, który zawierał ten komentarz. Właśnie poprzez dodanie `cascade = CascadeType.ALL` określamy jakie operacje mają się wykonać na powiązanych encjach. Po zajrzeniu do środka `CascadeType` widać, że można wybrać typ operacji, jaki ma następować kaskadowo. Może się okazać, że kiedyś będziemy potrzebowali tylko kaskadowego usuwania, więc wystarczy wtedy wybrać `CascadeType.REMOVE`

Rozwiązanie powyższego problemu jest też możliwe w prosty sposób, ale mniej wygodny. Wystarczyłoby zapisać najpierw komentarz. Otrzymaną encję ustawić jako komentarz encji `Post`. Jednak wyobraź sobie, że masz 6 takich relacji i każdą musisz tak obsłużyć, no nie jest to wygodne. Hiberante pozwala nam to zrobić trzema dodatkowymi wyrazami w i tak już istniejącej linijce.

## LazyInitializationException 

Kolejnym problemem, na jaki możemy natrafić, zaczynając naszą przygodę z Hibernatem jest właśnie sławny `LazyInitializationException`. Tłumacząc to dosyć luźno, problem polega na tym, że baza danych zwraca nam wynik zapytania o jakiś post. Następnie wynik zapytania jest mapowany na nasz obiekt Post, ale już komentarze do tego posta nie są pobierane z bazy. Hibernate inicjuje listę komentarzy swoją własną implementacją listy. Dopóki nie próbujesz dostać się do listy komentarzy, to nic się nie stanie. Dopiero w momencie, gdy zechcemy coś pobrać z tej listy, zostanie wyrzucony wyjątek. Powodem tego jest brak aktywnej sesji połączenia z bazą danych. Zobaczmy w takim razie, jak to będzie wyglądało w kodzie:

```java
public List<PostResponse> getPosts() {	//Pobieramy wszystkie posty i mapujemy je na responese.
    
    return postRepository.findAll().stream()
            .map(this::mapToPostResponse)
            .collect(Collectors.toList());
}

private PostResponse mapToPostResponse(Post post) { //Ta metoda mapuje nam post na response.
	
    PostResponse postResponse = new PostResponse();
    postResponse.id = post.getId();
    postResponse.titile = post.getTitle();
    postResponse.conntent = post.getContent();

    List<Comment> comments = post.getComments(); //1.Unable to evaluate the expression Method threw 'org.hibernate.LazyInitializationException' exception.
    postResponse.comments = comments.stream() // 2.org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: dev.bartmroczek.stackoferflow.Post.comments, could not initialize proxy - no Session
            .map(this::mapToCommentResponse)
            .collect(Collectors.toList());

    return postResponse;
}
```

W wyżej zamieszczonym przykładzie kodu widzimy linijkę oznaczoną `1.Unable to evaluate the expression...` taką wartość możemy zobaczyć, gdy zatrzymamy się tam w trybie debugowania, natomiast linijkę niżej zostanie już wyrzucony wyjątek: ```org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: dev.bartmroczek.stackoferflow.Post.comments, could not initialize proxy - no Session```. Wklejając to w Google, można znaleźć poradę, która mówi o tym, że wystarczy dodać 'fetch = FetchType.EAGER' i sprawa powinna być załatwiona. Oczywiście nie jest to najlepsze rozwiązanie, ale to jest temat na inny artykuł. Będzie to działało, dopóki nie dopiszemy sobie małej i niewinnej funkcjonalności.

## StackOverflowError

To jest jeszcze jeden fajny problem który przychodzi mi do głowy, na jaki można natrafić, bawiąc się Hibernatem. Otóż wyobraźcie sobie, że zachwyceni tym jak łatwo udało się nam rozwiązać poprzedni błąd, postanawiamy dopisać kolejną funkcjonalność. Chcemy mieć informację w komentarzu, do jakiego postu został dodany. Dodajemy sobie do naszego `CommentResponse` pole `public PostResponse post;` następnie modyfikujemy naszą metodę mapującą `Comment'`na `CommentResponse` wszystko wygląda jak poniżej.

```java
private CommentResponse mapToCommentResponse(Comment comment) {

    CommentResponse commentResponse = new CommentResponse();
    commentResponse.id = comment.getId();
    commentResponse.contennt = comment.getContent();
    commentResponse.post = mapToPostResponse(comment.getPost());
    return commentResponse;
}
```

Po uruchomieniu widzimy niemiły błąd: `java.lang.StackOverflowError` i nie za bardzo wiemy co się stało i o co chodzi. Przecież dopisaliśmy tylko odrobinę prawie nic nie robiącego kodu.  

Problem pojawia się z prostego powodu, mianowicie mamy zapętlone mapowania. Wyobraź sobie, że mamy nasz post, który ma jeden komentarz. Zaczyna się mapowanie i proces wygląda tak:

1. Mapujemy pola tekstowe w poście.
2. Natrafiamy na Komentarz, mapujemy pola tekstowe w komentarzu.
3. O mamy post, zmapujmy post.
4. Mapujemy pola tekstowe w Poście.
5. Natrafiamy na Komentarz, mapujemy pola tekstowe w komentarzu.
6. O mamy Post, zmapujmy post.
7. Mapujemy pola tekstowe w Poście.
8. Natrafiamy na Komentarz, mapujemy pola tekstowe w komentarzu.
9. ...

Dalsze punkty dopisz sobie sam, tylko nie wiem w sumie, kiedy masz przerwać to wypisywanie. Tak samo to wygląda u nas w aplikacji. Tylko JVM wie kiedy przerwać bo przepełnia jej się stos. Dla niedowiarków dodałem logi do konstruktorów, oto wynik:

```
Comment constructor executed.
Post constructor executed.
2020-04-29 15:21:35.323  INFO 21408 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: 
[... pozostałe logi ...]

Post constructor executed.
Post constructor executed.
Comment constructor executed.
Comment constructor executed.


java.lang.StackOverflowError
[...]
at dev.bartmroczek.stackoferflow.PostResponse.PostResponse.<init>(PostResponse.java:8)
	at dev.bartmroczek.stackoferflow.PostService.mapToPostResponse(PostService.java:45)
	at dev.bartmroczek.stackoferflow.PostService.mapToCommentResponse(PostService.java:63)
	... pozostałe logi
	at dev.bartmroczek.stackoferflow.PostService.mapToPostResponse(PostService.java:53)
	at dev.bartmroczek.stackoferflow.PostService.mapToCommentResponse(PostService.java:63)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195)
	... pozostałe logi
	at dev.bartmroczek.stackoferflow.PostService.mapToPostResponse(PostService.java:53)
	at dev.bartmroczek.stackoferflow.PostService.mapToCommentResponse(PostService.java:63)
	at java.base/java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:195)
	... pozostałe logi
```

Możemy zobaczyć, że konstruktory zostały wywołane kilka razy. Widać też wyraźnie, że powtarzają się 3 linijki w logach. Ja naliczyłem 103 powtórzenia.

To jest właśnie efekt szybkiego "naprawienia" błędu `LazyInitializationException`. Jest kilka sposobów na uniknięcie problemu`LazyInitializationException`. Zakładając że postanowiłeś zostawić to `fetch = FetchType.EAGER`, możesz się jeszcze ratować następujący sposób. Podczas mapowania encji na ResponseDto możemy wykorzystać coś w stylu SimpleRespons i przekazać tylko ID i jakąś nazwę obiektu do wyświetlenia(np. tytuł posta), do którego nasz główny obiekt ma relację. Troszkę bardziej skomplikowane rozwiązanie z punktu widzenia początkującego to zastosować `LEFT JOIN FETCH` do zainicjowania encji, z którymi nasza główna encja ma relacje. 

## Podsumowanie 

Jak widać łatwo zrobić błąd pracując z Hibernatem. Pokazane tutaj błędy są co prawda prostymi błędami i łatwo je znaleźć, jeżeli zgłębimy się tylko trochę w dokumentacji, czy w kodzie samego frameworka. Pracując w zespole z doświadczonymi programistami takie coś powinno zostać skorygowane już przy code review. Podczas nauki nie zawsze jest łatwo znaleźć kogoś kto pokaże Ci takie błędy i da radę jak z tego wyjść. Mam nadzieję że chociaż trochę pomogłem Ci ustrzec się od błędów w pracy z Hibernatem. 



