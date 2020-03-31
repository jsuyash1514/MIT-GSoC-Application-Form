# MIT-GSoC-Application-Form

Link to the Project Proposal - [here](https://drive.google.com/file/d/1B0TYxo-U5fylj7-3Tka_gOkniAlSfScO/view)

Link to Resume - [here](https://jsuyash1514.github.io/resume/)

Link to Long Answers - [here](https://jsuyash1514.github.io/MIT-GSoC-Application-Form/)

## Application Challenges

### 1.) Creating a Non Trivial App

Link to non trivial app in APK format - [here](https://drive.google.com/open?id=1tZbOhtBJJQH4nVormJ62qDD8OnLbXkQf)

Link to non trivial app in AIA format - [here](https://drive.google.com/open?id=1LeJESIGI5WCkwEgYKeitP7qRJJax5U7G)

### 2.) Designer Challenge

#### a.) Research on useFront Property of the Camera component

Using the camera of our device through an Android app can be done using 2 ways:-
* Start the existing Camera application via `Intent` and use the return data of the application to access the result.
* Directly integrate the camera into your application via the `Camera API`.

Let’s first explore both the ways, then we’ll have a look at the case with the App Inventor.

##### i.) Using Intent

```java
// Set a onClick listener on the button to open the camera
public void onclick( View v ){ 
  Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
  startActivityForResult(takePictureIntent,CAMERA_PIC_REQUEST_CODE); 
}
// Override the OnActivityResult as follows to set the recieved image in the image view.
protected void onActivityResult(int requestCode, int resultCode, Intent data) { 
  if (requestCode == CAMERA_PIC_REQUEST_CODE && resultCode == RESULT_OK) { 
    Bitmap imageData=(Bitmap)data.getExtras().get("data");
    imageView.setImageBitmap(imageData); 
  } 
}
```

For accessing the **front camera** we need to pass an extra parameter in the intent as shown below:
```java
takePictureIntent.putExtra("android.intent.extras.CAMERA_FACING", 1);
```

##### ii.) Using Camera API

This method allows the user to have a camera interface in the application. 

`Android.hardware.camera` - This package is the primary API for controlling device cameras. It can be used to take pictures or videos while making a Camera app.

To access the **front camera** we can pass the front camera ID in `Camera.open(int)` method. By default, `Camera.open()` method opens the primary (i.e back) camera of the device.

##### MIT App Inventor

In MIT App Inventor, we use the `Intent` method to access the device camera. The `useFront` property passes an extra parameter to the intent used for opening  the camera application. The following code patch can be found [here](https://github.com/mit-cml/appinventor-sources/blob/ucr/appinventor/components/src/com/google/appinventor/components/runtime/Camera.java#L170).
```java
if (useFront) {
  intent.putExtra("android.intent.extras.CAMERA_FACING", 1);
}
```
**Since this feature is undocumented, it has stopped working in nearly all the Android versions.** Hence the `useFront` property is now deprecated. We can rewrite the Camera component using the `Camera API`  provided by `Android.hardware.camera` package to make the component more customisable.

#### b.) Redesign the Camera component so that it takes picture automatically (without user intervention)

As mentioned above, the existing App Inventor uses the intent method to access the camera. This method doesn’t allow the user to automatically capture the image. However, we can achieve this functionality by implementing the Camera component using the `android.hardware.camera` Camera API to take the picture. 

For achieving this, I've modified the [TakePicture()](https://github.com/mit-cml/appinventor-sources/blob/ucr/appinventor/components/src/com/google/appinventor/components/runtime/Camera.java#L127) method as follows:-

```java
android.hardware.Camera camera = android.hardware.Camera.open();
SurfaceTexture surfaceTexture = new SurfaceTexture(0);
android.hardware.Camera.PictureCallback pictureCallback = new android.hardware.Camera.PictureCallback() {
        @Override
        public void onPictureTaken(byte[] data, android.hardware.Camera camera) {
          try {
            FileOutputStream fos = new FileOutputStream(image);
            fos.write(data);
            fos.close();
            camera.release();
          } catch (FileNotFoundException e) {
            Log.d("Camera", "File not found: " + e.getMessage());
            camera.release();
          } catch (IOException e) {
            Log.d("Camera", "Error accessing file: " + e.getMessage());
            camera.release();
          }
        }
      };
      try {
        camera.setPreviewTexture(surfaceTexture);
        camera.startPreview();
        camera.takePicture(null,null,null,pictureCallback);
      } catch (IOException e) {
        e.printStackTrace();
        Log.d("Camera",  e.getMessage());
      }
```

This allows user to automatically capture the image using the device camera and store it in the external storage. We can present this functionality before the users as they can now capture multiple pics at a given regular interval using the camera component. This functionality can be achieved as follows:

```java
@SimpleFunction
  public void TakeContinuousPicture(int gapInMillisec, int totalTime) {
    CountDownTimer timer = new CountDownTimer(totalTime,gapInMillisec) {
      @Override
      public void onTick(long millisUntilFinished) {
        Log.d("Cameratesting", "Picture request initiated");
          TakePicture();
      }
  
      @Override
      public void onFinish() {
        Log.d("Cameratesting", "Timer finished");
      }
    }.start();
  }
```
I created a new function in the [Camera.java class](https://github.com/mit-cml/appinventor-sources/blob/ucr/appinventor/components/src/com/google/appinventor/components/runtime/Camera.java) which takes the regular interval and the total time as a parameter. User can now simply use this function as shown below:

<img></img>

This block enables the user to click multiple pics at a regular time interval on a single button click.
