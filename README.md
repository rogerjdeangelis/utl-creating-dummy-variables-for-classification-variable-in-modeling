# utl-creating-dummy-variables-for-classification-variable-in-modeling
Creating dummy variables for classification variables in modeling
    %let pgm=utl-creating-dummy-variables-for-classification-variable-in-modeling;

    Creating dummy variables for classification variables in modeling

    github
    https://tinyurl.com/42s8ahuz
    https://github.com/rogerjdeangelis/utl-creating-dummy-variables-for-classification-variable-in-modeling

    Stackoverflow
    https://tinyurl.com/yckrdh3w
    https://stackoverflow.com/questions/70700795/columns-valorization-in-sas

      Theree Solutions
          1. Transreg (data-null)
               https://stackoverflow.com/users/2196220/data-null
          2. Proc freq (Reeza base sas solution)
               https://stackoverflow.com/users/1919583/reeza
          3. R  https://www.marsja.se/create-dummy-variables-in-r/
              There are a lot of options
    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    libname sd1 "d:/sd1";
    options validvarname=upcase;
    data have sd1.have;
       infile cards expandtabs;
       input id:$3. flag:$1.;
       cards4;
    012 A
    345 B
    678 C
    ;;;;
    run;quit;

    /*
    Up to 40 obs WORK.HAVE total obs=3 14JAN2022:10:19:53

    Obs    ID     FLAG

     1     012     A
     2     345     B
     3     678     C
    */

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    FROM TRANSREG

    Up to 40 obs WORK.WANT total obs=3 14JAN2022:10:22:18

    Obs    _TYPE_    _NAME_    FLAGA    FLAGB    FLAGC    FLAG

     1     SCORE      ROW1       1        0        0       A
     2     SCORE      ROW2       0        1        0       B
     3     SCORE      ROW3       0        0        1       C

    PROC FREQ

    Up to 40 obs from WANT total obs=3 14JAN2022:15:04:28

    Obs    ID     A    B    C

     1     012    1    0    0
     2     345    0    1    0
     3     678    0    0    1

    R

    Obs    ID     FLAG    FLAG_A    FLAG_B    FLAG_C

      1    012     A         1         0         0
      2    345     B         0         1         0
      3    678     C         0         0         1


    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
     _   _
    / | | |_ _ __ __ _ _ __  ___ _ __ ___  __ _
    | | | __| `__/ _` | `_ \/ __| `__/ _ \/ _` |
    | | | |_| | | (_| | | | \__ \ | |  __/ (_| |
    |_|  \__|_|  \__,_|_| |_|___/_|  \___|\__, |
                                          |___/
    */

    proc transreg data=have design;
       model class(flag / zero=none);
       output out=want(drop=INTERCEPT);
    run;quit;

    /*___                            __
    |___ \   _ __  _ __ ___   ___   / _|_ __ ___  __ _
      __) | | `_ \| `__/ _ \ / __| | |_| `__/ _ \/ _` |
     / __/  | |_) | | | (_) | (__  |  _| | |  __/ (_| |
    |_____| | .__/|_|  \___/ \___| |_| |_|  \___|\__, |
            |_|                                     |_|
    */

    proc freq data=have ;
    table id*flag / sparse out=long nopercent norow nocol;
    run;

    proc transpose data=long out=want (drop = _:);
    by id;
    id flag;
    var count;
    run;

    /*
     _ __
    | `__|
    | |
    |_|

    */

    %utlfkil("d:/xpt/want.xpt");

    proc datasets lib=work details nolist;
     delete want;
    run;quit;

    %utl_submit_r64('
    library(fastDummies);
    library(haven);
    library(SASxport);
    have<-read_sas("d:/sd1/have.sas7bdat");
    want <- as.data.frame(dummy_cols(have, select_columns = "FLAG"));
    want;
    write.xport(want,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";

    proc print data=xpt.want;
    run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */

















