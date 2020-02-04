---
layout: post
title:  "Git - dlaczego zawsze powinniśmy robić rebase."
date:   2020-02-03 21:30:00 +0530
categories: Git
---

Od dłuższego czasu obserwuję że dla sporej liczby programistów Git jest narzędziem z którym pracują codziennie, a jednak nie do końca wiedzą jak działa i jak z niego poprawnie korzystać. Pewnie zastanawiasz się na podstawie czego wyciągam takie wnioski. Otóż taka prosta rzecz jak aktualizacja swojego brancha na którym pracuje programista względem mastera (lub innego głównego brancha) potrafi przysporzyć sporo problemów. W tym wpisie postaram się po raz kolejny wytłumaczyć dlaczego warto używać funkcji rebase i dlaczego nie warto się bać modyfikacji historii którą to straszą przeciwnicy używania rebase.

Rebase i merge służą do tego samego, integrują zmiany w kodzie z dwóch różnych branchy, ale sposób w jaki to robią znacznie się od siebie różni.  Rebase aktualizuje brancha poprzez zmodyfikowanie historii brancha na którym został wywołany, natomiast merge stworzy commit mergujący i nie zmieni już istniejących commitów. Efektem ubocznym tego będzie to ze historia stanie się mało czytelna z powodu istnienia commitów megujących.  

### Merge

Zobaczmy jak wygląda branch podczas używania `git merge` w celu pobrania nowych zmian z mastera. 

![merge](\assets\Git_-_rebase\merge.jpg)

Opisując sytuację jaką chciałem przedstawić na powyższym diagramie posłużę się pewną historią z życia dwóch programistów: 

1. Feature branch został stworzony gdy w masterze były dwa commity M1 i M2, a programista Jan zaczął pracować nad nową funkcjonalnością. (niebieskie kropki)

2. W czasie gdy programista Jan pracował na nową funkcjonalnością drugi programista Marek skończył inne funkcjonalności i pojawiły się commity M3 i M4. 

3. Okazało się ze zmiany Marka są potrzebne Janowi, więc Jan zmergował mastera do swojego feature brancha. Wykonał polecenia: 

   ```bash
   git checkout feature
   git merge master
   ```

   Lub zrobił to jednym poleceniem:

   ```bash
   git merge feature master
   ```

4. Jan zadowolony z tego że zmiany Marka są w jego branchu kontynuował pracę nad swoimi funkcjonalnościami. 

No i w tym momencie może się wydawać że cały ten wpis jest bez sensu bo przecież cel został osiągnięty. Zmiany z mastera są w feature branchu  i wszyscy sztucznie pompują temat tego że nie powinno się używać merge tylko rebase. Popatrzmy jednak na tą zieloną kropkę, która symbolizuje commit.  To jest właśnie ten commit który jest robiony podczas mergowania branchy. O niego jest cała ta wojna, bo co jeżeli master jest bardzo aktywny i dochodzi do niego kilka commitów na godzinę, a Janek musi robić ciągle merge żeby mieć nowe funkcjonalności w swoim branchu. Może się okazać że będzie miał tyle samo commitów ze swoją pracą co merge commitów. Później gdy taki feature branch wchodzi do mastera to wszystkie te commity idą za nim, bo tak naprawdę mało kto robi squasha. W przyszłości podczas śledzenia tego co działo się w projekcie ciężko to wywnioskować jeżeli mamy tyle merge commitów. Poniżej pokazuję jak by wyglądała dalsza praca Jana gdyby stosował tylko merge. Widać  że merge commity przybywają w jego branchu i zaraz może ich być więcej niż normalnych commitów. 
![](\assets\Git_-_rebase\merge-2.jpg) 



### Rebase

Przyjrzyjmy się jak powinno wyglądać prawidłowe podejście do aktualizacji  brancha na którym pracujmy lokalnie i implementujemy nową funkcjonalność lub naprawiamy buga.  Mając tu na myśli poprawną aktualizację mówię o sytuacji w której commity z mastera wchodzą do naszego brancha bez tworzenia żadnych dodatkowych commitów typu "merge commit". 

![](\assets\Git_-_rebase\before-rebase.jpg)



Powyższy obraz przedstawia sytuację podobną do tej sprzed chwili gdzie programista Jan miał swojego brancha a Marek oddał swoje zmiany i pojawiły się commity w masterze. Jednak tutaj jesteśmy chwilę wcześniej niż poprzednio. Jak nie zabrał się za aktualizację swoich zmian. To co teraz powinien zrobić Jan aby mieć commity w Marka u siebie to wykonać rebase. Więc pora zacząć, Jan na początek musi pobrać sobie zmiany Marka ze zdalnego repozytorium i zakładając że ma aktualnego mastera może zabrać się za wykonywanie rebase. 

1. Na początek Jan musi przełączyć się na swojego brancha, czyli robi checkout po czym może rozpocząć wykonywanie rebase: 

   ```bash
   git checkout feature
   git rebase --autostash master
   ```

2.  Zakładając pozytywny scenariusz to wszystko, teraz Jan powinien mieć zmiany Marka już w swoim branchu.  W negatywnym może mieć do rozwiązania konflikty które rozwiązuje się w analogiczny sposób jak podczas mergowania. 

Zobaczmy jak wygląda w takim razie schemat branchy i commitów po wykonaniu rebase. 


 ![](\assets\Git_-_rebase\rebase-B.jpg)



Jak widać rebase powoduje że historia  staje się liniowa. Czyli tak naprawę  zostanie zmieniona podstawa  feature brancha. Wcześniej rozwidlenie następowało po commicie M2, obecnie feature branch odbija dopiero za commitem M5. Można to też przedstawić tak jak poniżej i stwierdzić że commity z mastera które pojawiły się po odbiciu rebase trafiają do brancha feature, ale "pod nowe commity".

![](\assets\Git_-_rebase\rebase-A.jpg) 

Jest to dosyć kolokwialne stwierdzenie i należy pamiętać że commity od M3 do M5 tak naprawdę nie są dublowane. A zmienia się jednynie podstawa feature brancha. 

> #### Ale robienie rebase, niszczy nam historię, dlatego nie powinniśmy go używać.

To stwierdzenie często można usłyszeć podczas próby przekonania kogoś do do robienia rebase. Jeżeli dobrze się przyjrzysz to zobaczysz że niebieskie kropki symbolizujące commity mają dodatkowe oznaczenie w postaci `, a wcześniej przed wykonaniem rebase nie miały tego dodatkowego symbolu. To jest właśnie ta "utrata historii" o której często się mówi, jak wiadomo każdy commit ma swój hash. W chwili robienia rebase Git tworzy identyczne commity jak te które już istniały na feature branchu tyle że przed tymi commitami umieszcza commity z brancha na który jest rebasowany, a te które już istniały po prostu kasuje. Zawartość commita pozostaje ta sama, message też się nie zmienia, jedynie hash zostaje wygenerowany na nowo. 

Dlatego często się mówi że nie powinno się robić rebase bo niszczy historię, ale nie precyzuje się tego dokładniej. Nie powinno się tego robić na branchach które są używane przez kilka osób, ponieważ może się okazać że te same commity będą miały inny hash, natomiast śmiało można robić rebase a nawet powinno się go robić zaciągając zmiany z głównego brancha do feature brancha. Zyskuje się w ten sposób czytelniejszą historię, niezaśmieconą merge commitami. Rozwiązując konflik podczas wykonywania rebase nie będziesz go musiał rozwiązywać po raz kolejny przy następnym rebase w przeciwieństwie do merge, gdzie ten sam konflikt może się pojawić podczas kilku kolejnych mergy.  

W następnym wpisie postaram się przedstawić to w praktyce i pokazać jak to wygląda na bardziej życiowym przykładzie. Stworzę jakąś przykładową stronę www i przejdę przez ten sam scenariusz za pomocą polecenia rebase i merge oraz porównam efekty tego działania. 

---

