## Fantomas: How to use

### Using the command line tool
---

For the overview how to use the tool, you can type the command

	Fantomas --help

You have to specify an input path and optionally an output path. 
The output path is prompted by `--out` e.g.

	Fantomas ../../../../tests/stackexchange/array.fs --out ../../../../tests/stackexchange_output/array.fs 

Both paths have to be files or folders at the same time. 
If they are folders, the structure of input folder will be reflected in the output one. 
The tool will explore the input folder recursively if you set `--recurse` option (see [Options section](#options)).
If you omit the output path, Fantomas will overwrite the input files.

#### Options

##### `--recurse`

traverse the input folder recursively (if it is really a folder) to get all F# source files.

##### `--force`

force writing original contents to output files. 
This is helpful if the tool fails on some unknown F# constructs.

##### `--stdin`

read input from standard input. This option is convenient to use with piping

    type input.fs | Fantomas --stdin --out output.fs

##### `--stdout`

write formatted source code to standard output e.g.
 
    Fantomas input.fs --stdout

##### `--fsi`

this option to be used with `--stdin` to specify that we are formatting F# signatures e.g.

    type input.fsi | Fantomas --fsi --stdin --stdout

#### Preferences

##### `--indent <number>`

`number` has to be between 1 and 10.

This preference sets the indentation (default = 4). 
The common values are 2 and 4. 
The same indentation is ensured to be consistent in a source file. 
To illustrate, here is a code fragment with `--indent 2`:

	```fsharp
	let inline selectRandom(f : _[]) = 
	  let r = random 1.0
	  let rec find = 
	    function 
	    | 0 -> fst f.[0]
	    | n when r < snd f.[n] -> fst f.[n]
	    | n -> find(n - 1)
	  find <| f.Length - 1
	```

##### `--pageWidth <number>`

`number` has to be an integer greater or equal to 60.
This preference sets the column where we break F# constructs into new lines.
The default value is 120. To see its effects, please take a look at some [output files](tests/stackexchange_output) with `--pageWidth 90` preference.

##### `--semicolonEOL`

add semicolons at the end of lines e.g.

	```fsharp
	let saturn = 
	  { X = 8.343366718;
	    Y = 4.124798564;
	    Z = -0.4035234171;
	    VX = -0.002767425107 * daysPerYear;
	    VY = 0.004998528012 * daysPerYear;
	    VZ = 2.304172976e-05 * daysPerYear;
	    Mass = 0.0002858859807 * solarMass }
	```
	
	vs.
	
	```fsharp
	let saturn = 
	  { X = 8.343366718
	    Y = 4.124798564
	    Z = -0.4035234171
	    VX = -0.002767425107 * daysPerYear
	    VY = 0.004998528012 * daysPerYear
	    VZ = 2.304172976e-05 * daysPerYear
	    Mass = 0.0002858859807 * solarMass }
	```

 ##### `--noSpaceBeforeArgument`

if being set, no space is inserted before a function name and its first argument. 
For example, `Seq.filter (fun x -> x > 2)` becomes `Seq.filter(fun x -> x > 2)`. 
This doesn't affect methods and constructors, e.g. `Console.WriteLine("Hello World")`.

##### `--spaceBeforeColon`

if being set, there is a space before `:` e.g.

	```fsharp
	type Planet = 
	  { mutable X: float
	    mutable Y: float
	    mutable Z: float
	    mutable VX: float
	    mutable VY: float
	    mutable VZ: float
	    Mass: float }
	```
	
	vs.
	
	```fsharp
	type Planet = 
	  { mutable X : float
	    mutable Y : float
	    mutable Z : float
	    mutable VX : float
	    mutable VY : float
	    mutable VZ : float
	    Mass : float }
	```

##### `--noSpaceAfterComma`

is useful if you would like to save spaces in tuples, arguments, etc. 
To illustrate, `(1, 2, 3)` is rewritten to `(1,2,3)`.

##### `--noSpaceAfterSemiColon`

saves spaces on records, arrays, lists, etc. Now 

	```fsharp
	let planets = [|sun; jupiter; saturn; uranus; neptune|]
	```

	becomes

	```fsharp
	let planets = [|sun;jupiter;saturn;uranus;neptune|]
	```

 ##### `--indentOnTryWith`

if being set, `with` blocks will be indented like in the following example:

	```fsharp
	try
	    if System.DateTime.Now.Second % 3 = 0 
		then raise(new System.Exception())
	    else raise(new System.ApplicationException())
	with
	    | :? System.ApplicationException -> 
	        printfn "A second that was not a multiple of 3"    
	    | _ -> 
	        printfn "A second that was a multiple of 3"
	```

##### `--noSpaceAroundDelimiter`

saves spaces around delimiters of records, arrays, lists e.g.

    ```fsharp
	let planets = [| sun; jupiter; saturn; uranus; neptune |]
	```

	becomes

	```fsharp
	let planets = [|sun; jupiter; saturn; uranus; neptune|]
	```

 ##### `--reorderOpenDeclaration`

if being set, all open statements in a block will be sorted in the lexicographical order.
This can lead to compilation problems, see https://github.com/fsprojects/fantomas/issues/645.

For that reason, this setting is considered **deprecated** and will be **removed in the next major version**.

##### `--strictMode`

if being set, pretty printing is only done via ASTs. Compiler directives, inline comments and block comments will be ignored. 

##### `--keepNewlineAfter`

if being set, newlines found in the source text will be kept in certain conditions.
 
 ```fsharp
let a =
    42
```

will remain the same, the newline after the `=` was detected and preserved.

```fsharp
let config =
    Builder()
      .A()
      .B()
      .C()
```

will remain the same, the newline before the `.` was detected and preserved.

```fsharp
match meh with
| Foo ->
  printfn "foo"
| Bar ->
  printfn "bar"
```

will remain the same, the newline after `->` was detected and preserved.


##### `--maxIfThenElseShortWidth  <number>`

`number` if being set, controls when if/then/else expressions will be formatted as single line or as multiple lines.

Fantomas tries to follow [the F# style guide](https://docs.microsoft.com/en-us/dotnet/fsharp/style-guide/formatting#formatting-if-expressions) as close as possible when formatting if expressions.

The style guide says:

> If either cond, e1 or e2 are longer, but not multi-line:

```fsharp
if cond
then e1
else e2
```

But what exactly is longer right? By default Fantomas will use 40 width to determine if the expression needs to be formatted to the example above or remain as a oneliner (`if cond then e1 else e2`).


So if either `cond`, `e1` or `e2` is longer than `maxIfThenElseShortWidth` but not multiline, it will format on three lines.
See [unit tests](https://github.com/fsprojects/fantomas/blob/9d4b499c09a1f06f5485835817844657cc51215b/src/Fantomas.Tests/IfThenElseTests.fs#L734) for more examples.

That said, most of the preferences are very simple. 
But they demonstrate the flexibility of Fantomas on a set of configurations. 
More preferences will be added depending on use cases.

##### `--config <Path to file or folder>`

Use a JSON configuration file based on a [schema](../src/Fantomas/schema.json) to set the formatting options.

A default configuration file would look like
```json
{
  "IndentSpaceNum":4,
  "PageWidth":120,
  "SemicolonAtEndOfLine":false,
  "SpaceBeforeArgument":true ,
  "SpaceBeforeColon":false,
  "SpaceAfterComma":true ,
  "SpaceAfterSemicolon":true ,
  "IndentOnTryWith":false,
  "ReorderOpenDeclaration":false,
  "SpaceAroundDelimiter":true ,
  "KeepNewlineAfter":false,
  "MaxIfThenElseShortWidth":40,
  "StrictMode":false
}
```

However, **a configuration file overwrites options** from [the default configuration](../src/Fantomas/FormatConfig.fs).

The argument passed after `--config ` can be a file named `fantomas-config.json` or a folder.
In both cases a Fantomas will try and locate `fantomas-config.json` in all the parent folders.
The found configuration files are then being applied to to the default configuration from top to bottom.

F.ex.

```
C:\
    Temp\
        fantomas-config.json
        MyProject\
            fantomas-config.json
```

Formatting with `dotnet fantomas MyFile.fs --config "C:\Temp\MyProject"` will first apply the settings in `C:\Temp\fantomas-config.json` and then those of `C:\Temp\MyProject\fantomas-config.json`.

If the `--config` is used in combination with other settings, the configuration is applied first and then the other arguments.

Warnings will be given if settings in the configuration no longer apply for the current version of Fantomas.

### Using the API

See [CodeFormatter.fsi](../src/Fantomas/CodeFormatter.fsi) to view the API of Fantomas.