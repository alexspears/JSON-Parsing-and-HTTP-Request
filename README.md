JSON-Parsing-and-HTTP-Request
=============================

Parse strings from a database using HTTP requests and JSON parsing while passing requested strings between activities.


package info.androidhive.jsonparsing;

import java.util.ArrayList;
import java.util.HashMap;

import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;

import android.app.ListActivity;
import android.app.ProgressDialog;
import android.content.Intent;
import android.os.AsyncTask;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.AdapterView;
import android.widget.AdapterView.OnItemClickListener;
import android.widget.ListAdapter;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.TextView;

public class MainActivity extends ListActivity {

	private ProgressDialog pDialog;

	// URL to get contacts JSON
	public static String url = "http://api.androidhive.info/contacts/";

	// JSON Node names
	private static final String TAG_CONTACTS = "contacts";
	private static final String TAG_ID = "id";
	private static final String TAG_NAME = "name";
	private static final String TAG_EMAIL = "email";
	private static final String TAG_ADDRESS = "address";
	private static final String TAG_GENDER = "gender";
	private static final String TAG_PHONE = "phone";
	private static final String TAG_PHONE_MOBILE = "mobile";
	private static final String TAG_PHONE_HOME = "home";
	private static final String TAG_PHONE_OFFICE = "office";

	// contacts JSONArray
	JSONArray contacts = null;

	// Hashmap for ListView
	ArrayList<HashMap<String, String>> contactList;

	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);

		contactList = new ArrayList<HashMap<String, String>>();

		ListView lv = getListView();

		// Listview on item click listener
		lv.setOnItemClickListener(new OnItemClickListener() {

			@Override
			public void onItemClick(AdapterView<?> parent, View view,
					int position, long id) {
				// getting values from selected ListItem
				String name = ((TextView) view.findViewById(R.id.name))
						.getText().toString();
				String cost = ((TextView) view.findViewById(R.id.email))
						.getText().toString();
				String description = ((TextView) view.findViewById(R.id.mobile))
						.getText().toString();

				// Launching new Activity on selecting single List Item
				Intent in = new Intent(MainActivity.this, SingleContactActivity.class);
				
				//getApplicationContext()
				
				// sending data to new activity
				in.putExtra(TAG_NAME, name);
				in.putExtra(TAG_EMAIL, cost);
				in.putExtra(TAG_PHONE_MOBILE, description);
				startActivity(in);

			}
		});

		// Calling async task to get json
		new GetContacts().execute();
	}

	/**
	 * Async task class to get json by making HTTP call
	 * */
	private class GetContacts extends AsyncTask<Void, Void, Void> {

		@Override
		protected void onPreExecute() {
			super.onPreExecute();
			// Showing progress dialog
			pDialog = new ProgressDialog(MainActivity.this);
			pDialog.setMessage("Please wait...");
			pDialog.setCancelable(false);
			pDialog.show();

		}

		@Override
		protected Void doInBackground(Void... arg0) {
			// Creating service handler class instance
			ServiceHandler sh = new ServiceHandler();

			// Making a request to url and getting response
			String jsonStr = sh.makeServiceCall(url, ServiceHandler.GET);

			Log.d("Response: ", "> " + jsonStr);

			if (jsonStr != null) {
				try {
					JSONObject jsonObj = new JSONObject(jsonStr);
					
					// Getting JSON Array node
					contacts = jsonObj.getJSONArray(TAG_CONTACTS);

					// looping through All Contacts
					for (int i = 0; i < contacts.length(); i++) {
						JSONObject c = contacts.getJSONObject(i);
						
						String id = c.getString(TAG_ID);
						String name = c.getString(TAG_NAME);
						String email = c.getString(TAG_EMAIL);
						String address = c.getString(TAG_ADDRESS);
						String gender = c.getString(TAG_GENDER);

						// Phone node is JSON Object
						JSONObject phone = c.getJSONObject(TAG_PHONE);
						String mobile = phone.getString(TAG_PHONE_MOBILE);
						String home = phone.getString(TAG_PHONE_HOME);
						String office = phone.getString(TAG_PHONE_OFFICE);

						// tmp hashmap for single contact
						HashMap<String, String> contact = new HashMap<String, String>();

						// adding each child node to HashMap key => value
						contact.put(TAG_ID, id);
						contact.put(TAG_NAME, name);
						contact.put(TAG_EMAIL, email);
						contact.put(TAG_PHONE_MOBILE, mobile);

						// adding contact to contact list
						contactList.add(contact);
					}
				} catch (JSONException e) {
					e.printStackTrace();
				}
			} else {
				Log.e("ServiceHandler", "Couldn't get any data from the url");
			}

			return null;
		}

		@Override
		protected void onPostExecute(Void result) {
			super.onPostExecute(result);
			// Dismiss the progress dialog
			if (pDialog.isShowing())
				pDialog.dismiss();
			/**
			 * Updating parsed JSON data into ListView
			 * */
			ListAdapter adapter = new SimpleAdapter(
					MainActivity.this, contactList,
					R.layout.list_item, new String[] { TAG_NAME, TAG_EMAIL, TAG_PHONE_MOBILE }, 
					new int[] { R.id.name,R.id.email, R.id.mobile });

			setListAdapter(adapter);
		}

	}

}
------------------------------------------------------------------------------------------------

package info.androidhive.jsonparsing;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.List;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URLEncodedUtils;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;

public class ServiceHandler {

	static String response = null;
	public final static int GET = 1;
	public final static int POST = 2;

	public ServiceHandler() {

	}

	/**
	 * Making service call
	 * @url - url to make request
	 * @method - http request method
	 * */
	public String makeServiceCall(String url, int method) {
		return this.makeServiceCall(url, method, null);
	}

	/**
	 * Making service call
	 * @url - url to make request
	 * @method - http request method
	 * @params - http request params
	 * */
	public String makeServiceCall(String url, int method,
			List<NameValuePair> params) {
		try {
			// http client
			DefaultHttpClient httpClient = new DefaultHttpClient();
			HttpEntity httpEntity = null;
			HttpResponse httpResponse = null;
			
			// Checking http request method type
			if (method == POST) {
				HttpPost httpPost = new HttpPost(url);
				// adding post params
				if (params != null) {
					httpPost.setEntity(new UrlEncodedFormEntity(params));
				}

				httpResponse = httpClient.execute(httpPost);

			} else if (method == GET) {
				// appending params to url
				if (params != null) {
					String paramString = URLEncodedUtils
							.format(params, "utf-8");
					url += "?" + paramString;
				}
				HttpGet httpGet = new HttpGet(url);

				httpResponse = httpClient.execute(httpGet);

			}
			httpEntity = httpResponse.getEntity();
			response = EntityUtils.toString(httpEntity);

		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (ClientProtocolException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		return response;

	}
}

---------------------------------------------------------------------------------------------------

package info.androidhive.jsonparsing;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;
import android.widget.TextView;

public class SingleContactActivity extends Activity {

	
	@Override
	    public void onCreate(Bundle savedInstanceState) {
	       super.onCreate(savedInstanceState);
	       this.setContentView(R.layout.single_list_item_view);
	        
	       // TextView txtContact = (TextView) findViewById(R.id.name);
	        
	       // Intent in = getIntent();
	        
	        //String name = in.getStringExtra("TAG_NAME");
	        //Log.d("intent","its: "+ name);
	        //txtContact.setText(name); 
	        
	        // getting attached intent data
	       //String contact = in.getStringExtra("TAG_NAME");
	       
	        // displaying selected contact name
	        //txtContact.setText("name");
	       Bundle extras = getIntent().getExtras(); 
	       if( extras != null ) {
	             String test = extras.getString("TAG_NAME");
	             Log.d("", "Value passed = " + test);
	       }
		}
	}





----------------------------------------------------------------------------------------------------------------
