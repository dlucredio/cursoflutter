# Primeiro contato com Dart

Este exemplo serve apenas como um primeiro contato com a linguagem Dart. Não é de forma alguma um tutorial completo ou uma descrição completa das suas funcionalidades. Apenas serve para ilustrar e esclarecer algumas coisas que irão aparecer nos exemplos em Flutter.

## 1. Dart é uma linguagem fortemente tipada, se você quiser!

Faça os exemplos a seguir em [https://dart.dev/#try-dart](https://dart.dev/#try-dart)

```dart
void main() {
  int x = 42;
  
  x = x + 2;
  
  print("O valor de x é $x");
}
```

Depois trocar x para `var`, e depois `final`.

Trocar pelo seguinte código:

```dart
void main() {
  String x = "42";
  
  x = x + "2";
  
  print("O valor de x é $x");
}
```

Se quiser, é possível fazer o seguinte:

```dart
void main() {
  dynamic x = "42";
  
  x = x + "2";
  
  print("O valor de x é $x");
  
  x = 42;
  x = x+2;
  print("O valor de x é $x");
}
```

É importante também entender como funcionam os parâmetros de funções. Dê uma olhada no [tour de funções e parâmetros em Dart](https://dart.dev/guides/language/language-tour#functions).

## 2. Dart tem suporte a execução assíncrona

Seguir o [tutorial de await/async](https://dart.dev/codelabs/async-await)

## 3. Dart tem um gerenciador de dependências

Seguir o [tutorial CLI de Dart](https://dart.dev/tutorials/server/cmdline)

Utilizar o arquivo `lib/dcat.dart` para o método `dcat`

Compilar com o comando `dart compile exe bin/dcat.dart`

## 4. Dart é orientada a objetos

4.1. Modificar o arquivo `lib/dcat.dart`:

```dart
import 'dart:io';
import 'dart:convert';

class Task {
  bool? readFromInput;
  bool? showLineNumbers;
  List<String>? paths;

  Task(bool? readFromInput, bool? showLineNumbers, List<String>? paths) {
    this.readFromInput = readFromInput;
    this.showLineNumbers = showLineNumbers;
    this.paths = paths;
  }

  Future<void> dcat() async {
    exitCode = 0;

    if (readFromInput ?? false) {
      await stdin.pipe(stdout);
    } else {
      for (final path in paths ?? []) {
        var lineNumber = 1;
        final lines = utf8.decoder
            .bind(File(path).openRead())
            .transform(const LineSplitter());
        try {
          await for (final line in lines) {
            if (showLineNumbers ?? false) {
              stdout.write('${lineNumber++} ');
            }
            stdout.writeln(line);
          }
        } catch (_) {
          await _handleError(path);
        }
      }
    }
  }

  Future<void> _handleError(String path) async {
    if (await FileSystemEntity.isDirectory(path)) {
      stderr.writeln('error: $path is a directory');
    } else {
      exitCode = 2;
    }
  }
}
```

## 5. Dart tem segurança contra valores nulos

5.1. Veja, no exemplo anterior que, como as variáveis podem ser nulas, há necessidade de verificações a todo momento. Modificar para evitar isso:

```dart
import 'dart:io';
import 'dart:convert';

class Task {
  bool readFromInput = false;
  bool showLineNumbers = false;
  List<String> paths = [];

  Task(bool? readFromInput, bool? showLineNumbers, List<String>? paths) {
    this.readFromInput = readFromInput ?? false;
    this.showLineNumbers = showLineNumbers ?? false;
    this.paths = paths ?? [];
  }

  Future<void> dcat() async {
    exitCode = 0;

    if (readFromInput) {
      await stdin.pipe(stdout);
    } else {
      for (final path in paths) {
        var lineNumber = 1;
        final lines = utf8.decoder
            .bind(File(path).openRead())
            .transform(const LineSplitter());
        try {
          await for (final line in lines) {
            if (showLineNumbers) {
              stdout.write('${lineNumber++} ');
            }
            stdout.writeln(line);
          }
        } catch (_) {
          await _handleError(path);
        }
      }
    }
  }

  Future<void> _handleError(String path) async {
    if (await FileSystemEntity.isDirectory(path)) {
      stderr.writeln('error: $path is a directory');
    } else {
      exitCode = 2;
    }
  }
}
```

5.2. Uma outra forma de fazer isso é usando parâmetros nomeados:

```dart
import 'dart:io';
import 'dart:convert';

class Task {
  bool readFromInput = false;
  bool showLineNumbers = false;
  List<String> paths = [];

  Task(
      {bool readFromInput = false,
      bool showLineNumbers = false,
      List<String> paths = const []}) {
    this.readFromInput = readFromInput;
    this.showLineNumbers = showLineNumbers;
    this.paths = paths;
  }

  Future<void> dcat() async {
    exitCode = 0;

    if (readFromInput) {
      await stdin.pipe(stdout);
    } else {
      for (final path in paths) {
        var lineNumber = 1;
        final lines = utf8.decoder
            .bind(File(path).openRead())
            .transform(const LineSplitter());
        try {
          await for (final line in lines) {
            if (showLineNumbers) {
              stdout.write('${lineNumber++} ');
            }
            stdout.writeln(line);
          }
        } catch (_) {
          await _handleError(path);
        }
      }
    }
  }

  Future<void> _handleError(String path) async {
    if (await FileSystemEntity.isDirectory(path)) {
      stderr.writeln('error: $path is a directory');
    } else {
      exitCode = 2;
    }
  }
}
```

Isso significa que é necessário mudar também na chamada do construtor:

```dart
  final task = Task(
      readFromInput: readFromInput,
      showLineNumbers: argResults[lineNumber] as bool,
      paths: paths);
```

Neste caso, todos os argumentos são opcionais. Caso queira que sejam obrigatórios, adicione a palavra reservada `required`:

```dart
class Task {
  bool readFromInput = false;
  bool showLineNumbers = false;
  List<String> paths = [];

  Task(
      {bool readFromInput = false,
      required bool showLineNumbers,
      List<String> paths = const []}) {
    this.readFromInput = readFromInput;
    this.showLineNumbers = showLineNumbers;
    this.paths = paths;
  }
```

5.3. Como neste caso o conteúdo do construtor é apenas a inicialização, é possível simplificar:

```dart
class Task {
  bool readFromInput;
  bool showLineNumbers;
  List<String> paths;

  Task(
      {this.readFromInput = false,
      this.showLineNumbers = false,
      this.paths = const []});
```

5.4. Ou, caso queira utilizar somente parâmetros posicionais:

```dart
class Task {
  bool readFromInput;
  bool showLineNumbers;
  List<String> paths;

  Task(this.readFromInput, this.showLineNumbers, this.paths);
```

Neste caso, é preciso voltar à chamada posicional também.

5.5. Também é possível criar um construtor nomeado:

```dart
  Task.fromInput()
      : readFromInput = true,
        showLineNumbers = false,
        paths = [];
```

Assim, quando for utilizá-lo, basta chamá-lo:

```dart
final task = Task.fromInput();
```

## 6. Em Dart, funções são objetos de primeira classe

6.1. Modifique o arquivo `bin/dcat.dart`:

```dart
import 'package:args/args.dart';
import 'package:dcat/dcat.dart';

const lineNumber = 'line-number';

void log(String txt) {
  print("${DateTime.now().toString()} : $txt");
}

void main(List<String> arguments) {
  final parser = ArgParser();
  parser.addFlag(lineNumber, negatable: false, abbr: 'n');
  ArgResults argResults = parser.parse(arguments);
  final paths = argResults.rest;

  final readFromInput = paths.isEmpty;

  final task = Task(readFromInput, argResults[lineNumber], paths);

  task.dcat(log);
}
```

6.2. Modifique o arquivo `lib/dcat.dart`:

```dart
import 'dart:io';
import 'dart:convert';

class Task {
  bool readFromInput;
  bool showLineNumbers;
  List<String> paths;

  Task(this.readFromInput, this.showLineNumbers, this.paths);

  Task.fromInput()
      : readFromInput = true,
        showLineNumbers = false,
        paths = [];

  Future<void> dcat(Function log) async {
    log("Job started!");
    exitCode = 0;

    if (readFromInput) {
      await stdin.pipe(stdout);
    } else {
      for (final path in paths) {
        var lineNumber = 1;
        final lines = utf8.decoder
            .bind(File(path).openRead())
            .transform(const LineSplitter());
        try {
          await for (final line in lines) {
            if (showLineNumbers) {
              stdout.write('${lineNumber++} ');
            }
            stdout.writeln(line);
          }
        } catch (_) {
          await _handleError(path);
        }
      }
    }

    log("Job completed!");
  }

  Future<void> _handleError(String path) async {
    if (await FileSystemEntity.isDirectory(path)) {
      stderr.writeln('error: $path is a directory');
    } else {
      exitCode = 2;
    }
  }
}
```

Veja como a função é passada como um parâmetro comum, e chamada normalmente.

6.3. Também é possível restringir os tipos de retorno e parâmetros da função:

```dart
    Future<void> dcat(int Function(String) log) async {
        ...
```

Tente passar uma função com argumentos/retorno diferentes do esperado.

6.4. É também possível utilizar funções anônimas:

```dart
  task.dcat((txt) {
    print("${DateTime.now().toString()} : $txt");
    return 0;
  });
```

6.5. Se a função anônima não faz nada além de retornar um valor, é possível simplificar:

No arquivo `lib/dcat.dart`:

```dart
  Future<void> dcat(
    String Function(String) log,
  ) async {
    print(log("Job started!"));
    ...
    print(log("Job completed!"));
  }
```

E no arquivo `bin/dcat.dart`:

```dart
  task.dcat((txt) {
    return "${DateTime.now().toString()} : $txt";
  });
```

ou

```dart
  task.dcat((txt) => "${DateTime.now().toString()} : $txt");

```

6.6. Dá para usar isso para facilmente manipular valores:

```dart
import 'package:args/args.dart';
import 'package:dcat/dcat.dart';

const lineNumber = 'line-number';
var logs = <String>[];

Future<void> main(List<String> arguments) async {
  final parser = ArgParser();
  parser.addFlag(lineNumber, negatable: false, abbr: 'n');
  ArgResults argResults = parser.parse(arguments);
  final paths = argResults.rest;

  final readFromInput = paths.isEmpty;

  final task = Task(readFromInput, argResults[lineNumber], paths);

  await task.dcat(log);

  Iterable<String> upperCaseLogs = logs.map((t) => t.toUpperCase());
  print("All uppercase logs:");
  for (var e in upperCaseLogs) {
    print(e);
  }
}

String log(String txt) {
  String logTxt = "${DateTime.now().toString()} : $txt";
  logs.add(logTxt);
  return logTxt;
}
```

6.7. Dá também para fazer coisas legais como transformar valores em objetos:

```dart
import 'package:args/args.dart';
import 'package:dcat/dcat.dart';

const lineNumber = 'line-number';
var logs = <String>[];

Future<void> main(List<String> arguments) async {
  final parser = ArgParser();
  parser.addFlag(lineNumber, negatable: false, abbr: 'n');
  ArgResults argResults = parser.parse(arguments);
  final paths = argResults.rest;

  final readFromInput = paths.isEmpty;

  final task = Task(readFromInput, argResults[lineNumber], paths);

  await task.dcat(log);

  Iterable<String> upperCaseLogs = logs.map((t) => t.toUpperCase());
  print("All uppercase logs:");
  for (var e in upperCaseLogs) {
    print(e);
  }

  Iterable<LogItem> detailedLogs = logs.map((t) {
    var splitted = t.split(' : ');
    var datetime = splitted[0];
    var text = splitted[1];
    return LogItem(text, DateTime.parse(datetime));
  });

  for (var element in detailedLogs) {
    print(element.toString());
  }
}

String log(String txt) {
  String logTxt = "${DateTime.now().toString()} : $txt";
  logs.add(logTxt);
  return logTxt;
}

class LogItem {
  final String text;
  final DateTime date;
  LogItem(this.text, this.date);

  @override
  String toString() {
    return "Log item:\n"
        "   Text: $text\n"
        "   Time: ${date.minute}:${date.second}";
  }
}
```

## Mais informações

Há muitos detalhes interessantes sobre a linguagem Dart. O site oficial tem um [tour completo baseado em exemplos](https://dart.dev/guides/language/language-tour) bastante interessante para ser estudado.