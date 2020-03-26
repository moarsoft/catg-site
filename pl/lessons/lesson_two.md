# Część druga: Yaml, INI i Jinja2.

## Wprowadzenie.

Zanim przejdziemy do dalszych zabaw z Ansible konieczne jest zapoznanie się ze składnią i formami zapisu z jakich przyjdzie korzystać.

Ansible korzysta z trzech form zapisu. Najważniejszym jest YAML ([https://yaml.org/](https://yaml.org), obok którego pojawia się także format "INI" ([https://en.wikipedia.org/wiki/INI_file](https://en.wikipedia.org/wiki/INI_file)). Trzecim formatem jest Jinja2 ([https://jinja.palletsprojects.com/en/2.11.x/](https://jinja.palletsprojects.com/en/2.11.x/)) czyli język opisu szablonów (**template**) znany większości osób używających Pythona.

Polecam zapoznać się z nimi dodatkowo samodzielnie ponieważ tutaj wspomnę tutaj tylko o podstawach koniecznych dla zrozumienia dalszych etapów zapoznawania się z Ansible.

## Format INI.

Znany z systemów Windows format używany głównie do zapisu konfiguracji jest prosty jak konstrukcja cepa. Oparty jest o schemat klucz=wartość (key=value) oraz na grupowaniu przy użyciu sekcji, które wyglądają tak: [sekcja1].

Czyli zawartość pliku w tym formacie, który zwykle ma (ale nie musi) rozszerzenie .ini wygląda mniej więcej tak:

	; some important comment
	server=my_server
	ip=127.0.0.1

	# hash can be a comment in some cases, Ansible for example
	[person]
	name=Jon Snow
	family=Starks
	perk="boring as hell"

## Format YAML.

Jest to język opisu struktur danych w celu ich późniejszej serializacji, który w założeniach ma być prostszy i bardziej czytelny niż inne formaty używane do tej pory jak XML i JSON.
Faktycznie, czysty YAML jest bardziej czytelny lecz zostało to okupione pewną wadą - czułością na błędy składniowe.
W YAMLu znaczenie ma niemal wszystko: ilość spacji przed i po wyrażeniu, inne białe znaki, użycie cudzysłowów i apostrofów, itd. 
W przypadku Ansible, gdzie YAML jest wykorzystywany także do tworzenia deskryptorów zadań, obsługi modułów i różnych innych
manipulacji także przy mieszanym użyciu składni Jinja2 a czasem nawet Pythona, prowadzi to do sporego bałaganu,
który przynajmniej z początku, może przyprawić o ból głowy. Co więcej wykrycie błędu w składni w jakiejś konkretnej linii pliku
Ansible nie koniecznie musi oznaczać, że on tam rzeczywiście wystąpił.

Jak wyglądają deskryptory zadań w YAML (wersja Ansible) opiszę gdy zajmiemy się nimi bezpośrednio. Teraz wystarczy krótki ogląd.

## Struktury danych w YAML.

Podstawową formą spotykaną w YAMLu jest **mapowanie klucz/wartość**, w Ansible taką strukturę nazywamy **słownikiem**
(ang. dictionary):

	key: value
	
Zwróćcie uwagę na spację po dwukropku. Jest ona konieczna, w przeciwnym razie wystąpi błąd.
Podobnie będzie jeśli postawimy spację po wartości.

Można zagnieżdżać mapowania:

	parent: {first: 1, second: 2, third: 3}

lub:

	parent: {
	    first: 1,
	    second: 2,
	    third: 3
	  }

lub, wersja używana w Ansible (preferowana) do której powyższe dwa sposoby zwykle są parsowane:

	parent:
	  first: 1
	  second: 2
	  third: 3

Ponownie. Zwróćcie uwagę na ilość "wcięć", są istotne!	

**Sekwencje** (albo inaczej, w Ansible, **listy**, ang. list) poprzedzane są myślnikiem "-":

	- first element
	- second element
	- third element

Znów, istotne są spacje po myślniku i na końcu.

Oczywiście sekwencje także można zagnieżdżać:

	- [first element, second element, third element]

Oraz używać kombinacji powyższych sposobów:

	key:
	  - first element
	  - second element
	  - third element
	  
	key2:
	  - first element
	  - second element
	  
	- 
	  first: 1
	  second: 2
	  third: 3
	  
	key3:
	  - list1:
	      - first element
		  - second element
		  - 
		    inside:
		      first: 1
		      second: 2
		      third:
			- first element
			- second element

I tak dalej.

## Wartości w YAML.

Skalary, czyli wartości mogą być zapisywane na wiele różnych sposobów. Podstawowe to te wymagające cytowania (quote)
i czyste (plain). Cytowanie pozwala na interpretację i wymaga wyłączania ("escape") elementów jeśli nie chcemy ich interpretować.
Działa to tak samo jak właściwie w każdym innym języku programowania:

Cudzysłowy interpretują:

	control: "\b1998\t1999\t2000\n"
	
Apostrofy nie interpretują:

    notcomment: '# this is not a comment'

## Typowanie danych w YAML.

Domyślnie w YAML wszystko jest ciągiem znaków (string), sekwencją (sequence) lub mapą (map) zaś interpretacja typu do 
jakiego należy wartość zależy od aplikacji, która ją przeczyta. Można jednak oznaczyć wartość jako konkretny typ przy użyciu 
specjalnych znaczników (tag) lub sposobu w jaki wartość zostanie zapisana jeśli chcemy wymusić konkretną interpretację.
Na przykład:

	canonical: 12345
	decimal: +12345
	octal: 0o14
	hexadecimal: 0xC

Po szczegóły odsyłam do dokumentacji YAMLa ([https://yaml.org/spec/1.2/spec.html](https://yaml.org/spec/1.2/spec.html)).

## Dokumenty w YAML.

Upraszczając. Początek dokumentu w YAML oznaczamy przy użyciu `---` , zaś koniec `...` . W jednym pliku można zawrzeć kilka dokumentów.

	---
	# one doc
	...
	---
	# second doc
	...
	
Tak naprawdę jest to trochę bardziej skomplikowane (znów, odsyłam do dokumentacji YAML po szczegóły), ale nam
wystarczy wiedza, że aby coś było zinterpretowane jako dokument YAML musi zaczynać się od `---`. 
Kończenie dokumentu przy użyciu `...` to dobra praktyka, ale nie jest to konieczne i brak nie spowoduje błędu parsera.


Polecam pobawić się trochę z YAMLem, żeby oswoić się ze składnią. Można użyć walidatora składni online
([http://www.yamllint.com/](http://www.yamllint.com/)) .

## YAML w Ansible.

YAML używany w Ansible ma kilka kruczków, na które należy uważać. 
Wynikają one ze specyfiki silnika i parsera Ansible, który używa elementów wspólnych z YAMLem do różnych celów. 
Na przykład "{" oznacza w YAMLu początek słownika, ale Ansible odwołuje się do zmiennych używając składni Jinja2 - {% raw %}{{ zmienna }}{% endraw %}. 
To wymusza każdorazowe cytowanie odwołań do zmiennych w Ansible:

	variable: {% raw %}{{ variable }}{% endraw %} # Error!
	variable: "{% raw %}{{ variable }}{% endraw %}" # Correct!
	
Co więcej dotyczy to całych wartości, które muszą być objęte cytowaniem:

	variable: "{% raw %}{{ variable }}{% endraw %}"/path/element # Wrong, error!
	variable: "{% raw %}{{ variable }}{% endraw %}/path/element" # Correct.
	
Jeśli zaś używamy cytowania, to znaki specjalne muszą zostać "wyłączone" (wyescapowane). 
Ogólnie jeśli nie chcemy, żeby coś było zinterpretowane przez Ansible "domyślnie" należy użyć cytowania. 
Przykładowo wartość "boolowska" może być zapisana różnie:

	boolean: yes
	boolean: False
	boolean: 0
	
Jeśli chcemy użyć np słowa "False" trzeba je zacytować:

	non-boolean: "False"

Tu polecam zapoznanie się z dokumentacją Ansible odnośnie YAMLa: [https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)

## Jinja2.

Znany pythonowcom język znaczników używanych w szablonach pozwala na dynamiczne uzupełnianie zawartości.
Ponownie, jest to zbyt obszerny temat żeby wchodzić w szczegóły więc postaram się ograniczyć do niezbędnego minimum. 
Zainteresowanych odsyłam do dokumentacji: [https://jinja.palletsprojects.com/en/2.11.x/](https://jinja.palletsprojects.com/en/2.11.x/)

W Ansible silnik i składnię Jinja2 używa się w dwóch sytuacjach. 
Pierwsza jest dokładnie taka, do jakiej ten język zaprojektowano, czyli do
dynamicznego uzupełniania treści szablonów (templates). 
Są one zwykłymi dokumentami tekstowymi (zwykle z rozszerzeniem .j2), w których znaczniki Jinja2 wskazują
gdzie i w jaki sposób zmienić ich zawartość.

Drugim miejscem użycia Jinja2 są deskryptory zadań gdzie elementy Jinja2 są używane do operacji na danych i manipulowania nimi.

## Szablony.

Szablonem (**template**) może być każdy tekst. O szablonach więcej napiszę w dalszych częściach,
na razie wstarczy wiedzieć, że przy pomocy Jinja2 można w oparciu o nie dynamicznie generować pliki z odpowiednią zawartością
wstawioną we właściwe miejsca.

## Ansible i Jinja2

Jinja2 udostępnia w Ansible swoje wyrażenia do wykorzystania także w deskryptorach zadań. 
Dzięki temu w Ansible w ogóle możemy odwoływać się do zmiennych poprzez:

	{% raw %}{{ variable }}{% endraw %}
	
Co więcej, dzięki tzw filtrom, możemy tymi zmiennymi manipulować.

	{% raw %}{{ some_string | lower }}{% endraw %} # convert string to lowercase
    {% raw %}{{ some_variable | default(7) }}{% endraw %} # Set default value to 7 if variable not defined

Daje to właściwe nieograniczone możliwości.
