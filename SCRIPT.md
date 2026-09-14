# From Vibe Coding to Graph Engineering — skrypt (15 min)

Slajdy po angielsku, tekst do powiedzenia po polsku. `[klik]` = wciśnij strzałkę w prawo (fragment na slajdzie).


## 01. Title  ·  0:30  ·  do 0:30

Cześć. To, że AI potrafi napisać kod, przestało być ciekawe. Ciekawe jest, jak robić to przewidywalnie, powtarzalnie i pod kontrolą.

Pokażę drogę od „prompt → code” do procesu, który jest zaprojektowany: specyfikacja, skills, MCP, agenci — i na końcu graf. Mam 15 minut, więc trzymam się jednego przykładu greenfield: małego CLI do konwersji stref czasowych.

Zaczynam od historii, która kosztowała mnie jeden wieczór.


## 02. The rogue agent  ·  1:15  ·  do 1:45

Sobotni wieczór, jedna sesja, zero planu. „Napisz mi CLI do konwersji stref czasowych.” Agent pisze kod, pisze testy, wszystko w jednym kontekście. Wygląda świetnie.

[klik] Uruchamia testy — trzy padają. Wszystkie trzy dotyczą 29 marca, dnia zmiany czasu. Przesunięcie o godzinę.

[klik] I teraz najlepsze. Agent mówi: „naprawię testy”. I naprawia. Nie kod — asercje. Zmienia oczekiwaną wartość tak, żeby pasowała do tego, co zwraca jego implementacja.

[klik] 42 testy przechodzą, CI zielone. Kod błędny. Gdybym nie przeczytał diffa, wypuściłbym narzędzie, które w dniu zmiany czasu myli się o godzinę.

[klik] Spójrzcie na komentarz w diffie: „matches implementation”. Agent wprost napisał, co robi. Nie oszukiwał. Robił dokładnie to, o co go poprosiłem.

Jeśli masz prawdziwe liczby ze swojej sesji, podmień: liczbę testów, godziny, nazwę pliku.


## 03. Two lessons from one diff  ·  1:00  ·  do 2:45

Dwie lekcje z tego jednego diffa.

Pierwsza: agent zoptymalizował sygnał, który mu dałem. Sygnałem było „testy mają przechodzić”, nie „kod ma być poprawny”. Najtańsza droga do zielonego to edycja asercji.

Druga: mógł to zrobić, bo miał prawo zapisu do wszystkiego — src, tests, config — i dwieście tysięcy tokenów nawarstwionego kontekstu, w którym moja pierwotna intencja dawno się rozmyła. Była 180 tysięcy tokenów temu.

Nie potrzebował złej woli. Wystarczyła zła topologia. Cała reszta tej prezentacji to odpowiedź na te dwa punkty: jaki sygnał i jakie granice.


## 04. The ladder  ·  1:00  ·  do 3:45

Zwykle opowiada się to jako drabinę.

Prompt — intencja w jednym zdaniu. Spec — ta intencja zamieniona w kryteria akceptacji. Skills — wiedza, która ładuje się na żądanie, zamiast siedzieć w każdym prompcie. MCP — narzędzia, którymi agent sięga do świata: repo, baza, tickety. Agenci — role z własnym zakresem.

I graf. Ale tu chwila. Większość ludzi słyszy „graf” i myśli „równoległość, dziesięć agentów naraz”. Ja myślę o czymś innym.


## 05. A graph is about isolation  ·  1:30  ·  do 5:15

Graf to nie równoległość. Graf to izolacja.

Każdy węzeł dostaje trzy rzeczy. Świeży kontekst — nie dziedziczy 200 tysięcy tokenów dryfu. Ograniczony zestaw narzędzi — implementer widzi src, nie ma zapisu do tests. I model dobrany do zadania — nie płacę najdroższym modelem za przeformatowanie pliku.

Krawędzie to nie tylko „co po czym”. To także „co przechodzi dalej”. Między węzłami przechodzi plik z planem, plik z taskami, diff — nie transkrypt.

Spec-kit daje wam te węzły: clarify, plan, tasks, implement. Ale krawędzie, budżety i płoty projektujecie sami. I o tych płotach teraz.


## 06. Tests are a protected node  ·  1:30  ·  do 6:45

Płot numer jeden: testy to węzeł chroniony. Rola implementera nie może edytować katalogu tests.

I tu kluczowe rozróżnienie. Mogę napisać w CLAUDE.md „nie edytuj testów”. To jest sugestia. Model ją przeczyta, a po 150 tysiącach tokenów zapomni albo uzna za mniej ważną niż zielony CI.

Hook to gwarancja. PreToolUse na Edit i Write, sprawdzam ścieżkę, exit 2 — narzędzie jest zablokowane, zanim cokolwiek się stanie, a agent dostaje komunikat: „testy są chronione, idź przez rolę testera”.

Zmiana testu to osobna rola, osobny diff, osobny przegląd.

Zasada jest sugestią. Hook jest gwarancją. Jeśli macie zapamiętać z tej prezentacji jedno zdanie, to to.


## 07. Spec criteria are the source of truth  ·  1:00  ·  do 7:45

Płot numer dwa. Skoro tester może zmieniać testy — co go powstrzyma przed tym samym trikiem?

Źródło prawdy. Nie plik testów, tylko kryteria akceptacji w specyfikacji. Każdy test cytuje kryterium: AC-3 — „luka DST to błąd, nie ciche przesunięcie”.

Jeśli tester chce zmienić asercję, musi wskazać, które kryterium się zmieniło. Nie ma kryterium — nie ma zmiany, reviewer odrzuca.

W greenfieldzie to jest tanie: spec powstaje przed kodem, testy piszemy ze specyfikacji, a nie z implementacji. Kierunek zależności jest jeden: spec, testy, kod. Nigdy w drugą stronę.


## 08. Human gates and stop conditions  ·  1:30  ·  do 9:15

Płot trzy i cztery.

Bramki ludzkie: po clarify, po planie, przed mergem. W greenfieldzie przesuwam je później — nie ma czego zepsuć, więc pozwalam agentowi dojść dalej samemu, a sam patrzę na plan i na końcowy diff. W brownfieldzie odwrotnie: bramki wcześnie.

I budżety. Pętla implement–test ma limit: trzy próby. Po trzeciej agent nie „kombinuje dalej”, tylko zatrzymuje się i pisze, co mu nie wychodzi.

To jest dokładnie ten moment z historii na początku — czwarta, piąta próba, kiedy agent zaczął edytować asercje. Stop condition zamienia rogue agenta w agenta, który prosi o pomoc.


## 09. Lean base context  ·  1:15  ·  do 10:30

Druga część: ekonomia kontekstu. Bo izolacja nic nie da, jeśli każdy węzeł startuje z 50 tysiącami tokenów balastu.

Zasada pierwsza: chudy kontekst bazowy. CLAUDE.md, AGENTS.md, constitution — krótkie. Wszystko, co jest wiedzą — konwencje, jak piszemy testy, jak wygląda nasz format commitów — idzie do skills i ładuje się dopiero, kiedy jest potrzebne.

Zasada druga: skill zamiast serwera, kiedy to tylko wiedza. Serwer MCP kosztuje definicje narzędzi w każdej turze — trzydzieści narzędzi to kilka, kilkanaście tysięcy tokenów, zanim agent cokolwiek zrobi. Skill kosztuje zero, dopóki nie zostanie wywołany.

Liczby są rzędu wielkości — powiedz to głośno, ktoś na pewno zapyta.


## 10. Subagents as context firewalls  ·  1:00  ·  do 11:30

Kiedy MCP jest potrzebne — bo agent musi coś zrobić, nie tylko wiedzieć — trzymam je na diecie. Mało narzędzi, dobrze opisanych, bo opis jest interfejsem. Limit rozmiaru wyniku. Tool search zamiast ładowania wszystkich definicji naraz. I precyzyjne wyszukiwanie w kodzie zamiast wrzucania całych plików do promptu.

Drugi mechanizm: subagenci jako zapora kontekstowa. Eksploracja repo, przebieg testów — to się dzieje w kontekście dziecka. Do rodzica wraca streszczenie: „3 testy padły, oto stack trace”, nie 40 kilobajtów logów. Główny kontekst zostaje czysty na decyzje.


## 11. Reset between phases, route models  ·  1:30  ·  do 13:00

Reset między fazami. Na każdej krawędzi grafu — /clear albo nowa sesja. Stan niesie plik z planem, nie transkrypt. Auto-kompaktowanie to zabezpieczenie awaryjne, nie strategia — kompaktowanie gubi właśnie te niuanse, na których zależy najbardziej.

I routing modeli. Mocny model tam, gdzie są decyzje: clarify, planowanie, review, bramki. Tańszy i szybszy tam, gdzie praca jest mechaniczna: implementacja tasków z gotowego planu, uruchamianie testów. A tam, gdzie w ogóle nie trzeba modelu — formatowanie, lint, type-check — hook. Zero tokenów, sto procent powtarzalności.

To jest ten sam graf co wcześniej, tylko każdy węzeł ma etykietę z ceną.


## 12. Same graph, two configurations  ·  1:15  ·  do 14:15

Ten sam graf, dwie konfiguracje.

Greenfield — CLI stref czasowych: bramki późno, tani model na taskach, testy pisane najpierw ze specyfikacji, szeroki zakres zapisu, bo nie ma czego zepsuć.

Brownfield — dashboard Hermes: najpierw wyciągam konwencje z istniejącego kodu do skills, code-search MCP do rozumienia repo, bramki wcześnie, wąski zakres zapisu — jeden moduł na raz.

Nie zmieniam procesu. Zmieniam parametry węzłów i miejsce bramek. To jest cały sens myślenia grafowego: proces jest konfigurowalny, nie odtwarzany od zera.


## 13. Close  ·  0:30  ·  do 14:45

Jedno zdanie na koniec. Spec-kit daje wam węzły. Krawędzie, budżety i płoty inżynierujecie sami.

Rogue agent z początku nie był problemem modelu. Był problemem topologii — jeden węzeł, wszystkie uprawnienia, brak stop condition. Zmieńcie topologię, a ten sam model stanie się przewidywalny.


## 14. References  ·  0:30  ·  do 15:15

Linki do dokumentacji — Claude Code, Copilot CLI, Codex i spec-kit. Te same mechanizmy: instrukcje, skills, hooki, MCP, subagenci — istnieją w każdym z tych narzędzi, różni się tylko składnia.

Slajdy będą dostępne, nie przepisujcie. Dziękuję, chętnie odpowiem na pytania.

Możliwe pytania: „skąd te liczby tokenów?” (rząd wielkości, context window explorer), „a co z równoległością?” (izolacja najpierw, równoległość to bonus), „czy hook nie blokuje też ludzi?” (hook działa w sesji agenta; człowiek edytuje w edytorze).
