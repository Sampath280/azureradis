# azureradis



```c
using StackExchange.Redis;
using System;
using System.Linq;
using System.Threading.Tasks;

public class RedisService
{
    private readonly IConnectionMultiplexer _connectionMultiplexer;
    private readonly IDatabase _redisDatabase;

    public RedisService(IConnectionMultiplexer connectionMultiplexer)
    {
        _connectionMultiplexer = connectionMultiplexer;
        _redisDatabase = _connectionMultiplexer.GetDatabase();
    }

    public async Task SetKeyAsync(string key, string value)
    {
        Console.WriteLine($"Setting key: {key} with value: {value}");
        bool setResult = await _redisDatabase.StringSetAsync(key, value);
        Console.WriteLine($"Set result: {setResult}");
    }

    public async Task DeleteAllKeysForProfileAsync(string profileId)
    {
        var endpoint = _connectionMultiplexer.GetEndPoints().First();
        var server = _connectionMultiplexer.GetServer(endpoint);
        var pattern = $"*{profileId}*";

        // Use SCAN instead of KEYS for better performance in production environments
        var keys = server.Keys(database: _redisDatabase.Database, pattern: pattern);

        Console.WriteLine($"Found {keys.Count()} keys matching pattern '{pattern}'.");

        foreach (var key in keys)
        {
            Console.WriteLine($"Deleting key: {key}");
            await _redisDatabase.KeyDeleteAsync(key);
        }
    }
}

public static class Program
{
    public static async Task Main(string[] args)
    {
        var redisConnectionString = "your_redis_connection_string"; // Replace with your Redis connection string
        var connectionMultiplexer = await ConnectionMultiplexer.ConnectAsync(redisConnectionString);
        var redisService = new RedisService(connectionMultiplexer);

        // Example of setting keys
        await redisService.SetKeyAsync("user:007:profile", "User Profile Data for 007");
        await redisService.SetKeyAsync("user:007:settings", "User Settings Data for 007");

        // Example of deleting keys
        string profileId = "007";
        await redisService.DeleteAllKeysForProfileAsync(profileId);

        Console.WriteLine("Operation completed. Press any key to exit.");
        Console.ReadKey();
    }
}
```
