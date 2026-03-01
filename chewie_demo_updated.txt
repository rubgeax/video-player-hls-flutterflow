import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:video_player/video_player.dart';
import 'package:chewie/chewie.dart';
import 'package:visibility_detector/visibility_detector.dart';
import 'package:wakelock_plus/wakelock_plus.dart';

class ChewieDemo extends StatefulWidget {
  ChewieDemo({
    Key? key,
    this.width,
    this.height,
    this.videoPath =
        "https://5ca9af4645e15.streamlock.net/lobodurango/videolobodurango/playlist.m3u8",
    this.autoPlay = true,
    this.looping = true,
    this.allowFullScreen = true,
  }) : super(key: key);

  final double? width;
  final double? height;
  final String videoPath;
  final bool autoPlay;
  final bool looping;
  final bool allowFullScreen;

  @override
  _ChewieDemoState createState() => _ChewieDemoState();
}

class _ChewieDemoState extends State<ChewieDemo> {
  VideoPlayerController? _videoPlayerController;
  ChewieController? _chewieController;

  @override
  void initState() {
    super.initState();
    initializePlayer();
  }

  @override
  void dispose() {
    WakelockPlus.disable();
    _videoPlayerController?.dispose();
    _chewieController?.dispose();
    super.dispose();
  }

  Future<void> initializePlayer() async {
    _videoPlayerController =
        VideoPlayerController.networkUrl(Uri.parse(widget.videoPath))
          ..initialize().then((_) {
            setState(() {});
            _createChewieController();
            WakelockPlus.enable();
          });
  }

  void _createChewieController() {
    _chewieController = ChewieController(
      videoPlayerController: _videoPlayerController!,
      autoPlay: widget.autoPlay,
      looping: widget.looping,
      allowFullScreen: widget.allowFullScreen,
      fullScreenByDefault: false,
      isLive: true,
      showControls: true,
      showOptions: false,
      allowMuting: true,
      allowPlaybackSpeedChanging: false,
      placeholder: Container(
        color: Colors.black,
      ),
      customControls: _LiveControls(),
      deviceOrientationsAfterFullScreen: [DeviceOrientation.portraitUp],
      deviceOrientationsOnEnterFullScreen: [
        DeviceOrientation.landscapeRight,
        DeviceOrientation.landscapeLeft,
      ],
    );

    _chewieController!.addListener(() {
      if (!_chewieController!.isFullScreen) {
        SystemChrome.setPreferredOrientations([DeviceOrientation.portraitUp]);
      }
    });
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      width: widget.width,
      height: widget.height,
      color: Colors.black,
      child: VisibilityDetector(
        key: const Key('chewie-video-key'),
        onVisibilityChanged: (VisibilityInfo info) {
          if (_chewieController == null) return;
          if (info.visibleFraction == 0 &&
              !_chewieController!.isFullScreen) {
            _videoPlayerController?.pause();
            WakelockPlus.disable();
          } else {
            WakelockPlus.enable();
          }
        },
        child: _chewieController != null &&
                _chewieController!
                    .videoPlayerController.value.isInitialized
            ? Chewie(controller: _chewieController!)
            : const Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    CircularProgressIndicator(
                      color: Colors.white,
                    ),
                    SizedBox(height: 20),
                    Text(
                      'Cargando...',
                      style: TextStyle(
                        color: Colors.white,
                        fontSize: 16,
                      ),
                    ),
                  ],
                ),
              ),
      ),
    );
  }
}

class _LiveControls extends StatefulWidget {
  @override
  _LiveControlsState createState() => _LiveControlsState();
}

class _LiveControlsState extends State<_LiveControls> {
  bool _showControls = true;
  bool _isMuted = false;
  late ChewieController _chewieController;
  late VideoPlayerController _videoController;

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    _chewieController = ChewieController.of(context);
    _videoController = _chewieController.videoPlayerController;
    _videoController.addListener(_onVideoChanged);
  }

  @override
  void dispose() {
    _videoController.removeListener(_onVideoChanged);
    super.dispose();
  }

  void _onVideoChanged() {
    if (mounted) setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    final chewieController = _chewieController;
    final videoController = _videoController;

    return GestureDetector(
      onTap: () {
        setState(() {
          _showControls = !_showControls;
        });
      },
      child: AnimatedOpacity(
        opacity: _showControls ? 1.0 : 0.0,
        duration: const Duration(milliseconds: 300),
        child: Container(
          color: Colors.transparent,
          child: Stack(
            children: [
              // Play/Pause centrado
              Center(
                child: GestureDetector(
                  onTap: () {
                    if (!_showControls) return;
                    setState(() {
                      if (videoController.value.isPlaying) {
                        videoController.pause();
                      } else {
                        videoController.play();
                      }
                    });
                  },
                  child: Container(
                    decoration: BoxDecoration(
                      color: Colors.black45,
                      shape: BoxShape.circle,
                    ),
                    padding: const EdgeInsets.all(12),
                    child: Icon(
                      videoController.value.isPlaying
                          ? Icons.pause
                          : Icons.play_arrow,
                      color: Colors.white,
                      size: 42,
                    ),
                  ),
                ),
              ),
              // Barra inferior: LIVE | espacio | mute | fullscreen
              Positioned(
                left: 0,
                right: 0,
                bottom: 0,
                child: Container(
                  decoration: const BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.bottomCenter,
                      end: Alignment.topCenter,
                      colors: [Colors.black54, Colors.transparent],
                    ),
                  ),
                  padding:
                      const EdgeInsets.symmetric(horizontal: 12, vertical: 8),
                  child: Row(
                    children: [
                      // LIVE badge
                      Container(
                        padding: const EdgeInsets.symmetric(
                            horizontal: 8, vertical: 4),
                        decoration: BoxDecoration(
                          color: Colors.red,
                          borderRadius: BorderRadius.circular(4),
                        ),
                        child: const Text(
                          'LIVE',
                          style: TextStyle(
                            color: Colors.white,
                            fontSize: 12,
                            fontWeight: FontWeight.bold,
                          ),
                        ),
                      ),
                      const Spacer(),
                      // Mute button
                      GestureDetector(
                        onTap: () {
                          if (!_showControls) return;
                          setState(() {
                            _isMuted = !_isMuted;
                            videoController.setVolume(_isMuted ? 0 : 1);
                          });
                        },
                        child: Icon(
                          _isMuted ? Icons.volume_off : Icons.volume_up,
                          color: Colors.white,
                          size: 24,
                        ),
                      ),
                      const SizedBox(width: 16),
                      // Fullscreen button
                      GestureDetector(
                        onTap: () {
                          if (!_showControls) return;
                          chewieController.toggleFullScreen();
                        },
                        child: Icon(
                          chewieController.isFullScreen
                              ? Icons.fullscreen_exit
                              : Icons.fullscreen,
                          color: Colors.white,
                          size: 24,
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
