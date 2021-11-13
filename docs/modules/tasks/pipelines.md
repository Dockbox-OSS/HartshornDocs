# Pipeline framework
Pipelines provide an easy, type-safe and exception-safe means of breaking up large operations into smaller steps, known as pipes; with the pipes being executed in series. A collection of these pipes form a pipeline, where the result of the previous pipe becomes the input for the next pipe. 

The pipeline API consists of 2 built-in pipelines: `ConvertiblePipeline<P, I>` and `Pipeline<I>`, along with an `AbstractPipeline<P, I>` from which they both derive. There are also a number of built-in pipes, which allow you to customise what inputs you receive for a particular step. The pipelines are designed to be pipe independent so that you can create and use custom pipes with the built-in pipelines without having to make your own pipeline class.

## Pipes

Pipes provided the means to manipulate the data being processed by the pipeline. Pipes can be split into two different categories, depending on whether they extend `StandardPipe<I, O>` or `ComplexPipe<I, O>`. Standard pipes primarily only provide alternative ways of presenting the data within the pipeline to that specific pipe. By default, the standard pipe takes in an `Exceptional<I>`, however, some of the built-in standard pipes have override this so that it can be displayed in different ways. For example: 

```java
Pipe#execute(I input, Throwable throwable) -> O

InputPipe#execute(I input) -> O
```

Complex pipes, on the other hand, can take advantage of public methods within the pipeline class to enable the formation of complex pipeline behaviour. Currently, the only built-in complex pipe is `CancellablePipe<I, O>`. 

### Custom Pipes

You can very easily create custom pipes by either extending `StandardPipe<I, O>` or `ComplexPipe<I, O>` and implementing their respective `Apply` methods, which should be used to call an abstract method with the parameters you desire. The apply method that you override is responsible for converting the input (An `Exceptional<I>` for standard pipes) into the input parameters you wish. You can also override their default implementations of `getType` if you need to know the specific class of the pipe, even when creating pipes through lambda expressions. Additionally, it's important to add a static `of` method to each pipe you create, so that they can be used as functional interfaces to create pipes through lambda expressions as the pipeline cannot differentiate between the different pipes directly.

As an example, let's see how you would create a `ListenerPipe` that takes in an input but doesn't return anything, with the input being automatically returned by the pipe interface. The first thing to consider is which type of pipe should be extended? As this is at its core, once again another way of displaying the information of the pipeline to the user it should extend `StandardPipe<I, O>`. All that must then be done is override the `apply` method, create an abstract `execute` method which just takes in the input and create the static `of` method. The complete implementation for such a pipe is shown below.

```java
@FunctionalInterface
public interface ListenerPipe<I> extends StandardPipe<I, I> {

    void execute(I input) throws Exception;

    @Override
    default I apply(Exceptional<I> input) throws Exception {
        this.execute(input.orNull());
        return input.orNull();
    }

    // A static method that allows you to create instances of this
    // pipe through lambda expressions for the pipeline.
    static <I> ListenerPipe<I> of(ListenerPipe<I> pipe) {
        return pipe;
    }
}
```

Another important thing to note here is how all the method signatures have `throws Exception`. This means you can avoid having to catch any exceptions thrown in your pipe as this is handled by the pipeline. Any exception thrown by a pipe will be automatically caught and passed to the next pipe in the pipeline.

### Built-in Pipes

All the Pipeline API's built-in pipes are summarised in the table below, where `I` is the input type and `O` is the output type:

Class            | Input Types                               | Return Type
-----------------|-------------------------------------------|--------------------------
StandardPipe     |`Exceptional<I>`                           |`O`
InputPipe        |`I`                                        |`O`
Pipe             |`I`, `Throwable`                           |`O` 
ListenerPipe     |`I`                                        |`void`
ComplexPipe      |`AbstractPipeline<?, I>`, `I`, `Throwable` |`O`
CancellablePipe  |`Runnable`, `I`, `Throwable`               |`O`

## Pipeline Usage

The first built-in pipeline is simply `Pipeline<I>`. It cannot have its type changed, so each pipe in the pipeline takes in the same type it outputs.

### Pipeline Basics

#### Adding Pipes

There are several different ways that pipes can be added to the pipeline, either individually, as varargs, or as a collection, utilising either lambda expressions or method references. You can also add another pipeline, which just internally adds the pipes of the new pipeline as a collection. The different ways are demonstrated in the pipeline below:

```java
private int doubleNumber(int number) {
    return number * 2;
}
    
private int squareNumber(int number) {
    return number * number;
}

//...

int output = new Pipeline<Integer>()
    .addPipe(InputPipe.of(this::doubleNumber))
    .addPipe(InputPipe.of(input -> input - 2))
    .addPipes(Arrays.asList(
        InputPipe.of(input -> input / 2),
        InputPipe.of(input -> input + 1)))
    .addVarargPipes(
        InputPipe.of(this::squareNumber),
        InputPipe.of(input -> input - 5))
    .processUnsafe(4); // 11
```

#### Processing Data

Pipelines also provide several different ways of processing data for both collections and individual pieces of data which are summarised in the table below, where `I` is the input type and `O` is the output type. Which one you use depends on your particular situation.

Method          | Input         | Returns              | Note                    
----------------|---------------|----------------------|-------------------
process         |`I`            |`Exceptional<O>`      | The output wrapped in an exceptional.
processUnsafe   |`I`            |`O`                   | The output returned may be null.
processAll      |`Collection<I>`|`List<Exceptional<O>>`| Each output wrapped in an exceptional.
processAllSafe  |`Collection<I>`|`List<O>`             | Null outputs are excluded from the returned list.
processAllUnsafe|`Collection<I>`|`List<O>`             | All the outputs are returned and may be null.

#### Passing Forwards

Let's consider what happens if a pipe throws an exception. In this situation, the thrown exception is automatically captured and passed to the next pipe, however, this also means that the pipe is unable to return an output. In this situation, the pipeline _passes forward_ the previous input. This is particularly significant in the case of something like an event bus where you don't want the pipeline to be reliant on the implementation of a particular listener. By _passing forward_ the previous input, the event can continue to be passed on to subsequent pipes, enabling the continuation of the event bus. 

#### Removing Pipes

There are two ways that you can remove pipes from a pipeline, using either `removeLastPipe` or `removePipeAt`, where pipes are stored in the order they are added to the pipeline. Both these methods check for the index being out of bounds, in which case nothing is removed.

```java
AbstractPipeline<Integer, Integer> pipeline = new Pipeline<Integer>()
    .addPipe(InputPipe.of(input -> input * 2))
    .addPipe(InputPipe.of(input -> input + 3))
    .addPipe(InputPipe.of(input -> input - 1));

int output = pipeline.processUnsafe(8); // 18

pipeline.removeLastPipe();
output = pipeline.processUnsafe(8); // 19
```

#### Cancel Behaviours

Pipelines can also be cancelled by utilising the built-in `CancellablePipe<I, O>`. By default, pipelines aren't cancellable and so if you try and add a cancellable pipe and then process a piece of data, this will cause an `IllegalPipeException`. This can be changed by calling `setCancelBehaviour`, which takes in a `CancelBehaviour`. The various `CancelBehaviours` determine how the pipeline should respond if it is cancelled and are summarised in the table below. It's important to note that not every `CancelBehaviour` can be used for both pipelines, trying to do so will cause an `UnsupportedOperationException`.

Cancel Behaviour | Supported Pipelines | Description
-----------------|---------------------|------------------------------
Uncancellable    | Both                | The default `CancelBehaviour` of pipelines which prevents it from being cancelled.
Discard          | Both                | The output of the pipeline up until it's cancelled is discarded and null is returned.
Return           | Both                | The output of the pipeline up until it's cancelled is returned as-is. In ConvertiblePipeline`s this can cause a `ClassCastException` if not correctly handled.             
Convert          |`ConvertiblePipeline`| The output of the pipeline up until it's cancelled is returned by first converting to the output type.


**Note:** _In `ConvertiblePipeline`s, setting a `CancelBehaviour` applies to all the pipelines and so it is not possible to have multiple `CancelBehaviour`s. Instead, the one set last is what will be used._

```java
AbstractPipeline<Integer, Integer> pipeline = new Pipeline<Integer>()
    .setCancelBehaviour(CancelBehaviour.RETURN)
    .addPipe(InputPipe.of(input -> input + 1))
    .addPipe(CancellablePipe.of((cancelPipeline, input, throwable) -> {
        if (2 < input) cancelPipeline.run();
        return input;
    }))
    .addPipe(InputPipe.of(input -> input * 2));

// Pipeline isn't cancelled
int output = pipeline.processUnsafe(0);
Assert.assertEquals(2, output);

// Pipeline is cancelled
output = pipeline.processUnsafe(4);
Assert.assertEquals(5, output);
```

### Pipeline Example

One example of where a pipeline may be useful is for an event bus. By utilising a pipeline for the event bus, this prevents you from having to deal with the distribution of events to each listener, catching exceptions thrown by listeners and ensuring that an error in one listener doesn't impact the other listeners in the event bus. Without getting too much into the finer details, let's see how an event bus may be implemented using a `Pipeline<I>`. 

Consider the following event listeners.

```java
public class EventListeners {

    @Listener
    public void helloEventListener(ChatEvent event) {
        if (event.getMessage().contains("Hello")) {
            // Say hi back.
            System.out.println("Hey there!");
        }
    }

    @Listener
    public void pipelinesAreCoolListener(ChatEvent event) {
        event.setMessage(
            event.getMessage()
                // Spice up the message.
                .replace("pipelines are cool", "~~~ Pipelines Are Cool ~~~")
        );
    }
}
```

This class has 2 event listeners; one which responds to "_Hello_" and the other that modifies the event's message if it contains "_pipelines are cool_". These can be very easily registered to the event bus by utilising the `ListenerPipe` we defined previously.

```java
private Pipeline<Event> eventBus = new Pipeline<>();

public void registerListeners(Object listener) {
    // Returns a collection of all the methods in the listener object annotated with @Listener.
    Collection<MethodContext<?, ?>> methods = TypeContext.of(listener).methods(Listener.class);

    for (MethodContext<?, ?>method : methods) {
        this.eventBus.addPipe(ListenerPipe.of(input -> method.invoke(listener, input)));
    }
}

//...

this.registerListeners(new EventListeners());
```

From there, we can either process events individually or as a collection.

```java
Event helloMessage = new ChatEvent("Hello Pumbas!");
Event pipelinesAreCoolMessage = new ChatEvent("Don't you think pipelines are cool?");

Exceptional<Event> output = this.eventBus.process(helloMessage);
List<Exceptional<Event>> outputs = this.eventBus.processAll(Arrays.asList(helloMessage, pipelinesAreCoolMessage)); 
```

## Convertible Pipeline Usage

The other built-in pipeline is the `ConvertiblePipeline<P, I>`, where `P` is the input type of the very first pipeline and `I` is the input type of the current pipeline. The convertible pipeline has all the functionality of `Pipeline<I>`, however, this pipeline can also be converted between types, while still remaining type-safe.

### Convertible Pipeline Source

A `ConvertiblePipeline<P, I>` cannot be directly instantiated. This is instead done through a `ConvertiblePipelineSource<I>` which extends `ConvertiblePipeline<I, I>` as the first `ConvertiblePipeline<P, I>` will always have the same types for `P` and `I`. When instantiating a `ConvertiblePipelineSource<I>`, you will need to pass in a `Class<I>`, which will be used to ensure type-safety.

### Converting

The `ConvertiblePipeline<P, I>` can be converted by calling `convertPipeline` which takes in a non-null `Function<? super I, K>`, as the converter and a non-null `Class<K>` where `K` is the input type of the new pipeline. This creates a new `ConvertiblePipeline<P, K>` which internally stores references of the next pipeline and previous pipeline in a linked-list like manner.

```java
float output = new ConvertiblePipelineSource<>(Integer.class)
    .addPipe(InputPipe.of(input -> input * 2))
    .convertPipeline(integer -> (float)integer, Float.class)
    .addPipe(InputPipe.of(input -> input / 5F))
    .addPipe(InputPipe.of(input -> input * 2))
    .processUnsafe(18); // 14.4F
```