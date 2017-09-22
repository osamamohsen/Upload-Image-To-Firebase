# Upload-Image-To-Firebase

## Add your Permission

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

## Add your Firbase Service 

### create Firebase project and copy your google.service file in app

### go to firebase >> storage >> Rules

      allow read, write: if true;

### build.gradle (project)
    dependencies {
        classpath 'com.google.gms:google-services:3.1.0' 
        // NOTE: Do not place your application dependencies here; they belong
        
    and your maven file
    
        repositories {
        jcenter()
        maven {
            url "https://maven.google.com" // Google's Maven repository
        }

    }
      
### build.gradle (Module)

    compile 'com.google.firebase:firebase-core:11.4.0'
    compile 'com.google.firebase:firebase-storage:11.4.0'
    compile 'com.google.firebase:firebase-auth:11.4.0'

    compile 'com.squareup.picasso:picasso:2.5.2'



## XML File

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="osama.uploadimagefirebase.MainActivity">

    <LinearLayout
        android:id="@+id/layoutButton"
        android:orientation="horizontal"
        android:layout_alignParentTop="true"
        android:weightSum="2"
        android:padding="16dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnChoose"
            android:text="Choose"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="wrap_content" />

        <Button
            android:id="@+id/btnUpload"
            android:text="Upload"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="wrap_content" />

    </LinearLayout>

    <ImageView
        android:id="@+id/imgView"
        android:layout_below="@+id/layoutButton"
        android:layout_width="match_parent"
        android:layout_height="200dp" />

    <ImageView
        android:id="@+id/imgFirebase"
        android:layout_below="@+id/imgView"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</RelativeLayout>


## MainActivity

 private Button btnChoose,btnUpload;
    private ImageView imageView;
    private ImageView imgFirebase;
    private Uri filePath;
    private final int PICK_IMAGE_REQUEST = 71;


#### Firebase
    FirebaseStorage storage;
    StorageReference storageReference;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btnChoose = (Button) findViewById(R.id.btnChoose);
        btnUpload = (Button) findViewById(R.id.btnUpload);
        imageView = (ImageView) findViewById(R.id.imgView);
        imgFirebase = (ImageView) findViewById(R.id.imgFirebase);


#### Firebase INIT
        storage = FirebaseStorage.getInstance();
        storageReference = storage.getReference();

        btnUpload.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                chooseImage();
            }
        });

        btnChoose.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                uploadImage();
            }
        });
    }
    
#### Upload image To Firebase

   private void uploadImage() {  
        if(filePath != null){
            final ProgressDialog progress = new ProgressDialog(this);
            progress.setTitle("Uploading....");
            progress.show();

            StorageReference ref= storageReference.child("images/"+ UUID.randomUUID().toString());
            ref.putFile(filePath).addOnSuccessListener(new OnSuccessListener<UploadTask.TaskSnapshot>() {
                @Override
                public void onSuccess(UploadTask.TaskSnapshot taskSnapshot) {
                    progress.dismiss();
                    Toast.makeText(MainActivity.this, "Uploaded successfully", Toast.LENGTH_SHORT).show();
                    String imageUrl = taskSnapshot.getDownloadUrl().toString(); /* Fetch url image */
                    Picasso.with(getBaseContext()).load(imageUrl).into(imgFirebase); 
                    /* use picasso to fetch url and display image */
                }
            }).addOnFailureListener(new OnFailureListener() {
                @Override
                public void onFailure(@NonNull Exception e) {
                    progress.dismiss();
                    Toast.makeText(MainActivity.this, "Failed "+e.getMessage(), Toast.LENGTH_SHORT).show();
                }
            }).addOnProgressListener(new OnProgressListener<UploadTask.TaskSnapshot>() {
                @Override
                public void onProgress(UploadTask.TaskSnapshot taskSnapshot) {
                    double progres_time = (100.0*taskSnapshot.getBytesTransferred()/taskSnapshot.getTotalByteCount());
                    progress.setMessage("Uploaded "+(int)progres_time+" %");
                }
            });
        }
    }

#### Fetch image from gallery
    private void chooseImage() {
        Intent intent = new Intent();
        intent.setType("image/*");
        intent.setAction(Intent.ACTION_GET_CONTENT);
        startActivityForResult(intent.createChooser(intent,"Select Picture"),PICK_IMAGE_REQUEST);
    }

#### on select image
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == PICK_IMAGE_REQUEST && resultCode == Activity.RESULT_OK &&
                data != null && data.getData() != null)
        {
            filePath = data.getData();
            try
            {
                Bitmap bitmap = MediaStore.Images.Media.getBitmap(getContentResolver(), filePath);
                imageView.setImageBitmap(bitmap);
            } catch (IOException e)
            {
                e.printStackTrace();
            }
        }
    }
