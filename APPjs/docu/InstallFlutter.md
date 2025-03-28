Collecting workspace informationPour transformer votre application en une application mobile avec Flutter en utilisant WSL (Windows Subsystem for Linux), suivez les étapes ci-dessous :
React Native

### 1. Installer WSL
Si vous n'avez pas encore installé WSL, suivez les instructions sur le site officiel de Microsoft : [Installer WSL](https://docs.microsoft.com/fr-fr/windows/wsl/install).

### 2. Installer Flutter
Ouvrez votre terminal WSL et suivez les étapes suivantes pour installer Flutter :

#### a. Télécharger Flutter SDK
```sh
cd ~
wget https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_2.5.3-stable.tar.xz
tar xf flutter_linux_2.5.3-stable.tar.xz
```

#### b. Ajouter Flutter au PATH
Ajoutez Flutter au PATH en modifiant le fichier `.bashrc` ou `.zshrc` :
```sh
echo 'export PATH="$PATH:$HOME/flutter/bin"' >> ~/.bashrc
source ~/.bashrc
```

#### c. Vérifier l'installation
Vérifiez que Flutter est correctement installé :
```sh
flutter doctor
```

### 3. Créer un Nouveau Projet Flutter
Créez un nouveau projet Flutter en utilisant la commande suivante :
```sh
flutter create trinity_mobile_app
```

### 4. Configurer les Dépendances
Ajoutez les dépendances nécessaires dans le fichier `pubspec.yaml` de votre projet Flutter. Par exemple, pour utiliser `http` pour les requêtes API :
```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.13.3
```

### 5. Créer des Services pour Consommer l'API
Créez un service pour consommer votre API NestJS. Par exemple, créez un fichier `lib/services/api_service.dart` :
```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class ApiService {
  final String baseUrl = 'http://localhost:4000';

  Future<String> getHello() async {
    final response = await http.get(Uri.parse('$baseUrl/'));
    if (response.statusCode == 200) {
      return json.decode(response.body);
    } else {
      throw Exception('Failed to load data');
    }
  }
}
```

### 6. Créer des Widgets pour l'Interface Utilisateur
Créez des widgets pour afficher les données de votre API. Par exemple, dans `lib/main.dart` :
```dart
import 'package:flutter/material.dart';
import 'services/api_service.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Trinity Mobile App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final ApiService apiService = ApiService();
  String message = '';

  @override
  void initState() {
    super.initState();
    fetchMessage();
  }

  void fetchMessage() async {
    String response = await apiService.getHello();
    setState(() {
      message = response;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Trinity Mobile App'),
      ),
      body: Center(
        child: Text(message),
      ),
    );
  }
}
```

### 7. Tester l'Application
Lancez votre application Flutter sur un émulateur ou un appareil physique :
```sh
flutter run
```

### 8. Déployer l'Application
Pour déployer votre application sur Android et iOS, suivez les instructions de déploiement de Flutter :
- [Déploiement Android](https://flutter.dev/docs/deployment/android)
- [Déploiement iOS](https://flutter.dev/docs/deployment/ios)

En suivant ces étapes, vous pouvez transformer votre application NestJS en une application mobile Flutter en utilisant WSL.