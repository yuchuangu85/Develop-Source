<h1 align="center">Kotlin library</h1>

[toc]

## 库

|                 | The Past--Android | The Past--iOS         | The Future--D-KMP     |
| --------------- | ----------------- | --------------------- | --------------------- |
| UI System       | Views/Fragments   | ViewControllers/UIKit | Compose & SwiftUI     |
| Observable      | LiveData          | Combine               | StateFlow             |
| HTTP Client     | Retrofit          | Alamofire             | Ktor Client           |
| Serialization   | Gson/Moshi        | Codable/JSONSerializ. | Kotlin Serialization  |
| Structured Data | Room              | CoreData              | SQLDelight            |
| Settings        | Sharepreferences  | NSUserDefaults        | MultiplatformSettings |
| Concurrency     | AsyncTask         | GrandCentralDispatch  | Coroutines            |
|                 |                   |                       |                       |
|                 |                   |                       |                       |



* [prof18/kmp-fatframework-cocoa: A Gradle plugin to generate and publish an iOs FatFramework or XCFramework on Kotlin Multiplatform projects. (github.com)](https://github.com/prof18/kmp-fatframework-cocoa)
* [**Ktor Http Client**](https://ktor.io/docs/http-client-multiplatform.html), developed by *JetBrains*. It’s the best KMP networking library, wrapping the native HTTP clients on each platform.
* [**Serialization**](https://github.com/Kotlin/kotlinx.serialization), developed by *JetBrains*. It provides a very simple way to serialize data. It’s typically used in conjunction with *Ktor Http Client*, to parse Json data.
* [**SqlDelight**](https://cashapp.github.io/sqldelight/), developed by *Square*. It provides multi-platform support to local SQLite databases.
* [**MultiPlatform Settings**](https://github.com/russhwolf/multiplatform-settings), developed by Russell Wolf (*TouchLab*). It’s wrapping *SharedPreferences* on Android, *NSUserDefaults* on iOS, *Storage* on Javascript.



