%{

// Register declration enumeration here
enum { R_L0=0, R_L1=1, R_L2=2, R_L3=3, R_A=5, R_B=4 };
enum { R_F0=0, R_F1=1, R_F2=2, R_F3=3 };

#include "c.h"

#define NODEPTR_TYPE Node
#define OP_LABEL(p) ((p)->op)
#define LEFT_CHILD(p) ((p)->kids[0])
#define RIGHT_CHILD(p) ((p)->kids[1])
#define STATE_LABEL(p) ((p)->x.state)

// Declare local routines to be used by IR record here
static void address(Symbol, Symbol, long);
static void blkfetch(int, int, int, int);
static void blkloop(int, int, int, int, int, int[]);
static void blkstore(int, int, int, int);
static void defaddress(Symbol);
static void defconst(int, int, Value);
static void defstring(int, char *);
static void defsymbol(Symbol);
static void doarg(Node);
static void emit2(Node);
static void export(Symbol);
static void clobber(Node);
static void function(Symbol, Symbol [], Symbol [], int);
static void global(Symbol);
static void import(Symbol);
static void local(Symbol);
static void progbeg(int, char **);
static void progend(void);
static void segment(int);
static void space(int);
static void target(Node);
static int isfptr(Node,int,int);

// Local vars here

static Symbol longreg[32];
static Symbol intreg[32];
static Symbol fltreg[32];

static Symbol longwldcrd;
static Symbol intwldcrd;
static Symbol fltwldcrd;

static int current_seg;

%}
%start stmt

%term CNSTF4=4113
%term CNSTI1=1045
%term CNSTI2=2069
%term CNSTI4=4117
%term CNSTP2=2071
%term CNSTU1=1046
%term CNSTU2=2070
%term CNSTU4=4118

%term ARGB=41
%term ARGF4=4129
%term ARGI2=2085
%term ARGI4=4133
%term ARGP2=2087
%term ARGU2=2086
%term ARGU4=4134

%term ASGNB=57
%term ASGNF4=4145
%term ASGNI1=1077
%term ASGNI2=2101
%term ASGNI4=4149
%term ASGNP2=2103
%term ASGNU1=1078
%term ASGNU2=2102
%term ASGNU4=4150

%term INDIRB=73
%term INDIRF4=4161
%term INDIRI1=1093
%term INDIRI2=2117
%term INDIRI4=4165
%term INDIRP2=2119
%term INDIRU1=1094
%term INDIRU2=2118
%term INDIRU4=4166

%term CVFF4=4209
%term CVFI2=2165
%term CVFI4=4213
%term CVIF4=4225
%term CVII1=1157
%term CVII2=2181
%term CVII4=4229
%term CVIU1=1158
%term CVIU2=2182
%term CVIU4=4230
%term CVPU2=2198
%term CVUI1=1205
%term CVUI2=2229
%term CVUI4=4277
%term CVUP2=2231
%term CVUU1=1206
%term CVUU2=2230
%term CVUU4=4278

%term NEGF4=4289
%term NEGI2=2245
%term NEGI4=4293

%term CALLB=217
%term CALLF4=4305
%term CALLI2=2261
%term CALLI4=4309
%term CALLP2=2263
%term CALLU2=2262
%term CALLU4=4310
%term CALLV=216

%term RETF4=4337
%term RETI2=2293
%term RETI4=4341
%term RETP2=2295
%term RETU2=2294
%term RETU4=4342
%term RETV=248

%term ADDRGP2=2311
%term ADDRFP2=2327
%term ADDRLP2=2343

%term ADDF4=4401
%term ADDI2=2357
%term ADDI4=4405
%term ADDP2=2359
%term ADDU2=2358
%term ADDU4=4406

%term SUBF4=4417
%term SUBI2=2373
%term SUBI4=4421
%term SUBP2=2375
%term SUBU2=2374
%term SUBU4=4422

%term LSHI2=2389
%term LSHI4=4437
%term LSHU2=2390
%term LSHU4=4438

%term MODI2=2405
%term MODI4=4453
%term MODU2=2406
%term MODU4=4454

%term RSHI2=2421
%term RSHI4=4469
%term RSHU2=2422
%term RSHU4=4470

%term BANDI2=2437
%term BANDI4=4485
%term BANDU2=2438
%term BANDU4=4486

%term BCOMI2=2453
%term BCOMI4=4501
%term BCOMU2=2454
%term BCOMU4=4502

%term BORI2=2469
%term BORI4=4517
%term BORU2=2470
%term BORU4=4518

%term BXORI2=2485
%term BXORI4=4533
%term BXORU2=2486
%term BXORU4=4534

%term DIVF4=4545
%term DIVI2=2501
%term DIVI4=4549
%term DIVU2=2502
%term DIVU4=4550

%term MULF4=4561
%term MULI2=2517
%term MULI4=4565
%term MULU2=2518
%term MULU4=4566

%term EQF4=4577
%term EQI2=2533
%term EQI4=4581
%term EQU2=2534
%term EQU4=4582

%term GEF4=4593
%term GEI2=2549
%term GEI4=4597
%term GEU2=2550
%term GEU4=4598

%term GTF4=4609
%term GTI2=2565
%term GTI4=4613
%term GTU2=2566
%term GTU4=4614

%term LEF4=4625
%term LEI2=2581
%term LEI4=4629
%term LEU2=2582
%term LEU4=4630

%term LTF4=4641
%term LTI2=2597
%term LTI4=4645
%term LTU2=2598
%term LTU4=4646

%term NEF4=4657
%term NEI2=2613
%term NEI4=4661
%term NEU2=2614
%term NEU4=4662

%term JUMPV=584

%term LABELV=600

%term VREGP=711

%term LOADI4=4325
%term LOADU4=4326
%term LOADI2=2277
%term LOADU2=2278
%term LOADP2=2279
%term LOADF4=4321
%term LOADB=233
%term LOADF8=8417
%term LOADI1=1253
%term LOADU1=1254
%%

reg8:	LOADI1(reg8)	"\tcopy\t%c,%0\n" move(a)
reg8:	LOADU1(reg8)	"\tcopy\t%c,%0\n" move(a)
reg:	LOADI2(reg)	"\tcopy\t%c,%0\n" move(a)
reg:	LOADU2(reg)	"\tcopy\t%c,%0\n" move(a)
reg:	LOADP2(reg)	"\tcopy\t%c,%0\n" move(a)
reg:	LOADI4(reg)	"\tCOPY32\t%c,%0\n" move(a)
reg:	LOADU4(reg)	"\tCOPY32\t%c,%0\n" move(a)
reg:	LOADF4(reg)	"\tCOPY32\t%c,%0\n" move(a)

reg8:	INDIRI1(VREGP)     "# read register\n"
reg8:	INDIRU1(VREGP)     "# read register\n"
reg:	INDIRI2(VREGP)     "# read register\n"
reg:	INDIRU2(VREGP)     "# read register\n"
reg:	INDIRI4(VREGP)     "# read register\n"
reg:	INDIRU4(VREGP)     "# read register\n"
reg:	INDIRP2(VREGP)     "# read register\n"
reg:	INDIRF4(VREGP)     "# read register\n"

stmt:	ASGNI1(VREGP,reg8)  "# write register\n"
stmt:	ASGNU1(VREGP,reg8)  "# write register\n"
stmt:	ASGNI2(VREGP,reg)  "# write register\n"
stmt:	ASGNU2(VREGP,reg)  "# write register\n"
stmt:	ASGNI4(VREGP,reg)  "# write register\n"
stmt:	ASGNU4(VREGP,reg)  "# write register\n"
stmt:	ASGNP2(VREGP,reg)  "# write register\n"
stmt:	ASGNF4(VREGP,reg)  "# write register\n"

reg: CVFF4(reg)	"\tCOPY32\t%c,%1\n" move(a)
reg: CVFI2(reg)	"\tCVTFI2\t%c,%0\n" 1
reg: CVFI4(reg)	"\tCVTFI4\t%c,%0\n" 1
reg: CVIF4(reg)	"\tCVTIF4\t%c,%0\n" 1
reg: CVII2(reg8) "?\tcopy\t%c,%0\n\tsex\t%c\n" 2
reg8: CVII2(reg8) "%0" move(a)
reg: CVII4(reg)	"\tCVTII4\t%c,%0\n" 1
reg8: CVIU1(reg)	"\tcopy\t%c,%0\n" move(a)
reg: CVIU2(reg)	"?\tcopy\t%c,%0\n\tand.16\t%c,0xff\n" 2
reg: CVIU2(reg8)	"?\tcopy\t%c,%0\n\tand.16\t%c,0xff\n" 2
reg: CVII4(reg)	"\tCVTII4\t%c,%0\n" 1
reg: CVPU2(reg)	"\tcopy\t%c,%0\n" move(a)
reg8: CVUI1(reg)	"\tcopy\t%c,%0\n" move(a)
reg: CVUI2(reg)	"?\tcopy\t%c,%0\n\tand.16\t%c,0xff\n" 2
reg: CVUI2(reg8)	"?\tcopy\t%c,%0\n\tand.16\t%c,0xff\n" 2
reg: CVUI4(reg)	"\tCVTUI4\t%c,%0\n" 1
reg: CVUP2(reg)	"\tcopy\t%c,%0\n" move(a)
reg: CVUU1(reg)	"\tcopy\t%c,%0\n" move(a)
reg8: CVUU1(reg)	"\tcopy\t%c,%0\n" move(a)
reg: CVUU2(reg)	"?\tcopy\t%c,%0\n\tand.16\t%c,0xff\n" 2
reg8: CVUU2(reg8) "%0" move(a)
reg: CVUU4(reg)	"\tCVTUU4\t%c,%0\n" 1

reg: con "\tld.16\t%c,%0\n" 1
reg8: con "\tld.8\t%c,%0\n" 1

c0: CNSTI2  "0"	range(a,0,0)
c1: CNSTI2  "1"	range(a,1,1)
c2: CNSTI2  "2"	range(a,2,2)

con1: CNSTI1  "%a"
con1: CNSTU1  "%a"

con2: CNSTI2  "%a"
con2: CNSTU2  "%a"

con4: CNSTI4  "%a"
con4: CNSTU4  "%a"
con4: CNSTP2  "%a"

con:	con1	"%0"
con:	con2	"%0"
con:	con4	"%0"
con:	c0	"%0"
con:	c1	"%0"
con:	c2	"%0"

reg:	con1	"\tld.8\t%c,%0\n"   1
reg:	con2	"\tld.16\t%c,%0\n"  1
reg:	con4	"\tCON32\t%c,%0\n"  2

reg:	ADDI2(reg,reg)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDI2(reg,con)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDI2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDU2(reg,reg)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDU2(reg,con)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDU2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDP2(reg,reg)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDP2(reg,con)	"?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2
reg:	ADDP2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tadd.16\t%c,%1\n" 2

reg8:	ADDI2(reg8,reg8)	"?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2
reg8:	ADDI2(reg8,con)	"?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2
reg8:	ADDI2(reg8,CVII2(INDIRI1(addr)))    "?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2
reg8:	ADDU2(reg8,reg8)	"?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2
reg8:	ADDU2(reg8,con)	"?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2
reg8:	ADDU2(reg8,CVUU2(INDIRU1(addr)))    "?\tcopy\t%c,%0\n\tadd.8\t%c,%1\n" 2

reg:	ADDI4(reg,reg)	"?\tTCOPY\t%c,%0\n\tADD32\t%c,%1\n" 2
reg:	ADDI4(reg,INDIRI4(addr))	"?\tTCOPY\t%c,%0\n\tADD32\t%c,%1\n" 2
reg:	ADDU4(reg,reg)	"?\tTCOPY\t%c,%0\n\tADD32\t%c,%1\n" 2
reg:	ADDU4(reg,INDIRU4(addr))	"?\tTCOPY\t%c,%0\n\tADD32\t%c,%1\n" 2
reg:	ADDF4(reg,reg)	"?\tTCOPY\t%c,%0\n\tFADD32\t%c,%1\n" 2
reg:	ADDF4(reg,INDIRF4(addr))	"?\tTCOPY\t%c,%0\n\tFADD32\t%c,%1\n" 2

reg:	SUBI2(reg,reg)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBI2(reg,con)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBI2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBU2(reg,reg)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBU2(reg,con)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBU2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBP2(reg,reg)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBP2(reg,con)	"?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2
reg:	SUBP2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tsub.16\t%c,%1\n" 2

reg8:	SUBI2(reg8,reg8)	"?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2
reg8:	SUBI2(reg8,con)	"?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2
reg8:	SUBI2(reg8,CVII2(INDIRI2(addr)))    "?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2
reg8:	SUBU2(reg8,reg8)	"?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2
reg8:	SUBU2(reg8,con)	"?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2
reg8:	SUBU2(reg8,CVUU2(INDIRU2(addr)))    "?\tcopy\t%c,%0\n\tsub.8\t%c,%1\n" 2

reg:	SUBI4(reg,reg)	"?\tTCOPY\t%c,%0\n\tSUB32\t%c,%1\n" 2
reg:	SUBI4(reg,INDIRI4(addr))	"?\tTCOPY\t%c,%0\n\tSUB32\t%c,%1\n" 2
reg:	SUBU4(reg,reg)	"?\tTCOPY\t%c,%0\n\tSUB32\t%c,%1\n" 2
reg:	SUBU4(reg,INDIRU4(addr))	"?\tTCOPY\t%c,%0\n\tSUB32\t%c,%1\n" 2
reg:	SUBF4(reg,reg)	"?\tTCOPY\t%c,%0\n\tFSUB32\t%c,%1\n" 2
reg:	SUBF4(reg,INDIRF4(addr))	"?\tTCOPY\t%c,%0\n\tFSUB32\t%c,%1\n" 2

reg:	BANDI2(reg,reg)	"?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2
reg:	BANDI2(reg,con)	"?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2
reg:	BANDI2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2
reg:	BANDU2(reg,reg)	"?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2
reg:	BANDU2(reg,con)	"?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2
reg:	BANDU2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tand.16\t%c,%1\n" 2

reg8:	BANDI2(reg8,reg8)	"?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2
reg8:	BANDI2(reg8,con)	"?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2
reg8:	BANDI2(reg8,CVII2(INDIRI2(addr)))    "?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2
reg8:	BANDU2(reg8,reg8)	"?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2
reg8:	BANDU2(reg8,con)	"?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2
reg8:	BANDU2(reg8,CVUU2(INDIRU2(addr)))    "?\tcopy\t%c,%0\n\tand.8\t%c,%1\n" 2

reg:	BANDI4(reg,reg)	"?\tTCOPY\t%c,%0\n\tAND32\t%c,%1\n" 2
reg:	BANDI4(reg,INDIRI4(addr))	"?\tTCOPY\t%c,%0\n\tAND32\t%c,%1\n" 2
reg:	BANDU4(reg,reg)	"?\tTCOPY\t%c,%0\n\tAND32\t%c,%1\n" 2
reg:	BANDU4(reg,INDIRU4(addr))	"?\tTCOPY\t%c,%0\n\tAND32\t%c,%1\n" 2

reg:	BORI2(reg,reg)	"?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2
reg:	BORI2(reg,con)	"?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2
reg:	BORI2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2
reg:	BORU2(reg,reg)	"?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2
reg:	BORU2(reg,con)	"?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2
reg:	BORU2(reg,INDIRI2(addr))    "?\tcopy\t%c,%0\n\tor.16\t%c,%1\n" 2

reg8:	BORI2(reg8,reg8)	"?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2
reg8:	BORI2(reg8,con)	"?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2
reg8:	BORI2(reg8,CVII2(INDIRI2(addr)))    "?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2
reg8:	BORU2(reg8,reg8)	"?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2
reg8:	BORU2(reg8,con)	"?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2
reg8:	BORU2(reg8,CVUU2(INDIRU2(addr)))    "?\tcopy\t%c,%0\n\tor.8\t%c,%1\n" 2


reg:	BORI4(reg,reg)	"?\tTCOPY\t%c,%0\n\tOR32\t%c,%1\n" 2
reg:	BORI4(reg,INDIRI4(addr))	"?\tTCOPY\t%c,%0\n\tOR32\t%c,%1\n" 2
reg:	BORU4(reg,reg)	"?\tTCOPY\t%c,%0\n\tOR32\t%c,%1\n" 2
reg:	BORU4(reg,INDIRU4(addr))	"?\tTCOPY\t%c,%0\n\tOR32\t%c,%1\n" 2

reg:	BXORI2(reg,reg)	"?\tcopy\t%c,%0\n\txor.16\t%c,%1\n" 2
reg:	BXORU2(reg,reg)	"?\tcopy\t%c,%0\n\txor.16\t%c,%1\n" 2

reg:	BXORI4(reg,reg)	"?\tTCOPY\t%c,%0\n\tXOR32\t%c,%1\n" 2
reg:	BXORI4(reg,INDIRI4(addr))	"?\tTCOPY\t%c,%0\n\tXOR32\t%c,%1\n" 2
reg:	BXORU4(reg,reg)	"?\tTCOPY\t%c,%0\n\tXOR32\t%c,%1\n" 2
reg:	BXORU4(reg,INDIRU4(addr))	"?\tTCOPY\t%c,%0\n\tXOR32\t%c,%1\n" 2

reg:	MULI2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$muli16\n" 2
reg:	MULU2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$mulu16\n" 2
reg:	MULI4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$muli32\n" 2
reg:	MULI4(reg,INDIRI4(addr))	"?\tCOPY32\t%c,%0\n\tMULI32\t%c,%1\n" 2
reg:	MULU4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$mulu32\n" 2
reg:	MULU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$mulu32\n" 2
reg:	MULF4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$mulf32\n" 2
reg:	MULU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$mulf32\n" 2

reg:	DIVI2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$divi16\n" 2
reg:	DIVU2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$divu16\n" 2
reg:	DIVI4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$divi32\n" 2
reg:	DIVI4(reg,INDIRI4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$divi32\n" 2
reg:	DIVU4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$divu32\n" 2
reg:	DIVU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$divu32\n" 2
reg:	DIVF4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$divf32\n" 2
reg:	DIVF4(reg,INDIRF4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$divf32\n" 2

reg:	MODI2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$modi16\n" 2
reg:	MODU2(reg,reg)	"?\tcopy\t%c,%0\n\tcall\t$modu16\n" 2
reg:	MODI4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$modi32\n" 2
reg:	MODI4(reg,INDIRI4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$modi32\n" 2
reg:	MODU4(reg,reg)	"?\tCOPY32\t%c,%0\n\tcall\t$modu32\n" 2
reg:	MODU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tcall\t$modu32\n" 2

reg:	NEGI2(reg)	"\tld.16\t%c,0\n\tsub.16\t%c,%0\n" 2
reg:	NEGI4(reg)	"tNEGI32\t%c,%0\n" 4
reg:	NEGI4(INDIRI4(addr))	"\tNEGI32\t%c,%0\n" 4
reg:	NEGF4(reg)	"\tNEGF32\t%c,%0\n" 4
reg:	NEGF4(INDIRF4(addr))	"\tNEGF32\t%c,%0\n" 4

reg:	LSHI2(reg,c1)	"?\tcopy\t%c,%0\n\tshl.16\t%c,%c\n" 2
reg:	LSHI2(reg,c2)	"?\tcopy\t%c,%0\n\tshl.16\t%c,%c\n\tshl.16\t%c,%c\n" 3
reg:	LSHI2(reg,con)	"?\tcopy\t%c,%0\n\tld.16\tc,%1\n\tvshl.16\t%c\n" 3
reg:	LSHI2(reg,reg)	"?\tcopy\t%c,%0\n\tcopy\tc,%1\n\tvshl.16\t%c\n" 3
reg:	LSHU2(reg,c1)	"?\tcopy\t%c,%0\n\tshl.16\t%c,%c\n" 2
reg:	LSHU2(reg,c2)	"?\tcopy\t%c,%0\n\tshl.16\t%c,%c\n\tshl.16\t%c,%c\n" 3
reg:	LSHU2(reg,con)	"?\tcopy\t%c,%0\n\tld.16\tc,%1\n\tvshl.16\t%c\n" 3
reg:	LSHU2(reg,reg)	"?\tcopy\t%c,%0\n\tcopy\tc,%1\n\tvshl.16\t%c\n" 3

reg:	LSHI4(reg,reg)	"?\tCOPY32\t%c,%0\n\tLSH32\t%c,%1\n" 4
reg:	LSHI4(reg,INDIRI4(addr))	"?\tCOPY32\t%c,%0\n\tLSH32\t%c,%1\n" 4
reg:	LSHU4(reg,reg)	"?\tCOPY32\t%c,%0\n\tLSH32\t%c,%1\n" 4
reg:	LSHU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tLSH32\t%c,%1\n" 4

reg:	RSHI2(reg,c1)	"?\tcopy\t%c,%0\n\tshr.16\t%c,%c\n" 2
reg:	RSHI2(reg,c2)	"?\tcopy\t%c,%0\n\tshr.16\t%c,%c\n\tshr.16\t%c,%c\n" 3
reg:	RSHI2(reg,con)	"?\tcopy\t%c,%0\n\tld.16\tc,%1\n\tvshr.16\t%c\n" 3
reg:	RSHI2(reg,reg)	"?\tcopy\t%c,%0\n\tcopy\tc,%1\n\tvshr.16\t%c\n" 3
reg:	RSHU2(reg,c1)	"?\tcopy\t%c,%0\n\tshr.16\t%c,%c\n" 2
reg:	RSHU2(reg,c2)	"?\tcopy\t%c,%0\n\tshr.16\t%c,%c\n\tshr.16\t%c,%c\n" 3
reg:	RSHU2(reg,con)	"?\tcopy\t%c,%0\n\tld.16\tc,%1\n\tvshr.16\t%c\n" 3
reg:	RSHU2(reg,reg)	"?\tcopy\t%c,%0\n\tcopy\tc,%1\n\tvshr.16\t%c\n" 3

reg:	RSHI4(reg,reg)	"?\tCOPY32\t%c,%0\n\tRSH32\t%c,%1\n" 4
reg:	RSHI4(reg,INDIRI4(addr))	"?\tCOPY32\t%c,%0\n\tRSH32\t%c,%1\n" 4
reg:	RSHU4(reg,reg)	"?\tCOPY32\t%c,%0\n\tRSH32\t%c,%1\n" 4
reg:	RSHU4(reg,INDIRU4(addr))	"?\tCOPY32\t%c,%0\n\tRSH32\t%c,%1\n" 4

stmt:	EQI2(reg,reg)		"\tcmpb.eq.16\t%0,%1,%a\n" 1
stmt:	EQI2(reg,con)		"\tcmpb.eq.16\t%0,%1,%a\n" 1
stmt:	EQI2(reg,INDIRI2(addr)) "\tcmpb.eq.16\t%0,%1,%a\n" 1
stmt:	EQU2(reg,reg)		"\tcmpb.eq.16\t%0,%1,%a\n" 1
stmt:	EQU2(reg,con)		"\tcmpb.eq.16\t%0,%1,%a\n" 1
stmt:	EQU2(reg,INDIRU2(addr)) "\tcmpb.eq.16\t%0,%1,%a\n" 1

stmt:	EQI2(reg8,reg8)		"\tcmpb.eq.8\t%0,%1,%a\n" 1
stmt:	EQI2(reg8,con)		"\tcmpb.eq.8\t%0,%1,%a\n" 1
stmt:	EQI2(reg8,CVII2(INDIRI1(addr))) "\tcmpb.eq.8\t%0,%1,%a\n" 1
stmt:	EQU2(reg8,reg8)		"\tcmpb.eq.8\t%0,%1,%a\n" 1
stmt:	EQU2(reg8,con)		"\tcmpb.eq.8\t%0,%1,%a\n" 1
stmt:	EQU2(reg8,CVUU2(INDIRU1(addr))) "\tcmpb.eq.8\t%0,%1,%a\n" 1

stmt:	EQI4(reg,reg)	"\tCMPBEQ32\t%0,%1,%a\n" 2
stmt:	EQI4(reg,INDIRI4(addr))	"\tCMPBEQ32\t%0,%1,%a\n" 2
stmt:	EQU4(reg,reg)	"\tCMPBEQ32\t%0,%1,%a\n" 2
stmt:	EQU4(reg,INDIRU4(addr))	"\tCMPBEQ32\t%0,%1,%a\n" 2
stmt:	EQF4(reg,reg)	"\tCMPBEQF\t%0,%1,%a\n" 2
stmt:	EQF4(reg,INDIRF4(addr))	"\tCMPBEQF\t%0,%1,%a\n" 2

stmt:	NEI2(reg,reg)		"\tcmpb.ne.16\t%0,%1,%a\n" 1
stmt:	NEI2(reg,con)		"\tcmpb.ne.16\t%0,%1,%a\n" 1
stmt:	NEI2(reg,INDIRI2(addr)) "\tcmpb.ne.16\t%0,%1,%a\n" 1
stmt:	NEU2(reg,reg)		"\tcmpb.ne.16\t%0,%1,%a\n" 1
stmt:	NEU2(reg,con)		"\tcmpb.ne.16\t%0,%1,%a\n" 1
stmt:	NEU2(reg,INDIRU2(addr)) "\tcmpb.ne.16\t%0,%1,%a\n" 1

stmt:	NEI2(reg8,reg8)		"\tcmpb.ne.8\t%0,%1,%a\n" 1
stmt:	NEI2(reg8,con)		"\tcmpb.ne.8\t%0,%1,%a\n" 1
stmt:	NEI2(reg8,CVII2(INDIRI1(addr))) "\tcmpb.ne.8\t%0,%1,%a\n" 1
stmt:	NEU2(reg8,reg8)		"\tcmpb.ne.8\t%0,%1,%a\n" 1
stmt:	NEU2(reg8,con)		"\tcmpb.ne.8\t%0,%1,%a\n" 1
stmt:	NEU2(reg8,CVUU2(INDIRU1(addr))) "\tcmpb.ne.8\t%0,%1,%a\n" 1

stmt:	NEI4(reg,reg)	"\tCMPBNE32\t%0,%1,%a\n" 2
stmt:	NEI4(reg,INDIRU4(addr))	"\tCMPBNE32\t%0,%1,%a\n" 2
stmt:	NEU4(reg,reg)	"\tCMPBNE32\t%0,%1,%a\n" 2
stmt:	NEU4(reg,INDIRU4(addr))	"\tCMPBNE32\t%0,%1,%a\n" 2
stmt:	NEF4(reg,reg)	"\tCMPBNEF\t%0,%1,%a\n" 2
stmt:	NEF4(reg,INDIRF4(addr))	"\tCMPBNEF\t%0,%1,%a\n" 2

stmt:	LTI2(reg,reg)		"\tcmpb.lt.16\t%0,%1,%a\n" 1
stmt:	LTI2(reg,con)		"\tcmpb.lt.16\t%0,%1,%a\n" 1
stmt:	LTI2(reg,INDIRI2(addr)) "\tcmpb.lt.16\t%0,%1,%a\n" 1
stmt:	LTU2(reg,reg)		"\tcmpb.ltu.16\t%0,%1,%a\n" 1
stmt:	LTU2(reg,con)		"\tcmpb.ltu.16\t%0,%1,%a\n" 1
stmt:	LTU2(reg,INDIRU2(addr)) "\tcmpb.ltu.16\t%0,%1,%a\n" 1

stmt:	LTI2(reg8,reg8)		"\tcmpb.lt.8\t%0,%1,%a\n" 1
stmt:	LTI2(reg8,con)		"\tcmpb.lt.8\t%0,%1,%a\n" 1
stmt:	LTI2(reg8,CVII2(INDIRI1(addr))) "\tcmpb.lt.8\t%0,%1,%a\n" 1
stmt:	LTU2(reg8,reg8)		"\tcmpb.ltu.8\t%0,%1,%a\n" 1
stmt:	LTU2(reg8,con)		"\tcmpb.ltu.8\t%0,%1,%a\n" 1
stmt:	LTU2(reg8,CVUU2(INDIRU1(addr))) "\tcmpb.ltu.8\t%0,%1,%a\n" 1

stmt:	LTI4(reg,reg)	"\tCMPBLT32\t%0,%1,%a\n" 2
stmt:	LTI4(reg,INDIRI4(addr))	"\tCMPBLT32\t%0,%1,%a\n" 2
stmt:	LTU4(reg,reg)	"\tCMPBLTU32\t%0,%1,%a\n" 2
stmt:	LTU4(reg,INDIRU4(addr))	"\tCMPBLTU32\t%0,%1,%a\n" 2
stmt:	LTF4(reg,reg)	"\tCMPBLTF\t%0,%1,%a\n" 2
stmt:	LTF4(reg,INDIRF4(addr))	"\tCMPBLTF\t%0,%1,%a\n" 2

stmt:	LEI2(reg,reg)		"\tcmpb.le.16\t%0,%1,%a\n" 1
stmt:	LEI2(reg,con)		"\tcmpb.le.16\t%0,%1,%a\n" 1
stmt:	LEI2(reg,INDIRI2(addr)) "\tcmpb.le.16\t%0,%1,%a\n" 1
stmt:	LEU2(reg,reg)		"\tcmpb.leu.16\t%0,%1,%a\n" 1
stmt:	LEU2(reg,con)		"\tcmpb.leu.16\t%0,%1,%a\n" 1
stmt:	LEU2(reg,INDIRU2(addr)) "\tcmpb.leu.16\t%0,%1,%a\n" 1

stmt:	LEI2(reg8,reg8)		"\tcmpb.le.8\t%0,%1,%a\n" 1
stmt:	LEI2(reg8,con)		"\tcmpb.le.8\t%0,%1,%a\n" 1
stmt:	LEI2(reg8,CVII2(INDIRI1(addr))) "\tcmpb.le.8\t%0,%1,%a\n" 1
stmt:	LEU2(reg8,reg8)		"\tcmpb.leu.8\t%0,%1,%a\n" 1
stmt:	LEU2(reg8,con)		"\tcmpb.leu.8\t%0,%1,%a\n" 1
stmt:	LEU2(reg8,CVUU2(INDIRU1(addr))) "\tcmpb.leu.8\t%0,%1,%a\n" 1

stmt:	LEI4(reg,reg)	"\tCMPBLE32\t%0,%1,%a\n" 2
stmt:	LEI4(reg,INDIRI4(addr))	"\tCMPBLE32\t%0,%1,%a\n" 2
stmt:	LEU4(reg,reg)	"\tCMPBLEU32\t%0,%1,%a\n" 2
stmt:	LEU4(reg,INDIRU4(addr))	"\tCMPBLEU32\t%0,%1,%a\n" 2
stmt:	LEF4(reg,reg)	"\tCMPBLEF\t%0,%1,%a\n" 2
stmt:	LEF4(reg,INDIRF4(addr))	"\tCMPBLEF\t%0,%1,%a\n" 2

stmt:	GTI2(reg,reg)		"\tcmpb.lt.16\t%1,%0,%a\n" 1
stmt:	GTI2(con,reg)		"\tcmpb.lt.16\t%1,%0,%a\n" 1
stmt:	GTI2(INDIRI2(addr),reg) "\tcmpb.lt.16\t%1,%0,%a\n" 1
stmt:	GTU2(reg,reg)		"\tcmpb.ltu.16\t%1,%0,%a\n" 1
stmt:	GTU2(con,reg)		"\tcmpb.ltu.16\t%1,%0,%a\n" 1
stmt:	GTU2(INDIRU2(addr),reg) "\tcmpb.ltu.16\t%1,%0,%a\n" 1

stmt:	GTI2(reg8,reg8)		"\tcmpb.lt.8\t%1,%0,%a\n" 1
stmt:	GTI2(con,reg8)		"\tcmpb.lt.8\t%1,%0,%a\n" 1
stmt:	GTI2(CVII2(INDIRI1(addr)),reg8) "\tcmpb.lt.8\t%1,%0,%a\n" 1
stmt:	GTU2(reg8,reg8)		"\tcmpb.ltu.8\t%1,%0,%a\n" 1
stmt:	GTU2(con,reg8)		"\tcmpb.ltu.8\t%1,%0,%a\n" 1
stmt:	GTU2(CVUU2(INDIRU1(addr)),reg8) "\tcmpb.ltu.8\t%1,%0,%a\n" 1

stmt:	GTI4(reg,reg)	"\tCMPBLT32\t%1,%0,%a\n" 2
stmt:	GTI4(INDIRI4(addr),reg)	"\tCMPBLT32\t%1,%0,%a\n" 2
stmt:	GTU4(reg,reg)	"\tCMPBLTU32\t%1,%0,%a\n" 2
stmt:	GTU4(INDIRU4(addr),reg)	"\tCMPBLTU32\t%1,%0,%a\n" 2
stmt:	GTF4(reg,reg)	"\tCMPBLTF\t%1,%0,%a\n" 2
stmt:	GTF4(INDIRF4(addr),reg)	"\tCMPBLTF\t%1,%0,%a\n" 2

stmt:	GEI2(reg,reg)		"\tcmpb.le.16\t%1,%0,%a\n" 1
stmt:	GEI2(con,reg)		"\tcmpb.le.16\t%1,%0,%a\n" 1
stmt:	GEI2(INDIRI2(addr),reg) "\tcmpb.le.16\t%1,%0,%a\n" 1
stmt:	GEU2(reg,reg)		"\tcmpb.leu.16\t%1,%0,%a\n" 1
stmt:	GEU2(con,reg)		"\tcmpb.leu.16\t%1,%0,%a\n" 1
stmt:	GEU2(INDIRU2(addr),reg) "\tcmpb.leu.16\t%1,%0,%a\n" 1

stmt:	GEI2(reg8,reg8)		"\tcmpb.le.8\t%1,%0,%a\n" 1
stmt:	GEI2(con,reg8)		"\tcmpb.le.8\t%1,%0,%a\n" 1
stmt:	GEI2(CVII2(INDIRI1(addr)),reg8) "\tcmpb.le.8\t%1,%0,%a\n" 1
stmt:	GEU2(reg8,reg8)		"\tcmpb.leu.8\t%1,%0,%a\n" 1
stmt:	GEU2(con,reg8)		"\tcmpb.leu.8\t%1,%0,%a\n" 1
stmt:	GEU2(CVUU2(INDIRU1(addr)),reg8) "\tcmpb.leu.8\t%1,%0,%a\n" 1

stmt:	GEI4(reg,reg)	"\tCMPBLE32\t%1,%0,%a\n" 2
stmt:	GEI4(INDIRI4(addr),reg)	"\tCMPBLE32\t%1,%0,%a\n" 2
stmt:	GEU4(reg,reg)	"\tCMPBLEU32\t%1,%0,%a\n" 2
stmt:	GEU4(INDIRU4(addr),reg)	"\tCMPBLEU32\t%1,%0,%a\n" 2
stmt:	GEF4(reg,reg)	"\tCMPBLEF\t%1,%0,%a\n" 2
stmt:	GEF4(INDIRF4(addr),reg)	"\tCMPBLEF\t%1,%0,%a\n" 2

addr:	ADDRGP2	"%a(pc)" isfptr(a,0,LBURG_MAX)
addr:	ADDRGP2	"%a(dp)" isfptr(a,LBURG_MAX,0)
addr:	ADDRLP2	"%a+%F(sp)"
addr:	ADDRFP2	"%a+4+%F(sp)"

stmt:	reg  ""
stmt:	LABELV	"%a:\n"
reg:	ADDRGP2	"\tlea\t%c,%a(pc)\n" isfptr(a,1,LBURG_MAX)
reg:	ADDRGP2	"\tlea\t%c,%a(dp)\n" isfptr(a,LBURG_MAX,1)
reg:	ADDRLP2	"\tlea\t%c,%a+%F(sp)\n" 1
reg:	ADDRFP2	"\tlea\t%c,%a+2+%F(sp)\n" 1

reg8:	INDIRI1(reg)	"\tld.8\t%c,0(%0)\n"	1
reg8:	INDIRI1(addr)	"\tld.8\t%c,%0\n"	1
reg8:	INDIRU1(reg)	"\tld.8\t%c,0(%0)\n"	1
reg8:	INDIRU1(addr)	"\tld.8\t%c,%0\n"	1
reg:	INDIRI2(reg)	"\tld.16\t%c,0(%0)\n"	1
reg:	INDIRI2(addr)	"\tld.16\t%c,%0\n"	1
reg:	INDIRU2(reg)	"\tld.16\t%c,0(%0)\n"	1
reg:	INDIRU2(addr)	"\tld.16\t%c,%0\n"	1
reg:	INDIRP2(reg)	"\tld.16\t%c,0(%0)\n"	1
reg:	INDIRP2(addr)	"\tld.16\t%c,%0\n"	1

reg:	INDIRI4(reg)	"\tCOPY32\t%c,0(%0)\n"	1
reg:	INDIRI4(addr)	"\tCOPY32\t%c,%0\n"	1
reg:	INDIRU4(reg)	"\tCOPY32\t%c,0(%0)\n"	1
reg:	INDIRU4(addr)	"\tCOPY32\t%c,%0\n"	1
reg:	INDIRF4(reg)	"\tCOPY32\t%c,0(%0)\n"	1
reg:	INDIRF4(addr)	"\tCOPY32\t%c,%0\n"	1

ar:	reg	"%0"
ar:	ADDRGP2	"%a"

reg:	CALLI2(ar)	"\tcall\t%0\n"	1
reg:	CALLU2(ar)	"\tcall\t%0\n"	1
reg:	CALLP2(ar)	"\tcall\t%0\n"	1
reg:	CALLI4(ar)	"\tcall\t%0\n\tst.16\t%c,a\n\tst.16\t2+%c,b\n"	3
reg:	CALLU4(ar)	"\tcall\t%0\n\tst.16\t%c,a\n\tst.16\t2+%c,b\n"	3
reg:	CALLF4(ar)	"\tcall\t%0\n\tst.16\t%c,a\n\tst.16\t2+%c,b\n"	3
stmt:	CALLV(ar)	"\tcall\t%0\n"	1
stmt:	CALLB(ar,reg)	"\tcall\t%0\n" 1

stmt:	RETI2(reg)	"# rtarget should have handled it\n" 1
stmt:	RETU2(reg)	"# rtarget should have handled it\n" 1
stmt:	RETP2(reg)	"# rtarget should have handled it\n" 1
stmt:	RETI4(reg)	"# emit2 needs to return in a/b regpair\n" 1
stmt:	RETU4(reg)	"# emit2 needs to return in a/b regpair\n" 1
stmt:	RETF4(reg)	"# emit2 needs to return in a/b regpair\n" 1
stmt:	RETV(reg)	"# ret\n" 1

stmt:	ARGP2(reg)	"# Let emit2 handle args\n" 1
stmt:	ARGI2(reg)	"# Let emit2 handle args\n" 1
stmt:	ARGU2(reg)	"# Let emit2 handle args\n" 1
stmt:	ARGI4(reg)	"# Let emit2 handle args\n" 1
stmt:	ARGU4(reg)	"# Let emit2 handle args\n" 1
stmt:	ARGF4(reg)	"# Let emit2 handle args\n" 1


stmt:	JUMPV(ar)	"\tbr\t%0\n"	1

stmt:	ASGNI1(reg,reg8)    	"\tst.8\t0(%0),%1\n"	1
stmt:	ASGNI1(addr,reg8)	"\tst.8\t%0,%1\n"	1
stmt:	ASGNU1(reg,reg8)	"\tst.8\t0(%0),%1\n"	1
stmt:	ASGNU1(addr,reg8)	"\tst.8\t%0,%1\n"	1
stmt:	ASGNI2(reg,reg)		"\tst.16\t0(%0),%1\n"	1
stmt:	ASGNI2(addr,reg)	"\tst.16\t%0,%1\n"	1
stmt:	ASGNU2(reg,reg)		"\tst.16\t0(%0),%1\n"	1
stmt:	ASGNU2(addr,reg)	"\tst.16\t%0,%1\n"	1
stmt:	ASGNP2(reg,reg)		"\tst.16\t0(%0),%1\n"	1
stmt:	ASGNP2(addr,reg)	"\tst.16\t%0,%1\n"	1

stmt:	ASGNI4(reg,reg)		"\tCOPY32\t0(%0),%1\n"	1
stmt:	ASGNI4(addr,reg)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNI4(addr,con4)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNI4(reg,con4)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNU4(reg,reg)		"\tCOPY32\t0(%0),%1\n"	1
stmt:	ASGNU4(addr,reg)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNU4(addr,con4)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNU4(reg,con4)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNF4(reg,reg)		"\tCOPY32\t0(%0),%1\n"	1
stmt:	ASGNF4(addr,reg)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNF4(addr,con4)	"\tCOPY32\t%0,%1\n"	1
stmt:	ASGNF4(reg,con4)	"\tCOPY32\t%0,%1\n"	1

stmt:	ASGNB(reg,INDIRB(reg))	"\tld.16\tc,%a\n\tmemcopy\n"
stmt:	ARGB(INDIRB(reg))	"# let emit2 handle %0\n"

%%

// Emitters
static void progbeg(int argc, char *argv[]) {
    {
	union {
	    char c;
	    int i;
	} u;
	u.i = 0;
	u.c = 1;
	swap = ((int)(u.i == 1)) != IR->little_endian;
    }
    parseflags(argc,argv);

    // Long reg symbols
    longreg[R_L0] = mkreg("L0",R_L0,1,IREG);
    longreg[R_L1] = mkreg("L1",R_L1,1,IREG);
    longreg[R_L2] = mkreg("L2",R_L2,1,IREG);
    longreg[R_L3] = mkreg("L3",R_L3,1,IREG);

    // Reg symbols
    intreg[R_A] = mkreg("a",R_A,1,IREG);
    intreg[R_B] = mkreg("b",R_B,1,IREG);

    // Float symbols
    fltreg[R_F0] = mkreg("F0",R_F0,1,FREG);
    fltreg[R_F1] = mkreg("F1",R_F1,1,FREG);
    fltreg[R_F2] = mkreg("F2",R_F2,1,FREG);
    fltreg[R_F3] = mkreg("F3",R_F3,1,FREG);

    // Set up sets
    longwldcrd = mkwildcard(longreg);
    intwldcrd = mkwildcard(intreg);
    fltwldcrd = mkwildcard(fltreg);

    // Set up temp regs
    tmask[IREG] = (1<<R_L0) | (1<<R_L1) | (1<<R_L2) | (1<<R_L3) | (1<<R_A) | (1<<R_B);
    // Set up register temps - none in our case
    vmask[IREG] = 0;

    // FP regs.
    tmask[FREG] = (1<<R_F0) | (1<<R_F1) | (1<<R_F2) | (1<<R_F3);
    // Set up register temps - none in our case
    vmask[FREG] = 0;
    
    print(";	Magic-1 assembly file, generated by lcc 4.2\n");
    print("_start:\n");
    print("	br	over_ivec\n");
    print("	defw	unhandled_exception	; Interrupt vector 1\n");
    print("	defw	unhandled_exception	; Interrupt vector 2\n");
    print("	defw	unhandled_exception	; Interrupt vector 3\n");
    print("	defw	unhandled_exception	; Interrupt vector 4\n");
    print("	defw	unhandled_exception	; Interrupt vector 5\n");
    print("	defw	unhandled_exception	; Interrupt vector 6\n");
    print("	defw	unhandled_exception	; Interrupt vector 7\n");
    print("	defw	unhandled_exception	; Interrupt vector 8\n");
    print("	defw	unhandled_exception	; Interrupt vector 9\n");
    print("	defw	unhandled_exception	; Interrupt vector a\n");
    print("	defw	unhandled_exception	; Interrupt vector b\n");
    print("	defw	unhandled_exception	; Interrupt vector c\n");
    print("	defw	unhandled_exception	; Interrupt vector d\n");
    print("	defw	unhandled_exception	; Interrupt vector e\n");
    print("	defw	unhandled_exception	; Interrupt vector f\n");
    print("unhandled_exception:\n");
    print("	halt\n");
    print("over_ivec:\n");
    print("	ld.16	a,0x8000\n");
    print("	copy	sp,a\n");
    print("	ld.16	a,0x4000\n");
    print("	copy	dp,a\n");
    print("	call	_main\n");
    print("	halt\n");
    
    current_seg = 0;
}

static Symbol rmap(int opk) {
    switch (optype(opk)) {
	case B:
	case P:
	    return intwldcrd;
	case I:
	case U:
	    if (opsize(opk) <= 2) {
	        return intwldcrd; 
	    } else {
		return longwldcrd;
	    }
	case F:
	    return fltwldcrd;
	default:
	    return 0;
    }
}

static void segment(int n) {
    if (n==current_seg)
	return;
    if (n == CODE)
	print("\tcseg\n");
    else if (n == LIT)
#if 1 // combine lit with dseg
	print("\tdseg\n");
#else
	print("\tlit\n");
#endif
    else if (n == DATA)
	print("\tdseg\n");
    else if (n == BSS)
	print("\tbss\n");
    else
	print("\tERROR - unknown segment %d\n",n);
    current_seg = n;
}

static void progend(void) {
    print("\tend\n");
}

int iscvt(Node p) {
    return ((generic(p->op)==CVI) || (generic(p->op)==CVU));
}

static void target(Node p) {
    assert(p);
    switch (specific(p->op)) {

	case NEG+I:
	case NEG+U:
	    if (opsize(p->op)<=2) {
		rtarget(p,0,intreg[R_B]);
		setreg(p,intreg[R_A]);
		if (iscvt(p->kids[0])) {
		    rtarget(p->kids[0],0,intreg[R_A]);
		}
	    }
	    break;

	case ADD+I:
	case ADD+U:
	case SUB+I:
	case SUB+U:
	case BOR+I:
	case BOR+U:
	case BAND+I:
	case BAND+U:
	case BXOR+I:
	case BXOR+U:
	case LSH+I:
	    if (opsize(p->op)<=2) {
	        rtarget(p,0,intreg[R_A]);
		if (iscvt(p->kids[0])) {
		    rtarget(p->kids[0],0,intreg[R_A]);
		}
	        setreg(p,intreg[R_A]);
	    }
	    break;
	
	case MUL+I:
	case MUL+U:
	case DIV+I:
	case DIV+U:
	case MOD+I:
	case MOD+U:
	    if (opsize(p->op)<=2) {
		rtarget(p,0,intreg[R_A]);
		if (iscvt(p->kids[0])) {
		    rtarget(p->kids[0],0,intreg[R_A]);
		}
		rtarget(p,1,intreg[R_B]);
		setreg(p,intreg[R_A]);
	    } 
	    break;

	case CALL+I:
	case CALL+U:
	case CALL+P:
	    if (opsize(p->op)<=2) {
   	        setreg(p,intreg[R_A]);
	    }
	    break;

	case EQ+I:
	case EQ+U:
	case NE+I:
	case NE+U:
	case LE+I:
	case LE+U:
	case LT+I:
	case LT+U:
	    if (opsize(p->op) <= 2) {
		rtarget(p,0,intreg[R_A]);
		if (iscvt(p->kids[0])) {
		    rtarget(p->kids[0],0,intreg[R_A]);
		}
		rtarget(p,1,intreg[R_B]);
	    } 
	    break;

	    // Swapping operands for these to normalize to LE & LT
	case GT+I:
	case GT+U:
        case GE+I:
	case GE+U:
	    if (opsize(p->op) <= 2) {
		rtarget(p,1,intreg[R_A]);
		if (iscvt(p->kids[1])) {
		    rtarget(p->kids[1],0,intreg[R_A]);
		}
		rtarget(p,0,intreg[R_B]);
	    }
	    break;
	
	case CALL+B:
	    rtarget(p,1,intreg[R_A]);
	    break;

	case RET+I:
	case RET+U:
	case RET+P:
	    if (opsize(p->op) <= 2) {
		rtarget(p,0,intreg[R_A]);
	    }
	    break;

        case CVI+U:
	case CVU+U:
	case CVU+I:
	case CVI+F:
	case CVF+I:
	    // Assumption here is that if result is 2, source can be any 4-byte pseudo reg
	    // If result is > 2, then source should be register A
	    if (opsize(p->op) == 2) {
		setreg(p,intreg[R_A]);
	    } else {
		rtarget(p,0,intreg[R_A]);
	    }
	    break;

	case ARG+B:
	    rtarget(p->kids[0],0,intreg[R_B]);
	    break;

	case ASGN+B:
	    rtarget(p,0,intreg[R_A]);
	    rtarget(p->kids[1],0,intreg[R_B]);
	    break;
        case INDIR+I:
        case INDIR+U:
        case INDIR+F:
	    if (opsize(p->op) > 2) {
		rtarget(p,0,intreg[R_A]);
	    }
	    break;
    }
}

static void clobber(Node p) {
    assert(p);
    switch(specific(p->op)) {
	case CALL+F:
	case CALL+I:
	case CALL+U:
	case CALL+V:
	    spill(1<<R_B,IREG,p);
	    break;
        case RET+I:
        case RET+U:
        case RET+F:
	    if (opsize(p->op) > 2) {
	        spill(1<<R_A,IREG,p);
	    }
	    break;
        case ARG+B:
	    // Already targeting B, so don't clobber it
	    spill(1<<R_A,IREG,p);
	    break;
	case INDIR+I:
	case INDIR+U:
	case INDIR+F:
	    if (opsize(p->op) >2) {
		spill(1<<R_B,IREG,p);
	    }
	    break;
	default:
	    if (opsize(p->op) > 2) {
		spill((1<<R_A)|(1<<R_B),IREG,p);
	    }
	    break;
    }
}


static void emit2(Node p) {
    int op = specific(p->op); 

    switch( op ) {
       case ARG+F: 
       case ARG+P: 
       case ARG+I: 
       case ARG+U: 
	   if (opsize(p->op) <= 2) {
	       print("\tst.16\t%d(sp),%s\n",p->syms[2]->u.c.v.i,p->kids[0]->syms[2]->x.name); 
	   }
           break;
	
        case ARG+B:
	       print("\tld.16\tc,%d\n\tlea\ta,%d(sp)\n\tmemcopy\n",p->syms[0]->u.c.v.i,p->syms[2]->u.c.v.i);
	   break;

        case RET+I:
	case RET+U:
	case RET+F:
	   if (opsize(p->op) > 2) {
	       print("\tld.16\ta,%s\n\tld.16\tb,4+%s\n",
		       p->kids[0]->syms[2]->x.name,p->kids[0]->syms[2]->x.name);
	   }
	   break;
    }
}

static void doarg(Node p) {
    static int argno;
    if (argoffset==0) {
	argoffset = 2;
	argno = 0;
    }
    p->x.argno=argno++;
    p->syms[2] = intconst(mkactual(1,p->syms[0]->u.c.v.i));
}

// Block operators not needed
static void blkfetch(int k, int off, int reg, int tmp) {}
static void blkstore(int k, int off, int reg, int tmp) {}
static void blkloop(int dreg, int doff, int sreg, int soff,int size, int tmps[]) {}

static void local(Symbol p) {
    if (isfloat(p->type)) {
	p->sclass = AUTO;
    }
    if (askregvar(p,(*IR->x.rmap)(ttob(p->type)))==0) {
	mkauto(p);
    }
}

static void function(Symbol f, Symbol caller[], Symbol callee[], int n) {
    int i;

    print("%s:\n",f->x.name);
    usedmask[0] = usedmask[1] = 0;
    freemask[0] = freemask[1] = ~(unsigned)0;

    offset = 0;
    for (i=0; callee[i]; i++) {
	Symbol p = callee[i];
	Symbol q = caller[i];
	assert(q);
	p->x.offset = q->x.offset = offset; /* + A_T0_STORE_SIZE; */
	p->x.name = q->x.name = stringf("%d",p->x.offset);
	p->sclass = q->sclass = AUTO;
	offset += roundup(q->type->size,2);
    }
    assert(caller[i] == 0);
    maxoffset = offset = 0;
    maxargoffset = 0;

    // Generate code
    gencode(caller,callee);

    // Allocate space for any pseudo regs we used
    for (i=R_L0;i<=R_L3;i++) {
	if (usedmask[0] & (1<<i)) {
	    maxoffset+=4;
	    longreg[i]->x.offset = -maxoffset;
	}
    }
    for (i=R_F0;i<=R_F3;i++) {
	if (usedmask[1] & (1<<i)) {
	    maxoffset+=4;
	    fltreg[i]->x.offset = -maxoffset;
	}
    }

    // Now, set the frame size
    framesize = maxoffset + maxargoffset + 2;

    // Rewrite names of used long and float regs now that we know framesize
    for (i=R_L0;i<=R_L3;i++) {
	if (usedmask[0] & (1<<i)) {
	    longreg[i]->x.name = stringf("%d(sp)",longreg[i]->x.offset+framesize);
	}
    }
    for (i=R_F0;i<=R_F3;i++) {
	if (usedmask[1] & (1<<i)) {
	    fltreg[i]->x.name = stringf("%d(sp)",fltreg[i]->x.offset+framesize);
	}
    }


    // Gen entry code
    print("\tenter\t%d\n",framesize-2);
    if (isstruct(freturn(f->type))) {
	print("\tst.16\t-2+%d(sp),a\n",framesize);
    }
    emitcode();
#if 0
    print("\tleave\n");
    printf("\tret\n");
#else
    print("\tpop\tsp\n");
    print("\tpop\tpc\n");
#endif
}

static void defsymbol(Symbol p) {
    if (p->scope >= LOCAL && p->sclass == STATIC)
	p->x.name = stringf("L%d", genlabel(1));
    else if (p->generated)
	p->x.name = stringf("L%s", p->name);
    else if (p->scope == GLOBAL || p->sclass == EXTERN)
	p->x.name = stringf("_%s",p->name);
    else if (p->scope == CONSTANTS
	    && (isint(p->type) || isptr(p->type))
	    && p->name[0] == '0' && p->name[1] == 'x')
	p->x.name = stringf("0%sH", &p->name[2]);
    else
	p->x.name = p->name;
}

static void address(Symbol q, Symbol p, long n) {
        if (p->scope == GLOBAL
        || p->sclass == STATIC || p->sclass == EXTERN)
                q->x.name = stringf("%s%s%D",
                        p->x.name, n >= 0 ? "+" : "", n);
        else {
                assert(n <= INT_MAX && n >= INT_MIN);
                q->x.offset = p->x.offset + n;
                q->x.name = stringd(q->x.offset);
        }
}

static void defconst(int suffix, int size, Value v) {
        if (suffix == I && size == 1)
                print("	defb 0%xH\n",   v.u & 0xff);
        else if (suffix == I && size == 2)
                print("	defw 0%xH\n",   v.i & 0xffff);
        else if (suffix == U && size == 1)
                print("	defb 0%xH\n", v.u & 0xff);
        else if (suffix == U && size == 2)
                print("	defw 0%xH\n",   v.i & 0xffff);
        else if (suffix == P && size == 2)
                print("	defw 0%xH\n", v.u & 0xffff);
        else if (suffix == F && size == 4) {
                float f = (float) v.d;
                print("	defw 0%xH\n", ((*(unsigned *)&f)>>16) & 0xffff);
                print("	defw 0%xH\n", (*(unsigned *)&f) & 0xffff);
        } else if (suffix == I && size == 4) {
                print("	defw 0%xH\n",   (v.i>>16) & 0xffff);
                print("	defw 0%xH\n",   v.i & 0xffff);
        } else if (suffix == U && size == 4) {
                print("	defw 0%xH\n",   (v.u>>16) & 0xffff);
                print("	defw 0%xH\n",   v.u & 0xffff);
	}
        else assert(0);
}
static void defaddress(Symbol p) {
        print("	defw %s\n", p->x.name);
}

static void defstring(int n, char *str) {
        char *s;
        for (s = str; s < str + n; s++)
                print("	defb %d\n", (*s)&0377);
}

static void export(Symbol p) {
    print("\tglobal %s\n", p->x.name);
}

static void import(Symbol p) {
    print("\textern %s\n", p->x.name);
}

static void global(Symbol p) {
        assert(p->type->align == 1);
        print("%s:\n", p->x.name);
        if (p->u.seg == BSS)
                print("	defs %d\n", p->type->size);
}

static void space(int n) {
        if (current_seg != BSS)
                print("db %d dup (0)\n", n);
}

static int isfptr(Node n , int iftrue , int iffalse) {
    if ( !n->syms[0]->generated && isfunc(n->syms[0]->type))
       return iftrue;
    else
       return iffalse;
}



Interface m1IR = {
        1, 1, 0,  /* char */
        2, 1, 0,  /* short */
        2, 1, 0,  /* int */
        4, 1, 1,  /* long */
        4, 1, 1,  /* long long */
        4, 1, 1,  /* float */
        4, 1, 1,  /* double */
        4, 1, 1,  /* long double */
        2, 1, 0,  /* T * */
        0, 1, 0,  /* struct */
        0,        /* little_endian */
        1,        /* mulops_calls */
        1,        /* wants_callb */
        1,        /* wants_argb */
        1,        /* left_to_right */
        0,        /* wants_dag */
        0,        /* unsigned_char */
        address,
        blockbeg,
        blockend,
        defaddress,
        defconst,
        defstring,
        defsymbol,
        emit,
        export,
        function,
        gen,
        global,
        import,
        local,
        progbeg,
        progend,
        segment,
        space,
        0, 0, 0, 0, 0, 0, 0,
        {1, rmap,
            blkfetch, blkstore, blkloop,
            _label,
            _rule,
            _nts,
            _kids,
            _string,
            _templates,
            _isinstruction,
            _ntname,
            emit2,
            doarg,
            target,
            clobber,
}
};
static char rcsid[] = "$Id: $";
