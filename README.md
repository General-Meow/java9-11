# Java 9-11
The following are notes on the changes from 9 to 11. Not all of them are here, only ones that mnight affect you as a developer

###Java 9
#####Smart compilation
- Just like build tools, smart compilation compiles files that only require compilation
- Also known as Smart Javac
- The tool sjavac
- Made to save time
- Is a smart wrapper around the javac tool

#####Interned Strings in Class Data Sharing (CSD)
- Java 5 to 8, the storage of interned strings was inefficient (both time and storage)
- It was Classes from System jar -> private internal representation -> dump to shared archive
- Strings were encoded as UTF 8 which takes up to 8 bits but most characters only really require 3
- The fix was include 2 new steps after dumping to the shared archive String space allocation -> String table compression
- So Java 9 gives a special area for Strings on the heap then they go into a table thats shared and deduped
- Java 10 improves on this whereby it adds application classes to the shared archive

###Java 10

###Java 11