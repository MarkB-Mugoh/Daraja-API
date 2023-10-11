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

Use M-Pesa Express (https://developer.safaricom.co.ke/APIs/MpesaExpressSimulate)  to make an SDK Push
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

## Step 4: Make API Requests

With the access token obtained, you can now make requests to the Safaricom Daraja API.

```csharp
// Replace with your actual access token
string accessToken = "YourAccessToken";
string darajaApiUrl = "https://api.safaricom.co.ke/daraja/";

using (HttpClient client = new HttpClient())
{
    var request = new HttpRequestMessage(HttpMethod.Get, darajaApiUrl + "your-endpoint-here");
    request.Headers.Add("Authorization", $"Bearer {accessToken}");
    // Add any necessary request parameters here

    HttpResponseMessage response = await client.SendAsync(request);

    if (response.IsSuccessStatusCode)
    {
        string responseContent = await response Content.ReadAsStringAsync();
        Console.WriteLine(responseContent);
        // Process the API response as needed.
    }
    else
    {
        Console.WriteLine($"Error: {response.StatusCode}");
    }
}
```

Make sure to replace "YourAccessToken" with the actual access token obtained earlier and use the appropriate Daraja API endpoint URL and request parameters for your specific integration.

This comprehensive guide outlines the process for integrating the Safaricom Daraja API into your ASP.NET application using C#. You can build upon this foundation to create the specific features you need for your application.

Remember to refer to the [official Safaricom Daraja API documentation](https://developer.safaricom.co.ke/daraja/apis/postman-collection) for detailed API specifications and additional resources.
```

In the above Markdown document, I've added a placeholder for the video link (replace `"https://www.youtube.com/watch?v=your-video-link"` with the actual URL) and a reference to the [official Safaricom Daraja API documentation](https://developer.safaricom.co.ke/daraja/apis/postman-collection). You can replace the placeholders with the actual video link and documentation URL.
