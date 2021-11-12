# Coding Conventions
## Code Style
We currently use a modified version of [Jetbrains' Java Style](https://www.jetbrains.com/help/idea/code-style-java.html) 
for our coding convention.  
The base repository has an [`Hartshorn Code Style.xml`](https://github.com/GuusLieben/Hartshorn/blob/hartshorn-main/Hartshorn%20Code%20Style.xml) file. If you use any Jetbrains IDE you can import this under `Editor > Code Style`.  
Alternatively, you can also use the [.editorconfig](https://github.com/GuusLieben/Hartshorn/blob/hartshorn-main/.editorconfig.xml) file with any IDE that supports it.  
Additionally, you should use [`Hartshorn Code Inspections.xml`](https://github.com/GuusLieben/Hartshorn/blob/hartshorn-main/Hartshorn%20Code%20Inspections.xml), which can be imported under `Editor > Inspections`.

## Type Naming
Naming conventions follow [Oracle's Code Conventions for Java](https://www.oracle.com/java/technologies/javase/codeconventions-namingconventions.html#:~:text=Class%20names%20should%20be%20nouns,such%20as%20URL%20or%20HTML).
Two additional rules to these conventions are present for types which are registered to Hartshorn's internal injector:
- Partial (abstract) implementations of interfaces are prefixed with `Default` (e.g. `DefaultCommandBus`)
- Complete (standalone) implementations of interfaces are suffixed with `Impl` (e.g. `EventBusImpl`)

## Modifiers
- Classes should always be `public`
- Modifiers should be in order:
    - `public` | `protected` | `private`
    - `static`
    - `abstract`
    - `synchronized`
    - `transient` | `volatile`
    - `final`
- `native` and `strictfp` are not used

Only team members can make exceptions to these rules.

# JavaDocs
## General
- All non `@tag` arguments start with a capital letter
- All `@link`, `@see` and similar tags use classname prefixes even if it is within the same type. E.g. `{@link X#doY}` 
instead of `{@link #doY}`
- General description ends with `.` but any `@tag` does not

## Structure
All JavaDocs follow the order:
> {method description}  
>   
> @param <T>  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{generic type description}  
> @param x  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{param description}  
>   
> @return {return description}    
> @throws {exception type}  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{exception description}  
>   
> @see {link}

- Code examples are wrapped in `<pre>{@code ... }</pre>`
- Additional paragraphs are wrapped in `<p> ... </p>`
- Lists are formatted using `<ul></ul>`
- Descriptive links are formatted using `{@link org.dockbox.hartshorn.ClassY y}` (shows as `y`)

Example:
```
/**
 * Performs X.
 *
 * <pre>{@code
 *    X x = new X();
 *    Object o = x.doX(y, z);
 * }</pre>
 *
 * <p>
 * Additional information about {@link Y y}.
 * </p>
 *
 * <ul>
 *    <li>A</li>
 *    <li>B</li>
 * </ul>
 *
 * @param <T>
 *        The type parameter for arg0
 * @param arg0
 *        The first argument
 * @param arg1
 *        The second argument
 *
 * @return The object X
 * @throws Exception
 *        When Y happens
 *
 * @see X#doY
 */
public <T> Object doX(T arg0, String arg1) throws Exception {
    // Do x
}
```