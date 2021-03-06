---
uid: arguments
---

# Arguments

When a command executes, the raw `string[] args` value can be separated into two different categories: options and arguments.

Arguments are positional and values are specified based by order.

Options are named and must be specified using a name. Options are covered in [this document.](xref:options)

Arguments are represented by the @McMaster.Extensions.CommandLineUtils.CommandArgument type.
They have one defining characteristic.

* Position - the order in which arguments appear on command line (after options have been parsed)

### [Using Attributes](#tab/using-attributes)

@McMaster.Extensions.CommandLineUtils.ARgumentAttribute can be used on properties to define arguments.
The argument order must be specified explicitly.

```c#
public class Program
{
    [Argument(0)]
    [Required]
    public string FirstName { get; }

    [Argument(1)]
    public string LastName { get; }  // this one is optional because it doesn't have `[Required]`

    private void OnExecute()
    {
        Console.WriteLine($"Hello {FirstName} {LastName}");
    }

    public static int Main(string[] args) => CommandLineApplication.Execute<Program>(args);
}
```

### [Using Builder API](#tab/using-builder-api)

```c#
public class Program
{
    public static int Main(string[] args)
    {
        var app = new CommandLineApplication();

        var firstNameArg = app.Argument("first name", "the first name of the person")
            .IsRequired();
        var lastNameArg = app.Argument("first name", "the first name of the person");

        app.OnExecute(() =>
        {
            Console.WriteLine($"Hello {firstNameArg.Value} {lastNameArg.Value}");
        });

        return app.Execute(args);
    }
}
```

***

## Remaining arguments

A common scenario is to allow a command line tool to take in a variable number of arguments.
These arguments are collected into a `string[]` array after all other arguments and options are parsed.

```
cat -b 123 file1.txt file2.txt file3.txt
```

> [!NOTE]
> Normally, unrecognized arguments is an error. To allow this scenario, you must set ThrowOnUnexpectedArgument to false.

See @McMaster.Extensions.CommandLineUtils.Conventions.RemainingArgsPropertyConvention for more details.

### [Using Attributes](#tab/using-attributes)

By default, attribute binding will set a `string[]` or `IList<string>` property named `RemainingArguments` or `RemainingArgs`
to include all values

```c#
[Command(ThrowOnUnexpectedArgument = false)]
public class Program
{
    [Option("-b")]
    public int BlankLines { get; }

    public string[] RemainingArguments { get; } // = { "file1.txt", "file2.txt", "file3.txt" }

    private void OnExecute()
    {
        if (RemainingArguments != null)
        {
            foreach (var file in RemainingArguments)
            {
                // do something
            }
        }
    }

    public static int Main(string[] args) => CommandLineApplication.Execute<Program>(args);
}
```

### [Using Builder API](#tab/using-builder-api)

```c#
public class Program
{
    public static int Main(string[] args)
    {
        var app = new CommandLineApplication(throwOnUnexpectedArg: false);
        var blankLines = app.Option<int>("-b <LINES>", "Blank lines", CommandOptionType.SingleValue);
        app.OnExecute(() =>
        {
            if (app.RemainingArguments != null)
            {
                foreach (var file in app.RemainingArguments)
                {
                    // do something
                }
            }
        });
        return app.Execute(args);
    }
}
```
