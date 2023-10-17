**Step 10: Initiating a Business to Customer (B2C) Payment Request**

In this step, we will demonstrate how to initiate a Business to Customer (B2C) payment request using the Safaricom API. This process allows you to send payments to individual customers from your business account. Ensure that you replace the placeholder values with your actual credentials and data.

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

        // Safaricom B2C Payment Request endpoint
        string b2cUrl = "https://sandbox.safaricom.co.ke/mpesa/b2c/v3/paymentrequest";

        // Your credentials
        string initiatorName = "YourInitiatorName";
        string securityCredential = "YourSecurityCredential";
        string businessShortCode = "YourBusinessShortCode"; // Part A code in the credentials
        string phone = "RecipientPhoneNumber"; // Replace with the recipient's phone number
        string amountSend = "AmountToSend"; // Replace with the amount to send
        string commandId = ""; // Replace with the desired command ID: SalaryPayment, BusinessPayment, PromotionPayment
        string remarks = "Umeskia Withdrawal";
        string queueTimeOutUrl = "YourQueueTimeOutURL"; // Replace with your QueueTimeOut URL
        string resultUrl = "YourResultURL"; // Replace with your Result URL
        string occasion = "Online Payment";

        using (HttpClient client = new HttpClient())
        {
            // Set request headers
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

            // Prepare B2C request data
            var b2cRequestData = new
            {
                InitiatorName = initiatorName,
                SecurityCredential = securityCredential,
                CommandID = commandId,
                Amount = amountSend,
                PartyA = businessShortCode,
                PartyB = phone,
                Remarks = remarks,
                QueueTimeOutURL = queueTimeOutUrl,
                ResultURL = resultUrl,
                Occasion = occasion
            };

            // Serialize request data to JSON
            string jsonData = Newtonsoft.Json.JsonConvert.SerializeObject(b2cRequestData);
            HttpContent content = new StringContent(jsonData, Encoding.UTF8, "application/json");

            // Send the B2C payment request
            var response = await client.PostAsync(b2cUrl, content);

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
---

**Where to Get Credentials:**

1. Go to the [Business to Customer API](https://developer.safaricom.co.ke/APIs/BusinessToCustomer).
2. Select your app in the Business To Customer API.
3. Press "Test Credentials" at the very bottom.

**Command Id:**

- `SalaryPayment`: Supports sending money to both registered and unregistered M-Pesa customers.
- `BusinessPayment`: A normal business to customer payment, supports only M-PESA registered customers.
- `PromotionPayment`: A promotional payment to customers. The M-PESA notification message is a congratulatory message. Supports only M-PESA registered customers.

**Important Information:**

- **Initiator Username:** This is the API operator's username as set on the portal when the user was created. For Sandbox users, the username is already created and assigned to them and is available on the test credentials page as `InitiatorName`.

- **Initiator Password:** This is the password assigned to the API operator after being created by the Business Administrator. For Sandbox users, this is available as `InitiatorPassword` on the test credentials page. Note: the password should be limited to specific special characters such as '#', '&', '%', and '$'. Other characters might cause issues, and the password may not be accepted. In addition, '@' is not a special character on M-Pesa; it is treated as a normal character.

- **Public Key Certificate:** This is the certificate used to encrypt the Initiator's plaintext password for use in the API calls. This is provided for both Sandbox and Production clients on the portal. You need to learn how to encrypt using your API language to be able to make API calls or find a way to encrypt beforehand and set the password as a static variable on the API call. The test credentials section offers the capability to encrypt your password.

**Note:**

- For you to use this API in production, you are required to apply for a Bulk Disbursement Account and get a Short code. You cannot make this payment from a Pay Bill or Buy Goods (Till Number). To apply for a Bulk disbursement account, follow this [link](https://www.safaricom.co.ke/business/sme/m-pesa-payment-solutions).

