zakaj to ne deluje dolzina = re.findall(r'<li class="ipc-inline-list__item" role="presentation">\w+</li>', film.decode_contents())ce imamo tako vrsto podatkov <li class="ipc-metadata-list-summary-item"><div class="ipc-metadata-list-summary-item__c"><div class="ipc-metadata-list-summary-item__tc"><span aria-disabled="false" class="ipc-metadata-list-summary-item__t ipc-btn--not-interactable"></span><div class="sc-fc35a1ef-1 lmHCrT dli-parent"><div class="sc-fc35a1ef-0 hTMtRz"><div class="sc-1f7bc70-0 gspVJR dli-poster-container"><div class="ipc-poster ipc-poster--base ipc-poster--media-radius ipc-poster--wTvoj regex ne deluje iz dveh razlogov:
1. \w+ ne pokriva presledkov
\w ujema samo črke, številke in podčrtaj — ne pa presledkov. Če je dolžina filma zapisana kot npr. 1h 55m, se \w+ ustavi na 1h in ne zajame celotnega niza, ker je vmes presledek. Tudi če bi se regex sicer ujel, bi dobil samo del besedila.
2. Regex na HTML-ju je krhek
Tvoj vzorec zahteva natančno ujemanje atributov (class="ipc-inline-list__item" role="presentation" v točno tem vrstnem redu, brez dodatnih atributov ali presledkov). Če IMDb doda še kak atribut (npr. data-testid) ali spremeni vrstni red atributov, se regex sploh ne ujame, čeprav je element vsebinsko enak. Ker uporabljaš BeautifulSoup (film.decode_contents()), bi bilo veliko bolj zanesljivo uporabiti njegove metode namesto regexa nad surovim HTML stringom.
Predlog popravka — uporabi BeautifulSoup namesto regexa:
pythonitems = film.find_all("li", class_="ipc-inline-list__item")
dolzina = [item.get_text(strip=True) for item in items]
To bo poiskalo vse <li> elemente s tem razredom (ne glede na vrstni red/dodatne atribute) in izvleklo besedilo, ne glede na presledke v njem.
Če pa res želiš popraviti regex (manj priporočljivo, a za hiter test):
pythondolzina = re.findall(r'<li class="ipc-inline-list__item" role="presentation">([^<]+)</li>', film.decode_contents())
Tu sem dodal [^<]+ namesto \w+ (zajame vse do naslednjega <, vključno s presledki), in oklepaje za zajem vsebine namesto celega taga.
Lahko pošlješ tudi konkreten HTML fragment za to <li> (tisti, ki vsebuje dolžino filma), pa ti povem natančno, zakaj se ne ujema v tvojem primeru.kako  preprecim, da bi se naslovi zapisali z znaki kot so &amp;
naslov = re.sub(r"\d+. ", "", neobdelan_naslov) 
neodbelan_naslov = Deadpool & WolverineProblem je, da BeautifulSoup včasih vrne HTML entitete (&amp;, &nbsp;, itd.) namesto pravih znakov. Rešitev je html.unescape():
pythonimport html

naslov = html.unescape(re.sub(r"\d+\. ", "", neobdelan_naslov))
# "Deadpool &amp; Wolverine" → "Deadpool & Wolverine"
Ali pa, če naslov dobiš iz BeautifulSoup elementa, uporabi .get_text() — ta že avtomatično dekodira entitete:
pythonneobdelan_naslov = element.get_text(strip=True)  # vrne že čist tekst
Majhna opomba — imaš tudi napako v regexu:
pythonr"\d+. "   # pika pomeni "katerikoli znak" — napaka
r"\d+\. "  # escaped pika — pravilno

Pogovor je bil uporabljen samo za to, da se popravi zapis v .txt datoteke pri posebnih znakih kot je &