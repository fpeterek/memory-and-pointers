# Paměť a pointery

Při programování využíváme dva druhy paměti, stack (*zásobník*), a heap (*halda*).

Stack je spravován automaticky a přístup do něj je velmi rychlý, ale objem paměti
alokované na stacku musí být znám již během kompilace. 

Paměť v heapu naopak musí být spravována manuálně a přístup k ní je pomalejší,
výhodou ovšem je, že množství paměti, jež chceme alokovat, nemusíme znát předem,
a že se nám obsah paměti nesmaže, dokud sami nechceme.

## Stack

Stack je často označován také jako `callstack`, a to z velmi dobrého důvodu --
mechanismus stacku je využíván při volání funkcí. Vždy, když voláme funkci, vytvoří
se na stacku rámec (*stack frame*). V něm se uloží pozice, odkud byla funkce volána,
aby program věděl, kde má pokračovat po návratu z funkce, nebo argumenty, které byly
funkci předány. Dále se alokuje přesně tolik paměti, kolik funkce využívá pro lokální
proměnné.

<details>
    <summary>VLA</summary>

    Novější standardy jazyka C podporují také VLA - variable length arrays. Kompilátory
    implementující VLA umožňují specifikovat délku pole na stacku až za běhu programu,
    což částečně vyvrací předchozí tvrzení. Je ovšem třeba zmínit, že ne všechny kompilátory
    VLA podporují. Pokud konkrétní kompilátor VLA nepodporuje, bude definovat makro 
    `__STDC_NO_VLA__`.
</details>

Při návratu z funkce je stack frame zahozen. Společně s ním jsou také automaticky zahozeny
všechny proměnné definované v dané funkci.

<details>
    <summary>Velikost stacku</summary>

    Linker `ld` defaultně nastavuje velikost stacku na 8 MB. Velikost stacku však lze
    nastavit jak při linkování, tak dynamicky syscallem, v Linuxu např. pomocí funkce
    [setrlimit](https://linux.die.net/man/2/setrlimit).
</details>

Přístup k proměnným na stacku je rychlý, protože k nim program přistupuje přímo -- ví, kde
se paměť nachází a může k ní rovnou přistoupit.

### Využití paměti na stacku

Proměnné jsou automaticky alokovány na stacku. Chceme-li tedy využít stack, stačí nám
definovat proměnnou.

```c
int variable;
int array[100];
```

Tyto proměnné však budou zahozeny při opouštění funkce. Chceme-li data zachovat, musíme
je zkopírovat a vrátit z funkce pomocí klíčového slova `return`. Podobně však musíme data
kopírovat, chceme-li hodnotu proměnné předat funkci, kterou voláme -- víme totiž, že funkce
nemá přístup k lokálním proměnným funkce, ze které byla zavolána, ale pouze k proměnným
vlastním a k proměnným globálním.

## Pointery

Ne vždy však chceme proměnné kopírovat. Někdy je objem dat příliš velký a kopírování by
trvalo dlouho, jindy zase můžeme chtít lokální proměnnou modifikovat ve volané funkci.
V takovém případě můžeme funkci jako argument předat ne kopii dat, ale jejich adresu.
Tehdy budeme místo dat kopírovat pouze jejich adresu, která má na moderních počítačích
většinou 64 bitů.

Adresu získáme pomocí unárního operátoru `&`.

```c
// Definujeme proměnnou
int variable = 10;

// Získáme adresu dat, které proměnná reprezentuje
&variable;
```

Této adrese se říká `pointer`. Standard jazyka C se snaží být nezávislý na hardwaru, aby
bylo možné jazyk C korektně implementovat kdekoliv, pro zjednodušení však postačí, když
si prozatím představíme pointer jako 64 bitové číslo, které počítači říká, kde v paměti
se nachází naše data.

Pointery, stejně jako všechny ostatní hodnoty v jazyce C, mají své datové typy. Datový
typ pointeru získáme tak, že za datový typ, na jehož instanci pointer odkazuje, připíšeme
asterisk. Pointer na `int` je tedy `int*`. Pointer na `float` bude `float*`. Pointer
na pointer bude `int**` -- pouze jsme připsali hvězdičku za typ pointeru na int. Pointer
na nespecifikovaná data je `void*`, případně `char*`, v takovém případě však `char` považujeme
za byte, ne nutně ASCII znak. Protože C datový typ pro byte nemá, používá se pro reprezentaci
jednotlivých bytů většinou datový typ `char`.

Pointery definujeme podobně jako všechny ostatní proměnné v jazyce C.

```c
// Definujeme proměnnou
int variable = 10;

// Deklarujeme pointer
int* pointer;

// Přiřadíme pointeru adresu proměnné variable

pointer = &variable;

// Deklaraci proměnné a přiřazení hodnoty lze u pointeru provést stejně
// jako u každé jiné proměnné

int* ptr = &variable;

// Definujeme pointer na pointer
int** ptrToPtr = &ptr;
```

Pro přístup k hodnotě, na kterou pointer odkazuje, využíváme unární operátor `*`.

```c

int x = 10;

int* ptr = &x;

// Nesmysl, hodnota proměnné ptr je adresa a nedává smysl s ní pracovat jako s číslem
int incorrect = ptr;

// Přiřadit adresu adrese samozřejmě není žádný problém
int* ptr2 = ptr;

// K hodnotě, na kterou se pointer odkazuje, přistoupíme operátorem indirekce
int xCopy = *ptr;

// Operátor indirekce nám vrátí hodnotu, na kterou se pointer odkazuje. Pro int* to
// je int. Nad intem poté přirozeně dává smysl provádět aritmetické operace
int xSquare = *ptr * *ptr2;
```

Jak bylo zmíněno výše, pointery lze využít například pokud chceme dovolit jiné funkci
modifikovat lokální proměnnou.

```c

// Funkce, která přijímá pointer
void double_in_place(int* data) {
    data = 2 * *data;
}

int main(int argc, const char* argv[]) {

    int data = 10;

    // Předáme funkci adresu
    double_in_place(&data);

    // Vypíše 20
    printf("%d", data);

    return 0;
}
```

Nakonec je třeba zmínit, že přístup k proměnným na stacku je rychlý -- a tedy je
rychlý přístup k pointerům, které máme uložené na stacku. Přístup k hodnotě,
na kterou se pointer odkazuje, je ovšem pomalejší. Počítač musí nejprve načíst
hodnotu pointeru, adresu, a to zvládne velmi rychle. Poté však musí adresu interpretovat
a najít správné místo v paměti, aby našel data, na které se naše adresa odkazuje. A právě
toto vyhledávání v paměti je pomalé.

### Null Pointer

Chceme-li znázornit absenci hodnoty, můžeme využít tzv. null pointer. Null pointer je pointer
ukazující na adresu `0x0`. Pokud bychom potřebovali náš pointer nastavit na null, můžeme jej
jednoduše nastavit na hodnotu `0`. Místo hodnoty `0` se často využívá makro `NULL`, které je 
sice definováno jako 0, programátorům však automaticky evokuje pointer a kód je tak čitelnější.

```
// Ekvivalentní
int* ptr = 0;
int* ptr2 = NULL;

// Chceme-li zkontrolovat, že se pointer neodkazuje na null, stačí jej porovnat s hodnotou makra NULL
// nebo přímo s nulou

if (ptr != NULL) {

}

// Nebo, protože 0 je považována za nepravdu

if (ptr) {
    // Provede se pouze pokud ptr není roven 0x0
}
```

Pozor -- přístup k hodnotě na adrese nula je považován za chybu a vede k pádu programu, protože
přistupujeme k 'neexistující hodnotě.'

## Heap

Zatímco stack je třeba zvětšovat manuálně, ale paměť alokovaná na stacku je spravována
automaticky, paměť alokovanou na heapu musíme spravovat manuálně, ovšem, poněkud ironicky,
heap se rozšiřuje automaticky podle potřeb programu a možností operačního systému.

Jelikož se paměť na heapu neuvolňuje automaticky, používáme heap, abychom si alokovali
paměť, která se nám nezahodí při zahození stack framu. Paměť na heapu se uvolní teprve
když o to manuálně zažádáme, případně po doběhnutí programu. Není nijak vázána na mechanismus
callstacku. Na heap tedy alokujeme paměť, kterou nechceme zbytečně kopírovat, podobně jako
bychom to museli udělat s pamětí na stacku. Dále na heapu ukládáme objemnější data, u kterých
by hrozilo, že velmi rychle zaplní celý stack.

### Využití paměti na heapu

K alokaci paměti na heapu slouží funkce `malloc`. Funkce `malloc` přijímá jeden jediný argument
-- kolik bytů má alokovat. `malloc` interně získá požadovanou paměť -- a v tom momentě nastává
problém. Paměť je alokovaná na nějakém náhodném místě v paměti počítače. Jak k ní přistoupíme?
Jak nám `malloc` řekne, kde se nachází naše paměť? Adresou. `malloc` po alokaci paměti vrátí
adresu -- pointer na místo v paměti.

```c
// Alokujeme 20 bytů a uložíme si adresu alokované paměti
char* ptr = malloc(20);

// Velikost paměti nemusí být známa předem, stačí ji vypočítat těsně před alokací
char* ptr2 = malloc(get_size());
```

S alokovanou pamětí poté můžeme nakládat, jak chceme, neměli bychom ale ztratit pointer na tuto
paměť, jinak paměť zůstane alokována a nebudeme ji moct využít ani uvolnit. Této situaci se říká
memory leak a je důsledkem nesprávného zacházení s pamětí. Na memory leaky trpělo například
Dragon Age: Origins, které bylo potřeba pravidelně restartovat, jinak hrozilo, že se hra nejprve
začne zpomalovat, až nakonec spadne úplně.

Abychom neztratili pointer na paměť, je třeba pointer vrátit z funkce, která jej alokuje, případně
pomocí pointeru na pointer modifikovat již existující pointer odjinud. Kvůli čitelnosti kódu bychom
měli preferovat první řešení.

```c
char* alloc_memory() {
    // Alokujeme 1GB paměti
    char* mem = malloc(1024 * 1024 * 1024);

    process(mem);

    // Vrátíme pointer na paměť
    return mem;
}

int main(int argc, const char* argv[]) {

    // Adresu alokované paměti si uložíme do proměnné, abychom s ní mohli dále pracovat
    char* mem = alloc_memory();

    ...

    return 0;
}
```

Paměť uvolníme pomocí funkce `free`. Funkce `free` přijímá jeden argument -- adresu paměti, kterou
chceme uvolnit. `free` bychom měli na každý blok alokované paměti volat přesně jednou, v momentě,
kdy už paměť nepotřebujeme. Paměť se tak uvolní pro další využití programem.


```c
void incorrect() {
    char* mem = malloc(1000);
    // Při opouštění funkce ani nevrátíme pointer na alokovanou paměť, ani paměť neuvolníme,
    // dochází k leaku
    return;
}

void correct1() {
    char* mem = malloc(1000);
    // Paměť uvolníme, když už ji nepotřebujeme, nedochází k leaku
    free(mem);
}

char* correct2() {
    // Paměť alokujeme a vrátíme její adresu
    // K leaku momentálně nedochází, paměť totiž můžeme uvolnit později podle potřeby
    // K leaku by však došlo, pokud bychom paměť neuvolnili v programu později
    return malloc(1000);
}
```

Funkce `malloc` pouze alokuje blok souvislé paměti, nijak už ale nediktuje, jak máme s pamětí
pracovat. Jedním z obvyklých use-casů je alokace paměti pro instanci datového typu. Jelikož
`malloc` netuší, jaká data chceme ukládat, musíme sami argumentem specifikovat, kolik bytů
potřebujeme pro instanci požadovaného datového typu.

Kolik bytů instance konkrétního typu potřebuje lze zjistit pomocí operátoru `sizeof`. Argumentem
`sizeof` je datový typ. Při zjišťování velikosti je vhodné využít operátor `sizeof` a nesnažit se
velikost odhadnout. Odhad totiž nemusí být ani korektní, ani aktuální. Například na většině moderních
strojů má `int` 4 byty. Standard jazyka C však nikde konkrétní velikost `int`u nespecifikuje. Standard
jazyka C specifikuje pouze minimální velikost `int`u, kterou uvádí jako 2 byty. Tvrdit tedy, že `int`
má 4 byty, není korektní, přenosné, a ani future-proof řešení. Podobně problematické můžou být velikosti
struktur. Velikost struktury nejenže nemusí být kvůli zarovnání rovna součtu velikostí svých atributů,
ale také se může změnit, když dané struktuře přidáme, odebereme nebo změníme atribut. Pokud
bychom velikost struktury odhadovali sami, museli bychom při každé změně definice struktury upravit také
všechny zmínky její velikosti. Oproti tomu operátor `sizeof` se na definici struktury dokáže podívat sám.
A jelikož velikost struktury je známá již při kompilaci, operátor `sizeof` nemá vůbec žádný dopad na dobu
běhu programu.

```c
typedef struct {
    int a;
    long b;
    float c;
    double d;
} myStruct;

// Alokujeme paměť přesně pro jeden int
malloc(sizeof(int));

// Alokujeme paměť přesně pro jednu instanci myStruct
malloc(sizeof(myStruct));

// Nechceme ale způsobit memory leak, proto si adresu alokované paměti
// uložíme jako pointer na myStruct
myStruct* ptr = malloc(sizeof(myStruct));

// Protože malloc vrací void* a ne myStruct*, může si kompilátor stěžovat
// že nám neodpovídají datové typy
// V takovém případě stačí pouze přetypovat pointer vrácený `malloc`em

myStruct* ptr = (myStruct*)malloc(sizeof(myStruct));


// Samozřejmě můžeme alokovat paměť pro pointery a ukládat na heapu i pointery

int** ptrToPtr = malloc(sizeof(int*));
```

Také je důležité zmínit, že není vhodné přistupovat mimo hranice alokované paměti. V lepším případě
přistoupíme k paměti, která nenáleží programu, a operační systém program zabije, v horším případě špatně
interpretujeme nebo modifikujeme úplně jiná data, než ta, se kterými chceme pracovat, a způsobíme chybu,
která se může projevit na úplně jiném místě v programu, než kde byla způsobena. Proto není vhodné přistupovat
k náhodným adresám nebo mimo blok paměti alokovaný `malloc`em (případně jiným alokátorem).

## Pointer Arithmetics a pole

Častým use-casem pro paměť alokovanou na heapu jsou také homogenní pole (datová struktura array). Chceme-li
alokovat paměť pro pole velikosti `n`, stačí nám alokovat paměť pro `n`-krát prvek v poli.

```c
// Alokujeme paměť pro 1024 intů
malloc(1024 * sizeof(int));

// Neměli zapomenout uložit adresu
myStruct* ptr = malloc(1024 * sizeof(myStruct));

// A případně také přetypovat pointer
myStruct* ptr = (myStruct*)malloc(1024 * sizeof(myStruct));

// Alokace pole pointerů je stejná jako alokace jakéhokoliv jiného pole
int** ptrToPtr = (int**)malloc(1024 * sizeof(int*));
```

Pole tedy alokovat umíme, a umíme také přistoupit k prvku, na který pointer ukazuje pomocí operátoru
indirekce. Nyní je vhodné si připomenout, že `malloc` alokuje souvislý blok paměti a vrátí pointer
na začátek tohoto bloku. Pokud alokujeme paměť pouze pro jeden prvek, není třeba nic víc řešit.
Pokud však alokujeme paměť pro více prvků, musíme si uvědomit, že budou naše prvky následovat jeden
po druhém těsně za sebou.

<details>
    <summary>Nebo by alespoň měly</summary>

    Prvky teoreticky nemusí být uspořádané těsně za sebou. Jak si naši paměť uspořádáme a jak ji budeme
    interpretovat je čistě na nás. Kompilátor ale má zabudovanou speciální podporu pro homogenní pole,
    kteoru je vhodné využívat, a navíc by bylo zbytečně neefektivní alokovat prázdná nevyužívaná místa.
    Proto budeme předpokládat, že prvky v poli následují jeden po druhém a nejsou mezi nimi mezery.
</details>

Pro přístup k jinému než prvnímu prvku pole tedy musíme inkrementovat pointer na adresu požadovaného prvku.
Za tímto účelem standard jazyka C definuje tzv. *aritmetiku pointerů* (*pointer arithmetics*). Aritmetika
pointerů je rozdílná od aritmetických operací nad čísly. Nedává moc smysl násobit adresy mezi sebou, podobně
tak nedává smysl počítat kosinus adresy. Dává ale smysl říct, která adresa bude následovat současnou adresu.
Jaká bude adresa prvku vzdáleného tři prvky od současné adresy. Kolik prvků (validních adres) se nachází mezi
dvěma adresami. A právě k tomuhle účelu slouží aritmetika pointerů.

Nejjednodušším příkladem je inkrementace pointeru. Kompilátor zde využívá znalosti datového typu dat, na které
pointer ukazuje. Například pracujeme-li s `int*`, kompilátor ví, že `int*` ukazuje na `int`y a zná tedy velikost
prvků v poli. Máme-li ukazatel ukazující na `int`, a chceme-li získat adresu následujícího `int`u, stačí
k našemu pointeru přičíst `1`. Výsledkem přičtení 1 k našemu pointeru bude pointer, který ukazuje na `int`
'o jednu adresu dále'. Přičtením 1 nepřičteme k adrese 1, přičtením 1 adresu posuneme o jeden prvek dále.

Pro představu prozatím postačí si představit, že přičtením 1 k `int*` se k adrese, na kterou ukazuje `int*`,
automaticky přičtou 4 byty, protože kompilátor ví, že `int` na naší architektuře využívá 4 byty, a že adresa
na následující `int` musí být adresa 'o 4 byty dále.' Tato představa není úplně korektní, protože není
dostatečně abstraktní, ale reálně se na moderních počítačích děje přesně toto.

Logicky lze odvodit (nebo induktivně definovat), že pro přístup k `n`-tému prvku stačí přičíst `n`.

Jakmile pomocí aritmetiky pointerů získáme námi požadovanou adresu, stačí k prvku přistoupit pomocí operátoru
indirekce.

```c
int* array = (int*)malloc(10 * sizeof(int));
//         ^  ^           ^    ^
//         |  |           |    Velikost prvku, který chceme v poli ukládat
//         |  |           Kolik prvků chceme v poli ukládat
//         |  Přetypujeme výsledek malloc z void* na int*          
//         Pointer vracený mallocem uložíme do námi definované proměnné

// Přístup k prvnímu prvku v poli pomocí operátoru indirekce
*array;

// Adresa druhého prvku v poli -- adresa prvku následujícího první prvek
array + 1;

// Přístup k druhému prvku pomocí operátoru indirekce
*(array + 1);

// Přístup k n-tému prvku pomocí operátoru indirekce
*(array + n);

// Naplníme array hodnotami 0 až 9
for (int i = 0; i < 10; ++i) {
    *(array + i) = i;
}

// Nakonec nesmíme zapomenout paměť uvolnit
free(array);
```

I v případě aritmetiky pointerů můžeme používat zkratky, které známe z aritmetiky čísel.

```c
int* ptr = (int*)malloc(10 * sizeof(int));

// Všechny následující výrazy jsou ekvivalentní
ptr = ptr + 1;
ptr += 1;
++ptr;

// Samozřejmě lze získat také adresu předcházejícího prvku
ptr = ptr - 1;
ptr -= 1;
--ptr;
```

Dále dává smysl odčítat adresu od adresy. Tato operace dává smysl, pokud známe dvě adresy,
ale nevíme, kolik prvků je odděluje.

```c

// Funkce najde adresu prvního výskytu prvku item
// Funkce předpokládá, že se prvek v poli vyskytuje, v opačném případě program spadne
int* find_first(int* arr, int item) {
    for (; *arr != item; ++arr);
    return arr;
}

int main(int argc, const char * argv[]) {

    int* arr = get_arr();
    int* first_eight = find_first(arr, 8);

    // Známe adresu začátku pole a adresu první osmičky
    // Můžeme dopočítat, kolik prvků dělí první osmičku
    // od začátku pole -- index první osmičky
    first_eight - arr;


    return 0;
}
```

Přičítat dvě adresy k sobě smysl nedává.

Pro přístup k n-tému prvku také existuje syntaktický cukr, který je ekvivalentní využití
aritmetiky pointerů a operátoru indirekce, ale vypadá o trochu lépe. Tímto syntaktickým
cukrem je myšlen operátor přístupu k prvku pole `[]`. Využití je jednoduché -- za pointer
připíšeme hranaté závorky, ve kterých uvedeme index prvku, ke kterému chceme přistoupit.
Operátor `[]` pak pointer automaticky dereferencuje, jeho použitím tedy získáme přímo prvek,
na který pointer ukazuje, ne adresu prvku.

```c
int* arr = (int*)malloc(10 * sizeof(int));

// Výrazy jsou ekvivalentní, výraz vpravo však ve spoustě případů vypadá lépe
*(arr + n) == arr[n];
```

### Změna velikosti bloku alokované paměti

Pokud nám velikost alokovaného bloku nestačí -- často např. pokud potřebujeme zvětšit velikost
pole, případně pokud je pole zbytečně velké a můžeme jej zmenšit, můžeme využít funkci `realloc`
pro změnu velikosti bloku již alokované paměti. Funkce `realloc` přijímá dva argumenty, pointer
na již alokovanou paměť, a požadovanou velikost. Interně funkce `realloc` alokuje nový blok 
paměti, zkopíruje data ze starého bloku do nového, a starý blok uvolní. Nakonec vrátí adresu
nového bloku.

```c
// Alokujeme paměť
int* ptr = (int*)malloc(10 * sizeof(int));

// Zvětšíme velikost bloku z 10 intů na 20
ptr = (int*)realloc(ptr, 20 * sizeof(int));

// Nakonec nesmíme zapomenout paměť uvolnit
free(ptr);
```

## Rekapitulace

```c

int x;

// Získání adresy hodnoty proměnné x
&x;

// Uložení adresy do proměnné
int* ptr = &x;

// Operátor indirekce
*ptr;
*ptr = 10;

x == 10; // true

// Alokace deseti bytů

malloc(10);

// Uložení adresy alokované paměti

char* bytes = malloc(10);

// Uvolnění paměti

free(bytes);

// Alokace paměti pro deset intů

malloc(10 * sizeof(int));

// Přetypování pointeru vráceného mallocem na int*

(int*)malloc(10 * sizeof(int));

// Uložení získaného pointeru do proměnné

int* ints = (int*)malloc(10 * sizeof(int));

// Alokace paměti pro deset int*
malloc(10 * sizeof(int*));

// Alokace paměti pro deset int**
malloc(10 * sizeof(int**));

// Pointer na int
int*

// Pointer na pointer na int
int**

// Pointer na pointer na pointer na int
int***

// Pointer na float
float*

// Pointer na následující prvek
ints + 1;

// Pointer na n-tý prvek
ints + n;

// n-tý prvek
*(ints + n);
ints[n];
```

## Const pointery

Poněkud matoucí může být také `const`-ness pointerů. Zde je třeba si uvědomit, že rozlišujeme
pointery na konstantní data a konstantní pointery. Pointer na konstantní data je nekonstantní,
můžeme jej mutovat, ale nemůžeme mutovat data, na které pointer ukazuje. Konstatní pointer
mutovat nemůžeme, protože se jedná o konstatní proměnnou. 

```c

// Nekonstantní pointer na nekonstantní data, můžeme mutovat, jak se nám zlíbí
int* ptr = get_arr();
// Můžeme mutovat pointer
++ptr;
// I data
*ptr = 10;


// Nekonstantní pointer na konstantní data, můžeme mutovat pouze pointer, data ne
const int* ptr = get_arr();
++ptr;

// Konstatní pointer na nekonstantní data. Můžeme mutovat data, pointer ne
// Pointer můžeme zkopírovat a pracovat se zkopírovanou hodnotou, ale hodnotu
// proměnné ptr nezměníme
int* const ptr = get_arr();
*(ptr + 10) = 20;

// Konstatní pointer na konstantní data. Můžeme k datům přistupovat, ale nemůžeme mutovat
// ani pointer, ani data
const int* const ptr = get_arr();
*(ptr + 10);
```

## Příklady využití pointerů a dynamické alokace paměti

```c

// Alokace pole

int* get_int_array(const long size) {
    return (int*)malloc(size * sizeof(int));
}

// Linked List

typedef struct List {
    int head;
    List* tail;
} List;

List* cons(const int item, List* const tail) {
    List* head = (List*)malloc(sizeof(List));
    head->head = item;
    head->tail = tail;

    return head;
}

List* get_list(const int size) {

    List* list = 0;

    for (int i = 0; i < size; ++i) {
        list = cons(i, list);
    }
    
    return list;
}

// 2D Matrix

int** alloc_matrix(const int rows, const int cols) {

    // Matice má rows řádků, proto alokujeme rows pointerů
    // Každý pointer ukazuje na jeden řádek tabulky
    int** matrix = (int**)malloc(sizeof(int*) * rows);

    // Matice má cols sloupců, proto pro každý řádek alokujeme
    // pole o velikosti cols intů
    for (int i = 0; i < rows; ++i) {
        matrix[i] = (int*)malloc(sizeof(int) * cols);
    }

    return matrix;
}

// Přístup k prvku matice na a-tém řádku v b-tém sloupci
// matrix[a][b];
```

