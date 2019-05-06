# utl_importing_r_created_v8_transport_files_into_sas_wps
There appear to be two SAS bugs when importing R created v8 transport files. 
    There appear to be two SAS bugs and one R bug when importing R created v8 transport files

     github
     https://github.com/rogerjdeangelis/utl_importing_r_created_v8_transport_files_into_sas_wps

     Corrected macro (may impact R to SAS/WPS formats - but I prefer no to have any formats)

      https://goo.gl/Adacx9
      https://github.com/rogerjdeangelis/utl_importing_r_created_v8_transport_files_into_sas_wps/blob/master/utl_importing_r_created_v8_transport_files_into_sas_wps_xpt2loc.sas


     The transport file looks ok but.

      SAS Bugs

        1. Formats and informats do not work correctly

           8747  +format SPECIES  $        . ;
                                          -
                                          85
           ERROR 85-322: Expecting a format name.

        2. Truncates strings to 200 bytes
           Transport file has the longer strings

           Generated code with trucating length - works correctly if you edit sas xpt2loc macro

           length CHR500 $ 200 ;
           input SPECIES $ASCII00010. SEPALLENGTH XPRTFLT8. SEPALWIDTH XPRTFLT8.
               PETALLENGTH XPRTFLT8. PETALWIDTH XPRTFLT8. CHR500 $ASCII00200. @@;


    INPUT   ( has long variable names and long character variable 500 bytes)
    =====

    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
      set sashelp.iris;
      chr500=repeat('A',499);
    run;quit;

    SD1.HAVE total obs=150

     SPECIES    SEPALLENGTH    SEPALWIDTH    PETALLENGTH    PETALWIDTH   CHR500

     Setosa          50            33             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          46            34             14             3       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          46            36             10             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          51            33             17             5       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          55            35             13             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          48            31             16             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          52            34             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          49            36             14             1       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          44            32             13             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          50            35             16             6       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          44            30             13             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          47            32             16             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          48            30             14             3       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          51            38             16             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          48            34             19             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          50            30             16             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          50            32             12             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          43            30             11             1       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          58            40             12             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          51            38             19             4       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          49            30             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          51            35             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          50            34             16             4       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          46            32             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          57            44             15             4       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          50            36             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          54            34             15             4       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          52            41             15             1       AAAAAAAAAAAAAAAAAAAAAAAAAAA...
     Setosa          55            42             14             2       AAAAAAAAAAAAAAAAAAAAAAAAAAA...


    PROCESS
    =======

      * create v8 transport file;

      %utl_submit_wps64('
      libname sd1 "d:/sd1";
      options set=R_HOME "C:/Program Files/R/R-3.3.2";
      libname wrk "%sysfunc(pathname(work))";
      proc r;
      submit;
      source("C:/Program Files/R/R-3.3.2/etc/Rprofile.site", echo=T);
      library(haven);
      have<-read_sas("d:/sd1/have.sas7bdat");
      write_xpt(have,"d:/xpt/have8.xpt",version=8);
      endsubmit;
      run;quit;
      ');

      * version 8;
      %utlopts;
      %xpt2loc(libref=work,
          memlist=DATASET,
          filespec='d:/xpt/have8.xpt' );

       * I turned off the format and informat statements

    FIXES for macro XPT2LOC
    =====

      Comment out lines 298 and 310

        298  /* put 'format' +1 nliteral +1 format ';'; */
        310 /* put 'informat' +1 nliteral +1 informat ';';*/

      Edit generated code (only if you want 

      data work.DATASET ;
      infile 'd:/xpt/fshr8.xpt' recfm=n lrecl=32767
      ;
           if _n_=1 then input @1601@;

      length CHR500  $ 200 ;                  * CHANGE 200 TO 500;

      length SPECIES  $ 10 ;
      label SPECIES ="Iris Species" ;
      length SEPALLENGTH    8 ;
      label SEPALLENGTH ="Sepal Length (mm)" ;
      length SEPALWIDTH    8 ;
      label SEPALWIDTH ="Sepal Width (mm)" ;
      length PETALLENGTH    8 ;
      label PETALLENGTH ="Petal Length (mm)" ;
      length PETALWIDTH    8 ;
      label PETALWIDTH ="Petal Width (mm)" ;
      input
      SPECIES $ASCII00010.
      SEPALLENGTH XPRTFLT8.
      SEPALWIDTH XPRTFLT8.
      PETALLENGTH XPRTFLT8.
      PETALWIDTH XPRTFLT8.

      CHR500 $ASCII00200.                     * CHANGE 200 TO 500;

      @@;
           output; if _n_=150 then stop;
      run;


    OUTPUT (after fixes)
    ======

       proc contents data=DATASET position;
       run;quit;

       /*
                    Variables in Creation Order

       #    Variable       Type    Len    Label

       1    SPECIES        Char     10    Iris Species
       2    SEPALLENGTH    Num       8    Sepal Length (mm)
       3    SEPALWIDTH     Num       8    Sepal Width (mm)
       4    PETALLENGTH    Num       8    Petal Length (mm)
       5    PETALWIDTH     Num       8    Petal Width (mm)
       6    CHR500         Char    500
       */

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    see above


