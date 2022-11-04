```
using System.Net.Http.Headers;
using System.Text.Json;
using System.Text.Json.Serialization;

public class Program
{
    public class Token
    {
        public string? access_token { get; set; }
    }

    public static async Task Main(string[] args)
    {
        System.Console.WriteLine("Start: " + DateTime.Now);

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

            string result = response.Content.ReadAsStringAsync().Result;

            System.Console.WriteLine(result);

            Token? token = JsonSerializer.Deserialize<Token>(result);

            System.Console.WriteLine(token?.access_token);

            for (var d = DateTime.Parse("11/1/2022"); d <= DateTime.Parse("11/3/2022"); d = d.AddDays(1))
            {
                System.Console.WriteLine(" Processing: " + d.ToString("MMMM dd yyyy"));
            }

            System.Console.WriteLine("End: " + DateTime.Now);
        }
    }
}
```
