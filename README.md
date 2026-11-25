# video_thumbnail

This plugin generates image thumbnails from video files or URLs on both Android and iOS. It returns an image in memory or saves it as a file with customizable format, resolution, and quality.

  [![pub ver](https://img.shields.io/badge/pub-v0.5.6-blue)](https://pub.dev/packages/video_thumbnail)
  [![license](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/justsoft/)

![video-file](https://github.com/justsoft/video_thumbnail/blob/master/video_file.png?raw=true) ![video-url](https://github.com/justsoft/video_thumbnail/blob/master/video_url.png?raw=true)

## iOS dependency managers

This fork supports Swift Package Manager (SPM) and CocoaPods using the same
Objective-C sources. Both provide JPEG, PNG and WebP through the existing Dart
API, `VideoThumbnailPlugin` registration and MethodChannel.

Requirements: Flutter 3.41 or newer, Dart 3.11 or newer, and Xcode compatible with
your Flutter SDK. The SPM target requires iOS 13 or newer; the application's
minimum iOS version must also satisfy Flutter and its other plugins. The package
SDK constraints apply to CocoaPods consumers too.

Add this fork as a path dependency during local development:

```yaml
dependencies:
  video_thumbnail:
    path: /absolute/path/to/video_thumbnail
```

For SPM, run `flutter config --enable-swift-package-manager`, then `flutter pub get`
and `flutter build ios` in the consuming app. Flutter discovers
`ios/video_thumbnail/Package.swift`, integrates the `video-thumbnail` product and
generates the local `FlutterFramework` dependency. Do not add that package to
Xcode manually. An app whose other plugins all support SPM needs no Pods for this
plugin.

For CocoaPods, run `flutter config --no-enable-swift-package-manager`, then
`flutter pub get` and `flutter build ios`. If switching an app already migrated
to SPM, follow Flutter's [SPM removal instructions](https://docs.flutter.dev/packages-and-plugins/swift-package-manager/for-app-developers#how-to-turn-off-swift-package-manager)
first. CocoaPods continues to use `ios/video_thumbnail.podspec` and its `libwebp`
dependency. Flutter configuration commands change your global Flutter settings.

SPM uses the source-built `libwebp` product from
[SDWebImage/libwebp-Xcode 1.6.0](https://github.com/SDWebImage/libwebp-Xcode/tree/1.6.0).
Its public `webp` headers support the same encoder calls on iOS devices and
simulators. libwebp is distributed under the BSD 3-Clause license; retain its
[license notice](https://github.com/SDWebImage/libwebp-Xcode/blob/1.6.0/LICENSE)
in binary distributions. CocoaPods resolves libwebp independently in the app's
`Podfile.lock`; the SPM version is pinned by this fork's manifest.

The checked-in example predates these SDK requirements. For current SDK testing,
use an isolated copy with a regenerated iOS host and updated example-only UI
dependencies, or a minimal host calling this package's Dart API. Keep SPM and
CocoaPods validation hosts separate to avoid cached Pods masking SPM problems.

## Methods
|function|parameter|description|return|
|--|--|--|--|
|thumbnailData|String `[video]`, optional Map<String, dynamic> `[headers]`, ImageFormat `[imageFormat]`(JPEG/PNG/WEBP), int `[maxHeight]`(0: for the original resolution of the video, or scaled by the source aspect ratio), [maxWidth]`(0: for the original resolution of the video, or scaled by the source aspect ratio), int `[timeMs]` generates the thumbnail from the frame around the specified millisecond, int `[quality]`(0-100)|generates thumbnail from `[video]`|`[Future<Uint8List>]`|
|thumbnailFile|String `[video]`, optional Map<String, dynamic> `[headers]`, String `[thumbnailPath]`(folder or full path where to store the thumbnail file, null to save to same folder as the video file), ImageFormat `[imageFormat]`(JPEG/PNG/WEBP), int `[maxHeight]`(0: for the original resolution of the video, or scaled by the source aspect ratio), int `[maxWidth]`(0: for the original resolution of the video, or scaled by the source aspect ratio), int `[timeMs]` generates the thumbnail from the frame around the specified millisecond, int `[quality]`(0-100)|creates a file of the thumbnail from the `[video]` |`[Future<String>]`|

Warning:
> Giving both the `maxHeight` and `maxWidth` has different result on Android platform, it actually scales the thumbnail to the specified maxHeight and maxWidth.
> To generate the thumbnail from a network resource, the `video` must be properly URL encoded.

## Usage

**Installing**
add [video_thumbnail](https://pub.dev/packages/video_thumbnail) as a dependency in your pubspec.yaml file.
```yaml
dependencies:
  video_thumbnail: ^0.5.6
```
**import**
```dart
import 'package:video_thumbnail/video_thumbnail.dart';
```
**Generate a thumbnail in memory from video file**
```dart
final uint8list = await VideoThumbnail.thumbnailData(
  video: videofile.path,
  imageFormat: ImageFormat.JPEG,
  maxWidth: 128, // specify the width of the thumbnail, let the height auto-scaled to keep the source aspect ratio
  quality: 25,
);
```

**Generate a thumbnail file from video URL**
```dart
final fileName = await VideoThumbnail.thumbnailFile(
  video: "https://flutter.github.io/assets-for-api-docs/assets/videos/butterfly.mp4",
  thumbnailPath: (await getTemporaryDirectory()).path,
  imageFormat: ImageFormat.WEBP,
  maxHeight: 64, // specify the height of the thumbnail, let the width auto-scaled to keep the source aspect ratio
  quality: 75,
);
```

**Generate a thumbnail file from video Assets declared in pubspec.yaml**
```dart
final byteData = await rootBundle.load("assets/my_video.mp4");
Directory tempDir = await getTemporaryDirectory();

File tempVideo = File("${tempDir.path}/assets/my_video.mp4")
  ..createSync(recursive: true)
  ..writeAsBytesSync(byteData.buffer.asUint8List(byteData.offsetInBytes, byteData.lengthInBytes));

final fileName = await VideoThumbnail.thumbnailFile(
  video: tempVideo.path,
  thumbnailPath: (await getTemporaryDirectory()).path,
  imageFormat: ImageFormat.PNG,  
  quality: 100,
);
```

## Notes
Fork or pull requests are always welcome. Currently it seems have a little performance issue while generating WebP thumbnail by using libwebp under iOS.
