# Testing
## General
- Tests cover relevant use-cases
- The target coverage for all tests is 60%
- Tests are performed using Java 16

## Unit Testing
- Tests are located in `src/test/java`
- Test packages are equal to the package of the target class
- Test classes follow the naming convention `${TestedClass}Tests`
- Tests follow the [AAA pattern](https://medium.com/@pjbgf/title-testing-code-ocd-and-the-aaa-pattern-df453975ab80)
- Tests use JUnit 5 (`org.junit.jupiter.api`)

For example, `org.dockbox.hartshorn.common.ClassX` is tested in `org.dockbox.hartshorn.common.ClassXTests`