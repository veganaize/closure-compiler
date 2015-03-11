# Introduction

TODO: add details about the translation process.

# Extracting Messages

Here is a simple class that used the Closure Compiler's Java API to extract messages.

```java
    import java.io.IOException;
    import java.lang.CharSequence;
    import java.lang.String;
    import java.util.Collection;
    
    import com.google.javascript.jscomp.JsMessage;
    import com.google.javascript.jscomp.JsMessageExtractor;
    import com.google.javascript.jscomp.SourceFile;
    import com.google.javascript.jscomp.GoogleJsMessageIdGenerator;
    
    public class ExtractMessages {
        // args[0]: project name
        // args[1]: file name
        public static void main(String[] args) throws IOException {
            JsMessageExtractor extractor = new JsMessageExtractor(
              new GoogleJsMessageIdGenerator(args[0]), JsMessage.Style.CLOSURE);
            Collection<JsMessage> messages = extractor.extractMessages(
              SourceFile.fromFile(args[1]));
            for (JsMessage message : messages) {
                System.out.println("desc: " + message.getDesc());
                System.out.println("id: " + message.getId());
                System.out.println("key: " + message.getKey());
                System.out.println("source name: " + message.getSourceName());
                System.out.println("Parts:");
                for (CharSequence cs : message.parts()) {
                    System.out.println("...." + cs.toString());
                }
            }
        }
    }
```

# Example Message

```javascript
    /** @desc context menu - make a duplicate of the selected block */
    MyProject.MSG_DUPLICATE_BLOCK = goog.getMsg("Duplicate");
```

# Example Messages File

The translation id was found by running the above program with the command line:
    java -classpath .:compiler.jar ExtractMessages MyMessages messages.js

fr.xmb:
```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE translationbundle SYSTEM "translationbundle.dtd">
    <translationbundle lang="fr">
      <translation id="5213219513082471002">Dupliquer</translation>
    </translationbundle>
```

# Using the translation file

    java -jar compiler.jar --js messages.js --translations_project MyMessages --translations_file fr.xtb 

# Notes

The "translation project" is used as part of the message id, so the same value must be provided to the message extractor and to the compiler when replacing messages.