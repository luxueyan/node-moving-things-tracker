# node-moving-things-tracker

node-moving-things-tracker is a javascript implementation of a _"tracker by detections"_ for realtime multiple object tracking (MOT) in node.js / browsers. 

Commissioned by moovel lab for [Beat the Traffic X](http://beatthetraffic.moovellab.com/) and the [Open Data Cam](http://opendatacam.moovellab.com/) project.

## Problem

How to track persistently multiple moving things from frame-by-frame object detections inputs? How to assign an unique identifier to frame-by-frame object detection results?

Often object detection framework don't have any memory of their detection results over time e.g. Yolo provides every frame an array of detections results in the form of `[[x,y,w,h,confidence,name] ...]`, note that there isn't any unique ID to indentify the same detected object in future frames.

**Detections Input**

Raw Yolo detection results

![detections](https://user-images.githubusercontent.com/533590/33817459-030e3822-de40-11e7-979d-0c8071ea2a94.gif)

**Tracker Output**

Yolo detection results enhanced with unique IDs after beeing processed by node-moving-things-tracker. Note that now every object has been assigned an unique ID

![tracker](https://user-images.githubusercontent.com/533590/33817550-6913047c-de40-11e7-8552-f284c738f1c0.gif)

## Installation

```bash
# Install globaly to use as command line tool
npm install -g node-moving-things-tracker 

# Install localy your node.js / javascript project
npm install --save node-moving-things-tracker 
```

## Usage

**As a node module**

```javascript
const Tracker = require('node-moving-things-tracker').Tracker;

Tracker.updateTrackedItemsWithNewFrame(detectionScaledOfThisFrame, currentFrame);

const trackerDataForThisFrame = Tracker.getJSONOfTrackedItems();
```

Example: [`Opendatacam.js`](https://github.com/opendatacam/opendatacam/blob/master/server/Opendatacam.js#L228) of the Open Data Cam project.

**Command line usage**

node-moving-things-tracker takes as an input a txt file generated by [node-yolo](https://github.com/moovel/node-yolo) and outputs a `tracker.json` file that assigns unique IDs to the YOLO detections bbox.

The detections entry file could also be generated by another object detection algorithm than YOLO, it just needs to respects the [same format](https://github.com/tdurand/node-moving-things-tracker#detections-input-and-tracker-output-format).

> NOTE : usage is customized for the use case of [Beat the Traffic X](https://beatthetraffic.moovellab.com) 

```bash
node-moving-things-tracker --input PATH_TO_YOLO_DETECTIONS.txt
# This will output a tracker.json file in the same folder containing the tracker data
```

## Detections Input and tracker output format

See example [here](https://github.com/tdurand/node-moving-things-tracker/tree/master/example): 

**Coordinate space:**

![coordinates of tracker input](https://user-images.githubusercontent.com/533590/35881673-60f4e8f8-0b60-11e8-837c-b2b8ec3bff67.jpg)

**Detections Input**

rawdetections.txt

```json
{"frame":0,"detections":[{"x":699,"y":99,"w":32,"h":19,"confidence":34,"name":"car"},{"x":285,"y":170,"w":40,"h":32,"confidence":26,"name":"car"},{"x":259,"y":178,"w":75,"h":46,"confidence":42,"name":"car"},{"x":39,"y":222,"w":91,"h":52,"confidence":61,"name":"car"},{"x":148,"y":199,"w":123,"h":55,"confidence":53,"name":"car"}]}
{"frame":1,"detections":[{"x":699,"y":99,"w":32,"h":19,"confidence":31,"name":"car"},{"x":694,"y":116,"w":34,"h":23,"confidence":25,"name":"car"},{"x":285,"y":170,"w":40,"h":32,"confidence":27,"name":"car"},{"x":259,"y":178,"w":75,"h":46,"confidence":42,"name":"car"},{"x":39,"y":222,"w":91,"h":52,"confidence":61,"name":"car"},{"x":148,"y":199,"w":123,"h":55,"confidence":52,"name":"car"}]}
```

**Tracker Output**

_Normal mode:_

```javascript
{
  // Tracker data for each frame
  "43": [
    {      
      "id": 0,
      "x": 628,
      "y": 144,
      "w": 48,
      "h": 29,
      "confidence": 80,
      "name": "car",
      "isZombie": false
    },
    {
      "id": 1,
      "x": 620,
      "y": 154,
      "w": 50,
      "h": 35,
      "confidence": 80,
      "name": "car",
      "isZombie": true
    }
  ]
}
```

_Debug mode:_

```bash
#Run with ---debug flag at the end
node-moving-things-tracker --debug --input PATH_TO_YOLO_DETECTIONS.txt
```

```javascript
{
  // Tracker data for each frame
  "43": [
    {
      "id": "900e36a2-cbc7-427c-83a9-819d072391f0",
      "idDisplay": 0,
      "x": 628,
      "y": 144,
      "w": 48,
      "h": 29,
      "confidence": 80,
      "name": "car",
      "isZombie": false,
      "zombieOpacity": 1,
      "appearFrame": 35,
      "disappearFrame": null
    },
    {
      "id": "38939c38-c977-40a9-ad6a-3bb916c37fa1",
      "idDisplay": 1,
      "x": 620,
      "y": 154,
      "w": 50,
      "h": 35,
      "confidence": 80,
      "name": "car",
      "isZombie": false,
      "zombieOpacity": 1,
      "appearFrame": 43,
      "disappearFrame": null
    }
  ]
}
```

## Limitations

No params tweaking is possible via command-line for now, it is currently optimized for tracking cars in traffic videos.

## License

[MIT License](LICENSE)


## Acknowledgments

node-moving-things-tracker is based on the ideas and work of the following people. References are listed chronologicaly how we encounter them.

- [OpenCV Matching Faces Over Time](http://shiffman.net/general/2011/04/26/opencv-matching-faces-over-time/) by Daniel Shiffman
- [Filter for interupting values](https://discourse.vvvv.org/t/filter-for-interupting-values/15601) related thread in the VVVV forum with source code and approaches. Pointer via [Joreg](https://github.com/joreg) 
- [IOU Tracker](https://github.com/bochinski/iou-tracker/) by Erik Bochinski, Volker Eiselein and Thomas Sikora, based on this [paper](elvera.nue.tu-berlin.de/files/1517Bochinski2017.pdf).
- [SORT Tracker](https://github.com/abewley/sort) by Alex Bewley, based on this [paper](https://arxiv.org/abs/1602.00763).
