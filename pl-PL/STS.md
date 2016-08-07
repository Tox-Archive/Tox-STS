Zarys Jednolitego Standard Tox v.0.0.2
===

###Zidentyfikacja Użytkowników
- Użytkownicy wymieniają ciąg 76 charakterów który zawiera klucz publiczny, wartość nospam, i checksum do użycia jako identyfikator. To powinno się nazywać **Tox ID**. Warianty jak na przykład "Identyfikacja", "Adres Tox", lub inne są nie prawdziwe i nie spełniają wymagania STS.
- Użytkownicy mogą sobie założyć **Imię** czy **Nazwę**, jako łatwo się przedstawiać do innych po zaakceptowania zaproszenia. Te nazwy są zmienne, tymczasowe, i nie mają żadnego wpływu na Tox ID. "Username", "Przezwisko", czy inne warianty nie spełniają wymagania STS.

###Dodawanie Znajomych
Są trzy pole tekstowe do wypełnienia przy dodawaniu innego użytkownika. Bezwzględnie na to że dwóch z nich są fakultatywne, jest wymagana że klienty przedstawiają wszystkie trzy. One powinno być oznakowane następująco:

###Norma Tox URI
Format Tox URI wygląda następująco: tox://. Klient powinien zaakceptować {CLIENT_NAME} tox://{PASSED}. Klient powinien wtedy sprawdzać czy to jest typowy Tox ID czy ID odkryty przez DNS. Jeżeli to jest standardowy Tox ID, klient powinien przedstawiać ID i pytać się czy powinien go dodać. Odpowieć zaprzeczająca powinna zamknąć klienta, a potwierdzająca dodać ID do kontaktów. Jeżeli ID ma być znaleziony przez DNS, to klient powinien to rozpoznać i przedstawiać ID i email. Klient się powinien tak samo zamknąć przy zaprzeczeniu, i dodać do kontaktów przy potwierdzeniu. Jeżeli ID nadane przez użytkownika jest nie prawdziwe, klient powinien to przedstawiać użytkownikowi i się zamknąć.

###Odkrycie DNS
ID odkryty przez DNS wygląda następująco: user@domain. Użytkownik nie powinien dodawać ten rodzaj ID inaczej niż by dodawali standardowy ID. Klient powinien sam rozpoznawać rodzaj ID, i domyśleć się który jest który. Przy dodaniu ID odkryty przez DNS, klient powinien znaleźć rekord DNS TXT dla wartości user._tox.domain, gdzie @ jest zmienione na _tox, pozwalając użycie sub-domen i jednocześnie potwierdzając że rekord jest Tox. Wartość rekordu Tox DNS wygląda v=version;id=Tox ID. Versja będzie zawsze tox1, i Tox ID zawsze będzie Tox ID użytkownika. Ze zwyczaju rekordy nie będą mieli spacji, ale klient powinien być przygotowany na wyjątki. Zalecane jest użycie DNSSEC, ale to nie jest wymagane.
