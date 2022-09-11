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

Pokud nám 

## Pointer Arithmetics
