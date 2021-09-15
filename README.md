# Cobol Copybook JSONifier

[CHANGELOG.md](CHANGELOG.md)

----

## What is that for?

The motivator for that was the necessity to transfer and process some data files from Mainframe on cloud in a easy way for cloud systems.

The main idea here is to convert files from Mainframe that was generated by cobol programs using copybooks into json's registry format.

Cobol copybooks are like data schemas for raw positional files.

To use the package you need:
1. Extract a cobol copybook structure to generate a python dictionary. This dictionary contains objects neededs to build the parser.

2. Build a Parser based on the previous copybook's struture extracted.The parser contains python objects that will handle each kind of data field in the Mainframe's file. That file could be eater EBCDI or ASCII.
    > **Note:**  
    > Types such as: `Binary` _(BINARY, COMP)_, `COMP3`, `Numeric Signed` _( S9(n) )_ are supported in EBCDIC type and needs to be transfer in binary mode from Mainframe.  
    > ASCII mode suports formats like `PIC 9(n)`, `PIC X(n)` and Masked `PIC +99999.99`. These formats can be converted from EBCDIC to ASCII when transfered by FTP, Connect Dirct or other tools from mainframe to cloud or another platform.


**These data lines parsed into json could be used to create events, call rest apis and what else you can imagine.**


![coboljsonifier image](docs/COBOL_JSONIFIER.png)


----

## Straight to the point

Take a look at [pypi.org](https://pypi.org/project/coboljsonifier/)

``` bash
pip install coboljsonifier
```

Use the script `prepare_test.py` to generate files that will be used for test scripts `parser_ebcdic.py` and `parser_ascii.py`:
* `EBCDIC_BOOK.cob` & `EBCDIC_DATA.bin`
* `ASCII_BOOK.cob` & `ASCII_DATA.bin`

### prepare_test.py
``` python

print('STARTED...')

# --------------------------- EBCDIC BINARY CONTENT ----------------------------
print('Generating for ECBDIC')

copybookfname = 'EBCDIC_BOOK.cob'
datafname     = 'EBCDIC_DATA.bin'

copybook_ebcdic = """
      *================================================================*00010000
      *                 COBOL COPYBOOK EXAMPLE - EBCDIC                *00020000
      * -------------------------------------------------------------- *00021000
      * NOTE: This copybook handle binary content that in general are  *00022000
      *       written in EBCDIC content in Mainframes and to use it    *00023000
      *       in other platforms it's necessary to transfer than in    *00024000
      *       binary to preserv their contents.                        *00025000
      *       In FTP transfer use BINARY.                              *00026000
      *                                Joao Roberto Perin - 2021-09-15 *00027000
      *================================================================*00030000
      *                                           REGISTRY LENGHT: 100 *00040000
      *================================================================*00050000
       01 :DATA1:-DETAIL-REGISTRY.                                      00060000
          03 :DATA1:-REGISTRY-TYPE             PIC 9(002).              00070000
             88 :DATA1:-REGISTRY-TYPE-HEADER     VALUE 01.              00080000
             88 :DATA1:-REGISTRY-TYPE-DETAIL     VALUE 02.              00090000
             88 :DATA1:-REGISTRY-TYPE-TRAILLER   VALUE 99.              00100000
          03 :DATA1:-COMPANY                   PIC 9(003).              00110000
          03 :DATA1:-USER-ACCOUNT              PIC X(019).              00120000
          03 :DATA1:-BIRTH-DATE                PIC X(010).              00130000
COMM  *      BIRTH-DATE: YYYY-MM-DD                                     00140000
          03 :DATA1:-NAME                      PIC X(040).              00150000
          03 :DATA1:-CREDIT-LIMIT              PIC S9(07) USAGE BINARY. 00160000
          03 :DATA1:-LIMIT-USED                PIC S9(05)V99 COMP-3.    00170000
          03 :DATA1:-STATUS OCCURS 4 TIMES.                             00180000
             05 :DATA1:-STATUS-FLAG            PIC X(001).              00190000
RESERV    03 FILLER                            PIC X(014).              00200000
      *================================================================*00210000
      *                               END                              *00220000
      *================================================================*00230000
"""

ebcdicdata1 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF1\xF1\xF1\xF1\xF1\xF9\xF7\xF1\x60\xF0\xF1\x60\xF2\xF1\xD1\xD6\xC8\xD5\x40\xD9\xD6\xC2\xC5\xD9\xE3\x40\xD7\xC5\xD9\xC9\xD5\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x00\x00\x03\xE9\x01\x00\x01\x0D\xF1\xF2\xF3\xF4\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'
ebcdicdata2 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF2\xF2\xF2\xF2\xF1\xF9\xF8\xF2\x60\xF0\xF2\x60\xF2\xF2\xD4\xC1\xD9\xE8\x40\xC1\xD5\xD5\x40\xC7\xC1\xD9\xC3\xC9\xC1\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\xFF\xFF\xFF\x85\x01\x00\x02\x0C\xF5\xF6\xF7\xF8\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'
ebcdicdata3 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF3\xF3\xF3\xF3\xF1\xF9\xF9\xF3\x60\xF0\xF3\x60\xF2\xF3\xD7\xC5\xE3\xC5\xD9\x40\xD7\xC1\xD9\xD2\xC5\xD9\x40\xC2\xD9\xC1\xE9\xC9\xD3\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x00\x00\x00\x7B\x03\x00\x03\x0C\xF9\xF0\xC1\xC2\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'

print('Generating EBCDIC Copybook...')
with open(copybookfname, 'w') as f:
    f.write(copybook_ebcdic)

print("Generating Data File...")

with open(datafname, 'wb') as f:
    f.write(ebcdicdata1)
    f.write(ebcdicdata2)
    f.write(ebcdicdata3)

# --------------------------- ASCII TEXT CONTENT -------------------------------
print('Generating for ASCII')

copybookfname = 'ASCII_BOOK.cob'
datafname     = 'ASCII_DATA.bin'

copybook_ascii = """
      *================================================================*00010000
      *                 COBOL COPYBOOK EXAMPLE - ASCII                 *00020000
      * -------------------------------------------------------------- *00021000
      * NOTE: This copybook handle ascii text content that in general  *00022000
      *       is written in EBCDIC codepage in Mainframes and is       *00023000
      *       converted in transfer methods to ASCII. To convert it,   *00024000
      *       the content needs to be in plain EBCDIC text such as     *00025000
      *       PIC X(n), PIC A(n), PIC +99999.99, PIC +ZZZZZ9.99.       *00026000
      *       In FTP transfer use TEXT.                                *00027000
      *                                Joao Roberto Perin - 2021-09-15 *00028000
      *================================================================*00030000
      *                                           REGISTRY LENGHT: 100 *00040000
      *================================================================*00050000
       01 :DATA1:-DETAIL-REGISTRY.                                      00060000
          03 :DATA1:-REGISTRY-TYPE             PIC 9(002).              00070000
             88 :DATA1:-REGISTRY-TYPE-HEADER     VALUE 01.              00080000
             88 :DATA1:-REGISTRY-TYPE-DETAIL     VALUE 02.              00090000
             88 :DATA1:-REGISTRY-TYPE-TRAILLER   VALUE 99.              00100000
          03 :DATA1:-COMPANY                   PIC 9(003).              00110000
          03 :DATA1:-USER-ACCOUNT              PIC X(019).              00120000
          03 :DATA1:-BIRTH-DATE                PIC X(010).              00130000
COMM  *      BIRTH-DATE: YYYY-MM-DD                                     00140000
          03 :DATA1:-NAME                      PIC X(040).              00150000
          03 :DATA1:-CREDIT-LIMIT              PIC +9999999.            00160000
          03 :DATA1:-LIMIT-USED                PIC +99999.99.           00170000
          03 :DATA1:-STATUS OCCURS 4 TIMES.                             00180000
             05 :DATA1:-STATUS-FLAG            PIC X(001).              00190000
RESERV    03 FILLER                            PIC X(003).              00200000
      *================================================================*00210000
      *                               END                              *00220000
      *================================================================*00230000
"""

asciidata1 = "0200400400000000900011111971-01-21JOHN ROBERT PERIN                       +0001001-00100.101234   " + "\n"
asciidata2 = "0200400400000000900011111971-01-21MARY ANN GARCIA                         -0000123+01000.205678   " + "\n"
asciidata3 = "0200400400000000900011111971-01-21JPETER PARKER BRAZIL                    +0000123+03000.3090AB   " + "\n"

print('Generating ASCII Copybook...')

with open(copybookfname, 'w') as f:
    f.write(copybook_ascii)

print("Generating Data File...")

with open(datafname, 'w') as f:
    f.write(asciidata1)
    f.write(asciidata2)
    f.write(asciidata3)

print("ENDED...")
```

### parser_ebcdic.py

``` python
import simplejson
from coboljsonifier.copybookextractor import CopybookExtractor
from coboljsonifier.parser import Parser
from coboljsonifier.config.parser_type_enum import ParseType

bookfname='EBCDIC_BOOK.cob'

dict_structure = CopybookExtractor(bookfname).dict_book_structure
print(simplejson.dumps(dict_structure))

# Use book structure to build a parser (FLAT_ASCII / BINARY_EBCDIC)
parser = Parser(dict_structure, ParseType.BINARY_EBCDIC).parser
size = parser.size
print("// Registry calculated lenght:", size)
print("// " + "-" * 70)

datafname = 'EBCDIC_DATA.bin'
i = 0
''' Important Note! 
    ebcdic file: Open the file with rb "read binary" and f2.read(size)
    ascii file : Open the file with  r "read text" and f2.readline()
'''
with open(datafname, 'rb') as f2:
    while True:

        # EBCDIC
        data = f2.read(size)
        if not data:
            break
        i += 1

        print(f"\n// Registry : {i} ","-" * 50)

        if data[0:2] == b'\xF0\xF2':    # for EBCDIC
            parser.parse(data)
            dictvalue = parser.value

            # ALERT: Don't use json.dumps. It doesn't threat Decimal formats - float for monetary values 
            # Use simplejson instead, it has support for Decimals
            print(simplejson.dumps(dictvalue))
        else:
            print(f'// Registry type {data[0:2]} not processed')

print(f"// " + "-" * 70)
print(f"// Total processed {i}")
print()
```

**DATA1_BOOK.cob**
``` cobol
      *================================================================*00010000
      *                      COBOL EXAMPLE COPYBOOK                    *00020000
      * -------------------------------------------------------------- *00021000
      * NOTE: This copybook handle binary content that in general are  *00022000
      *       written in EBCDIC content in Mainframes and to use it    *00023000
      *       in other platforms it's necessary to transfer than in    *00024000
      *       binary to preserv their contents.                        *00025000
      *                                Joao Roberto Perin - 2021-09-15 *00026000
      *================================================================*00030000
      *                                           REGISTRY LENGHT: 100 *00040000
      *================================================================*00050000
       01 :DATA1:-DETAIL-REGISTRY.                                      00060000
          03 :DATA1:-REGISTRY-TYPE             PIC 9(002).              00070000
             88 :DATA1:-REGISTRY-TYPE-HEADER     VALUE 01.              00080000
             88 :DATA1:-REGISTRY-TYPE-DETAIL     VALUE 02.              00090000
             88 :DATA1:-REGISTRY-TYPE-TRAILLER   VALUE 99.              00100000
          03 :DATA1:-COMPANY                   PIC 9(003).              00110000
          03 :DATA1:-USER-ACCOUNT              PIC X(019).              00120000
          03 :DATA1:-BIRTH-DATE                PIC X(010).              00130000
COMM  *      BIRTH-DATE: YYYY-MM-DD                                     00140000
          03 :DATA1:-NAME                      PIC X(040).              00150000
          03 :DATA1:-CREDIT-LIMIT              PIC S9(07) USAGE BINARY. 00160000
          03 :DATA1:-LIMIT-USED                PIC S9(05)V99 COMP-3.    00170000
          03 :DATA1:-STATUS OCCURS 4 TIMES.                             00180000
             05 :DATA1:-STATUS-FLAG            PIC X(001).              00190000
RESERV    03 FILLER                            PIC X(010).              00200000
      *================================================================*00210000
      *                               END                              *00220000
      *================================================================*00230000
```

**DATA1_DATA.bin - Generator**
``` python
# This little python script generates 3 lines in ebcdic
# To be parsed based on DATA1_BOOK.cob

print('STARTED...')

line1 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF1\xF1\xF1\xF1\xF1\xF9\xF7\xF1\x60\xF0\xF1\x60\xF2\xF1\xD1\xD6\xC8\xD5\x40\xD9\xD6\xC2\xC5\xD9\xE3\x40\xD7\xC5\xD9\xC9\xD5\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x00\x00\x03\xE9\x01\x00\x01\x0D\xF1\xF2\xF3\xF4\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'
line2 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF2\xF2\xF2\xF2\xF1\xF9\xF8\xF2\x60\xF0\xF2\x60\xF2\xF2\xD4\xC1\xD9\xE8\x40\xC1\xD5\xD5\xC7\xC1\xD9\xC3\xC9\xC1\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\xFF\xFF\xFF\x85\x01\x00\x02\x0C\xF5\xF6\xF7\xF8\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'
line3 = b'\xF0\xF2\xF0\xF0\xF4\xF0\xF0\xF4\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF0\xF9\xF0\xF0\xF0\xF3\xF3\xF3\xF3\xF1\xF9\xF9\xF3\x60\xF0\xF3\x60\xF2\xF3\xD7\xC5\xE3\xC5\xD9\x40\xD7\xC1\xD9\xD2\xC5\xD9\xC2\xD9\xC1\xE9\xC9\xD3\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x00\x00\x00\x7B\x03\x00\x03\x0C\xF9\xF0\xC1\xC2\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40\x40'

datafname = './tests/test_files/DATA1_DATA.bin'

with open(datafname, 'wb') as f:
    f.write(line1)
    f.write(line2)
    f.write(line3)

print("ENDED...")
```

**DATA1_BOOK.cob Structure Extracted**
``` json
{"DATA1-REGISTRY-TYPE": {"type": "NUMERIC", "level": 3, "name": "DATA1-REGISTRY-TYPE", "format": "9(002)", "subformat": "NORMAL", "decimals": 0, "size": 2, "start": 0, "occurs": null}, "DATA1-COMPANY": {"type": "NUMERIC", "level": 3, "name": "DATA1-COMPANY", "format": "9(003)", "subformat": "NORMAL", "decimals": 0, "size": 3, "start": 2, "occurs": null}, "DATA1-USER-ACCOUNT": {"type": "ALPHANUMERIC", "level": 3, "name": "DATA1-USER-ACCOUNT", "format": "X(019)", "subformat": "NORMAL", "decimals": 0, "size": 19, "start": 5, "occurs": null}, "DATA1-BIRTH-DATE": {"type": "ALPHANUMERIC", "level": 3, "name": "DATA1-BIRTH-DATE", "format": "X(010)", "subformat": "NORMAL", "decimals": 0, "size": 10, "start": 24, "occurs": null}, "DATA1-NAME": {"type": "ALPHANUMERIC", "level": 3, "name": "DATA1-NAME", "format": "X(040)", "subformat": "NORMAL", "decimals": 0, "size": 40, "start": 34, "occurs": null}, "DATA1-CREDIT-LIMIT": {"type": "NUMERIC_BINARY", "level": 3, "name": "DATA1-CREDIT-LIMIT", "format": "S9(07)", "subformat": "BINARY", "decimals": 0, "size": 4, "start": 74, "occurs": null}, "DATA1-LIMIT-USED": {"type": "NUMERIC_COMP3", "level": 3, "name": "DATA1-LIMIT-USED", "format": "S9(05)V99", "subformat": "COMP-3", "decimals": 2, "size": 4, "start": 78, "occurs": null}, "DATA1-STATUS": [{"DATA1-STATUS-FLAG": {"type": "ALPHANUMERIC", "level": 5, "name": "DATA1-STATUS-FLAG", "format": "X(001)", "subformat": "NORMAL", "decimals": 0, "size": 1, "start": 82, "occurs": null}}, {"DATA1-STATUS-FLAG": {"type": "ALPHANUMERIC", "level": 5, "name": "DATA1-STATUS-FLAG", "format": "X(001)", "subformat": "NORMAL", "decimals": 0, "size": 1, "start": 82, "occurs": null}}, {"DATA1-STATUS-FLAG": {"type": "ALPHANUMERIC", "level": 5, "name": "DATA1-STATUS-FLAG", "format": "X(001)", "subformat": "NORMAL", "decimals": 0, "size": 1, "start": 82, "occurs": null}}, {"DATA1-STATUS-FLAG": {"type": "ALPHANUMERIC", "level": 5, "name": "DATA1-STATUS-FLAG", "format": "X(001)", "subformat": "NORMAL", "decimals": 0, "size": 1, "start": 82, "occurs": null}}], "FILLER-1": {"type": "ALPHANUMERIC", "level": 3, "name": "FILLER-1", "format": "X(010)", "subformat": "NORMAL", "decimals": 0, "size": 10, "start": 82, "occurs": null}}
```

**Json Data from file parsed based in the cobolbook**
``` json
// Registry calculated lenght: 100
// ----------------------------------------------------------------------

// Registry : 1  --------------------------------------------------
{"DATA1-REGISTRY-TYPE": 2, "DATA1-COMPANY": 4, "DATA1-USER-ACCOUNT": "0040000000090001111", "DATA1-BIRTH-DATE": "1971-01-21", "DATA1-NAME": "JOHN ROBERT PERIN", "DATA1-CREDIT-LIMIT": 1001, "DATA1-LIMIT-USED": -1000.10, "DATA1-STATUS": [{"DATA1-STATUS-FLAG": "1"}, {"DATA1-STATUS-FLAG": "2"}, {"DATA1-STATUS-FLAG": "3"}, {"DATA1-STATUS-FLAG": "4"}], "FILLER-1": null}

// Registry : 2  --------------------------------------------------
{"DATA1-REGISTRY-TYPE": 2, "DATA1-COMPANY": 4, "DATA1-USER-ACCOUNT": "0040000000090002222", "DATA1-BIRTH-DATE": "1982-02-22", "DATA1-NAME": "MARY ANNGARCIA", "DATA1-CREDIT-LIMIT": -123, "DATA1-LIMIT-USED": 1000.20, "DATA1-STATUS": [{"DATA1-STATUS-FLAG": "5"}, {"DATA1-STATUS-FLAG": "6"}, {"DATA1-STATUS-FLAG": "7"}, {"DATA1-STATUS-FLAG": "8"}], "FILLER-1": null}

// Registry : 3  --------------------------------------------------
{"DATA1-REGISTRY-TYPE": 2, "DATA1-COMPANY": 4, "DATA1-USER-ACCOUNT": "0040000000090003333", "DATA1-BIRTH-DATE": "1993-03-23", "DATA1-NAME": "PETER PARKERBRAZIL", "DATA1-CREDIT-LIMIT": 123, "DATA1-LIMIT-USED": 3000.30, "DATA1-STATUS": [{"DATA1-STATUS-FLAG": "9"}, {"DATA1-STATUS-FLAG": "0"}, {"DATA1-STATUS-FLAG": "A"}, {"DATA1-STATUS-FLAG": "B"}], "FILLER-1": null}
// ----------------------------------------------------------------------
// Total processed 3
```

-----

## Pre-requisites

* To save in json formats is necessary to use <spam style="color:#009900">`simplejson`</spam> package. It's to keep compatibility of `Decimal` pakage that is used to handle monetary values.
* The common `json` library has incompatibility with Decimal values

* Install `simplejson` just running `pip install simplejson`.
* Use `simplejson` the same way of `json`. 
    > Ex.   
    > `install simplejson as json`   
    > It will keep compatibility with the main functionalities.

-----

## Local environment

```bash
$ python -m venv venv
$ source venv/bin/activate
$ pip install -r requirements.txt

```

## Testing

**Undestanding the use with:** `'parser_tester.py'`

``` bash
$ python parser_tester.py
```

**Unit Tests:**
``` bash
# Running only the unittest
$ python -m unittest discover

# Checking test coveraging
$ coverage run -m unittest discover
$ coverage report -m
$ coverage html
```


**Cobol Copybook Example**

``` cobol 
COPY DT00RL
002200 01 :DT00:-RECORD.                                 
002300    03 :DT00:-KEY.                                
002400       05 :DT00:-ORG-ACCT.                       
002500           07 :DT00:-ORG  PIC 999.               
002600           07 :DT00:-ACCT PIC X(19).             
002700       05  :DT00:-STMT-ID-CODE                    
002800                           PIC S9(7)  BINARY.

```

## References

* [Python Packaging Projects](https://packaging.python.org/tutorials/packaging-projects/)

* [Where to put tests](http://pythonchb.github.io/PythonTopics/where_to_put_tests.html)

* [Python Modules](https://docs.python.org/3/tutorial/modules.html)

* [Absolute vs Relative python imports](https://realpython.com/absolute-vs-relative-python-imports/)

* [Python UnitTest](https://pythontesting.net/framework/specify-test-unittest-nosetests-pytest/)