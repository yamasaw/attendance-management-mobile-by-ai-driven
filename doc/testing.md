# 勤怠管理システム - テスト計画

## 1. テスト基本方針

テストは「何が」テストされているかではなく、「どのように」「どのような観点で」テストするかに焦点を当てています。本モバイルアプリケーション開発では主に以下の種類のテストを実施します：

1. 単体テスト（Unit Tests）
2. ウィジェットテスト（Widget Tests）
3. 統合テスト（Integration Tests）
4. ユーザビリティテスト

これらのテストは自動化され、CI/CDパイプラインに統合されることで、継続的な品質保証を実現します。

## 2. 単体テスト（Unit Tests）

### テスト技術スタック
- テストフレームワーク: **Flutter Test**
- モックライブラリ: **Mockito**
- カバレッジツール: **lcov**

### テスト観点

#### ビジネスロジック
- **状態管理**: Riverpodプロバイダーが適切な状態を管理しているか
- **データ変換**: モデルクラスのシリアライズ/デシリアライズが正しく動作するか
- **バリデーション**: 入力検証ロジックが期待通り機能するか

#### サービスレイヤー
- **API通信**: APIクライアントが正しいリクエストを送信し、レスポンスを処理できるか
- **ローカルストレージ**: Hiveやshared_preferencesによるデータ永続化が正しく機能するか
- **認証処理**: ログイン/ログアウトのロジックが正しく動作するか

### テスト手法

#### 独立テスト
- 外部依存をモックに置き換えてクラス/メソッドを単体でテスト

```dart
test('勤怠記録サービス: 正常なリクエストで勤怠記録が作成される', () async {
  // モックセットアップ
  final mockApiClient = MockApiClient();
  when(mockApiClient.postAttendance(any)).thenAnswer(
    (_) async => AttendanceRecord(id: '123', timestamp: DateTime.now())
  );
  
  final service = AttendanceService(apiClient: mockApiClient);
  final result = await service.recordAttendance(
    type: AttendanceType.clockIn,
    photoData: 'base64data',
  );
  
  expect(result.isSuccess, true);
  expect(result.data?.id, '123');
});
```

#### 例外ケーステスト
- エラー状態や異常系での挙動を検証

```dart
test('ネットワークエラー時に適切なエラーを返す', () async {
  final mockApiClient = MockApiClient();
  when(mockApiClient.postAttendance(any)).thenThrow(DioException(
    requestOptions: RequestOptions(path: '/attendance'),
    error: 'Network error',
  ));
  
  final service = AttendanceService(apiClient: mockApiClient);
  final result = await service.recordAttendance(
    type: AttendanceType.clockIn,
    photoData: 'base64data',
  );
  
  expect(result.isSuccess, false);
  expect(result.error?.type, ErrorType.network);
});
```

## 3. ウィジェットテスト（Widget Tests）

### テスト技術スタック
- テストフレームワーク: **Flutter Test**
- ウィジェットテスト: **flutter_test パッケージ**
- UIインタラクション: **WidgetTester**
- スクリーンショット: **golden_toolkit**

### テスト観点

#### ウィジェット機能
- **インタラクション**: ボタンタップなどのユーザー操作に対する反応
- **状態変化**: ウィジェットの状態変更が正しく表示に反映されるか
- **入力処理**: フォーム入力とバリデーションが正しく機能するか

#### 視覚的整合性
- **レイアウト**: 要素の配置、サイズ、アライメントが設計通りか
- **テーマ適用**: カラー、フォント、スタイルが正しく適用されているか
- **レスポンシブ性**: 異なる画面サイズでの表示が適切か

### テスト手法

#### ウィジェット機能テスト
- ウィジェットをレンダリングし、操作とその結果を検証

```dart
testWidgets('ログインフォーム: IDとパスワードを入力し、ログインボタンを押せる', (WidgetTester tester) async {
  // ウィジェットのビルド
  await tester.pumpWidget(MaterialApp(home: LoginScreen()));
  
  // テキスト入力
  await tester.enterText(find.byType(TextField).at(0), 'employee123');
  await tester.enterText(find.byType(TextField).at(1), 'password123');
  
  // ボタンタップ
  await tester.tap(find.byType(ElevatedButton));
  await tester.pump();
  
  // 検証: ログイン処理が呼ばれたことを確認
  verify(mockAuthService.login('employee123', 'password123')).called(1);
});
```

#### ゴールデンテスト（スクリーンショット比較）
- レンダリングされたウィジェットを基準画像と比較し、視覚的一貫性を検証

```dart
testWidgets('ホーム画面のレイアウトが設計通りである', (WidgetTester tester) async {
  await tester.pumpWidget(MaterialApp(home: HomeScreen()));
  
  await expectLater(
    find.byType(HomeScreen),
    matchesGoldenFile('home_screen.png'),
  );
});
```

## 4. 統合テスト（Integration Tests）

### テスト技術スタック
- テストフレームワーク: **integration_test パッケージ**
- デバイスコントロール: **flutter_driver**
- 一貫性検証: **flutter_test**

### テスト観点

#### エンドツーエンドのワークフロー
- **主要ユースケース**: ログインから勤怠記録までの一連の流れ
- **データ永続化**: アプリ再起動後のデータ保持
- **バックグラウンド処理**: アプリがバックグラウンドに移行した際の挙動

#### プラットフォーム固有の機能
- **カメラアクセス**: 実機でのカメラ操作と写真撮影
- **バイオメトリクス**: 指紋/顔認証の統合
- **プッシュ通知**: 通知の送受信と表示

### テスト手法

#### シナリオベースのテスト
- 実際のユースケースに基づく一連の操作フローを検証

```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();
  
  testWidgets('勤怠記録の完全フロー: ログイン → 出勤打刻 → 履歴確認', (WidgetTester tester) async {
    // アプリの起動
    app.main();
    await tester.pumpAndSettle();
    
    // ログイン
    await tester.enterText(find.byKey(ValueKey('employee_id')), 'user123');
    await tester.enterText(find.byKey(ValueKey('password')), 'pass123');
    await tester.tap(find.byType(ElevatedButton));
    await tester.pumpAndSettle();
    
    // ホーム画面に移動したことを確認
    expect(find.text('こんにちは、User123さん'), findsOneWidget);
    
    // 出勤ボタンをタップ
    await tester.tap(find.byKey(ValueKey('clock_in_button')));
    await tester.pumpAndSettle();
    
    // カメラ画面が表示されたことを確認
    expect(find.byType(CameraPreview), findsOneWidget);
    
    // カメラ操作とアップロードは実機依存のため擬似的に処理
    // ...
    
    // 履歴画面に移動
    await tester.tap(find.byIcon(Icons.history));
    await tester.pumpAndSettle();
    
    // 今日の勤怠記録が表示されていることを確認
    expect(find.textContaining('出勤'), findsOneWidget);
  });
}
```

#### クロスプラットフォームテスト
- iOS、Androidの両方でテストを実行し、プラットフォーム固有の問題を検出

```bash
# CI/CDスクリプト例
flutter drive \
  --driver=test_driver/integration_test_driver.dart \
  --target=integration_test/app_test.dart \
  --flavor=dev \
  -d [device-id]
```

## 5. ユーザビリティテスト

### テスト技術スタック
- **Firebase App Distribution**: テスターへの配布
- **Firebase Crashlytics**: クラッシュレポート
- **Firebase Analytics**: ユーザー行動分析
- **In-App Feedback**: アプリ内フィードバック収集

### テスト観点

#### 実ユーザー体験
- **使いやすさ**: 直感的にアプリを操作できるか
- **エラー回復**: エラー発生時にユーザーが理解し対処できるか
- **満足度**: ユーザーがアプリに満足しているか

#### 実環境での動作
- **様々なデバイス**: 異なる画面サイズ、OSバージョンでの動作
- **ネットワーク状態**: 弱いネットワーク環境やオフライン状態での挙動
- **バッテリー消費**: 長時間使用した際のバッテリー消費量

### テスト手法

#### ベータテスト
- 社内ユーザーによる事前利用とフィードバック収集

```
1. Firebase App Distributionでベータ版をリリース
2. テスターに特定のタスクを実行してもらう
3. アプリ内フィードバック機能でコメントを収集
4. 収集したデータを分析し、改善点を特定
```

#### 実地テスト
- 実際の勤務環境での使用を想定したテスト

```
1. 様々なライティング条件下でのカメラ機能のテスト
2. オフィス環境でのネットワーク切替時の挙動確認
3. 一日中使用した際のバッテリー消費率の測定
```

## 6. CI/CDパイプラインへの統合

### テスト自動化フロー
- プルリクエスト作成時に自動テストを実行
- テスト失敗時はマージをブロック
- nightlyビルドで統合テストを実行

### CI/CDワークフロー例

```yaml
name: Flutter Test
on: [pull_request, push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.10.0'
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test --coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        
  build-ios:
    needs: test
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter build ios --no-codesign
        
  build-android:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
      - run: flutter build appbundle
```

## 7. テスト環境と本番環境の分離

### 環境構成
- **開発環境**: モックAPIサーバー、デバッグツール有効
- **ステージング環境**: 本番と同等のバックエンド、テストデータ
- **本番環境**: 実際のデータを使用

### フレーバー設定
Flutter環境分離のためのフレーバー設定と構成:

```dart
// lib/config/environment.dart
enum Environment {
  development,
  staging,
  production,
}

class AppConfig {
  final Environment environment;
  final String apiBaseUrl;
  final bool enableAnalytics;
  
  static AppConfig? _instance;
  
  factory AppConfig({
    required Environment environment,
    required String apiBaseUrl,
    required bool enableAnalytics,
  }) {
    _instance ??= AppConfig._internal(
      environment: environment,
      apiBaseUrl: apiBaseUrl,
      enableAnalytics: enableAnalytics,
    );
    return _instance!;
  }
  
  AppConfig._internal({
    required this.environment,
    required this.apiBaseUrl,
    required this.enableAnalytics,
  });
  
  static AppConfig get instance => _instance!;
  bool get isDevelopment => environment == Environment.development;
  bool get isProduction => environment == Environment.production;
}
```

## 8. テスト駆動開発(TDD)の実践

### TDDサイクル
- **Red**: 失敗するテストを書く
- **Green**: テストをパスするコードを最小限実装
- **Refactor**: コードをリファクタリング

### 具体例

```dart
// Step 1: Red - 失敗するテストを書く
test('勤怠記録サービス: 出勤時間と退勤時間から労働時間を計算できる', () {
  final clockIn = DateTime(2023, 4, 1, 9, 0);
  final clockOut = DateTime(2023, 4, 1, 18, 0);
  
  final workingHours = AttendanceCalculator.calculateWorkingHours(
    clockIn: clockIn,
    clockOut: clockOut,
  );
  
  expect(workingHours, 9.0); // 9時間
});

// Step 2: Green - 最小限の実装
class AttendanceCalculator {
  static double calculateWorkingHours({
    required DateTime clockIn,
    required DateTime clockOut,
  }) {
    return (clockOut.difference(clockIn).inMinutes / 60);
  }
}

// Step 3: Refactor - より堅牢な実装
class AttendanceCalculator {
  static double calculateWorkingHours({
    required DateTime clockIn,
    required DateTime clockOut,
  }) {
    if (clockOut.isBefore(clockIn)) {
      throw ArgumentError('退勤時間は出勤時間より後である必要があります');
    }
    
    final differenceInMinutes = clockOut.difference(clockIn).inMinutes;
    return double.parse((differenceInMinutes / 60).toStringAsFixed(2));
  }
}
```

この包括的なテスト計画により、高品質な勤怠管理モバイルアプリケーションを開発し、継続的に改善することが可能になります。 