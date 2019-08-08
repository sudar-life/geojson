# Geojson

[![pub package](https://img.shields.io/pub/v/geojson.svg)](https://pub.dartlang.org/packages/geojson) [![Build Status](https://travis-ci.org/synw/geojson.svg?branch=master)](https://travis-ci.org/synw/geojson) [![Coverage Status](https://coveralls.io/repos/github/synw/geojson/badge.svg?branch=master)](https://coveralls.io/github/synw/geojson?branch=master)

Utilities to work with geojson data in Dart. Features:

- **Parser**: simple functions are available to parse geojson
- **Reactive api**: streams are available to retrieve the geojson features as soon as they are parsed

Note: the data is parsed in an isolate to avoid slowing down the main thread

## Simple functions

**[featuresFromGeoJson](https://pub.dev/documentation/geojson/latest/geojson/featuresFromGeoJson.html)**: get a [FeaturesCollection](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonFeatureCollection-class.html) from geojson string data. Parameters:

- `data`: a string with the geojson data, required
- `nameProperty`: the property used for the geoserie name, automaticaly set if null
- `verbose`: print the parsed data if true

**[featuresFromGeoJsonFile](https://pub.dev/documentation/geojson/latest/geojson/featuresFromGeoJsonFile.html)**: get a [FeaturesCollection](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonFeatureCollection-class.html) from a geojson file. Parameters:

- `file`: the file to load, required
- `nameProperty`: the property used for the geoserie name, automaticaly set if null
- `verbose`: print the parsed data if true

## Reactive api

Typed streams are available to retrieve the features as soon as they are parsed. Example: add assets on a Flutter map:

```dart
  import 'dart:math' as math;
  import 'package:pedantic/pedantic.dart';
  import 'package:flutter/services.dart' show rootBundle;
  import 'package:geojson/geojson.dart';
  import 'package:flutter_map/flutter_map.dart';

  /// Data for the Flutter map polylines layer
  final lines = <Polyline>[];

  Future<void> parseAndDrawAssetsOnMap() async {
    final data = await rootBundle
        .loadString('assets/railroads_of_north_america.geojson');
    final geojson = GeoJson();
    geojson.processedLines.listen((GeoJsonLine line) {
      final color = Color((math.Random().nextDouble() * 0xFFFFFF).toInt() << 0)
          .withOpacity(0.3);
      setState(() => lines.add(Polyline(
          strokeWidth: 2.0, color: color, points: line.geoSerie.toLatLng())));
    });
    geojson.endSignal.listen((_) => geojson.dispose());
    unawaited(geojson.parse(data, verbose: true));
  }
```

Check the examples for more details

### Available streams

- `processedFeatures`: the parsed features: all the geometries
- `processedPoints`: the parsed points
- `processedMultipoints`: the parsed multipoints
- `processedLines`: the parsed lines
- `processedMultilines`: the parsed multilines
- `processedPolygons`: the parsed polygons
- `processedMultipolygons`: the parsed multipolygons
- `endSignal`: parsing is finished indicator

## Supported geojson features

All the data structures use [GeoPoint](https://pub.dev/documentation/geopoint/latest/geopoint/GeoPoint-class.html) and [GeoSerie](https://pub.dev/documentation/geopoint/latest/geopoint/GeoSerie-class.html) from the [GeoPoint](https://github.com/synw/geopoint) package to store the geometry data. Data structures used:

**[GeoJsonFeatureCollection](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonFeatureCollection-class.html)**:

- `String` **name**
- `List<GeoJsonFeature>` **collection**

**[GeoJsonFeature](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonFeature-class.html)**:

- `GeoJsonFeatureType` **type**: [types](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonFeatureType-class.html)

- `Map<String, dynamic>` **properties**: the json properties of the feature
- `dynamic` **geometry**: the geometry data, depends on the feature type, see below

**[GeoJsonPoint](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonPoint-class.html)**:

- `String` **name**
- `GeoPoint` **geoPoint**: the geometry data

**[GeoJsonMultiPoint](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonMultiPoint-class.html)**:

- `String` **name**
- `GeoSerie` **geoSerie**: the geometry data: this will produce a geoSerie of type `GeoSerieType.group`

**[GeoJsonLine](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonLine-class.html)**:

- `String` **name**
- `GeoSerie` **geoSerie**: the geometry data: this will produce a geoSerie of type `GeoSerieType.line`

**[GeoJsonMultiLine](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonMultiLine-class.html)**:

- `String` **name**
- `List<GeoJsonLine>` **lines**

**[GeoJsonPolygon](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonPolygon-class.html)**:

- `String` **name**
- `List<GeoSerie>` **geoSeries**: the geometry data: this will produce a list of geoSerie of type `GeoSerieType.polygon`*

**[GeoJsonMultiPolygon](https://pub.dev/documentation/geojson/latest/geojson/GeoJsonMultiPolygon-class.html)**:

- `String` **name**
- `List<GeoJsonPolygon>` **polygons**

Note: none of the parameters is final for all of these data structures
