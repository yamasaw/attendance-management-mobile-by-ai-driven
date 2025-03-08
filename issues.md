# 環境構築用Issue: FlutterアプリケーションのHello World実装

## 目的
Flutterの開発環境を構築し、基本的なHello Worldアプリケーションを実装して動作確認を行う。

## タスク内容

### 1. Flutter SDK のインストールと設定
- [ ] Flutter SDKをダウンロードしてインストール (https://docs.flutter.dev/get-started/install)
- [ ] 環境変数PATHにFlutterのbinディレクトリを追加
- [ ] `flutter doctor`コマンドを実行して、必要な依存関係をすべてインストール
- [ ] Android Studio または Visual Studio Code に Flutter と Dart プラグインをインストール

### 2. プロジェクトの作成
- [ ] 以下のコマンドで新しいFlutterプロジェクトを作成:
  ```bash
  flutter create attendance_management_mobile
  cd attendance_management_mobile
  ```
- [ ] 作成したプロジェクトのディレクトリ構造を確認

### 3. Hello World アプリの実装
- [ ] `lib/main.dart`ファイルを以下のコードで置き換え:
  ```dart
  import 'package:flutter/material.dart';

  void main() {
    runApp(const MyApp());
  }

  class MyApp extends StatelessWidget {
    const MyApp({super.key});

    @override
    Widget build(BuildContext context) {
      return MaterialApp(
        title: '勤怠管理アプリ',
        theme: ThemeData(
          colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
          useMaterial3: true,
        ),
        home: const MyHomePage(),
      );
    }
  }

  class MyHomePage extends StatelessWidget {
    const MyHomePage({super.key});

    @override
    Widget build(BuildContext context) {
      return Scaffold(
        appBar: AppBar(
          title: const Text('勤怠管理アプリ'),
          backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        ),
        body: Center(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              const Text(
                'Hello World!',
                style: TextStyle(fontSize: 24),
              ),
              const SizedBox(height: 20),
              const Text(
                '勤怠管理システム - 共有端末版',
                style: TextStyle(fontSize: 18, color: Colors.grey),
              ),
            ],
          ),
        ),
      );
    }
  }
  ```

### 4. アプリの実行とテスト
- [ ] エミュレータを起動するか、実機をUSBデバッグモードで接続
- [ ] 以下のコマンドでアプリを実行:
  ```bash
  flutter run
  ```
- [ ] アプリが起動し、「Hello World!」と「勤怠管理システム - 共有端末版」が表示されることを確認

### 5. プロジェクト構造の整備（オプション）
- [ ] 以下のディレクトリ構造を作成し、今後の開発に備える:
  ```
  lib/
  ├── data/
  │   ├── models/
  │   └── services/
  ├── device/
  │   ├── hardware/
  │   ├── kiosk/
  │   └── monitoring/
  ├── presentation/
  │   ├── pages/
  │   ├── providers/
  │   └── widgets/
  └── utils/
  ```

## 完了条件
- Flutter開発環境が正常に構築されている
- Hello Worldアプリが正常に実行され、テキストが表示される
- プロジェクト構造が整備されている（オプション）

## 参考リンク
- Flutter公式ドキュメント: https://docs.flutter.dev/
- Flutter入門ガイド: https://docs.flutter.dev/get-started/codelab
- Material Design 3ガイドライン: https://m3.material.io/ 