# utl-unusual-updates-of-master-table-using-transaction-table-without-join-or-additional-table
Unusual updates of master table using transaction table without a join or additional table (sort of)
    %let pgm=utl-unusual-updates-of-master-table-using-transaction-table-without-join-or-additional-table;

    %stop_submission;

    Too long to post here see github

    Unusual updates of master table using transaction table without a join or additional table (sort of)

    sas communities
    https://tinyurl.com/yrkww79c
    https://communities.sas.com/t5/SAS-Procedures/creating-flag-if-id-exists-in-different-dataset/m-p/743776#M80606


    In some cases the solutions do not update in place;
    All solutions use just the master table and the transaction table and output updated master.

     SOLUTIONS (all sas)

        1 sql in select (runs everywhere)
        2 dosubl shared storage (could use this to share large amounts od storage)
        3 sas hash
        4 utl_concat macro
        5 datastep update
        6 sql alter update (true update in plasce)
        7 utl_concat macro

    github
    https://tinyurl.com/mu3nprah
    https://github.com/rogerjdeangelis/utl-unusual-updates-of-master-table-using-transaction-table-without-join-or-additional-table

    related repo
    https://tinyurl.com/56dypcbe
    https://github.com/rogerjdeangelis/utl-sharing-a-common-block-of-memory-with-dosubl-using-sas-peek-and-poke

    /**************************************************************************************************************************/
    /* INPUT                      | PROCESS                                   |                                               */
    /* =====                      | ========                                  |                                               */
    /* WORK.MASTER                | 1 SQL IN SELECT (RUNS ANYWHERE)           |    OUTPUT                                     */
    /*                            | ===============================           |    WORK.MASTER                                */
    /* ID Y1 Y2 Y3                |                                           |               MATCH                           */
    /*                            | Flag records in master                    |    ID Y1 Y2 Y3 FLAG                           */
    /* 54 55 79 55                | that are in transaction                   |                                               */
    /* 55 83 61 55                | Self explatory sql code                   |    54 55 79 55   1                            */
    /* 56 74 77 60                |                                           |    55 83 61 55   1                            */
    /* 57 64 88 56                | %runit;/*--recreate master---*/           |    56 74 77 60   1                            */
    /* 60 87 78 83                | proc sql;                                 |    57 64 88 56   1                            */
    /* 61 84 87 77                |  create                                   |    60 87 78 83   1                            */
    /* 62 78 68 50                |    table master as                        |    61 84 87 77   0                            */
    /* 63 89 59 52                |  select                                   |    62 78 68 50   0                            */
    /* 64 74 77 60                |    *                                      |    63 89 59 52   0                            */
    /* 65 55 79 55                |   ,id in (                                |    64 74 77 60   0                            */
    /*                            |     select                                |    65 55 79 55   0                            */
    /* WORK.TRANSACTION           |       id                                  |                                               */
    /*                            |     from transactions) as flag            |                                               */
    /* ID X1 X2                   |  from                                     |                                               */
    /*                            |     master                                |                                               */
    /*                            | ;quit;                                    |                                               */
    /* 54  9  3                   |                                           |                                               */
    /* 55  3  2                   |                                           |                                               */
    /* 56  2  0                   |-------------------------------------------------------------------------------------------*/
    /* 57  4  4                   |                                           |                                               */
    /* 58  4  1                   | 2 DOSUBL SHARED STORAGE                   |    WORK.MASTER                                */
    /* 59  6  8                   | =======================                   |               MATCH                           */
    /* 60  1  7                   |                                           |    ID Y1 Y2 Y3 FLAG                           */
    /*                            | * for testing;                            |                                               */
    /*                            | %symdel varadr / nowarn;                  |    54 55 79 55   1                            */
    /*                            | %deletesasmacn;                           |    55 83 61 55   1                            */
    /* %macro runit;              | %runit;/*--recreate master---*/           |    56 74 77 60   1                            */
    /* proc sql;                  |                                           |    57 64 88 56   1                            */
    /* create                     | * macro on end;                           |    60 87 78 83   1                            */
    /*   table master             | data master;                              |    61 84 87 77   0                            */
    /*   (ID num, Y1 num,         |   %commonn(id,action=INIT);               |    62 78 68 50   0                            */
    /*    Y2 num, Y3 num);        |   set master;                             |    63 89 59 52   0                            */
    /* insert into master         |    rc=dosubl('                            |    64 74 77 60   0                            */
    /* values(54,55,79,55)        |     data _null_;                          |    65 55 79 55   0                            */
    /* values(55,83,61,55)        |      set transactions                     |                                               */
    /* values(56,74,77,60)        |       (rename=id=id_trans);               |                                               */
    /* values(57,64,88,56)        |      %commonn(id,action=GET);             |                                               */
    /* values(60,87,78,83)        |      if id=id_trans then do;              |                                               */
    /* values(61,84,87,77)        |         id=-1*id;                         |                                               */
    /* values(62,78,68,50)        |         %commonn(id,action=PUT);          |                                               */
    /* values(63,89,59,52)        |      end;                                 |                                               */
    /* values(64,74,77,60)        |     run;quit;                             |                                               */
    /* values(65,55,79,55)        |    ');                                    |                                               */
    /* ;quit;                     |   flag=(id<0);                            |                                               */
    /* %mend runit;               |    id=abs(id);                            |                                               */
    /*                            | drop rc;                                  |                                               */
    /* %runit;                    | run;quit;                                 |                                               */
    /*                            |                                           |                                               */
    /* data transactions;         |-------------------------------------------------------------------------------------------*/
    /*  input ID X1 X2;           | 3 SAS HASH                                |    WORK.MASTER                                */
    /* cards4;                    | ==========                                |               MATCH                           */
    /* 54 9 3                     |                                           |    ID Y1 Y2 Y3 FLAG                           */
    /* 55 3 2                     | %runit; /*- recreate master -*/           |                                               */
    /* 56 2 0                     | data master;                              |    54 55 79 55   1                            */
    /* 57 4 4                     | set master;                               |    55 83 61 55   1                            */
    /* 58 4 1                     | if _n_ = 1                                |    56 74 77 60   1                            */
    /* 59 6 8                     |  then do;                                 |    57 64 88 56   1                            */
    /* 60 1 7                     |   declare hash d1                         |    60 87 78 83   1                            */
    /* ;;;;                       |      (dataset:"transactions");            |    61 84 87 77   0                            */
    /* run;quit;                  |   d1.definekey("id");                     |    62 78 68 50   0                            */
    /*                            |   d1.definedone();                        |    63 89 59 52   0                            */
    /*                            |  end;                                     |    64 74 77 60   0                            */
    /*                            | flag = (d1.check() = 0);                  |    65 55 79 55   0                            */
    /*                            | run;                                      |                                               */
    /*                            |                                           |                                               */
    /*                            |                                           |                                               */
    /*                            |-------------------------------------------------------------------------------------------*/
    /*                            | 4 UTL_CONCAT MACRO                        |   WORK.MASTER                                 */
    /*                            | ==================                        |              MATCH                            */
    /*                            |                                           |   ID Y1 Y2 Y3 FLAG                            */
    /*                            | %runit;/*-recreate master-*/              |                                               */
    /*                            | data master;                              |   54 55 79 55   1                             */
    /*                            |  set master;                              |   55 83 61 55   1                             */
    /*                            |  if id in (%utl_concat(                   |   56 74 77 60   1                             */
    /*                            |         transactions                      |   57 64 88 56   1                             */
    /*                            |        ,var=id                            |   60 87 78 83   1                             */
    /*                            |        ,unique=Y                          |   61 84 87 77   0                             */
    /*                            |        ,od=%str(,))) then flag=1;         |   62 78 68 50   0                             */
    /*                            |  else flag=0;                             |   63 89 59 52   0                             */
    /*                            | run;quit;                                 |   64 74 77 60   0                             */
    /*                            |                                           |   65 55 79 55   0                             */
    /*                            |                                           |                                               */
    /*                            |-------------------------------------------------------------------------------------------*/
    /*                            | 5 DATASTEP UPDATE                         |   WORK.MASTER                                 */
    /*                            | =================                         |                                               */
    /*                            |                                           |   ID Y1 Y2 Y3 FLAG                            */
    /*                            | %runit;                                   |                                               */
    /*                            | data master;                              |   54 55 79 55   1                             */
    /*                            | update                                    |   55 83 61 55   1                             */
    /*                            |  master (in=m)                            |   56 74 77 60   1                             */
    /*                            |  transactions (in=t);                     |   57 64 88 56   1                             */
    /*                            | by id;                                    |   60 87 78 83   1                             */
    /*                            | if m;                                     |   61 84 87 77   0                             */
    /*                            | if t then flag=1;                         |   62 78 68 50   0                             */
    /*                            | else flag=0;                              |   63 89 59 52   0                             */
    /*                            | drop x1 x2;                               |   64 74 77 60   0                             */
    /*                            | run;quit;                                 |   65 55 79 55   0                             */
    /*                            |                                           |                                               */
    /*                            |                                           |                                               */
    /*                            | update                                    |                                               */
    /*                            |   master                                  |                                               */
    /*                            | set flag = 1                              |                                               */
    /*                            | where                                     |                                               */
    /*                            |   id in (                                 |                                               */
    /*                            |    select                                 |                                               */
    /*                            |      id                                   |                                               */
    /*                            |    from                                   |                                               */
    /*                            |    transactions);                         |                                               */
    /*                            | quit;                                     |                                               */
    /**************************************************************************************************************************/

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    %macro runit;
    proc sql;
    create
      table master
      (ID num, Y1 num,
       Y2 num, Y3 num);
    insert into master
    values(54,55,79,55)
    values(55,83,61,55)
    values(56,74,77,60)
    values(57,64,88,56)
    values(60,87,78,83)
    values(61,84,87,77)
    values(62,78,68,50)
    values(63,89,59,52)
    values(64,74,77,60)
    values(65,55,79,55)
    ;quit;
    %mend runit;

    %runit;

    data transactions;
     input ID X1 X2;
    cards4;
    54 9 3
    55 3 2
    56 2 0
    57 4 4
    58 4 1
    59 6 8
    60 1 7
    ;;;;
    run;quit;

    /**************************************************************************************************************************/
    /* WORK.MASTER             |  WORK.TRANSACTONS                                                                            */
    /*   ID    Y1    Y2    Y3  |  ID    X1    X2                                                                              */
    /*   54    55    79    55  |  54     9     3                                                                              */
    /*   55    83    61    55  |  55     3     2                                                                              */
    /*   56    74    77    60  |  56     2     0                                                                              */
    /*   57    64    88    56  |  57     4     4                                                                              */
    /*   60    87    78    83  |  58     4     1                                                                              */
    /*   61    84    87    77  |  59     6     8                                                                              */
    /*   62    78    68    50  |  60     1     7                                                                              */
    /*   63    89    59    52  |                                                                                              */
    /*   64    74    77    60  |                                                                                              */
    /*   65    55    79    55  |                                                                                              */
    /**************************************************************************************************************************/

    /*             _   _                  _           _
    / |  ___  __ _| | (_)_ __    ___  ___| | ___  ___| |_
    | | / __|/ _` | | | | `_ \  / __|/ _ \ |/ _ \/ __| __|
    | | \__ \ (_| | | | | | | | \__ \  __/ |  __/ (__| |_
    |_| |___/\__, |_| |_|_| |_| |___/\___|_|\___|\___|\__|
                |_|
    */

    %runit;/*--recreate master---*/
    proc sql;
     create
       table master as
     select
       *
      ,id in (
        select
          id
        from transactions) as flag
     from
        master
    ;quit;

    /**************************************************************************************************************************/
    /* OUTPUT                                                                                                                 */
    /* WORK.MASTER                                                                                                            */
    /*            MATCH                                                                                                       */
    /* ID Y1 Y2 Y3 FLAG                                                                                                       */
    /*                                                                                                                        */
    /* 54 55 79 55   1                                                                                                        */
    /* 55 83 61 55   1                                                                                                        */
    /* 56 74 77 60   1                                                                                                        */
    /* 57 64 88 56   1                                                                                                        */
    /* 60 87 78 83   1                                                                                                        */
    /* 61 84 87 77   0                                                                                                        */
    /* 62 78 68 50   0                                                                                                        */
    /* 63 89 59 52   0                                                                                                        */
    /* 64 74 77 60   0                                                                                                        */
    /* 65 55 79 55   0                                                                                                        */
    /**************************************************************************************************************************/

    /*___       _                 _     _       _                        _      _
    |___ \   __| | ___  ___ _   _| |__ | |  ___| |__   __ _ _ __ ___  __| | ___| |_ ___  _ __ __ _  __ _  ___
      __) | / _` |/ _ \/ __| | | | `_ \| | / __| `_ \ / _` | `__/ _ \/ _` |/ __| __/ _ \| `__/ _` |/ _` |/ _ \
     / __/ | (_| | (_) \__ \ |_| | |_) | | \__ \ | | | (_| | | |  __/ (_| |\__ \ || (_) | | | (_| | (_| |  __/
    |_____| \__,_|\___/|___/\__,_|_.__/|_| |___/_| |_|\__,_|_|  \___|\__,_||___/\__\___/|_|  \__,_|\__, |\___|
                                                                                            |___/
    */

    * for testing;
    %symdel varadr / nowarn;
    %deletesasmacn;
    %runit;/*--recreate master---*/

    * macro on end;
    data master;
      %commonn(id,action=INIT);
      set master;
       rc=dosubl('
        data _null_;
         set transactions
          (rename=id=id_trans);
         %commonn(id,action=GET);
         if id=id_trans then do;
            id=-1*id;
            %commonn(id,action=PUT);
         end;
        run;quit;
       ');
      flag=(id<0);
       id=abs(id);
    drop rc;
    run;quit;

    /**************************************************************************************************************************/
    /* same output as above                                                                                                   */
    /**************************************************************************************************************************/

    /*____                   _               _
    |___ /   ___  __ _ ___  | |__   __ _ ___| |__
      |_ \  / __|/ _` / __| | `_ \ / _` / __| `_ \
     ___) | \__ \ (_| \__ \ | | | | (_| \__ \ | | |
    |____/  |___/\__,_|___/ |_| |_|\__,_|___/_| |_|

    */

    %runit; /*- recreate master -*/
    data master;
    set master;
    if _n_ = 1
     then do;
      declare hash d1
         (dataset:"transactions");
      d1.definekey("id");
      d1.definedone();
     end;
    flag = (d1.check() = 0);
    run;

    /**************************************************************************************************************************/
    /* same output as above                                                                                                   */
    /**************************************************************************************************************************/

    /*  _           _   _                             _
    | || |    _   _| |_| |   ___ ___  _ __   ___ __ _| |_   _ __ ___   __ _  ___ _ __ ___
    | || |_  | | | | __| |  / __/ _ \| `_ \ / __/ _` | __| | `_ ` _ \ / _` |/ __| `__/ _ \
    |__   _| | |_| | |_| | | (_| (_) | | | | (_| (_| | |_  | | | | | | (_| | (__| | | (_) |
       |_|    \__,_|\__|_|__\___\___/|_| |_|\___\__,_|\__| |_| |_| |_|\__,_|\___|_|  \___/

    */

    %runit;/*-recreate master-*/
    data master;
     set master;
     if id in (%utl_concat(
            transactions
           ,var=id
           ,unique=Y
           ,od=%str(,))) then flag=1;
     else flag=0;
    run;quit;

    /**************************************************************************************************************************/
    /* same output as above                                                                                                   */
    /**************************************************************************************************************************/

    /*___        _       _            _                              _       _
    | ___|    __| | __ _| |_ __ _ ___| |_ ___ _ __   _   _ _ __   __| | __ _| |_ ___
    |___ \   / _` |/ _` | __/ _` / __| __/ _ \ `_ \ | | | | `_ \ / _` |/ _` | __/ _ \
     ___) | | (_| | (_| | || (_| \__ \ ||  __/ |_) || |_| | |_) | (_| | (_| | ||  __/
    |____/   \__,_|\__,_|\__\__,_|___/\__\___| .__/  \__,_| .__/ \__,_|\__,_|\__\___|
                                             |_|          |_|
    */

    %runit;
    data master;
      update
        master (in=m)
        transactions (in=t);
      by id;
      if m;
      if t then flag=1;
      else flag=0;
      drop x1 x2;
    run;quit;

    /**************************************************************************************************************************/
    /* same output as above                                                                                                   */
    /**************************************************************************************************************************/
    /*__               _         _ _              _        _     _
     / /_    ___  __ _| |   __ _| | |_ ___ _ __  | |_ __ _| |__ | | ___
    | `_ \  / __|/ _` | |  / _` | | __/ _ \ `__| | __/ _` | `_ \| |/ _ \
    | (_) | \__ \ (_| | | | (_| | | ||  __/ |    | || (_| | |_) | |  __/
     \___/  |___/\__, |_|  \__,_|_|\__\___|_|     \__\__,_|_.__/|_|\___|
                    |_|
    */

    %runit;
    proc sql;

     alter
      table master
     add
      flag numeric;

     update
      master
     set
      flag = 0;

    update
      master
    set flag = 1
    where
      id in (
       select
         id
       from
       transactions);
    quit;

    /**************************************************************************************************************************/
    /* same output as above                                                                                                   */
    /**************************************************************************************************************************/

    /*____         _   _                            _
    |___  |  _   _| |_| |  ___ ___  _ __   ___ __ _| |_   _ __ ___   __ _  ___ _ __ ___
       / /  | | | | __| | / __/ _ \| `_ \ / __/ _` | __| | `_ ` _ \ / _` |/ __| `__/ _ \
      / /   | |_| | |_| || (_| (_) | | | | (_| (_| | |_  | | | | | | (_| | (__| | | (_) |
     /_/     \__,_|\__|_|_\___\___/|_| |_|\___\__,_|\__| |_| |_| |_|\__,_|\___|_|  \___/
                       |___|
    */

    %macro commonn(var,action=init);
       %if %upcase(&action) = INIT %then %do;
          retain &var 0;
          call symputx("varadr",put(addrlong(&var.),hex16.),"G");
       %end;
       %else %if "%upcase(&action)" = "PUT" %then %do;
          call pokelong(put(&var,rb8.),"&varadr."x,8,8);
       %end;
       %else %if "%upcase(&action)" = "GET" %then %do;
          &var = input(peekclong("&varadr."x,8),rb8.);
       %end;
    %mend commonn;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
