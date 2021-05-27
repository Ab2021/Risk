/* https://support.sas.com/resources/papers/proceedings/pdfs/sgf2008/166-2008.pdf */

data abc1; /* don't need dataset*/
 infile datalines firstobs=2; /* raw file in */
 input; /* read a record */
 put _infile_; /* put buffer in log*/
 if _n_ > 50 then /* stop after 50 */
 stop; /* adjust as needed */
datalines;
City Number Minutes Charge
Jackson 415-555-2384 <25> <2.45>
Jefferson 813-555-2356 <15> <1.62>
Joliet 913-555-3223 <65> <10.32>
;
run;

data abc;
 length city number $16. minutes charge 8;
 infile datalines firstobs=2 _infile_=phonebuff;
 input @;
 _infile_ = compress(phonebuff, '<>');
 input city number minutes charge;
 put city= number= minutes= charge=;
datalines;
City Number Minutes Charge
Jackson 415-555-2384 <25> <2.45>
Jefferson 813-555-2356 <15> <1.62>
Joliet 913-555-3223 <65> <10.32>
;
run;

/* READING PAST THE END OF A LINE */
/* By default, if the INPUT statement tries to read past the end of the current input data record, it then moves the input */
/* pointer to column 1 of the next record to read the remaining values. This default behavior is governed by the */
/* "FLOWOVER" option and a message is written to the SAS log. This is useful in reading data that flows over into */
/* several lines of input. */
/* Example: Read a file of pet names and 6 readings per animal. */
data readings;
 infile datalines;
 input Name $ R1-R6;
 datalines;
Gus 22 44 55
 33 32 14
Gaia 24 22 23
 31 76 31
;
proc print data=readings;
 title 'Readings';
 run;
 
/*  If you use the MISSOVER option in the INFILE statement, then the DATA step creates five observations. However, all */
/* the values that were read from records that were too short are set to missing. */
/* The INFILE TRUNCOVER option tells INPUT to read as much as possible and that the value will be chopped off */
 data numbers2;
 infile datalines dsd dlm= ' ' missover;
 input testnum 5. ran $10.;
 datalines;
1    abc
22   abdc 
333  abcbdb
4444 ababbddd
55555 abbabdabd
 ;
 run;
 
/*  The problem with MISSOVER is that when you used formatted input and specify an informat width of N characters */
/*  and the line only has M characters left if M is less than N the MISSOVER option sets the result to missing.   */
/*  The TRUNCOVER option instead will have SAS read the existing characters.   */
/*  So MISSOVER option might be acceptable if the last field on the line is a field that has to be exactly */
/*  a fixed number of character long and you want SAS to automatically set the field missing if it is too short. */
 
 data want;
infile cards dsd dlm=',' truncover;
input id $4.  age weight;
cards;
ABC,1,25
XYZD,23,170
ZZZ,,201
;
run;


filename phonebk1 "/home/abhishek200219940/sasuser.v94/abc";
data _null_;
 file phonebk1;
 input line $80.;
 put line;
 datalines;
101 USA 1-20-1999 3295.50
3034 EUR 30JAN1999 1876,30
101 USA 1-30-1999 2938.00
128 USA 2-5-1999 2908.74
1345 EUR 6FEB1999 3135,60
109 USA 3-17-1999 2789.10
 ;
run; 

data abcd;
 infile phonebk truncover scanover;
 input SalesID $ Location $ @; if Location="USA" then
input SaleDate : mmddyy10.
Amount;
else if Location="EUR" then
input SaleDate: date9.
Amount: commax8.;
format SaleDate Date9.;
run;

filename phonebk2 "/home/abhishek200219940/sasuser.v94/abc1";
data _null_;
 file phonebk2;
 input line $80.;
 put line;
 datalines;
E00973 1400 E09872 2003 E73150 2400 
E45671 4500 E34805 1980
;
run;

Data work.retire; 
Length EmpId $6; 
infile phonebk2; 
input EmpID $ Contrib @@; Run;
