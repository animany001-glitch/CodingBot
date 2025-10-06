using System;
using System.Text;
using System.Text.Json;
using System.Security.Cryptography;

public class User
{
    // Properties use PascalCase
    public string Name { get; set; }
    public string Email { get; set; }
    public string Password { get; set; }

    // Encrypt password using Base64 encoding
    public void EncryptData()
    {
        if (!string.IsNullOrEmpty(Password))
        {
            Password = Convert.ToBase64String(Encoding.UTF8.GetBytes(Password));
        }
    }

    // Generate a SHA256 hash for the serialized object
    public string GenerateHash()
    {
        using (SHA256 sha256 = SHA256.Create())
        {
            byte[] hashBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(ToString()));
            return Convert.ToBase64String(hashBytes);
        }
    }

    // Convert the object into a JSON string
    public override string ToString()
    {
        return JsonSerializer.Serialize(this);
    }
}

public class Program
{
    public static void Main()
    {
        Console.WriteLine("=== Secure User Serialization Example ===");

        // Create user object
        User user = new User
        {
            Name = "Alice",
            Email = "alice@example.com",
            Password = "SecureP@ss123"
        };

        // Step 1: Validate user input
        if (!ValidateUser(user))
        {
            Console.WriteLine("Invalid user data. Serialization aborted.");
            return;
        }

        // Step 2: Serialize user data securely
        string serializedData = SerializeUserData(user);
        Console.WriteLine("\nSerialized Data:\n" + serializedData);

        // Step 3: Demonstrate trusted deserialization
        string trustedSourceData = serializedData;
        User deserializedUser = DeserializeUserData(trustedSourceData, isTrustedSource: true);

        if (deserializedUser != null)
        {
            Console.WriteLine("\nDeserialization successful for trusted source.");
            Console.WriteLine($"User Name: {deserializedUser.Name}");
            Console.WriteLine($"User Email: {deserializedUser.Email}");
            Console.WriteLine($"Encrypted Password: {deserializedUser.Password}");
        }
    }

    // Serialize user object into JSON string
    public static string SerializeUserData(User user)
    {
        user.EncryptData();
        return JsonSerializer.Serialize(user);
    }

    // Deserialize JSON data back to user object (only if trusted)
    public static User DeserializeUserData(string jsonData, bool isTrustedSource)
    {
        if (!isTrustedSource)
        {
            Console.WriteLine("⚠️ Deserialization blocked: Untrusted source.");
            return null;
        }

        return JsonSerializer.Deserialize<User>(jsonData);
    }

    // Validate name, email, and password before serialization
    public static bool ValidateUser(User user)
    {
        if (string.IsNullOrWhiteSpace(user.Name) ||
            string.IsNullOrWhiteSpace(user.Email) ||
            string.IsNullOrWhiteSpace(user.Password))
        {
            return false;
        }

        // Basic email check
        if (!user.Email.Contains("@") || !user.Email.Contains("."))
        {
            return false;
        }

        return true;
    }
}
