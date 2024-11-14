# NeonCode
A safe programming language

> This is the documentation repository that describes the syntax of the language.

## Syntax

```
package hello_world;

class main
{
  public static int main(array<string> args)
  {
    // "env" is a hidden variable (the only one) that references an object
    // it is automatically passed from the caller of any method
    env.log(new log(INFO, "Program started."));
    env.output("Hello, world!");
  }
}

```
