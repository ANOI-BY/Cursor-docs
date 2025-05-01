
# Flutter iOS Application Documentation

## Table of Contents
1. [Project Architecture](#project-architecture)
2. [Clean Architecture Overview](#clean-architecture-overview)
3. [Cupertino and Human Interface Guidelines](#cupertino-and-human-interface-guidelines)
4. [Database Setup with sqflite](#database-setup-with-sqflite)
5. [Database Helper (database_helper.dart)](#database-helper-database_helperdart)
6. [Color Constants (colors.dart)](#color-constants-colorsdart)
7. [Root Widget Configuration (app.dart)](#root-widget-configuration-appdart)
8. [Project Dependencies (pubspec.yaml)](#root-project-dependencies-pubspec.yaml)
---

## 1. Project Architecture

Before writing any code, it is crucial to understand the overall architecture of the project. This application follows **Clean Architecture** principles, dividing concerns into separate layers to improve maintainability and testability.

The `lib` directory is organized as follows:

```yaml
lib:
  - core:
      - app
      - common
      - constants
      - utils

  - features:
      - main:
          - data:
              - models
              - repositories
          - domain:
              - entities
              - repositories
              - use_cases
          - presentation
      - onboarding:
          - data:
              - models
              - repositories
          - domain:
              - entities
              - repositories
              - use_cases
          - presentation:
              - manager
              - pages
              - widgets
```

Each layer has a single responsibility:

- **core**: Shared code, including app initialization, constants, utilities, and common widgets.
- **features**: Feature-specific modules. Each feature has three layers:
  - **data**: Models and repository implementations.
  - **domain**: Business logic, entities, repository interfaces, and use cases.
  - **presentation**: UI code, state managers, pages, and widgets.

---

## 2. Clean Architecture Overview

Clean Architecture divides the codebase into the following layers:

1. **Presentation Layer**: Flutter widgets, state management (e.g., Provider, Bloc, or Riverpod).
2. **Domain Layer**: Pure Dart. Contains entities, repository interfaces, and use cases.
3. **Data Layer**: External data sources (e.g., local database, network). Contains concrete repository implementations and data models.

Dependencies flow inward: Presentation → Domain → Data.

---

## 3. Cupertino and Human Interface Guidelines

Since the app is iOS-only, use **Cupertino** widgets exclusively to achieve a native look and feel.

- Import Cupertino library:
  ```dart
  import 'package:flutter/cupertino.dart';
  ```
- Follow Apple’s [Human Interface Guidelines (HIG)](https://developer.apple.com/design/human-interface-guidelines/ios/overview/themes/):
  - Use standard navigation bars, tab bars, and controls.
  - Maintain consistent typography and spacing.
  - Support Dynamic Type for accessibility.

---

## 4. Database Setup with sqflite

Use the `sqflite` package (version `^2.4.2`) for local persistence.

Add the dependency in `pubspec.yaml`:
```yaml
dependencies:
  sqflite: ^2.4.2
  path_provider: ^2.0.0
```

---

## 5. Database Helper (database_helper.dart)

Create **`database_helper.dart`** in `lib/core/utils` to manage database initialization and CRUD operations.

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static final DatabaseHelper _instance = DatabaseHelper._internal();
  factory DatabaseHelper() => _instance;

  static Database? _database;

  DatabaseHelper._internal();

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  Future<Database> _initDatabase() async {
    final dbPath = await getDatabasesPath();
    final path = join(dbPath, 'app_database.db');

    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  Future _onCreate(Database db, int version) async {
    // Create tables
    await db.execute('''
      CREATE TABLE example(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT
      )
    ''');
  }

  // Add CRUD helper methods here
}
```

Key responsibilities:
- Initialize the database.
- Provide singleton access.
- Define table creation scripts.
- Expose CRUD methods.

---

## 6. Color Constants (colors.dart)

Maintain a central file for color definitions in `lib/core/constants/colors.dart`.

```dart
import 'package:flutter/cupertino.dart';

class AppColors {
  static const CupertinoColor primary = CupertinoColors.systemBlue;
  static const CupertinoColor secondary = CupertinoColors.systemGray;
  static const CupertinoColor background = CupertinoColors.extraLightBackgroundGray;
  // Add additional colors as needed
}
```

Usage:
```dart
Container(
  color: AppColors.background,
);
```

---

## 7. Root Widget Configuration (app.dart)

Replace the root widget implementation in `lib/core/app/app.dart` with the following code to integrate Riverpod, localization, onboarding logic, and Talker logging:

```dart
import 'package:flutter/cupertino.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
import 'package:hooks_riverpod/hooks_riverpod.dart';
import 'package:talker_flutter/talker_flutter.dart';

import '../../features/main/presentation/pages/main.dart';
import '../constants/strings.dart';
import '../utils/preferences.dart';
import 'logger.dart';

class Application extends StatelessWidget {
  const Application({super.key});

  @override
  Widget build(BuildContext context) {
    // Determine if onboarding has completed
    final onBoardingIsComplete = true; // AppPreferences.I.appLaunched;

    return ProviderScope(
      child: CupertinoApp(
        debugShowCheckedModeBanner: false,
        theme: CupertinoThemeData(brightness: Brightness.dark),
        localizationsDelegates: const [
          GlobalCupertinoLocalizations.delegate,
          GlobalMaterialLocalizations.delegate,
          GlobalWidgetsLocalizations.delegate,
        ],
        supportedLocales: const [Locale('en', '')],
        navigatorObservers: [TalkerRouteObserver(talker)],
        home: onBoardingIsComplete ? const MainPage() : const SizedBox(),
      ),
    );
  }
}
```

Responsibilities:
- Wrap the app in a `ProviderScope` for Riverpod state management.
- Configure `CupertinoApp` with dark theme and disable debug banner.
- Add localization delegates for Cupertino, Material, and Widgets.
- Observe navigation with Talker for logging via `TalkerRouteObserver`.
- Conditionally show `MainPage` or a placeholder based on onboarding flag.

---

## 8. Project Dependencies (pubspec.yaml)

Below is an example `pubspec.yaml` showing all dependencies used in this project:

```yaml
name: tt_223
description: "A new Flutter project."
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: ^3.5.4

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.8
  hooks_riverpod: ^2.6.1
  flutter_hooks: ^0.20.5
  riverpod_annotation: ^2.6.1
  shared_preferences: ^2.5.2
  in_app_review: ^2.0.10
  modal_bottom_sheet: ^3.0.0
  intl: ^0.19.0
  flutter_launcher_icons: ^0.14.3
  flutter_native_splash: ^2.4.5
  google_fonts: ^6.2.1
  fl_chart: ^0.70.2
  flutter_svg: ^2.0.17
  path_provider: ^2.1.5
  flutter_animate: ^4.5.2
  flagsmith: ^6.0.0
  firebase_core: ^3.11.0
  firebase_messaging: ^15.2.2
  talker: ^4.6.11
  talker_flutter: ^4.6.11
  talker_dio_logger: ^4.6.11

  flutter_localizations:
    sdk: flutter
  package_info_plus: ^8.1.4
  flutter_email_sender: ^7.0.0
  sqflite: ^2.4.2
  webview_flutter_wkwebview: 3.16.3
  webview_flutter: 4.10.0
  dart_openai: ^5.1.0
  flutter_markdown: ^0.7.6+2

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
  riverpod_generator: 2.6.3
  build_runner: ^2.4.15
  riverpod_lint: ^2.6.3
  flutter_gen_runner: ^5.9.0
  objectbox_generator: ^4.1.0

flutter_gen:
  output: lib/core/gen/
  line_length: 80

flutter_native_splash:
  image: assets/images/icon_cropped.png
  color: '#DA150D'

flutter_launcher_icons:
  ios: true
  remove_alpha_ios: true
  image_path: "assets/images/icon.png"

flutter:
  uses-material-design: true

  assets:
    - assets/images/
```

*End of Documentation*

