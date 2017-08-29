## ServiceStack.Azure

ServiceStack.Azure package provides support to Azure ServiceBus and Azure Blob Storage. All features are incapsulated in single ServiceStack.Azure package. To install package run from NuGet


    PM> Install-Package ServiceStack.Azure

ServiceStack.Azure includes implementation of the following ServiceStack providers:

- AzureBlobVirtualPathProvider - Virtual file system based on Azure Blob Storage
- AzureTableCacheClient - Cache client over Azure Table Storage
- ServiceBusMqServer - [MQ Server](http://docs.servicestack.net/messaging) for invoking ServiceStack Services via Azure ServiceBus


### ServiceBusMqServer

The code to configure and start an ServiceBus MQ Server is similar to other MQ Servers:

    container.Register<IMessageService>(c => new ServiceBusMqServer(ConnectionString));

    var mqServer = container.Resolve<IMessageService>();
    mqServer.RegisterHandler<ServiceDto>(ExecuteMessage);
    mqServer.Start();

Where ConnectionString is connection string to Service Bus, how to obtain it from Azure Portal you can find in [Get Started with Service Bus queues](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-dotnet-get-started-with-queues) article

When an MQ Server is registered, ServiceStack automatically publishes Requests accepted on the "One Way" pre-defined route to the registered MQ broker. The message is later picked up and executed by a Message Handler on a background Thread.



## Virtual FileSystem backed by Azure Blob Storage

You can use an Azure Blob Storage Container to serve website content with the **AzureBlobVirtualPathProvider**.

    public class AppHost : AppHostBase
    {
        public override void Configure(Container container)
        {
            //All Razor Views, Markdown Content, imgs, js, css, etc are served from an Azure Blob Storage container

            //get azure storage account from Azure Storage connection string
            // or you can use DevelopmentStorageAccount for testing purposes    
            //var storageAccount = CloudStorageAccount.DevelopmentStorageAccount;
            var storageAccount = CloudStorageAccount.Parse(connectionString);

            //containerName  is the name of Azure Storage Blob container
            var blobContainer = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
            blobContainer.CreateIfNotExists();

            VirtualFiles = new AzureBlobVirtualPathProvider(blobContainer);
        }

        public override List<IVirtualPathProvider> GetVirtualFileSources()
        {
            //Add Azure Blob Container as lowest priority Virtual Path Provider 
            var pathProviders = base.GetVirtualFileSources();
            pathProviders.Add(VirtualFiles);
            return pathProviders;
        }
    }


## Caching support with Azure Table Storage

The AzureTableCacheClient implements [ICacheClientExteded](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/ICacheClientExtended.cs) and  
[IRemoveByPattern](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/IRemoveByPattern.cs) using Azure Table Storage.  This provider is implemented as "last writer wins", so concurrent writes to a single cache entry are discouraged.

    public class AppHost : AppHostBase
    {
        public override void Configure(Container container)
        {
            string cacheConnStr = "UseDevelopmentStorage=true;";
            container.Register<ICacheClient>(new AzureTableCacheClient(cacheConnStr));
        }
    }

