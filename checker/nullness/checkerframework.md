
# checker framework

Version: checker-framework-2.3.0

> Outputs have been simplified for brevity.

The nullness hierarchy contains these qualifiers:

| qualifier | description |
| --- | --- |
| **@Nullable** | Indicates a type that includes the null value. For example, the type Boolean is nullable: a variable of type Boolean always has one of the values TRUE, FALSE, or null. |
| **@NonNull** | Indicates a type that does not include the null value. The type boolean is non-null; a variable of type boolean always has one of the values true or false. The type @NonNull Boolean is also non-null: a variable of type @NonNull Boolean always has one of the values TRUE or FALSE — never null. Dereferencing an expression of non-null type can never cause a null pointer exception. |

> **Important:** The default state of any object in Checker Framework is ***@NonNull***.

The Nullness Checker supports several annotations that specify method behavior. These are 
declaration annotations, not type annotations as they apply to the method itself rather than to 
some particular type:

| annotation | description |
| --- | --- |
| **@RequiresNonNull** | Indicates a method precondition: The annotated method expects the specified variables (typically field references) to be non-null when the method is invoked. |
| **@EnsuresNonNull** | |
| **@EnsuresNonNullIf** | Indicates a method postcondition. With @EnsuresNonNull, the given expressions are non-null after the method returns; this is useful for a method that initializes a field, for example. With @EnsuresNonNullIf, if the annotated method returns the given boolean value (true or false), then the given expressions are non-null. |

**Checker fully qualified names:**
1. org.checkerframework.checker.nullness.NullnessChecker

## intra-procedural

[nullness/IntraProcedural.java](https://github.com/michaelemery/staticanalysis/blob/master/checker/nullness/IntraProcedural.java)

```java
package nullness;

/**
 * Intra-procedural assignment of a null reference.
 */
public class IntraProcedural {

    String s;

    public IntraProcedural(String s) {
        this.s = s;
    }

    public static void main(String[] args) throws NullPointerException {

        // intra-procedural assignment of a non-null reference (correct)
        IntraProcedural foo = new IntraProcedural("text");
        System.out.println(foo.s.toString());  // "text"

        // intra-procedural assignment of a null reference (fail)
        foo = null;
        System.out.println(foo.s.toString());  // NullPointerException

    }

}
```

**results:**

```
$ javac -processor org.checkerframework.checker.nullness.NullnessChecker nullness/IntraProcedural.java

nullness/IntraProcedural.java:22: 
error: [dereference.of.nullable] dereference of possibly-null reference foo
        System.out.println(foo.s.toString());  // NullPointerException
                           ^

1 error
```

| True Pos | False Pos | False Neg |
| :---: | :---: | :---: |
| 1 | 0 | 0 |

## alias

[nullness/Alias.java](https://github.com/michaelemery/staticanalysis/blob/master/checker/nullness/Alias.java)

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
$ javac -processor org.checkerframework.checker.nullness.NullnessChecker nullness/Alias.java

nullness/Alias.java:24: 
error: [dereference.of.nullable] dereference of possibly-null reference bar
        System.out.println(bar.s.toString());  // NullPointerException
                           ^

1 error
```

| True Pos | False Pos | False Neg |
| :---: | :---: | :---: |
| 1 | 0 | 0 |

## inter-procedural

[nullness/InterProcedural.java](https://github.com/michaelemery/staticanalysis/blob/master/checker/nullness/InterProcedural.java)

```java
package nullness;

/**
 * Inter-procedural assignment to a null reference.
 */
public class InterProcedural {

    String s;

    public InterProcedural(String s) {
        this.s = s;
    }

    public static void main(String[] args) throws NullPointerException {

        // inter-procedural assignment of a non-null reference (correct)
        InterProcedural foo = new InterProcedural(returnReceivedString("text"));
        System.out.println(foo.s.toString());  // "text"

        // inter-procedural assignment of a null reference (fail)
        InterProcedural bar = new InterProcedural(returnReceivedString(null));
        System.out.println(bar.s.toString());  // NullPointerException

    }

    public static String returnReceivedString(String s) {
        return s;
    }

}
```

**results:**

```
$ javac -processor org.checkerframework.checker.nullness.NullnessChecker nullness/InterProcedural.java

nullness/InterProcedural.java:21: 
error: [argument.type.incompatible] incompatible types in argument.
        InterProcedural bar = new InterProcedural(returnReceivedString(null));
                                                 ^
  found   : null
  required: @Initialized @NonNull String

1 error
```

| True Pos | False Pos | False Neg |
| :---: | :---: | :---: |
| 1 | 0 | 0 |

## reflection

[nullness/Reflection.java](https://github.com/michaelemery/staticanalysis/blob/master/checker/nullness/Reflection.java)

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
$ javac -processor org.checkerframework.checker.nullness.NullnessChecker nullness/Reflection.java

nullness/Reflection.java:24: error: [argument.type.incompatible] incompatible types in argument.
            Reflection foo = new Reflection((String) m.invoke(null));
                                            ^
  found   : @Initialized @Nullable String
  required: @Initialized @NonNull String

nullness/Reflection.java:24: 
error: [argument.type.incompatible] incompatible types in argument.
            Reflection foo = new Reflection((String) m.invoke(null));
                                                              ^
  found   : null
  required: @Initialized @NonNull Object

nullness/Reflection.java:29: 
error: [argument.type.incompatible] incompatible types in argument.
            Reflection bar = new Reflection((String) m.invoke(null));
                                            ^
  found   : @Initialized @Nullable String
  required: @Initialized @NonNull String

nullness/Reflection.java:29: 
error: [argument.type.incompatible] incompatible types in argument.
            Reflection bar = new Reflection((String) m.invoke(null));
                                                              ^
  found   : null
  required: @Initialized @NonNull Object

nullness/Reflection.java:45: 
error: [return.type.incompatible] incompatible types in return.
        return null;
               ^
  found   : null
  required: @Initialized @NonNull String

5 errors
```

| True Pos | False Pos | False Neg |
| :---: | :---: | :---: |
| 1 | 3 | 0 |
