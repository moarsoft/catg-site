# Cześć Pierwsza: "DevOps, CI/CD, Ansible i cała reszta".

## Tytułem wstępu.

Razem z modą na tak zwany "DevOps" objawiła się masa narzędzi, które ten cały "DevOps" mają wspierać. Osobiście jestem zwolennikiem tezy, że kultura "DevOps" to nic innego jak zastosowanie narzędzi typowo developerskich w pracy szeroko pojętego "operations" i w drugą stronę, uświadomienie developerom, że robiąc swoje w przemyślany sposób można bardzo ułatwić życie opsowcom. Żebyśmy się dobrze zrozumieli, większość tych narzędzi istniała w takiej lub innej formie od zarania dziejów IT i koncepcje tu prezentowane raczej nie są przełomem czy nowością (choć zazwyczaj tak są promowane), a większość ludzi z branży IT je zna i wykorzystuje, niekoniecznie świadomie. Moda na "DevOps" ubrała je tylko w ładne wdzianko, opatrzyła odpowiednią etykietką, co zwiększyło ich popularność i PR. Tak jest na przykład z CI/CD (Continues Integration/Continues Deployment), zakładające dostarczanie i integrację zmian raczej w momencie gdy owe zmiany powstaną niż gromadzenie ich w jakieś duże paczki, które dopiero wtedy są kompilowane, wrzucane na środowiska i testowane. Od kiedy pamiętam (a będzie już prawie 20 lat w branży) istniały jakieś systemy wersjonowania i zarządzania zmianą (CVS, Rational ClearCase, Subversion, Git, Mercurial, Perforce), automaty do tworzenia "build systemów" i śledzenia zależności (GNUMake, Ant, Maven), automaty do rejestracji zmian (Bugzilla, Jira), czy też w końcu narzędzia do instalacji aplikacji na środowiskach (RPM, PiP, APT, Yum, itp...). Dużo firm miało własne, domowe rozwiązania, które skutecznie wykorzystywały i spinały te narzędzia w jednolite systemy.

Postępująca automatyzacja procesów wytwarzania oprogramowania doprowadziła do powstania rozwiązań jej dedykowanych. Powstały więc tak zwane **systemy dostawcze** (z ang. provisioning systems) wspomagające zarządzanie dostawami i konfigurację środowisk oraz aplikacji. Najpopularniejsze z nich to Chef (https://www.chef.io/), Puppet (https://puppet.com/) i Ansible (https://www.ansible.com/). My zajmiemy się tym ostanim.

Wszystkie systemy dostawcze sprowadzają się do jednego: **Przechowywania stanu i logistyki środowisk w formie opisowej**.
Cała reszta mechanizmów jest pochodną tego podstawowego zadania. Taki sposób zarządzania środowiskami określa się jako "infrastructure as a code" (**IAAC**). Tu właśnie dochodzimy do Ansible'a, który jest modelowym przykładem takiego systemu.

## Ansible od bebechów.

Ansible to tak naprawdę zestaw modułów i skryptów napisanych w Pythonie co ma tą zaletę, że właściwie na każdym systemie typu Unix jest Python instalowany niejako domyślnie (pomijam instalacje typu minimal). Oczywiście sam "silnik" Ansible'a (także napisany w Pythonie) musi być zainstalowany na maszynie, którą będziemy określać jako **"master"** lub **"control node"**. Tam rezyduje konfiguracja i wszystkie moduły, zestawy zadań itp. Po stronie środowisk docelowych jedynymi wymaganiami są obecność Pythona i połączenie ssh do komunikacji z masterem. Można wymusić użycie innego protokołu komunikacji używając wtyczek (np. winrm) i wskazując je w konfiguracji Ansible, ale to sobie zostawimy na później.

W dokumentacji do Ansible można wyczytać, że jest to narzędzie typu "agentless" co nie do końca jest prawdą. Fakt, na maszynach docelowych nie ma żadnego rezydentnego demona czy innego procesu, który jest wymagany do komunikacji. Nie oznacza to jednak, że agenta w ogóle nigdy nie ma. Użyto tu pewnej sztuczki ponieważ agent jest umieszczany na maszynie docelowej tylko na czas potrzebny do wykonania zadań, a zasada działania jest prosta. Na masterze wskazywane są zadania i moduły, które mają być użyte do ich wykonania. Ansible przesyła to co potrzebuje do katalogu tymczasowego na maszynie docelowej, wykonuje zadania i po ich zakończeniu obojętne, pozytywnym czy negatywnym, usuwa ten katalog wraz z zawartością.
Ma to konsekwencje w postaci dodatkowego narzutu (raczej niewielkiego) w ruchu sieciowym oraz prowadzi czasem do błędów gdy moduł, który ma być użyty, wymaga dodatkowych, niestandardowych elementów na maszynie docelowej albo nie ma dostępu do wymaganych przezeń narzędzi systemowych.

Polecam ten artykuł: https://www.slashroot.in/how-does-ansible-work jako lekturę uzupełniającą.

## Instalacja i konfiguracja Ansible.

Co do instalacji - odsyłam do dokumentacji Ansible (https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installation-guide ).

Z konfiguracją jednak wiąże się kilka dość istotnych kwestii, które warto wymienić.

Cała konfiguracja trzymana jest w plikach .cfg (np. /etc/ansible/ansible.cfg ). Brany jest pod uwagę pierwszy plik konfiguracyjny jaki zostanie znaleziony na masterze. Kolejność wyszukiwania jest następująca:

- Plik wskazany w zmiennej środowiskowej ANSIBLE_CONFIG;
- Plik ansible.cfg w aktualnym katalogu, z którego został wywołany ansible;
- Ukryty plik ~/.ansible.cfg w katalogu domowym użytkownika wywołującego ansible;
- Plik w ogólnej ścieżce /etc/ansible/ansible.cfg .

Można to wykorzystać by zmieniać konfigurację na aktualnie potrzebną podczas wywołania. Trzeba tylko pamiętać, że raz przeczytana konfiguracja zostaje (choć poszczególnymi parametrami da się manipulować "w locie"). Jeśli nie ma żadnego z plików - parametry przyjmują wartości domyślne.

Żeby sprawdzić, który plik konfiguracyjny zostanie wykorzystany można użyć opcji --version dostępnej w większości poleceń ansible.

    ansible --version

Pełniejszy dostęp do konfiguracji zapewnia polecenie ansible-config. I tak na przykład komenda:

    ansible-config list 

Prezentuje wszystkie skonfigurowane parametry w danym pliku konfiguracyjnym. Można wskazać, który plik konfiguracyjny ma zostać odczytany, w przeciwnym wypadku obowiązuje pierwszy znaleziony według podanej wyżej reguły.

Dość użyteczną funkcją jest komenda:

    ansible-config dump --only-changed

Pokazuje ona tylko te parametry konfiguracyjne, które różnią się od domyślnych.

Jeśli chodzi o opcje konfiguracyjne to jest ich mnóstwo i nie ma sensu opisywać każdego z nich. Kompletny wzór pliku ansible.cfg znajdziecie pod tym adresem: https://github.com/moarsoft/catg-example-files/blob/master/ansible.cfg

W dalszych częściach będę czasem odwoływał się do konfiguracji więc na razie wystarczy by było wiadomo gdzie ją znaleźć i jak podglądać.

## Ansible CLI.

Ansible umożliwia korzystanie z linii poleceń przy użyciu kilku narzędzi, niektóre z nich pojawiły się już powyżej. Zawsze użyteczny jest tryb "verbose" uruchamiany przez dodanie -v (lub wielokrotności v np -vvvv) do wywołania komendy co umożliwia uszczegółowienie opisu wykonywanych działań. Polecam sprawdzić samemu:

    ansible -m ping localhost
    ansible -m ping localhost -vvvvv

Przydatne jest to zwłaszcza w sytuacjach wymagających diagnostyki.

Wracając do komend. Przykład powyżej obrazuje prosty sposób na wywołanie konkretnego modułu Ansible'a (przełącznik -m). Można w ten sposób używać właściwie dowolnych modułów pod warunkiem, że składnia będzie poprawna.

    ansible -m file -a "path=./somefile.test state=touch" localhost

Przykład powyżej wykonuje polecenie "touch" w aktualnym katalogu na podanym pliku przy użyciu modułu "file". Schemat wywołania jest prosty:

**-m** oznacza, że chcemy wywołać jakiś moduł Ansible'a (domyślnie wołany jest moduł "command" co łatwo sprawdzić <code>ansible -a "env" localhost</code> vs <code>ansible -m command -a "env" localhost</code><br>
**-a** to zestaw atrybutów jakie przekazujemy do modułu.<br>
**localhost** czyli docelowy adres maszyny, na której chcemy wykonać moduł. Można go podać na początku jak i na końcu (<code>ansible localhost -m ...</code>), można podać listę hostów (np. "host1,host2" zamiast nazw można użyć adresów IP) lub grupę zdefiniowaną w inventory. 

O właśnie.

Można łatwo zauważyć ostrzeżenie podczas wykonania powyższego polecenia: 

**[WARNING]: No inventory was parsed, only implicit localhost is available**

Oznacza to, że nie wskazaliśmy źródła, z którego Ansible czerpie wiedzę o docelowych hostach, tzw. **inventory** więc jedynym dostępnym celem jest aktualna maszyna. O tym w osobnym wpisie, bo to dość obszerny temat, na chwilę obecną wystarczy wiedza, że do wskazania pliku z listą hostów służy przełącznik **-i**.    

Aby przetestować połączenie z mastera do docelowej maszyny można użyć modułu "ping".

    ansible -m ping localhost

Z ciekawszych modułów jakie można sobie wywołać (dla treningu) to moduł "setup".

    ansible -m setup localhost

Wywołanie go pozwala na zebranie danych (**faktów**) o docelowej maszynie, które można później wykorzystać. 

Tutaj mała dygresja. Zbieranie faktów przez ansibla odbywa się poprzez użycie zestawu poleceń systemowych (np. ifconfig -a) na docelowej maszynie co oznacza, że trzeba mieć do tego właściwe uprawnienia. Na przykład u mojego providera gdzie mam hosting moduł setup zgłasza błąd:

    localhost | FAILED! => {
        "changed": false,
        "cmd": "/sbin/ifconfig -a",
        "msg": "[Errno 13] Permission denied: '/sbin/ifconfig'",
        "rc": 13
    }

Można to obejść używając właściwego konta użytkownika. Ansible oczywiście umożliwia wskazanie jako kto chcemy wykonać zadania na docelowej maszynie. Rzecz jasna konto musi tam istnieć, mieć właściwe uprawnienia, dostępy, ustawione klucze ssh itd.
Tu objawia się wada tego, że Ansible nie ma rezydentnego agenta, który mógłby być systemowo zainstalowany i zaopatrzony we wszystko co potrzebne. Wystarczyłoby tylko go użyć i reszta by nas nie interesowała. W przypadku Ansible'a trzeba niestety wyposażyć docelową maszynę np w użytkowników technicznych co czasem bywa kłopotliwe.

Aby użyć specyficznego konta użytkownika można go wskazać przełącznikiem -u.

    ansible -m setup -u root 

Oczywiście rzadko się zdarzy, że tak po prostu się zalogujemy (zwłaszcza na roota ;)). Można więc podać hasło (o ile je znamy):

    ansible -m setup -u root --ask-pass (lub -k)

Można też, jeśli mamy, użyć klucza ssh:

    ansible -m setup -u root --private-key=<ścieżka do klucza>

Kolejną alternatywą jest użycie "sudo", o ile możemy. Służy do tego przełącznik "--become", który wykonuje moduł poprzez sudo z danego konta.

    ansible m setup --become

Można np zapytać o hasło do sudo przez "--ask-become-pass".

Inne polecenia CLI:

- ansible-playbook - używany do uruchamiania list zadań. Chyba najczęściej wykorzystywana komenda, zajmiemy się nią w dalszych częściach.
- ansible-console - interaktywna konsola ansible wywoływana w kontekście docelowego hosta. Umożliwia "ręczne" wykonywanie zadań.
- ansible-config - wyświetla konfigurację Ansible'a.
- ansible-inventory - wyświetla zawartość wskazanego inventory.
- ansible-vault - służy do maskowania zawartości plików lub ciągów znaków. Najczęściej wykorzystywany do zabezpieczania haseł. Opiszę to narzędzie przy innej okazji.
- ansible-galaxy - narzędzie pozwalające na dostęp do repozytorium ról. Także opiszę je później.
- ansible-doc - umożliwia podgląd listy zainstalowanych modułów, pluginów oraz ich dokumentacji.
- ansible-pull - pozwala na ściągnięcie na maszynę (najczęściej inną niż control node) list zadań (playbooków) z systemu kontroli wersji. Obsługiwane obecnie to  git, subversion, mercurial (hg) i bazaar (bzr). Domyślny jest git.



[Powrót na stronę główną](https://moarsoft.github.io/catg-site/)
