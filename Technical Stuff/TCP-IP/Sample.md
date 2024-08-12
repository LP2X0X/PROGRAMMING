- To implement a command-based communication system where the server can send messages to the client and vice versa, you can modify the server and client applications to continuously listen for user input and send/receive messages accordingly. Here's how you can do it:

**Server Application:**

```csharp
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;

class Server
{
    static TcpListener listener;
    static TcpClient client;
    static NetworkStream stream;

    static void Main()
    {
        // Set up TCP listener
        listener = new TcpListener(IPAddress.Any, 12345);
        listener.Start();
        Console.WriteLine("Server started. Waiting for connections...");

        // Accept client connection
        client = listener.AcceptTcpClient();
        Console.WriteLine("Client connected.");

        // Get client stream
        stream = client.GetStream();

        // Start separate thread to handle incoming messages from client
        Thread receiveThread = new Thread(ReceiveMessage);
        receiveThread.Start();

        // Main loop to send messages to client
        while (true)
        {
            Console.Write("Server: ");
            string messageToSend = Console.ReadLine();
            SendMessage(messageToSend);
        }
    }

    static void ReceiveMessage()
    {
        // Receive messages from client
        while (true)
        {
            byte[] receiveData = new byte[1024];
            int bytesReceived = stream.Read(receiveData, 0, receiveData.Length);
            string receivedMessage = Encoding.ASCII.GetString(receiveData, 0, bytesReceived);
            Console.WriteLine("Client: " + receivedMessage);
        }
    }

    static void SendMessage(string message)
    {
        // Send message to client
        byte[] sendData = Encoding.ASCII.GetBytes(message);
        stream.Write(sendData, 0, sendData.Length);
    }
}
```

**Client Application:**

```csharp
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;

class Client
{
    static TcpClient client;
    static NetworkStream stream;

    static void Main()
    {
        // Connect to server
        client = new TcpClient("127.0.0.1", 12345);
        Console.WriteLine("Connected to server.");

        // Get client stream
        stream = client.GetStream();

        // Start separate thread to handle incoming messages from server
        Thread receiveThread = new Thread(ReceiveMessage);
        receiveThread.Start();

        // Main loop to send messages to server
        while (true)
        {
            Console.Write("Client: ");
            string messageToSend = Console.ReadLine();
            SendMessage(messageToSend);
        }
    }

    static void ReceiveMessage()
    {
        // Receive messages from server
        while (true)
        {
            byte[] receiveData = new byte[1024];
            int bytesReceived = stream.Read(receiveData, 0, receiveData.Length);
            string receivedMessage = Encoding.ASCII.GetString(receiveData, 0, bytesReceived);
            Console.WriteLine("Server: " + receivedMessage);
        }
    }

    static void SendMessage(string message)
    {
        // Send message to server
        byte[] sendData = Encoding.ASCII.GetBytes(message);
        stream.Write(sendData, 0, sendData.Length);
    }
}
```

With these modifications, both the server and the client will continuously listen for user input and send/receive messages accordingly. When one side sends a message, the other side will receive and display it. This allows for bidirectional communication between the server and the client.