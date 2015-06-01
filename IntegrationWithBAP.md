# Deadline is September 29th #

![http://i.imgur.com/nRJoPnc.jpg](http://i.imgur.com/nRJoPnc.jpg)

# Details #
```
  * getAddressForName -- like main is at 0x8048200
  * tags[address]['name', 'comment', 'len'] -- both read and write
  * getFunctionAddressIsIn -- return -1 for no function
  * getInstructionsInFunctionAndFlow -- return list of instructions in a function, codepath flow
  * getInstructionsFromAddress -- takes a count of things to get
  * createFunctionAtAddress -- IDA's p key
  * makeCodeAtAddress -- IDA's c key
  * getXrefsTo -- return xrefs to this instruction or data

  # Tags will hang around for the javascript to python layer
```