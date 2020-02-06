---
layout: post
title:  "Merge vs Rebase - czy naprawdę widać różnicę? Czy historia jest taka ważna? "
date:   2020-02-06 09:07:00 +0530
categories: Git
---

Po ostatnim wpisie mówiącym o tym że powinno się używać rebase zamiast merge postanowiłem przeprowadzić  mały eksperyment. Wymyśliłem sobie że zrobię jakiś mały "projekt", w którym zasymuluję pracę kilku programistów nad prostą stroną www, poziom taki trochę jak na lekcjach informatyki w szkole podstawowej, ale to nic. Zrobiłem pierwsze podejście gdzie nie używałem polecenia rebase, oraz drugie i tam już postanowiłem używać rebase. Starałem się mniej więcej w obu podejściach robić to samo jeżeli chodzi o dodawany kod i commity, ale inaczej używać Gita i muszę przyznać że po scaleniu wszystkich branchy efekt widać gołym okiem. Zobaczcie sami jak wygląda graf obu projektów i wyobraźcie sobie że w projekcie jest więcej programistów i na raz odbija się więcej branchy, które są rozwijane, czasem zawieszane, ale wszystkie muszą być aktualizowane żeby mieć w miarę spójną wersję aplikacji na każdym branchu. 

![](\assets\Git_-_rebase\withMerge.PNG)

![](\assets\Git_-_rebase\withRebase.PNG)



Jest różnica, nie? Jak się przyjrzysz dokładniej to pewnie zobaczysz że ten pierwszy to efekt mergowania w obie strony, bo widać merge commity. W drugim przypadku mamy już jedną pionową linie z kropkami i to jest właśnie efekt rebase, oczywiście to jest wizualizacja z SourceTree. Zobacz jak to wygląda w Gitk.

![](\assets\Git_-_rebase\merge-gitk.png)

Jak widać wizualizacja jest trochę inna niż w SourceTree, jednak w mojej opinie wygląda to lepiej jeżeli używa się rebase. Nie ma dziwnych poziomych linii pomiędzy różnymi branchami, commity są umieszczone jeden pod drugim, dokładnie w takiej kolejności w jakiej trafiały do głównego brancha. Jeszcze jako przykład pokaże wam jak wygląda to jeżeli mamy repozytorium na Gitlabie, jednak tam już bez opisu poszczególnych commitów bo to już projekty komercyjne, ale po lewej stronie jest drzewo gdzie rebase nie był stosowany a po prawej już tak:

![](\assets\Git_-_rebase\Gitlab.jpg) 



Kilka lat temu patrząc na te dziwne linie z kropkami nie potrafiłem z nich nic wywnioskować, były ze sobą poplątane i nie tworzyły żadnego sensownego połączenia. Dopiero zrozumienie Gita i tego jak budowana jest historia pozwoliło mi się w tym odnaleźć. W momencie gdy zacząłem jeszcze razem z zespołem stosować rebase przed zmergowaniem brancha to już całkiem przyjemnie patrzyło się na graf z historią zmian i potrafiłem dużo wywnioskować co się ostatnio działo. 

Obecnie gdy wchodzę do projektu to zawsze patrzę na ten wykres i widzę czy programiści wiedzą jak poprawnie budować historię czy nie, jeżeli da się coś z tego wywnioskować bez nadmiernego wysiłku patrzę czy zadania są podzielone na mniejsze i czy code review jest rzetelnie robiony. Jak wracam z urlopu czy innej nieobecności to też zawsze sprawdzam co tam w grafie się dzieje i pokazuje mi to co się zmieniło w projekcie. Uważam że budowanie czytelnej i zrozumiałem historii w Gicie to równie ważna rzecz co pisanie czytelnego i schludnego kodu. Patrząc na historię łatwiej ogarnąć co się działo w projekcie niż czytać kod klasa po klasie czy dopytywać zespołu. Podobno można jeszcze patrzeć w zrealizowane zadania na Jirze czy innym systemie organizacji pracy , ale komu by się chciało tam grzebać. Dlatego pamiętaj, utrzymanie historii w Gicie to równie ważna rzecz jak pisanie kodu. 
