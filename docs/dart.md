---
title: Dart
date: 2020-10-08 23:31:00
slug: dart
---

#dart #programming

[TOC]



## Installation

- Windows
  - Installed using choco
    - cmd (Admin)
    - `choco install dart-sdk`
    - `choco upgrade dart-sdk`
  - sdk located in `c:\dart-sdk`

## IDE

Use IntelliJ IDEA with the Dart plugin

- IDE support is limited
  - Simple refactors work
  - Imports aren't always suggested. Manual import is needed for extension methods.



## Testing

`pubspec.yaml`

```yaml
dev_dependencies:
  test: 1.15.1
```

Use the following structure

`*_test.dart`

```dart
import 'package:test/test.dart';

void main() {
  group('list', () {
    test('empty list length is 0', () {
      expect([].length, 0);
    });
  });
}
```

:information_source: Variables defined in `main()` and `group()` are shared (they are normal functions). Take care not to unintentionally mutate them.

```dart
import 'package:test/test.dart';

void main() {
  var scenario = [1,2,3];

  group('length', () {
    
    test('initial length is 3', () {
      var l = scenario.clone();
      expect(l.length, 3);
    });
      
    test('adding 1 makes length 4', () {
      var l = scenario.clone();
      l.add(4)
      expect(l.length, 4);
    });
  });
}
```



## Language

### Unique features

```dart
int n;

n ?? = 10; // assigns 10 if null
print(n ??= 10) // assigns if null, prints value of n
```



### How to create extensions on Lists?

```dart
import 'package:test/test.dart';

extension Coordinates<V> on List<V> {
  V get x => this[0];
  V get y => this[1];
  V get z => this[2];
}

void main() {
  test('access by property', () {
    var position = [5, 4, -2];
    expect(position.x, 5);
    expect(position.y, 4);
    expect(position.z, -2);
  });
}
```



### How to initialize final fields in the constructor?

Use the constructor initialization block 

```dart
class MyClass {
    final List<int> list;
    final int length;
    
	MyClass(this.list) : length = list.length {}   
}
```



### How to iterate over a list with index?

```dart
myList.asMap().forEach((index, element) {
   print(index);
   print(element);
});
```



Use this snippet:

```dart
// Adapted from: https://stackoverflow.com/questions/54898767/enumerate-or-map-through-a-list-with-index-and-value-in-dart/54899730
extension IterableMapExtensions<T, TO> on Iterable<T> {
  Iterable<TO> mapIndexed<TO, T>( TO Function(T item, int index) projection) sync* {
    var index = 0;
    for (final item in this) {
      yield projection(item as T, index);
      index++;
    }
  }
}
```

Then

```dart
var list = [1, 2, 3]
list.mapIndexed((value, index) => {
	return value;
})
```



### How to cast boolean to integer?

You can't :cry:.

```dart
v ? 1 : 0;
```



### Constructors, named constructors, factory methods

```dart
class Post {
  String text;
  
  // constructor
  Post(this.text);
    
  // named constructors
  Post.empty(): text = '';
  Post.emphasized(String text): this('${text}!');
    
  // factory
  factory Post.fromText(String text) => Post(text);

  // static method
  static Post create(String text) => Post(text);
}
```



### How to tear-off a factory method or constructor?

> Keywords: method reference, tear off

You can't (dart 2.8) (see https://github.com/dart-lang/language/issues/216)

But you can create a _static method_ which can be torn-off.

```dart
class Post {
  String text;
  
  // constructor: cannot be torn off
  Post(this.text);
    
  // factory: cannot be torn off
  factory Post.fromText(String text) => Post(text);

  // static method
  static Post create(String text) => Post(text);
}

var postTexts = <String>[];
postTexts.map(Post.create);
```

### How do I destructure lists?

This is not possible :disappointed:



https://github.com/dart-lang/language/issues/546



### How do i add values or methods to Enums?

This is not possible directly, but you can use constants or extensions to achieve this.

**static consts**

```dart
class Number {
  static const one = Number._('one', 1);
  static const two = Number._('two', 2);
  static const three = Number._('three', 3);

  final int value;

  final String _string;

  const Number._(this._string, this.value);

  static const _values = {1: one, 2: two, 3: three};

  factory Number.fromInt(int value) => _values[value] ?? (throw ArgumentError('Invalid Number: $value'));

  factory Number.fromString(String value) => Number.fromInt(int.parse(value));

  @override
  String toString() => '$runtimeType.$_string';
}
```

- Use the intellij live template `classEnum` to generate this class. 
- Use the intellij live template `enumValue` to generate another value.

key points

- factories are supported
- private const constructor, so that we can use object equality to compare instances
- internal string representation is needed to print the enum values

**extensions**

```dart
enum Numbers {
  one,
  two,
  three,
}

// Numbers.one.value == 1
```

example with list

```dart
extension NumbersExtensionList on Numbers {
  static const values = [1, 2, 3];
  int get value => values[this.index];
}
```

example with map

```dart
extension NumbersExtensionMap on Numbers {
    static const valueMap = const {
        Numbers.one: 1,
        Numbers.two: 2,
        Numbers.three: 2,
    };
    int get value => valueMap[this];
}
```

### How do I emulate an Enum with a class?

https://gist.github.com/boukeversteegh/f67e58f500f3c0c7126640bb43e11197

```dart
class MyEnum {
  final String string;
  const PaymentMethod._(this.string);

  static const online = MyEnum._('online');
  static const transfer = MyEnum._('transfer');
  static const cash = MyEnum._('cash');

  static const values = [online, transfer, cash];
  static const _strings = {
    'online': online,
    'transfer': transfer,
    'cash': cash
  };

  static PaymentMethod parse(String value) =>
      _strings[value] ??
      (throw ArgumentError.value(value, 'value', 'Not a valid PaymentMethod'));

  @override
  String toString() {
    return 'PaymentMethod.$string';
  }
}
```



### How to use Regex?

```dart
static final pattern = RegExp(r'(?<name>.*?) <(?<email>.*)>');

main(String line) {
    final match = _userPattern.firstMatch(line);
    if (match != null) {
        match.namedGroup('name');
        match.namedGroup('email');
    }
}
```

### How to pretty print JSON with indent?

```dart
JsonEncoder.withIndent('··').convert(someObject);
```



### How to read, convert, decode files asynchronously?

Keywords: `ByteConversionSink`, `startChunkedConversion`

```dart
myFile
    .openRead()
    .transform(gzip.decoder)
    .transform(utf8.decoder)
    .transform(const LineSplitter())
    .listen((line) {
        print(line);
    });
```

### How to catch specific exceptions?

```dart
try {
	
} on UnresolvablePathException catch (e) {
	
}
```

### What is the difference between `const` and `final`?

Both declare a reference that cannot be changed

```dart
const a = 0;
final b = 0;

a = 2; // error
b = 2; // error
```

`const` 

- initialized only once during the application life cycle.

- always the same value, so its reused even in loops, or inside function bodies.

  ```dart
  for(var n in [1, 2, 3, 4, 5]) {
      const someList = [1, 2, 3]; // 1 allocation
  }
  ```

- can only contain objects with constant fields (it needs a `const` constructor), so its static recursively.

  ```dart
  const numbers = [1, 2, 3];
  numbers[0] = 10; // error
  ```

`final`

- initialized every time the line is reached.

  ```dart
  for(var n in [1, 2, 3, 4, 5]) {
      final someList = [1, 2, 3]; // 5 allocations
  }
  ```

- only the reference to the object is unchangeable, but the content of the object isn't.

  ```dart
  final numbers = [1, 2, 3];
  numbers[0] = 10; // OK
  ```

  

### How do I get a map key and throw an exception if it is missing?

```dart
myMap[someKey] ?? (throw ArgumentError.value(value, 'value', 'Not a valid key'));
```

### How to replace strings by regex?

```dart
"hello world".replaceFirst(RegExp(r'[helo]+'), 'goodbye');
```

Dart does not support using _group-references_ in the replacement string, but you can use `replaceAllMapped`.

**replaceAllMapped**

_replace markdown links with html_

```dart
final markdown = "example [link](http://example.com) replacement";
final html = markdown.replaceAllMapped(
  RegExp(r'\[([^\]]+)\]\(([^)]+)\)'),
  (match) => '<a href="${match.group(2)}">${match.group(1)}</a>',
);
// "example <a href="http://example.com">link</a> replacement"
```

### How to use await in a method that returns a Stream?

The problem

```dart
// Functions marked async must return Future
Stream<int> getStream() async {
    final object = await getObject();
    return object.dataStream;
}
```

```dart
// Can't return a value from a generator function (using the 'async*' modifier).
Stream<int> getStream() async* {
    final object = await getObject();
    return object.dataStream;
}
```

Solutions:

```dart
Stream<int> getStream() async* {
    final object = await getObject();
    yield* object.dataStream;
  }
}
// credit: Simon Binder <simolus3>
```

```dart
Stream<int> getStream() {
  return Stream
      .fromFuture(getObject())
      .asyncExpand((obj) => obj.dataStream);
}
// credit: Simon Binder <simolus3>
```

```dart
import 'package:async';
Stream<int> getStream() {
	return StreamCompleter.fromFuture(getObject().then((o) => o.dataStream));
}
// credit: Nate Bosch <natebosch>
```

### Async code is slow in dart2native

#perf

The following code runs at 50% of the speed in native `.exe` compared to running in the `dart` vm.

```dart
const GB = 1024*1024*1024;
final handle = file.openSync();
var total = 0;
do {
    total += blockSize;
    await handle.read(blockSize).then((Uint8List result) {
        // process result
    });
} while (total < GB);
```

There are two reasons why the native code is slower

- It is compiled ahead of time (AOT).
  This means the compiler cannot optimized based on the runtime behavior of the app
- It needs to allocate memory for `Uint8List` on each iteration.
  The JIT compiler of the VM figures out quickly that the list is always the same size, and will reuse the address.

Lesson:

> In **async** code that will be compiled natively, avoid reallocating memory which could have been reused.
>
> In **sync** code, the AOT compiler can figure out by static analysis that this block of memory is reusable, and there's no performance difference.

```dart
final handle = file.openSync();

var total = 0;
var result = Uint8List(blockSize);
do {
    total += blockSize;
    await handle.readInto(result).then((n) {
        // process result
    });
} while (total < length);

```

### FFI - Failed to load dynamic library

- path must use `\` on windows!
  - 👉 doesn't seem the case
- paths must be relative
  - 👉 doesn't seem the case

### FFI - Failed to load dynamic library (193)

- Your dll is not compatible due to CPU architecture type mismatch (32bit / 64bit)

See Cpp.md



### FFI - Failed to lookup symbol (127)

> Cannot find symbol from C++ compiled DLL on Windows

- include a `.def` file in the cpp project

- ```def
  LIBRARY   readfile_library
  EXPORTS
     readfile
  ```

- `CMakeLists.txt`

  - ```
    add_library(readfile_library SHARED readfile.cpp readfile.def)
    ```

- add `extern "C"` before exported functions

Fixes that **do not work**, and are not needed:

- using a header file (`.h`)
- adding `set(WINDOWS_EXPORT_ALL_SYMBOLS 1)` to `CMakeLists.txt`
- setting a public header for the library
  `set_target_properties(readfile_library PROPERTIES PUBLIC_HEADER readfile.h)`

### FFI - allocate<> not available

add `ffi: 0.1.3` to pubspec

### How to use DevTools with Dart (not flutter)?

https://dart.dev/tools/dart-devtools#using-devtools-with-a-command-line-app

- Add breakpoint before code of interest
- Start app in debugging mode
- Copy observatory url
- Open Intellij Actions: `Start Devtools`
- Connect with _observatory url_

### How to read binary data?

```dart
final handle = await File('pack123.pack').open(mode: FileMode.read); // RandomAccessFile
final header = await handle.read(12); // Uint8List
final headerBytes = header.buffer.asByteData(); // ByteData

final signature = ascii.decode(header.sublist(0, 4)); // String
final version = headerBytes.getUint32(4); // int
final entries = headerBytes.getUint32(8); // int
```

### How to improve performance?

Use _local_ variables:

```dart
int check = 0xFF;

void bad() {
    for(var i=0; i < 10000; i++) {
        check ^= i;
    }
}

void good() {
    int _check = 0xFF;
    for(var i=0; i < 10000; i++) {
        _check ^= i;
    }
    check = _check;
}
```

Avoid slow async functions:

```yaml
# analysis_options.yaml
linter:
  rules:
    - avoid_slow_async_io
```

Avoid using lots of callbacks:

- reduce
- map
- ...

Avoid `Uint8List.sublist`

Avoid `stdout.add` or `print`

- Instead write to a file

## Testing

### How to test exceptions?

```dart
expect(() => someFunction, throwsA(MyException));
expect(() => someFunction, throwsA((e) => e is MyException && e.flag));
```

## Building and running

### Web project configuration

`yourproject.iml` file should look like this, in the root:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<module type="WEB_MODULE" version="4">
  <component name="NewModuleRootManager" inherit-compiler-output="true">
    <exclude-output />
    <content url="file://$MODULE_DIR$">
      <excludeFolder url="file://$MODULE_DIR$/.dart_tool" />
      <excludeFolder url="file://$MODULE_DIR$/.pub" />
      <excludeFolder url="file://$MODULE_DIR$/build" />
    </content>
    <orderEntry type="sourceFolder" forTests="false" />
    <orderEntry type="library" name="Dart SDK" level="project" />
    <orderEntry type="library" name="Dart Packages" level="project" />
  </component>
</module>
```

`.idea/modules.xml` should look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project version="4">
  <component name="ProjectModuleManager">
    <modules>
      <module fileurl="file://$PROJECT_DIR$/yourproject.iml" filepath="$PROJECT_DIR$/yourproject.iml" />
    </modules>
  </component>
</project>
```



### How to run webdev from intellij

- Open the pubspec.yaml
- pub get
- new run configuration
- dart web
- choose web/index.html
- run



### How do I compile to Javascript / run in the browser?

**Recommended method:**

- Create a web project

  - Use [stagehand](https://github.com/dart-lang/stagehand)
  - *IntelliJ*
    - new module > dart > web

- Run `webdev serve`

  *IntelliJ*: from run configurations

   or

- `webdev build`

  - *IntelliJ*: from `pubspec.yaml`, click *Webdev build...*
  - `pub global run webdev build`

**_Not_ recommended method:**

- Writing dart code
- Compiling with `dart2js`
- Having no idea how to load the file into the Html



### How do I use dart2js?

No idea! But if you like to try, see what output `webdev build` generates to get an idea.

### How do I configure webdev / build.yaml?

Some docs are here: https://github.com/dart-lang/build/blob/master/docs/faq.md

In the root of the project, create a `build.yaml`

```yaml
targets:
  # Here you can create multiple targets.
  # Targets should not have any overlapping output files.
  # I haven't been able to achieve that. Even when sources and generate_for
  # don't overlap, the modules generate some metadata files that are overlapping.
  my_app:    # should match pubspec.yaml:name
  $default:  # default target
    # define which sources to use while building, and what to copy
    sources:
      # 
      - $package$         # needed to include packages
      - lib/**            # needed for loading modules
      - modules/**        # include extra source folders
      - web/**            # include everything from your public html folder
    builders:
      build_web_compilers|entrypoint:
        # These are globs for the entrypoints you want to compile.
        generate_for:
          # Specifies the source files for entry points
          # Other referenced source files are not automatically included
          # You need to add them to sources above
          - test/**.dart
          - web/**.dart
        options:
          compiler: dart2js
          # List any dart2js specific args here, or omit it.
          dart2js_args:
            - -O2
```

### How do I add dart2js web to an existing dart module?

Add `build.yaml`

```yaml
targets:
  $default:
    sources:
      - $package$
      - lib/**
      - web/**
    builders:
      build_web_compilers|entrypoint:
        generate_for:
          - web/**.dart
        options:
          compiler: dart2js
          dart2js_args:
            - -O2
```

Add `dev_dependencies`:

```yaml
dev_dependencies:
  build_runner: ^1.10.0
  build_web_compilers: ^2.11.0
```

Add web content:

```
web/
  index.html
  styles.css
  main.dart
```

`index.html`

```html
<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Demo</title>
    <link rel="stylesheet" href="styles.css">
    <script defer src="main.dart.js"></script>
</head>
```

`main.dart`

```dart
import 'dart:html';

void main() {
  querySelector('body').dataset.remove('loading');
}
```



### How do I generate multiple entry-points (js files)?

Define a single build-target, and specify multiple entry-points in `generate_for`.

```yaml
# build.yaml
targets:
  $default:
    sources:
      - $package$
      - lib/**
      - web/** # include all files from web
    builders:
      build_web_compilers|entrypoint:
        generate_for:
          - web/bookmarks/main.dart  # entrypoint for bookmarks module
          - web/notes/main.dart      # entrypoint for notes module
        options:
          compiler: dart2js
          dart2js_args:
            - -O2
```

### Is it safe to keep .dart files in web/?

Yes. When building for release, these files are cleaned up from the output.



### How to import files from the lib folder?

The `/lib` exposes files for import in the `<project package>`.

Example

```yaml
# pubspec.yaml
name: my_project                          #  <-- package name
```

```yaml
lib/
	foobar.dart
web/
	main.dart
pubspec.yaml
```

```dart
// web/main.dart
import 'package:my_project/foobar.dart'   // <-- import from package
```



### What is the library keyword?

Seems that its only used when you publish stuff on `pub`.

You don't need it. Not even when you are exposing files through your `lib` folder.

It's also not the name of the package when you import (e.g. `import 'package:foobar'` , `foobar` is the name of the package, not the library namespace).

> `library` is intended for JS-interop & visibility several years ago. Now every single file is a library by default. And `package` is a more useful way to define modular code.

### How to run with coverage

Unclear how I got it to work in intellij, but you don't need a package in pubspec.

Maybe it was `pub global activate coverage`.

## Issues

### Error: Target of URI doesn't exist

Do not create nested dart projects (e.g. multiple `pubspec.yaml`  files in parent/child directories).

```yaml
# bad
rootproject/
	pubspec.yaml      # <-- remove
	api/
		pubspec.yaml
```

Instead, create multiple sibling modules, and let them import each other.

```yaml
# good
rootproject/
	api/
		pubspec.yaml
	common/
		pubspec.yaml
```

```yaml
# api/pubspec.yaml
dependencies:
  common:
    path: ../common
```



### Dart doesn't run all my tests

:information_source: Your test files need to end in `_test.dart`



### Cannot run tests for web target

> Not found: 'dart:html'

:point_right: Assuming you are writing a *common* module with code used by both backend and web.

You're running web-targeted code in a module that doesn't compile to web. Backend modules (dart VM) don't have `dart:html`.

Workaround for now:

- Keep test code that needs `dart:html` (or other browser libraries) in separate web modules

Better solution, but haven't tried yet:

- Create multiple build targets for your module in `build.yaml` and compile the browser specific code separately with different config

### Webdev scripts do not run on `localhost:53322`

Kaspersky Antivirus may block this connection

- Web-Protection
  - Trusted URLS
    - add `http://127.0.0.1:55322/*`



### Pub get does not work from Intellij

> I click on `Pub get` in the toolbar, and nothing happens

Check if your Dart-SDK path is correct (frameworks & languages).

It was overwritten from `C:\tools\dart-sdk` to `C:\Apps\flutter\bin\cache\dart-sdk` (or similar) by installing flutter.

Reverting to the original dart-sdk fixed the issue.



### I get errors when I run my application from WSL

If you've run `pub get` from Windows, you must first run `pub get` from within WSL.

When switching back to Windows, you must run again `pub get` on Windows.

### Error while running test: The getter 'path' was called on null

You have a test `group()` without a `test()`

### Excluding a file in `analysis_options.yaml` doesn't work

Should be solved in `dart 2.9`

- See: https://github.com/dart-lang/sdk/issues/25551#issuecomment-608501698

### dart2native command not found

On Windows, `dart2native` is only installed as a `dart2native.bat` by Chocolatey.

If you run on Windows from a _Bash_ shell, use:

```bash
dart2native.bat
```

## FAQ

### What is Dartium?

A _deprecated_ development browser, based on Chrome for Dart `1.0`. Intended to run Dart natively in the browser, so you don't need to recompile.

Dart can now compile very fast to Javascript, using `webdev`, which uses the modular compiler `dartdevc` internally.



### What is Dart Web UI?

A _deprecated_ UI library, superseded by Polymer



### What is Dart Editor?

A _deprecated_ editor for Dart `1.0`



## Flutter

See [flutter.md](flutter.md) 

