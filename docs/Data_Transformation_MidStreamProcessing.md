using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using System;
using System.Threading.Tasks;

namespace Event_Hub_Message_Generator
{
    public class Function1
    {
        [FunctionName("Function1")]
        public async Task Run(
            [TimerTrigger("*/1 * * * *")] TimerInfo theTimer,
            [EventHub("dest", Connection = "EventHubConnectionAppSetting")] IAsyncCollector<string> theEventHub,
            ILogger theLogger)
        {
            string output = "{\"rows\":[{\"id\":"+Guid.NewGuid()+ "},{\"id\":"+Guid.NewGuid()+"},{\"id\":"+Guid.NewGuid()+"}]}";

            theLogger.LogInformation(output);

            await theEventHub.AddAsync(output);
        }
    }
}
