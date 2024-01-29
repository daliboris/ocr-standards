# ORBIS PICTUS

## Anotace grafiky

### Dosavadní diskuze

> Když si uživatel v systému vyhledá nějaký obrázek a proklikne se přes něj na konkrétní stránku v dokumentu, tak si myslím, že by byla zajímavá a užitečná funkcionalita, kdyby se mu např. vysvítily řádky, které s obrázkem souvisí (popisují obrázek, nebo se na něj odkazují).
>
> Ještě mě napadlo jedno řešení, jak provázat odkazy/aluze na obrázek, které se vyskytují v textu, a to pomocí `<StructureTag ID="rtf.pohreb-kanarka-001" TYPE="Reference" LABEL="RelatedToFigure" DESCRIPTION="Pohřeb kanárka"/>`
>
> Zavedl bych `@LABEL` s hodnotou `RelatedTo...`, tj. `RelatedToFigure`, `RelatedToTable`, `RelatedToFormula` ap.
> V samotném textu by se na tento tag odkazovalo pomocí `@TAGREFS`.
> Takhle bychom zcela využili ALTO, nemuseli spoléhat na data ve vlastním formátu.
>
> jestli ta definovaná propojení jsou správně - jsou definovány dva elementy `<StructureTag>` (1. pro popis obrázku, 2. pro spojení obrázku s řádky textu), obrázek má na oba Tagy odkaz v `@TAGREFS` a příslušný řádek má taktéž v `@TAGREFS` odkaz na propojovací Tag.
>
> Co mi tam teď chybí, je informace, jak se to propojí s tím obrázkem. Chápu, že řádek textu se propojí se StructureTag pomocí `@TAGREFS` a `@ID`, ale jak je řešeno propojení `<StructureTag>` s obrázkem, taky přes `@TAGREFS` elementu `<Illustration>`? Pokud ano, pak by v `<Illustration>` byly dva odkazy: první na metadata obrázku (popis, klíčová slova, ...) a druhý na propojovací `<StructureTag>` k řádkům?

### Navrhované řešení

- Atribut `@TAGREFS` umožňuje přidat k obsahovému prvku/elementu, který u sebe má tento atribut, *další informace* pomocí odkazů na definované označení (elementy typu Tag).
  - obsahový element = element, který zachycuje textový či netextový obsah na stránce  
- **Obrázek** je součástí sazby (stránky), vytváří její vzhled, proto jde o popisný element `<LayoutTag LABEL="Graphic | Linedrawing | Photo | Map"/>`.
  - popisný element = element, jehož součástí není obsah na stránce, ale přidaná informace o tomto obsahu
- Pokud **obrázek** obsahuje podstatné informace, [doporučení](https://altoxml.github.io/documentation/use-cases/tags/ALTO_tags_usecases.html) navrhuje použít `<StructureTag TYPE="Functional" LABEL="Illustration" DESCRIPTION="Pohřeb kanárka"><XmlData> </XmlData></StructureTag>`.
- Pokud **text** obsahuje odkaz na obrázek (typu `viz obr. 2`), použije se `<StructureTag TYPE="Functional" LABEL="FigureReference" DESCRIPTION="obr. 2"/>`.
  - > Pokud je myšlenková hodnota **obrázku** vysoká, lze ji zachytit pomocí strukturální značky.
- **Popisek obrázku** je součástí strukturace obsahu, proto jde o `<StructureTag TYPE="Functional" LABEL="FigureCaption"/>`.
  - Popravdě řečeno je příklad `<StructureTag TYPE="Functional" LABEL="FigureCaption"/>` uveden jenom v [doporučeních](https://altoxml.github.io/documentation/use-cases/tags/ALTO_tags_usecases.html), žádný další výskyt na GitHubu jsem nenašel.
- Odkazování na obrázek, tj. text vztažený k obsahu obrázku, nepatří do žádného z pevně daných prvků (vzhled stránky, strukturace obsahu, role osoby při vzniku textu, rozpoznaná entita), proto navrhuju používat element `<OtherTag TYPE="Content" LABEL="RelatedToFigure" DESCRIPTION="pohřbili toho kanárka" />`, kde `@DESCRIPTION` obsahuje text, na jehož základě PERO OCR přiřadil danou pasáž k obrázku.
  - proč nenavrhuju `<StructureTag TYPE="Functional" LABEL="FigureReference"/>`: strukturní prvky mají ustálenou podobu, která je vyčleňuje z okolního textu/obsahu (titulní strany, indexy, nadpisy, popisky ap.), popř. ustálenou formu (např. `viz obr. 2`), zatímco „volná“ formulace v textu (která se nějakým způsobem vztahuje k obrázku ap.) tyto atributy nemá
  - výjimkou může být případ, kdy se text vztahující k obrázku rovná rozpoznané entitě (na obrázku je člověk, firemní značka, stavba ap.), pak se místo `<OtherTag>` použije `<NamedEntityTag>`.
  - proč **nenavrhuju** použít u elementu `<Illustration>` odkaz na více tagů v atributu `@TAGREFS`: pokud bude u `<Illustration>` odkaz na `<LayoutTag LABEL="Illustration">`, znamená to, že prvek na stránce (ilustrace) reprezentuje obrázek s nějakým tématem; pokud by tam byl odkaz i na `<OtherTag TYPE="Content" LABEL="RelatedToFigure">`, znamenalo by to, že prvek na stránce (ilustrace) reprezentuje zmíněný text, což ale není pravda
    - jak by v případě dvou odkazů vypadala identifikace dodatečných informací (při zpracování stránky ALTO pro zobrazení):
      - u prvku `<Illustration>` by byly identifikovány 2 odkazy na elementy typu `<XyzTag>`: jeden by popisoval, co je vidět na obrázku; druhý by definoval text, který s obrázkem (nějak) souvisí; aplikace by na základě elementů (`<StructureTag LABEL="FigureCaption">` a `<OtherTag LABEL="RelatedToFigure">`) určila, o jaký typ informace se jedná a v prvním případě nabídla např. tooltip s popisem obsahu obrázku (v češtině/angličtině) + musela by vyhledat, u jakých elementů v textu (typu `<TextBlock>`, `<TextLine>`, `<String>`) se vyskytuje druhý identifikátor a ty např. zvýraznit
      - to by ale znamenalo zpracovat stejným způsobem popisek obrázku, protože popisek má k obrázku ještě blíž než volný text – takový způsob ale ALTO ve svých příkladech nikde neuvádí (nenašel jsem ale ani žádné další příklady na popisek obrázku řešený pomocí `@TAGREFS`)
      - použití dvou a více identifikátorů jako hodnoty pro atribut `@TAGREFS` se v [doporučeních](https://altoxml.github.io/documentation/use-cases/tags/ALTO_tags_usecases.html) uvádí pro případy, kdy se v souvislém textu překrývá více rozpoznaných entit, např. v označení *Parlament České republiky* je *Parlament České republiky* označením instituce, přičemž *České republiky* zároveň označuje státní celek, proto bude u posledních dvou slov v `@TAGREFS` odkaz na více značek (jde o značky stejného typu), takže u obrázků (`<Illustration>`) bych si dovedl představit několik odkazů v rámci `@TAGREFS`, ale nesly by stejnou informaci, např. identifikaci tématu obrázku pomocí dvou různých systémů (např. PERO OCR a Azure Vision)

#### Zachycení vztahu mezi netextovými a textovými prvky na stránce

Jedná se např. o vztah mezi obrázkem a popiskem nebo mezi obrázkem a textem odstavce, který má k obrázku (jeho obsahu) relevantní vztah.

V systému ALTO vztah mezi prvky vyjadřuje návazností (lineárním řazením) elementů, které spolu souvisí: pokud bude obrázek hned za/před popiskem, patří tyto prvky k sobě. Pokud se nelze spoléhat na bezprostřední pořadí prvků na stránce, dá se použít atribut `@IDNEXT`, který obsahuje identifikátor prvku, který je další v pořadí, nebo pomocí elementů v sekci `<ReadingOrder>` (kombinace prvků `<OrderedGroup>`, popř. `<UnorderedGroup>` a jejich podřízeného prvku `<ElementRef>`). Toto uspořádání se ale týká jenom textu, nepočítá s návazností textových a netextových prvků.

Pro explicitní zachycení vztahů mezi textovými a netextovými prvky nevidím v současném standardu ALTO žádný mechanizmus, proto navrhuju využít metadata generovaná systémem PERO OCR, která jsou součástí elementů typu `<XyzTag>`. V těchto metadatech může být i odkaz na další související prvky.

##### Výhody

- není potřeba se spoléhat na formát ALTO,
- možná bude stejný způsob možné použít i u jiných formátů

##### Nevýhody

- při práci s formátem ALTO je potřeba jít do interních dat PERO OCR
- PAGE XML s *interními* metadaty (např. PERO OCR) neumí pracovat (tj. nelze je vložit do dokumentu ve formátu XML), bude se muset najít jiný způsob

##### Ukázka

```xml
<Tags>
  <!-- Samotný obrázek -->
  <StructureTag ID="fig.illustration_001" TYPE="Structural" LABEL="Illustration" DESCRIPTION="Obrázek dětí v krajině, které pohřbívají kanárka">
    <XmlData>
      <pc:pero-ocr confidence="0.92" xml:id="fig.illustration_001" type="figure" related="#fc.illustration_001 #crtf.illustration_001">
        <pc:label xml:lang="cs">Obrázek dětí v krajině, které pohřbívají kanárka</pc:label>
        <pc:label xml:lang="en">Picture of children in the landscape burying a canary</pc:label>
        <pc:keywords xml:lang="cs">obrázek, děti, vůz, les, pohřeb</pc:keywords>
        <pc:keywords xml:lang="en">picture, children, cart, forest, funeral</pc:keywords>
      </pc:pero-ocr>
    </XmlData>
  </StructureTag>
  <!-- Popisek obrázku -->
  <StructureTag ID="fc.illustration_001" TYPE="Functional" LABEL="FigureCaption" DESCRIPTION="Pohřeb kanárka">
    <XmlData>
      <pc:pero-ocr confidence="0.90" xml:id="fc.illustration_001" type="caption" related="#fig.illustration_001" />
    </XmlData>
  </StructureTag>
  <!-- Pasáž textu, která s obrázkem nějakým způsobem souvisí (netýká se označení obrázku nebo odkazu na něj) -->
  <OtherTag ID="crtf.illustration_001" TYPE="Content" LABEL="RelatedToFigure" DESCRIPTION="děti její že bez kaše">
    <XmlData>
      <pc:pero-ocr confidence="0.83" xml:id="crtf.illustration_001" type="content" related="#fig.illustration_001" />
    </XmlData>     
  </OtherTag>
</Tags>
```

### K dořešení

K rozdílu mezi pojmy obrázek (`figure`) a ilustrace (`illustration`) ([zde](https://graphicdesign.stackexchange.com/a/126174)):

> Typically, *figures* are directly referenced in the text or are visual implementation of what is being explained in the text. i.e. "See figure x". Figures are used to better explain through visuals or to increase retention of ideas/concepts being explained within text. They can be "general" figures, such as a "Tips" icon whenever a tip is in the text.
> *Illustrations* generally have no direct connection to the text. They may be loosely related in concept but nothing is directly connected to the text and the illustration could be removed without degrading the retention of the text.
> Loosely.. one could state that all figures are illustrations, but not all illustrations are figures.

Obrázek je součástí textu, sdělení, s textem může být propojen pomocí odkazů. Dojde-li k jeho vypuštění, obsahová stránka textu tím utrpí. Ilustrace je z tohoto pohledu *nadbytečný* doprovod k samotnému textu; může text reflektovat, ale text je pochopitelný i bez ilustrace.

ALTO rozlišuje mezi prvky strukturními a sazebními (využívanými pro sazbu), takže obrázky lze považovat za strukturní, ilustrace za prvky sazební. (I když z [příkladů](https://altoxml.github.io/documentation/use-cases/tags/ALTO_tags_usecases.html)) to není úplně zřejmé, např. mapy, fotografie nebo grafika (grafy?)

Bude možné v projektu Orbis Pictus při klasifikaci netextových prvků rozlišovat mezi obrázky a ilustracemi? Např. na základě pomocných prvků jako je formalizovaný popisek obrázků (číslování, návěští typu `Obr.`, odkazy na *identifikátor* obrázku v textu).

Pokud ne, navrhuju pro oba případy používat element `<LayoutTag LABEL="Illustration">`, v opačném případě rozlišovat `<StructureTag TYPE="Functional" LABEL="Figure">` pro obrázky a `<LayoutTag LABEL="Illustration">` pro ilustrace.
