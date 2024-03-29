# Titanium MLKit module

<b>Current features:</b>

-   QR/bacrode scanning (one shot or continuous)
-   OCR/Text recognition
-   Basic pose detection

The camera is displayed as a view so you can place it where you want and add other Titanium elements over it.

## How to get the module

This is a <b>paid module</b>. It will help to keep the module up-to-date and and maintain the compatibility with the latest Titanium SDK. If you are interested in the module contact me on [TiSlack](tislack.org/), [Twitter](twitter.com/michaelgangolf) or using my [contact form](https://www.migaweb.de/#contact).
For a demo check the <b>Release</b> section.

## Installation

-   tiapp.xml:

```xml
<android xmlns:android="http://schemas.android.com/apk/res/android">
	<manifest xmlns:android="http://schemas.android.com/apk/res/android" android:versionCode="1">
		<uses-feature android:name="android.hardware.camera.any"/>
		<uses-permission android:name="android.permission.CAMERA"/>
		<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
		<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
	</manifest>
</android>
```

```xml
<modules>
	<module>ti.mlkit</module>
</modules>
```

If you use it with Ti &lt; 12 add this to your build.gradle

    android {
        compileSdkVersion 33
    }

## constants

for **focusMode**:

-   TiMLKit.FOCUS_MODE_AUTO_FOCUS
-   TiMLKit.FOCUS_MODE_CONTINUOUS_AUTO_FOCUS
-   TiMLKit.FOCUS_MODE_LOCKED

for **torchMode**:

-   TiMLKit.TORCH_MODE_OFF
-   TiMLKit.TORCH_MODE_ON
-   TiMLKit.TORCH_MODE_AUTO

for **formats**:

-   TiMLKit.BARCODE_FORMAT_ALL (or empty)
-   TiMLKit.BARCODE_FORMAT_CODE_128
-   TiMLKit.BARCODE_FORMAT_CODE_39
-   TiMLKit.BARCODE_FORMAT_CODE_93
-   TiMLKit.BARCODE_FORMAT_CODA_BAR
-   TiMLKit.BARCODE_FORMAT_DATA_MATRIX
-   TiMLKit.BARCODE_FORMAT_EAN_13
-   TiMLKit.BARCODE_FORMAT_EAN_8
-   TiMLKit.BARCODE_FORMAT_ITF
-   TiMLKit.BARCODE_FORMAT_QR_CODE
-   TiMLKit.BARCODE_FORMAT_UPCA
-   TiMLKit.BARCODE_FORMAT_UPCE
-   TiMLKit.BARCODE_FORMAT_PDF_417
-   TiMLKit.BARCODE_FORMAT_AZTEC

for **mode**:

-   TiMLKit.SCAN_TEXT
-   TiMLKit.SCAN_BARCODE
-   TiMLKit.SCAN_POSE
-   TiMLKit.SCAN_BARCODE_TEXT

## parameters

-   **mode**: int (constants), default `SCAN_BARCODE`. Use `SCAN_BARCODE` or `SCAN_TEXT`
-   **analyzerWidth**: int, default `1920`. Width of the ImageAnalyser
-   **analyzerHeight**: int, default `1080`. Height of the ImageAnalyser
-   **focusMode**: constant from above (before opening the camera)
-   **torchMode**: constant from above
-   **formats**: Array with constants or nothing for all
-   **isFrontCameraActive**: boolean, default `false`. Set it to `true` for front camera
-   **oneShot**: boolean, default `false`. Set it to `true` to scan only one barcode
-   **captureFirst**: same as `oneShot`
-   **showOverlay**: boolean, default `false`. Set it to `true` to highlight the elements inside the camera view
-   **overlayColor**: color, default `white`. Color of the highlight
-   **overlayWidth**: color, default `4`. Border width of the highlight
-   **captureImage**: boolean, default `false`. Will attach the scanned image for barcode/qrcodes in the `scan` event
-   **scanCards**: boolean, default `false`. When mode is `SCAN_TEXT` it will return `cardNumber`,`cardExpirationMonth`,`cardExpirationYear`,`cardOwner`
-   **scanRegion**: object with {top,left,right,bottom}. When set it will only scan inside that region.
-   **showScanRegion**: boolean, default `false`. Set it before starting the camera to display the scanRegion in your view. Use it only for debugging!
-   **zoom**: boolean, default `false`. Enables pinch and zoom.
-   **zoomFactor**: float, Set the zoom value.
-   **minZoom**: float (getter), returns the min zoom level.
-   **maxZoom**: float (getter), returns the max zoom level.
-   **returnAllBarcodes**: boolean, default `false`. Will return all potential barcodes even if they cannot be decoded.

## methods

#### TiMLKit

-   **recognizeText({image:Blob, callback: function })**: text recognition on a local blob. Parameters are image (blob!) and a callback function. This will contain `text` as a return property
-   **recognizeCode({image:Blob, callback: function })**: qr/barcode recognition on a local blob. Parameters are image (blob!) and a callback function. This will contain `text` as a return property

#### CameraView

-   **start()**: start camera view
-   **stop()**: stop camera view
-   **pause()**: pause the barcode recognition, camera is still active

## events

-   **ready**: view is ready
-   **scan**:
    -   found barcodes and text; returns `result` as an array with `valueType, displayValue, rawValue, format, rect[centerX,centerY,left,top,width,height], image`.
    -   Using `SCAN_POSE` you will get elements like `leftArm` or `rightArm`
    -   if `scanCards` is enabled it will return `result` and `cardNumber`, `cardExpirationYear`, `cardExpirationMonth`, `cardOwner`.
-   **success**

## Analyze local image

```js
import TiMLKit from 'ti.mlkit';
const file = Ti.Filesystem.getFile(Ti.Filesystem.resourcesDirectory, "/images/test.png");
TiMLKit.recognizeText({
	image: file.read(),
	callback: function(e) {
		console.log(e.text);
	}
});
```

## Simple text scan example

```js
import TiMLKit from 'ti.mlkit';

const win = Ti.UI.createWindow();
const cameraView = TiMLKit.createCameraView({
	width: Ti.UI.FILL,
	top: 0,
	bottom: 200
});
const logView = Ti.UI.createTextArea({
	height: 200,
	left: 0,
	width: Ti.UI.FILL,
	bottom: 0,
	editable: false
});

cameraView.addEventListener('scan', event => {
	if (event.result) {
		for (var i = 0; i < event.result.length; ++i) {
			if (event.result[i].rawValue) {
				logView.value += event.result[i].rawValue + ' | ';
			}
		}
	}
});

cameraView.addEventListener('ready', function() {
	cameraView.focusMode = TiMLKit.FOCUS_MODE_CONTINUOUS_AUTO_FOCUS;
	cameraView.mode = TiMLKit.SCAN_TEXT;
	cameraView.analyzerWidth = 1024;
	cameraView.analyzerHeight = 768;
	cameraView.showOverlay = true;
});

win.add([logView, cameraView]);

win.addEventListener('open', function() {
	Ti.Media.requestCameraPermissions(function(event) {
		if (!event.success) {
			alert("No permissions!");
			return;
		}
		cameraView.start();
	})
});

win.addEventListener('close', function() {
	cameraView.stop();
});

win.open();
```

## Full example

```js
import TiMLKit from 'ti.mlkit';

var currentMode = 0;
var isOverlay = false;
var isCropMode = false;

const win = Ti.UI.createWindow();
const view_cam = Ti.UI.createView({
	top: 0,
	width: Ti.UI.FILL,
	bottom: 250
})
const cameraView = TiMLKit.createCameraView({
	width: Ti.UI.FILL,
	height: Ti.UI.FILL
});
const logView = Ti.UI.createTextArea({
	height: 200,
	left: 0,
	width: "50%",
	bottom: 50,
	editable: false
});
const btn_mode = Ti.UI.createButton({
	title: "switch mode",
	bottom: 0,
	left: 0
});
const btn_overlay = Ti.UI.createButton({
	title: "overlay",
	bottom: 0
});
const btn_crop = Ti.UI.createButton({
	title: "crop mode",
	right: 0,
	bottom: 0
});
const img = Ti.UI.createImageView({
	height: 200,
	bottom: 50,
	right: 0,
	width: "50%",
	scalingMode: Titanium.Media.IMAGE_SCALING_ASPECT_FIT
})
const view_overlay = Ti.UI.createView({
	top: 90,
	left: 90,
	right: 90,
	bottom: 90,
	borderWidth: 2,
	borderColor: "#f00",
	touchEnabled: false
})
view_cam.add(cameraView);
view_cam.add(view_overlay);

cameraView.addEventListener('scan', event => {

	// prefilled fields for scanCard = true
	if (event.cardNumber != null && event.cardOwner != null && event.cardExpirationMonth != null) {
		logView.value += "Card no: " + event.cardNumber + "\n" + "Owner: " + event.cardOwner + "\n" + "Exp. date: " + event.cardExpirationMonth + "/" + event.cardExpirationYear + "\n";
	}

	for (var i = 0; i < event.result.length; ++i) {
		logView.value += event.result[i].rawValue + ' | ';
	}
	if (event.result[0] && event.result[0].image) {
		img.image = event.result[0].image
	}
});

// Do the config once the camera is ready
cameraView.addEventListener('ready', function() {
	cameraView.focusMode = TiMLKit.FOCUS_MODE_CONTINUOUS_AUTO_FOCUS; // OR: FOCUS_MODE_AUTO_FOCUS
	cameraView.torchMode = TiMLKit.TORCH_MODE_OFF; // OR: TORCH_MODE_OFF
	cameraView.mode = TiMLKit.SCAN_BARCODE;
	cameraView.overlayColor = "#ff0";
	cameraView.captureImage = false;
	cameraView.analyzerWidth = 1280;
	cameraView.analyzerHeight = 720;
	cameraView.overlwayWidth = 50;
	cameraView.zoom = true;
	cameraView.showOverlay = true;

	// cameraView.showScanRegion = false;
	// cameraView.scanCards = false;
	cameraView.formats = [ // OR: TiMLKit.BARCODE_FORMAT_ALL
		TiMLKit.BARCODE_FORMAT_CODE_128,
		TiMLKit.BARCODE_FORMAT_CODE_39,
		TiMLKit.BARCODE_FORMAT_CODE_93,
		TiMLKit.BARCODE_FORMAT_CODA_BAR,
		TiMLKit.BARCODE_FORMAT_DATA_MATRIX,
		TiMLKit.BARCODE_FORMAT_EAN_13,
		TiMLKit.BARCODE_FORMAT_EAN_8,
		TiMLKit.BARCODE_FORMAT_ITF,
		TiMLKit.BARCODE_FORMAT_QR_CODE,
		TiMLKit.BARCODE_FORMAT_UPCA,
		TiMLKit.BARCODE_FORMAT_UPCE,
		TiMLKit.BARCODE_FORMAT_PDF_417,
		TiMLKit.BARCODE_FORMAT_AZTEC
	];

	// show an overlay - only items inside this region will be scanned
	//
	// Tip: make it a bit bigger as the region your are showing on screen.
	//      use cameraView.showScanRegion = true; to debug it
	cameraView.scanRegion = {
		top: view_overlay.top - 5,
		left: view_overlay.left - 5,
		right: view_overlay.right - 5,
		bottom: view_overlay.bottom - 5
	}
});

win.add([view_cam, logView, btn_mode, btn_crop, btn_overlay, img]);

btn_mode.addEventListener('click', function() {
	currentMode++;
	if (currentMode > 3) {
		currentMode = 0;
	}

	cameraView.mode = currentMode;
	if (currentMode == 0) {
		console.log("Barcode mode");
	} else if (currentMode == 1) {
		console.log("Text mode");
	} else if (currentMode == 2) {
		console.log("Pose mode");
	} else if (currentMode == 3) {
		console.log("Barcode/text mode");
	}
	cameraView.start();
});


btn_overlay.addEventListener('click', function() {
	isOverlay = !isOverlay;
	cameraView.showOverlay = isOverlay;
});

btn_crop.addEventListener('click', function() {
	isCropMode = !isCropMode;
	cameraView.captureImageRectOnly = isCropMode;
});

win.addEventListener('open', function() {
	Ti.Media.requestCameraPermissions(function(event) {
		if (!event.success) {
			alert("No permissions!");
			return;
		}

		cameraView.start();
	})
});

win.addEventListener('close', function() {
	cameraView.stop(); // Stop session
});

win.open();
```
