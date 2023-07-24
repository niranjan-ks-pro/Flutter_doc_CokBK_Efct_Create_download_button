# Flutter_doc_CokBK_Efct_Create_download_button
 https://docs.flutter.dev/cookbook/effects/download-button

Create a download button
========================

1.  [Cookbook](https://docs.flutter.dev/cookbook)
2.  [Effects](https://docs.flutter.dev/cookbook/effects)
3.  [Create a download button](https://docs.flutter.dev/cookbook/effects/download-button)

Apps are filled with buttons that execute long-running behaviors. For example, a button might trigger a download, which starts a download process, receives data over time, and then provides access to the downloaded asset. It's helpful to show the user the progress of a long-running process, and the button itself is a good place to provide this feedback. In this recipe, you'll build a download button that transitions through multiple visual states, based on the status of an app download.

The following animation shows the app's behavior:

![The download button cycles through its stages](https://docs.flutter.dev/assets/images/docs/cookbook/effects/DownloadButton.gif)

[](https://docs.flutter.dev/cookbook/effects/download-button#define-a-new-stateless-widget)Define a new stateless widget
------------------------------------------------------------------------------------------------------------------------

Your button widget needs to change its appearance over time. Therefore, you need to implement your button with a custom stateless widget.

Define a new stateless widget called `DownloadButton`.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    // TODO:
    return const SizedBox();
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#define-the-buttons-possible-visual-states)Define the button's possible visual states
-------------------------------------------------------------------------------------------------------------------------------------------------

The download button's visual presentation is based on a given download status. Define the possible states of the download, and then update `DownloadButton` to accept a `DownloadStatus` and a `Duration` for how long the button should take to animate from one status to another.

content_copy

```
enum DownloadStatus {
  notDownloaded,
  fetchingDownload,
  downloading,
  downloaded,
}

@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.transitionDuration = const Duration(
      milliseconds: 500,
    ),
  });

  final DownloadStatus status;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    // TODO: We'll add more to this later.
    return const SizedBox();
  }
}
```

info Note: Each time you define a custom widget, you must decide whether all relevant information is provided to that widget from its parent or if that widget orchestrates the application behavior within itself. For example, `DownloadButton` could receive the current `DownloadStatus` from its parent, or the `DownloadButton` could orchestrate the download process itself within its `State` object. For most widgets, the best answer is to pass the relevant information into the widget from its parent, rather than manage behavior within the widget. By passing in all the relevant information, you ensure greater reusability for the widget, easier testing, and easier changes to application behavior in the future.

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-button-shape)Display the button shape
--------------------------------------------------------------------------------------------------------------

The download button changes its shape based on the download status. The button displays a grey, rounded rectangle during the `notDownloaded` and `downloaded` states. The button displays a transparent circle during the `fetchingDownload` and `downloading` states.

Based on the current `DownloadStatus`, build an `AnimatedContainer` with a `ShapeDecoration` that displays a rounded rectangle or a circle.

Consider defining the shape's widget tree in a separated `Stateless` widget so that the main `build()` method remains simple, allowing for the additions that follow. Instead of creating a function to return a widget, like `Widget _buildSomething() {}`, always prefer creating a `StatelessWidget` or a `StatefulWidget` which is more performant. More considerations on this can be found in the [documentation](https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html) or in a dedicated video in the Flutter [YouTube channel](https://www.youtube.com/watch?v=IOyq-eTRhvo).

For now, the `AnimatedContainer` child is just a `SizedBox` because we will come back at it in another step.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.transitionDuration = const Duration(
      milliseconds: 500,
    ),
  });

  final DownloadStatus status;
  final Duration transitionDuration;

  bool get _isDownloading => status == DownloadStatus.downloading;

  bool get _isFetching => status == DownloadStatus.fetchingDownload;

  bool get _isDownloaded => status == DownloadStatus.downloaded;

  @override
  Widget build(BuildContext context) {
    return ButtonShapeWidget(
      transitionDuration: transitionDuration,
      isDownloaded: _isDownloaded,
      isDownloading: _isDownloading,
      isFetching: _isFetching,
    );
  }
}

@immutable
class ButtonShapeWidget extends StatelessWidget {
  const ButtonShapeWidget({
    super.key,
    required this.isDownloading,
    required this.isDownloaded,
    required this.isFetching,
    required this.transitionDuration,
  });

  final bool isDownloading;
  final bool isDownloaded;
  final bool isFetching;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    var shape = const ShapeDecoration(
      shape: StadiumBorder(),
      color: CupertinoColors.lightBackgroundGray,
    );

    if (isDownloading || isFetching) {
      shape = ShapeDecoration(
        shape: const CircleBorder(),
        color: Colors.white.withOpacity(0),
      );
    }

    return AnimatedContainer(
      duration: transitionDuration,
      curve: Curves.ease,
      width: double.infinity,
      decoration: shape,
      child: const SizedBox(),
    );
  }
}
```

You might wonder why you need a `ShapeDecoration` widget for a transparent circle, given that it's invisible. The purpose of the invisible circle is to orchestrate the desired animation. The `AnimatedContainer` begins with a rounded rectangle. When the `DownloadStatus` changes to `fetchingDownload`, the `AnimatedContainer` needs to animate from a rounded rectangle to a circle, and then fade out as the animation takes place. The only way to implement this animation is to define both the beginning shape of a rounded rectangle and the ending shape of a circle. But, you don't want the final circle to be visible, so you make it transparent, which causes an animated fade-out.

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-button-text)Display the button text
------------------------------------------------------------------------------------------------------------

The `DownloadButton` displays `GET` during the `notDownloaded` phase, `OPEN` during the `downloaded` phase, and no text in between.

Add widgets to display text during each download phase, and animate the text's opacity in between. Add the text widget tree as a child of the `AnimatedContainer` in the button wrapper widget.

content_copy

```
@immutable
class ButtonShapeWidget extends StatelessWidget {
  const ButtonShapeWidget({
    super.key,
    required this.isDownloading,
    required this.isDownloaded,
    required this.isFetching,
    required this.transitionDuration,
  });

  final bool isDownloading;
  final bool isDownloaded;
  final bool isFetching;
  final Duration transitionDuration;

  @override
  Widget build(BuildContext context) {
    var shape = const ShapeDecoration(
      shape: StadiumBorder(),
      color: CupertinoColors.lightBackgroundGray,
    );

    if (isDownloading || isFetching) {
      shape = ShapeDecoration(
        shape: const CircleBorder(),
        color: Colors.white.withOpacity(0),
      );
    }

    return AnimatedContainer(
      duration: transitionDuration,
      curve: Curves.ease,
      width: double.infinity,
      decoration: shape,
      child: Padding(
        padding: const EdgeInsets.symmetric(vertical: 6),
        child: AnimatedOpacity(
          duration: transitionDuration,
          opacity: isDownloading || isFetching ? 0.0 : 1.0,
          curve: Curves.ease,
          child: Text(
            isDownloaded ? 'OPEN' : 'GET',
            textAlign: TextAlign.center,
            style: Theme.of(context).textTheme.labelLarge?.copyWith(
                  fontWeight: FontWeight.bold,
                  color: CupertinoColors.activeBlue,
                ),
          ),
        ),
      ),
    );
  }
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#display-a-spinner-while-fetching-download)Display a spinner while fetching download
------------------------------------------------------------------------------------------------------------------------------------------------

During the `fetchingDownload` phase, the `DownloadButton` displays a radial spinner. This spinner fades in from the `notDownloaded` phase and fades out to the `fetchingDownload` phase.

Implement a radial spinner that sits on top of the button shape and fades in and out at the appropriate times.

We have removed the `ButtonShapeWidget`'s constructor to keep the focus on its build method and the `Stack` widget we've added.

content_copy

```
@override
Widget build(BuildContext context) {
  return GestureDetector(
    onTap: _onPressed,
    child: Stack(
      children: [
        ButtonShapeWidget(
          transitionDuration: transitionDuration,
          isDownloaded: _isDownloaded,
          isDownloading: _isDownloading,
          isFetching: _isFetching,
        ),
        Positioned.fill(
          child: AnimatedOpacity(
            duration: transitionDuration,
            opacity: _isDownloading || _isFetching ? 1.0 : 0.0,
            curve: Curves.ease,
            child: ProgressIndicatorWidget(
              downloadProgress: downloadProgress,
              isDownloading: _isDownloading,
              isFetching: _isFetching,
            ),
          ),
        ),
      ],
    ),
  );
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#display-the-progress-and-a-stop-button-while-downloading)Display the progress and a stop button while downloading
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

After the `fetchingDownload` phase is the `downloading` phase. During the `downloading` phase, the `DownloadButton` replaces the radial progress spinner with a growing radial progress bar. The `DownloadButton` also displays a stop button icon so that the user can cancel an in-progress download.

Add a progress property to the `DownloadButton` widget, and then update the progress display to switch to a radial progress bar during the `downloading` phase.

Next, add a stop button icon at the center of the radial progress bar.

content_copy

```
@override
Widget build(BuildContext context) {
  return GestureDetector(
    onTap: _onPressed,
    child: Stack(
      children: [
        ButtonShapeWidget(
          transitionDuration: transitionDuration,
          isDownloaded: _isDownloaded,
          isDownloading: _isDownloading,
          isFetching: _isFetching,
        ),
        Positioned.fill(
          child: AnimatedOpacity(
            duration: transitionDuration,
            opacity: _isDownloading || _isFetching ? 1.0 : 0.0,
            curve: Curves.ease,
            child: Stack(
              alignment: Alignment.center,
              children: [
                ProgressIndicatorWidget(
                  downloadProgress: downloadProgress,
                  isDownloading: _isDownloading,
                  isFetching: _isFetching,
                ),
                if (_isDownloading)
                  const Icon(
                    Icons.stop,
                    size: 14.0,
                    color: CupertinoColors.activeBlue,
                  ),
              ],
            ),
          ),
        ),
      ],
    ),
  );
}
```

[](https://docs.flutter.dev/cookbook/effects/download-button#add-button-tap-callbacks)Add button tap callbacks
--------------------------------------------------------------------------------------------------------------

The last detail that your `DownloadButton` needs is the button behavior. The button must do things when the user taps it.

Add widget properties for callbacks to start a download, cancel a download, and open a download.

Finally, wrap `DownloadButton`'s existing widget tree with a `GestureDetector` widget, and forward the tap event to the corresponding callback property.

content_copy

```
@immutable
class DownloadButton extends StatelessWidget {
  const DownloadButton({
    super.key,
    required this.status,
    this.downloadProgress = 0,
    required this.onDownload,
    required this.onCancel,
    required this.onOpen,
    this.transitionDuration = const Duration(milliseconds: 500),
  });

  final DownloadStatus status;
  final double downloadProgress;
  final VoidCallback onDownload;
  final VoidCallback onCancel;
  final VoidCallback onOpen;
  final Duration transitionDuration;

  bool get _isDownloading => status == DownloadStatus.downloading;

  bool get _isFetching => status == DownloadStatus.fetchingDownload;

  bool get _isDownloaded => status == DownloadStatus.downloaded;

  void _onPressed() {
    switch (status) {
      case DownloadStatus.notDownloaded:
        onDownload();
        break;
      case DownloadStatus.fetchingDownload:
        // do nothing.
        break;
      case DownloadStatus.downloading:
        onCancel();
        break;
      case DownloadStatus.downloaded:
        onOpen();
        break;
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _onPressed,
      child: const Stack(
        children: [
          /* ButtonShapeWidget and progress indicator */
        ],
      ),
    );
  }
}
```

Congratulations! You have a button that changes its display depending on which phase the button is in: not downloaded, fetching download, downloading, and downloaded. Now, the user can tap to start a download, tap to cancel an in-progress download, and tap to open a completed download.
