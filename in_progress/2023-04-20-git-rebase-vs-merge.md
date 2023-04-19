---
title: "git rebase vs merge"
categories:
  - Blog
tags:
  - git
---


Git rebase vs. Git merge?

Co powinienem wybrać?

Już tłumaczę i objaśniam.

1️⃣ Gdy pracujesz na swoim feature branchu, a w międzyczasie pojawią się zmiany na głównym, to zachodzi potrzeba synchronizacji.

         A---B---C feature
         /
D---E---F---G---H---I master

W tym wypadku, do feature brancha chcesz dołączyć zmiany z commitów F-G-H-I.

2️⃣ Do wyboru masz dwie opcje:

- git merge
- git rebase

3️⃣ Git merge sprawi, że kod z mastera zostanie wciągnięty do twojego brancha dodatkowym commitem mergującym J.

         A---B---C---[J] feature
         /                 /
D---E---F---G---H---I master

W ten sposób nie zmieniasz historii feature brancha, ale też umieszczasz w nim commitów F-G-H-I wprost, a poprzez dodatkowy sztuczny commit J.

4️⃣ Git rebase natomiast przepnie początek twojej gałęzi na ostatni commit z mastera.

                                A'--B'--C' feature
                                /
D---E---F---G---H---I master

Dzięki temu cała historia feature brancha jest w pełni sekwencyjna z historią z mastera.

5️⃣ W ten sposób w momencie mergowania do mastera możesz:

1) zachować wszystkie commity z feature brancha i otrzymać ładną historię zmian

2) zesquashować wszystkie zmiany i otrzymać jeden, ładny pojedynczy commit bez potrzeby rozwiązywania konfliktów w ostatecznym etapie pracy z feature branchem

3) dodatkowo będąc na feature branchu możesz w łatwy sposób podejrzeć zmiany poszczególnych commitów z master brancha

6️⃣ Co więc wybrać, zapytasz? 🤔

Ja tylko i wyłącznie wybieram GIT REBASE.

Dzięki temu otrzymuję czystą historię zmian, a mergowanie feature brancha do mastera jest już czystą przyjemnością.

Daj znać, który styl Ty i Twój zespół preferujecie.