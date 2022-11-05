```
using Microsoft.Azure.Management.Fluent;
using Microsoft.Azure.Management.ResourceManager.Fluent;
using System.Net.Http.Headers;
using System.Text;
using System.Text.Json;

public class Program
{
    public class OAuth2
    {
        public string? access_token { get; set; }
    }

    public static async Task Main(string[] args)
    {
        Console.WriteLine("Start: " + DateTime.Now);

        var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(
            clientId: "75afc8e9-f297-4ba4-8b5b-5ce3495258a1",
            clientSecret: "YXe8Q~SKQ6_zM1FvDDXbhx7zxWoKlvrMDIz1Pb6G",
            tenantId: "16b3c013-d300-468d-ac64-7eda0820b6d3",
            environment: AzureEnvironment.AzureGlobalCloud
        );

        /* ************************* Iterate: Subscription >> Resource Group >> Date */

        foreach (string subscription in new string[] { "ed7eaf77-d411-484b-92e6-5cba0b6d8098" }) /* Iterate: Subscription */
        {
            var azure = Azure
                .Authenticate(credentials)
                .WithSubscription(subscriptionId: subscription)
                ;

            foreach (var resourceGroup in azure.ResourceGroups.List()) /* Iterate: Resource Group */
            {
                Console.WriteLine(" Processing: " + resourceGroup.Id + " >> Getting Token");

                /* ************************* Get Token */

                string token;

                using (HttpClient client = new HttpClient())
                {
                    FormUrlEncodedContent content = new FormUrlEncodedContent(new[]{
                        new KeyValuePair<string, string>("grant_type", "client_credentials"),
                        new KeyValuePair<string, string>("client_id", "75afc8e9-f297-4ba4-8b5b-5ce3495258a1"),
                        new KeyValuePair<string, string>("client_secret", "YXe8Q~SKQ6_zM1FvDDXbhx7zxWoKlvrMDIz1Pb6G"),
                        new KeyValuePair<string, string>("resource", "https://management.azure.com/")
                    });

                    content.Headers.ContentType = new MediaTypeHeaderValue("application/x-www-form-urlencoded");

                    HttpResponseMessage response = await client.PostAsync(requestUri: new Uri("https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token"), content: content);

                    OAuth2? oauth2 = JsonSerializer.Deserialize<OAuth2>(response.Content.ReadAsStringAsync().Result);

                    token = oauth2.access_token;
                }

                for (var date = DateTime.Parse("11/1/2022"); date <= DateTime.Parse("11/3/2022"); date = date.AddDays(1)) /* Iterate: Date */
                {
                    Console.WriteLine(" Processing: " + resourceGroup.Id + " >> " + date.ToString("MMMM dd yyyy"));

                    using (HttpClient client = new HttpClient())
                    {
                        client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                        HttpRequestMessage request = new HttpRequestMessage(
                            method: new HttpMethod("POST"),
                            requestUri: "https://management.azure.com/" + resourceGroup.Id + "/providers/Microsoft.CostManagement/query?api-version=2021-10-01");

                        //request.Headers.TryAddWithoutValidation("Authorization", "Bearer " + token);
                        request.Content = new StringContent(
                            content: "{'dataset':{'aggregation':{'totalCost':{'function':'Sum','name':'PreTaxCost'}},'granularity':'Daily','grouping':[{'name':'ResourceGroupName','type':'Dimension'},{'name':'ResourceType','type':'Dimension'},{'name':'ResourceId','type':'Dimension'},{'name':'ResourceLocation','type':'Dimension'},{'name':'MeterCategory','type':'Dimension'},{'name':'MeterSubCategory','type':'Dimension'},{'name':'Meter','type':'Dimension'},{'name':'ServiceName','type':'Dimension'},{'name':'PartNumber','type':'Dimension'},{'name':'PricingModel','type':'Dimension'},{'name':'ChargeType','type':'Dimension'},{'name':'ReservationName','type':'Dimension'},{'name':'Frequency','type':'Dimension'}]},'timePeriod':{'from':'" + date + "','to':'" + date + "'},'timeframe':'Custom','type':'Usage'}",
                            encoding: Encoding.UTF8,
                            mediaType: "application/json");

                        HttpResponseMessage response = client.SendAsync(request).Result;

                        Console.WriteLine(response);
                    }
                }
            }
        }
        Console.WriteLine("End: " + DateTime.Now);
    }
}
```
