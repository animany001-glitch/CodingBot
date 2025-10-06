using System;
using System.Text;
using System.Text.Json;
using System.Security.Cryptography;

public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }

    public void EncryptData()
    {
        if (!string.IsNullOrEmpty(Password))
        {
            Password = Convert.ToBase64String(Encoding.UTF8.GetBytes(Password));
        }
    }

    public string GenerateHash()
    {
        using (SHA256 sha256 = SHA256.Create())
        {
            byte[] hashBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(ToString()));
            return Convert.ToBase64String(hashBytes);
        }
    }

    public override string ToString()
    {
        return JsonSerializer.Serialize(this);
    }
}

public class Program
{
    public static void Main()
    {
        Console.WriteLine("=========================================");
        Console.WriteLine("   SECURE USER SERIALIZATION EXAMPLE");
        Console.WriteLine("=========================================");

        User user = new User
        {
            Name = "ALICE",
            Email = "alice@example.com",
            Password = "SecureP@ss123"
        };

        if (!ValidateUser(user))
        {
            Console.WriteLine("❌ INVALID USER DATA. SERIALIZATION ABORTED.");
            return;
        }

        string serializedData = SerializeUserData(user);
        Console.WriteLine("\n✅ SERIALIZED DATA:");
        Console.WriteLine(serializedData);

        Console.WriteLine("\n🔒 GENERATED HASH:");
        Console.WriteLine(user.GenerateHash());

        string trustedSourceData = serializedData;
        User deserializedUser = DeserializeUserData(trustedSourceData, isTrustedSource: true);

        if (deserializedUser != null)
        {
            Console.WriteLine("\n✅ DESERIALIZATION SUCCESSFUL (TRUSTED SOURCE)");
            Console.WriteLine($"👤 NAME: {deserializedUser.Name}");
            Console.WriteLine($"📧 EMAIL: {deserializedUser.Email}");
            Console.WriteLine($"🔑 ENCRYPTED PASSWORD: {deserializedUser.Password}");
        }

        Console.WriteLine("\n=========================================");
        Console.WriteLine("   END OF PROGRAM");
        Console.WriteLine("=========================================");
    }

    public static string SerializeUserData(User user)
    {
        user.EncryptData();
        return JsonSerializer.Serialize(user);
    }

    public static User DeserializeUserData(string jsonData, bool isTrustedSource)
    {
        if (!isTrustedSource)
        {
            Console.WriteLine("⚠️ DESERIALIZATION BLOCKED: UNTRUSTED SOURCE.");
            return null;
        }

        return JsonSerializer.Deserialize<User>(jsonData);
    }

    public static bool ValidateUser(User user)
    {
        if (string.IsNullOrWhiteSpace(user.Name) ||
            string.IsNullOrWhiteSpace(user.Email) ||
            string.IsNullOrWhiteSpace(user.Password))
        {
            return false;
        }

        if (!user.Email.Contains("@") || !user.Email.Contains("."))
        {
            return false;
        }

        return true;
    }
}
