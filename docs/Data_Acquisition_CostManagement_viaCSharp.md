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

        string clientid = "{YOUR CLIENT ID}",
            clientsecret = "{YOUR CLIENT SECRET}",
            tenantid = "{YOUR TENANT ID}",
            startDate = "11/4/2022",
            endDate = "11/5/2022";

        string[] subscriptions = new string[] { "{YOUR SUBSCRIPTION ID}", "{YOUR SUBSCRIPTION ID}" };

        var credentials = SdkContext.AzureCredentialsFactory.FromServicePrincipal(
            clientId: clientid,
            clientSecret: clientsecret,
            tenantId: tenantid,
            environment: AzureEnvironment.AzureGlobalCloud
        );

        /* ************************* Iterate Subscriptions */

        foreach (string subscription in new string[] { "ed7eaf77-d411-484b-92e6-5cba0b6d8098" })
        {
            var azure = Azure
                .Authenticate(credentials)
                .WithSubscription(subscriptionId: subscription)
                ;

            /* ************************* Iterate Resource Groups */

            foreach (var resourceGroup in azure.ResourceGroups.List())
            {
                /* ************************* Get Token */

                string token;

                using (HttpClient client = new HttpClient())
                {
                    FormUrlEncodedContent content = new FormUrlEncodedContent(new[]{
                        new KeyValuePair<string, string>("grant_type", "client_credentials"),
                        new KeyValuePair<string, string>("client_id", clientid),
                        new KeyValuePair<string, string>("client_secret", clientsecret),
                        new KeyValuePair<string, string>("resource", "https://management.azure.com/")
                    });

                    content.Headers.ContentType = new MediaTypeHeaderValue("application/x-www-form-urlencoded");

                    HttpResponseMessage response = await client.PostAsync(requestUri: new Uri("https://login.microsoftonline.com/16b3c013-d300-468d-ac64-7eda0820b6d3/oauth2/token"), content: content);

                    OAuth2? oauth2 = JsonSerializer.Deserialize<OAuth2>(response.Content.ReadAsStringAsync().Result);

                    token = oauth2.access_token;
                }

                /* ************************* Iterate Dates */

                for (var date = DateTime.Parse(startDate); date <= DateTime.Parse(endDate); date = date.AddDays(1))
                {
                    Console.WriteLine(Environment.NewLine + resourceGroup.Id + " >> " + date.ToString("MMMM dd yyyy"));

                    /* ************************* Request Cost Management Data */

                    using (HttpClient client = new HttpClient())
                    {
                        client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                        HttpRequestMessage request = new HttpRequestMessage(
                            method: new HttpMethod("POST"),
                            requestUri: "https://management.azure.com/" + resourceGroup.Id + "/providers/Microsoft.CostManagement/query?api-version=2021-10-01");

                        request.Content = new StringContent(
                            content: "{'dataset':{'aggregation':{'totalCost':{'function':'Sum','name':'PreTaxCost'}},'granularity':'Daily','grouping':[{'name':'ResourceGroupName','type':'Dimension'},{'name':'ResourceType','type':'Dimension'},{'name':'ResourceId','type':'Dimension'},{'name':'ResourceLocation','type':'Dimension'},{'name':'MeterCategory','type':'Dimension'},{'name':'MeterSubCategory','type':'Dimension'},{'name':'Meter','type':'Dimension'},{'name':'ServiceName','type':'Dimension'},{'name':'PartNumber','type':'Dimension'},{'name':'PricingModel','type':'Dimension'},{'name':'ChargeType','type':'Dimension'},{'name':'ReservationName','type':'Dimension'},{'name':'Frequency','type':'Dimension'}]},'timePeriod':{'from':'" + date + "','to':'" + date + "'},'timeframe':'Custom','type':'Usage'}",
                            encoding: Encoding.UTF8,
                            mediaType: "application/json");

                        HttpResponseMessage response = client.Send(request);

                        Console.WriteLine(response.Content.ReadAsStringAsync().GetAwaiter().GetResult());
                    }
                }
            }
        }
        Console.WriteLine("End: " + DateTime.Now);
    }
}
```
