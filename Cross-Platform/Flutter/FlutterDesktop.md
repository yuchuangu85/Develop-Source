<h1 align="center">Flutter Desktop开发</h1>



## 技术名词

* sidebar
* treeview



## 参考资料

* [Desktop support for Flutter | Flutter](https://docs.flutter.dev/desktop)
* https://github.com/bdlukaa/fluent_ui
* https://github.com/jpnurmi/awesome-flutter-linux
* https://github.com/leanflutter/awesome-flutter-desktop
* https://github.com/Solido/awesome-flutter
* https://github.com/bitsdojo/bitsdojo_window
* https://github.com/PotatoProject/Leaflet
* https://github.com/AppFlowy-IO/appflowy
* [purocean/yn: A Hackable Markdown Note Application for Programmers. Documents encryption, code snippet running, integrated terminal, chart embedding, HTML widgets, plug-in, and macro replacement. (github.com)](https://github.com/purocean/yn)
* https://stackoverflow.com/questions/60924384/creating-resizable-view-that-resizes-when-pinch-or-drag-from-corners-and-sides-i
* https://github.com/GroovinChip/macos_ui
* https://github.com/GroovinChip/macos_ui/blob/dev/lib/src/layout/resizable_pane.dart
* https://pub.dev/packages/flutter_sidebar
* https://pub.dev/packages/macos_ui
* https://pub.dev/packages/tree_view
* https://pub.dev/packages/expandable_tree_menu
* [filesystem_picker | Flutter Package (pub.dev)](https://pub.dev/packages/filesystem_picker)



## 基础命令

```bash
// set all platforms config
flutter config --enable-windows-desktop
flutter config --enable-macos-desktop
flutter config --enable-linux-desktop
flutter config --enable-windows-uwp-desktop
 
// check devices
flutter devices

// create app
flutter create myapp

// run the app
flutter run -d windows
flutter run -d macos
flutter run -d linux
lutter run -d winuwp

// Build a release app
flutter build windows
flutter build macos
flutter build linux

// Add desktop support to an existing Flutter app
flutter create --platforms=windows,macos,linux .

```

