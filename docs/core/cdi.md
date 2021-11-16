# Context and dependency injection (CDI)
To understand how CDI is applied in Hartshorn, it is important to understand how the framework views the application lifecycle. Hartshorn is extendable and flexible, allowing it to be used for diverse applications. Applications managed by Hartshorn are separated into two types: components and services. Components are objects which define context, but lack functionality. Services on the other hand are objects which accept context, and offer functionality.

## Components
Components range from POJO's to `Context` types. The common property of all components is that they lack functionality, and only provide context. In domain objects the provided context is the entity definition, in persistent entities the context is the data represented by the entity, and in the case of the [`Application context`](./application-context.md) it is information about all application components.  

Components do not require explicit identification, but can be annotated with [`@Component`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/annotations/component/Component.java). When annotated with `@Component`, the component will be registered to the `ApplicationContext`. By default, `@Component` marks the component as an injectable element through the selected [`ComponentType.INJECTABLE`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/ComponentType.java).

## Services
Like regular components, services will always allow injection. Services however do not define context, and instead act based on provided context. They can provide a wide range of functionality based on specific contexts. Services should always be identified with `@Service`, or [an extension of it](./annotations.md).

Extensions of `@Service` include [`@RestController`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-web/src/main/java/org/dockbox/hartshorn/web/annotations/RestController.java), [`@Configuration`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-config/src/main/java/org/dockbox/hartshorn/config/annotations/Configuration.java), and [`@CacheService`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-cache/src/main/java/org/dockbox/hartshorn/cache/annotations/CacheService.java). Each extension of `@Service` specifies a specific type of service. For example, `@Configuration` allows defining a data source for the configuration, by adding a `#source(): String` attribute. 

Services define a default `ComponentType`, namely [`ComponentType.FUNCTIONAL`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/ComponentType.java). Functional components can be processed by [`ComponentProcessor`s](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/services/ComponentProcessor.java) and be modified by [`ComponentModifier`s](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/services/ComponentModifier.java). In the case of services, the [`ServiceProcessor`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/services/ServiceProcessor.java) and [`ServiceModifier`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/services/ServiceModifier.java) directly target service definitions.

As services are able to provide functionality based on a given context, all services are singletons by default. This allows you to better optimize your application, and to ensure a state can safely be stored inside services.

# Inversion of control
This chapter covers Hartshorn's implementation of the Inversion of Control (IoC) principle.

## Dependency injection
Dependency injection is a commonly used design pattern which implements the IoC principle. Hartshorn defines two core packages responsible for dependency injection:

- [`org.dockbox.hartshorn.core.binding`](https://github.com/GuusLieben/Hartshorn/tree/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/binding): The definition for base [providers](./providers.md) and [hierarchies](./hierarchies.md).
- [`org.dockbox.hartshorn.core.context`](https://github.com/GuusLieben/Hartshorn/tree/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/context): The core contexts, including default [`Context`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/context/Context.java) definitions and the [`ApplicationContext`](https://github.com/GuusLieben/Hartshorn/blob/develop/hartshorn-core/src/main/java/org/dockbox/hartshorn/core/context/ApplicationContext.java).


Applications rarely consist of a single object, even simple application often consist of several components all working together. The dependency injection pattern defines that components should not be responsible for the components and services it requires to function. Not only does this improve testability by allowing you to mock services in test environments, it also drastically enhances the maintainability of your application.

To better understand this principle, we'll consider the following example. Imagine a simple application which allows users to register and login. In this application users can register and login by entering their username and preferred password in a command line environment. Let's see what that would look like in code.

=== "UserService.java"
    ```java
    public class UserService {

        private CommandLineInputProvider input = new CommandLineInputProvider();

        public boolean login() {
            String username = input.listenForUsername();
            String password = input.listenForPassword();
            return "admin".equals(username) && "admin".equals(password);
        }
    }
    ```
=== "CommandLineInputProvider.java"
    ```java
    public class CommandLineInputProvider {

        private final Scanner scanner = new Scanner(System.in);

        public String listenForUsername() {
            System.out.print("Username: ");
            return scanner.nextLine();
        }

        public String listenForPassword() {
            System.out.print("Password: ");
            return scanner.nextLine();
        }
    }
    ```

However, what if we no longer want to use a command line environment, and instead wish to use a graphical interface? In this case we'd need to replace all usages of the `CommandLineInputProvider` with a new `GUIInputProvider`. With dependency injection, instead of replacing all usages, you use a common interface `InputProvider`. Hartshorn enables you to add an implementation for the `InputProvider` dynamically, so your `UserService` never needs to know anything besides the `InputProvider` interface.

=== "UserService.java"
    ```java
    @Service
    public class UserService {

        @Inject
        private InputProvider input;

        public boolean login() {
            String username = input.listenForUsername();
            String password = input.listenForPassword();
            return "admin".equals(username) && "admin".equals(password);
        }
    }
    ```
=== "CommandLineInputProvider.java"
    ```java
    public interface InputProvider {
        String listenForUsername();
        String listenForPassword();
    }
    ```

To learn how to make `CommandLineInputProvider` the given implementation of `InputProvider`, see [Binding and providing](./providers.md). Services are 

## Context injection
TBD 

### Automatically created context
TBD