# Safaricom Daraja API Integration Guide
## What is M-Pesa Daraja API?
M-Pesa Daraja API is a powerful tool provided by Safaricom, Kenya's leading mobile network operator. It allows developers to interact with M-Pesa's mobile payment platform, enabling seamless transactions, account management, and other financial services.

## Step 1: Set up Your ASP.NET Project

. Ensure you have the necessary packages and libraries installed to handle HTTP requests and JSON data. You can add the `System.Net.Http` package to your project. Refer to the [official Microsoft documentation](https://docs.microsoft.com/en-us/dotnet/api/system.net.http?view=net-6.0) for more details.

   ```bash
   dotnet add package System.Net.Http --version 4.3.4
   ```
## step 2: Create a safaricom developer Account
Follow the link below to create an account : https://developer.safaricom.co.ke/

## step 3 : Create an APP in the safaricom dashboard
link : https://developer.safaricom.co.ke/MyApps

Which product(s) should be mapped to this app?  Click all checkboxes

## Ste 4: Choosing the API to use
we will use:
- Authorization (OAuth) : This API generates the tokens for authenticating your API calls. This is the first API you will engage with within the set of APIs available because all the other APIs require authentication information from this API to work.
- M-Pesa Express : Merchant initiated online payments
- Customer To Business Register URL
- 
## Step 5: Obtain the Access Token

To interact with the Safaricom Daraja API, you must obtain an access token from the OAuth API [https://developer.safaricom.co.ke/APIs/Authorization] after creating an app in the developers dashboard

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // Safaricom OAuth API URL from the developers dashboard
        string oauthUrl = "https://sandbox.safaricom.co.ke/oauth/v1/generate";

        // Your client ID and client secret fetched from your dashboard
        string clientId = "YourClientID";
        string clientSecret = "YourClientSecret";

        // Base64 encode your client ID and client secret 
        string base64Credentials = Convert.ToBase64String(Encoding.UTF8.GetBytes($"{clientId}:{clientSecret}"));

        using (HttpClient client = new HttpClient())
        {
            // Set the request parameters
            var request = new HttpRequestMessage(HttpMethod.Get, oauthUrl);
            request.Headers.Add("Authorization", $"Basic {base64Credentials}");
            request.Headers.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));

            HttpResponseMessage response = await client.SendAsync(request);

            if (response.IsSuccessStatusCode)
            {
                string responseContent = await response.Content.ReadAsStringAsync();
                Console.WriteLine(responseContent);
                // Parse the access token from the responseContent and store it for later use.
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
            }
        }
    }
}
```

In this code, replace "YourClientID" and "YourClientSecret" with your actual client ID and client secret from the daraja API.

Your response in this part should be your access token and the time bound of the token.

## Step 6: Parse the Access Token

Parse the access token from the response content and store it for subsequent API requests.

## step 7 : STK Push

Use M-Pesa Express (https://developer.safaricom.co.ke/APIs/MpesaExpressSimulate)  to make an SDK Push.

Lipa na M-PESA online API also known as M-PESA express (STK Push/NI push) is a Merchant/Business initiated C2B (Customer to Business) Payment.

Once we integrate with the API, we will be able to send a payment prompt on the customer's phone (Popularly known as STK Push Prompt) to our customer's M-PESA registered phone number requesting them to enter their M-PESA pin to authorize and complete payment.

This eliminates the challenge of having to remember business pay bill numbers and account numbers and allows customers to confirm the transaction by entering their M-PESA PIN on their mobile phones. 

For the business, this API enables us to preset all the correct info in the payment request and reduce the chances of wrong payments being performed to their systems. It is a C2B transaction, but the initiator is the organization instead of the customer. Since the organization has the option of presetting all required variables in the request before sending the request, this API has no Validation-Confirmation process like C2B API.

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;
using System.Collections.Generic;
using System.Net.Http.Headers;
using Newtonsoft.Json;

class Program
{
    static async Task Main(string[] args)
    {
        // Include your access token here
        string accessToken = "YourAccessToken";

        // Safaricom STK Push API endpoint and callback URL
        string stkPushUrl = "https://sandbox.safaricom.co.ke/mpesa/stkpush/v1/processrequest";
        string callbackUrl = "https://your-callback-url.com/callback";
         //Remember to edit the callback url to fit the application, this is where when each and every successful transaction will get a JSON response.

        // Business credentials
        string businessShortCode = ""; // shpuld provide from the business
        string passkey = ""; // shpuld provide from the business

        // Construct timestamp
        string timestamp = DateTime.Now.ToString("yyyyMMddHHmmss");

        // Construct password (base64 encoded)
        string password = Convert.ToBase64String(Encoding.UTF8.GetBytes(businessShortCode + passkey + timestamp));

        // STK Push parameters
        string phone = ""; // Phone number to receive the STK push e.g 254710777093
        string money = "1";
        string partyA = phone;
        string partyB = ""; e.g //254710777093
        string accountReference = ""; //e.g SoarmaxElite ltd
        string transactionDesc = "stkpush test";
        string amount = money;

        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            var stkPushData = new Dictionary<string, string>
            {
                { "BusinessShortCode", businessShortCode },
                { "Password", password },
                { "Timestamp", timestamp },
                { "TransactionType", "CustomerPayBillOnline" },
                { "Amount", amount },
                { "PartyA", partyA },
                { "PartyB", businessShortCode },
                { "PhoneNumber", partyA },
                { "CallBackURL", callbackUrl },
                { "AccountReference", accountReference },
                { "TransactionDesc", transactionDesc }
            };

            string jsonData = JsonConvert.SerializeObject(stkPushData);
            HttpContent content = new StringContent(jsonData, Encoding.UTF8, "application/json");

            var response = await client.PostAsync(stkPushUrl, content);

            if (response.IsSuccessStatusCode)
            {
                string responseContent = await response.Content.ReadAsStringAsync();
                var data = JsonConvert.DeserializeObject<dynamic>(responseContent);

                string checkoutRequestID = data.CheckoutRequestID;
                string responseCode = data.ResponseCode;

                if (responseCode == "0")
                {
                    Console.WriteLine($"The CheckoutRequestID for this transaction is: {checkoutRequestID}");
                }
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
            }
        }
    }
}

```
### handling the STK Push callback response to C# in our ASP.NET application and store the transaction details in a database
you can use the following code as an example:

```csharp
using System;
using System.IO;
using System.Net;
using System.Web.Script.Serialization;

class Program
{
    static void Main(string[] args)
    {
        HttpListener listener = new HttpListener();
        listener.Prefixes.Add("http://localhost:8080/"); // Adjust the URL as needed
        listener.Start();

        Console.WriteLine("Listening for STK Callback...");

        while (true)
        {
            HttpListenerContext context = listener.GetContext();
            HttpListenerRequest request = context.Request;
            HttpListenerResponse response = context.Response;

            if (request.HttpMethod == "POST")
            {
                // Read the STK Callback response
                using (Stream body = request.InputStream)
                {
                    using (StreamReader reader = new StreamReader(body, request.ContentEncoding))
                    {
                        string stkCallbackResponse = reader.ReadToEnd();
                        Console.WriteLine("Received STK Callback Response: " + stkCallbackResponse);

                        // Parse the JSON response
                        JavaScriptSerializer serializer = new JavaScriptSerializer();
                        dynamic data = serializer.Deserialize<dynamic>(stkCallbackResponse);

                        string MerchantRequestID = data["Body"]["stkCallback"]["MerchantRequestID"];
                        string CheckoutRequestID = data["Body"]["stkCallback"]["CheckoutRequestID"];
                        int ResultCode = data["Body"]["stkCallback"]["ResultCode"];
                        string Amount = data["Body"]["stkCallback"]["CallbackMetadata"]["Item"][0]["Value"];
                        string TransactionId = data["Body"]["stkCallback"]["CallbackMetadata"]["Item"][1]["Value"];
                        string UserPhoneNumber = data["Body"]["stkCallback"]["CallbackMetadata"]["Item"][4]["Value"];

                        // Check if the transaction was successful (ResultCode = 0)
                        if (ResultCode == 0)
                        {
                            // Store the transaction details in your database
                            // Here, you should use your database connection and perform an insert operation
                            // Example: InsertTransaction(MerchantRequestID, CheckoutRequestID, ResultCode, Amount, TransactionId, UserPhoneNumber);

                            // For this example, we'll just print the details
                            Console.WriteLine("Transaction Successful");
                            Console.WriteLine("MerchantRequestID: " + MerchantRequestID);
                            Console.WriteLine("CheckoutRequestID: " + CheckoutRequestID);
                            Console.WriteLine("Amount: " + Amount);
                            Console.WriteLine("TransactionId: " + TransactionId);
                            Console.WriteLine("UserPhoneNumber: " + UserPhoneNumber);
                        }
                        else
                        {
                            Console.WriteLine("Transaction Failed. ResultCode: " + ResultCode);
                        }
                    }
                }
            }

            response.Close();
        }
    }

    // Implement the method to insert transaction details into your database
    // Replace this with your actual database code
    static void InsertTransaction(string MerchantRequestID, string CheckoutRequestID, int ResultCode, string Amount, string TransactionId, string UserPhoneNumber)
    {
        // Implement your database insertion logic here
        Console.WriteLine("Inserting Transaction into Database");
        Console.WriteLine("MerchantRequestID: " + MerchantRequestID);
        Console.WriteLine("CheckoutRequestID: " + CheckoutRequestID);
        Console.WriteLine("Amount: " + Amount);
        Console.WriteLine("TransactionId: " + TransactionId);
        Console.WriteLine("UserPhoneNumber: " + UserPhoneNumber);
    }
}
```
### STK Callback Response Example

Below is an example of the JSON response from an STK (Lipa Na M-Pesa) Callback:

```json
{
  "Body": {
    "stkCallback": {
      "MerchantRequestID": "29637-66576708-1",
      "CheckoutRequestID": "ws_CO_19072023184338586768168060",
      "ResultCode": 0,
      "ResultDesc": "The service request is processed successfully.",
      "CallbackMetadata": {
        "Item": [
          {
            "Name": "Amount",
            "Value": 1.00
          },
          {
            "Name": "MpesaReceiptNumber",
            "Value": "RGJ6XCEJSK"
          },
          {
            "Name": "Balance"
          },
          {
            "Name": "TransactionDate",
            "Value": 20230719184207
          },
          {
            "Name": "PhoneNumber",
            "Value": 254710777093
          }
        ]
      }
    }
  }
}
```
## Step 8: STK Push transaction status 

Check the status of a Lipa Na M-Pesa Online Payment.

IS the transaction successful or did the user cancel or he doesn't have enough money in their account / Fuliza ?

```csharp
using System;
using System.Net.Http;
using System.Text;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // Include your access token here
        string accessToken = "YourAccessToken";

        // Safaricom STK Push Query API endpoint
        string stkQueryUrl = "https://sandbox.safaricom.co.ke/mpesa/stkpushquery/v1/query";

        // Business credentials
        string businessShortCode = "174379";
        string passkey = "bfb279f9aa9bdbcf158e97dd71a467cd2e0c893059b10f78e6b72ada1ed2c919";

        // Construct timestamp
        string timestamp = DateTime.Now.ToString("yyyyMMddHHmmss");

        // Construct password (base64 encoded)
        string password = Convert.ToBase64String(Encoding.UTF8.GetBytes(businessShortCode + passkey + timestamp));

        // The CheckoutRequestID obtained from the STK Push request
        string checkoutRequestID = "ws_CO_03072023054410314768168060";

        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Accept.Add(new System.Net.Http.Headers.MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            var stkQueryData = new
            {
                BusinessShortCode = businessShortCode,
                Password = password,
                Timestamp = timestamp,
                CheckoutRequestID = checkoutRequestID
            };

            string jsonData = Newtonsoft.Json.JsonConvert.SerializeObject(stkQueryData);
            HttpContent content = new StringContent(jsonData, Encoding.UTF8, "application/json");

            var response = await client.PostAsync(stkQueryUrl, content);

            if (response.IsSuccessStatusCode)
            {
                string responseContent = await response.Content.ReadAsStringAsync();
                dynamic data = Newtonsoft.Json.JsonConvert.DeserializeObject(responseContent);

                if (data.ResultCode != null)
                {
                    int resultCode = data.ResultCode;
                    string message = GetResultMessage(resultCode);
                    Console.WriteLine(message);
                }
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
            }
        }
    }

    static string GetResultMessage(int resultCode)
    {
        switch (resultCode)
        {
            case 1037:
                return "1037 Timeout in completing transaction";
            case 1032:
                return "1032 Transaction has been cancelled by the user";
            case 1:
                return "1 The balance is insufficient for the transaction";
            case 0:
                return "0 The transaction was successful";
            default:
                return "Unknown Result Code";
        }
    }
}
```

make sure to replace "YourAccessToken" with your actual access token and adjust other parameters as needed for your specific STK Push Query. The GetResultMessage method maps the ResultCode to the corresponding message based on the Safaricom Daraja API documentation.

## Step 9 : Customer To Business URL

Register URL API works hand in hand with Customer to Business (C2B) APIs and allows receiving payment notifications to your paybill. This API enables you to register the callback URLs via which you shall receive notifications for payments to your pay bill/till number.

There are two URLs required for Register URL API: Validation URL and Confirmation URL.

-Validation URL: This is the URL that is only used when a Merchant (Partner) requires to validate the details of the payment before accepting. For example, a bank would want to verify if an account number exists in their platform before accepting a payment from the customer.

-Confirmation URL:  This is the URL that receives payment notification once payment has been completed successfully on M-PESA.

NB: C2B Transaction Validation is an optional feature that needs to be activated on M-Pesa, the owner of the shortcode needs to make this request for activation by emailing us at apisupport@safaricom.co.ke or M-pesabusiness@safaricom.co.ke  if they need their transactions validated before execution.

### URL Requirements
- Use publicly available (Internet-accessible) IP addresses or domain names.
- All Production URLs must be HTTPS, on Sandbox you're allowed to simulate using HTTP.
- Avoid using keywords such as M-PESA, M-Pesa, M-Pesa, Safaricom, exe, exec, cmd, SQL, query, or any of their variants in either upper or lower cases in your URLs.
- Do not use public URL testers e.g. ngrok, mockbin, or requestbin, especially on production, they are also usually blocked by the API. Use your own application URLs and do not make them public or share them with any of your peers.

A comprehensive guide on this and a flowchart is available at : https://developer.safaricom.co.ke/APIs/CustomerToBusinessRegisterURL

### code for registering C2B URLs 

```csharp
using System;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Text;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        // Include your access token here
        string accessToken = "YourAccessToken";

        // Safaricom C2B Register URL endpoint
        string registerUrl = "https://sandbox.safaricom.co.ke/mpesa/c2b/v1/registerurl";

        // Your business short code
        string businessShortCode = "YourBusinessShortCode";

        // Confirmation and validation URLs
        string confirmationUrl = "https://your-callback-url.com/confirmation_url";  //Remeber to adjust this to fit our application
        string validationUrl = "https://your-callback-url.com/validation_url"; //Remeber to adjust this to fit our application


        using (HttpClient client = new HttpClient())
        {
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            var c2bRegistrationData = new
            {
                ShortCode = businessShortCode,
                ResponseType = "Completed",
                ConfirmationURL = confirmationUrl,
                ValidationURL = validationUrl
            };

            string jsonData = Newtonsoft.Json.JsonConvert.SerializeObject(c2bRegistrationData);
            HttpContent content = new StringContent(jsonData, Encoding.UTF8, "application/json");

            var response = await client.PostAsync(registerUrl, content);

            if (response.IsSuccessStatusCode)
            {
                string responseContent = await response.Content.ReadAsStringAsync();
                Console.WriteLine(responseContent);
            }
            else
            {
                Console.WriteLine($"Error: {response.StatusCode}");
            }
        }
    }
}
```
### Confirmation URL - code for handling the C2B callback confirmation response

```csharp
using System;
using System.IO;
using System.Net;
using System.Text;

class Program
{
    static void Main(string[] args)
    {
        HttpListener listener = new HttpListener();
        listener.Prefixes.Add("http://localhost:8080/"); // Adjust the URL as needed
        listener.Start();

        Console.WriteLine("Listening for C2B Callback...");

        while (true)
        {
            HttpListenerContext context = listener.GetContext();
            HttpListenerRequest request = context.Request;
            HttpListenerResponse response = context.Response;

            if (request.HttpMethod == "POST")
            {
                // Read the C2B Callback response
                using (Stream body = request.InputStream)
                {
                    using (StreamReader reader = new StreamReader(body, request.ContentEncoding))
                    {
                        string c2bCallbackResponse = reader.ReadToEnd();
                        Console.WriteLine("Received C2B Callback Response: " + c2bCallbackResponse);

                        // Log the callback response to a file
                        string logFile = "C2bPesaResponse.json";
                        File.AppendAllText(logFile, c2bCallbackResponse);

                        // Send a confirmation response
                        string confirmationResponse = "{ \"ResultCode\": 0, \"ResultDesc\": \"Confirmation Received Successfully\" }";
                        byte[] buffer = Encoding.UTF8.GetBytes(confirmationResponse);

                        response.ContentLength64 = buffer.Length;
                        Stream output = response.OutputStream;
                        output.Write(buffer, 0, buffer.Length);
                        output.Close();
                    }
                }
            }

            response.Close();
        }
    }
}

```
we listen for POST requests at the specified URL (http://localhost:8080/). When a POST request is received (which should be the C2B Callback response), it's logged to a file and a confirmation response is sent back to the Safaricom server.

Make sure to adjust the URL, log file name, and other details as needed for your specific C2B callback handling.
















Remember to refer to the [official Safaricom Daraja API documentation](https://developer.safaricom.co.ke/daraja/apis/postman-collection) for detailed API specifications and additional resources.
```

In the above Markdown document, I've added a placeholder for the video link (replace `"https://www.youtube.com/watch?v=your-video-link"` with the actual URL) and a reference to the [official Safaricom Daraja API documentation](https://developer.safaricom.co.ke/daraja/apis/postman-collection). You can replace the placeholders with the actual video link and documentation URL.
