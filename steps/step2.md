---
layout: post
title: Rationale - Handling Android Permission.
excerpt: "Android Permission grant access to User's data. Sensitive data. Most times, user's aren't exactly sure why your application need access to a particular permission. Today, I will be sharing how to reduce the risk of having constant denial of permission by users and how you can implement important techiniques found in apps like WhatApp by using Rationale."
tag: [android,java]
modified: 2017-01-08
comments: true
---

Android Permission grant access to User's data. Sensitive data. Most times, user's aren't exactly sure why your application need access to a particular permission. Today, I will be sharing how to reduce the risk of having constant denial of permission by users and how you can implement important techiniques found in apps like WhatApp by using <a href="https://github.com/KingsMentor/Rationale" target="_blank"> Rationale</a>.

**At the end of the lesson, you should be able to :** <br>
1. Handle Permission request better. <br>
2. Integrate and use <a href="https://github.com/KingsMentor/Rationale" target="_blank"> Rationale</a>
{: .notice}

### Content Outline
1. The Permission Flow
2. Introduction Rationale
3. Using Rationale
4. Wrap Up

#### The Flow
Android 6.0 Marshmallow introduced a new permissions model that lets apps request permissions from the user at runtime, rather than prior to installation. Apps that support the new model request permissions when the app actually requires the services or data protected by the services. While this doesn't (necessarily) change overall app behavior, it does create a few changes relevant to the way sensitive user data is handled. This introduces a Flow Similar to :


##### Components of the Flow:

##### 1. Check the Paltform :
Runtime permission is only applicable for API Level 23 (Marshmallow) and above. Other versions used install time permission. Check the platform to determine if runtime permission request is required.

``` java
// check if permission requires runtime request
private boolean needsPermissionRequest(String permission) {

        PermissionDetails permissionDetails = new PermissionDetails().getPermissionDetails(this, permission, -1);
        boolean needsPermissionRequest = permissionDetails.getProtectionLevel() != PermissionInfo.PROTECTION_NORMAL;
        return needsPermissionRequest && deviceNeedsPermission();

    }
// check the platform level . Only API level >= Version M requires runtime permission
private boolean deviceNeedsPermission() {
        return Build.VERSION.SDK_INT >= Build.VERSION_CODES.M;
    }
```

##### 2. Check the Permission:
Check if the permission has been granted access.

``` java

private boolean isPermissionGranted(String permission) {
        return (ContextCompat.checkSelfPermission(this, permission)
                == PackageManager.PERMISSION_GRANTED);
    }

```

##### 3. Explain the permission :
Here, the reason for requesting the permission can be conveyed to the user. This is a delicate step. The user can deny the permission if the reason is not satisfying. If the permission is requested for the first time, the user is presented with a dialog to either accept or deny. On subsequent prompt, a check box is added to permanenently deny the permission. If the user deny the permission with the box checked,subsequent request for same permission will not be honoured by the android system. You will have to launch the settings screen for the user to turn the permission on.

```java

// check of permission is permanently denied
private boolean isPermissionPermanentlyDenied(String permission) {
        return !ActivityCompat.shouldShowRequestPermissionRationale(this, permission);
    }
// load the permission setting screen
 private void loadPermissionPage(Activity context) {
        Intent intent = new Intent();
        intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        Uri uri = Uri.fromParts("package", context.getPackageName(), null);
        intent.setData(uri);
        context.startActivityForResult(intent, 0);
    }

```

Taking a look at **Explain the permission** , I haven't really seen a well implemented approach than what was implemented on WhatsApp. The WhatsApp approached is quiet simple.

1. Show permission dialog containing information on why the request should be granted.
2. If the permission was prevoiusly denied, the dialog provide a way to launch the permission settings screen and it also detect if the permission has been granted `onResume`. This means that method call dependent on permission can continue.



##### Introducing Rationale
<a href="https://github.com/KingsMentor/Rationale" target="_blank"> Rationale</a> is an android library that helps manage permission request.  With Rationale, the permission flow can be reduced to :

All it requires is a list of permission passed to it,it then handles the entire interaction with the users, and return list of permission to be requested for. 

**It takes away all the pain of hanlding permission, letting you know what it to be requested**

##### Why You should consider this approach
1. A better experience for the use.
2. Less headache in managing permission messages

<p class="pic"><img src="http://share.gifyoutube.com/g5K0Y9.gif" alt="Drawing" /></p>




# Using Rationale 

**Step 1** Add it in your root build.gradle at the end of repositories:

```gradle

allprojects {
        repositories {
            ...
            maven { url 'https://jitpack.io' }
        }
    }

```

**Step 2** Add the dependency

```gradle
dependencies {
            compile 'com.github.KingsMentor:Rationale:v1.0'
    }
```

#### Implementing Permission with Rationale



```java
 private final int PERM = 2017;

// when using an Activity
final PermissionDetails smsPermissionDetails = new PermissionDetails().getPermissionDetails(this, Manifest.permission.READ_SMS, R.drawable.ic_sms_white_24dp);
        Rationale.withActivity(this)
                .requestCode(PERM)
                .addSmoothPermission(new SmoothPermission(smsPermissionDetails))
                .includeStyle(R.style.Beliv_RationaleStyle).build(true);

// when using a fragment
        Rationale.withFragment(this)
                .requestCode(PERM)
                .addSmoothPermission(new SmoothPermission(smsPermissionDetails))
                .includeStyle(R.style.Beliv_RationaleStyle).build(true);
```

#### Retrieving Response from Rationale

Rationale returns permission through `onActivityResult`

``` java
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (Rationale.isResultFromRationale(requestCode, PERM)) {
            RationaleResponse rationaleResponse = Rationale.getRationaleResponse(data);
            if (rationaleResponse.shouldRequestForPermissions()) {
                // a list of permission to be requested for
                ArrayList<SmoothPermission> smoothPermissions = rationaleResponse.getSmoothPermissions();

                // request for permissions
            } else if (rationaleResponse.userDecline()) {
                Toast.makeText(this, "user does not want to do this now", Toast.LENGTH_LONG).show();
            } else {
                Toast.makeText(this, "permission is accepted", Toast.LENGTH_LONG).show();
            }
        }

    }
```

#### Wraping Up
More details about using **Rationale** and **How to Contribute** can be found on <a href="https://github.com/KingsMentor/Rationale" target="_blank"> the github repo</a><br>
{: .notice}


Yea! This wasn't boring afterall. You should probably leave a comment. <br>Thanks for reading.<br>


