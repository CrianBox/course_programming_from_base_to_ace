---
description: Exceptions are used to indicate exceptional situations that may fail the application.
title: 30 - Exceptions
---

# Chapter 30 - Exceptions

One of the main goals to keep in mind when developing software - being it an application for users or a library for developers - is to make it user-friendly. When software is not user-friendly, the clients of our software will not use it.

To make software user-friendly the developer needs to try to handle exceptional cases as good as possible. Of course not all situations can be handled, but those that can be foreseen should be handled with care. For the rest a safety net should be provided so the user at least does not suffer data loss.

## Examples

Let's take a couple of examples of problems that may arise with systems and how they should or could be handled.

### Requesting Input

Imagine an application that requests the user to enter a numeric size for a font-size. What happens when the user does not enter a number or mistypes? When the developer of the application did not foresee this situation, the application would crash. This is one of the worst outcomes for the end-user. What if he/she has been inputting data for more than 20 minutes and all that is lost. Would you use that application daily?

This situation needs to be handled better. The application should catch this mistake and request a valid value from the user.

![User Input](./img/number-format-exception.png)

### Cloud Settings

Games these days often store their configurations in the cloud. Think about your video-card settings, key-bindings and all such configuration options.

Now take for example that the internet connection drops while this configuration is being synced to the cloud. What if the game does not handle this situation correctly and does not keep a backup locally. Then the user may have lost all his/her personalized settings.

There is also another side to this story. If the cloud-side does not handle this situation the user-configuration may even become corrupt. A good solution here would be to validate the configuration cloud-side before storing it in for example the database.

![Syncing problem](./img/syncing-settings.png)

### Printing

Consider printing a document of 50 pages. However your printer only has 49 sheets of paper. The developer of the printer-driver did not give this situation to much attention and the driver just restarts the printing process once paper is provided. As a user you did not get any option to continue printing where you left of. You are stuck with 99 sheets of paper on the table.

![Printing Problem](./img/printing.png)

### Video Streaming

Imagine yourself watching your favorite movie on Netflix on a smartphone on the train. There is no WiFi in the train so you watch it on 4G. The moment you enter the train station your smartphone detects the free WiFi network and switches over to it. The Netflix app loses it's internet connection for a moment and fails to restore the video stream. It crashes. You are annoyed but restart the app. It could not save the video-timestamp. To continue watching the movie where you left of, you need to skip through half the movie.

By the time you found the scene you we're watching the train leaves the current station and the internet connection switches back to 4G.

![Watching NetFlix](./img/netflix.jpg)

## C-style Languages

Not all programming languages have dedicated mechanisms for handling error cases. Take for example the C-language. Typically in these languages, functions will return error codes to indicate if the function executed successfully or an error has occurred. This often leads to extreme ugly code. It can also lead to very unuser-friendly application crashes.

![Application Crashes with Error Code](./img/app-crash-code.png)

Take some pseudo-code as an example

```text
read_file(filename, lines):
  file_does_not_exist?  => return -1;
  could_not_read_file?  => return -2;
  file_empty?           => return -3;
  not_ascii_text?       => return -4;

  all_ok?
    | => read into lines
    | => return number_of_lines

  return -255;  // Unknown error occurred
```

C applications are notoriously known for their long listings of global error codes. Take an example from a Linux kernel.

```cpp
#define EPERM            1      /* Operation not permitted */
#define ENOENT           2      /* No such file or directory */
#define ESRCH            3      /* No such process */
#define EINTR            4      /* Interrupted system call */
#define EIO              5      /* I/O error */
#define ENXIO            6      /* No such device or address */
#define E2BIG            7      /* Arg list too long */
#define ENOEXEC          8      /* Exec format error */
#define EBADF            9      /* Bad file number */
#define ECHILD          10      /* No child processes */
#define EAGAIN          11      /* Try again */
...
#define EMEDIUMTYPE     124     /* Wrong medium type */
```

So basically the programmer needs to invent a numeric value for every situation that could go wrong. The error value is then returned by the function to the function that called this one. That code might look something like this:

```text

result = read_file(user_file, 100)

if result > 0: print file to terminal
if result == -1: request file name again from user
else if result == -2: output "Please check if permissions are correct", retry
else if result == -3: output "File is empty"
else if result == -4: output "File did not contain text"
```

What a mess indeed. And this is no fiction, often one will find these constructs in applications written in the C-language. The error handling code often becomes very complicated because each error needs to be handled differently.

On top of that, often multiple functions are called to handle more complex tasks, take the following pseudo-code example where a configuration file is downloaded, read and parsed:

```text
call download_file()
if (errors) {
  handle them
} else {
  call read_file()
  if (errors) {
    handle them
  } else {
    call parse_file()
    if (errors) {
      handle them
    } else {
      ...
    }
  }
}
```

So every time a function is called that is needed in a more complex function, the possible errors need to be handled.

Another disadvantage of this approach is the fact that it is really hard to pass errors from deeper functions up the call stack to the higher functions. Why is this required? Because an error often occurs in deeper nested functions in a place where it cannot be handled correctly, so it needs to bubble up the call chain to the higher functions where the error can be handled accordingly.

So while the C-programming language has no constructs to handle errors better, the approach of using return values as error code, there are some **serious drawbacks to this approach**:

* Error codes are not always clear, often cryptic.
* Distinction between error code and valid value is not always clear.
* Functions can only return a single value.
* Hard/Messy to pass error codes upward if it cannot be handled.
* Often leads to nasty error handling code.
* This approach does not integrate well with constructors (do not return values).
* Error handling code is mixed with normal control flow of code.

## Exceptions

Because of all these drawbacks, higher programming languages provide a special mechanism especially for handling error cases. In modern programming languages like C#, these "problems" are typically modeled using **exceptions**.

::: tip Exceptions
Jeff Atwood (of StackOverflow fame) once called exceptions "the bread and butter of modern programming languages." That should give you an idea of how important this construct is.
:::

Exceptions (exceptional events) are **problems that occur during the execution of an application**, which **disrupt the normal flow** of the application's code. If the exception is not handled by the application code itself, the process terminates abnormally.

Exceptions are a type of problem that occurs during the execution of an application. Errors are typically problems that are not expected. Whereas, exceptions are expected to happen within the application’s code for various reasons. Applications use exception handling logic to explicitly handle the exceptions when they happen.

An exception can occur on many different occasions. Some common reasons are:

* A user has entered invalid data.
* A file that needs to be opened cannot be found.
* A network connection has been lost in the middle of communications.
* A number is being divided by zero.

Most exceptions are caused by the user providing wrong input or by physical resources that have failed in some manner.

## Why is this Topic Important

Unfortunately, the management of errors and other problems in the code is one of the most neglected topics in the education of new software developers. In most tutorials or lessons that feature the creation of a demo app, when it comes the time to deal with error management it's always the same story: "In a real application, you'd handle the exceptions and probably log them, but we're skipping that, for brevity's sake."

And this is a shame since we could argue that software development is all about handling those kinds of situations. For "brevity's sake", many programming students become programming professionals who have no clue how exception handling works in the real world.

## The Exception Process

Exceptions provide **a way to transfer control from one part of a program to another**. C# exception handling is built upon four keywords: `try`, `catch`, `finally`, and `throw`.

Let us a closer look at a fictional example of a game that is started and needs to load a configuration file from the local filesystem. Methods/functions that are nested in other functions are indented towards the right.

![Load Game Config](./img/read-game-config.png)

What would happen if the `OpenFile()` method fails because the configuration file could not be found on the local filesystem?

1. The method `LoadGameConfiguration()` **tries** to load the game config file by called the method `LoadGameConfigFile().
2. However the file could not be opened and the method `OpenFile()` indicates this by **throwing** an exception object of type `FileNotFoundException` (real C# exception class). If something goes wrong in the code, it can warn the higher methods by *throwing* a special object of an exception class.
3. The rest of the code stops from executing and the exception is handed to the method that called the `OpenFile()` method, in other words the exception object is handed to `LoadGameConfigFile()`.
4. The `LoadGameConfigFile()` actually has no solution for the problem here. It should ignore the exception object. In that case the object **bubbles upward** to the calling method `LoadGameConfiguration()`.
5. The method `LoadGameConfiguration()` **catches** the exception object of type `FileNotFoundException` because it can handle the problem. The system has a backup config stored in a special directory for these types of problems. The `LoadGameConfiguration()` methods calls the `LoadGameConfigFile()` again but this time with a different path for the file.
6. All goes well and the rest of the application executes.

![Open File Fails](./img/open-file-fails.png)