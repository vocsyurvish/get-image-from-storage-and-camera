# get-image-from-storage-and-camera

<Details>
  <Summary> View xml side: </Summary>
  
  #### Add Permission to `manifest.xml`.

```xml
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.CAMERA" />
```

#### For Camera add features to `manifest.xml`

```xml
    <uses-feature android:name="android.hardware.camera.autofocus" />
  
    <uses-feature android:name="android.hardware.camera"
        android:required="true" />
```

#### add for android 10

```xml
        android:requestLegacyExternalStorage="true"
```

#### Add provider to `manifest.xml`

```xml
    <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.provider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths" />
    </provider>
```


#### Create xml file as a`provider_paths.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
  <paths>
    <external-path
        name="external"
        path="." />
    <external-files-path
        name="external_files"
        path="." />
    <cache-path
        name="cache"
        path="." />
    <external-cache-path
        name="external_cache"
        path="." />
    <files-path
        name="files"
        path="." />
</paths>
```

### Here you have finish code of all xml
  
</Details>

<Details>
  <Summary> Setup Permission Code: </Summary>
  
  #### Create Global variables.
  
  ```java
    private int mRequestCode = 0;
    private String mPermission = "";
    
    // Permission handler...
    private ActivityResultLauncher<String> requestPermissionLauncher =
            registerForActivityResult(new ActivityResultContracts.RequestPermission(), isGranted -> {
                if (isGranted) {
                    afterPermission(); // create method in your activity
                } else {
                    Toast.makeText(this, "Please allow permission", Toast.LENGTH_SHORT).show();
                }
            });
  ```
  
  #### Include get permission method
  
  ```java
  public void startPermission(Activity activity, String permission, int mRequestCode) {
        this.mRequestCode = mRequestCode;
        this.mPermission = permission;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (ContextCompat.checkSelfPermission(activity, permission) == PackageManager.PERMISSION_GRANTED) {
                afterPermission();
            } else if (activity.shouldShowRequestPermissionRationale(permission)) {
                AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setTitle("Permission");
                builder.setMessage("You need to grant the " + permission + " permission for use our application");
                builder.setCancelable(false);
                builder.setPositiveButton("I UNDERSTAND", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        dialogInterface.dismiss();
                        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                        Uri uri = Uri.fromParts("package", getPackageName(), null);
                        intent.setData(uri);
                        startActivity(intent);
                    }
                });
                builder.show();
            } else {
                requestPermissionLauncher.launch(permission);
            }
        }
    }
  ```
  #### Create `afterPermission()`
  
  ```java
  private void afterPermission() {
    // after you get success permission in your application..
    // write your logic here...
  }
  ```
  
</Details>

<Details>
  <Summary> How to get Image from Gallery & Camera: </Summary>
  
  #### on Gallery & Camera button...
  
  ```java
  // get permission first...
  
  GALLERY_BUTTON.setOnClickListener((view -> {
            startPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE, 101);
        }));
        
  CAMMERA_BUTTON.setOnClickListener((view -> {
            startPermission(MainActivity.this, Manifest.permission.CAMERA, 102);
        }));
  
  ```
  
  #### in `afterPermission()` method...
  
  ```java
  private void afterPermission() {
        switch (mRequestCode) {
            case 101:
                pickFromGallery(501);
                break;
            case 102:
                startPermission(MainActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE, 103);
                break;
            case 103:
                chooseFromCamera(502);
                break;
        }
    }
  
  ```
  
  #### init Global Variables...
  
  ```java
    private Bitmap bitmap = null;
    private File photoFile = null;
  ```
  
  #### Setup Methods...
  
  ```java
   private void pickFromGallery(int requestCode) {
        Intent intent = new Intent();
        intent.setType("image/*");
        intent.setAction("android.intent.action.PICK");
        startActivityForResult(Intent.createChooser(intent, "Select Picture from"), requestCode);
    }
    
    private void chooseFromCamera(int requestCode) {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        try {
            photoFile = createImageFile();
            if (photoFile != null) {
                Uri photoURI = FileProvider.getUriForFile(this, BuildConfig.APPLICATION_ID + ".provider", photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoURI);
                startActivityForResult(takePictureIntent, requestCode);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    private File createImageFile() throws IOException {
        long timeStamp = Calendar.getInstance().getTimeInMillis();
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(
                imageFileName,  /* prefix */
                ".jpg",   /* suffix */
                storageDir      /* directory */
        );
        return image;
    }
    
  ```
  
  #### Get Bitmap Result...
  
  ```java
  
  @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if (resultCode == RESULT_OK) {
            switch (requestCode) {
                case 501:
                    if (data != null) {
                        try {
                            bitmap = MediaStore.Images.Media.getBitmap(this.getContentResolver(), data.getData());
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                    break;
                case 502:
                    if (photoFile != null) {
                        bitmap = BitmapFactory.decodeFile(photoFile.getAbsolutePath());
                    }
                    break;
            }

            if (bitmap != null) {
                // you got bitmap here
            }
        }
        super.onActivityResult(requestCode, resultCode, data);
    }
  
  ```
  
</Details>

<Details>
  <Summary> import classes and variables: </Summary>
 

#### you must be have import this classes in your activity...



```java

import android.Manifest;
import android.app.Activity;
import android.content.DialogInterface;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.os.Environment;
import android.provider.MediaStore;
import android.provider.Settings;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;
import androidx.core.content.FileProvider;

import com.example.vocsy.background.service.BuildConfig;
import com.example.vocsy.background.service.R;

import java.io.File;
import java.io.IOException;
import java.util.Calendar;

```

</Details>
