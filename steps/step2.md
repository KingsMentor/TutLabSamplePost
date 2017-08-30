
#### Article Content

<ol type ="1">
  <li>Perequisite</li>
  <li>Setting Up Volley</li>
  <li>Asynchronous Volley
    <ol type ="1">
    <li>Handling responses in Asynchronous Volley</li>
    <li>Issues With Asynchronous Volley</li>
    </ol>
  </li>
  <li>Synchronous Volley
    <ol type ="1">
    <li>When to make Synchronous Calls</li>
    <li>Handling responses in Synchronous Volley</li>
    </ol>
  </li>
  <li>Conclusion</li>
</ol>

**At the end of the lesson, you should be able to** : Effortlessly use volley to make synchronous and asynchronous call. <br>
{: .notice}

### Perequisite

To be able to grasp the content of the article, it is advisable to have prior knowledge of:

* Android Development - Beginner Level
* Making Network Calls in Android

You see, it is does not require a `unicorn horn` !  So, let's get started 

### Setting Up Volley

There are a lot of post about how to set up volley. I will have to cover just the basics to get started.

**Step 1 - Add Dependency to gradle**

```gradle
compile 'com.android.volley:volley:1.0.0'
``` 

**Step 2 - Use Request Queue**

All requests in Volley are placed in a queue first and then processed, here is how you will be creating a request queue:

```java
RequestQueue mRequestQueue = Volley.newRequestQueue(this); // 'this' is Context
```

Don't forget to add `internet permission`

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

Ideally you should have one centralized place for your Queue, and the best place to initialize queue is in your Application class. Here is how this can be done:

```java
public class ApplicationController extends Application {

    /**
     * Log or request TAG
     */
    public static final String TAG = "VolleyPatterns";

    /**
     * Global request queue for Volley
     */
    private RequestQueue mRequestQueue;

    /**
     * A singleton instance of the application class for easy access in other places
     */
    private static ApplicationController sInstance;

    @Override
    public void onCreate() {
        super.onCreate();

        // initialize the singleton
        sInstance = this;
    }

    /**
     * @return ApplicationController singleton instance
     */
    public static synchronized ApplicationController getInstance() {
        return sInstance;
    }

    /**
     * @return The Volley Request queue, the queue will be created if it is null
     */
    public RequestQueue getRequestQueue() {
        // lazy initialize the request queue, the queue instance will be
        // created when it is accessed for the first time
        if (mRequestQueue == null) {
            mRequestQueue = Volley.newRequestQueue(getApplicationContext());
        }

        return mRequestQueue;
    }

    /**
     * Adds the specified request to the global queue, if tag is specified
     * then it is used else Default TAG is used.
     * 
     * @param req
     * @param tag
     */
    public <T> void addToRequestQueue(Request<T> req, String tag) {
        // set the default tag if tag is empty
        req.setTag(TextUtils.isEmpty(tag) ? TAG : tag);

        VolleyLog.d("Adding request to queue: %s", req.getUrl());

        getRequestQueue().add(req);
    }

    /**
     * Adds the specified request to the global queue using the Default TAG.
     * 
     * @param req
     * @param tag
     */
    public <T> void addToRequestQueue(Request<T> req) {
        // set the default tag if tag is empty
        req.setTag(TAG);

        getRequestQueue().add(req);
    }

    /**
     * Cancels all pending requests by the specified TAG, it is important
     * to specify a TAG so that the pending/ongoing requests can be cancelled.
     * 
     * @param tag
     */
    public void cancelPendingRequests(Object tag) {
        if (mRequestQueue != null) {
            mRequestQueue.cancelAll(tag);
        }
    }
}
```


### Asynchronous Volley

Volley provides the following utility classes which you can use to make asynchronous HTTP requests:

<a href="https://android.googlesource.com/platform/frameworks/volley/+/43950676303ff68b23a8b469d6a534ccd1e08cfc/src/com/android/volley/toolbox/JsonObjectRequest.java" target="_blank">JsonObjectRequest</a> — To send and receive JSON Object from the Server

<a href="https://android.googlesource.com/platform/frameworks/volley/+/43950676303ff68b23a8b469d6a534ccd1e08cfc/src/com/android/volley/toolbox/JsonArrayRequest.java" target="_blank">JsonArrayRequest</a> — To receive JSON Array from the Server

<a href="https://android.googlesource.com/platform/frameworks/volley/+/43950676303ff68b23a8b469d6a534ccd1e08cfc/src/com/android/volley/toolbox/StringRequest.java" target="_blank">StringRequest</a> — To retrieve response body as String (ideally if you intend to parse the response by yourself)

Note: To send parameters in request body you need to override either getParams() or getBody() method of the request classes (as required) described below.
{: .notice}

### Handling Response from Asynchronous Volley
Since Volley already handles it's call on the background, it means that there is no wait on the call. You can continue to do other things while the network call is being made. To continue operation on data returned from volley asynchronous call, it needs to be done on the callback. Here are ways you can achieve that:

* **Using Volley Callback** 

```java

    public void loadJsonObject(String url, String keyTrack, int action, JSONObject jsonRequest, final Map<String, String> params, Response.Listener<JSONObject> volleyCallbackResponse, Response.ErrorListener volleyErrorResponse) {


        JsonObjectRequest request = new JsonObjectRequest(action, url, jsonRequest, volleyCallbackResponse, volleyErrorResponse) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };
        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());
    }
```

* **Using CustomCallBack**

```java

    public interface ResponseCallBack{
        void onResponse(Object response);
        void error(Object errorObj);
    }

    public void loadJsonObject(String url, String keyTrack, int action, JSONObject jsonRequest, final Map<String, String> params, final ResponseCallBack responseCallBack) {


        JsonObjectRequest request = new JsonObjectRequest(action, url, jsonRequest, new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                responseCallBack.onResponse(response);
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                responseCallBack.error(error);
            }
        }) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };
        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT)); // for retry policy
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());
    }
```

### Issues with Asynchronous Volley

When volley returns response, it does so on the UI thread - even when you call from a Service. . The implication of this is that operations done inside the callback will happen on the UI thread. Well, this wouldn't bother you unless in some scenerios.
Let's Assume 

1. you want to pull some data from the server
2. Process the data 
3. Save the data in database
4. Reflect update on the UI

If you decide to handle retrieving of data with an asychronous volley, since volley returns response on UI Thread, making database operation on the UI thread will affect performance.

I know what you are thinking. You will handle the database operation using AsyncTask. Well, that is what you tried avoiding in the first place. Isn't it better to just handles everything using Asyntask then ? 

Asychronous Volley has its great numerous advantages. But it should be worthy of note that there are cases were it can't do much for you. One will expect for volley to return response to same thread it was called. It doesn't work that way. Any operation done on the callback happens on the UI threas, thereby affecting performance. I learned the hard way.

### Sychronous Volley

Issues with Asynchronous Volley made me search for ways I handle Volley synchronously. Volley hanldes synchronous call using a feature call Future.
With `Future` - more like a `promise` , you can make a request and wait for the response. This wait happens synchronously.


### When to make Synchronous Volley Calls

Well, these are scenerios I considered neccessary to make synchronous calls

1. Using Volley in a Service
2. There is a need to get the data before further operations


# Handling Response in Synchronous Volley

Here is a snippet of how I make Synchronous call in volley

```java

    public VolleyResponse<JSONObject> loadFutureJsonObject(String url, String keyTrack, int action, JSONObject jsonRequest, final Map<String, String> params) {

        RequestFuture<JSONObject> future = RequestFuture.newFuture();

        JsonObjectRequest request = new JsonObjectRequest(action, url, jsonRequest, future, future) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };

        request.setRetryPolicy(new DefaultRetryPolicy(600000, 15, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());

        try {
            JSONObject response = future.get(2, TimeUnit.MINUTES);
            return new VolleyResponse<>(true, response);
        } catch (InterruptedException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        } catch (ExecutionException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        } catch (TimeoutException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        }

    }
```

The request was added to the RequestQueue before calling `future.get`
{: .notice}

The good side about using Future is that you can decide how long to wait. If the time elapses and a result is not returned yet, there is a 
`TimeoutException`

### Conclusion

You can also use same technique to make <a href="https://developer.android.com/training/volley/request-custom.html" target="_blank">Custom Volley Request 
</a>.

Contributions to this post is welcome. By the way, here is a `VolleyHelper` that I use to make  Volley Calls less tedious. I keep updating it over time. 

```java
import com.android.volley.AuthFailureError;
import com.android.volley.DefaultRetryPolicy;
import com.android.volley.Response;
import com.android.volley.toolbox.JsonArrayRequest;
import com.android.volley.toolbox.JsonObjectRequest;
import com.android.volley.toolbox.RequestFuture;
import com.android.volley.toolbox.StringRequest;
import com.appzonegroup.zone.zonedata.ApplicationController;
import com.appzonegroup.zone.zonedata.utils.Utilities;

import org.json.JSONArray;
import org.json.JSONObject;

import java.util.Calendar;
import java.util.Map;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.TimeoutException;


public class VolleyHelper {



    public void loadJsonObject(String url, String keyTrack, int action, JSONObject jsonRequest, final Map<String, String> params, Response.Listener<JSONObject> volleyCallbackResponse, Response.ErrorListener volleyErrorResponse) {


        JsonObjectRequest request = new JsonObjectRequest(action, url, jsonRequest, volleyCallbackResponse, volleyErrorResponse) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };
        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());
    }


    public VolleyResponse<JSONObject> loadFutureJsonObject(String url, String keyTrack, int action, JSONObject jsonRequest, final Map<String, String> params) {

        RequestFuture<JSONObject> future = RequestFuture.newFuture();

        JsonObjectRequest request = new JsonObjectRequest(action, url, jsonRequest, future, future) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };

        request.setRetryPolicy(new DefaultRetryPolicy(600000, 15, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());

        try {
            JSONObject response = future.get(2, TimeUnit.MINUTES);
            return new VolleyResponse<>(true, response);
        } catch (InterruptedException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        } catch (ExecutionException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        } catch (TimeoutException e) {
            // exception handling
            return buildErrorMessage(e.getMessage());
        }

    }

    public VolleyHelper.VolleyResponse<JSONArray> loadFutureJsonArray(String url, String keyTrack, int action, JSONArray jsonRequest, final Map<String, String> params) {

        RequestFuture<JSONArray> future = RequestFuture.newFuture();

        JsonArrayRequest request = new JsonArrayRequest(action, url, jsonRequest, future, future) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

        };

        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());
        try {
            JSONArray response = future.get(2, TimeUnit.MINUTES);
            return new VolleyHelper.VolleyResponse<JSONArray>(true, response);
        } catch (Exception e) {
            return buildErrorMessage(e.getMessage());
        }
    }

    public VolleyHelper.VolleyResponse<String> loadFutureStringObject(String url, String keyTrack, int action, final Map<String, String> params) {
        RequestFuture<String> future = RequestFuture.newFuture();
        StringRequest request = new StringRequest(action, url, future, future) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

            @Override
            public String getBodyContentType() {
                return body.trim().isEmpty() ?
                        super.getBodyContentType() : "application/json; charset=utf-8";
            }

            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                return super.getHeaders();
            }

            @Override
            public byte[] getBody() throws AuthFailureError {
                if (!body.trim().isEmpty())
                    return body.getBytes();
                return "".getBytes();
            }

        };
        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());


        String response = null;
        try {
            response = future.get(2, TimeUnit.MINUTES);
            if (!Utilities.isEmpty(response))
                return new VolleyHelper.VolleyResponse<>(true, response);
            else
                return buildErrorMessage("");
        } catch (InterruptedException e) {
            e.printStackTrace();
            return buildErrorMessage(e.getMessage());
        } catch (ExecutionException e) {
            e.printStackTrace();
            return buildErrorMessage(e.getMessage());
        } catch (TimeoutException e) {
            e.printStackTrace();
            return buildErrorMessage(e.getMessage());
        }


    }

    public void loadStringObject(String url, String keyTrack, int action, final Map<String, String> params, final Response.Listener<String> volleyCallbackResponse, Response.ErrorListener volleyErrorResponse) {

        StringRequest request = new StringRequest(action, url, new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                volleyCallbackResponse.onResponse(response);
            }
        }, volleyErrorResponse) {
            @Override
            protected Map<String, String> getParams() throws AuthFailureError {
                if (params != null)
                    return params;
                return super.getParams();
            }

            @Override
            public String getBodyContentType() {
                return body.trim().isEmpty() ?
                        super.getBodyContentType() : "application/json; charset=utf-8";
            }

            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                return super.getHeaders();
            }

            @Override
            public byte[] getBody() throws AuthFailureError {
                if (!body.trim().isEmpty())
                    return body.getBytes();
                return super.getBody();
            }

        };
        request.setRetryPolicy(new DefaultRetryPolicy(300000, 0, DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        ApplicationController.getInstance().addToRequestQueue(request, keyTrack + Calendar.getInstance().getTimeInMillis());
    }

    String body = "";

    public void loadStringObject(String url, String keyTrack, int action, String body, final Map<String, String> params, Response.Listener<String> volleyCallbackResponse, Response.ErrorListener volleyErrorResponse) {
        this.body = body;
        loadStringObject(url, keyTrack, action, params, volleyCallbackResponse, volleyErrorResponse);
    }

    public VolleyHelper.VolleyResponse<String> loadFutureStringObject(String url, String keyTrack, int action, String body, final Map<String, String> params) {
        this.body = body;
        return loadFutureStringObject(url, keyTrack, action, params);
    }


    public interface VolleyResponseInterface<T> {
        void onResponse(T t);

        void onGotoError(JSONObject error);

        void onError(String error);
    }

    public VolleyResponse buildErrorMessage(String errorMessage) {
        VolleyResponse volleyResponse = new VolleyResponse(false, null);
        volleyResponse.setErrorMessage(errorMessage);
        return volleyResponse;
    }

    public class VolleyResponse<T> {
        private boolean isSuccess = false;
        private T response;
        private String errorMessage;

        public String getErrorMessage() {
            return this.errorMessage;
        }

        public VolleyHelper.VolleyResponse setErrorMessage(String errorMessage) {
            this.errorMessage = errorMessage;
            return this;
        }

        public VolleyResponse(boolean isSuccess, T response) {
            this.isSuccess = isSuccess;
            this.response = response;
        }

        public boolean isSuccess() {
            return this.isSuccess;
        }

        public VolleyHelper.VolleyResponse setSuccess(boolean success) {
            this.isSuccess = success;
            return this;
        }

        public T getResponse() {
            return this.response;
        }

        public VolleyHelper.VolleyResponse setResponse(T response) {
            this.response = response;
            return this;
        }
    }


}
```
I really hope this helps. <br>
Thanks.
