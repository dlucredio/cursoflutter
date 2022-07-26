# Acesso a dados na rede

Alguns tutoriais para entender como a navegação imperativa em Flutter funciona:

[Fetch data from the Internet](https://docs.flutter.dev/cookbook/networking/fetch-data)

* Esse tutorial é interessante para o primeiro acesso. No final, tente adicionar uma espera para melhor visualizar o efeito da demora:

```dart
await Future.delayed(const Duration(seconds: 5));
```

* E também é possível modificar a URL para provocar um erro e visualizar o efeito.

[Send data to the Internet](https://docs.flutter.dev/cookbook/networking/send-data)

[Update data over the Internet](https://docs.flutter.dev/cookbook/networking/update-data)

[Delete data on the Internet](https://docs.flutter.dev/cookbook/networking/delete-data)

[Parse JSON in the background](https://docs.flutter.dev/cookbook/networking/background-parsing)

* Neste exemplo, é interessante para visualizar o travamento que ocorre, colocar um pequeno delay no parsing do JSON e remover o uso de `compute` para visualizar o efeito:

```dart
Future<List<Photo>> fetchPhotos(http.Client client) async {
  final response = await client
      .get(Uri.parse('https://jsonplaceholder.typicode.com/photos'));

  // Use the compute function to run parsePhotos in a separate isolate.
  //return compute(parsePhotos, response.body);
  return parsePhotos(response.body);
}

...

factory Photo.fromJson(Map<String, dynamic> json) {
    sleep(const Duration(milliseconds: 2));
    print([json['id']]);
    return Photo(
      albumId: json['albumId'] as int,
      id: json['id'] as int,
      title: json['title'] as String,
      url: json['url'] as String,
      thumbnailUrl: json['thumbnailUrl'] as String,
    );
  }
```