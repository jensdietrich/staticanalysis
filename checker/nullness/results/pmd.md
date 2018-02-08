
# pmd

Version: pmd-bin-6.0.1

> **Note:**
>
> The `pmd` command shown in the following results is a bash alias created as follows; <br />
> `alias pmd='$PMD_HOME/bin/run.sh pmd'`
>
> JDK must be set to 1.8 or lower.

PMD checkers belonging to the "error prone" category are the only ones required for these tests.

## direct

```java
package nullness;

/**
 * Direct assignment to a null reference.
 */
public class Direct {

    String s;

    public Direct(String s) {
        this.s = s;
    }

    public static void main(String[] args) throws NullPointerException {

        // direct assignment to non-null (correct)
        Direct foo = new Direct("text");
        System.out.println(foo.s.toString());  // "text"

        // direct assignment to null (fail)
        foo = null;
        System.out.println(foo.s.toString());  // NullPointerException

    }

}
```

**results:**

```
$ pmd -d Direct.java -f text -R category/java/errorprone.xml

/Users/michaelemery/Developer/staticanalysis/checker/nullness/Direct.java:8:    
Found non-transient, non-static member. Please mark as transient or provide accessors.
/Users/michaelemery/Developer/staticanalysis/checker/nullness/Direct.java:21:   
Assigning an Object to null is a code smell.  Consider refactoring.
```

*Output has been simplified.*

## alias

```java
package nullness;

/**
 * Assignment to a null reference by alias.
 */
public class Alias {

    String s;

    public Alias(String s) {
        this.s = s;
    }

    public static void main(String[] args) throws NullPointerException {

        // assignment to non-null by alias (correct)
        Alias foo = new Alias("text");
        Alias bar = foo;
        System.out.println(bar.s.toString());  // "text"

        // assignment to null by alias (fail)
        foo = null;
        bar = foo;
        System.out.println(bar.s.toString());  // NullPointerException

    }

}
```

**results:**

```
$ pmd -d Alias.java -f text -R category/java/errorprone.xml

/Users/michaelemery/Developer/staticanalysis/checker/nullness/Alias.java:8: 
Found non-transient, non-static member. Please mark as transient or provide accessors.
/Users/michaelemery/Developer/staticanalysis/checker/nullness/Alias.java:22:    
Assigning an Object to null is a code smell.  Consider refactoring.
```

*Output has been simplified.*

## call

```java
package nullness;

/**
 * Assignment to a null reference from method call.
 */
public class Call {

    String s;

    public Call(String s) {
        this.s = s;
    }

    public static void main(String[] args) throws NullPointerException {

        // assignment to a non-null reference from method call (correct)
        Call foo = new Call(returnReceivedString("text"));
        System.out.println(foo.s.toString());  // "text"

        // assignment to a null reference from method call (fail)
        Call bar = new Call(returnReceivedString(null));
        System.out.println(bar.s.toString());  // NullPointerException

    }

    public static String returnReceivedString(String s) {
        return s;
    }

}
```

**results:**

```
$ pmd -d Call.java -f text -R category/java/errorprone.xml

/Users/michaelemery/Developer/staticanalysis/checker/nullness/Call.java:8:  
Found non-transient, non-static member. Please mark as transient or provide accessors.
```

*Output has been simplified.*

## reflection

```java
package nullness;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * Assignment to a null reference by reflection.
 */
public class Reflection {

    String s;

    public Reflection(String s) {
        this.s = s;
    }

    public static void main(String[] args) {

        Method m;

        try {
            // assignment to a non-null reference by reflection (correct)
            m = nullness.Reflection.class.getDeclaredMethod("returnText");
            Reflection foo = new Reflection((String) m.invoke(null));
            System.out.println(foo.s.toString());  // "text"

            // assignment to a null reference by reflection (fail)
            m = nullness.Reflection.class.getDeclaredMethod("returnNull");
            Reflection bar = new Reflection((String) m.invoke(null));
            System.out.println(bar.s.toString());  // NullPointerException
            
        } catch (NoSuchMethodException |
                IllegalAccessException |
                InvocationTargetException x) {
            x.printStackTrace();
        }

    }

    public static String returnText() {
        return "text";
    }

    public static String returnNull() {
        return null;
    }

}
```

**results:**

```
$ pmd -d Reflection.java -f text -R category/java/errorprone.xml

/Users/michaelemery/Developer/staticanalysis/checker/nullness/Reflection.java:11:   
Found non-transient, non-static member. Please mark as transient or provide accessors.
```

*Output has been simplified.*
