import 'dart:async';
import 'dart:io';
import 'package:flutter/material.dart';
import 'package:path_provider/path_provider.dart';
import 'package:youtube_explode_dart/youtube_explode_dart.dart' as yt;

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'YouTube Audio Downloader',
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
  String _link = '';
  String _destinationFolder = '';
  String _status = '';
  double _progressValue = 0.0;

  void _chooseFolder() async {
    Directory directory = await getExternalStorageDirectory();
    setState(() {
      _destinationFolder = directory.path;
      _status = "Pasta de destino selecionada: $_destinationFolder";
    });
  }

  Future<void> _downloadAudio() async {
    if (_link.isEmpty) {
      setState(() {
        _status = "Por favor, insira um link válido.";
      });
      return;
    }

    if (_destinationFolder.isEmpty) {
      setState(() {
        _status = "Por favor, escolha uma pasta de destino.";
      });
      return;
    }

    final yt.YoutubeExplode ytExplode = yt.YoutubeExplode();
    final yt.Video video = await ytExplode.videos.get(_link);
    final yt.AudioStreamInfo audio = video.audio.first;
    final File file = File('$_destinationFolder/${video.title}.wav');

    await for (final List<int> chunk in ytExplode.videos.streamsClient.get(audio)) {
      file.writeAsBytesSync(chunk, mode: FileMode.append);
    }

    setState(() {
      _status = "Download completo: ${file.path}";
    });

    await ytExplode.close();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('YouTube Audio Downloader'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: <Widget>[
            TextField(
              onChanged: (value) {
                setState(() {
                  _link = value;
                });
              },
              decoration: InputDecoration(labelText: 'Link do vídeo'),
            ),
            SizedBox(height: 16.0),
            RaisedButton(
              onPressed: _chooseFolder,
              child: Text('Escolher Pasta'),
            ),
            SizedBox(height: 16.0),
            RaisedButton(
              onPressed: _downloadAudio,
              child: Text('Baixar Áudio'),
            ),
            SizedBox(height: 16.0),
            LinearProgressIndicator(value: _progressValue),
            SizedBox(height: 16.0),
            Text(_status),
          ],
        ),
      ),
    );
  }
}
