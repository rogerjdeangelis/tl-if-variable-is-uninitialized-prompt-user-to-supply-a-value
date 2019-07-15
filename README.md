# utl-if-variable-is-uninitialized-prompt-user-to-supply-a-value
If variable is uninitialized prompt user to supply a value

    If variable is uninitialized prompt user to supply a value

    You can also do this interactively with the datastep debugger, but the interactive script cannot be reused.

    Only dosubl can trap on an error in an compiling/executing datastep and pop up a request to fix the error.
    I hope I don't regret this statement. Dosubl can also trap on executing sql code.

    Great question.

    This question has ramifications for error checking.

       Algorithm

              1. I put the problematic code minus the 'data' statement in a macro variable.
                 You could place the code in a file.
                 %let code=%str(
                        lengtht gender $1;
                        set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
                        age=33;
                    run;quit;);
              2. Run the code at compilation time using DOSUBL.
              3. Trap on notes, errors and warnings.
              4  You will get a message that gender is unitialized.
              5. Pop up a window requesting a value to intialize gender


    github
    https://tinyurl.com/y2qvy3qx
    https://github.com/rogerjdeangelis/utl-if-variable-is-uninitialized-prompt-user-to-supply-a-value

    SAS Forum
    https://tinyurl.com/y4torvjj
    https://communities.sas.com/t5/SAS-Programming/Whether-variable-exists-in-a-dataset/m-p/573194

       Problem:

            * gender is not initialized ;
            data want;
              lengthwant gender $1;
              set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
              age=33;
           run;quit;

       Solution:

           Window pops up requesting an initial value for variable gender, we enter "F"

           data want;
              retain gender "F";
              length gender $1;
              set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
              age=33;
           run;quit;

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    NOTE: Variable GENDER is uninitialized.

     %let code=%str(
       length gender $1;
       set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
        age=33;
    run;quit;);

    WORK.WANT total obs=5

       NAME    GENDER   AGE

      Joyce       .      33
      Louise      .      33
      Alice       .      33
      Jane        .      33
      Janet       .      33

    NOTE: Variable GENDER is uninitialized.

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    ;
    This window will pop up

    Command ===> end <enter>

    +-----------------------------------------------------------------+
    |                                                                 |
    |  ERROR: Variable GENDER is uninitialized.                       |
    |  Supply a value and press the Enter key then home key.          |
    |                                                                 |
    |                                                                 |
    |  Value: F <enter> <home>                                        |
    |                                                                 |
    |                                                                 |
    |  When you are finished entering variable value, hit ,enter.     |
    |  then end at the classic 1980s DMS command line.";              |
    |                                                                 |
    +-----------------------------------------------------------------+

    Corrected dataset

    WORK.WANT total obs=5

       NAME   GENDER   AGE

      Joyce     F       33
      Louise    F       33
      Alice     F       33
      Jane      F       33
      Janet     F       33


    NOTE: There were 5 observations read from the data set SASHELP.CLASSFIT.
          WHERE name in ('Alice ', 'Jane ', 'Janet ', 'Joyce ', 'Louise');
    NOTE: The data set WORK.WANT has 5 observations and 3 variables.

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    %let code=%str(
       length gender $1;
       set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
        age=33;
    run;quit;);

    %symdel var val systxt statement / nowarn;
    data want ;

       if _n_=0 then do;%let rc=%sysfunc(dosubl('
           %symdel  var val / nowarn;
           %let statement=retain;
           options dsoptions=note2err;
            data want;
              &code;
           options dsoptions=default;
           %let systxt=&syserrortext;
           options dsoptions=default;
           %if &syserr ^= 0 %then %do;
              data _null_;
                 var=scan("&systxt",2);
                 length val $1;
                 window start
                    #3  "&systxt"
                    #4  "Supply a value and press the Enter key then home key."
                    #7  "Value:" +1 val attr=underline
                    #11 "When you are finished entering variable value, hit ,enter."
                    #12 "then end at the classic 1980s DMS command line.";
                 display start;
                 call symputx("val",val);
                 call symputx("var",var);
              run;quit;
              %let statement=retain &var "&val";
           %end;
           '));
           &statement;  /* compilation time retain;
       end;

       &code;

    run;quit;

    SYMBOLGEN:  Macro variable VAR resolves to GENDER
    SYMBOLGEN:  Macro variable VAL resolves to F
    1489         &statement;
    SYMBOLGEN:  Macro variable STATEMENT resolves to retain GENDER "F"
    1490     end;
    1491     &code;
    SYMBOLGEN:  Macro variable CODE resolves to     length gender $1;
    set sashelp.class(keep=name Where=(name in ( "Joyce " "Louise" "Alice " "Jane " "Janet ")));
                age=33; run;quit;




