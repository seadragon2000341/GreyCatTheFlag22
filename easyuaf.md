In this challenge, we were given the following code with a hint that we should exploit the Use After Free (UAF) heap vulnerability. 
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define BANNER "NameCard Printing Service v0.1"
#define CAPACITY 128

void ezflag()
{
    system("cat ./flag.txt");
}

typedef struct person
{
    char name[24];
    int id;
    int age;
    int personal_num;
    int business_num;
} person;

typedef struct org
{
    char name[24];
    int id;
    void (*display)(struct org*, struct person*);
} org;

person* persons[CAPACITY];
org* orgs[CAPACITY];

void display1(org* org, person *person)
{
    puts("-------------------------------------");
    printf("*** Org:  %-8s  %d ***\n", org->name, org->id);
    printf("--- Name: %s\n", person->name);
    printf("--- ID:   %d\n", person->id);
    printf("--- Age:  %d\n", person->age);
    printf("--- Personal Contact:  %d\n", person->personal_num);
    printf("--- Business Contact:  %d\n", person->business_num);
    puts("-------------------------------------");
}

void display2(org* org, person *person)
{
    puts("-------------------------------------");
    printf("=== Org:  %-8s  %d ===\n", org->name, org->id);
    printf("+++ Name: %s\n", person->name);
    printf("+++ ID:   %d\n", person->id);
    printf("+++ Age:  %d\n", person->age);
    printf("+++ Personal Contact:  %d\n", person->personal_num);
    printf("+++ Business Contact:  %d\n", person->business_num);
    puts("-------------------------------------");
}

void display3(org* org, person *person)
{
    puts("-------------------------------------");
    printf("### Org:  %-8s  %d ###\n", org->name, org->id);
    printf(">>> Name: %s\n", person->name);
    printf(">>> ID:   %d\n", person->id);
    printf(">>> Age:  %d\n", person->age);
    printf(">>> Personal Contact:  %d\n", person->personal_num);
    printf(">>> Business Contact:  %d\n", person->business_num);
    puts("-------------------------------------");
}

void usage()
{
    puts("---------------------");
    puts("1. New person");
    puts("2. New org");
    puts("3. Delete org");
    puts("4. Print name card");
    puts("5. Exit");
    puts("---------------------");
}

void prompt()
{
    printf("> ");
}

void readstr(char* dest, int len)
{
    fgets(dest, len, stdin);
    if (dest[strlen(dest)-1] == '\n') dest[strlen(dest)-1] = 0;
}

int readint()
{
    char input[256]; fgets(input, 256, stdin);
    if (input[0] == '\n') input[0] = ' ';
    return strtol(input, 0, 10);
}

void new_person()
{
    person *res = (person*) malloc(sizeof(person));
    // printf("res: %p\n", res);

    while (1)
    {
        printf("ID (0-%d): ", CAPACITY-1);
        res->id = readint();
        if (res->id < 0 || res->id >= CAPACITY) puts("Invalid ID");
        else if (persons[res->id] != 0)
            printf("ID %d is already used by another person. Choose a different ID.\n", res->id);
        else break;
    }

    printf("Name (max 23 chars): ");
    readstr(res->name, 24);

    printf("Age: ");
    res->age = readint();

    printf("Personal Contact Number: ");
    res->personal_num = readint();

    printf("Business Contact Number: ");
    res->business_num = readint();

    persons[res->id] = res;
}

void new_org()
{
    org *res = (org*) malloc(sizeof(org));
    // printf("res: %p\n", res);

    while (1)
    {
        printf("ID (0-%d): ", CAPACITY-1);
        res->id = readint();
        if (res->id < 0 || res->id >= CAPACITY) puts("Invalid ID");
        else if (orgs[res->id] != 0)
            printf("ID %d is already used by another org. Choose a different ID.\n", res->id);
        else break;
    }

    printf("Name (max 23 chars): ");
    readstr(res->name, 24);

    int style;
    while (1)
    {
        printf("Style (1-3): ");
        style = readint();
        if (style >= 1 && style <= 3) break;
        puts("Invalid style.");
    }

    if (style == 1) res->display = display1;
    if (style == 2) res->display = display2;
    if (style == 3) res->display = display3;

    orgs[res->id] = res;
}

void delete_org()
{
    printf("ID (0-%d): ", CAPACITY-1);
    int id = readint();
    if (id < 0 || id >= CAPACITY) puts("Invalid ID");
    else if (orgs[id] == 0)
        printf("No org created with ID %d.\n", id);
    else
    {
        free(orgs[id]);
        printf("Deleted org %d.\n", id);
    }
}

void print_card()
{
    int org_id;
    while (1)
    {
        printf("Org ID (0-%d): ", CAPACITY-1);
        org_id = readint();
        if (org_id < 0 || org_id >= CAPACITY) puts("Invalid org ID");
        else if (orgs[org_id] == 0)
            printf("No org created with ID %d. Choose a different ID.\n", org_id);
        else break;
    }

    int person_id;
    while (1)
    {
        printf("Person ID (0-%d): ", CAPACITY-1);
        person_id = readint();
        if (person_id < 0 || person_id >= CAPACITY) puts("Invalid person ID");
        else if (persons[person_id] == 0)
            printf("No person created with ID %d. Choose a different ID.\n", org_id);
        else break;
    }

    org *o = orgs[org_id];
    person *p = persons[person_id];

    // printf("display func @ %p\n", o->display);
    o->display(o, p);
}

void reset()
{
    memset(persons, 0, sizeof(persons));
    memset(orgs, 0, sizeof(orgs));
}

void setup_io()
{
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

int main()
{
    // org* o1 = (org*)malloc(sizeof(org));
    // person* p1 = (person*)malloc(sizeof(person));
    // printf("o1: %p\n", o1);
    // printf("p1: %p\n", p1);
    // free(o1);
    // person* p2 = (person*)malloc(sizeof(person));
    // printf("p2: %p\n", p2);

    reset();

    setup_io();
    puts(BANNER);
    usage();

    // printf("%d %d\n", sizeof(org), sizeof(person));

    int opt;
    do {
        prompt();
        opt = readint();

        switch (opt)
        {
        case 1:
            new_person();
            break;
        case 2:
            new_org();
            break;
        case 3:
            delete_org();
            break;
        case 4:
            print_card();
            break;
        default:
            break;
        }
    } while (opt != 5);

    puts("Thanks for using this service. Please come again next time.");

    return 0;
}
```

Disas main
What is UAF?  Pointers in a program refer to data sets in dynamic memory. If a data set is deleted or moved to another block but the pointer, instead of being cleared (set to null), continues to refer to the now-freed memory, the result is a dangling pointer. If the program then allocates this same chunk of memory to another object (for example, data entered by an attacker), the dangling pointer will now reference this new data set. In other words, UAF vulnerabilities allow for code substitution.
(source: https://encyclopedia.kaspersky.com/glossary/use-after-free/)

Basically, we can allocate, fill, and then free an instance of the org structure by calling the new_org function and the delete_org function. Then we make another allocation (new_person this time), fill it, and then improperly reference the freed org structure. Due to how glibc's allocator works, the new person struct will actually get the same memory as the original org allocation, which in turn gives us the ability to control o->display pointer. This could be used to execute ezflag function to get the flag. 
We can first load the program in gdb and set a breakpoint towards the end of main. We then enter a new organization.
 
![image](https://user-images.githubusercontent.com/78403168/173195431-1378009b-d77f-487d-ab59-4a93dd0980c3.png)

Orgs is an array of pointers to structures org. We can see that orgs[0] (0x4044c0) contains the address of the org strcture that we just entered. Let us go to that address.
 ![image](https://user-images.githubusercontent.com/78403168/173195442-746ef024-e6ea-4852-aff6-b1811693e7ae.png)
![image](https://user-images.githubusercontent.com/78403168/173195446-e58aef28-3ab4-401a-9177-13236656868f.png)

 
We can see the data AAAA (41414141) that we entered. More crucially, we see that 0x4052c0 contains the address of the display function. Our goal is to overwrite that with the address of the ezflag function, which we can find out the address.
 ![image](https://user-images.githubusercontent.com/78403168/173195450-92efa094-384e-4fbe-94f5-79b199f00d67.png)

We then delete the org structure, and see that the data corresponding to the memory of org structure has been “freed”, tho there is some residual data that is not our concern. Also, we notice that orgs[0] still contains the pointer to the address of freed orgs structure. 
 
 ![image](https://user-images.githubusercontent.com/78403168/173195454-9900057d-84a5-47d9-88c1-72939032c397.png)
![image](https://user-images.githubusercontent.com/78403168/173195457-6e29b709-4ef6-4e46-a23c-22e4de08c20d.png)
![image](https://user-images.githubusercontent.com/78403168/173195458-f28f219a-9920-4b68-8c94-eb65bbb7b6ab.png)

 

We then create a new person structure, and due to how GLIBC allocation works, this persons structure will be allocated the memory of the previously freed org structure. We can confirm this by examining persons[0], which is an array of pointers to persons structures.
 ![image](https://user-images.githubusercontent.com/78403168/173195463-bc00c128-f1eb-4b59-82dd-a1b043adcfaa.png)
![image](https://user-images.githubusercontent.com/78403168/173195466-1b63f068-9633-4257-afc8-6e46a9d4501d.png)


 
We have successfully overwritten the display function! We can now call print and it will execute o->display, which is successfully overwritten with our ezflag function
 
![image](https://user-images.githubusercontent.com/78403168/173195469-217db449-c751-45b5-9c08-efb247ee9f88.png)



I was too lazy to write a script, so I entered everything manually. (Address of ezflag is converted from hex to decimal)

 ![image](https://user-images.githubusercontent.com/78403168/173195472-e0ec57fd-1963-4874-8d8d-5c48a7b19f08.png)


