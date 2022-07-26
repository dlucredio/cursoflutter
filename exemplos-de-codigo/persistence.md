# Persistência

## Usando SQLite (apenas Android e iOS)

1. Criar um novo projeto Flutter:

```sh
flutter create persistence
```

2. Adicionar as dependências ao arquivo `pubspec.yaml`:

```diff
dependencies:
  flutter:
    sdk: flutter
+  path: ^1.8.1
+  uuid: ^3.0.6
+  http: ^0.13.4
```

Atualizar as dependências:

```sh
flutter pub get
```

3. Criar um arquivo `lib/anime_quote.dart`:

```dart
const randomAnimeQuoteUrl = 'https://animechan.vercel.app/api/random';

class AnimeQuote {
  final String id;
  final String anime;
  final String character;
  final String quote;
  bool favorite;

  AnimeQuote({
    required this.id,
    required this.anime,
    required this.character,
    required this.quote,
    required this.favorite,
  });

  factory AnimeQuote.fromMap(Map<String, dynamic> map) {
    return AnimeQuote(
        id: map['id'],
        anime: map['anime'],
        character: map['character'],
        quote: map['quote'],
        favorite: map['favorite'] == 'true');
  }

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'anime': anime,
      'character': character,
      'quote': quote,
      'favorite': favorite ? 'true' : 'false',
    };
  }
}
```

4. Vamos primeiro criar um aplicativo que funciona em memória:

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:persistence/anime_quote.dart';
import 'package:http/http.dart' as http;
import 'package:uuid/uuid.dart';

void main() async {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Anime quotes',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Flutter persistence'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key, required this.title}) : super(key: key);

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  late List<AnimeQuote> _quotes;

  @override
  void initState() {
    super.initState();
    _quotes = _fetchQuotesFromDatabase();
  }

  List<AnimeQuote> _fetchQuotesFromDatabase() {
    return List.generate(
      10,
      (index) => AnimeQuote(
        id: const Uuid().v1(),
        anime: "Anime $index",
        character: "Character $index",
        quote: "Quote $index",
        favorite: false,
      ),
    );
  }

  Future<AnimeQuote> _fetchRandomQuote() async {
    final response = await http.get(Uri.parse(randomAnimeQuoteUrl));

    if (response.statusCode == 200) {
      final map = jsonDecode(response.body);
      map['id'] = const Uuid().v1();
      return AnimeQuote.fromMap(map);
    } else {
      throw Exception('Failed to load random quote');
    }
  }

  void _addRandomQuote() async {
    final randomQuote = await _fetchRandomQuote();
    setState(() {
      _quotes.add(randomQuote);
    });
  }

  void _toggleFavorite(AnimeQuote quote) async {
    setState(() {
      for (var q in _quotes) {
        if (q.id == quote.id) {
          q.favorite = !quote.favorite;
        }
      }
    });
  }

  void _deleteQuote(AnimeQuote quote) async {
    setState(() {
      _quotes.removeWhere((q) => q.id == quote.id);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: QuotesList(
          quotes: _quotes,
          toggleFavoriteCallback: _toggleFavorite,
          deleteQuoteCallback: _deleteQuote,
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _addRandomQuote,
        tooltip: 'Add Random Quote',
        child: const Icon(Icons.add),
      ),
    );
  }
}

class QuotesList extends StatelessWidget {
  final List<AnimeQuote> quotes;
  final void Function(AnimeQuote) toggleFavoriteCallback;
  final void Function(AnimeQuote) deleteQuoteCallback;

  const QuotesList(
      {required this.quotes,
      Key? key,
      required this.toggleFavoriteCallback,
      required this.deleteQuoteCallback})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: quotes.length,
      itemBuilder: (context, index) {
        return GestureDetector(
          onHorizontalDragEnd: (details) => deleteQuoteCallback(quotes[index]),
          child: ListTile(
            title: Row(
              crossAxisAlignment: CrossAxisAlignment.end,
              children: [
                Expanded(
                  child: Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      children: [
                        Text(
                          quotes[index].quote,
                          textAlign: TextAlign.justify,
                        ),
                        Text(
                          "${quotes[index].character} from ${quotes[index].anime}",
                          style: DefaultTextStyle.of(context)
                              .style
                              .apply(fontSizeFactor: 0.7)
                              .apply(fontStyle: FontStyle.italic),
                          textAlign: TextAlign.end,
                        ),
                      ],
                    ),
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.all(2.0),
                  child: IconButton(
                    icon: Icon(
                      quotes[index].favorite
                          ? Icons.favorite
                          : Icons.favorite_border,
                      color: quotes[index].favorite ? Colors.red : Colors.grey,
                    ),
                    onPressed: () => toggleFavoriteCallback(quotes[index]),
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

5. Agora vamos adicionar persistência. Primeiro adicione a dependência ao `pubspec.yaml` (e atualize rodando `flutter pub get`):

```diff
dependencies:
  flutter:
    sdk: flutter
  path: ^1.8.1
  uuid: ^3.0.6
  http: ^0.13.4
+  sqflite: ^2.0.3
```

6. Crie uma classe chamada `lib/database.dart`:

```dart
import 'dart:async';

import 'package:path/path.dart';
import 'package:persistence/anime_quote.dart';
import 'package:sqflite/sqflite.dart';

const tableName = 'quotes';

class DBProvider {
  DBProvider._();
  static final DBProvider db = DBProvider._();
  static Database? _database;
  Future<Database> get database async => _database ?? await _initDB();

  Future<Database> _initDB() async {
    return openDatabase(
      join(await getDatabasesPath(), 'anime_quotes.db'),
      onCreate: (db, version) {
        return db.execute(
          'CREATE TABLE $tableName(id TEXT PRIMARY KEY,'
          'anime TEXT,'
          'character TEXT,'
          'quote TEXT,'
          'favorite TEXT)',
        );
      },
      version: 1,
    );
  }

  Future<List<AnimeQuote>> getAllAnimeQuotes() async {
    final db = await database;
    final List<Map<String, dynamic>> maps = await db.query(tableName);
    return List.generate(maps.length, (i) => AnimeQuote.fromMap(maps[i]));
  }

  Future<void> insertNewQuote(AnimeQuote quote) async {
    final db = await database;
    await db.insert(tableName, quote.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace);
  }

  Future<void> updateQuote(AnimeQuote quote) async {
    final db = await database;
    await db.update(tableName, quote.toMap(),
        where: 'id = ?', whereArgs: [quote.id]);
  }

  Future<void> deleteQuote(AnimeQuote quote) async {
    final db = await database;
    await db.delete(tableName, where: 'id = ?', whereArgs: [quote.id]);
  }
}
```

7. Agora vamos modificar o código principal:

```diff
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:persistence/anime_quote.dart';
+import 'package:persistence/database.dart';
import 'package:http/http.dart' as http;
import 'package:uuid/uuid.dart';

void main() async {
+  WidgetsFlutterBinding.ensureInitialized();
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Anime quotes',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MyHomePage(title: 'Flutter persistence'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({Key? key, required this.title}) : super(key: key);

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
-  late List<AnimeQuote> _quotes;
+  late Future<List<AnimeQuote>> _quotes;

  @override
  void initState() {
    super.initState();
    _quotes = _fetchQuotesFromDatabase();
  }

-  List<AnimeQuote> _fetchQuotesFromDatabase() {
-    return List.generate(
-      10,
-      (index) => AnimeQuote(
-        id: const Uuid().v1(),
-        anime: "Anime $index",
-        character: "Character $index",
-        quote: "Quote $index",
-        favorite: false,
-      ),
-    );
-  }
+  Future<List<AnimeQuote>> _fetchQuotesFromDatabase() async {
+    return DBProvider.db.getAllAnimeQuotes();
+  }

  Future<AnimeQuote> _fetchRandomQuote() async {
    final response = await http.get(Uri.parse(randomAnimeQuoteUrl));

    if (response.statusCode == 200) {
      final map = jsonDecode(response.body);
      map['id'] = const Uuid().v1();
      return AnimeQuote.fromMap(map);
    } else {
      throw Exception('Failed to load random quote');
    }
  }

  void _addRandomQuote() async {
    final randomQuote = await _fetchRandomQuote();
+    await DBProvider.db.insertNewQuote(randomQuote);
    setState(() {
-      _quotes.add(randomQuote);
+      _quotes = _fetchQuotesFromDatabase();
    });
  }

  void _toggleFavorite(AnimeQuote quote) async {
+    quote.favorite = !quote.favorite;
+    await DBProvider.db.updateQuote(quote);
    setState(() {
-      for (var q in _quotes) {
-        if (q.id == quote.id) {
-          q.favorite = !quote.favorite;
-        }
-      }
+      _quotes = _fetchQuotesFromDatabase();
    });
  }

  void _deleteQuote(AnimeQuote quote) async {
+    await DBProvider.db.deleteQuote(quote);
    setState(() {
-      _quotes.removeWhere((q) => q.id == quote.id);
+      _quotes = _fetchQuotesFromDatabase();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
-        child: QuotesList(
-          quotes: _quotes,
+        child: FutureBuilder<List<AnimeQuote>>(
+          future: _quotes,
+          builder: (context, snapshot) {
+            if (snapshot.hasData) {
+              return QuotesList(
+                quotes: snapshot.data!,
                toggleFavoriteCallback: _toggleFavorite,
                deleteQuoteCallback: _deleteQuote,
 +             );
 +           } else if (snapshot.hasError) {
 +             return Text('${snapshot.error}');
 +           }
 +           return const CircularProgressIndicator();
 +         },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _addRandomQuote,
        tooltip: 'Add Random Quote',
        child: const Icon(Icons.add),
      ),
    );
  }
}

class QuotesList extends StatelessWidget {
  final List<AnimeQuote> quotes;
  final void Function(AnimeQuote) toggleFavoriteCallback;
  final void Function(AnimeQuote) deleteQuoteCallback;

  const QuotesList(
      {required this.quotes,
      Key? key,
      required this.toggleFavoriteCallback,
      required this.deleteQuoteCallback})
      : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: quotes.length,
      itemBuilder: (context, index) {
        return GestureDetector(
          onHorizontalDragEnd: (details) => deleteQuoteCallback(quotes[index]),
          child: ListTile(
            title: Row(
              crossAxisAlignment: CrossAxisAlignment.end,
              children: [
                Expanded(
                  child: Padding(
                    padding: const EdgeInsets.all(8.0),
                    child: Column(
                      crossAxisAlignment: CrossAxisAlignment.stretch,
                      children: [
                        Text(
                          quotes[index].quote,
                          textAlign: TextAlign.justify,
                        ),
                        Text(
                          "${quotes[index].character} from ${quotes[index].anime}",
                          style: DefaultTextStyle.of(context)
                              .style
                              .apply(fontSizeFactor: 0.7)
                              .apply(fontStyle: FontStyle.italic),
                          textAlign: TextAlign.end,
                        ),
                      ],
                    ),
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.all(2.0),
                  child: IconButton(
                    icon: Icon(
                      quotes[index].favorite
                          ? Icons.favorite
                          : Icons.favorite_border,
                      color: quotes[index].favorite ? Colors.red : Colors.grey,
                    ),
                    onPressed: () => toggleFavoriteCallback(quotes[index]),
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```