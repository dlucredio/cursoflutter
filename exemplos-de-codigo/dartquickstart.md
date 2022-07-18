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

## 2. Dart tem um gerenciador de dependências e suporte a execução assíncrona

Seguir o [tutorial CLI de Dart](https://dart.dev/tutorials/server/cmdline)

Utilizar o arquivo `lib/dcat.dart` para o método `dcat`

Compilar com o comando `dart compile exe bin/dcat.dart`

## 3. Dart é orientada a objetos

3.1. Modificar o arquivo `lib/dcat.dart`:

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

## 4. Dart tem segurança contra valores nulos

4.1. Veja, no exemplo anterior que, como as variáveis podem ser nulas, há necessidade de verificações a todo momento. Modificar para evitar isso:

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

4.2. Uma outra forma de fazer isso é usando parâmetros nomeados:

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

4.3. Como neste caso o conteúdo do construtor é apenas a inicialização, é possível simplificar:

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

4.4. Ou, caso queira utilizar somente parâmetros posicionais:

```dart
class Task {
  bool readFromInput;
  bool showLineNumbers;
  List<String> paths;

  Task(this.readFromInput, this.showLineNumbers, this.paths);
```

Neste caso, é preciso voltar à chamada posicional também.

## Mais informações

Há muitos detalhes interessantes sobre a linguagem Dart. O site oficial tem um [tour completo baseado em exemplos](https://dart.dev/guides/language/language-tour) bastante interessante para ser estudado.