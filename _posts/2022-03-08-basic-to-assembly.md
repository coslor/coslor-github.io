# Converting BASIC to Assembly

|Type           |BASIC structure                |Assembly Version                       |Notes                                |
|---------------|-------------------------------|---------------------------------------|-------------------------------------|
| Variables     | `AU=42`                       | AnswerToUniverse:  .byte 42           | Defines the variable[^1], and loads it with the numeric value 42. Labels can be any length. AnswerToUniverse is a single byte, range -128 to 127 or 0-255 |
|               | `AU=PEEK(42)`                 | `lda 42`<BR> `sta AnswerToUniverse`<BR> | Translation: "Load the current value of memory location 42 into the accumulator, and store it in the location represented by AnswerToUniverse."<br>Assumes that AnswerToUniverse has already been defined and is a single byte. Note also that this will overwrite anything previously in the acculumator. |
|               | 

[^1]: But what address does AnswerToUniverse actually represent? No, it's not 42; 42 is the value that the address gets set to when the program first runs. So what is the address? That depends
